---
title: Typora+PicGo+阿里云OSS搭建私人图床
copyright: true
date: 2021-06-27 01:39:22
tags: 随笔
---

最近又开始维护这个博客了，上一篇博客还是2020年8月份更新的，时隔一年胡三汉又回来了，中间换过一段时间域名，当时买了腾讯云的ESC服务器，就顺便买了个域名，后来这边工作一直在用阿里云，刚好腾讯云的服务器也快到期了，就又迁移回来了。

<!--more-->

用next+typora写博客一个最麻烦的就是图片链接问题了，在本地的编辑的时候默认都是本地的路径，博客发布到GitHub Page后外网访问不了。

开始是用七牛云+阿里云的OSS对象存储，后来OSS存储到期了也就没有继续使用了，而且每次都要手动修改图片链接也实在是麻烦。有段时间一段时间把github当图床，这种方式需要先把图片上传到github上，然后使用的时候复制github的raw地址。最近工作中刚好用到了阿里云的OSS对象存储，用来做图片的上传，Excel文件的临时存储，所以将图床有换回了阿里云OSS，前段时间有恰好Typora刚好开始支持PicGo了，也避免了手动替换图片链接的问题。  



### 阿里云OSS配置

我之前就配置过bucket了，这次就不再写新建bucket的操作了，非常简单，网上的博客或者阿里的文档都有详细说明。

1. 查看自己的bucket列表（记住这个bucket名称，PicGo中会需要）

   ![image-20210627020455389](https://dengwq.oss-cn-hangzhou.aliyuncs.com/img/20210627020502.png)

2. 点击进去后，会进入概览页面，里面有个Endpoint地域节点，域名去掉`.aliyuncs.com`就是你的地域节点了

   ![image-20210627021013805](https://dengwq.oss-cn-hangzhou.aliyuncs.com/img/20210627021013.png)

3. 创建一个RAM用户，并且给予OSS存储管理权限

   鼠标放在右上角图片上面，选择访问控制

   ![image-20210627021322560](https://dengwq.oss-cn-hangzhou.aliyuncs.com/img/20210627021322.png)

   选择左边列表里面的人员管理 --> 用户，然后点击创建用户

   ![image-20210627021512523](https://dengwq.oss-cn-hangzhou.aliyuncs.com/img/20210627021512.png)

   填入登陆名称，显示名称，访问方式需要选择编程访问，点击确定后给你发送验证码，需要验证

   ![image-20210627021640043](https://dengwq.oss-cn-hangzhou.aliyuncs.com/img/20210627021640.png)

   用记事本记录 AccessKey ID 和  AccessKey Secret，PicGo需要通过这个账户连接你的OSS

   ![image-20210627021917204](https://dengwq.oss-cn-hangzhou.aliyuncs.com/img/20210627021917.png)

   返回用户列表页面，点击右边的添加权限按钮，给予OSS管理权限

   ![image-20210627022139664](https://dengwq.oss-cn-hangzhou.aliyuncs.com/img/20210627022139.png)

   ![image-20210627022225433](https://dengwq.oss-cn-hangzhou.aliyuncs.com/img/20210627022225.png)

4. 至此，阿里云OSS对象存储就配置完成了，需要记录四个参数Bucket名称，Endpoint地域节点， RAM账户的AccessKey ID， AccessKey Secret。



### PicGo配置

PicGo是我们国人开发的一款图床软件，可以在这里下载[Releases · Molunerfinn/PicGo (github.com)](https://github.com/Molunerfinn/PicGo/releases)，PicGo的官网地址[PicGo](https://picgo.github.io/PicGo-Doc/zh/guide/)。

1. 选择自己操作系统对应的版本

   ![](https://dengwq.oss-cn-hangzhou.aliyuncs.com/img/20210627023531.png)

2. 下一步，下一步安装

3. 图床设置，配置阿里云OSS，使用上面记录的内容即可，注意粘贴时的空格

   ![image-20210627023629117](https://dengwq.oss-cn-hangzhou.aliyuncs.com/img/20210627023629.png)

4. 上传一张图片测试一下

   ![image-20210627023727392](https://dengwq.oss-cn-hangzhou.aliyuncs.com/img/20210627023727.png)

5. PicGo配置完成



### Typora配置

1. 进入Typora偏好配置，图像配置，选择下面的策略即可

   ![image-20210627023949985](https://dengwq.oss-cn-hangzhou.aliyuncs.com/img/20210627023950.png)

2. 点击验证图片上传选项，进行验证

   ![image-20210627024048184](https://dengwq.oss-cn-hangzhou.aliyuncs.com/img/20210627024048.png)

3. Typora配置完成，之后你就可以只要在Typora编辑自己的博客，而不用操心图片的问题啦。

