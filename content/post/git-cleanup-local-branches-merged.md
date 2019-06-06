---
title: "Git Cleanup Local Branches Merged"
date: 2017-06-15T10:42:21+09:00
lastmod: 2017-06-15T10:42:21+09:00
draft: false
keywords: []
description: ""
tags: ['git']
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

