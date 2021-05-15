---
layout: post
title: 我和墙斗争的那些年
featured: true
date: '2019-11-26 05:46:51'
permalink: "/days-with-gfw/"
tags:
- firewall
- free-internet
- openwrt
- gfw
---

记得刚上网的那时候，还没有墙，Google 还没有退出中国，谷歌在中国的域名是：[g.cn](http://g.cn)。如此简短又优雅的域名，放眼当今互联网，我想，也只有相声界大师罗老师的 [t.tt](http://t.tt/) 能与之一战了。

后来，也记不清什么时候，谷歌就退出中国了，所有 [g.cn](http://g.cn) 的请求都会被重定向到 [google.com.hk](https://www.google.com.hk/)，但也只是重定向，没有不可以访问。

再到后面一些时候，访问 Google 时会偶尔提示无法访问，慢慢地情况愈演愈烈，到最后就完全无法访问了。那应该我在读高中，也掌握了一点点计算机的知识，大概知道网络封锁是怎么一回事了。被封锁的网站通常的表现是，页面一闪而过，然后浏览器提示连接中断。

<figure class="kg-card kg-image-card kg-card-hascaption"><img src="/content/images/2019/11/---.PNG-2.png" class="kg-image"><figcaption>「自由门」软件截图（图片来源于网络）</figcaption></figure>

亏得我高中语文老师是个激进派，让我接触到了很多当时反封锁的手段。当时用的最多的是“自由门”、“无界”这种程序，如果早期翻墙的用户，应该都对这两款的软件很熟悉吧。这两款软件虽然免费，后台服务器也众多，但在学校网络下经常无法连接上，就算连上去速度也不快。而且仅仅只能通过电脑的浏览器使用，非常不便。后来不知道怎么的在网上瞎逛的时候，了解到了一款商用的 VPN 服务，可以使用各个平台都支持的 PPTP 的协议连入，这是我第一次接触到这样的商用服务。当时我把这个方案推荐给了那个语文老师，最终她掏的钱买了服务，我偶尔借着用一用，就像现在蹭 Wi-Fi 似的。后来用着用着，不知道怎么的服务器也连不上了，现在想想，估计也是跑路了吧。

到了高三，没有那么多时间用来上网，就没有在这方面继续了解。上了大学，开始使用起当时火热的 [GAEProxy](https://github.com/madeye/gaeproxy)。各家都在做 PaaS 平台，新浪有 SAE，谷歌有它的 GAE。Google 也给开发者免费提供了 5 个 GAE 实例的配额，每天每个实例可以有 1 GB 的免费流量。而这一些只需要注册一个 Google 账户即可享用。下载 GAEProxy 并按照它的操作，一步步部署到 Google App Engine 后即可开始翻墙，整个过程花不了半小时。不过，GAEProxy 最大的问题便是需要在电脑上安装自签名的证书，这带来了一些潜在的安全性问题。不过，能够提供免费的翻墙的服务可不多，GAEProxy 既不花钱且速度又能保证，不算复杂的部署脚本成为了当时翻墙的首选。

智能手机的时代来临，自己拥有了第一台 iPhone，便开始研究如何在 iPhone 上翻墙。为此，尝试过自建 PPTP、L2TP/IPSec、OpenVPN 等等。也就是在那个时候，开始使用了 DigitalOcean、Vultr 等等虚拟主机供应商的服务。当时还没有现在网上流行的一键安装的脚本，所有的命令都需要自己手动来敲。这个过程中也算学到了不少关于 Linux 的知识。使用的过程还算方便，但唯一的缺陷就是，一旦连到 VPN 之后，就算是国内的请求，也要跑到国外绕一圈回来，所以使用不同应用时，需要频繁开关 VPN。为了解决这个问题，又开始着手研究 Cisco AnyConnect VPN，因为它是当时为数不多的支持按需连接和路由分流的 VPN 服务。

又是在一个偶然机遇下，了解到曲径这款商用服务，当时它最吸引我的一点是，通过安装一个描述文件，可以让 iPhone 在移动网络环境下直接达到翻墙的效果。当时也购买试用了一段时间，曲径的连接方式有别于当时的 VPN 服务，用起来有一些特殊，也诞生了一个新词叫 APNP，具体的大家可以看他们自己在 Tumblr 上写的[介绍文章](https://getqujing.tumblr.com/post/82246191116/闲谈vpnapn和曲径)。

<figure class="kg-card kg-image-card kg-card-hascaption"><img src="/content/images/2019/11/Shadowsocks_logo.png" class="kg-image"><figcaption>shadowsocks</figcaption></figure>

后来，开源界出现了一个神器——[shadowsocks](https://github.com/shadowsocks/shadowsocks)。不再需要 VPN 那样的复杂设置过程，也不需要像 GAEProxy 那样使用自签的证书，只需要花几分钟在本地和服务器上简单配置部署，就可以访问和使用，多平台支持。不管你是 Windows 还是 Linux 亦或是 OS X，都有客户端的支持。当时 OS X 上有一个知名的 GAEProxy GUI 客户端叫做 [GoAgentX](https://github.com/OldFrank/GoAgentX)。GoAgentX 对 shadowsocks 协议做了支持，导致可以无缝从 GAEProxy 切换到 shadowsocks。所以在很短的时间内，我就切换到了 shadowsocks。

<figure class="kg-card kg-image-card kg-card-hascaption"><img src="/content/images/2019/11/goagentx.jpg" class="kg-image"><figcaption>GoAgentX</figcaption></figure>

> 这里，请大家记住 shadowsocks，因为现在无论是自用还是商用的翻墙服务，可以说在背后做技术支撑的软件， 90% 以上就是 shadowsocks 或其变种。shadowsocks 俨然已经成为翻墙界 de facto 的标准。shadowsocks 的创始人 clowwindy，可以说为整个世界的自由互联网做出了极大的贡献，值得被大家永远记住。

回到正题。大二的时候，我进了大学里的一个计算机俱乐部。俱乐部里原来有一台服务器，大家给了它一个昵称——“46”。“46” 是一台运行 Debian 发行版的 Linux Server，上面跑了很多的服务，例如 FTP、SAMBA 等等，同时兼具网关的作用。当时学校正好里给俱乐部批下来一台刀片服务器，用于支持学院的一些工作，俱乐部的老师希望可以把这台服务器做虚拟化，这样不同的工作可以独立运行在不同的虚拟机中，互不干扰，资源也可控。借此契机，我开始着手研究 ESXi 和相关的集群。

<figure class="kg-card kg-image-card kg-card-hascaption"><img src="/content/images/2019/11/1280px-Openwrt_Logo.svg.png" class="kg-image"><figcaption>OpenWRT</figcaption></figure>

由于对网络方向比较感兴趣，外加大一的时候因为学院的科技创新项目，对 [OpenWRT](https://openwrt.org) 有所耳闻，所以开始翻阅一些 OpenWRT 的相关知识。OpenWRT 作为路由器开源操作系统方面的开山鼻祖，强大的生态和插件的支持让我十分心动。所以在家里房子重新装修的时候，买了好几台 NETGEAR 的 WNDR3800。当时吸引我购买此款的一点就是，它有着 OpenWRT 的官方支持，而且刷机特别方便，可玩性特别高。

有了 ESXi，有了虚拟服务器，玩过了 OpenWRT，我开始在想，能否把网关从 Linux 服务器中分离出来，并使用 OpenWRT 重新构建。这个举动在当时非常新潮，互联网上能够找到的资料也不多，不过好在 OpenWRT 官方就已经提供了固件的 vmdk 版本，这个过程中没有遇到什么问题，没有花费太多的时间就顺利完成了迁移。虚拟化的网关，现在很多人都家用网络中采用这种方式，大家给它起了一个好听的名称——“软路由”。

<figure class="kg-card kg-image-card kg-card-hascaption"><img src="/content/images/2019/11/210247w49lf9suj2z8uj9s.png" class="kg-image"><figcaption>现在大家主要使用的是 KoolShare 论坛发布的 LEDE 软路由</figcaption></figure>

随着 [shadowsocks-libev](https://github.com/shadowsocks/shadowsocks-libev) 的出现，可以让 shadowsocks 跑在嵌入式的平台上了，同时 shadowsocks-libev 的 README 中也提到了，这个嵌入式的版本可以做成透明代理。这样的话，这个路由下的任何一个客户端，都不需要配置任何的代理程序，直接可以达到翻墙的效果。“_在终端上安装代理软件的时代要结束了！_”这是一个多么令人激动的想法。但当时 shadowsocks-libev 的 README 只给出了一个全局翻墙的示例，没有提供分地区、国家进行代理的示例。当时根据路由表进行 VPN 分流的方案已经很成熟了，我根据 GitHub 上的 [ashi009/bestroutetb](https://github.com/ashi009/bestroutetb)，自己写了一个 iptables 的生成脚本，成为了最早在路由器上用 shadowsocks 做透明代理的那批玩家。这个方案最终在上面提到的那台俱乐部的 OpenWRT 网关上得到了最先应用。x86 的处理器处理速度够快，庞大的 iptables 也可以应对自如，相比之下家中的 WNDR3800 的 MIPS CPU 就略显吃力了。

“无缝翻墙”成了俱乐部的一大特色，给无数的师兄和师弟提供了便利。当时学校的教育网可以拿到原生的 IPv6 地址，且不限速，我使用的 Linode 也支持了 IPv6 网络，所以强制要求 shadowsocks 以 IPv6 协议连到我在美国的服务器。学校对 IPv4 连接做了 2 Mbps 的限速，所以当时居然出现了访问国外居然还要比访问国内还要的快的现象，那是一段快乐的时光。

再到后来，[luci-app-shadowsocks](https://github.com/shadowsocks/luci-app-shadowsocks) 出现了，我的自制脚本也顺利退出了历史舞台，改用 luci-app-shadowsocks 了。 **我也算经历了 OpenWRT 上的 shadowsocks 生态一步一步发展到现在的整个过程吧。**

大四的时候，帮一个创业公司搭建网络的机缘，接触到了 UBNT 的相关网络设备，这是我第一次接触到了商用级的网络设备。当时的路由器选用的是 UBNT 的 EdgeRouter，但 EdgeRouter 上还没有直接可用的 shadowsocks-libev 二进制文件，尝试在路由器上安装所需的 toolchain 和 lib 进行编译，但未成功。这条路不通，只能[研究如何在电脑上进行跨平台编译](https://blog.evianzhow.com/compile-shadowsocks-libev-and-chinadns-for-edgerouter-poe/)。最终，使用 luci-app-shadowsocks 中 ss-rules 脚本，外加使用 QEMU 虚拟机编译下的 ss-redir，在 EdgeRouter 上顺利用上了透明代理方案。应该也属于全网研究这种方案的第一人吧。

<figure class="kg-card kg-image-card kg-card-hascaption"><img src="/content/images/2019/11/9ccd186a7471f7a363da03cacb88efeeb48259a7.png" class="kg-image"><figcaption>Surge</figcaption></figure>

这一切都没有解决在 iOS 上使用 shadowsocks 协议的痛苦，Android 上有 [shadowsocks-android](https://github.com/shadowsocks/shadowsocks-android)，iOS 却因为系统限制，一直没有一个好的 App，直到 Surge 这个软件的出现。Surge 的出现使得大家可以轻松通过规则配置自己想要的代理方案，甚至可以做出负载均衡、高可用等等一系列原本是高难度的操作。Surge 后来高昂的售价被人所诟病，但目前，还没有见到任何一个对标的软件有着 Surge 这样的功能和软件质量。

再到后来，墙的封锁越来越厉害，开始使用 DPI、主动探测机制封锁 IP 或端口之后，我便不再这个方向上继续折腾，开始改用了一些商用服务，因为折腾花费的时间成本和得到的收益并不成正比。如果要顺畅地翻墙，至少要开好几台不同地域的 VPS 主机，做高可用和负载均衡。 **Internet is hard, trust me.**

目前也出现了不少其他的翻墙方案，例如 V2Ray 等等，有短暂的了解但始终没有尝试过。纵观我和墙斗争的那些历史，我想，只要有墙存在的一天，就有拥有大量创造力的程序员与与之搏斗的一天，这场持久的拉锯战中，没有一方有绝对的优势，也没有一方曾经打败过任何一方，这场没有硝烟的战争，还将继续进行。

