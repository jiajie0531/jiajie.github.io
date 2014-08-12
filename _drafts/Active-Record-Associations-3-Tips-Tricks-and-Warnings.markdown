---
layout: post
title:  "Active Record Associations 3 提示，技巧，警告"
date:   2014-09-12 21:25:55
categories: Rails
tags: Rails Active Record Associations
---

![P1000759](/assets/2014/08/12/P1000759.gif)

### 3 提示，技巧，警告

这里一些内容你应该要知道，可以在你的Rails应用中更有效率的使用Active Record association（关联性）。

* 控制缓存
* 避免命名冲突
* 更新schema
* 控制关联性范围
* 双向关联

### 3.1 控制缓存

所有关联性的方法都是围绕缓存来建立，为了能执行进一步的操作而保持最近的查询结果。缓存甚至能在跨方法共享。例如：

{% highlight ruby %}
customer.orders                 # retrieves orders from the database
customer.orders.size            # uses the cached copy of orders
customer.orders.empty?          # uses the cached copy of orders
{% endhighlight %}

但假使你想要重新载入缓存，因为数据可能已经被应用的其他部分改变了？只要给关联性调用传入true即可：

{% highlight ruby %}
customer.orders                 # retrieves orders from the database
customer.orders.size            # uses the cached copy of orders
customer.orders(true).empty?    # discards the cached copy of orders
                                # and goes back to the database
{% endhighlight %}

### 3.2 避免命名冲突

你不可以随便命名你的关系。因为创建一个关系就是意味着增加一个方法，会给model一个名字，比较糟的办法是给一个关系的名字就是已经被ActiveRecord::Base的方法实例所使用的。