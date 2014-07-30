---
layout: post
title:  "Active Record Callbacks 5 跳过回调函数 6 停止执行"
date:   2014-07-30 22:45:55
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