---
layout: post
title: Porting lrzsz to iOS
date: '2016-10-14 07:15:57'
tags:
- ios
- jailbreak
---

tl; dr
对于想直接安装 lrzsz 的用户，可以到 BigBoss 源中进行搜索，或者下载 deb 文件: [Dropbox](https://dl.dropboxusercontent.com/u/68922104/lrzsz_0.12.20-3_iphoneos-arm.deb)

--------

许多 iOS 的开发者手中必不可少的是一台越狱后的备用 iOS 设备，这台设备常常承担了以下的功能:

- 安装插件扩展 iOS 的可用性
- iOS 9 以前的设备用于翻墙
- 使用 Reveal, Cycript 等插件逆向某些 App

在使用这台越狱设备的过程中，文件的传输是一个比较麻烦的问题。从 Mac 向 iOS 传输数据可以使用 scp 命令，而从 iOS 向 Mac 传输数据只能靠一些 App 中 SFTP 功能来实现，比如 [Cyberduck](https://cyberduck.io) 等。

Linux 上有一个非常棒的工具 lrzsz，加上[简单的 iTerm 2 的配置](https://github.com/mmastrac/iterm2-zmodem)，就可以快速在两台设备间传输数据，例如我常用的发送一个文件到远程机器的命令:

> Remote Machine: `sz -e <filename>`

或者从远程机器接收一个文件的命令

> Remote Machine: `rz -e`

在 Cydia 中搜索了一圈，并没有找到对应的 port version，于是便自己动手，丰衣足食。

对于将一个现有的库 porting 到另外一个架构的设备上这种行为，有两个解决方法:

- 在 Host 机器上执行 cross-compile 的操作
- 在 Target 机器上安装 make, ld 等工具，然后直接编译

我这里采用的是 cross-compile 的方法，命令比较简单，就不一行一行解释了。
这里非常推荐阅读本文下方的 References 部分，本文的主要意义在于当你遇到类似的问题时，此篇文章可以提供一些索引和有价值的建议。

```
cd lrzsz-0.12.20/
./configure --includedir=$(xcrun --sdk iphoneos --show-sdk-path)/usr/include
make CC="$(xcrun --sdk iphoneos --find clang) -isysroot $(xcrun --sdk iphoneos --show-sdk-path) -arch armv7 -arch armv7s -arch arm64"
scp src/lrz src/lsz <iphone>:/usr/bin/
```

在编译过程中可能会遇到一些错误，根据对应错误修改源代码即可，一般来说通常是缺少某些头文件的引用导致的，很少需要修改实现。

对于第二种直接编译的方法，这里以一个简单的 C 程序举例，读者可以自行扩展到编译任意的文件:

```
// Minimal C example
#include <stdio.h>
int main()
{
   printf("This works\n");
   return 0;
}
```

1. 在 Cydia 中搜索安装 make, LLVM+Clang, Tape archive, iOS Toolchain, ldid, wget, OpenSSH
2. 获取对应 iOS 系统版本的 SDK: https://sdks.website/，iOS 系统正常情况下均没有包含 Standard C Library 头文件，只有在 SDK 中才包含，所以需要获取对应版本的 SDK
3. 将对应版本的 SDK 下载解压至任意目录，这里选择 `/var/sdks/` (没有特殊注明的情况下，目录均指 iOS 设备中的目录)
4. `cd /var/sdks/ && ln -s iPhoneOS<version_number>.sdk Latest.sdk`
5. `clang -isysroot /var/sdks/Latest.sdk main.c -o main`
6. 执行 main，如果遇到签名错误，执行 `ldid main`

最后，将生成的 binary 文件打包到一个 deb 中，提交到软件源。

### References

- 交叉编译的命令参考: [http://stackoverflow.com/questions/24735362/errors-when-cross-compiling-openssl-on-os-x-for-ios-with-clang-instead-of-gcc](http://stackoverflow.com/questions/24735362/errors-when-cross-compiling-openssl-on-os-x-for-ios-with-clang-instead-of-gcc)
- 目标平台编译过程参考: [http://iphonedevwiki.net/index.php/Compiling_iOS_applications_on-device](http://iphonedevwiki.net/index.php/Compiling_iOS_applications_on-device)
- 一个过时的目标平台编译过程参考: [https://cxwangyi.wordpress.com/2012/01/28/how-to-install-gcc-4-2-on-iphone/](https://cxwangyi.wordpress.com/2012/01/28/how-to-install-gcc-4-2-on-iphone/)
- Clang 命令行参数: [http://clang.llvm.org/docs/CommandGuide/clang.html](http://clang.llvm.org/docs/CommandGuide/clang.html)
- 创建 deb 包，注意打包命令有误，应为 `dpkg-deb -b <package_identifier>`: [https://pve.proxmox.com/wiki/Manipulate_your_first_DEB_Package](https://pve.proxmox.com/wiki/Manipulate_your_first_DEB_Package)