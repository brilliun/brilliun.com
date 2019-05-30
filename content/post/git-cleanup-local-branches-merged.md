---
title: "Git Cleanup Local Branches Merged"
date: 2017-06-15T10:42:21+09:00
lastmod: 2017-06-15T10:42:21+09:00
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

## TL;DR

```
$ git branch --merged develop | egrep -v "(^\*|master|dev)" | xargs -n 1 git branch -d
```
## BTW

```
git branch --no-merged
```
can be used to check the branches to keep.

## Breakdown


### 1. Find branches merged

```
# list up branches merged into 'develop'
git branch --merged develop
```
OR
```
# list up branches merged into current branch
git branch --merged
```

### 2. Exclude critical branches

```
# Exclude branches with prefix of "*"(current), "master", or "dev"
egrep -v "(^\*|master|dev)"
```

**`egrep` is equivalent to `grep -E`

### 3. Delete branches

```
xargs -n 1 git branch -d
```
With `-d` option, only merged branches can be successfully deleted, while `-D` means force deletion.

The `-n 1` option ensures at most one argument can be passed by `xargs` to following command.

