---
layout: post
title: Compile shadowsocks-libev and ChinaDNS for EdgeRouter PoE
date: '2016-05-04 15:09:29'
permalink: "/compile-shadowsocks-libev-and-chinadns-for-edgerouter-poe/"
tags:
- edgerouter
- shadowsocks
---

#### Intro
EdgeRouter PoE is an enterprise-level router based on [Vyatta](http://vyos.net/wiki/Vyatta) operating system, which contains a dual-core MIPS CPU called Cavium CN5020 according to the [wiki](https://wikidevi.com/wiki/Ubiquiti_Networks_EdgeRouter_POE). 

Official shadowsocks-libev package source doesn't contain MIPS pre-compiled version, also EdgeOS has some compatibility issues prohibiting us from compiling packages on that system, so we have to compile by ourselves, in **an more-completed** system. 

Several people tried before, and from their research, we known that we have to use [QEMU](http://www.qemu.org) to emulate the same MIPS architecture and directly compile shadowsocks-libev and ChinaDNS on that system. 

### 0x7c00

At first, we should prepare an machine installed Linux and QEMU on it. 1GB memory and 15GB disk space is a lot enough. 

Then, download pre-installed Debian system disk image from [https://people.debian.org/~aurel32/qemu/](https://people.debian.org/~aurel32/qemu/) so you don't need to install system additionally, which will cost much time due to low efficiency under QEMU. Be careful to choose the right version. 

After running the image from instructed command, follow below ones. Explanation were made to help you understand better.

You might be ask to do a little interaction during the installation process. The whole compiling process might take up to an hour to finish, drink a beer and relax. 

```
# Clear out "no public key available" error
apt-get install -y --force-yes debian-keyring debian-archive-keyring
# Re-update
apt-get update
# Added backports to sources.list
echo "\ndeb http://ftp.debian.org/debian wheezy-backports main" >> /etc/apt/sources.list
# Install git in order to clone shadowsocks-libev and chinadns
apt-get install -y git
# Install shadowsocks-libev build dependencies
apt-get install -y build-essential autoconf libtool libssl-dev gawk debhelper dh-systemd init-system-helpers pkg-config
git clone https://github.com/shadowsocks/shadowsocks-libev.git
cd shadowsocks-libev
git checkout v2.4.6
dpkg-buildpackage -us -uc -i
cd ..
# Install chinadns build dependencies (optional)
apt-get install -y automake
git clone https://github.com/shadowsocks/ChinaDNS.git
cd ChinaDNS
git checkout v1.3.2
./autogen.sh
./configure && make
tar czf ../chinadns.tar.gz src/
cd ..
```

Also, if you prefer not to compile by yourself, tried these debs [https://www.dropbox.com/sh/adptfr0lufffw1c/AACFgDfEPsHM4HMozYwyPaFqa?dl=0](https://www.dropbox.com/sh/adptfr0lufffw1c/AACFgDfEPsHM4HMozYwyPaFqa?dl=0)

Remember to add `local_address":"0.0.0.0"` to your shadowsocks configuration to let client bind as 0.0.0.0 instead of 127.0.0.1, or iptables rules might not connect any more! 

Finally, use [this small tool](https://github.com/shadowsocks/luci-app-shadowsocks/blob/master/files/root/usr/bin/ss-rules) to help you create custom iptables rules to redirect proper packages. 

#### Updates

- More detailed information can be obtained from [https://www.lifetyper.com/2016/12/bypass-gfw-with-shadowsocks-and-dnsmasq-on-edgerouter-x.html](https://www.lifetyper.com/2016/12/bypass-gfw-with-shadowsocks-and-dnsmasq-on-edgerouter-x.html)

#### Ref

- http://www.bdwm.net/bbs/bbstcon.php?board=Networking&threadid=15744113
- https://www.onlyos.com
- http://www.cnblogs.com/zerosoul/p/5153651.html
