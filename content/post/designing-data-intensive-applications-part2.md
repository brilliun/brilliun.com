
---
title: "Designing Data Intensive Applications - Part II"
description: ""
keywords: []
tags: ['database', 'distributed system']
categories: ['Reading']
date: 2019-12-09T21:48:35+09:00
lastmod: 2019-12-09T21:12:54+09:00

draft: true
isCJKLanguage: false

comments: true
showTags: true
showPagination: true
showSocial: true
showDate: true
showMeta: true
showActions: true

---

<!-- toc -->


# CH03. Storage and Retrieval

Why should application developers care how the database handles storage and retrieval internally? Because there is a big difference between storage engines that are optimized for transactional workloads and those that are optimized for analytics.

There are two families of commonly used storage engines: *log-structured storage engines*, and *page-oriented storage engines* such as B-trees.