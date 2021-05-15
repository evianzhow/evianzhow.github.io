---
layout: post
title: 使用项目模板创建 React Native App
date: '2018-05-16 09:33:51'
permalink: "/build-react-native-apps-with-template/"
tags:
- react-native
---

**阅读前，请注意版本与时效性！**

部分内容翻译自 https://medium.com/dailyjs/the-1-2-3s-of-react-native-templates-1f5dda037e11

### 什么是项目模板

简单来说，项目模板是一个包含你自己基础 Codebase 的脚手架（熟悉 RoR 的同学应该不陌生）。一旦项目模板创建后，你便可以在不同项目间中使用它，达到复用目录结构、快速构建项目的目的。有可能你会说——“哇，我从来没有使用过，这是一个多么新鲜的功能啊！”，但是事实是——每一次你利用 `react-native init` 创建应用的时候，`react-native-cli` 都会从 `HelloWorld` 这个预设的模板中创建应用，而非从头（Build from Scratch）构建起。也就是说

`react-native init AwesomeProject`

等于是

`react-native init AwesomeProject --template HelloWorld`。

React Native 预设有两个模板，一个是 `HelloWorld`，一个是 `HelloNavigation`，你可以在 [这里](https://github.com/facebook/react-native/tree/master/local-cli/templates) 看到。

### 创建项目模板的步骤

1. 创建一个符合命名规范的 npm 项目
2. 放入自己的 Codebase，整理目录结构，修改部分配置文件名称
3. Publish & Use

听起来挺简单的？那我们开始一步步操作吧！

1. 创建 npm 项目

因为目前脚本的一些 [限制](https://github.com/facebook/react-native/blob/48dccf18b884f3165c4b179383e4965b0a58d2c7/local-cli/generator/templates.js#L151)，对于 npm registry 上的 template，仅支持前缀为 `react-native-template-` 的命名，所以你也要遵循相同的命名法则。

创建好文件夹，在终端中键入 `npm init`，填入相应的信息，完成 npm 项目的初始配置。

2. 放入自己的 Codebase，整理目录结构

我自己的 Codebase 目录结构在 [f8app](https://github.com/fbsamples/f8app) 的基础之上进行了修改，仅供参考：

<script src="https://gist.github.com/EvianZhow/ebd999430ad88a7f650d8e26ddc7c7b5.js"></script>

需要注意的是，`HelloWorld` 在这里是一个全文的关键词，将会被通过 `react-native init` 传入的项目名称所替换，小写的 `HelloWorld` 也会被替换成对应名称的 **小写形态**。如果你要在新的 App 中使用到新的项目名称，这里填入 `HelloWorld` 即可达到自动修改的目的。

接着，将 dependencies 和 devDependencies 分别创建两个文件 `dependencies.json` 和 `devDependencies.json`，放入期望引入的依赖和开发依赖，例如：

```
{
    "react-navigation": "^1.0.0-beta.15"
}
```

> React Native 0.54.4 版本之后，支持从项目模板中引入 devDependencies 的功能，具体的讨论请参考 https://github.com/facebook/react-native/pull/18164#issuecomment-377866390

如果你需要自定义 `.babelrc` 或者是 `.flowconfig`，将前缀的 `.` 替换成 `_` 放入项目模板中，[目前](https://github.com/facebook/react-native/blob/48dccf18b884f3165c4b179383e4965b0a58d2c7/local-cli/generator/copyProjectTemplateAndReplace.js#L127) 支持

- `_gitattributes`
- `_babelrc`
- `_flowconfig`
- `_buckconfig`
- `_watchmanconfig`

的文件名替换，因为是递归查找替换，所以非根目录的这些文件的名称也会被替换。

3. 将此 npm 包上传至公有或者私有的 npm registry

### 使用项目模板

在前文中提到的，要是用模板创建 React Native 应用，仅需在 `react-native init` 时指定 `--template` ，这里的参数可以是某个 npm 包名、`file://`、`http://`、`https://` 或者 `git://`，如果你使用的是 npm 包名的这种形式话，此处切记 **不要输入指定的前缀**，否则会出错，提示找不到该 npm 包，因为此过程会自动加上前缀。

```
// 你的 npm 包命名应该形如 react-native-template-{template-name}
react-native init AwesomeProject --template template-name
```

在此过程中，如果有原生的组件需要 `react-native link` 的操作，也会一并执行。

这里原文作者给出了一个 React Native 的项目模板：

https://github.com/geirman/react-native-template-geirman

### Fin.

由于这个功能必须通过 `react-native-cli` 来使用，这就需要开发者有 Xcode 或者是 Android Studio 的开发环境。如果 `create-react-native-app` 能够在未来加上这个功能的话那就更棒了！

通过项目的模板，一个公司内部的 React Native 应用可以构建于同一个 Codebase 之上，显著降低了配置花费的时间成本。

### References:

- [CLI: 支持 templates 命令](https://github.com/facebook/react-native/commit/3a6dff4f4fe3d910c15bed7fa625681865f79f3a)
- https://medium.com/dailyjs/the-1-2-3s-of-react-native-templates-1f5dda037e11