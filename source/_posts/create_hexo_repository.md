---
title: hexo的git仓库构建
date: 2019-02-27 16:00:00
updated: 2019-02-27 16:00:00
tags: 

---

# hexo的仓库构造

1. ## github创建新repositories

   进入github，登录账号后，在左侧Repositories预览栏中，点击New进入新建页，输入hexo作为Repository name，点击最下方Create repository即可。

   <!--more-->

2. ## 关联本地仓库

   1. 新建本地仓库文件

   2. `在仓库文件内`，按住Shift，点击鼠标右键，打开Powershell，输入cmd，进入命令行功能

   3. 初始化本地仓库
      `C:\Users\61961\Documents\Typora>git init
      Initialized empty Git repository in C:/Users/61961/Documents/Typora/.git/`

   4. 添加所有文件至本地仓库

        add .  <!--添加所有文件-->

      `C:\Users\61961\Documents\Typora>git add .`

   5. 提交所有文件至本地仓库

        -m “first create”  <!--带注释的提交命令-->
      `C:\Users\61961\Documents\Typora>git commit -m "first create"
      [master (root-commit) 02c6a64] first create
       1 file changed, 1 insertion(+)
       create mode 100644 demo.md`

   6. 关联本地仓库和远程仓库，将“repo_url”替换为新建的“hexo”的地址
      `C:\Users\61961\Documents\Typora>git remote add origin repo_url`
      如果出现

   7. 提交

      `C:\Users\61961\Documents\Typora>git push -u origin master
      Enumerating objects: 3, done.
      Counting objects: 100% (3/3), done.
      Writing objects: 100% (3/3), 214 bytes | 107.00 KiB/s, done.
      Total 3 (delta 0), reused 0 (delta 0)
      To https://github.com/g1204485703/hexo.git `
      `* [new branch]      master -> master
      Branch 'master' set up to track remote branch 'master' from 'origin'.`

   8. 

3. ## 