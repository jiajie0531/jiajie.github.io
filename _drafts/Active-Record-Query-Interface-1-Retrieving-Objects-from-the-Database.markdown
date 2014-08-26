---
layout: post
title:  "Active Record Query Interface 1 从数据库获取对象"
date:   2014-08-26 17:05:55
categories: Rails
tags: Rails Active Record Query Interface
---

如果你过去使用原生SQL来查找数据库记录，那么你将会在Rails里发现有其他的方式来操作原来相同的操作。Active Record可以让你在大部分情况下不需要使用原生的SQL。

在这份教程里的代码例子将会关联到一个或者多个接下来的models：

>information:
>
>下面所有的models使用id来作为主键，除非是另外说明的。

{% highlight ruby %}
class Client < ActiveRecord::Base
  has_one :address
  has_many :orders
  has_and_belongs_to_many :roles
end
{% endhighlight %}

{% highlight ruby %}
class Address < ActiveRecord::Base
  belongs_to :client
end
{% endhighlight %}

{% highlight ruby %}
class Order < ActiveRecord::Base
  belongs_to :client, counter_cache: true
end
{% endhighlight %}

{% highlight ruby %}
class Role < ActiveRecord::Base
  has_and_belongs_to_many :clients
end
{% endhighlight %}

Active Record 将会为你在数据库里执行查询，能兼容大部分的数据库系统（MySql，PostgreSQL，SQLite等等）。无论你使用的是哪个数据库系统，Active Record方法格式都是一样的。

# 1 Retrieving Objects from the Database
