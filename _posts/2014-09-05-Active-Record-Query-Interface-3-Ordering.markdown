---
layout: post
title:  "Active Record Query Interface 3 排序"
date:   2014-09-05 17:05:55
categories: Rails
tags: Rails Active Record Query Interface
---

## 3 Ordering 排序

为了按照特定的顺序从数据库里获取数据记录，你可以使用order方法。

例如，如果你在数据库里正获得一堆数据记录，想要给他们按照created_at字段做升序排序：

{% highlight ruby %}
Client.order(:created_at)
# OR
Client.order("created_at")
{% endhighlight %}

你也可以声明ASC（升序）还是DESC（倒序）：

{% highlight ruby %}
Client.order(created_at: :desc)
# OR
Client.order(created_at: :asc)
# OR
Client.order("created_at DESC")
# OR
Client.order("created_at ASC")
{% endhighlight %}

或者多个字段的排序：

{% highlight ruby %}
Client.order(orders_count: :asc, created_at: :desc)
# OR
Client.order(:orders_count, created_at: :desc)
# OR
Client.order("orders_count ASC, created_at DESC")
# OR
Client.order("orders_count ASC", "created_at DESC")
{% endhighlight %}

如果你想要调用多次排序，类似于在不同的上下文里，新的排序会附加到上一个排序后。

{% highlight ruby %}
Client.order("orders_count ASC").order("created_at DESC")
# SELECT * FROM clients ORDER BY orders_count ASC, created_at DESC
{% endhighlight %}



