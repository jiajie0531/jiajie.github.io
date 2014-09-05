---
layout: post
title:  "Active Record Query Interface 2 查询条件"
date:   2014-09-03 17:35:55
categories: Rails
tags: Rails Active Record Query Interface
---

## 2 Conditions 查询条件

where方法允许你指定查询条件来限制所返回的数据记录，表示的是SQL语句里WHERE的部分。查询条件可以是字符串，数组，或者哈希(Hash)。

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

类似于（？）替代型参数，你也能够在你的数组查询条件里指定key/value 哈希值：

{% highlight ruby %}
Client.where("created_at >= :start_date AND created_at <= :end_date",
  {start_date: params[:start_date], end_date: params[:end_date]})
{% endhighlight %}

如果你有大量的变量查询条件，这使得可读性变得更清晰。

## 2.3 Hash Conditions 哈希查询条件

Active Record也允许你传入哈希查询条件里，这能够提升你查询条件语法的可读性。有了哈希查询条件，你传入哈希里你所要查询条件的keys字段，以及你所要查询条件的values。

>note
>
>只要在哈希（Hash）查询条件里的赋值等号，范围和子集校验可用即可。

### 2.3.1 Equality Conditions 赋值等号查询条件

{% highlight ruby %}
Client.where(locked: true)
{% endhighlight %}

该字段名称也能够是一个字符串：

{% highlight ruby %}
Client.where('locked' => true)
{% endhighlight %}

在一个belongs_to关系的例子里，一个关联性key能够被用来指定一个model，如果一个Active Record object被指定为该值时。这个方法也是用于多态性关联里的。

{% highlight ruby %}
Post.where(author: author)
Author.joins(:posts).where(posts: { author: author })
{% endhighlight %}

>note
>
>这些值不能够是symbols。例如，你不够执行Client.where(status:active)

### 2.3.2 Range Conditions 范围查询条件

{% highlight ruby %}
Client.where(created_at: (Time.now.midnight - 1.day)..Time.now.midnight)
{% endhighlight %}

这将会查找所有昨天新建的clients，通过使用一个BETWEEN SQL语句。

{% highlight sql %}
SELECT * FROM clients WHERE (clients.created_at BETWEEN '2008-12-21 00:00:00' AND '2008-12-22 00:00:00')
{% endhighlight %}

这个示范例子可以参考[Array Conditions](http://guides.rubyonrails.org/active_record_querying.html#array-conditions)

### 2.3.3 Subset Conditions 子集查询条件

如果你想要查找数据记录使用IN表达式，你可以把一个数组传入查询条件的哈希里（hash）。

{% highlight ruby %}
Client.where(orders_count: [1,3,5])
{% endhighlight %}

该代码所产生的SQL类似于：

{% highlight sql %}
SELECT * FROM clients WHERE (clients.orders_count IN (1,3,5))
{% endhighlight %}

## 2.4 NOT Conditions

NOT SQL 查询可以通过where.not来建立。

{% highlight ruby %}
Post.where.not(author: author)
{% endhighlight %}

也就是说，这个查询更够被没有参数的where所响应而生成代码，随后立即禁止在传入where的查询条件。

