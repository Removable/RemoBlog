---
title: "Android去除感叹号或叉号与搭建“204服务器”"
date: 2017-09-04T15:29:21+08:00
draft: false
tags: ["Android"]
categories: ["玩机经验"]
series: []
---

# Android 8.0/7.1.x去除感叹号/叉号

> 由于Google进行过数次的调整，因此以下方法仅支持7.1.x及以上版本的Android。

#### 为什么会有感叹号/叉号

从Android 5.0开始，Google为它添加了一个网络状况评测机制，这个机制的原理也非常简单粗暴——就是通过访问google.com域名下的一个地址，并通过访问情况来判断网络状况。然而由于众所周知的原因，google.com在大陆地区并不能很好的访问，所以Android就判断网络存在问题，因此就会一直显示着那个恼人的叹号/叉号，并且在该情况下，会增大电量的消耗。

#### 准备工作

- 首先需要在开发者模式（Developer options）中开启USB调试（USB debugging）。原生Android开启开发者模式的方法为在“关于手机”中快速点击“版本号”（Build number）7次即可。
- 另外需要安装好对应手机的驱动，一般在Windows 10下连接上手机即可自动安装好驱动，如果没有装好，请自行Google或等待博主不知道什么时候才会有的下一篇博文。→_→
- 准备好adb工具。没有的朋友可以在这里下载——[Google Drive分享链接](https://web.archive.org/web/20210125205530/https://drive.google.com/open?id=0B9TuJwB-7ME9RHVueDdPM2IyazQ)，下载好后解压出来，然后将命令提示符（cmd）定位到adb.exe所在文件夹即可。
- 然后将手机连接到电脑上

#### 一个命令就搞定

准备工作看起来倒是挺多，但是实际操作只有一行命令而已。就是在cmd中执行如下命令：

```
adb shell settings put global captive_portal_https_url xxxxxxxx
```

xxxxxxxx就是要更换的新的地址，下面列出一些可用的地址：

- https://www.google.cn/generate_204 （注意是cn后缀）
- https://www.v2ex.com/generate_204
- https://www.noisyfox.cn/generate_204

用以上任一地址替换xxxxxxxx后执行命令即可~

执行完毕后开启再关闭一下飞行模式就大功告成了！

# 搭建属于自己的generate_204服务器

#### 对于Nginx服务器

nginx服务器搭建这个服务很简单，只需要再nginx的服务器配置中，添加一个location即可：

```
location /generate_204 { return 204; }
```

然后重启一下nginx服务就好了。

#### 对于Apache服务器

首先要确保已经安装了rewrite模块，不然以下方法不适用。

在.htaccess文件中（如果没有，就在根目录下新建一个空文件并命名为.htaccess即可）添加以下代码：

```
RewriteEngine On  
RewriteCond %{REQUEST_URI} /generate_204$  
RewriteRule $ / [R=204]  
```

#### 大功告成

那么现在只需要将Android的检测地址改为你自己的域名 + /generate_204即可~