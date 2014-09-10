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

要注意，因为这也就意味着你只要选择几个字段就能初始化一个model对象。如果你想要读取一个字段，该字段不会存在于你将会获取到的被初始化的数据记录里。

{% highlight ruby %}
ActiveModel::MissingAttributeError: missing attribute: <attribute>
{% endhighlight %}

对于\<attribute>是你所要求的属性。id方法将不会抛出ActiveRecord::MissingAttributeError，
