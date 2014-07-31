---
layout: post
title:  "Active Record Callbacks 5 跳过回调函数 6 停止执行"
date:   2014-07-31 22:45:55
categories: Rails
tags: Rails Active Record Callbacks
---

只在当校验时，使用下面的方法也能够跳过回调函数：

*  decrement
*  decrement_counter
*  delete
*  delete_all
*  increment
*  increment_counter
*  toggle
*  touch
*  update_column
*  update_columns
*  update_all
*  update_counters

这些方法应该谨慎使用，然而，因为重要的业务规则和应用逻辑可能会包含在回调函数里面。绕过他们会失去一些潜在的细节，这会导致一些无效的数据。

### 6 停止执行

你开始在你的模型中注册新的回调函数，然后他们会按照顺序来执行。这个顺序会包括你模型所有的校验，已注册的回调函数，以及数据库的操作都会被执行。

整个回调函数序列被封装在一个事务里。如果任何*before*方法返回的是false或者抛出一个异常，这个执行链会停止，一个ROLLBACK发生；*after*回调函数只能在抛出异常的情况下才能完成。

>warning
>
>在回调函数链被停止的时候，只要不是ActiveRecord::Rollback的异常都被Rails重复抛出。抛出一个不是ActiveRecord::Rollback的异常能打断代码，不期望类似于save和update_attributes（通常会尝试返回true或者false）的方法去抛出异常。

