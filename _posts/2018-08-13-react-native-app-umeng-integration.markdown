---
layout: post
title: React Native App 集成友盟页面统计
date: '2018-08-13 15:52:00'
permalink: "/react-native-app-umeng-integration/"
tags:
- react-native
---

在原生开发中，集成友盟之后，它的 SDK 会自动帮助我们统计页面的流转情况，iOS 中它收集的是 `ViewController` 的类名，同样的，安卓中它收集的是 `Activity` 的名称。 

> 页面统计集成正确，才能够获取正确的页面访问路径、访问深度（PV）的数据。

以 iOS 为例，那么 React Native 应用在某种程度上是单页应用，全局只有一个 `ViewController`，安卓上也是只有一个 `Activity`。那么自动的页面统计就失去了效果，幸亏友盟的 SDK 暴露了手动记录页面统计的 API，让这个统计功能重新可用。看了网上很多的所谓 React Native 下的教程，对这部分只是浅尝辄止，或者是根本都不提，所以我想我有必要分享一下我的解决经验。

最初能想到的有两种解决方法。第一种是按照 React 组件的 [生命周期](https://5a046bf5a6188f4b8fa4938a--reactjs.netlify.com/docs/react-component.html)，类似于 `- (void)viewWillAppear:(BOOL)animated;` 和 `- (void)viewWillDisappear:(BOOL)animated;`，在 `componentWillMount()` 和 `componentWillUnmount()` 中加入统计的代码，但这要求所有的组件 **均继承自这个 BaseComponent**，且子组件在覆写这两个方法是要保证调用 super 方法，因此对现有的代码改动过大，也不符合 React 的 [Composition over Inheritance](https://en.wikipedia.org/wiki/Composition_over_inheritance) 原则，所以产生了第二种的解决方法。

假设你的 App 现在已经在使用 `redux`、`react-native-router-flux` 和 `redux-sagas` 这一套主流的框架了。根据观察，`react-native-router-flux` 底层的 `react-navigation` 在页面流转时，会先 dispatch 一个代表当前页面 disappear 的 action，再 dispatch 一个代表新页面 appear 的 action。这个反映到 `react-native-router-flux` 就是 `ActionConst.BLUR` 和 `ActionConst.FOCUS` 的两个 action。咨询过友盟的客服，这个流程和友盟要求的页面统计方法是一致的（先退出前一个页面，再进入新页面），所以此方案可行。

首先在声明式路由的实现文件中，将 `react-native-router-flux` 的路由中产生的所有 action 都 dispatch 到 redux 的 store 中，大致的代码如下：

```javascript
reducerCreate = (params: any) => {
  const defaultReducer = new Reducer(params);
  return (state: Object, action: Object) => {
    typeof this.props.dispatch === 'function' && this.props.dispatch(action);
    return defaultReducer(state, action);
  };
};
```

这样当底层的 `react-navigation` 进行页面的流转时，不同的 action 会被正确传到 redux 的 store 中，因为没有任何一个匹配的 reducer，所以不会对 redux 的 store 造成任何影响。

下一步就是监听这些 actions 了。我们创建两个 generator function，把它放到 `redux-sagas` 中，大致的代码如下：

<script src="https://gist.github.com/EvianZhow/1eba043dd45ed61a9988095ddac10498.js"></script>

这样当页面的流转时，友盟的 SDK 已经可以正常记录了。

最后有一个细节问题是，这个方法会导致第一个页面记录的失常。比如你在声明式路由的实现中指定了某个组件是 `initial` 的，这个组件会成为 App 打开进入的第一个页面，但进入这个页面的事件并不会被触发，所以需要手动补充。我是在 App 根组件 的 `componentDidMount()` 中手动补上了。

至此，友盟应该可以以声明式路由中各 Component 的 `key` 值来记录各个页面的流转信息。

如图所示，最终效果和原生开发的自动收集相差无几了：

![Screen-Shot-2018-08-13-at-11.42.35-PM](/assets/img/archived/Screen-Shot-2018-08-13-at-11.42.35-PM.png)