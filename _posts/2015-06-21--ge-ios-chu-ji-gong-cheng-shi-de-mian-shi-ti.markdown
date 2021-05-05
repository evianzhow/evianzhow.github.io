---
layout: post
title: iOS 初级工程师的面试题
date: '2015-06-21 07:38:26'
tags:
- ios
- interview
---

之前由于在 D.G.Z 实习的时候，团队需要 Recruit 一些另外的 Cocoa/iOS 开发者，而 D.G.Z 目前还没有一个标准化的面试流程，所以用了半个下午的时间，结合之前面试知乎的经历整理了一些面试题。

我觉得，面试题的话主要应该是考察一个开发者是否有快速学习和解决问题的能力，所以并没有涉及非常深层次的问题，面试的过程中也应该以沟通能力和解决问题的思路为主。

部分题目内容来自：https://github.com/lzyy/iOS-Developer-Interview-Questions/blob/master/README.md

**一般性问题**

- 平时遇到问题是怎么解决的
- 介绍一下平时使用的开发者工具
- 有接触过 Swift 吗？简单介绍一下 Swift
- 平时使用的版本控制，介绍一下平时开发中怎么使用它们的
- 平时经常阅读的网站或者技术博客
- 最近使用的非常有意思的 App

**知识性问题**

- 介绍一下你经常使用的第三方库
- iOS 中实现多任务的几种方式
- Property 和 Instance Variable 是什么[关系](http://stackoverflow.com/questions/9544370/objective-c-property-and-synthesize-logic)？-> 开放性问题 #1
- Objective-C 中的 nil 有什么特性？
- 简单介绍一下 Delegate 的设计模式 -> 开放性问题 #5
- Objective-C 的 id 对象
- ViewController 的生命周期？ -> 开放性问题 #7
- NSNotification
- Auto Layout
- Retain Cycle 是怎么形成的
- HTTP 协议中定义了哪些动词，都分别代表什么意思？
- TCP 和 UDP 的区别，以及使用场景

**开放性问题**

1. 如果一个 Property 在 Header 里声明为 readonly，如何让它在实现里可以被读写？
2. 如果是 readonly 且实现了 getter method，那么 synthesize 这个 property 会发生什么事情？ `@synthesize topSpeed = _topSpeed` 是什么意思？
3. 声明方法的前缀为 alloc/new/copy 会导致什么问题？
4. const NSString * 和 NSString *const 有什么[区别](http://stackoverflow.com/questions/3196491/objective-c-const-nsstring-vs-nsstring-const)？
5. 不使用 IBAction 和 Storyboard，如何知道一个 ViewController 上的某个按钮被触摸了？
6. 怎么检查一个 id 对象内部到底是什么类型的？
7. 从网络上载入一个数据并显示在一个 ViewController 上，应该放在哪个 View Controller 的 Life Cycle Method 中？会阻塞主线程吗？有什么解决方式？
8. 设计一个可以自动滚动的 Slide（UICollectionView）
9. HTTP 协议中 If-Modified-Since 是有什么功能？如果让你自己来实现这个功能你会怎么做？
10. 如果一个 HTTPS 网站提示证书不可信，可能有几种情况？
11. 你觉得操作系统的核心是什么？