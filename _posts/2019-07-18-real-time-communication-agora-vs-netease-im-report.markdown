---
layout: post
title: 实时音视频：声网 v.s. 网易云信——调研报告
date: '2019-07-18 08:36:04'
permalink: "/real-time-communication-agora-vs-netease-im-report/"
tags:
- report
- agora-io
- frontend
- rtc
- wang-yi-yun-xin
---

> 本文编写于 2019 年 7 月 17 日，请注意时效性！

## 前言

网易云信和声网（agora）两家都提供了实时音视频的 SDK，提供给各类开发者使用。网易云信提供了一整套的解决方案，从 IM 到音视频都有覆盖。相对的，声网主要聚焦在音视频部分，但目前也在逐步提供 IM 的部分功能，例如：实时的文本消息推送，但仍处于 Beta 的阶段（实时消息 SDK）。

我们的业务功能之一是医生和用户之间的远程咨询和诊疗，所以需要用到 IM 和实时音视频两个部分。IM 用于图文的咨询，实时音视频用于更为实时的远程咨询，例如心理咨询等。

我们的技术选型是：移动端 react-native 框架，前端 React 框架。最开始，我们基于以往的经验和在部分朋友的推荐下，选择了网易云信作为平台的一整套 IM + 实时音视频的解决方案提供商，但在使用的过程中却遇到了不少的问题，对于这些问题，网易云信的技术支持水平较差，导致部分 Bug 没有办法解决。幸好的是，这部分 Bug 这并不影响关键业务。

对于一个创业公司的小团队，我们有大量的业务需求需要处理，没有大量的人手用于开发、优化音视频部分，所以我们急切希望找到一个支持跨平台的、开箱即用的解决方式，让我们可以不花太大的力气对界面进行一些定制，达到一个总体较好的效果。但从实践中看下来，网易云信目前不是很能满足这一点，具体原因后面会解释到。

考虑再三，我们评估下来，网易云信实时音视频的性价比较低。所以目前有意向将实时音视频部分功能单独切换至声网，保留网易云信的 IM 部分，故进行了本次调研。 **所以本报告仅代表以我们的技术选型之下的评估，对于其他的技术选型，还请自行判断。**

## 现况

这里，先简单介绍一下网易云信的生态：

以下先对各个概念进行一个简单的解释：

> Demo：UI 和 SDK 的组合，是一个对于终端用户开箱即用（out-of-box）的程序，包含了即时通信的功能

> SDK：不包含终端用户交互操作界面（UI）的业务逻辑核心库，功能为实现业务、提供调用接口的类库。我们这里定义，一个完整的 SDK 由三部分 SDK 组成，即时消息 SDK（IM SDK）、实时音视频 SDK（RTC SDK）、推送 SDK（Push SDK）

<figure class="kg-card kg-image-card"><img src="/content/images/2019/07/NIM.png" class="kg-image"></figure>

**对于 Web 前端** ，IM 部分官方提供了 Web SDK，实时音视频部分官方提供了 Web API（有 WebRTC 版本，也有 NRTC）。官方有提供了一个 Demo，其 UI 非常简单，定制难度较大，无法适用于具体业务场景。所以，UI 需要由我们自行实现。

**对于移动端部分** ，IM 部分官方目前提供了 Native 和 react-native 两种 SDK。react-native SDK 从 18 年 7 月开始第一次提供，是将 Web SDK 基于 react-native 的 JavaScript 执行环境进行了适配。在此之前，仅提供了 Native SDK，需要由用户自己做 react-native 的 binding。GitHub 上目前比较常用的一个 binding 是 [react-native-netease-im](https://github.com/reactnativecomponent/react-native-netease-im)。

实时音视频方面，目前只有官方提供 Native SDK 部分，且只有在官方 [Native Demo](https://netease.im/im-demo)中有完整功能实现和演示，故 UI 部分需要开发者自行开发或从官方 Native Demo 中抽离。对于安卓项目，官方已经拆分为了子项目 avchatkit，包含了 UI。但 iOS 没有另外拆分，需要自行拆分，不仅要拆分业务代码，还需要拆分 UI 部分。需要注意的是，实时音视频 SDK 依赖于 IM 的 Native SDK。

官方提供了一个 Demo：[NIM\_ReactNative\_Demo](https://github.com/netease-im/NIM_ReactNative_Demo)，第三方也提供了一个比较完整的 Demo：[react-native-chat-demo](https://github.com/reactnativecomponent/react-native-chat-demo)，从 Demo 中可以抽离 UI 部分。第三方 Demo 出现时间较早，早于官方。对比官方和第三方的 Demo，官方 Demo 仅简单实现了发送、接收消息的功能，例如：最初版本连发送语音消息的功能都未实现。后续在缓慢迭代中终于支持，目前迭代的频率为几个月一次，未来会不会持续更新还是一个未知数，官方仓库中 Issue 也无人处理。不过好在此 Demo 的 UI 部分使用 JavaScript 编写，没有用到 Native 部分，后期定制可能性尚可。第三方 Demo 使用了基于大量 Native 组件的仿微信 UI 界面 [react-native-imui](https://github.com/reactnativecomponent/react-native-imui)第三方库，可以基本做到开箱即用。虽然目前 UI 层面的完成度和还原度比较高，但实现原理是在极光开源的 [AuroraIMUI](https://github.com/jpush/aurora-imui)的基础上进行大量修改，开发者支持偏弱，后期定制可能性几乎没有，维护成为了一个比较大的问题。

* * *

综上所述，以下是目前几种可能的实现方案组合：

<!--kg-card-begin: markdown-->

| IM SDK | 实时音视频 SDK | IM UI | 实时音视频 UI | Pros | Cons |
| --- | --- | --- | --- | --- | --- |
| [react-native-netease-im](https://github.com/reactnativecomponent/react-native-netease-im) | 提取自官方 [Demo](https://netease.im/im-demo)，并自己做 binding | [react-native-imui](https://github.com/reactnativecomponent/react-native-imui) | 提取自官方 [Demo](https://netease.im/im-demo) | 
- 比较完整的解决方案，目前也是最稳定的解决方案
 | 
- 音视频版本受制于 [react-native-netease-im](https://github.com/reactnativecomponent/react-native-netease-im) 中包含的 SDK 版本
- 实时音视频话单反馈在安卓平台上有问题
- 调用 Native 的组件和 UI，跨平台开发调试非常麻烦和不便
- 软件包体积过大不便于升级 react-native 版本
 |
| Web SDK | 提取自官方 [Demo](https://netease.im/im-demo)，并自己做 binding | 提取自 [NIM\_ReactNative\_Demo](https://github.com/netease-im/NIM_ReactNative_Demo) | 提取自官方 [Demo](https://netease.im/im-demo) | 
- IM 和 UI 部分都是 JavaScript 原生，跨平台，方便维护
- UI 部分可以方便替换为 [react-native-gifted-chat](https://github.com/FaridSafi/react-native-gifted-chat) 或者 [aurora-imui](https://github.com/jpush/aurora-imui) 的版本
 | 
- 官方未提供音视频的解决方法，导致依旧需要使用到原生 IM SDK
- 音视频占用另外一个登陆终端数
- 软件包体积过大
- 通话话单无法同步到 IM 的聊天界面中
- IM 和实时音视频部分交互困难
 |

<!--kg-card-end: markdown-->

如果选用网易的一体化解决方案，有以下几个问题：

1. 因为账号体系的问题，实时音视频无论如何都避免不了引用 Native 的 IM SDK 情况，应用包体积难以控制；
2. 官方对 react-native 的支持偏弱，[NIM\_ReactNative\_Demo](https://github.com/netease-im/NIM_ReactNative_Demo)中也没有看到未来支持实时音视频的计划；
3. 如果选择 [react-native-netease-im](https://github.com/reactnativecomponent/react-native-netease-im)，则需要保证实时音视频库的版本号与其内部 NIM SDK 版本号保持一致，尽可能减少问题的发生，否则可能会出现意料之外的问题；
4. UI 层面，如果提取自 [NIM\_ReactNative\_Demo](https://github.com/netease-im/NIM_ReactNative_Demo)，需要进一步定制开发，如果直接引用 react-native-imui，则自身改动的可能性不大，依赖于第三方的开发者来更新。无论是采用官方 Demo 还是第三方库，其支持都比较弱；
5. 对于没有独立的安卓和 iOS 研发工程师的团队，对实时音视频界面定制化可能性不高，唯一选择就是从官方 Native [Demo](https://netease.im/im-demo)中提取；

而且，在使用过程中，我们还发现了一些小问题，例如：Web 端的信令解释与 Native SDK 对信令的解释不同。因为信令枚举值采用的是数字表示，Native SDK 发送某个信令在 Web SDK 中解释成了不同的信令。这可能是因为版本不同造成，但 SDK 本身开发的过程中应该注意到一致性和向前兼容性，且咨询工程师后无果。此问题一直没有处理。

## 概览

可以通过现况了解到，网易的实时音视频的业务逻辑是附加在其 IM 之上的，这个和声网形成了差异。声网作为一家专业的音视频解决方案提供商，实时音视频作为其主要的业务提供，且独立提供，没有提供 IM 的完整功能，只提供了部分功能：例如实时发送文本消息等。附加功能是为了信令而服务。

声网提供了音视频通话 SDK 和实时消息 SDK 两种 SDK，虽然均只提供了 Native 版本，但在开源社区的维护之下，react-native 的版本与官方几乎可以做到同步更新。同样的，社区中的 Demo 数量众多，文档较网易更为清晰。因为暂未付费，所以仅站在免费用户角度上来评估，官方的支持论坛响应时间短，且有工程师直接进行对接支持，已经可以达到了网易云信付费的水平。有一个完整的社区，比通过 QQ 群进行咨询，效率高出了不少，也便于查找和留存。

## 主要差异

在进行调研之后，我们发现，如果要把实时音视频从网易云信切换到声网，会遇到以下几个问题：

1. 用户体系区别：网易云信音视频，沿用的是网易云信 IM 的那套账户体系；声网在实时音视频中，没有用户体系的概念，只有在信令系统中才有账户体系的概念；
2. 概念区别：就像上面说的，网易云信的音视频业务逻辑是附加在其 IM 之上的，所以与单个用户建立“点对点”音视频通话，就类似手机直接拨打该用户的电话（ID）一样直接，由网易云信的服务器负责呼叫对方。但声网逻辑不同，声网是以“多人会议”的思路来处理点对点的音视频通话的，点对点的音视频通话只是多人音视频会议的一个特例。首先需要创建一个频道（为了辅助理解，我们称之为会议室），然后各方 **主动** 加入这个会议室中，加入时仅需要提供一个自身的 UID 即可，这个 UID 的要求是——只要保证会议室内的用户间不重复即可。主动加入，那也是主动离开。如果任何一方挂断本次通话，不会影响另外一方。可以类比为，会议室是一个房间，大家可以进入这个房间，也可以离开这个房间，但会议室本身不会消失，如果会议室里面没有人，就代表没有进行中的音视频通话。这是概念的区别；
3. 信令的区别：在 2 的认知基础上，网易云信实时音视频 SDK 部分包含了针对信令的部分处理，例如被叫用户响铃、被叫用户是否接受通话、主叫被叫用户是否挂断等信号传输，且如果从官方 [Demo](https://netease.im/im-demo) 提取了实时音视频的 UI 部分的话，同时会将信令接收后的处理逻辑一并提取，做到开箱即用。但声网则需要运用到他家的另外一项服务：信令或者实时消息（Beta）来传输这些内容，传输数据格式没有定义，需要开发者自行定义。用户需要先登录信令服务器，然后才能接受信令消息，最后对接收到的信令进行处理。这就对开发者提出了设计的要求；
4. 离线推送：在 2 和 3 的认知基础上，如果用户不在线，例如 App 的使用者，在没有打开 App 的情况下，网易云信 IM SDK 配置了推送功能，并支持小米、华为、vivo 等推送渠道，可以快速将被叫的推送信息推送到用户手机中。但声网并没有提供这个功能，也就是说，需要实现这个功能，需要由开发者自身建立一个推送渠道，并且在服务器上实现推送逻辑。例如：需要另外搭配极光推送使用；

以下用一个表格进行网易云信和声网实时音视频业务方面的对比：

<!--kg-card-begin: markdown-->

| | 网易云信 | 声网 |
| --- | --- | --- |
| 用户账户 | 与 IM 挂钩，用户信息在 IM 中存储 | 加入音视频通话时只需提供 UID，也可以不提供，由服务器自动生成；信令系统有账户的概念 |
| 支持类型 | 点对点、多人会议 | 多人会议 |
| 加入方式 | 主动邀请 / 被动接受 | 主动加入 |
| 信令系统 | 内置 | 外置 / 亦可开发者自行实现 |
| 推送通知 | 内置 | 无，需开发者自行实现 |
| 音视频之间切换 | 需重新加入，不可无缝切换 | 无需重新加入，关闭摄像头即可，可单方面开启摄像头 |

<!--kg-card-end: markdown--><figure class="kg-card kg-image-card kg-card-hascaption"><img src="/content/images/2019/07/---------.png" class="kg-image"><figcaption>图片引用自网易云信多人通话的文档，我们可以看到可以看到声网的逻辑，和网易云信的多人通话是一致的。但网易多人通话集成了声网的信令传送部分，更为简单。</figcaption></figure>
## 流程介绍

下面介绍一个完整声网音视频通话流程次序，流程参考了 GitHub 上 [AgoraIO/Advanced-Video](https://github.com/AgoraIO/Advanced-Video)仓库中 OpenDuo 实现：

1. 系统中的用户 A 准备呼叫另外一个用户 B，用户 A 端称之为主叫端，用户 B 端称之为被叫端
2. 主叫端使用 App ID 初始化 Agora SDK
3. 主叫端请求应用服务器，获取用于登陆信令服务器的 _Token A_
4. 主叫端使用 _Token A_登陆信令服务器
5. 主叫端在 UI 界面中选择用户，并点击呼叫按钮
6. 主叫端向应用服务器发送一个请求，表明要呼叫另外一个用户（被叫者），由服务器端返回加入频道的 _Token B_和频道名 _Channel Name_
7. 主叫端使用 \<_Token B_, &nbsp;_Channel Name_\> 加入（创建）音视频频道
8. 主叫端使用 \<_Token A_, _Channel Name_\> 加入信令频道，并开始监听频道事件
9. 主叫端的 UI 显示呼叫中，同时通过信令服务器向被叫端发起远程通话邀请
10. 这里我们假设被叫端在线，且已登陆信令系统。被叫端收到 _RemoteInvitation_消息，_RemoteInvitation_消息中包含了音视频频道名 _Channel Name_，被叫端的 UI 界面上显示请求通话的相关内容，被叫端可以选择接受，也可以选择拒绝
11. 被叫端选择接受主叫者的音视频通话邀请，并加入音视频 _Channel Name_和信令频道 _Channel Name_
12. 被叫端加入房间后，监听信令频道的事件
13. 主叫端接收到由信令系统传输的“接受邀请”信息，界面从呼叫中转换为视频中
14. 主叫和被叫进行视频通话
15. 通话结束，任意一方点击挂断，离开音视频频道，同时也离开了信令频道
16. 另外一方从信令频道监听到离开频道事件，如业务上需要自动挂断，则响应该消息：离开音视频频道，同时也离开了信令频道

当然，更多的情况例如：被叫端不在线、通话过程中断线等等异常情况，则需要更详细的处理，此处只是列举了音视频通话的一种常见的情况。我们可以看到，如果基于声网要进行一个带有主叫和被叫的音视频通话，在建立一个音视频频道的基础之上，还需要同步建立一个信令频道，这两个频道互不干扰，名称可以保持一致，也可以不一致。主要思想即是，由信令来控制音视频频道的相关操作。同时对于接收到的信令，需要自行处理。

以网易的文档进行对比：

**呼叫 / 接听流程** ：

<!--kg-card-begin: markdown-->

| | 网易云信 | 声网 |
| --- | --- | --- |
| 主叫发起通话请求 | 主动调用 SDK | 信令系统提供 API |
| 被叫收到通话请求回调 | SDK 回调 | 信令系统提供 API |
| 被叫响应通话请求 | 主动调用 SDK | 信令系统提供 API |
| 主叫收到被叫响应回调 | SDK 回调 | 信令系统提供 API |
| 呼入的通话已经被该帐号其他端处理回调 | SDK 回调 | 无 |
| 通话建立结果回调 | SDK 回调 | 无 |

<!--kg-card-end: markdown-->

**挂断流程** ：

<!--kg-card-begin: markdown-->

| | 网易云信 | 声网 |
| --- | --- | --- |
| 结束通话 | 主动调用 SDK | 主动离开房间 |
| 收到对方结束通话回调 | SDK 回调 | 需要自行实现 |
| 通话异常断开 | SDK 回调 | 无法实现 |
| 话单通知 | SDK 回调 | 需要自行实现 |

<!--kg-card-end: markdown-->
## 解决方式

从网易云信迁移到声网的最主要的差异，一是用户账户系统，二是信令设计、处理的部分。

对于用户账户系统，可以沿用业务系统中的 User ID，但需要根据业务的不同考虑后续的扩展：例如会不会有一个用户的多台设备同时加入音视频会议？对于信令部分，呼叫、接听流程没有太多需要开发的工作，但挂断流程，声网没有像网易云信提供一些封装好的接口，这个会导致开发者需要做一些额外的工作量。同时如何安全处理断网掉线的情况，也是在设计之初就需要考虑的。我们在上面的讨论中都没有提及离线推送的内容，如果加上离线推送，开发的复杂性会更上一层楼。考虑到离线推送，也可以抛弃声网提供的信令 SDK，由开发者自身完全搭建一套包含离线推送的信令系统也是一个选择。具体选择哪种，还视情况而定。可以参考上面提到的 [Demo](https://github.com/AgoraIO/Advanced-Video/tree/master/Video-Call-with-Chat/OpenDuo-Web) 中代码实现。

## 总结

从价格方面考虑，网易云信如果开通实时音视频，要求每个月最低消费 1000 元，其中包含音视频各 1000 分钟，超过部分按照使用量收费；而声网每个月赠送 10000 分钟的实时音视频通话，后续再按照使用量收费。两者计算价格方式不同，但对于音视频通话量级较少的企业，如果要快速集成音视频功能，同时也没有什么历史包袱的话，采用声网是一个不错的选择。

站在开发者的角度，如果企业拥有独立的前端工程师和移动端工程师，网易云信实时音视频未尝不是一个好的解决方案。但是对于人员编制相对紧凑的团队来说，如果一个类库或者组件，官方的支持不够强大，而且也没有开源社区支持的情况下，就应该考虑是否可以切换到其他更“对开发者友好”的替代产品。无论是从 Demo 数量还是 GitHub 社区的支持程度上说，声网都要比网易云信好不少。

参考目前用户的口碑，声网在实时音视频上的技术实力应该是强于网易云信的，国内大量的创业企业也都采用了他们的产品。虽然在迁移的过程中会带来痛苦，但后续的扩展性应该会更好。

**无论如何，进行这种迁移都会带来风险，需要和团队之间进行深度的讨论再做决定** 。

## 参考

声网实时音视频 SDK：

- [https://docs.agora.io/cn/Agora%20Platform/downloads](https://docs.agora.io/cn/Agora%20Platform/downloads)
- [https://github.com/syanbo/react-native-agora](https://github.com/syanbo/react-native-agora)

声网 Demo：

- [https://github.com/AgoraIO-Community/Agora-RN-Quickstart](https://github.com/AgoraIO-Community/Agora-RN-Quickstart)
- [https://github.com/AgoraIO-Community/OpenAgoraWeb-React](https://github.com/AgoraIO-Community/OpenAgoraWeb-React)

声网技术资料：

- [https://rtcdeveloper.com/t/topic/15255/3](https://rtcdeveloper.com/t/topic/15255/3)
- [https://github.com/AgoraIO/Advanced-Video/tree/master/Video-Call-with-Chat](https://github.com/AgoraIO/Advanced-Video/tree/master/Video-Call-with-Chat)
- [https://rtcdeveloper.com/t/topic/571](https://rtcdeveloper.com/t/topic/571)
