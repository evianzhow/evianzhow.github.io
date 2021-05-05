---
layout: post
title: 使用 sniproxy 隐藏 SSL 握手时的域名以躲过 ISP 嗅探
date: '2019-11-27 06:16:44'
tags:
- https
- isp
- shang-hai-dian-xin
---

最近在论坛上有不少同学反馈，由于将家中 NAS 的服务暴露到了公网，其中包含了管理页面的 HTTP 或者 WebDAV，导致 ISP 发现并断网发文通知整改。其中有部分回帖称其没有使用 HTTPS 而导致流量被运营商嗅探到。但使用了 HTTPS 服务就万无一失吗？不是的。就算使用了 HTTPS ，也有可能在 SSL 握手时域名遭到泄漏，导致运营商可以通过域名形式访问到用户开启的网页服务。

**当然，运营商如何操作的，对于我们来说是一个黑盒，运营商可能通过抓包等方式获取请求中的明文的 SNI 信息。要确保万无一失，您还需要使用加密的 SNI。**

> 需要注意的是，如果您的 ISP 未分配给您互联网上的地址，此篇教程不适用您的情况。请移步 frp 进行端口映射。

* * *

我们假设以下的这种情况：

> 某用户使用了群晖的 NAS，默认 5001 为 DSM 的 HTTPS 连接方式，默认的 5006 为 WebDAV 的 HTTPS 连接方式。而且某用户使用了群晖自带的 DDNS 服务，注册的域名为：customname.synology.me。

> 用户在路由器上做好了端口映射的设置（5001:5001，5006:5006），暴露给公网的端口，已经不包含 HTTP 明文传输的网页服务了。

某段时间，该个用户因为使用了 BT 等软件造成短时间的流量过大，导致进入运营商的关注列表。运营商使用了端口扫描的方式嗅探该用户的端口。

扫描下来，发现用户的路由器上开放了两个端口，5001 和 5006，且嗅探时返回可能为 HTTPS 协议。

运营商使用：openssl s\_client -connect x.x.x.x:5001 嗅探 HTTPS 的握手信息：

<!--kg-card-begin: markdown-->

    …
    CONNECTED(00000003)
    depth=2 O = Digital Signature Trust Co., CN = DST Root CA X3
    verify return:1
    depth=1 C = US, O = Let's Encrypt, CN = Let's Encrypt Authority X3
    verify return:1
    depth=0 CN = customname.synology.me
    …

<!--kg-card-end: markdown-->

运营商使用 https://customname.synology.me:5001 顺利打开 NAS 的管理页面，判定用户违规架设了 HTTP 类服务。一键三连。

* * *

在这个过程中，关键之处在于： **运营商一定要掌握到你的域名地址信息，且能通过 IP + 端口或者域名 + 端口的组合形式访问到** 。例如 DSM 使用的 Web 服务器，默认只绑定了一个域名和一个证书，在 SSL 握手时服务器会主动发送该证书，导致域名的泄漏。所以我们要做的，就是 **始终不让运营商掌握到我们使用的域名（通过 HTTPS），而且要保证，当运营商起疑心，通过技术手段扫描我们的端口时，不能使其访问成功（通过 sniproxy）** 。

通过 [sniproxy](https://github.com/dlundquist/sniproxy) ，我们可以做到：

- 不暴露自己的域名信息
- 阻断 IP + 端口的 HTTPS 访问（不出现证书错误页）
- 只有匹配到正确的域名时才允许访问

* * *

下面直接给出设置的过程，以我自己的 DS918+ 为例，DS918+ 支持 Docker，我将使用 Docker 方式运行 sniproxy。如果您的 NAS 并未提供 Docker 支持，您也可以通过其他方式运行 sniproxy，例如树莓派等等。

1. 在 DSM 的 Docker 中，搜索 Image：_[austinchou0126/sniproxy]( https://hub.docker.com/r/austinchou0126/sniproxy/)_ 
2. 下载 Image，创建一个 Container，创建时设置如下：Port Settings 中，删去容器端口为 80 的条目，手动设置一个未占用的端口，例如将 15001 映射至容器的 443 端口（请不要选择默认的 Auto，否则会由 Docker 自动分配一个端口）
3. Environment 中，添加环境变量：_SNIPROXY\_LISTEN0\_PROTO=tls_、_SNIPROXY\_LISTEN0\_PORT=443_、_SNIPROXY\_LISTEN0\_FALLBACK=127.0.0.1:4443_（这里选择任一一个无法访问的端口，以阻断不带域名的 HTTPS 请求）、_SNIPROXY\_TABLE0\_SRC0=customname.synology.me_（这里填入您的 DDNS 域名）、_SNIPROXY\_TABLE0\_DEST0=x.x.x.x:5001_（这里填入您的 NAS IP 和 DSM 的端口），更多的环境变量请参考 README
4. 创建并运行这个 Container，此时，您就拥有了一个端口为 15001 的 sniproxy 实例，将会把带有 customname.synology.me 域名访问的 HTTPS 请求转发给 x.x.x.x:5001
5. 使用 openssl s\_client 进行验证和测试：如果通过 IP + 端口访问，将不返回任何一个证书；如果通过域名 + 端口访问，将返回一个正确的证书信息。测试方式：（openssl s\_client -connect x.x.x.x:5001 与 openssl s\_client -connect x.x.x.x:5001 -servername customname.synology.me）
6. 配置您的路由器的端口映射，将原本直接映射至 NAS 的端口经由 sniproxy 进行转发
7. 对于其他的 HTTPS 服务（例如 WebDAV），执行同样的步骤

> 配置后流量：Router[:5001] \<-—\> Docker on NAS[:15001] \<-—\> NAS[:5001]

相比于使用 VPN 或者内网穿透这两种方式，使用 sniproxy 有以下的优点：

- 延续原来的操作方式，而不必改动其他设置，几乎兼容所有的应用
- 公网可访问
- 不需要经过中间服务器中转，速度即宽带的最大上行速度

* * *

最后，我建议大家，如果您继续选择将服务暴露给在互联网上，请做好以下的防范措施：

- 尽量开启 NAS 管理的二步验证
- 在出口路由器上关闭 ICMP 响应
- 任何暴露的网页类型服务，请务必使用 HTTPS
- 使用 sniproxy 来隐藏 SSL 握手时的域名信息

这里补充说明一下，使用默认端口的好处是：在群晖的 App 中，无需再次输入自定义的端口号，只需要输入域名即可。您也可以不适用默认端口，这都取决于您。

如果您觉得这篇文章对您有帮助，欢迎在 [GitHub](https://www.github.com/EvianZhow/docker-sniproxy) 上给我的这个项目点个 Star，并且分享给有需要的人。

## 拓展阅读

- [V2EX - 论如何伪装家庭宽带 HTTP 服务](https://www.v2ex.com/t/622336)
- [V2EX - 家庭宽带 私设 web 被检测 魔都电信被停宽带](https://www.v2ex.com/t/608821#reply407)
- [V2EX - 调查一下，上海电信私设 web 服务被封的，有没有用 https 的](https://www.v2ex.com/t/622013#reply30)
