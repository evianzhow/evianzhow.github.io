---
layout: post
title: 极光推送 react-native SDK 使用中遇到的问题记录
date: '2018-03-26 07:05:02'
tags:
- react-native
- ji-guang-tui-song
---

**阅读前，请注意版本与时效性！**

版本：jpush-ract-native [2.1.11](https://github.com/jpush/jpush-react-native/commit/c3b91ed88b53cde199b6dd86f326899cc4f1be5d)

最近因为项目的需求，使用了在 GitHub 上开源的 [jpush-ract-native](https://github.com/jpush/jpush-react-native) 极光推送官方的 react-native SDK，在使用过程中踩了不少的坑，简单整理一下：

1. 错误码没有文档
2. 没有**同步方法**用于获取客户端与极光服务器 TCP 长连接（下简称极光服务器）的连接状态，只有一个注册回调的 `JPushModule.addConnectionChangeListener` 方法（iOS 还有另外一个方法 `JPushModule.addnetworkDidLoginListener`，但是由于非跨平台共用，不建议考虑）
3. 操作 Alias、Tags 必须与极光服务器连通才可以，即满足条件 2；连通状态下，两次调用操作 Alias、Tags 间隔必须超过 20s，解决方法是利用 `lodash.throttle` 做限制，大概的流程如下：
  <script src="https://gist.github.com/EvianZhow/87f15ae7066994fd924c251860efb18f.js"></script>
4. iOS 默认不会连接极光服务器，必须授权推送权限后，即在 AppDelegate 中回调了 `- (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken;`，然后调用了 `[JPUSHService registerDeviceToken:deviceToken];` 后，才会发起连接
5. iOS 首次授权推送权限，利用 2 中提到的 `JPushModule.addConnectionChangeListener` 监听会返回 false、true、false，正常应该仅是 false、true，多一个 false 回调但连接已成功，导致上层应用和 SDK 中的网络状态不一致
6. Custom Message，iOS 收到的 message，extends 栏为解析好的 JSON 对象，Android 收到的的是序列化后的 JSON 对象字符串
