---
title: "WebGL Part2"
date: 2015-10-03T22:41:51+09:00
lastmod: 2015-10-03T22:41:51+09:00
draft: false
keywords: []
description: ""
tags: ['WebGL', 'Tesspathy', 'javascript']
categories: ['Learning']

isCJKLanguage: false

comments: true
showTags: true
showPagination: true
showSocial: true
showDate: true
showMeta: true
showActions: true

---

WebGL === 3D ?

这是最常见的一个误解。MDN上的[WebGL入门教程](https://developer.mozilla.org/en-US/docs/Web/API/WebGL_API/Tutorial/Getting_started_with_WebGL)开篇第一句话就是：

> WebGL enables web content to use an API based on OpenGL ES 2.0 to perform 3D rendering in an HTML canvas in browsers that support it without the use of plug-ins.

<!--more-->

随意在网上搜索一下WebGL的介绍和入门，经常就会看到一切就这么从一个转个不停的3D立方体开始了，一个初学者的脑中就这样被植入了一个3D图像的印象。之后的一系列应用实例或者API介绍也都离不开这样一些主题：旋转，跳跃，再来一束强光造个影什么的。另一方面，那些能够登上HN或者reddit的夺人眼球的WebGL作品，几乎是清一色的3D游戏。最终导致了WebGL等同于3D的认识。

这样的认识也确实没有错，有了WebGL，在浏览器里渲染3D图像变得更方便了。但也只是更方便了。在正反两个方面都并非绝对排他的。
首先，没有WebGL，哪怕用Canvas 2D的API，只要你有这个闲工夫，撇开性能因素不谈，实现和WebGL的渲染效果一样的3D效果完全是可行的。
反过来，WebGL并不止能用来渲染3D图像，包括OpenGL在内的GL系API的实质其实也是为了在2D平面上渲染图像，既然3D空间中的对象可以通过API映射到这个2D平面上，那么2D空间中的对象更不成问题了不是么。

所以，抛开3D这张面具，WebGL也可以很“平易近人”。毕竟2D内容依然是现有站点的主流，如果能够在2D内容的渲染上方便的运用，WebGL的实用价值就大大提升了。唯一的问题是，这并不容易。

要说清楚为什么用WebGL来渲染2D图像很不容易，首先看下面这张图

![https://signalizenj.wordpress.com/2015/01/29/vector-vs-raster/](/images/webgl/vector_raster.gif)

所谓的2D图像一般可以分为两类，一类是以矢量为基本元素的vector graphics（矢量图），另一类是以像素为基本元素的raster graphics（位图）。

在WebGL出现之前，普遍的在网页中绘制2D图像的手段无非有CSS，SVG，Canvas 2D API这几种，其中又只有Canvas 2D API能够通过js做到实时的控制。使用Canvas 2D，无论是矢量图还是位图都很容易绘制，因为各自都有现成的专用API：
 
```javascript
var canvas = document.getElementById("myCanvas");
var ctx = canvas.getContext("2d");
ctx.beginPath();
ctx.moveTo(x1, y1);
ctx.lineTo(x2, y2);
ctx.quadraticCurveTo(x3, y3, x4, y4);
ctx.stroke();
ctx.fill();
```
用类似这样的一组API，把矢量的轮廓（直线或曲线）描一遍，最后fill一下，再复杂的图形都能一举填充完成。

```javascript
var canvas = document.getElementById("myCanvas");
var ctx = canvas.getContext("2d");
ctx.drawImage(
  img,
  srcX, srcY,
  srcWidth,
  srcHeight,
  x, y,
  width, 
  height
);
```
位图更简单了，四个顶点一确定，一个长方形的位图就能画出来了。

相比之下，WebGL的API基本就是OpenGL API的翻版，有相关知识的都知道，这套API从开始就不是为了画矢量图而设计的。WebGL中的绘图API（又称为draw call）只有下面两个
```javascript
var canvas = document.getElementById("myCanvas");
var gl = canvas.getContext("webgl");
gl.drawArrays(mode, first, count);
gl.drawElements(mode, count, type, offset);
```
而这里的```mode```有哪些选择呢：

* gl.POINTS
* gl.LINES
* gl.LINE_LOOP
* gl.LINE_STRIP
* gl.TRIANGLES
* gl.TRIANGLE_STRIP
* gl.TRIANGLE_FAN

总结起来就三种模式：画点，画直线，画三角形。

要用如此单调的API来绘制2D图像是怎么样一种体验呢，先看简单的位图。

![WebGL绘制位图](/images/webgl/raster_by_webgl.png)

位图的轮廓必然是长方形，因此如上图，方法就是把这个长方形沿对角线看成是由两个三角形拼接而成的，然后用WebGL中画三角形的API画出这两个三角形并为每个点指定它在原图中的坐标作为纹理坐标，最后上传这张位图作为铺设在这两个三角形上的纹理即可。

下面再来看矢量图：

![WebGL绘制矢量图(红点为矢量顶点](/images/webgl/vector_by_webgl.png)

这样一个不规则且包含曲线的轮廓，无论是单纯描边还是填充颜色，要怎样通过画点，画直线，或者画三角形的API来完成呢？

一个最直观的想法就是事先把所有要用的矢量图全部转换成位图，但矢量图所具有的多项优点也就随之丧失：

* 首先，矢量图本身没有分辨率（图像尺寸）的定义，因此可以自然的适应各种尺寸需求而不影响画质。
* 其次，因为不需要像位图那样描述每一个像素的颜色，矢量图的文件大小要远小于位图
* 最后，许多现有资源都会依赖矢量图，如swf文件等。

![矢量图的分辨率优势（This work is a derivative of a photo(https://commons.wikimedia.org/wiki/File%3AVectorBitmapExample.svg) by Darth Stabro, used under CC BY-SA.）](/images/webgl/VectorBitmapExample.png)

那么其他那些web上的图像／动画制作的渲染引擎是如何解决这个问题的呢？这里我列举三个知名度较高的，同时支持WebGL的渲染引擎来对比。

#### CreateJS

CreateJS在2014年就较早的[宣布](http://blog.createjs.com/webgl-support-easeljs/)在满足条件的浏览器内支持使用WebGL而不是一直以来的Canvas 2D来渲染2D图像。但仔细看这篇介绍就会发现它的支持是有限的，原因是它只支持在渲染**位图**的时候使用WebGL，也就是我在上文介绍的将长方形切割成两个三角形的简单方法。

这样一种位图用WebGL，矢量图用Canvas 2D的策略[据称](http://blog.createjs.com/webgl-support-easeljs/)已经能带来可观的性能提升了。但是需要注意的是这带来了一个副作用。

熟悉HTML Canvas的读者应该都知道，网页中一个单个的canvas元素可以且仅可以通过```canvas.getContext()```生成一种绘图模式，2D的```CanvasRenderingContext2D```或是WebGL的```WebGLRenderingContext```。无论哪一种，一旦生成一次之后再也无法在同一个canvas元素上生成另一种。这意味着，在使用CreateJS的时候，一旦一个场景中既使用了位图又使用了矢量图，这两类图像是绝对无法被绘制在同一个canvas元素里的。CreateJS也[承认](http://blog.createjs.com/webgl-support-easeljs/)了，在这种情况下需要使用它们称之为「Layered Renderers”」的方法，通俗点说就是在一个canvas上再叠加一个canvas，一个专门用WebGL，一个专门用Canvas 2D。

![Layered renderers](/images/webgl/createjs.png)

那么，熟悉浏览器运行机制的读者很容易就发现了，这样的叠加极有可能给浏览器本身的渲染引擎带来额外的负担，尤其当处在上层的canvas包含一些半透明图像的时候，在一定程度上浏览器的渲染性能会受到影响。

#### pixi.js

这是另一个在业内也颇有名气的2D动画／游戏制作引擎，并且也在最近的几个版本开始了优先使用WebGL的支持。首先，pixi.js并不限定仅仅在位图的渲染时使用WebGL，在矢量图上他们的开发者也做了不小的[努力](http://www.goodboydigital.com/pixi-js-v1-6-0-in-da-house/)。

pixi.js用WebGL渲染矢量图的方法核心是借助stencil buffer。在[这篇文章](http://www.cs.sun.ac.za/~lvzijl/courses/rw778/grafika/OpenGLtuts/Big/graphicsnotes014.html)中的相关小节有对这个方法的详细解说。

![http://www.cs.sun.ac.za/~lvzijl/courses/rw778/grafika/OpenGLtuts/Big/graphicsnotes014.html](/images/webgl/pixijs.png)

这种方法的优点很明显，就是可以轻松的应对各种复杂的多边形，但同时它的缺点也很明显。

首先，上图中的B，C，E，G，H区域虽然在最终的渲染结果中不会显现，但是在渲染的过程中，这些区域中的像素其实也经历了大部分的渲染流程并在stencil buffer记录下了相应的数值，这整个过程就占用了GPU的计算资源。

再有，不仅仅是这些看不见的区域，上图中大部分的区域其实都被重复性的渲染了若干次，最终留下那些被渲染了奇数次的像素们。这又是对GPU等相关计算资源的不小的浪费。

最后，stencil buffer并不是WebGL标准里的必须配置，也就是说，要使用这种方法必须首先确保用户所用的浏览器和硬件支持stencil buffer。


#### three.js

在一定程度上，提到WebGL就不得不提three.js，甚至在很多人眼里要用WebGL就必须用three.js。作为一个“正统派”的WebGL库，three.js的API主要还是针对3D应用而设计，但是他们同时也对那些非主流的需要用WebGL绘制2D图像的用户提供了相应的基本支持，并且他们的做法是我个人更欣赏的直接了当的一种选择：既然WebGL几乎只能画三角形，那就把所有其他的图形都切割成由三角形组成的。这个过程称为triangulation

![http://mathworld.wolfram.com/Triangulation.html](/images/webgl/triangulation.gif)

但是three.js提供的triangulation的API有一定的局限性，具体来说就是构成这个图形的所有向量的顶点必须被分为两大个子集来传递给这个API，这两个子集分别是外围轮廓上的点和内部轮廓（内部的洞，穴的轮廓）。但在实际应用中要满足这一点要求并不总是很容易。特别是通过其他2D制作软件绘制的图形直接输出向量后在传递给three.js来调用WebGL渲染前，要逐个分析确认每个图形是否包含“洞”，如果包含就必须把内外向量分开，事情就变得复杂无比。

### Tesspathy

![Tesspathy](/images/webgl/tesspathy_logo.jpg)

除了上面介绍的这些方法外，我在实际项目中为了解决这个问题还自己写了一个开源项目，名为[Tesspathy](http://gree.github.io/tesspathy/)。项目的地址是https://github.com/gree/tesspathy

Tesspathy是一个将2D矢量图形切割成由三角形的矢量图的工具，它有别于现有其他类似工具的特点有：

* 不对原始数据设置前提要求和区分（轮廓点的顺时针／逆时针方向，单个或多个图形的集合，是否有洞，哪些向量是洞等等）
* 不仅能切割直线轮廓的多边形，而且能切割含有曲线轮廓的图形（目前版本暂只支持quadratic Bezier曲线）
* 曲线仍然保持曲线形式而非分割成细小的三角形，因此切割后仍是完全不依赖分辨率的矢量图形。
* 不仅能切割填充图形，还能用于切割直线和曲线

来看一个实际的例子。下图是一个由矢量线段和曲线构成的包含“洞”的图形：

![切割前](/images/webgl/tesspathy_1.png)

为了能使用WebGL渲染它，我们用Tesspathy将其转换（切割）成完全有三角形构成的矢量图：

![切割后](/images/webgl/tesspathy_2.png)

这样最后只需将这些三角形传递给WebGL的API即可用WebGL来渲染2D矢量图像了。特别注意这里的曲线并没有被切割成大量细小的三角形，这也就意味着即使被放大至较高的分辨率也不需要进行再次的切割。感兴趣的读者可以自己动手[实际尝试](http://gree.github.io/tesspathy/demos/drawing_pad/)一下上面这个例子。

看到这里也许有人会问，费这么大劲用这么复杂的方式画2D图像太别扭太奇怪了，既然这样为什么不干脆从一开始就用Canvas 2D的API而非要走WebGL这条路？

这里面有两个问题需要阐释。

首先，用这种“大费周章“的方式画2D图像一点也不奇怪，因为浏览器的渲染引擎就是这么做的。Canvas 2D的API之所以看上去那么简洁好用，是因为渲染引擎在内部帮助完成了实际的极为复杂的渲染流程。

口说无凭，来看看Chrome的渲染引擎Skia的[相关代码](https://skia.googlesource.com/skia/+/master/src/gpu/batches/)：

![Source code of Skia" %}
](/images/webgl/skia.png)

在这个目录下包含了用于渲染矢量图形（Path）的各类“预处理“，之所以说各类是因为作为Skia这样一个关键的浏览器渲染引擎，对性能的追求是极为严苛的，因此开发者分别针对的是不同条件的矢量图形做了优化，选择最适合的预处理方式。比如，当一个图形是完全Convex的多边形时，可以跳过很多繁琐的步骤而直接利用GL系API中的GL_TRIANGLE_FAN来直接渲染。

尤其值得一提的是[GrTessellatingPathRenderer.cpp](https://skia.googlesource.com/skia/+/master/src/gpu/batches/GrTessellatingPathRenderer.cpp)这个文件，其中的内容就是对于没有特定特征的任意矢量图形将其切割成三角形的组合。是不是很眼熟？这正是上文提到的[Tesspathy](http://gree.github.io/tesspathy/)所做的工作。至于为什么既然浏览器已经替我们完成了这部分工作（并且是用C++而非JS）还要用Tesspathy来自己动手呢？详细的原因会在下一篇中阐述，但概括起来就是两个字：性能。

你逗我呢在浏览器里跑的C++代码性能会输给在客户端沙盒里运行的JS？我这里所指的性能并非切割三角形的性能，而是最终的图像渲染性能。用Tesspathy＋WebGL的方式渲染2D矢量图像的性能可以大大超过用原生Canvas 2D并开启硬件加速（即在Skia中切割三角形从而调用底层OpenGL或Direct3D进行渲染）的方式，为此我专门做了一个[Demo](http://gree.github.io/tesspathy/demos/canvas_vs_webgl/)来实际比较两者的差别，尤其在手机上二者的差距应该更为明显。(由于HTML5的canvas元素的渲染模式无法热切换，在Demo中更换模式时请先刷新一次页面)

关于WebGL以及原生Canvas 2D的性能问题我会在下一篇中具体阐释。