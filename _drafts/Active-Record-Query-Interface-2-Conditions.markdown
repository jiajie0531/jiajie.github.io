---
layout: post
title:  "Active Record Query Interface 2 查询条件"
date:   2014-09-03 17:35:55
categories: Rails
tags: Rails Active Record Query Interface
---

# 2 Conditions 查询条件

where方法允许你指定条件来限制所返回的数据记录，表示的是SQL语句里WHERE的部分。查询条件可以是字符串，数组，或者哈希。

## 2.1 Pure String Conditions 纯字符串的查询条件

如果你想要在你的查询中增加条件，你就可以在指定的位置增加即可，类似于 Client.where("orders_count = '2'")。这就会查询当orders_count字段的值为2的所有的clients。

>warning
>
>建立你自己的查询条件作为纯字符串，这可以让你变得脆弱对于SQL注入的攻击。例如，Client.where("first_name LIKE '%#{params[:first_name]}%'")是不安全的。看下一个部分，是更好的方式来处理查询条件，就是使用数组。

## 2.2 Array Conditions 数组查询条件

现在如果那个数字是能够变化的，也就是说来自于某个？的参数，查询就会以短语的形式：

{% highlight ruby %}
Client.where("orders_count = ?", params[:orders])
{% endhighlight %}

Active Record将会通过在查询条件中第一个元素的值，在第一个元素里另外的元素将会替代问号（？）。

如果你想要指定多个查询条件：

{% highlight ruby %}
Client.where("orders_count = ? AND locked = ?", params[:orders], false)
{% endhighlight %}

在这个例子里，第一个问号标志将会被params[:orders]的值所替代，第二个将会被SQL表达式false所替代，这些都依赖于适配器。

这个代码更适于：

{% highlight ruby %}
Client.where("orders_count = ?", params[:orders])
{% endhighlight %}

相对于这个代码：

{% highlight ruby %}
Client.where("orders_count = #{params[:orders]}")
{% endhighlight %}

因为参数化安全。把变量直接放入查询条件的字符串里，这会导致传输变量到数据库为as-is。这意味着它将是一个非转义的变量，对于心怀恶意的用户而言。如果这样做了，你会把整个数据库置于危险里，因为一旦一个用户找到方法可以攻击你的数据库，他们就可以对它做任何事情。所以任何时候都不要把你的参数直接置于查询条件的字符串里。

>information
>
>关于SQL注入更多的危险性，可以参考[ Ruby on Rails Security Guide.](http://guides.rubyonrails.org/security.html#sql-injection)

### 2.2.1 Placeholder Conditions 占位符查询条件




