---
layout: post
title: Things you should know about using IBInspectable / IBDesignable
date: '2015-10-14 15:38:29'
---

Thanks to NSHipster's article [IBInspectable / IBDesignable](http://nshipster.com/ibinspectable-ibdesignable/)

Apple 在 Xcode 5 中加入了新的功能 IBInspectable / IBDesignable，可以让我们自定义的 UIView 在 Storyboard 中无需编译后运行就可以看到预期的结果，不过这个功能也有一定的局限性，不过对于我这种代码 + Storyboard 的开发者来说，节省了不少的时间在编写例如 `button.cornerRadius = 2.0f` 等等的工作上。

> But there is No Silver Bullet

**首先**遇到的一个局限是，对于 IBDesignable 的 UIView 在编译时遇到 Error 的话，是无法在 Build Log 里看到详细的编译过程，而且 Error 的描述也非常的简单，一般只能通过 Google 来查看其他人在什么一个情景下遇到这个错误的，然后去解决。

**第二个局限**，对一个 Class A（UIView 子类）添加 IBDesignable，和对一个属性添加 IBInspectable 没有问题，但是这个“性质”不会继承到这个 Class 的子类，如果你的设计中包含了继承的关系，父类的属性是无法在子类的 Inspector 里看到的。

**第三个局限**，对于使用 `#import <>` 方式引入的 UIView 子类，是无法在 Storyboard 中被 Compile 的，所以你需要将源代码添加到 Project 中才可以显示。

**第四个局限**，只有一部分的类型可以在添加 IBInspectable 后能在 Inspector 中被选择，目前测试下来基本上是只只吃数值类型，比如 CGFloat, NSInteger, NSUInteger 等等。

以上是我在目前的项目中使用 IBInspectable / IBDesignable 遇到的一些问题，希望能够对你有所帮助。如果你遇到了以上的问题但是和我的描述不符合，可以留言给我，我们一起来解决你所遇到的这个问题。