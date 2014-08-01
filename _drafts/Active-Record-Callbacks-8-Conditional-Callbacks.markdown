---
layout: post
title:  "Active Record Callbacks 8 条件回调函数"
date:   2014-07-31 23:05:55
categories: Rails
tags: Rails Active Record Callbacks
---

如同校验一样，我们也可以调用一个有条件的回调函数，在满足给定的判断条件下。我们能够使用:if和:unless属性来实现这个，后面可以跟一个标志，一个字符串，一段代码或者一个数组。当你想要指定回调函数应该什么条件下被调用，你可以使用:if属性。如果你想要指定回调函数在什么条件下不应该被调用，那么你可以使用:unless属性。

### 8.1 使用:if和:unless和标志

你能够把:if和:unless属性和标志关联起来，这个标志对应的是一个断言方法的名称，这个方法在回调函数之前被调用。当使用:if选项时，如果断言方法返回false，回调函数将不会被执行；当使用:unless选项时，如果断言方法返回true，回调函数将不会被执行。