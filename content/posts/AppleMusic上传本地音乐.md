+++
date = '2025-04-17T15:20:17+08:00'
draft = false
title = 'AppleMusic上传本地音乐'

+++

AppleMusic国区的很多音乐都没有，更不用说被禁掉的李志了，幸好AppleMusic支持上传本地音乐到资料库，但是缺点是无法显示滚动歌词，不过也够用了。

## AppleMusic支持导入的格式

![image-20250417153015424](https://dengwq.oss-cn-hangzhou.aliyuncs.com/img/image-20250417153015424.png)

## 格式转换

当前网上的音乐资源大多是`Mp3`或者`flac`格式的，这时候就需要转换一下音频格式了。

期间尝试了格式工厂（转换的aac文件AppleMusic无法识别），万兴优转（普通用户只能转换音频的三分之一），Aiseesoft（免费用户只能转换五分钟内的音频），最后选择`foobar2000`。

选择`foobar2000`的原因有两个：

1. 操作简单，选择文件右键就能转换，而且能够保存原文件的`id3`标签信息；
2. 插件丰富，通过ESLyric可以很方便的往`id3`标签里面写入歌词；
3. 歌词写入，ESLyric + Preview（预览插件，一首歌可以只播放几秒钟），实现自动批量的歌词写入；

**转换歌曲格式**

![image-20250417154405411](https://dengwq.oss-cn-hangzhou.aliyuncs.com/img/image-20250417154405411.png)

**匹配歌词 + 写入歌词**

![image-20250417154704962](https://dengwq.oss-cn-hangzhou.aliyuncs.com/img/image-20250417154704962.png)

**Mp3tag处理id3元数据**

![image-20250417155300793](https://dengwq.oss-cn-hangzhou.aliyuncs.com/img/image-20250417155300793.png)

## 上传歌曲到资料库

![image-20250417155616605](https://dengwq.oss-cn-hangzhou.aliyuncs.com/img/image-20250417155616605.png)
