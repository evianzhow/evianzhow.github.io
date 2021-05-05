---
layout: post
title: 关于比亚迪云服务 App 逆向过程的经验
date: '2017-06-08 04:40:30'
tags:
- ios
- reverse-engineering
---

在 IT 公论的某期通讯（「IT 公论周五通讯：物联网和共产主义（2015.04.10）」）里写道：

> 当然，接下来就是为什么的问题。和 geek 们的信念相反，「因为我能」从来就不构成足够的理由。我能，我上。互联网早期的先锋们正是这么做的。卷起袖子干的实用主义精神延续到了现在，演化成为对行动力的礼赞和快速发布反复迭代的行动纲领。

由于个人实在是无力吐槽国产品牌车配套的应用设计水准，秉持着上面提到的「我行我上」准则，想自行为比亚迪云服务设计一个 App，在逆向过程中踩了一些坑，而且坑比较难填，暂时放弃，现在将分析思路的大概过程写在这里供后来人参考。

1. 首先从网络请求下手，在进行了一些 HTTPS 抓包实验之后，发现其所有的 API 请求和响应均经过一个叫 [梆梆安全公司](https://www.bangcle.com/solutions/index.html?category_id=9) 的加固 SDK 进行加密，大部分的请求和响应都类似于 `{ "Data": "密文" }`。由于该 SDK 是非公开的，很有可能是为了比亚迪的 IoT 设备进行了特殊的定制，后续没有深究下去
2. 既然从网络请求层面走不通，尝试逆向二进制文件，从 PP 助手网站下载越狱版的 IPA，即是脱壳后的版本，无需我们再有越狱机器去脱壳
3. IPA 解压后，将 Binary 拖入 Hopper Disassembler 分析，同时使用 class-dump 导出头文件。网络请求的大部分函数在 `Communication` 类下，且均是类方法。其中尤其注意：
`+ (void)requestResultByHttps:(id)arg1:(id)arg2:(id)arg3:(id)arg4:(long long)arg5:(id)arg6`
和`+ (id)packageData:(id)arg1 withIdentifier:(id)arg2 key:(id)arg3 parser:(long long)arg4`。前者是网络请求主函数，其中调用后者进行数据的加密，最终调用了 BangSafeSDK 的 `checkcode:` 方法得到最终发送的字符串
4. 既然单纯的逆向 API 无法得到有用的结果，那么换种思路将 Binary 尝试嵌入我们的工程中，思路参考 [http://blog.imjun.net/posts/convert-iOS-app-to-dynamic-library/](http://blog.imjun.net/posts/convert-iOS-app-to-dynamic-library/)，具体的步骤：
    - 用 lipo 把 Binary 拆成 armv7 和 arm64 两个 Slice
    - 利用杨君给出的工具 [app2dylib](https://github.com/tobefuturer/app2dylib)，对两个 Slice 分别转换成 dylib
    - 再把两个 dylib 使用 lipo 合并成一个 dylib
    - 新建 Xcode 工程，拖入并通过 Objective-C 动态调用机制调用函数
5. 最终发现部分敏感函数（例如网络请求）依旧被保护，调用即闪退，时间仓促没有查明有何种问题，但怀疑与 ptrace、sysctl 等反调试机制有关

如果后来者想继续这方面的研究的话，个人建议从以下两个方向下手：

- 逆向 BangSafeSDK 的 `checkcode:` 和 `decheckcode:` 实现
- 破除反调试保护，可以选择尝试反-反调试手段，具体可以访问 [iOSRE](http://iosre.com) 论坛

后话：
相比于 TESLA 和其他合资品牌厂商，国内厂商做的云服务或者是车联网的 App 简直惨不忍睹，横向比较一下，比亚迪云服务的 App 居然算是做的不错的了。明明已经赶上合资品牌车的价格了，但是内部智能系统的设计还是一团糟，对于现在软件开发工程师和用户界面设计师并不缺乏的互联网时代，实在是显得非常落后。如果他们做不好，就应该把这个权力回归于用户。Tesla 在这点上做的还算不错，虽然没有公开申明开放 API，但是它做的相当的规范和标准、易于理解，也很开放。相比国内公司，真是天壤之别了。