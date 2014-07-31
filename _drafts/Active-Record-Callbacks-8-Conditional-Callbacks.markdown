---
layout: post
title:  "Active Record Callbacks 8 条件回调函数"
date:   2014-07-31 23:05:55
categories: Rails
tags: Rails Active Record Callbacks
---

如同校验一样，我们也可以调用一个有条件的回调函数，在满足给定的判断条件下。我们能够使用:if和:unless属性来实现这个，后面可以跟一个标志，一个字符串，一段代码或者一个数组。