---
layout: post
title: 初探 tvOS 的一些坑
date: '2015-12-10 16:01:03'
---

tvOS 是 Apple TV 4 Gen 之后搭载的操作系统，其源自于 iOS，和 iOS 共通一些 Framework，不仅如此，还提供了一种面向前端开发者更加友好的 [TVML](https://developer.apple.com/library/tvos/documentation/LanguagesUtilities/Conceptual/ATV_Template_Guide/index.html) 来构建 Client-Server 结构的 TV App。使得开发人员甚至是不需要使用 Objective-C 或者 Swift 就可以利用 TVML 的 Template 构建一个看起来不错的 App。

正好赶上一个小的 Break，我利用短暂的时间研究了一下 tvOS 上的开发流程，时间不长，但是遇到了一些比较烦人的坑。因此在这里贴一些小的 Tips 给大家，希望大家可以跳过我遇到的这些坑。

1. TVML 是什么？

  TVML 是一种 Markup Language，乍一看非常类似 XML，是拥有一些领域专用语言(DSL)的一种描述方法。利用官方提供的 [UI 模版](https://developer.apple.com/library/tvos/documentation/LanguagesUtilities/Conceptual/ATV_Template_Guide/TextboxTemplate.html#//apple_ref/doc/uid/TP40015064-CH2-SW8) 可以不用写 Objective-C 和 Swift 就可以构建一个大致符合苹果设计准则的界面。

2. TVML 可以和原生 UIKit 混用吗？

  可以，从 TVML 中调用 UIKit 的方式参考这里：https://forums.developer.apple.com/thread/18430。但是没有找到 UIKit 调用 TVML 的相关的答案，但是根据 TVApplicationController 继承自 NSObject 的关系，应该是没有办法在 Navigation Stack 上进行 Push 的操作的。

  >I find it odd that I have to create a new UIWindow for the TVML to show up at all - I tried using the existing window but no joy. 

  一种可行的方式是 Present TVApplicationController 实例的 navigationController 属性来达到类似的效果。

3. 为什么提示 **Can't find variable: navigationDocument**

  可能是因为你使用了 Objective-C 开发了 tvOS 应用程序，我也遇到了这个问题，只能切换到 Swift 进行开发，怀疑这个苹果的一个 Bug。https://forums.developer.apple.com/thread/22361 

4. 我应该如何选择 UIKit 和 TVML？

  我的建议是，如果你的 App 逻辑比较简单，而且内容经常变化的话，可以采用 TVML 来进行开发，因为可以很快加入原生的 View Controller，反之则比较困难。但是 TVML 是需要与服务器连接启动的，这个在使用的时候要斟酌一下。

5. 有没有一些比较好的教程呢？

  - http://www.raywenderlich.com/114886/beginning-tvos-development-with-tvml-tutorial
  - SDK Header is your best friend!