---
title: "使用diskpart释放WSL2的磁盘空间"
date: 2023-06-13T10:03:07+08:00
draft: false
tags: ["WSL2", "Windows", "转载"]
categories: ["开发工具"]
series: [""]
---

[原文链接](https://www.gscblog.com/shi-yong-diskpartshi-fang-wsl2de-ci-pan-kong-jian/)

当我们在WSL里清除一些docker 镜像，或者删除一些文件时，发现WSL的虚拟磁盘文件大小并没有减少，这时我们可以用DiskPart来收缩WSL的虚拟磁盘。

## **❓**什么是DiskPart？

DiskPart取代了它的前身 —— [fdisk](https://so.csdn.net/so/search?q=fdisk&spm=1001.2101.3001.7020)，是一个命令行实用程序，可以管理自Windows 2000以来运行所有操作系统版本的计算机中的磁盘、分区或卷，还包括最新的Windows 11。用户可以输入DiskPart命令直接组织硬盘分区，或创建文本文件脚本来执行多个命令。您可以在磁盘管理工具中使用的大多数命令都集成在DiskPart中。

## **📢**关闭WSL2

在执行压缩命令之前需要先关闭WSL2, 使用命令`wsl --shutdown`
![在这里插入图片描述](https://i.v2ex.co/0w1k0Fq0.png)

## 🧪使用DiskPart释放WSL2的磁盘空间

使用快捷键  `window + r` 打开运行窗口，输入diskpart, 然后点击OK。
![在这里插入图片描述](https://i.v2ex.co/9hPA80er.png)

可以打开DiskPart 的命令行工具窗体
可以使用 `help select vdisk` 命令来查看帮助。
![在这里插入图片描述](https://i.v2ex.co/wUG9Qa53.png)

在目录`C:\Users\你的用户名\AppData\Local\Packages` 中查找 `ext4.vhdx` 文件

![在这里插入图片描述](https://i.v2ex.co/uN4w3t5Z.png)

选择虚拟磁盘文件（就是上一步中查找到的文件）
`select vdisk file="C:\Users\你的用户名\AppData\Local\Packages\TheDebianProject.DebianGNULinux_76v4gfsz19hv4\LocalState\ext4.vhdx"`

![在这里插入图片描述](https://i.v2ex.co/RgE5j952.png)

执行压缩磁盘命令`compact vdisk`

![在这里插入图片描述](https://i.v2ex.co/NRx2QX31.png)

当 100% 完成时，关闭命令行窗口即可。
