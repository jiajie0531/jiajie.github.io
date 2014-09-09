---
layout: post
title:  "Active Record Query Interface 4 选择指定的字段"
date:   2014-09-09 23:05:55
categories: Rails
tags: Rails Active Record Query Interface
---

## 4 选择指定的字段

默认情况下，Model.find使用select * 从结果集选择所有的字段.

为了从结果集里只选择多个字段的一个子集，你通过select方法能够指定一个子集。

例如，为了只选择字段viewable_by和locked：

{% highlight ruby %}
Client.select("viewable_by, locked")
{% endhighlight %}

通过find的调用SQL查询就会类似于：

{% highlight sql %}
SELECT viewable_by, locked FROM clients
{% endhighlight %}