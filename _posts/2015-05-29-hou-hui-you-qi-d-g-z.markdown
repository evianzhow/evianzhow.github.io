---
layout: post
title: 后会有期 D.G.Z
date: '2015-05-29 07:27:32'
---

今年二月份的时候，在一个比较巧的时间点，我收到了来自 D.G.Z 团队的实习机会，主要工作是 iOS / OS X 应用的相关开发，工作形式是 Remote。

从三月初开始到五月份中旬的实习结束，总共在这个团队里呆了不到三个月的时间，在北京团队的 Leader 下，完成了 [ditto 视频](https://itunes.apple.com/cn/app/ditto-shi-pin-you-zhi-duan/id979919086?l=en&mt=8) 这款 App 的开发，以及 GitCafe for Mac 的 Alpha 版本的发布。

首先在这个过程中，我想感谢北京团队的 Leader [ZH](https://twitter.com/bjshdq)，无论是我最后由于个人原因离职还是平时的沟通之中，我们之间的交流都非常的愉快，同时也给予了我充分成长的空间，而我觉得这是很难在一个 Big Corp 中能体验到的。同时我也想感谢 D.G.Z 的 CEO [Thomas](https://twitter.com/ghosTM55)，给我一个加入他们团队的机会，虽然时间短暂，而且是 Remote 形式的工作，但是依旧能够感受到团队中的活力，而这个活力也是和 Thomas 息息相关的。

另外还感谢团队中其他的成员。

感谢完了，接下来想对自己的参与的一些工作做一个小结，也算是给自己交一个答卷。

1. 完成了 [ditto 视频](https://itunes.apple.com/cn/app/ditto-shi-pin-you-zhi-duan/id979919086?l=en&mt=8) 的开发

  采用 Swift 进行开发，也是我学习 Swift 后第一个真枪实弹的一个 App，在开发的过程中踩了不少 Swift 的坑，但是结果都还算不错。该 App 还在推广的过程中，每天新增的用户大约在 250 人左右。
2. GitCafe for Mac Alpha

  之前并没有过 Cocoa 应用的开发经验，但设计模式与 iOS 应用还是比较类似的，不过 Cocoa Framework 的功能更强一些。但是也正是因为它非常早就出现了，所以导致目前的资料和代码比较旧，之前找的 NSRulerView 的 Demo 都是基于 10.6 而开发的。

  在我进入这个开发组之后，与 ZH 一起完成了以下几个比较核心的功能：

    - 使用内嵌 Git Binary 的方式调用 Git 命令，完成一些远程的操作，并通过正则表达式获取进度
    - 使用 ObjectiveGit 的 API，完成一些本地的操作
    - 参考 GitX 完成了 Password Helper，在其基础上优化了 HTTPS 模式和 SSH 模式的提示
    - .gitignore 文件管理工具
    - Xcode Server Nightly Build + Dropbox Upload

  这些功能大部分都是在 GitHub 客户端中实现的，而 Alpha 版本阶段的目标就是功能上的完成度。

以上就是大致的工作内容，经过不到三个月时间的实习工作，我觉得对我个人的工作方式和工作效率都有了比较大的提升，今后如果让我 Leading 某个队伍的话，应该也不会是太大的问题。

感谢 D.G.Z 团队让我成长，后会有期。