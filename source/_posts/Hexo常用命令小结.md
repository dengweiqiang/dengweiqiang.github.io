---
title: Hexo常用命令小结
copyright: true
date: 2020-04-14 21:08:37
tags: 工具
---

最近开始重新使用Hexo写博客，但是有一些常用的命令又时常想不起来，所以写下这篇文章总结一下Hexo常用的命令。

<!--more-->

### 常用命令

```shell
hexo n "新的博客"	#新建博客
hexo g & hexo s	   #编译并运行
hexo g & hexo d	   #编译并部署
------------------------------------
git branch -a      #查看所有分支
git push --set-upstream origin hexo_bak  #推送到指定分支
```



### 简写

```sh 
hexo n "我的博客" => hexo new "我的博客" #新建文章
hexo p => hexo publish
hexo g => hexo generate#生成
hexo s => hexo server #启动服务预览
hexo d => hexo deploy#部署
```

### 服务器

```shell
hexo server #Hexo 会监视文件变动并自动更新，您无须重启服务器。
hexo server -s #静态模式
hexo server -p 5000 #更改端口
hexo server -i 192.168.1.1 #自定义 IP

hexo clean #清除缓存 网页正常情况下可以忽略此条命令
hexo g #生成静态网页
hexo d #开始部署
```

### 设置文章摘要

```
以上是文章摘要 <!--more--> 以下是余下全文 
```

