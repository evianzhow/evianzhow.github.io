---
layout: post
title: Certificate, Entitlement, Provisioning Profile and App Re-sign
date: '2016-03-11 05:40:15'
permalink: "/certificate-entitlement-provisioning-profile-and-app-re-sign/"
tags:
- ios
- cerificate
- provisioning-profile
- signing
---

对于 App 的打包和签名过程，是许多 iOS 开发者在很长一段时间内不会接触到的东西，开发者只需要关心编译的时候会不会出现什么问题，而不会关心编译之后的发布上传操作做了什么。而 Xcode，已经可以很智能的帮我们修复大部分的问题，直到...

**你需要重签名一个 App 的 IPA 文件。**

我强烈建议你在阅读下面内容之前，先阅读一下 objc.io 里的 [Inside Code Signing](https://www.objc.io/issues/17-security/inside-code-signing/) 文章好对标题里的那些名词有所了解。我不想在多费口舌在这些名词的解释上，让我们把重点集中在 App 的重签名上。*我们主要步骤参考了 iOSRE 论坛中 [这篇文章](http://bbs.iosre.com/t/targets-app/2664)，同时补充和修复了一些错误。*

假设我们有一个应用，现在有重签名它的需求，我们需要先准备好：

- 一台越狱设备
- 一个砸壳的软件，dumpdecrypted 和 Clutch 都可以
- 此应用正轨途径获取的 IPA 文件

准备好以上这些，我们就可以开始了。

#### 砸壳

对于上传到 iTunes Store 的应用，Apple 在后台做了两件事 —— 重新签名证书、[FairPlay](https://en.wikipedia.org/wiki/FairPlay) 加密。FairPlay 技术将应用的二进制文件进行了加密，导致直接对一个正规渠道获取的 IPA（iTunes Store 下载 / iPhone 回传已购项目），你是无法通过 class-dump 获取其头文件的。

你可以通过在解压 IPA 之后的目录中使用
`otool -arch armv7 -l OUTPUT_FOLDER/Payload/APP_NAME.app/APP_NAME | grep crypt` 来查看是否二进制文件被加密。未加密的二进制文件 grep 后是没有任何的内容的。

对于一个应用的重签名，我们需要先解密 FairPlay，Dump 出未加密的二进制文件才行。这需要你有一台越狱设备并在设备上安装 dumpdecrypted 库或者 Clutch。具体的步骤可以访问以下链接获取：

1. [用dumpdecrypted给App砸壳](http://bbs.iosre.com/t/dumpdecrypted-app/22)
2. [KJCracks/Clutch: Fast iOS executable dumper - GitHub](https://github.com/KJCracks/Clutch)

到了这一步，假设你已经有 Dump 出的 IPA 文件了，下一步根据 Inside Code Signing 那篇文章里的说明，我们应该编写一个 Entitlements 文件。这里需要注意的是，我们其实没有必要手写一个 Entitlements 文件。在没有砸壳的二进制文件中，之前的 codesign 已经包含了 Entitlements 文件，所以我们这里也可以直接 Dump 出来。主要利用的工具有 [ldid](http://iphonedevwiki.net/index.php/Ldid) 和 codesign 两种。

#### Entitlements

使用正轨途径获取的 IPA 文件，获得 Entitlements 文件。砸壳之后的应用由于没有了原来的签名，所以也无法获取 Entitlements 文件，因为 Entitlements 是写在签名里的。

如果你是使用 ldid，使用 `ldid -e OUTPUT_FOLDER/Payload/APP_NAME.app/APP_NAME ` 获取 Entitlements 文件。

如果你使用的是 codesign，使用 `codesign -d --entitlements :- OUTPUT_FOLDER/Payload/APP_NAME.app > app.entitlements` 获取 Entitlements 文件。

**如果为多 Target 的 IPA 文件，需要获取所有的 Entitlements 文件。**

#### Bundle ID

对于 IPA 目录内的 Info.plist, .xcent 和 Entitlements 文件，更换其 Bundle ID 为你自己的 Identifier，同时更换所有的 Team Identifier 为你自己的 Team Identifier，**对于多 Target 的应用，需要修改相应的 Appex 中的内容。**

#### Provisioning Profile

获取了 Entitlements 文件之后，下一步是生成所对应的 Provisioning Profile。如果你的应用不需要任何特殊权限的话，例如 iCloud Container 或者 Personal VPN，可以使用带有通配符的 AdHoc Provisioning Profile，否则请新建一个 App ID，再创建相应的 Provisioning Profile。**对于多 Target 的应用，需要创建对应的 App ID 和 Provisioning Profile。**

创建完毕后，将下载的对应的 Provisioning Profile 移动至对应的目录，即 `OUTPUT_FOLDER/Payload/APP_NAME.app/` 下，并重命名为 embedded.mobileprovision，Appex 扩展的 Provisioning Profile 也是执行一样的操作，但是是移动到扩展的目录下。

#### Re-sign

用你自己的开发者证书签名新的 App，注意签名的顺序，扩展类的应用先签名，最后再签最外层的 .app 目录，需要遵循其文件结构的顺序。

重新签名的操作如下：

```
codesign -f -s "CERTIFICATE_NAME_IN_KEYCHAIN" --entitlements EXTENSION_NAME_ENTITLEMENT.plist OUTPUT_FOLDER/Payload/APP_NAME.app/PlugIns/EXTENSION_NAME.appex
codesign -f -s "CERTIFICATE_NAME_IN_KEYCHAIN" --entitlements APP_NAME_ENTITLEMENT.plist OUTPUT_FOLDER/Payload/APP_NAME.app
```

#### Re-package

将文件夹重新打包成 IPA。
```
cd OUTPUT_FOLDER
zip -qr ../RESIGNED_APP_NAME_.ipa .
cd ..
```

**记得重新打包的时候删除我们产生的 Entitlements 文件。**


### Install

将创建后的 IPA 拖动到 Xcode - Devices 中就可以进行安装了，下方可以看到安装过程的日志。

#### FAQ

1. 安装的时候出现了问题？

  在 Inside Code Signing 一文里，你应该熟悉了应用签名的流程，出现相应的错误提示，查找相应的步骤是否做的正确。

2. 特殊权限的应用无法安装或者工作不正常？

  例如 Surge for iOS 这个应用，其用到了苹果给签发的 Entitlements，只有签发了这个 Entitlements 的开发者账户才可以创建具有特殊权限的 Provisioning Profile，所以会导致签署之后的应用出现无法安装，安装后无法加载相应 VPN 模块的权限问题。

3. 签名所用的证书有什么特殊要求？

  没有，个人和企业的都可以签名，只是个人的 Distribution 证书签名的 IPA 只能安装在 Provisioning Profile 内含 UUID 的设备上。

如果你重新签名的过程中遇到了任何的问题，欢迎留言和我探讨。

#### References

- http://reverseengineering.stackexchange.com/questions/1594/possibilities-for-reverse-engineering-an-ipa-file-to-its-source
- https://medium.com/@ssowonny/resigning-ios-app-ipa-with-multiple-targets-and-provisioning-profiles-d868e5a9f70f
- http://www.jianshu.com/p/ad29445eb91c
- http://bbs.iosre.com/t/topic/275
- http://stackoverflow.com/questions/25329510/ios8-extension-needs-own-provisioning-profile