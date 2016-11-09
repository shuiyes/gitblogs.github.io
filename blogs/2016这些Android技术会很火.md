# 2016这些Android技术会很火

在Android开发中，新技术不断涌现。对于GitHub上如此众多的项目，有人不断Mark，有人分享自己的经验，不管怎么样，如果能让你真的有所学习有所收获，我们的目的也就达到了。

## 1、DataBinding  

今年的 Google IO 大会上，Android 团队发布了一个数据绑定框架（Data Binding Library）。Data Binding Library 是一个 support 库，支持 Android 2.1+ 版本 (API level 7+)。 
在2015年，它还是beta版本，但是就 Android Studio 2 的 Preview 版本发展来看，Google 在这个库上还是很花心思的，我们有理由相信，在2016年 DataBinding 将会迎来第一个正式版。

## 2、MVP模式

MVVM 与 MVP 模式，正在 Android 开发中越来越流行。在这里为大家强烈推荐我的：TheMVP 项目，可以直接引入项目作为 module 依赖。(详情请在 github 搜索 TheMVP )

## 3、热修复
在2015年，涌现出了一大批热修复动态加载技术：HotFix、Nuwa、DroidFix、AndFix 等等，以及同样原理的插件化技术：DroidPlugin、DynamicAPK。就连 Android  Studio 2 的 Preview 版本中体现的 Instant Run 功能，本质上也是一种热修复技术。
最近开源界涌现了很多热补丁项目，但从方案上来说，主要包括Dexposed、AndFix、ClassLoader（来源是原QZone，现淘宝的工程师陈钟，在15年年初就已经开始实现）三种。前两个都是阿里巴巴内部的不同团队做的（淘宝和支付宝），后者则来自腾讯的QQ空间团队。
2016年9月24号，MDCC 大会上腾讯的 Tinker 也开源了

## 4、RxJava
优雅(也许仅体现在lambda表达式)的链式表达，轻松的线程切换，让 RxJava 在 2015 年已然得以如日中天。如果此时你还不了解 RxJava 究竟是什么的话，我建议你一定要仔细反思一下自己是否已与世界脱轨。

## 5、RxVolley
RxVolley，让 Volley 支持了 RxJava 后，让你的代码很轻松的脱离了回调地狱。同时移除掉了复杂的 HttpClient ，以及可选支持 OkHttp 与 ImageLoader，让你使用自己习惯编码风格的同时极大缩减了项目体积。 

## 6、RxBus、RxBinding 
得益于 RxJava 繁多的操作符与特性，结合此类基于 RxJava 的库，将使你的代码更加简洁，开发效率大大提高。
RxBus，值得一提的是 RxBus 并不是一个库，而是一种设计思维，它可以巧妙利用 RxJava 的特性，完美替换掉了原事件总线类库(EventBus/Otto等)  
RxBinding，	RxJava 封装的 View 事件处理，事件的改变以流的形式进行传递。 

## 7、Kotlin 语言
作为 Android 阵营的 Swift ，在2015年也迎来了它的正式版。Kotlin 拥有很多 Java 所不具备的特性， 比如空指针安全，函数默认参数，默认包含模板类，对 lambda 的原生支持(在 Android 开发中, 常常使用 RxKotlin )等特性。
