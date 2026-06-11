---
title: "R studio创建Hugo网站，并部署到Github Pages"
date: 2026-04-29
---

**一、使用 R 搭建 Hugo 静态网站（完整步骤）:**       
*R使用的是4.3.3版本*      

1️⃣ 安装必要 R 包      
install.packages(c("blogdown", "xfun", "rmarkdown"), update = TRUE)
说明：        
blogdown：R 与 Hugo 的接口，管理网站生成      
xfun：提供一些辅助函数       
rmarkdown：支持 RMarkdown 文件渲染成网页

2️⃣ 安装 Hugo引擎（静态网站生成器）            
blogdown::install_hugo()       
说明：         
Hugo 是静态网站生成器，将 Markdown / RMarkdown 转为 HTML       
默认安装到系统路径，可直接使用

3️⃣ 创建网站项目            
第一次搭建，在电脑上新建一个空文件夹（比如blogdown_default）
setwd("c:/blogdown_default")          
blogdown::new_site()         
说明：           
setwd() 指定你要存放网站文件夹的位置        
new_site() 会在当前目录生成 Hugo 默认网站结构，         
核心文件夹作用：      
content/      # 网站内容（.Rmd / .md 文件）            
themes/       # 网站主题（皮肤、样式、布局）       
public/       # 编译后的成品网站（部署用）        
config.yaml   # 网站配置文件（改标题、菜单、主题）
     
Hugo + Blogdown 默认主题 = Lithium       
Lithium 主题默认导航只有 3 项：About（关于）GitHub（链接）Twitter（链接）     
更换主题：国内网不能用install_theme(),可以去https://github.com 下载ZIP，解压，放到themes文件夹下，修改config.yaml    
      
打开链接 → 点 Code → Download ZIP     
解压后把文件夹重命名（比如 papermod / blowfish）      
放到 themes/ 目录下     
修改 config.yaml 的 theme: 字段为文件夹名即可    
重启预览：blogdown::serve_site()    

## 注意：      
C:\blogdown_default\content\post 路径下的东西会自动出现在网站首页的文章列表里，
比如 my-first-post.md，网站首页就会显示这篇文章的标题 + 日期，点进去就能看全文。          
C:\blogdown_default\content\about.md：是「独立的页面」
它不会出现在首页的文章列表里，而是一个单独的页面：
访问方式：网站导航栏的「About」链接，或者直接访问 http://你的网址/about/

4️⃣ 启动本地网站预览
# 启动本地服务器，浏览器打开网站
blogdown::serve_site()         
说明：        
在RStudio 的右下面板可以实时看效果        
热更新：修改 RMarkdown 或 Markdown 文件后，浏览器会自动刷新

5️⃣ 网站更新
新建的网站文件夹里有个content 文件夹。这里就是更新网站内容的地方。你只需用记事本或RStudio，打开其中的.md 或.Rmd
文件，修改成自己的内容后保存，然后在R 里运行建站函数：
blogdown::build_site()

如果要发表新帖子，那么有两种方式最简单：         
• 方法一：将原有的.md 或.Rmd 拷贝粘贴，改一下文件名和文件内容
即可；              
• 方法二：在RSudio 代码窗口点击Addins – New Post，按提示填写
即可。           
写完保存，运行建站函数，上传。完毕。         
build_site()会清空旧的public/文件夹，重新编译所有内容，生成完整的静态网站文件      
            
            
            
new_site() → 创建网站骨架
content/ → 编辑文章和页面
serve_site() → 本地实时预览
build_site() → 生成最终 public/ 静态网站   
         
          
Rshiny  
# 1.环境准备  
安装shiny包       
# 2.创建shiny应用    
在hugo项目根目录 c:/blogdown_default/ 里，新建一个文件夹shiny-demo/  
在 shiny-demo/ 里新建 app.R文件  
本地运行shiny 
setwd("c:/blogdown_default/shiny-demo")            
shiny::runApp()
运行成功后，会弹出一个窗口，控制台会显示类似 http://127.0.0.1:7470 的地址，把这个地址复制下来。       
保持这个窗口 / 进程不关闭，否则 Shiny 服务会停止。        
# 3.在hugo中创建shiny页面     
在 Hugo 项目的 content/ 目录下，新建文件 shiny-page.md，更新shiny运行后的地址    
把页面添加到hugo导航栏    
# 4.启动hugo预览服务   
blogdown::serve_site()     
 
若需要把shiny工具做成浏览器访问的形式，需要先把shiny app部署到浏览器上                 


**二、管理代码(Git+Github)**    
Git 帮你管理代码版本，GitHub 当作远程仓库存放网站源文件。  
   
**Github创建仓库**   
登录 GitHub → 新建仓库    
Repository name: hugo-site    
不要勾选 Initialize with README / .gitignore / License     

获取仓库地址，例如：
https://github.com/sheila72xx/hugo-site.git   
新建仓库后，config.yaml中baseURL需要同步更新


**在本地初始化Git并关联Github**    
打开R studio Terminal(Tools,Terminal,new Terminal)   
输入：   
#初始化：git init    
#添加文件：git add .     
#提交：git commit -m "Initial commit"   
#连接Github仓库：git remote add origin https://github.com/sheila72xx/hugo-site.git   
#推送到Github:   
git branch -M main    
git push -u origin main    
    
    
 
**三、配置Github Actions 自动部署Hugo网站**    
GitHub Actions 就是一个“自动机器人”，帮你把 Hugo 的源文件生成最终网页。  

在R studio中 项目根目录创建目录.github/workflows/    
创建文件deploy.yml，内容示例:     
name: hugo-deploy

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v6
        with:
          hugo-version: '0.146.0'
      - name: Build
        run: hugo --minify
      - name: Deploy
        uses: peaceiris/actions-gh-pages@v6
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public      
            
在R studio Terminal提交并推送  
git add .github/workflows/deploy.yml
git commit -m "Add GitHub Actions workflow for Hugo deploy"
git push     
 
在 GitHub → Actions 可以看到自动构建状态：
绿色对号 ✅ 表示成功
红色 × 表示失败，需要查看报错      
 
以上步骤机器人会自动把网站生成好并放到 gh-pages。
 
 
 
**四、托管网站，开启Github Pages**      
GitHub Pages 就像是你的网站的“服务器”，帮你把 gh-pages 的静态网页展示出来  

打开 GitHub 仓库 → Settings → Pages
Source：选择 gh-pages branch 或 Actions 自动生成的分支
保存 → 几分钟后即可访问网站
  
  
    
**五、更新网站**      
更新内容后，在后端运行：   
git add .
git commit -m "更新文章：新文章标题"
git push origin main   
GitHub Actions 自动生成新的 public/ → 自动部署




