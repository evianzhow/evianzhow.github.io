---
layout: post
title: 在 OpenWRT 上编译 ss-server
date: '2014-06-16 04:00:00'
tags:
- openwrt
---

目前在 OpenWRT 上比较[常用](http://openwrt-dist.sourceforge.net)的 ShadowSocks 预编译的安装包都没有提供对 ss-server 的支持，而且根据 shadowsocks-libev 版的作者的话，不建议在嵌入式设备上运行 ss-server。但是，如果你对自己的 Router 性能比较有信心的话，而且你有在 Router 上运行 ShadowSocks 服务端的需求的话，你需要手动编译 ipk 安装包了。

此处假设你已经拥有了一个完整 build OpenWRT的环境，然后按照 [shadowsocks-libev](https://github.com/madeye/shadowsocks-libev) 上的 README，修改 package/shadowsocks-libev/openwrt/ 下的Makefile文件。 大约在 60 行左右的位置有这一行： 
    
    $(INSTALL_BIN) $(PKG_BUILD_DIR)/src/ss-{local,redir,tunnel} $(1)/usr/bin

将 server 加入花括号内。由于 ss-server 需要 libpthread 库，所以需要在 DEPENDS 后加上 +libpthread，重新 make 就能生成带 ss-server 的 ipk 了。 安装的时候需要注意先安装 libpthread 库，再安装 shadowsocks-libev，libpthread 库在 build 的过程中会自动生成，故直接 scp 到 Router 上安装即可。