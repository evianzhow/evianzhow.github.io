---
layout: post
title: Some customizations to XLForm
date: '2015-11-21 04:25:12'
---

[XLForm](https://github.com/xmartlabs/XLForm) 是 [Xmartlabs](http://xmartlabs.com) 开源的一个 Form 组件，其设计的核心思想类似 HTML 中 Form 的编写方法，如果你对 HTML 中的表单不是很熟悉，可以看一下[这里](http://www.w3schools.com/html/html_forms.asp)。

在典型的 Web 开发中，我们通过 `<form>` 标记去定义一个 Form，然后通过不同的 `<input>` 来定义不同的输入。这样的表示方式非常的直观，也非常容易修改。但是 iOS 中并没有对 Form 类型的 UI 有任何可用的系统控件，使得开发类似表单提交的页面变得非常的麻烦。你需要处理一堆 UITableView 的 DataSource 和 Delegate 方法，还要处理一堆一堆的 Section，Row。很容易就把人弄晕了，代码也写的非常的乱。更不要说在运行时动态的改变 Form 的外观。

所以有了 XLForm 这么一个库的存在。至于它有多么的 Elegant，你可以看一下官方 Demo 中如何用不到两百行的代码制作出一个和 iOS Calendar App 创建 Event 一模一样的 Form 页面。

我的一个项目中，在用到了 XLForm的同时，也遇到了其设计上的一些局限性。这样的一个通用型的框架固然好，但是面对复杂的业务需求，可能你就要做一些自定义的操作了。在这里，我把我的做的一些自定义分享给大家，如果在你的项目中遇到了相同的需求，你也可以通过这种方式来解决。

1. Cell 的初始化过程和初始化步骤

  这里假设你对 XLForm 已经有大致的了解，在新建一个表单的时候，我们新建的是一个 XLFormRowDescriptor，通过设置 XLFormRowDescriptor 的三个 NSMutableDictionary，可以配置 Cell 的一些状态和行为。

  XLFormRowDescriptor 有以下三个 NSMutableDictionary

  1. cellConfig - 这个字典会在 `-(void)tableView:(UITableView *)tableView willDisplayCell:(UITableViewCell *)cell forRowAtIndexPath:(NSIndexPath *)indexPath` 调用 `- (void)update` 的过程中被枚举并设值（KVC），所以不适合做一些 Control 的**配置性**的工作。

  2. cellConfigAtConfigure - 这个字典会在 `- (void)config` 中被枚举并设值，此方法会在 `-(XLFormBaseCell *)cellForFormController:(XLFormViewController *)formController` 中初始化 Cell 时调用 `- (void)configureCellAtCreationTime` 被枚举并设值，非常适合做一个些 Control 的配置性的工作，且此 Dictionary 只会被调用一次。

  3. cellConfigIfDisabled - 这个字典会在初始化 rowDescriptor 且表单为 disable 的状态下被枚举并设值。

  由于我的项目中使用到了一个需要一个 rowCount 和 columnCount 来初始化，在初始化之后再通过对一个属性设值来让这个控件布局其中的某行某列的内容。如果将初始化参数放在 cellConfigAtConfigure 中，由于这个枚举的过程是按照字典序来进行的，所以在初始化的时候无法控制 “先设置 rowCount 和 columnCount 的值，再设置属性的值” 这样的顺序（除非你的属性的 KeyPath 按照字典序排列，就像 Test Case 那样），这样会导致这个控件的显示不正常。而我们又没有办法将这些配置填写在 `cellConfig` 中，因为这样会在 `willDisplayCell` 中覆盖我们之前选中的状态（XLFormViewController 的 tableView 并没有使用 reuse 机制，所以不会自动 reuse cell 而发生配置被重置掉的问题）。

  因此，我们在 XLFormCheckboxRowDescriptor 的子类添加了一个设值词典，cellConfigAfterConfigure，这个方法继承并重写了 `- (void)configureCellAtCreationTime` ，首先调用了父类的同名方法（父类没有暴露这个接口，通过 Category 方法添加实现），然后枚举并设值，最后，如果 cell 实现了 `- (void)prefix_configure` 方法，则会调用此方法。

  至此，我们细分了 Cell 的初始化阶段，将原来只有一个的初始化阶段分成了两个。应对我们自定义的 XLFormBaseCell 的业务需求。

2. SelectorPush 时设置 ViewController 属性方法

  对于某些 Row，我们可以通过声明其 rowType 为 XLFormRowDescriptorTypeSelectorPush 来使点击时跳转到某个我们的 View Controller。官方的 Demo 里是使用了一个 MapViewController 来介绍这个问题。Form 和跳转的 View Controller 之间通过 rowDescriptor 来进行数据交换。

  但是有时候我们希望能够对 Push 过去的 View Controller 设置一些属性的值，比如某些提示信息（NSString），那么有两种方法可以解决这个问题，一种是通过面向对象的继承关系去做，继承的子 View Controller 中的初始化方法里设置需要的这些属性的值。另外一种方法可以参考 XLForm 对 Cell 的初始化过程，在 XLFormRowDescriptor 添加一个 viewControllerConfig 字典并在 Push 的过程中枚举它并用 KVC 方式设值。

  显然第二种方式更好一些，这样更灵活。但是唯一的缺点是你需要修改的 XLForm 代码，因为你没有办法通过 Method Swizzling 来达到同样的效果。所以这里我推荐使用 Developments Pods 方式引入这个库。

以上两个自定义的内容就是目前我在项目使用到的。官方的框架本身已经非常的通用，能够满足大部分项目的开发需求。之所以分享出来的原因，是因为这两个问题在开发的需求中还是比较普遍的。但是对于不同的项目需求，自定义的内容也不一样。如果我上面说的这两点没有解决你的需求的话，可以查看一下官方 GitHub 上的 Issue 和 PR，有可能其他人的自定义能够帮到你。