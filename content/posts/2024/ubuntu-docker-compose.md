---
title: "【转】在 Ubuntu 上安装 Docker CE 及 Docker-compose"
date: 2024-01-29T21:00:00+08:00
draft: false
tags: ["ubuntu", "harbor", "debian"]
categories: ["开发工具"]
series: [""]
---

> 原文来自：https://hackmd.io/@rpwis/SJSaav9Fj ，本文稍作修改，以适应Ubuntu的使用

- CE: Community Edition

1. 安装依赖

```
sudo apt update
sudo apt -y install apt-transport-https ca-certificates curl gnupg2 software-properties-common
```

1. 加入 Docker 官方的 GPG key

```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/docker-archive-keyring.gpg
```

1. 下载 Docker CE 的 repository

```
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```

1. 安装 Docker CE 和 docker-compose

```
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io -y
# 启动 docker
sudo systemctl enable --now docker
# 安装 docker-compose
sudo apt install -y curl wget
curl -s https://api.github.com/repos/docker/compose/releases/latest | grep browser_download_url  | grep docker-compose-linux-x86_64 | cut -d '"' -f 4 | wget -qi -
chmod +x docker-compose-linux-x86_64
sudo mv docker-compose-linux-x86_64 /usr/local/bin/docker-compose
```

1. 新增当前用户到 docker

```
sudo usermod -aG docker $USER
newgrp docker
```

1. 测试是否成功，输入`docker version`看到有 Server 项即为成功，看不到就表示使用者没有权限看到 Server 的选项，第五步失败。

```
$ docker version
Client: Docker Engine - Community
 Version:           20.10.17
 API version:       1.41
 Go version:        go1.17.11
 Git commit:        100c701
 Built:             Mon Jun  6 23:03:17 2022
 OS/Arch:           linux/amd64
 Context:           default
 Experimental:      true

Server: Docker Engine - Community
 Engine:
  Version:          20.10.17
  API version:      1.41 (minimum version 1.12)
  Go version:       go1.17.11
  Git commit:       a89b842
  Built:            Mon Jun  6 23:01:23 2022
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.6.6
  GitCommit:        10c12954828e7c7c9b6e0ea9b0c02b01407d3ae1
 runc:
  Version:          1.1.2
  GitCommit:        v1.1.2-0-ga916309
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
```

1. 输入` docker-compose version`，若得到如下结果，则表示成功:

```
$ docker-compose version
Docker Compose version v2.14.0
```

### 参考文献

[Install Docker CE and Docker Compose on Debian 11/10](https://computingforgeeks.com/install-docker-and-docker-compose-on-debian/)
[How To Install Docker Compose on Linux](https://computingforgeeks.com/how-to-install-latest-docker-compose-on-linux/)
