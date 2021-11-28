---
title: "解决Google Play报错DF-DFERH-01问题"
date: 2019-04-03T12:50:02+08:00
draft: false
tags: ["Android"]
categories: ["玩机经验"]
---



# 解决Google Play报错DF-DFERH-01问题

博主最近换了港版三星S10+，刚开始用港行系统的时候，Google Play使用一直没有任何问题。后来刷成国行系统后，在某一次系统重启之后，Google Play突然开始无法连接，显示“从服务器检索信息时出错。DF-DFERH-01”这个错误。

![DF-DFERH-01](https://vip1.loli.net/2020/01/01/MVTAvzrIquoXcWL.png)

经过一番搜索，发现是因为国行的谷歌框架会请求一个.cn后缀的域名“services.googleapis.cn”，这个.cn后缀的域名会自动被识别为国内域名，从而绕开代理，导致Google Play连接出错。解决方法有两个：

- 开启全局代理或者强制让“googleapis.cn”这个域名走代理，以KoolShare的OpenWrt软路由系统为例，只需在代理工具内选择“黑白名单”，然后将该域名添加到域名黑名单内即可。添加并保存后稍等几分钟或者清空一下DNS即可生效。
  ![添加域名黑名单](https://i.v2ex.co/f7L6f0ZB.png)
- 如果上面的方法没有作用，还可以使路由器强制将services.googleapis.cn这个地址解析到216.58.197.195这个IP。还是以KoolShare的OpenWrt软路由系统为例，在代理工具内选择“DNS设定”，然后在“自定义dnsmasq”文本框内添加一行“address=/services.googleapis.cn/216.58.197.195”后保存即可。
  ![自定义dnsmasq](https://i.v2ex.co/u5nB6452.png)