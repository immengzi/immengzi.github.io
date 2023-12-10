---
title: WPF 开发体会
date: 2023-12-11 02:29:46 +0800
categories: [C#]
tags: [WPF,C#,.NET,DI,IoC,UI,ML.NET,MVVM,Git]
math: true
---

> 仍在完善中。

## 0 引言

最近集中时间精力准备《.net 架构程序设计》这门课程的大作业，我得以深入了解 WPF 相关内容，用到了不少高级特性，感觉收获不少。

下面的内容主要结合 WPF 和广泛的技术理念来展开。

## 1 WPF UI 库

我在开发时选择了 HandyControl，开源的，能用，但有点丑。

下次可以考虑换 WPFUI，也是开源的，高仿 Win11 的毛玻璃风格，颜值还不错。

也会考虑使用一些付费的 UI 库。

引用知乎一位老哥的回答：

> 正经开发一堆界面和产品的话就推荐 devexpress，syncfusion 和 telerik，他们是 .net 全平台控件，基础控件全面，多套皮肤，office api，自定义报表，要啥有啥，这样就不需要买 aspose 这种这么贵的 office 库，也要比免费的库强大太多了，特别是支持 pdf，内部功能也更齐，不需要再搞这个 excel 开源库，那个 word 开源库，在搞个 itextsharp 来搞 pdf，所谓的开源库还有开源协议限制，有些作者的脾气你们也懂的，很多还开始转商业了，用起来并不能放心，而这些控件套装就都包含 office api 了，也不需要单独购买报表了，单独的报表也没有比他们强多少，平台也不一定有他们支持得多
>
> ...
>
> 我就很喜欢大套装，各个平台都有，既然买了直接拿来用，开发一点多平台的东西很方便，wpf 开发一套，web 开发一套，手机开发一套，很方便齐活儿
>
> 原文链接：https://www.zhihu.com/question/311160143/answer/1828970146

其中提到“搞这个 excel 开源库，那个 word 开源库，在搞个 itextsharp 来搞 pdf”，太真实了！我个人对很多事情都是倾向于一劳永逸少折腾，~~和我不太喜欢 VSCode 一个道理~~。

## 2 WPF TitleBar

> 那些是“非客户端”区域，由 Windows 控制。可以将窗口的 WindowStyle 设置为"None"，然后构建自己的窗口界面。
>
> [How can I style the border and title bar of a window in WPF? - Stack Overflow](https://stackoverflow.com/questions/9978444/how-can-i-style-the-border-and-title-bar-of-a-window-in-wpf)

部分 UI 库支持更改 TitleBar 的前景色/背景色等内容，如 HandyControl 不支持，但刚才发现 WPFUI 支持，我在它的仓库找到了自定义 TitleBar 组件的证据：[wpfui/src/Wpf.Ui/Controls/TitleBar/TitleBar.xaml](https://github.com/lepoco/wpfui/blob/538af8bc8acda730935de7c45ad6e052e960a6af/src/Wpf.Ui/Controls/TitleBar/TitleBar.xaml)，可惜我已经用 HandyControl 糊完项目了欸。

## 3 ML.NET

Rubia 负责这一块，她自己整理的数据集，训练出来准确率 17%，拿网上在 python 平台上得出 80% 准确率的数据集过来跑，只剩下 9% 了。我人都傻了，不禁怀疑 ML.NET 设计理念是是什么，有什么用。

根据网上搜集到的讯息，我大胆做一个总结：

1. 使用模型：ML.NET 使 .NET 工程师以原生的方式直接调用训练好的模型
2. 训练模型：ML.NET 使即使对机器学习毫无了解的人也能够快速地构建自己的模型

## 4 DI/IoC

写代码的时候，有一个需求，即在 HomePage 中更改 MainWindow 中控件 MediaPlayer 的 Source 属性。注：HomePage 是我自定义的 UserControl。

~~第一秒：全局共享！~~但随便一想就觉得不行——global 满天飞绝对不是啥好事，再说，我已过了能跑就行的学习阶段了，该有点追求了吧~

搜索了一通，知道了 DI（依赖注入）/IoC（控制反转）。DI 是软件设计模式，IoC 是一种软件设计原则。

但我只是简单地使用了 DI，并没有使用 IoC 容器。IoC 的首要好处是能够集中配置依赖项，我只有一项要配，所以...

我注意到一些关于是否有必要使用 IoC 容器的有趣的讨论：[dependency injection - Why do I need an IoC container as opposed to straightforward DI code? ](https://stackoverflow.com/questions/871405/why-do-i-need-an-ioc-container-as-opposed-to-straightforward-di-code)

## 5 写好 Style 的简单提议

## 6 Git Commit 信息

下面是 CornWorld 在用的 git commit 范式，我觉得很规范，长期来看对项目有帮助，所以也沿用了。

[Workflow-Regulation/git_commit_message.md](https://github.com/HadTeam/Workflow-Regulation/blob/main/git_commit_message.md)

## 7 对 MVVM 架构的理解

### 7.1 Repository 模式

### 7.2 不要排斥 Code behind

在开发过程中，我有若干次犹豫不决。秉持“我在使用 MVVM 架构”的理念，我总是避免在视图的隐藏 C# 文件中编码。

直到现在，我大脑仍然未建立一种标准，去评判哪些情况下可以忽视 avoid code behind 原则。这可能需要更多的开发经验来习得了。

或许未来会有共鸣：[MVVM and Code Behind](https://gwerren.com/Blog/Posts/mvvmCodeBehind)