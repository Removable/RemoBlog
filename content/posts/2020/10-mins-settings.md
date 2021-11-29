---
title: "【转】10 分钟服务器安全设置，Ubuntu安全设置入门"
date: 2020-05-21T12:50:02+08:00
draft: false
tags: ["Linux", "Ubuntu"]
categories: ["云"]
series: [""]
---

> 英文：Cody Littlewood
>
> 译者：伯乐在线 – Panblack
>
> 链接：http://blog.jobbole.com/103344/



Bryan Kennedy 的《5分钟服务器安全设置》很好地介绍了对多数服务器攻击的防御对策。我们对他的方法做了一些修改，记录下来，作为推广我们的流程和最佳实践的一部分。还增加一些额外的解释，年轻的工程师们应该可以从中受益。



我每天上午检查 logwatch 邮件的时候，看到那些成百上千的登录尝试，几乎没有成功的，完全就像是一种享受。（有很多非常令人无语，比如用 1234 做 root 密码反反复复的登录）。这篇入门文章适用于 Debian/Ubuntu服务器，这是我们最喜欢的服务器发行版，通常我们这些服务器只是作为 docker 容器的宿主机，不过原理仍然适用。下一次我们再深入讲解如何保护专门用作 docker 宿主机的服务器。



在大规模系统上，当然最好是用 Ansible 或 Shipyard 这类工具完全自动化配置。不过偶尔的，你只是搭建一台独立服务器或为一个 Ansible recipe 准备基准设置，这就是本文要涵盖的内容。



声明：本文仅仅是入门和基础，你需要根据自己的需求来扩展。



### **重要的事先说**



我们还没给 root 设密码呢，密码要随机、要复杂。我们用一种密码管理软件的密码生成器，调至最复杂设定。PW 管理软件将密码加密保存，并且由一个很长的主密码保护着。这里提供了多项冗余措施——长、复杂、随机的密码 + 加密保存/另一个长密码保护。不管你是用 PW 管理软件或其他手段，要妥善保管密码并且要加密保存。你只有在忘了 sudo 密码时才会用到这个密码。



> \# passwd



\* 注：在 HN 和 Redit 上有很多关于 root 密码的讨论，值得一读。



下一步就需要更新软件库并升级系统、应用最新的补丁。我们后面有个章节专门介绍如何自动安装安全更新。



> apt–**get** update
>
> apt–**get** upgrade



### **添加用户**



永远不要以 root 登录服务器。在用户名方面我们跟 Bryan 遵循类似的惯例，但是你可以根据自己的喜好使用任何惯例。在小团队里，大家共用一个用户名不是问题；但是队伍再大一些的话，最好是给不同的用户设定不同的权限级别，只给精心挑选的少数用户赋予 sudo 权限。



> useradd deploy
>
> **mkdir** /home/deploy
>
> **mkdir** /home/deploy/.ssh
>
> **chmod** 700 /home/deploy/.ssh



给用户 deploy 配置你喜欢的 shell，这里我们用 bash：



> usermod –s /bin/bash deploy



记住， chmod 700 表示“所有者可以读、写、执行”。我们现在还在 root 帐号下，马上就要将此文件夹设为属于 deploy 用户和 deploy组，只有这个用户对 .ssh 文件夹有完全控制权。



### **使用 ssh key 验证**



我们倾向于避免用密码登录服务器。Bryan 当初那份指南发表后，关于这个话题有很多讨论，但我愿意加入这个阵营。下面是一些要点：



1. ssh keys 比密码更好，因为它包含并要求更多信息;
2. 密码可以被暴力破解，猜测一个公钥基本上不可能，可以认为是完美的安全措施；
3. 电脑丢失了怎么办？是的，你的私钥到了别人手里。不过废除 ssh-key 很容易，只需要将公钥从 authorized_keys 里删除。你还应该给你的私钥加上安全的、足够长的密码短语。见下一条；
4. 以上这些都好用的前提是：必须用安全的、足够长的密码短语来保护你的私钥。重要的事情至少说两遍。

好了，我们把服务器密码验证抛到脑后吧。把你电脑里的 id_rsa.pub 的内容复制到服务器的 authorized keys 文件里。



> vim /home/deploy/.ssh/authorized_keys



我们基于 Linux 安全性的“最少特权原则”来设置正确的权限：



> **chmod** 400 /home/deploy/.ssh/authorized_keys
>
> **chown** deploy:deploy /home/deploy –R



chmod 400 将文件权限设为“仅所有者可读”。第二个命令 chown ，使用户 deploy 和 组 depoly 成为它们家目录的所有者。我们先前提到过这个，记得吗？在设这这个目录权限为“所有者可以读、写、执行”的时候。



我们顺利测试 deploy 用户并设置 sudo 之后，再回来禁止 root 登录和强制实行仅密钥认证。



### **测试 deploy 用户、设置 sudo**



我们来测试一下用 deploy登录，同时不要关闭 root 帐号的 ssh 连接以备万一。如果没有问题，我们就用 root 用户给 deploy 设置密码，因为我们没会禁用密码登录，这个密码是 sudo 的时候用的。还是用一个密码管理器来生成一个复杂随机的密码、加密保存、在团队中共享（同步加密的 pw 文件）。



> passwd deploy



设置 sudo 很简单，用这个命令打开 sudo 文件：



> visudo



如下，在 root 用户下面添加 %sudo 组（在sudo文件里，用户名没有前缀，组名需要用 %前缀）。确保用 # 注释掉其他任何用户和组，新安装的系统一般不会有，但还是确认一下好。



> root  ALL=(ALL) ALL
>
> %sudo  ALL=(ALL:ALL) ALL



然后把 deploy 用户添加到 sudo 组。



> usermod –aG sudo deploy



deploy 现在具备了sudo 权限，一般来说你需要退出并重新登录来使用这个权限，不过有个避免这个步骤的小伎俩：



> exec su –l deploy



这个命令为 deploy 用户启动了一个新的交互 shell，并带有了 sudo 组的新权限。你需要输入 deploy 的密码，但是感觉上比注销再登录回来能更快些。「译者注：这个感觉我没有 ![;)](https://hackingforfun.online/wp-content/themes/Git_theme-alpha-722eaee2c78bb347559eb0135760d2ef48df1f95Git_theme.git/assets/img/smilies/icon_wink.gif) 」



> 编辑注释：
> 感谢 Reddit 上的 ackackacksyn 指出来，不应该直接把用户加到 sudoer （即sudo file），而是要加入到具备 sudo 权限的组，文章已经按此修改。
> 感谢 /r/netsec 的 FredFS456 指出来，需要注销并重新登录才能使新加入的组权限生效，文章已经按此修改。



### **强制 ssh 密钥登录**



服务器的 ssh 设置在这里：



> vim /etc/ssh/sshd_config



你需要修改或添加下面这几行，我觉得它们相当的简单明了。你可以填上你用来登录的 IP ，我们有个用 OpenVPN 搭建的公司 VPN 服务器，带加密验证的，所以为了连接到服务器，你必须首先连上 VPN 。



> PermitRootLogin no
>
> PasswordAuthentication no
>
> AllowUsers deploy@你的VPN或固定IP
>
> AddressFamily inet



重新启动 ssh 服务来让这些规则生效，你很可能需要重新连接（这回就用 deploy 用户吧！）



> service ssh restart



\* 注：感谢 HN 上的 raimue 和 mwpmaybe 指出来，我们在后面要安装的 fail2ban现在还不支持 IPv6，所以我在 ssh_config 里添加了 AddressFamily inet 这一行，意思是只允许 IPv4。



### **设置防火墙**



这里有两个阵营，一个直接使用 iptables，另一个使用一款叫做 ufw 的简便接口（在 iptables 之上的一个层，目的是简化操作）。对安全来说，简单化通常是更好的，DigitalOcean ufw 真心不错而且绝不只是个基本工具。



Ubuntu 上 ufw 是默认安装的，在 Debian 上只需要执行一下 sudo apt-get install ufw 。



默认情况下 ufw 应该拒绝一切入站连接、允许全部出站连接，但是这行不通（那样的话你怎么连进来呢？）。我们来一步步地显式允许我们视为OK的连接。



首先我们需要保证支持 IPV6 ，只需打开配置文件：



> vim /etc/default/ufw



允许 IPv6



> IPV6=yes



其它要打开的端口我们用 ufw 工具在命令行添加，这非常方便：



> sudo ufw allow from 你的IP **to** any port 22
>
> sudo ufw allow 80
>
> sudo ufw allow 443
>
> sudo ufw disable
>
> sudo ufw enable



第一个命令是个冗余措施，保证只有从我们的 IP 才能连接 ssh标准端口，第二个和第三个用来打开 http 和 https 。



> 注：感谢 chrisfosterelli 指出来，如果你要设置第一条规则（你应该），要确保你有个固定 IP 或安全的 VPN，动态 IP 会在某天把你锁到服务器外面。[ 译者注：这是废话，但比较贴心，呵呵 ]

### 

### **自动应用安全更新**



我喜欢这个，安全更新并不完美，但及时安装补丁还是更好些。



> apt–get install unattended–upgrades
>
> vim /etc/apt/apt.conf.d/10periodic



照下面的例子编辑文件10periodic ：



> APT::Periodic::Update–**Package**–Lists “1”;
>
> APT::Periodic::Download–Upgradeable–Packages “1”;
>
> APT::Periodic::AutocleanInterval “7”;
>
> APT::Periodic::Unattended–Upgrade “1”;



我基本同意 Bryan 的说法，禁用普通更新，只启用安全更新。因为你不希望一个应用在你毫无察觉的时候 down 掉，只是因为某个软件包被更新了。相反，安全更新很少在应用程序方面产生噩梦般的依赖问题。



> vim /etc/apt/apt.conf.d/50unattended–upgrades



这样修改文件：



> Unattended–Upgrade::Allowed–Origins {
>
>   “Ubuntu lucid-security”;
>
>   *//”Ubuntu lucid-updates”;*
>
> };



到这里你已经准备就绪了。



### **fail2ban**



![10 分钟服务器安全设置，Ubuntu安全设置入门](https://hackingforfun.online/wp-content/uploads/2019/03/frc-3d6f3f6adcd30dc52977be98a4ef6ad6.jpeg)



Fail2ban 软件包主动拦阻可疑行为。根据官方 wiki，Fail2ban 扫描日志文件（比如：/var/log/apache/error_log ），通过在 iptables 添加规则来阻止那些有恶意迹象（多次密码错误，探寻漏洞等…）的 IP。



Fail2Ban 自带多种协议过滤器（HTTPS, STMP, SSH等），它还与 Apache 、Nginx等众多服务集成，提供一定程度的 DDoS 或暴力破解防护。不过使用的时候要小心，因为封堵 DDoS 攻击的源地址，可能也会阻止正常用户一段时间。它带有大量的配置选项，包括集成SendMail，当某个 IP 被封堵时发出邮件通知。你尽可以点击上述这些链接，看看哪些选项对你的胃口。



我们来安装这个软件，作为开始，只用默认配置来保护 ssh：



> apt–get install fail2ban

### 

### **双重认证**



构建任何敏感系统时，双重认证(2 Factor Authentication, 2FA)不是额外的要求，而是必须的。理论上，如果你强制启用 2FA (加上以上所有措施），那么为了登录你的服务器（应用程序漏洞除外），攻击者必须：



1. 获得你的证书来登录VPN
2. 获得你的电脑、拿到你的私钥
3. 获得你私钥的密码短语
4. 获得你的电话以遍完成双重认证

这就需要跨越不少的障碍了，即使都得手了，攻击者还是需要 deploy 用户的密码才能通过 sudo 获得 root 权限，而密码可是用 AES 加密保存的。



安装这个软件包



> apt–get install libpam–google–authenticator



运行这个命令来设置，根据提示配置：



> su deploy
>
> google–authenticator



2FA非常简单，而且增加了很强的一层安全性。



### **Logwatch**



这不仅仅是个简单的小快乐和监控工具，用来事后查看服务器发生了什么。Logwatch 监控你的日志文件，经过配置后，它可以每天发送格式精美的报告邮件。邮件内容看起来非常有喜感，你会惊讶每天有那么多要进入你服务器的企图。我装这个软件包没有别的原因，就是给团队显示一下好的安全性配置有多么重要。



DigitalOcean 有一篇关于 logwatch 安装配置的详细文档，但是如果时间限制在 10 分钟以内，我们只会完成安装，然后配置 cron 计划任务来每天发送邮件。



> apt–get install logwatch



添加 cron 计划任务



> vim /etc/cron.daily/00logwatch



把这一行加进去：



> /usr/sbin/logwatch —output mail —mailto you@example.com —detail high



完工！



好啦，完成这一切之后，你的主要漏洞关注点将是你的应用程序和服务，这完全是另一盘菜了。