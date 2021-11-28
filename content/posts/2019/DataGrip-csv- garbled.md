---
title: "DataGrip导出CSV文件中文乱码"
date: 2019-12-17T12:50:02+08:00
draft: false
tags: ["JetBrains", "DataGrip"]
categories: ["开发工具"]
series: [""]
---

# DataGrip导出CSV文件中文乱码

最近用DataGrip导出查询结果为CSV文件后，用Excel打开发现中文出现乱码的情况：

![中文乱码](https://i.v2ex.co/38J6biSO.png)

搜出来很多解决DataGrip中文乱码的方法都不对。后来发现我错怪DataGrip了，原来这是Excel的一个特性导致的：Excel读取文件时，不是默认按照utf-8编码读取的，而是会根据文件的BOM头来识别，如果没有BOM头，则按照unicode编码处理。

而DataGrip导出的CSV文件是不包含BOM头的，所以Excel按照unicode编码读取就会出现中文乱码。

### 解决方案一

对于已导出的CSV文件，可以用Notepad++或其他文本编辑器打开，然后菜单栏选择“编码”-->“转为UTF-8 BOM编码”，然后保存即可。

![转为utf-8 Bom编码](https://i.v2ex.co/m740VLXt.png)

这时候再用Excel打开，中文就可以正常显示了。

![UTF-8 BOM编码时正常显示](https://i.v2ex.co/i61s4h7U.png)



### 解决方案二

不直接导出文件，查询出结果后，在结果栏右上角选择导出格式为Tab-separated(TSV)，然后选择导出到剪贴板，或者Ctrl+A全选结果后复制，然后直接粘贴到Excel中即可。

![选择Tab-separated(TSV)](https://i.v2ex.co/376BdKMJ.png)