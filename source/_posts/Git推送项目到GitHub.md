---
title: Git推送项目到GitHub
copyright: true
date: 2020-04-14 21:03:29
tags: 学习
---

使用Git推送项目到Github，这个功能非常常用，所以记录下来。

<!--more-->

### …or create a new repository on the command line

从本地仓库推项目到远程仓库

```
echo "# springboot_crud" >> README.md
git init  #初始化本地仓库
git add . #添加全部已经修改
git commit -m "first commit" #提交说明，将修改文件提到本地仓库
git remote add origin https://github.com/dengweiqiang/springboot_crud.git  #连接到远程仓库
git push -u origin master  #将本地代码强制推送到origin仓库中的master分支
```

### …or push an existing repository from the command line

从命令行推送现有的库

```
git remote add origin https://github.com/dengweiqiang/springboot_crud.git
git push -u origin master
```

