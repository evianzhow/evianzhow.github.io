---
layout: post
title: 让 ShadowSocks 透明代理与 Privoxy 一起工作
date: '2014-07-16 04:00:00'
tags:
- openwrt
- shadowsocks
- privoxy
---

在我之前的文章中提到了可以在路由器上部署 ShadowSocks 客户端并转为透明代理供连接路由器的设备直接翻墙的方法。但是如果在 3G/4G 网络环境下的时候，想要翻墙便成为一件十分痛苦的事情，如果你不想使用更稳定的商业服务的话，可以参考本文。

一般来说没有越狱的 iPhone 普遍是通过 VPN 的方式来翻墙，越狱后的 iPhone 可以安装 Cydia 中 ShadowSocks 的完全体来翻墙（App Store 里的阉割版本基本没有办法在蜂窝数据下使用）。但是，还有一种方式可以不使用 VPN 来翻墙，这就是——采用 APN 代理的方式。整个过程简单的说就是安装一个描述文件，描述文件里定义了 Access Point Name 和 HTTP 代理。这样在 3G/4G 网络环境下也可以直接翻墙。ShadowSocks 是一个 Socket 代理，先可以利用 Privoxy 或 Squid 等软件来转化成 HTTP 代理，再让所有的HTTP 流量由经代理出去。

但是如果是透明代理的方式呢？Google 了一下，发现很少有人尝试过 ss-redir 搭配 Privoxy 使用。根据 Transparent Proxy 原理，我们在 nat 表中创建了一个 SHADOWSOCKS 的 Chain，然后利用这个 Chain 去过滤国内 IP。然后我们对 PREROUTING 应用 SHADOWSOCKS 这个 Chain，至此，对于国内的 IP，其流量便不会走 ShadowSocks。

理解了透明代理的原理之后，我们开始进入正文，首先安装 Privoxy，安装完后就只需修改一下 Listen Address，把这个改成 0.0.0.0，不需要设置 Upstream。这时如果你使用这个代理服务器的话，你会很神奇的发现就算是国外流量，也没有走 ShadowSocks。问题就在于，Privoxy 把 Packet 直接交给了 nat 表中的 OUTPUT 去处理。解决方法也很简单，也就是在 OUTPUT 上创建一条规则，如果是 Privoxy 发来的包就 Jump 到 SHADOWSOCKS 这个 Chain。 
    
    iptables -t nat -A OUTPUT -m owner --uid-owner privoxy -j SHADOWSOCKS

不过先别急着执行，假如是 Ubuntu 版本的话，执行这条命令是完全没有问题的，因为 Privoxy 以 privoxy 用户的身份去运行的。但是像 OpenWRT 这种只有一个 root 用户的话，肯定会出现找不到 privoxy 用户这样的错误。

一种解决方式就是将 privoxy 用户改成 root，这种算比较 Dirty 的方法。那么还有一种方式就是安装 shadow-adduser 的 package，然后添加一个 privoxy 用户，然后将 /etc/privoxy 文件夹和 /var/log/privoxy 文件的所有者改成 privoxy 用户，然后修改 /etc/init.d/privoxy 中的 start 参数，加上 
    
    --user privoxy

，然后再启动。 这样的话就可以成功将 Privoxy 与 ss-redir 搭配使用了。如果你的 Router 分配到的是公网 IP 的话，你就可以利用 Apple Configurator 生成带 APN 设置的描述文件，在 3G/4G 下翻墙了。不过由于某些开启了 SPDY 的 App 依旧是没有办法的。

**18 May 15 更新**：目前比较稳定的解决方式是使用 AnyConnect VPN，具体的教程可以参考 http://blog.zhowkev.in/2014/03/20/anyconnect-iphone/