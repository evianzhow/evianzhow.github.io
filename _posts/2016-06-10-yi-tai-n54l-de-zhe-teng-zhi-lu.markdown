---
layout: post
title: 一台 N54L 的折腾之旅
date: '2016-06-10 16:49:45'
permalink: "/yi-tai-n54l-de-zhe-teng-zhi-lu/"
tags:
- esxi
- n54l
---

很早之前入了一台 HP 的 N54L，主要由于它强大的性价比，使得市面上所有的 4-Bay 的存储系列产品都比不上这款 HP 出品的微型服务器。

按照我这个折腾的性格，不对 N54L 进行什么改造是不太可能的，查阅了网上的一些资料之后，采购物料，也算终于搞定了 ESXi + Synology DSM 的方案，这篇文章算是对 N54L 的折腾之旅进行一个总结。

### Hardware Upgrade

- 我这台 N54L 原装了 4G 的内存，但由于共享显存，划分走了一小段内存，导致在 ESXi 5.0 后续版本安装中没有满足最小需求 4GB 的条件，我加了一条 [KVR16E11S8/4](http://item.jd.com/783267.html)。价格 CNY 256。
- ESXi 不支持板载的软 Raid，需要自己安装硬件 RAID 卡，比较之下，选择了 HP P410 Smart Array + 512M Cache + Battery 模组。选择理由比较简单，一个是原厂出品的 RAID 控制卡，兼容性肯定能够保证，第二是目前淘宝拆机的这种卡比较多，看上去应该像洋垃圾，但相比于从美国邮一个新的控制卡相比，个人觉得能正常工作就可以。价格约为 CNY 160。
- 第二块网络卡，随便找了一个[半高挡板的卡](https://www.amazon.cn/gp/product/B00H8H1HJG/ref=oh_aui_detailpage_o01_s00?ie=UTF8&psc=1)，Realtek 芯的。价格 CNY 76。
- ESXi 启动 U 盘，以小为主，容量够用就可以。我选择了[SanDisk CZ43 USB3.0 16GB](http://item.jd.com/1194107.html)。价格 CNY 35。

**改造成本 < CNY 550**

准备齐全后，进行软件方面的改造，基本思路就是 ESXi 做虚拟化。之前惊奇的发现了 VMWare Fusion Pro 7 提供了对 ESXi 主机访问的支持，让我对这个解决方案的好感度又上升了一个层次，再也不用开一台 Windows 机器跑 vSphere Client 了。

### Software Upgrade

由于安装的 ESXi 5.5 系统停止了对 Realtek 消费级的网卡驱动支持，所以需要使用 [ESXi-Customizer-PS](http://www.v-front.de/p/esxi-customizer-ps.html) 脚本创建 net-r8168 和 net-r8169 的支持。但是需要注意的是不适用于 ESXi 6.0 的方式，具体的支持方式在脚本的 Tutorial 中有。

使用定制的 ESXi 启动安装没有遇到任何的问题，RAID 也可以顺利识别，创建完毕后可能会遇到这个问题：
```
Configuration Issue: XXX esx.problem.syslog.nonpersistent.formatOnHost not found XXX
```
VMWare 的[这个 KB](https://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=2050689) 里也很好解释了。

DSM 按照这个[教程](http://xpenology.me/installing-dsm-5-1-vmware-esxi5-5u1/)安装就可以，可以直接使用网站上提供的最新 PAT、ISO 和 VMDK 文件。过程中需要注意手动指定安装的 DSM 文件，而不是通过网络方式下载安装，否则 DSM 6.0 不兼容 Xpenology 的引导。

### Summary

总体上来说，安装完虚拟化之后，对服务器的整体可玩性增强了，各个虚拟机的职能划分也更加清晰。这篇文章主要提供给大家一个改造的思路，至少我使用这些硬件都是兼容 N54L。

另外根据改造的成本核算下来，也比买群晖官方的 4-Bay 系列来的便宜得多，唯一的缺点就是没有办法始终升级到最新的 DSM 系统了。