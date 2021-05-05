---
layout: post
title: 通过 usbmux 真机调试 React Native iOS App 的方法
date: '2018-03-14 14:52:01'
tags:
- ios
- react-native
- usbmux
- peertalk
---

### 前言

在开发调试 React Native App 的过程中，我们需要将 Development Bundle 传输至模拟器或真实的设备以运行或者浏览变更，对于 Android 的开发调试过程，我们利用 `adb reverse tcp:8081 tcp:8081` 命令，可以将手机上的 8081 端口反向代理至电脑上的 Bundler 监听的端口，但是对于 iOS，则没有这样的命令，只能通过 Wi-Fi 方式进行传输。

在 Wi-Fi 情况不佳的环境下，这个传输过程变得相当缓慢，同时由于 Development Bundle 的环境下，Bundler 没有将 Image Assets 包括到 App 中来，导致 `require('./some_image.png')` 需要通过网络方式加载获取，或多或少都遇到一些加载不出来的情况。于是我在 React Native 的 ProductPains 中提出了这么一个 [Proposal](https://react-native.canny.io/feature-requests/p/ios-connect-development-server-over-usb-like-android-adb-reverse)。

第一个想到的方法就是利用 [usbmux](https://www.theiphonewiki.com/wiki/usbmux) 进行有线方式的传输。相信玩过 iOS 越狱的朋友应该对这个东西和它的 daemon 不陌生吧。目前基于这个方法进行数据传输的第三方库数 [PeerTalk](https://rsms.me/PeerTalk/) 最出名了，一个基于它的典型的应用就是 [Duet Display](https://www.duetdisplay.com)。

于是今天在查阅相关资料的时候，发现 Facebook 早就已经进行过了相同的实践，并公开了核心的代码，但是由于某种的原因，在最新版本的源代码中删除了这部分。我们在这个 [commit](https://github.com/facebook/react-native/tree/c4699d8b739c2b22017f8d88a3143bac48e3a2fa/Tools/FBPortForwarding) 中还可以看到相关的代码。

### RTFSC

要理解这部分的代码，首先要明白以下这幅流程图，流程图来自 FBPortForwarding 的 `README.md`：

<script src="https://gist.github.com/EvianZhow/3f0e3a1b41237f386addf60f61dd5294.js"></script>

接下来为大家详细解释这张流程图中的内容。之前，我们需要了解一下前序知识。

首先 `usbmux` 虽然底层是基于 UNIX socket 的，但是由于其拥有特殊的数据结构，在这里我们理解为比特流传输。此外 `usbmux` 支持 [多路复用](https://en.wikipedia.org/wiki/Multiplexing) 传输，我们后面会提到 PeerTalk 中对多路复用的支持。

PeerTalk 提供了 `usbmux` 上的一层抽象和封装，在此基础上的任何传输都需要创建 `PTChannel` 类的一个实例，其设计遵循了 iOS 的 delegate 模式，实现了数据的异步接收。`PTChannel` 中最小的传输单位为帧（Frame），借鉴 UNIX 中管道（Pipe）的思想 —— “一端的输入是另一端的输出”，一个 `PTChannel` 可以看成是一个 Pipe，一个 socket 连接也可以看成是一个 Pipe。

帧载有 `payload` 内容，也包含了 meta 信息。通过 API 可以了解到，有 `type` 和 `tag` 两个字段，`type` 是自定通信协议的一部分，`tag` 是多路复用情况下 demux 的依据。和 socket 类似的 `PTChannel` 中有服务器端和客户端两种角色，服务端无法主动发起连接，只能指定一个端口号监听；客户端根据 IP 地址和端口号进行连接，也可以使用

`- (void)connectToPort:(int)port overUSBHub:(PTUSBHub*)usbHub deviceID:(NSNumber*)deviceID callback:(void(^)(NSError *error))callback`

通过 USB 方式连接。

除了 PeerTalk 以外，FBPortForwarding 依赖于 [CocoaAsyncSocket](https://github.com/robbiehanson/CocoaAsyncSocket)，这个库可以进行异步的 socket 请求，后面会提到，也是通过 delegate 的模式实现异步的过程。CocoaAsyncSocket 中创建的 `GCDAsyncSocket` 对象，是普通的 UNIX socket，可以直接与 Bundler 通信，也可以直接传入某个端口设置监听。

接下来让我们分析一下从 App 请求 Bundle 到 Bundler 返回数据的整个流程。

通过 Wi-Fi 连接到 Bundler，大概是分为两个步骤：

1. 向 Bundler 发起一个请求 Bundle 的 Request，具体表现为带有 URL 参数的 `jsCodeLocation`
2. 以 HTTP 服务器模式监听的 Bundler 收到请求，进行文件的打包，并返回响应的数据

乍一看流程相当的简单，但实际上请求大概有判断 Bundler 地址、Bypass ATS 规则、选择调用的 RCTBridge 等等的步骤，而具体响应的实现则在 NSURLSession 上包装了一下，增加了更多的功能——例如判断服务器是否支持 multipart 分片传输、如何使用最佳传输方式等等。具体可以参考 `React/Core/Base` 中的具体实现。

如果我们要通过 `usbmux` 代理连接到 Bundler，大概有以下几步。

1. 设置 Development Bundle 的读取路径为本机（iOS）中的一个端口，具体表现为主机名为 `localhost` 的 `jsCodeLocation`
2. 在 iOS 端创建一个 socket 监听来自 App 的读取 Bundle 请求
3. 将 socket 接收到的数据经由 `PTChannel` 转发至 Bundler
4. Bundler 解析请求，向 `PTChannel` 返回响应的数据
5. `PTChannel` 向发起请求的 socket 转发 Bundler 的响应数据

```
App <-> socket <-> PTChannel <-> Bundler
```

还记得我们上面提到的 `PTChannel` 的 C/S 结构吗？事实上这个流程是这样的：

```
App <-> socket <-> PTChannel Server <-> PTChannel Client <-> Bundler
```

`PTChannel` 的 Server 运行在 iOS 端，Client 运行在 Mac 端。

如何在 `PTChannel`上模拟 socket 连接建立和断开的过程是一个难题，FBPortForwarding 给出了一个解决方案。它将 socket 建立、传输的动作定义为以下三种 `type`：

- `OpenPipe`
- `WriteToPipe`
- `ClosePipe`

以此设置为例：**8025** 做双方的 `PTChannel` 通信端口，Bundle 请求从 iOS 本地的 **8082** 端口转发到 **8025**， Mac 端从本地的 **8025** 转发到 Bundler 的 **8081**，配置 `jsCodeLocation` 为 `http://localhost:8082/...`。

1. App 发起一个连接到 `localhost:8082` 的 socket 连接，连接到了 `GCDAsyncSocket` 创建的监听 socket 上，随之转换成 `PTChannel`的 `OpenPipe` 的操作，同时在 App 端字典中分配一个自增序的 `tag`。此时如果没有一个可用的 `PTChannel` 则会持续重试 （[FBPortForwardingServer.m#L169](https://github.com/EvianZhow/react-native-with-peertalk/blob/88915ffb2e6d885d7a831842279c540b27841b40/ios/FBPortForwarding/FBPortForwardingServer.m#L169)），如果已有一个可使用的 `PTChannel` 则马上连接并发送
2. Mac 端收到 `OpenPipe` 帧，创建了一个对应 tag 的 `GCDAsyncSocket` 连接至 Metro Bundler 的连接，并记录在 Mac 端的字典中以便后续管理
3. App 向 `localhost:8082` 的 socket 写入 HTTP 请求内容，`GCDAsyncSocket` delegate 方法被调用 （[FBPortForwardingServer.m#L173](https://github.com/EvianZhow/react-native-with-peertalk/blob/88915ffb2e6d885d7a831842279c540b27841b40/ios/FBPortForwarding/FBPortForwardingServer.m#L173)），转换成 `PTChannel`的 `WriteToPipe` 操作
4. Mac 端收到 `WriteToPipe` 操作，从字典中获取之前创建的 socket 连接，并向其中写入请求，Bundler 接收到 App 发送的请求内容
5. Bundler 向请求 socket 返回数据，`GCDAsyncSocket` 的  方法被调用 （[FBPortForwardingClient.m#L162](https://github.com/EvianZhow/react-native-with-peertalk/blob/88915ffb2e6d885d7a831842279c540b27841b40/ios/FBPortForwarding/FBPortForwardingClient.m#L162)），转换成 `PTChannel` 的 `WriteToPipe` 操作
6. App 端接收到 `WriteToPipe` 帧，从字典中查询到对应 tag 的 socket，并向其中写入数据

大致的流程就是如此，如果你已经阅读了源代码的话，你会发现其他的代码几乎都在处理 `PTChannel` 终止、意外中端的情况，但核心代码就是这些。

### Fin.

在理解了整个流程后，接下来就是如何将它运用到你的工程中了，我这里给出了一个简单的方法，请移步 [https://github.com/EvianZhow/react-native-with-PeerTalk](https://github.com/EvianZhow/react-native-with-PeerTalk) 查看和使用。

### References

- [https://www.theiphonewiki.com/wiki/usbmux](https://www.theiphonewiki.com/wiki/usbmux)
