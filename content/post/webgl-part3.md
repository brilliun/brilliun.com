---
title: "WebGL Part3"
date: 2015-10-05T22:40:37+09:00
lastmod: 2015-10-05T22:40:37+09:00
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

WebGL === 性能提升 ?

提到WebGL大家最先想到的除了3D应用的渲染之外，另一个主要的共识就是使用WebGL会给图像性能带来颇大的提升，但是这个逻辑是怎么来的呢？而实际应用中是否真的只要切换成WebGL就能自然而然的立马得到想要的性能提升呢？

<!--more-->

首先，大家普遍存在这样的印象是因为在关于WebGL的优点介绍中几乎都少不了这样一个关键词：硬件加速（hardware acceleration）。所以顾名思义，WebGL既然使用了硬件加速，那必然是加速了，在硬件能力的辅助下，性能必然是提高了。

但这里我想说的是，硬件加速并不能**保证**图像性能的提升。就拿[上一篇文章](/2015/10/webgl-part2/)中最后所介绍的那个[Demo](http://gree.github.io/tesspathy/demos/canvas_vs_webgl/)来说吧，WebGL版的性能表现要大大超过Canvas 2D版，这其中的差别就是因为WebGL使用了硬件加速吗？如果你的浏览器是Chrome，不妨在地址栏中输入「chrome://gpu/」看一下

![chrome://gpu](/images/webgl/gpu.png)

可以发现在现代的浏览器中，几乎在所有可能的渲染方式中都尽可能的使用了硬件加速，也就是说，在Demo中体现出的巨大性能差异，是在Canvas 2D和WebGL同时使用了硬件加速的情况下依然存在的。

这里不得不提一下硬件加速的实质，其实就是当我们意识到普通的CPU并不擅长处理一些特殊应用场景（例如渲染一个几百万像素的图像）的时候，选择其他更适合的计算机硬件（主要是GPU）来代替CPU更高效的完成工作，因为GPU相比于CPU的优势在于大量的并发和相对简单直接的pipeline，因此更适合处理大量可以同时进行且互不影响的密集运算，例如为每个像素点计算它应有的RGB值。与此同时，CPU可以腾出手来完成其他复杂的逻辑计算而不会被block住。硬件加速的最后一个优点在于，现代浏览器例如Chrome都已经使用GPU来完成页面上各部分的最终整合，即各个部分都会被渲染在各自的Texture上直到要渲染整个页面时就可以直接将这些现成的Texture拼贴在一起就可以了。而如果像HTML Canvas这类元素本身就已经使用GPU加速进行渲染，最后整合页面的时候就可以直接将已经存在于GPU上的渲染结果拿来拼贴而省去了生成Texture并上传的工序。

硬件加速具有如此优秀的特点，但是在与其pipeline进行交互的时候需要注意的方面也更多。还是来看刚才的那个Demo，这一次我们深入探究一下同样使用了硬件加速的Canvas 2D和WebGL究竟是什么导致了二者性能的差异。

这里用到的工具是Chrome的一项「隐藏功能」，其实只是并未大力宣传和尚未得到广泛应用，这也是理所当然的因为这个工具仍然在积极开发中。打开方式就是在地址栏中输入「chrome://tracing」，然后点击左上角的record按钮就可以选择需要捕捉的浏览器底层信息。这次我们主要比较渲染性能的差异，因此我选择了下面这些项目：

![Tracing record settings](/images/webgl/tracing_config.png)

而用来进行对比的demo就使用Tesspathy主页上的这个[Demo](http://gree.github.io/tesspathy/demos/canvas_vs_webgl/)。
首先来看canvas 2D：

![Canvas 2D](/images/webgl/canvas2d.png)

添加到500个星星，可以看到画面已经略有卡顿。这个时候切换到刚才的chrome:tracing的tab，按下Record开始记录，然后马上切换回demo的tab，约2，3秒后就可以切换回来结束记录。随后就会生成下面这样的profiling结果：

![Canvas 2D Record](/images/webgl/canvas2d_frame.png)

这个图中的圆圈部分就是实际浏览器渲染的一帧，此外在这里收集的大量数据中因为我们想要比较的是同样使用了硬件加速的Canvas2D和WebGL到底存在什么区别，因此我们只关心GPU相关数据，并将范围定在相邻两帧之间，选取出的数据如下：

![Canvas 2D Record](/images/webgl/canvas2d_cmd.png)

可以看到有以下几个项目耗时较长，分别是：
* kBindTexture (GPU Texture切换操作)
* kTexSubImage2D (GPU Texture的修改／更新并上传)
* kDrawElements (实际的绘图命令)

最终结果就是每一帧的总耗时超过了26毫秒，而一般如果要播放60fps的动画，留给每一帧的时间只有大约16毫秒，也就是说在这次的demo中使用canvas2D已经不可能实现了。
再仔细分析一下这些数据，还可以揭示出的Canvas2D的GPU加速的一些细节：
* 首先，每一帧发生的draw call大约是500多一点，这大致就是我们在这个动画中所加入的星星的数量，也就是说每一次Canvas2D API中context.fill()的调用都会反映为一次GPU上的draw call。
* 其次，在短短一帧的过程中，发生了如此多次的Texture的修改和切换，而作为开发者我完全没有这类意图但却被迫地承受了由之带来的大量时间损失，显然浏览器为了考虑出我以外的其他各类应用都能在平均水平上输出足够优秀的图像性能而在背后进行了某些优化或者说准备，但显然在我这个应用上这个优化是失败的。

接下来看WebGL版的表现：

![WebGL](/images/webgl/webgl.png)

同样添加到500个星星，可以看到画面依然十分流畅。同样的方式我们也来记录一下用WebGL绘图时浏览器的背后究竟发生了哪些GPU相关的操作：

![WebGL Record](/images/webgl/webgl_cmd.png)

相比Canvas 2D时繁忙拥挤的时间轴，WebGL版的时间轴就空旷了很多，具体数据提取出来也确实如此，比较明显的耗时项目只有下面这一个：
* kDrawElements

而且，这次的次数只有区区105次，一帧内的总耗时也只有2.43毫秒，这样，即使是60fps的动画或者游戏，渲染部分也不会占据过大的时间。至于为什么同样是绘制500个对象同时移动的动画我的WebGL版只需要100次左右的draw call，原因很简单，就是因为我是这个动画或者应用的开发者，我知道这500个星星必然是一起出现在同一个场景中，因此我选择将他们每若干个一组用一个draw call来画，而非一个一个画。由于可以使用WebGL，具体的渲染命令几乎完全在我开发者的掌控下，所以可以根据我自己这个应用的特点选择最适合的实现方式。反观Canvas2D那边，因为实际底层渲染部分的任务完全交给浏览器来全权负责了，所以浏览器既需要照顾到所有用户的性能，实际上又不知道每一个用户的应用到底有什么特点，因此只能做出相对保守的选择。比如，由于不知道这500个星星会不会在某一秒后开始需要分奇偶序数的分开出现，浏览器不敢贸贸然将他们的几何数据以连续的方式存储起来并一起绘制，而是只能逐个分离以保证今后可以应对各种需要的灵活性。

## 结论

同样使用了硬件加速的WebGL和Canvas 2D在性能表现上依然可以出现如此大的差异，归根结底硬件加速本身并不是“银弹”，WebGL当然也不是。要想获得更突出的图像性能表现，对现代GPU渲染的整个pipeline中各个环节的理解和优化才是根本之道，才能将WebGL赋予我们的能力发挥到最大。最后给出一些我自己在实践中总结的WebGL性能小贴士作为这一篇的结尾：

* 善用Texture
    * 注意不同设备所支持的最大Texture尺寸可能不同
    * 根据自己应用的需求来设定MAG_FILTER, MIN_FILTER, MIPMAP
    * 频繁切换绑定的Texture也是不小的开销
    * 更新Texture更是巨大的开销，一定要尽可能减少频率，选择好时机，只更新发生改变的区域
* 善用ArrayBuffer来缓存原始几何数据
* 精确控制draw call，因为这几乎是影响图像性能的最重要参数
    * 尽可能将一起出现的部分在一次draw call中绘制
* 当性能依旧不如意的时候，尤其是画面中不透明的重叠部分较多时，可以考虑从前往后的绘制顺序并利用GPU的z-culling功能减少无用的像素处理
* 避免在shader中，尤其是FragmentShader中编写大量的逻辑分支，GPU可以处理但是其pipeline不擅长。
