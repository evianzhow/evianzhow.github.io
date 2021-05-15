---
layout: post
title: Resolving Apache config problem after upgrading to OS X Server 5
date: '2015-09-17 10:19:29'
permalink: "/resolve-apache-config-problem-after-upgrading-to-os-x-server-5/"
---

今天，Apple 在 Release iOS 9 的同时对 OS X Server 也更新到了正式版本 (5.0)，OS X Server 虽然拥有一大堆非常华丽的[宣传语](http://www.apple.com/osx/server/)，但是实际在工作环境中使用到它的情况却是不多的。其原因有二：

1. 过度简化了配置操作，导致很多的配置文件的“非标准化”
2. 大量的 Production Server 采用的是 Linux 内核系统和 Debian 等发行版操作系统，导致出现了问题很难查询相关资料

虽然有以上的两大严重缺点，但是和 OS X 的完美结合、更新缓存、Xcode Server 依旧是 OS X Server 的魅力所在。另外 OS X Server 5 添加了 iCloud Drive 的内部缓存功能，可以大大减少同一账号下不同设备上 iCloud 数据的下载量了。

我在 Mac mini 上跑了 GitLab 和 Phabricator 两个 Web App，使用了 OS X Server 内置的 Apache 来当 HTTP Server。这两个 Web App 的配置文件以 Apache conf 文件形式存放于

`/Library/Server/Web/Config/apache2/other`

而 `/Library/Server/Web/Config/apache2/httpd_server_app.conf` 会通过 IncludeOptional 形式引入这之中所有的 *.conf 文件。

就在更新之后，我意外的发现这两个网站无法访问了，现象是访问这两个内网域名，会直接访问到 OS X Server 默认的 Introduction 页面。目测是因为 OS X Server 升级造成了配置文件的错误或者冲突。

首先我从 Apache 的版本入手，在查询 Apache 的大版本没有进行升级之后，我仔细查看了一下更新前后的目录更变 (Thanks to Time Machine)，然后我发现在 `/Library/Server/Web/Config` 之下多出了 Proxy 和 ProxyServices 文件夹。而解决问题的关键也是了解这两个文件夹是做什么的。

Before upgrading

![Before upgrading](/assets/img/archived/os_x_server_before.png)

After upgrading

![After upgrading](/assets/img/archived/os_x_server_after.png)

简单解释一下 Proxy 这个文件夹是做什么的。我们都知道 OS X Server 中集成了非常多的服务，比如 Wiki、Xcode Server 等，而这些服务几乎都有 Web 页面。苹果在 OS X Server 5 中将这些 Web App 都做成了 Reserve Proxy 的形式，而 Proxy 文件夹正是保存这些 Reserve Proxy 配置文件的地方，按照 conf 的处理顺序，原来 Bind 443 端口的 GitLab 的处理顺序就会在这些 Web App 之后，所以无法被访问到。

新版的处理方式也非常的简单，直接在 `/Library/Server/Web/Config/apache2/sites` 里添加你自己的网站配置文件，然后将 VirtualHost 绑定至 127.0.0.1 上的某个端口，ServerName 中填写对应端口。举例，我的 Phabricator 网站的配置文件叫 `0000_127.0.0.1_34580_ph.in.ald.io.conf`，那么配置文件内的 VirtualHost 是 `<VirtualHost 127.0.0.1:34580>`，ServerName 是 `ServerName your.internal.domain.name:34580`。这里注意，如果不是使用 OS X Server 预设的 34580 端口或者是 34543(SSL)，则需要在 `virtual_host_global.conf` 里添加 LISTEN 端口选项。

另外，如果放在 `/Library/Server/Web/Config/apache2/others` 里应该也是没有问题的，因为 IncludeOptional 这个选项依旧存在。

按照这样操作之后应该就可以解决由于更新 OS X Server 而造成无法访问网站的问题了。

另外我也在思考 OS X Server 的一些 Best Practice，比如是否应该使用 [webapp.plist](http://apple.stackexchange.com/questions/179366/configure-yosemite-server-webapp-with-launchd) 来代替传统的 conf，如果你有好的建议，欢迎分享。