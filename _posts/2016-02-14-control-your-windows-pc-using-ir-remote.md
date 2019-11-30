---
layout: post
title: Control your Windows PC using IR-Remote
comments: true
---

自从买了小米盒子看电视就变得容易得多了, 但是很多资源盒子上还是没有 PC 上多, 而且盒子类的产品都是不支持 Flash 类型网站的视频的。所以很多情况下, 不得已还是要用 PC 那, 什么姿势看电视才最舒服呢？ 答案当然是 “躺着” 。

我在想，如果 PC 也可以像盒子类产品一样可以用遥控器操作，那该多好呀

于是我抄起手中的遥控器开始尝试对其解码, 我们知道现在家庭中常见的[遥控器](https://en.wikipedia.org/wiki/Remote_control)基本都是基于[「红外」](https://en.wikipedia.org/wiki/Infrared) .

我很早之前写过一篇[文章](/2014/05/lubuntu-on-cubietruck.html)说过, 我有一个 [Cubietruck](http://cubieboard.org/tag/cubietruck/) 它是基于 ARM 架构的 miniPC， Master board 上自带了 IR-Receiver 功能

为了实现这个功能，我另外写了两篇文章介绍相关细节

+ [Linux Low-level Input Event Reading](/2016/02/linux-low-level-input-event-reading.html)
+ [Remotely Turn On Your PC Over the Internet](/2016/02/remotely-turn-on-your-pc-over-the-internet.html)

完整代码如下：

<script src="https://gist.github.com/song940/ace194795bee627b5a61.js"></script>
