# ANDROID 热补丁实践之路

版权声明：本文为 stormzhang 原创文章，可以随意转载，但必须在明确位置注明出处！！！

大约在15年下半年开始，热补丁方案开始大量涌现，一时间热补丁修复技术在 Android 圈非常火爆，比较有代表性的开源实现有 Dexposed、AndFix、Nuwa 以及前段时间微信开源的 Tinker，至于他们的原理以及优缺点比较并不是本文要讲的，网上已经有一大堆资料进行介绍了，感兴趣的可以看下这几篇文章：


  [安卓App热补丁动态修复技术介绍](https://mp.weixin.qq.com/s?__biz=MzI1MTA1MzM2Nw==&mid=400118620&idx=1&sn=b4fdd5055731290eef12ad0d17f39d4a)

  [Android热补丁之AndFix原理解析](http://w4lle.github.io/2016/03/03/Android%E7%83%AD%E8%A1%A5%E4%B8%81%E4%B9%8BAndFix%E5%8E%9F%E7%90%86%E8%A7%A3%E6%9E%90/)

  [Instant Run工作原理及用法中文翻译稿](http://www.jianshu.com/p/2e23ba9ff14b)

  [从Instant run谈Android替换Application和动态加载机制](http://w4lle.github.io/2016/05/02/%E4%BB%8EInstant%20run%E8%B0%88Android%E6%9B%BF%E6%8D%A2Application%E5%92%8C%E5%8A%A8%E6%80%81%E5%8A%A0%E8%BD%BD%E6%9C%BA%E5%88%B6/)

  [各大热补丁方案分析和比较](http://blog.zhaiyifan.cn/2015/11/20/HotPatchCompare/)


我一直认为对于客户端开发来说热补丁修复技术不是必须的，但是却很有必要的，我们身为开发者虽然都不想自己的程序出 bug，但是没有一个开发人员能保证自己的程序一定不出 bug 的，之前出了问题只能重新发布版本，然后用户下载更新，这个代价还是蛮大的，但是有了热更新技术，这个就变得很简单了。

所以在半年前我们也评估了以上几种热修复框架，准备用在项目中。

首先考虑的就是阿里开源的 Dexposed 和 AndFix 两个框架，前者是手淘开源的，后者是支付宝团队开源的，都是 native hook 的方案，但是 Dexposed 不支持 art，在未来这是个很大的隐患，所以直接抛弃我们选择了 AndFix。

评估下来，我们觉得 AndFix 虽说有一些限制，比如并不支持类替换、资源文件替换和 so 替换等，但是毕竟支持全平台，唯一担心的是 AndFix 基于 native hook 的方案，不是属于 java 层，属于 jni 层，在国内这么复杂的大环境下，稳定性与兼容性是个很大的考验，不过想到毕竟是支付宝团队出品，应该不用过渡担心。

于是我们开始着手在项目中集成 AndFix，过程还算顺利，实际测试下来效果也蛮好的，直到真的一次线上版本出现了 bug，考验 AndFix 的时候到了，像集成的时候一样，QA发布补丁，测试ok然后发布到正式环境。但是接下来并没有像我们想象的一样，错误率并没有下降，而且从后台错误统计看到反而产生了新的莫名其妙的bug，所以我们就觉得兼容性有问题了，赶紧修复紧急发布了新的版本。

这次经验证明了，native hook 的方案兼容性确实有很大问题，而且 AndFix 框架本身也有坑，从 GitHub 上该项目的 Issues 数量也可以看到，目前仍有近 200 个 issue 没有解决。

我们中途考虑采用 Nuwa，毕竟 multidex 方案是属于 java 层面的，兼容性肯定没有问题，但是评估之后觉得 Nuwa 会带来性能问题，而且这个时候微信的热补丁方案 Tinker 已经放出来，并且表示即将开源，所以我们考虑等 Tinker 开源了再说。

今年的9月24号，MDCC 大会上腾讯的 Tinker 终于开源了，我们在第一时间进行了评估。

总体看下来，虽说 Tinker 也有一些限制，但是综合下来优势很明显，下图是微信官方曝出的一张各大热修复方案对比图，看下来很直观：
![为什么使用tinker](https://raw.githubusercontent.com/shuiyes/gitblogs.github.io/master/attachments/ANDROID%20%E7%83%AD%E8%A1%A5%E4%B8%81%E5%AE%9E%E8%B7%B5%E4%B9%8B%E8%B7%AF/tinker.png)

除了技术上有优势之外，还有就是微信覆盖的人群太广了，全国几亿人，各种设备都有，在兼容性方面微信肯定是首选考虑的，所以兼容性方面评估下来应该不是问题。

于是安排团队成员果断把 AndFix 替换成 Tinker，主要是我们项目依赖了 resGuard 来进行资源混淆，所以集成过程中稍微有点小麻烦，但总体来说还算比较顺利，毕竟 Tinker 官方文档很齐全，而且我认识 Tinker 作者，有问题甚至都可以直接进行请教。

就在前几天我们发布了新版，其中出了一个 bug，又到了检验 Tinker 效果的时候了，这一次 Tinker 没有让我们失望，补丁发布之后出错率降的很明显，实践证明 Tinker 在兼容性方面完全没问题。为了更有说服力，上一张真实的友盟出错率：
![友盟出错率](https://raw.githubusercontent.com/shuiyes/gitblogs.github.io/master/attachments/ANDROID%20%E7%83%AD%E8%A1%A5%E4%B8%81%E5%AE%9E%E8%B7%B5%E4%B9%8B%E8%B7%AF/um_error.png)

现在的技术与资料越来越多，只看网上的理论永远没有任何说服力，只有亲自实践才能是最好的说服力。之前很多人都问过我，说热补丁修复框架到底哪一个好，我都没有回答，那是因为我们还没有亲自实践，不能只单纯的从理论来进行分析，结果很重要。而如今，如果你想把热补丁框架应用到你们项目中的话，那么我推荐把 Tinker 作为最优选择，起码现阶段来说是最优选择。
