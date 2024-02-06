---
title: "Ubuntu 使用 Nexus 3 添加 Docker 和 Apt 仓库代理"
date: 2024-02-03T20:00:00+08:00
draft: false
tags: ["ubuntu", "nexus", "debian"]
categories: ["开发工具"]
series: [""] 
---

## 通过 Docker 运行 Nexus 3

> 确保机器上已经安装了 Docker 以及 Docker-compose

可通过以下配置文件运行（自行根据需要调整）：

```yaml
version: "3.4"

volumes:
  nexus-data:
    driver: local

services:
  nexus3:
    image: sonatype/nexus3:latest
    container_name: nexus
    restart: always
    environment:
      TZ: Asia/Shanghai
      ports:
        - 8081:8081
        - 8082:8082
    volumes:
      - nexus-data:/nexus-data
```

成功运行后，可通过以下命令获取 admin 用户的初始密码（其中的nexus3-server对于上述配置文件的`container_name`）：

```bas
docker exec nexus3-server cat /nexus-data/admin.password
```

反向代理等相关步骤在此略过。



## 配置 Docker 代理仓库

我们这里以 Docker 官方仓库和 Microsoft Artifact Registry 为例，配置代理，以解决国内访问这些 docker 仓库的网络问题。

#### 1. 添加 Docker Hub 代理仓库

使用管理员账号登录后，点击顶部的齿轮图标进入设置页。选择 Repository => Repositories => Create repository 项，仓库类型选择 `docker(proxy)`。

我们主要需要对图中画红框的项目进行配置，其他项保持默认即可。

![docker hub代理配置.jpg](https://cdn.sa.net/2024/02/05/sNOrHJ1WQIFyYl2.jpg)

配置完毕后点击页面底部的`Save`保存。

#### 2. 配置 Microsoft Artifact Registry 代理仓库

继续新建`docker(proxy)`，如下配置（其他第三方的 Docker 仓库可以参考此例）。

![MAR代理配置](https://cdn.sa.net/2024/02/05/T8OzbviXF6CH9LZ.jpg)

#### 3. 配置代理组

你也许已经注意到了，上面两个代理仓库的配置中，我们都没有对HTTP/HTTPS这部分进行配置。那么要如何才能访问到这两个代理仓库呢？答案是使用`docker(group)`类型的仓库对它们进行聚合，方便统一使用一个出入口进行访问。

新建一个`docker(group)`类型的仓库，`Name`项自行填写：

![docker聚合仓库配置](https://cdn.sa.net/2024/02/05/am1Yxpz9fvgNsV7.jpg)

图中的三项配置：

- HTTP 配置这里的8082端口对应之前的 docker-compose 配置文件中的 8082 端口，如果需要自定义其他端口的话，记得一并修改。

- 如果你想要不用登录就可以使用`docker pull`命令从仓库拉取镜像，那么就将`Allow anonymous docker pull`这一项勾选。如果勾选此项，还需要配置`Docker Bearer Token Realm`，下面会讲到。
- 在`Group`项配置中，将你需要聚合的仓库都选入到右侧的 Members 栏中。
- 该仓库其他配置项可根据需要自行配置。

#### 4. 配置 Realm（可选）

在设置页左侧选择 Security => Realms ，将`Docker Bearer Token Realm`项移到右侧 Active 栏中。

#### 5. 使用

完成上述配置后，如需使用域名访问，请自行做好相关配置。为了方便起见，建议对聚合仓库单独配置一个地址。比如 Nexus3 的管理页面地址为`nexus.example.com`，那么可以将聚合仓库配置为`hub.example.com`。

- 登录（如果配置允许匿名访问的话，则跳过这一步）
  ```bash
  docker login hub.example.com
  ```

- 拉取镜像
  ```bash
  docker pull hub.example.com/{用户名/组织名}/{镜像名}:{标签}
  ```

  - 对于 Docker 官方发布的镜像，`{用户名/组织名}`为 library ，如 `docker pull hub.example.com/library/mariadb:latest`
  - 对于 Docker Hub 中其他用户/组织发布的镜像，或者其他第三方仓库内的镜像（如上述`mcr.microsoft.com`）：`docker pull hub.example.com/dotnet/aspnet:8.0`



## 配置 APT 代理

因为个人常用 Ubuntu 系统，所以这里以配置 Ubuntu 的 APT 库代理为例。

#### 1. 设置代理仓库

设置页选择 Repository => Repositories => Create repository 项，选择`apt(proxy)`。

比如我们安装 docker ce 时常常遇到的无法连接/下载速度缓慢的问题，我们就以代理 Docker 官方的 Ubuntu 仓库为例。

![nexus3配置apt代理](https://cdn.sa.net/2024/02/05/pMw1sVajXJiIoct.jpg)

其中的 Remote storage 为要代理的仓库地址。配置完成后保存。

#### 2. 访问方法

配置好后，我们在设置页进入刚刚添加的仓库的详情可以看到这个仓库的地址，可以通过相应命令将该仓库添加到系统中：

![apt代理仓库地址](https://cdn.sa.net/2024/02/06/qO8irzDXTwvNGoI.jpg)

默认情况下，Nexus 未启用匿名用户，因此要使用该仓库的话需要登录，直接添加该仓库地址并使用 `apt update`的话，会报 401 错误。因此我们需要配置允许匿名访问，或者在 Ubuntu 中通过账号密码的方式使用：

###### 2.1 配置匿名访问 APT 代理仓库

由于默认的匿名用户对应的权限`nx-anonymous`是可以访问所有仓库的，因此我们需要对其进行限制。在这里我将其限制为近允许访问所有的 APT 仓库。

在设置页选择 Security => Roles，系统默认的`nx-anonymous`权限我们无法修改，因此我们新建一个新的权限。

![新建匿名权限](https://cdn.sa.net/2024/02/06/mVSiYHv3UpORNGu.jpg)

新权限的 ID 和 Name 随意。在 Available 栏搜索 `view-apt`，将如图两个权限添加到 Given 栏。如果你只想让匿名用户访问之前配置的 apt-docker 仓库，那么就只选择 `nx-repository-view-apt-apt-docker-browse/read`这两个。然后点击保存。

接着在设置页选择 Security => Users ，编辑 `Anonymous` 用户。将默认的 `nx-anonymous`权限移除，把刚刚添加的新权限分配给它。然后保存。

最后到设置页选择 Security => Anonymous Access ，勾选允许匿名用户访问，保存。

现在就可以直接运行 `apt update`命令了。

###### 2.2 配置 Ubuntu 使用账号密码访问

> 建议参考上面的步骤新建一个专门用于拉取 APT 仓库的账号，直接使用 admin 账号有风险。

在 `/etc/apt/auth.conf.d` 目录下新建一个文件，比如 `nexus.conf`：

```bash
sudo vim /etc/apt/auth.conf.d/nexus.conf
```

文件内容为（自行替换相应内容）：`machine 仓库地址 login 用户名 password 密码`

为安全起见，我们还可以设置只允许 root 用户访问该文件：

```bash
sudo chmod 600 /etc/apt/auth.conf.d/nexus.conf
```

保存之后，即可正常使用 `apt update` 命令了。
