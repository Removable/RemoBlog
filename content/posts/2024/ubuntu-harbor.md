---
title: "在 Ubuntu 上部署 Harbor 并配置反代"
date: 2024-01-29T22:00:00+08:00
draft: false
tags: ["ubuntu", "harbor", "debian", "docker"]
categories: ["开发工具"]
series: [""]
---

> Harbor 是 VMware 公司开源的企业级 DockerRegistry 项目,其目标是帮助用户迅速搭建一个企业级的 Dockerregistry 服务.它以 Docker 公司开源的 registry 为基础,提供了管理 UI,基于角色的访问控制(Role Based Access Control),AD/LDAP 集成、以及审计日志(Auditlogging) 等企业用户需求的功能,同时还原生支持中文.



## 最终目标

成功在 Docker 中运行 Harbor 服务（不使用默认的80和443端口），并通过配置 Nginx 反代以从外部访问 Harbor



## 安装 Docker 及 docker-compose

请参考这篇博文：[【转】在 Ubuntu 上安装 Docker CE 及 Docker-compose](https://blog.imguan.com/2024/01/29/ubuntu-docker-compose/)



## 下载 Harbor 安装包

>  Harbor 安装包的发布页地址为：https://github.com/goharbor/harbor/releases

#### 1. 下载

官方提供了在线安装包和离线安装包两种类型，请自行选择想要的版本。

- 在发布页下载安装包后，上传至 Ubuntu 服务器

或

- 运行如下命令：
  ```bash
  wget https://github.com/goharbor/harbor/releases/download/v2.9.2/harbor-online-installer-v2.9.2.tgz
  ```

  **请注意修改对应版本和在线/离线安装包**

#### 2. 解压

```bash
tar xf ./harbor-online-installer-v2.9.2.tgz
```

**请注意修改对应的文件名**



## 配置并安装 Harbor

#### 1. 配置

解压之后应该会出现一个`harbor`文件夹，里面主要有以下几个文件：

- install.sh
- prepare
- common.sh
- harbor.yml.tmpl

我们在该文件夹下运行命令：

```bash
cp harbor.yml.tmpl harbor.yml
```

接着运行以下命令以编辑配置文件：

```bash
vim harbor.yml
```

> Harbor 默认会使用 80 和 443 端口以提供外部访问。但是一般来说，我们的系统上都已经安装了 Nginx （或其他同类软件），所以这里我们会修改默认的端口，后面通过 Nginx 反向代理来使外部可访问。

我们需要修改以下几处：

- 将`hostname`修改为你的域名（此处不重要，原因见第三条）

- 将`http:`下的`port`修改为其他未被占用的端口，比如`8081`。这是基于我的服务器的80端口已经被使用。

- 将`https:`以及其下的`port`,`certificate`,`private_key`配置项全部注释掉（在每行的开头加上#号）。这是因为我们将通过 nginx 反代配置https访问，如果你希望直接设置https访问，请自行配置。

- 找到默认注释的`external_url`配置项，取消注释。修改值为你要用于访问 Harbor 的地址，如：`https://register.example.com`。（需提前配置好解析）

- （可选）配置`harbor_admin_password`，这将是默认的登录密码。

- 配置数据库，可采用默认数据库（第一条）或者外部数据库（第二条）：
  - 修改`database:`下的`password`，其他项可保持默认。
  - 若想使用已有的 PostgreSql 作为数据库，则需将上述的`database`及其下的所有配置项都注释掉。然后找到`external_database`配置项（默认是注释状态），取消注释并修改对应的配置。

- （可选）修改`data_volume`配置项为你想要的文件夹。

#### 2. 安装

运行如下命令：

```bash
sudo ./install.sh
```

如果以上配置都正确的话，Harbor 所需的各项服务应该都会正常运行了。



## 配置反代

> 我们采用 Nginx Proxy Manager 来配置反向代理，这是个非常方便的工具，推荐使用。如果需要手动配置 Nginx 来添加反代，请自行解决。

运行如下命令：

```bash
docker network inspect bridge --format='{{json .IPAM.Config}}'
```

正确运行的话，会出现类似`[{"Subnet":"172.17.0.0/16","Gateway":"172.17.0.1"}]`的结果。

在 Nginx Proxy Manager 中添加反代配置：

- 在`Domain Names`中填写为 Harbor 准备的域名（上面提到过的那个）。

- 在 `Forward Hostname / IP`中填写上面结果中的`Gateway`的IP。
- `Forward Port`填写之前配置的`http`的`port`项的端口（如8081）。
- 在`SSL`Tab中配置证书和其他项。

## 完成！

现在可以在浏览器中访问了，记得修改默认管理员密码哦~
