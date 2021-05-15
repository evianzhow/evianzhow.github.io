---
layout: post
title: Apple Music like Navigation Bar
date: '2015-07-18 12:32:25'
permalink: "/apple-music-like-navigation-bar/"
tags:
- ios
- apple-music
- navigation-bar
---

**Thanks to MoZhouqi, this repository [KMNavigationBarTransition](https://github.com/MoZhouqi/KMNavigationBarTransition) provides a cleaner way to achieve this effect. This article is discontinued.**

在 iOS 8.4 版本中，Apple 把原来的 Music App 改版了一下，同时增加了 Apple Music 功能，虽然我觉得在国内环境下使用实在是太蛋疼的一件事（同时伴随使用中的一系列问题），但是从 UI 的层面上，还是有不少值得研究的事情。

在使用中，我遇到了一个非常棒的效果，可以看一下这段视频。

YouTube:

<iframe width="420" height="315" src="https://www.youtube.com/embed/CasdrinpQL8" frameborder="0" allowfullscreen></iframe>

与以往的 Navigation Bar 不同的是，居然能够在滑动返回的过程中既保留了淡入淡出效果，而且能够保持两种 Navigation Bar 的样式的不一致。

如果你觉得上面那段话比较晦涩难懂的话，我们用代码来说明一下。

一般来说，要实现“透明”的效果，我们首先把 Navigation Bar 设置为 translucent。再需要像下面这样设置 Navigation Bar：

```
// in UINavigationController subclasses
[self.navigationBar setShadowImage:[UIImage new]];
[self.navigationBar setBackgroundImage:[UIImage new] forBarMetrics: UIBarMetricsDefault];
```

这里我们要说明一下这几个函数的调用顺序：

```
[self.navigationController pushViewController:animated:] (always success) 
|-> [newVC viewWillAppear] 
|-> [newVC viewDidAppear]

[self.navigationController popViewControllerAnimated] (fail or not)
|-> [oldVC viewWillAppear] 
|-> [oldVC viewDidAppear]
```

了解了函数的调用顺序，我们就可以开始动手了。首先在第二个 View Controller 中的 viewWillAppear 中设置 Navigation Bar 为透明，且设置 shadowImage 属性为 [UIImage new]，在第一个 View Controller 中的 viewDidAppear 中设置为正常（不透明），恢复 shadowImage 为 nil。

这样做的之后，当 pushViewController 开始之后，Navigation Bar 就会突然变成透明且没有 Navigation Bar 底下的一小条细阴影，然后动画完成。
当 popViewController 开始之后，Navigation Bar 会**突然**变成原来的非透明状态，并且恢复底部的 shadowImage 显示，非常的突兀，我们并不希望给用户呈现的是这么一个效果。

然后我就开始猜测苹果是怎么做的，有一个可以确定地方的是，苹果肯定不是重写了 Navigation Bar，用的是 UIKit 提供的 Navigation Bar。

于是想到的一个 Hack 方法也是最直观的方法，就是把 Navigation Bar 始终设置为透明的，在第一个 View Controller 的 view 上再添加一个 UINavigationBar 来达到以假乱真的目的。然后我根据这个的想法去做，大致思路正确，但是遇到了一些小问题：

1. 设置 backgroundImage 时会自动设置 shadowImage，有的时候这么做会产生一些我们不期望的行为。

2. 由于 shadowImage 被设置为 [UIImage new]，所以在设置为 nil，恢复的一瞬间，Navigation Bar 的 titleLabel 会向上偏移，这对于用户体验来说也是不可忍受的。

另外，在实际遇到这个问题的时候，由于第二个透明的 View Controller 是通过一个 Router 去唤起的，所以不确定是从哪个页面 Push 过去的，所以就必须做到耦合性比较低。

一个比较合适处理的地方就是 Navigation Controller，可以通过覆写其中的 pushViewController:animated: 和 popViewControllerAnimated: 来让Push 的过程中对之前的 View Controller 自动插入一个 Navigation Bar，Pop 过程**成功结束**的时候删除插入的这个 Navigation Bar。

对于 titleLabel 偏移的问题，我通过设置 Navigation Bar 的一个私有属性 _backgroundView 的 alpha 来让其变成透明。

大致的实现思路就是如此，具体的代码细节我就不在这里给出了，感兴趣的同学可以去我的 [GitHub](https://github.com/Her0n/Apple-Music-Navigation-Bar) 看看。Demo 代码写的有点仓促，如果有问题的话欢迎给我留言。

所以，最终的实现效果是这样的：

YouTube:

<iframe width="420" height="315" src="https://www.youtube.com/embed/wOKA09GDSXY" frameborder="0" allowfullscreen></iframe>

回过头来，我们再看看 Apple 是用了什么黑科技来做呢？我们在一台越狱的设备上安装了 Reveal Loader。

![Apple Music Navigation Bar](/assets/img/archived/15.png)

是的，原来苹果的开发人员也会为了设计做各种各样的 Hack 呀！😄
