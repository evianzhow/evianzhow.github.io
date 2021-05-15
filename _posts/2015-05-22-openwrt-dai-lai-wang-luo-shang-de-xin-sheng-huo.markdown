---
layout: post
title: OpenWRT 带来网络上的新生活
date: '2015-05-22 18:04:46'
permalink: "/openwrt-dai-lai-wang-luo-shang-de-xin-sheng-huo/"
tags:
- openwrt
---

自从把自己的根据地转移回寝室之后，我就在琢磨怎么提升一下寝室内的住宿环境，而说到住宿环境，作为马斯洛最底层需求 —— 网络就变的非常重要了。

首先简单介绍一下寝室里的网络状况以及“改造”后的情况：

网络接入：

- 联通 10Mb 以太网接入，采用 PPPoE 认证方式，公网 IP 地址
- 学校教育网 100Mb 接入，采用锐捷验证方式，认证后 DHCP，内网 IP 地址，原生 IPv6

改造前：

- 仅联通网络接入

改造后：

- 联通网络和校园网双栈接入
- ChinaDNS + ShadowSocks 透明代理 + IPv6 不限速
- 校内站点单独路由规则

如果你的网络状况与我类似，同时你也想让你的寝室网络变得更好，不妨继续读下去。

由于原来只有联通网络一种接入方式，而由于本学期开始联通网络使用人数的急剧上升（这一切都是由于校园网的不给力导致的），网络产生了僧多粥少的局面，直接导致网络的质量下降。而国内 ISP 的国际出口在晚间高峰时间的不给力，也给我带来了相当的困扰，最直接的影响就是 ShadowSocks 变慢了。外加上原来路由器上没有做透明代理，iPhone 使用起来经常需要开 VPN，耗电严重。

在以上种种痛苦之下，我选择了给寝室换一个能刷 OpenWRT 的路由器。选择 OpenWRT 主要考虑到其第三方软件的种类非常多，相关的 Wiki 也比较全面，同时其社区也非常的活跃。在参考了 OpenWRT 网站上的 [Supported Devices](http://wiki.openwrt.org/toh/start) 之后，我决定购买 [Netgear WNDR3700](http://detail.tmall.com/item.htm?id=36432721444&spm=a1z09.2.9.137.EdWzFj&_u=ncl9urha66f) 这款。

**注意，我以下说的内容可能仅限于我的设备和我的网络环境，请勿在没有弄懂的情况下直接 Copy & Paste！**

拿到路由器后，首先我们应该是配置双 WAN 口的操作。大致的流程如下：

首先，我们打开 WNDR3700 的 [Wiki](http://wiki.openwrt.org/toh/netgear/wndr3700) 页面了解其内部的交换机的结构。
然后，SSH 到路由器上将一个 LAN 口从局域网的 VLAN 中去除，参考现有的 WAN 口 VLAN 划分，将这个 Switch Port 划入一个新的 VLAN 中。你的 `/etc/config/network` 的最下面几行应该看起来是这样的：

```
config switch
        option name 'switch0'
        option reset '1'
        option enable_vlan '1'

config switch_vlan
        option device 'switch0'
        option vlan '1'
        option ports '0t 1 2 3'
        option vid '1'

config switch_vlan
        option device 'switch0'
        option vlan '2'
        option ports '0t 5'
        option vid '2'

config switch_vlan
        option device 'switch0'
        option vlan '3'
        option ports '0t 4'
        option vid '3'
```

配置完 VLAN 之后，我们就要为这个新的 VLAN 创建一个 Interface 了。这里我们可以摆脱“丑陋”的命令行，改用 Web 上的 LuCI 来创建了。

过程很简单，在 Network -> Interfaces 下添加，为了方便识别，我们将这个新的 Interface 称之为 WAN2。选择相应的协议，不要勾选 **Create a bridge over multiple interfaces**，VLAN 选择新创建的 VLAN。

创建之后，需要将这个 Interface 添加到 WAN 这个 Firewall Zone 里，具体的细节我就不一一阐述了，JFGI。

这样，我们就让一个 LAN 口变成了 WAN 口。

接下来，由于校园网是通过锐捷认证的，所以我们需要一个运行在路由器上的 mentohust。OpenWRT 的 Build 环境同样不在本文中阐述了，我这里默认你有一定的 OpenWRT 使用经验。

将下面的 Makefile 放置到 `package/mentohust` 下。

```
#
# Copyright (C) 2006-2011 Xmlad.com
#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#

include $(TOPDIR)/rules.mk

PKG_NAME:=mentohust
PKG_VERSION:=0.3.1
PKG_RELEASE:=1

PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION).tar.gz
PKG_SOURCE_URL:=http://mentohust.googlecode.com/files/
PKG_MD5SUM:=c7033ba8d8e75294924ed03f4b7b0c45

PKG_INSTALL:=1

include $(INCLUDE_DIR)/package.mk

define Package/mentohust
	SECTION:=net
	CATEGORY:=Network
	DEPENDS:=+libpcap
	TITLE:=An CERNET client daemon
	URL:=http://code.google.com/p/mentohust/
	SUBMENU:=CERNET
endef

define Package/mentohust/description
An CERNET client daemon,
Most usually used in China collages.
endef

define Build/Prepare
	$(call Build/Prepare/Default)
	$(SED) 's/dhclient/udhcpc -i/g' $(PKG_BUILD_DIR)/src/myconfig.c
endef

CONFIGURE_ARGS += \
	--disable-encodepass \
	--disable-notify

# XXX: CFLAGS are already set by Build/Compile/Default
MAKE_FLAGS+= \
	OFLAGS=""

define Package/mentohust/conffiles
/etc/mentohust.conf
endef

define Package/mentohust/install
	$(INSTALL_DIR) $(1)/usr/bin
	$(INSTALL_BIN) $(PKG_INSTALL_DIR)/usr/bin/mentohust $(1)/usr/bin/
	$(INSTALL_DIR) $(1)/etc
	$(INSTALL_CONF) $(PKG_INSTALL_DIR)/etc/mentohust.conf $(1)/etc/
endef

$(eval $(call BuildPackage,mentohust))
```

> Tips from [Cross-compile for OpenWRT](http://worldend.logdown.com/posts/276005-cross-compile-for-openwrt): 縮排必須是一格 tab，不能是空白鍵，否則會產生 Makefile:17: *** missing separator. Stop. 錯誤。

然后重新 `make menuconfig`，并编译即可产生对应平台的 mentohust 安装文件。你可能需要另外编写一个 init script，不过我相信这并不能难倒你。

现在，有了 mentohust 并在路由器上安装好之后，我们就可以尝试认证一下了。如果不出意外的话，应该没有问题。如果你遇到了问题，可以在底下留言并指明你出错的地方，我并不保证一一解答。

认证完毕之后，你可以尝试一下使用这个 Interface 去 ping 一下外网，看看是否连通，如果不连通的话，可能是因为网关设置的原因，当然也有可能是其他的原因。如果你确定是由网关的引起的，可以按照我这个做法进行修改：在配置 Interface 页面，勾选 WAN2 的 **Use default gateway**。然后重新 SSH 到路由器上，继续对 `/etc/config/network` 这个文件进行修改。在 WAN 和 WAN2 配置下分配不同的 Metric，越小的数值优先级越高，我给它们分配的分别为 10 和 20。

[Metric](http://en.wikipedia.org/wiki/Metrics_(networking)) 是用于路由决策的一个参数，这里你只需要知道，当两个 Interfaces 都配置成 Default Gateway 的时候，通过 Metric 参数来决定哪个的优先级更高。

至此，我们就拥有了双 WAN 接入的路由器，而且还带有 Fail-Over 功能，当主线路失去链接的时候，自动切换至次线路。

更有趣，同时也是更有价值的的做法是将两条线路的带宽叠加起来做成 Load Balance，具体的实现可以利用 [mwan3](http://wiki.openwrt.org/doc/howto/mwan3) 这个 Package。我相信官方的 Wiki 比我这里写的更详细。

然后我们配置 ShadowSocks 和 ChinaDNS。感谢 [aa65535](https://github.com/aa65535) 给我们提供的 ShadowSocks 路由器优化版，我们可以直接添加 [http://openwrt-dist.sourceforge.net](http://openwrt-dist.sourceforge.net) 这上面的源并按照其说明进行安装，整个配置过程几乎不需要用到命令后。

透明代理也完成了，下一步呢？没错，就是 IPv6，我们如何将教育网优质的 IPv6 线路利用起来呢？大致上也是有两个思路。第一，我们可以使用 radvd 或者配置 OpenWRT Barrier Breaker 中的 odhcp6d 来做 IPv6 穿透，让路由器后的设备同样可以分配到来自 ISP 的原生 IPv6 地址。第二，我们可以利用拥有 IPv6 地址的国外 VPS，让 ShadowSocks 的流量走 IPv6。

我在 Barrier Breaker 早期版本中使用第一种方法的时候经常会遇到路由器后的机器无法获取到 IPv6 地址的情况，需要定时重启 odhcp6d 才可以，我不知道这个问题现在是否已经得到了修复。

配置 IPv6 也是非常容易，只需要修改 Barrier Breaker 中自带的 WAN6 Interface 的配置如下。

```
config interface 'wan6'
        option ifname '@wan2'
        option proto 'dhcpv6'
```

这里的 @wan2 对应的是我们第一步新创建的 Interface 的名字。需要注意的是，我记得官方 Wiki 中并没有加上 @ 的前缀，至少在我之前的测试下，如果不加 @，那么会导致 IPv6 配置的失败。

如果你的 ShadowSocks 设置 IPv6 地址失败，可以阅读一下这个 [Issue](https://github.com/shadowsocks/openwrt-shadowsocks/issues/32)。我通过设置一个名为 ipv6 的子域名并只提供 AAAA 记录来绕过这个限制的，不知道这个问题在新的版本中得到修复了没有。

我们的路由器改造已经几乎接近完成了！最后还缺的一个功能是针对校内的路由做优化。我从我们学校网络中心的这个 [页面](http://ito.hit.edu.cn/news/sub_swzn.asp?id=271) 中获取到 IP 地址段并在 `etc/hotplug.d/iface/` 下创建了一个脚本。这个目录下的脚本会在 Interface 重启的时候被执行。

```
#!/bin/sh

[ "$ACTION" = ifup ] && [ "$INTERFACE" = "wan2" ] || exit 0

logger -t route "$INTERFACE up, adding custom routes"

gateway=$(ip route show 0/0 | grep "dev eth0.3" | sed -e 's/.* via \([^ ]*\).*/\1/')

# ip range from http://ito.hit.edu.cn/news/sub_swzn.asp?id=271
ip route add 61.167.60.0/24 via $gateway
ip route add 222.171.7.128/25 via $gateway
ip route add 222.171.178.0/24 via $gateway
ip route add 221.212.176.0/24 via $gateway
ip route add 202.118.224.0/19 via $gateway
ip route add 219.217.224.0/19 via $gateway
ip route add 210.46.64.0/20 via $gateway
ip route add 202.118.168.0/22 via $gateway
ip route add 118.203.0.0/18 via $gateway
ip route add 118.203.64.0/20 via $gateway
```

注意这里的 eth0.3 需要修改成我们第一步添加的 VLAN Interface 名称。
这个 Hook Script 大致的意思就是，如果某个指定的 Interface 上线的时候，我在路由器的路由表里添加这些特定的路由规则，让符合规则的网段经过这个 Interface。

最后一步，我们需要检测 mentohust 是否因为 Interface 的突然断开而终止认证了，我们在定时任务里每分钟执行这个脚本。

```
#!/bin/sh

/etc/init.d/mentohust start

if [ -z $(pidof mentohust) ]; then
  /etc/init.d/mentohust start
fi

if [ -z $(pidof ss-redir) ] || [ -z $(pidof ss-tunnel) ]; then
  /etc/init.d/shadowsocks restart
fi

exit 0
```

脚本会检测 mentohust/ss-redir/ss-tunnel 中任意一个是否停止运行，如果停止则重启该服务。

至此，我们的路由器算是完全改造成功了。现在你拥有了一台功能能够秒杀其他 99% 的路由器了，希望这能够给你的生活带来新的体验。
