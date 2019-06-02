---
title: "WebGL Part4"
date: 2015-12-06T22:39:17+09:00
lastmod: 2015-12-06T22:39:17+09:00
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

作为这个系列的最后一篇，我想谈一下自己对于WebGL这项技术的本质的理解。

这里想要顺带提一下的是“Extensible Web”这个理念或者说运动。它的核心主要是下面两条准则：

* Web标准应该越来越多地将各种相关的底层功能对Javascript开放
* 现有的高层特性应该尽可能地转化为可以由Javascript访问或者说控制

读上去比较抽象，通俗一点的解释就是不要再把Javascript程序当作幼儿一般“照料”和“限制”，稍微“危险”一点的事情要么禁止要么由监护人（浏览器）代劳。Javascript以及由其驱动的Web应用正在茁壮成长，以往只赋予native app的那些权限和能力今后应该尽可能多地对Javascript开放。值得欣慰的是，近年来web技术已经在这方面取得了长足的发展，看看 [What Web Can Do Today](https://whatwebcando.today/) 就能发现大部分以往专属于native app的功能如今在web应用中也能实现了。
