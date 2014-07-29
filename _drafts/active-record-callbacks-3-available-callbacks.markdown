---
layout: post
title:  "Active Record Callbacks 3 可用的回调函数"
date:   2014-07-29 21:02
categories: Rails
tags: Rails Active Record Callbacks
---

这里罗列的是所有可用的Active Record的回调函数，罗列在同一个规则中，在各自的操作中他们会被调用：

### 3.1 创建一个对象

*  before_validation
*  after_validation
*  before_save
*  around_save
*  before_create
*  around_create
*  after_create
*  after_save

### 3.2 更新一个对象

*  before_validation
*  after_validation
*  before_save
*  around_save
*  before_update
*  around_update
*  after_update
*  after_save

### 3.3 销毁一个对象

*  before_destroy
*  around_destroy
*  after_destroy

>warning
>
>after_save在create和update中都有运行，但通常来说after比较经典的回调函数是after_create和after_update，无论在哪个规则中宏的调用都能被执行

### 3.4 after_initialize和after_find

