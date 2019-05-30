---
title: "Webgl Part1"
date: 2015-10-02T22:42:48+09:00
lastmod: 2015-10-02T22:42:48+09:00
draft: false
keywords: []
description: ""
tags: []
categories: []
author: ""

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: false
toc: false
autoCollapseToc: false
postMetaInFooter: false
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: false
mathjax: false
mathjaxEnableSingleDollar: false
mathjaxEnableAutoNumber: false

# You unlisted posts you might want not want the header or footer to show
hideHeaderAndFooter: false

# You can enable or disable out-of-date content warning for individual post.
# Comment this out to use the global config.
#enableOutdatedInfoWarning: false

flowchartDiagrams:
  enable: false
  options: ""

sequenceDiagrams: 
  enable: false
  options: ""

---

谈一谈WebGL这个看上去很时髦但始终没有进入主流应用的技术。

### 为什么现在是时候考虑应用WebGL了

在WebGL的普及上最大的一个问题是开发者对浏览器支持的担忧。需要特别说明的是如果仍然需要用户手动的从某个菜单中选择“启用WebGL”那就不能算真正的“支持”。毕竟没有人有信心多少用户会愿意做这个举动并且成功找到这个菜单而不是直接认为这个网页很可疑从而离脱。

<!--more-->

近年来主流桌面浏览器（Chrome，Firefox，Safari，IE11＋）基本已经默认启用WebGL，但是在移动端越发重要的今天，如果移动设备的浏览器上不能做到完美支持，那愿意付出这个代价开发WebGL应用的开发者或公司可以说只能停留在小打小闹而非实际（商业）项目。

幸而，Android的Chrome从version 30开始了对WebGL的支持，从而为移动端打开了半壁江山（关于Android设备中另一大类所谓的stock browser稍后详细说明），但是剩下的iPhone用户仍然是开发者无论如何无法忽视的。

直到iOS8的正式发布。

从iOS8搭载的Safari开始，WebGL功能终于被默认打开，从此用户不需要任何手动的操作就能体验到WebGL应用带来的改变。桌面，Android，iOS，三大平台终于普遍支持了WebGL，那么开发者们还在等什么呢？

![http://caniuse.com/#feat=webgl](/images/webgl/caniuse_webgl.png)

浏览器支持的障碍基本扫除之后，剩下就是为什么要使用WebGL了。

在这个人人大谈HTML5的时代，提升Web应用的用户体验使之不断逼近Native应用是很多Web开发者的目标，那么WebGL就是图像性能上的一个大杀器。原因下文具体阐述。

除此之外，当然了，如果想在你的Web应用中使用3D图像，那更没理由不用WebGL了。

但除了那些“明显”的3D应用外，在一些意想不到的2D页面中适当的加入一些WebGL的渲染效果也是近年来一个创新的趋势。例如近两年Apple就经常在产品主页中玩这些小花样，今年的[iPhone6s主页](http://www.apple.com/iphone-6s/)就会让用户在滚动页面的同时看到由WebGL渲染出的随之变化的反射效果。从页面源码就能看出：

![Part of source of http://www.apple.com/iphone-6s/](/images/webgl/iphone6s_source.png)

### 问题与现状

上一节说了关于WebGL的各种乐观情况，下面来泼一些冷水。

首先是在移动端的有着若干待解决问题：

##### 难以统一的具体实现

到目前为止在这篇blog中以及其他很多地方提到对于WebGL的支持时，通常只会有支持和不支持这两种简单的状态，但实际情况远非这么简单，WebGL在某个特定设备和浏览器中的具体实现包含的是一整套涉及软件和硬件的参数与标准，一百台同样号称支持WebGL的设备上理论上都有可能找不出两台完全一样的WebGL具体实现。想知道自己浏览器上的实现细节的话可以访问一下这个网站：http://webglreport.com/

从GLSL的各项参数，到所支持的extension集合，细节的复杂程度可见一斑。这种复杂性主要是由于WebGL本身就是一套为了充分利用硬件而抽象并约定而成的API，因此与硬件及驱动的耦合和由之带来的限制可以说是难以避免。在其之上再叠加上浏览器开发方的具体调用和实现（当然还有bug），就形成了这样一个难以统一实现的局面。

以往前端开发中最头疼的浏览器兼容问题，现在再加上针对不同硬件（主要是GPU）和驱动的兼容，看上去有些让人绝望。但也没有必要过分悲观，前提是不要过分依赖特定设备和浏览器的实现，而是尽量取自己的主要对象用户群体中绝大多数实现的交集进行兼容就容易许多。比如不要设想Texture容量必须拥有最新旗舰机的超大标准，或者只有个别GPU和驱动才支持的某项extension。最后，在应用中提前提取用户所用WebGL实现的各项相关参数并据此作出调整就可以了。

##### 移动端条件尚不十分成熟

目前主要的问题有两点：

* WebGL渲染所需的mesh数据体积庞大，可能会使应用的加载时间过长，还有耗费用户过高的流量
* 与PC相比仍有巨大差距的CPU/GPU性能，特别是运行大型3D应用（如游戏）恐怕依然有些吃力

这也就是为什么目前我们能看到的WebGL应用大致只有两类，一类是试验性的Demo应用或游戏(一般每隔一两个月就会在HN上出现一次:P)，这类应用通常不用考虑盈利问题所以也无所谓移动端体验和各类设备及浏览器的兼容性；另一类就是例如Apple在其产品页面上做的小效果，不会给整体用户体验带来负担，当然一个副作用也是很多用户可能根本难以察觉某处的“一番苦心”。

这样看，在大型3D应用中使用WebGL，尤其是考虑移动端表现的情况下，目前的软硬件条件还并非十分成熟。但是另一方面，在2D应用中还是大有可为的，因为2D应用不需要大量的mesh数据，尤其是矢量图像的话数据流量大大减少，对GPU的负荷也会随着mesh数量的锐减而相对减轻，即使在相对较老的移动终端上也有机会跑出不掉帧的图像表现。

##### 开发成本问题

首先，对大部分没有3D应用开发经验的前端工程师来说，掌握GL系的pipeline和API需要一定的学习成本。另外，3D模型的制作和开发也是另一项较大的开发成本。因此，对于传统的主要针对2D应用或网页的项目来说，如果能够活用已有的2D资产和人力来开发具有更高性能表现的WebGL应用是最理想不过的了。


读到这里可能很多人会产生疑问，用WebGL开发2D应用，可以吗？适合吗？WebGL是这样用的吗？而且2D应用的技术已经有很多并且很成熟了，有用WebGL的必要吗？

近年来关于WebGL的入门和介绍文章有很多，但是由于WebGL本身的一些特性，导致初学者如果只是阅读这类入门介绍容易对WebGL及其应用场景产生一定的误解，其中最常见的误解我认为有以下三点：

* 只有3D图像或应用才需要用WebGL
* 用了WebGL，就能为应用的性能自然而然的带来显著提高
* WebGL是又一个由浏览器封装并提供的高层抽象特性

在这个系列的后续文章里会主要针对以上三个方面作重点说明，帮助初学者重新认识一下WebGL。

*这个系列是结合我在GREE公司主办的GREE Tech Talk #8和Google主办的Chrome Tech Talk Night #8上做的发表内容扩展而写的，当时的发表材料请参考下面的链接:*

* *GREE Tech Talk #8: [event主页](http://techtalk.labs.gree.jp/08/),[发表材料](http://www.slideshare.net/GuangyaoLiu/you-dont-know-webgl),[视频](https://youtu.be/rF1NgSxg-9o) （由于是在日本做的发表，日语见谅）*
* *Chrome Tech Talk Night #8: [event主页](http://googledevjp.blogspot.jp/2015/07/chrome-tech-talk-night-8.html),[发表材料](http://www.slideshare.net/greetech/beating-canvas-2-d-in-its-own-territory-webgltesspathy)*