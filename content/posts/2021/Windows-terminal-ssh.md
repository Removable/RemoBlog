---
title: "使用Windows Terminal进行SSH登录"
date: 2021-12-13T10:47:02+08:00
draft: false
tags: ["Windows", "Ssh", "Linux"]
categories: ["开发工具"]
series: [""]
---

## 安装Windows Terminal

> 注意：Windows Terminal 需要 Windows 10 1903 (build 18362) 或更新版本

- 通过微软商店 (Microsoft Store) 安装【官方推荐】
  在微软商店搜索“Windows Terminal”安装即可

- 在 Github 上下载
  在 Terminal 官方仓库的 [Releases page](https://github.com/microsoft/terminal/releases) 页面下载。选择最新版本的`Microsoft.WindowsTerminalPreview_<版本号>_8wekyb3d8bbwe.msixbundle`下载，双击安装即可。

- 通过 Chocolatey 安装（非官方）
  在已安装 Chocolatey 的情况下，执行命令：

  ```
  choco install microsoft-windows-terminal
  ```

- 其他更多方式请参见 Windows Terminal 官方 [GitHub](https://github.com/microsoft/terminal) 仓库



## 设置SSH登录

首先进入 Windows Terminal 设置界面：

![进入设置界面](https://vip2.loli.io/2021/12/13/oVl7ZJH5rCpnTjx.png)

在设置解密左侧最下方点击“打开JSON文件”，这时系统会调用默认文本编辑器打开配置文件。

在文本编辑器中找到 profiles => list 节点：

![list节点](https://vip2.loli.io/2021/12/13/hS4aDUZqBAPd8Ht.png)

在list中添加一个节点：

```json
{
	"guid": "{a10c1013-d3f4-479c-bb69-7899e7597871}",
	"hidden": false,
	"name": "my_server",
	"commandline" : "ssh -i <username>@<ip/url> -p<ssh端口号>",
	"icon": "<icon path>"
}
```

- guid：连接表示，全局唯一，可以通过[Online GUID / UUID Generator](https://www.guidgenerator.com/online-guid-generator.aspx)生成
- hidden：是否隐藏，默认即可
- name：在标签栏显示的名称
- commandline：命令行命令，请将尖括号相关内容替换成自己的
- icon：标签栏显示的图标，可以是本地图片，也可以是网络图片

如果你要连接的远程设备是通过账号密码登录的话，那么此时保存配置并重新打开 Windows Terminal 即可使用。



## 通过 SSH 密钥登录

这是更安全也更方便的方法。关于如何生成密钥对以及如何在Linux服务器配置公钥并设置密钥验证，网上有很多其他教程，这里就不赘述了。

只说一下 Windows Terminal 如何设置使用私钥：

还是打开刚刚的配置文件，将 commandline 修改为 `ssh -i \"<私钥地址>\" <username>@<ip/url> -p<ssh端口号>`。比如我将文件名为 `TestKey`的私钥存放于 `C:\Users\<用户名>\.ssh`目录内，则应修改为：

```
ssh -i \"~/.ssh/TestKey\" <username>@<ip/url> -p<ssh端口号>
```

此时保存配置文件并重新打开 Windows Terminal 即可！

注：若提示 permission dine之类的错误，可以尝试删除`C:\Users\<用户名>\.ssh`目录下的`known_hosts`文件试试。