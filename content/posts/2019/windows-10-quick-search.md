---
title: "Windows 10无法快捷搜索程序或设置"
date: 2019-02-12T12:50:02+08:00
draft: false
tags: ["Windows"]
categories: ["玩机经验"]
series: []
---

自从系统升级到 windows 10以后，就开始越来越依赖它的快捷搜索功能了。

比如我想打开计算器，以前可能需要到开始菜单的附件那里去找，或者在桌面弄个快捷方式。而现在只需要在搜索框里输入“jsq”，也就是计算器的拼音首字母，就能快速的定位到计算器应用上：

![通过拼音缩写搜索](https://i.v2ex.co/jzy8feU6.jpeg)

不过偶尔我们会发现这个方便的功能失效了，输入“jsq”以后，只显示网络搜索结果，而不会显示计算器这个应用。不要慌，这时候只需按下Win + X组合键，在弹出菜单中选择 Windows PowerShell (管理员)，然后在打开的深蓝色背景的 PowerShell 界面上输入：

```shell
Get-AppXPackage -Name Microsoft.Windows.Cortana | Foreach {Add-AppxPackage -DisableDevelopmentMode -Register "$($_.InstallLocation)\AppXManifest.xml"}
```

然后回车执行即可

![输入shell命令](https://i.v2ex.co/Ns5TZm2i.jpeg)

稍等片刻，待命令执行完毕后，搜索功能应该就已经恢复了~

**（如果PowerShell报错“部署失败，原因是 HRESULT: 0x80073D02, 无法安装程序包，原因是它修改的资源当前正在使用中。”，可以尝试在任务管理器中结束名为“Windows Shell Experience Host”的进程后重试）**