---
layout: post
title:  "Active Record Query Interface 1 从数据库获取对象"
date:   2014-08-26 22:05:55
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

# 1 Retrieving Objects from the Database 从数据库里获取对象

为了从数据库里获取对象，Active Record 提供了多个查询方法。每个查找方法允许你把参数代入，用来执行对于数据库的某些查询而不用写原声SQL。

这些方法是：

* bind
* create_with
* distinct
* eager_load
* extending
* from
* group
* having
* includes
* joins
* limit
* lock
* none
* offset
* order
* preload
* readonly
* references
* reorder
* reverse_order
* select
* uniq
* where

所有上述的方法都会返回一个ActiveRecord::Relation实例。

Model.find(options)主要操作可以总结为：

* 把所提供的参数选项转换成等价的SQL查询。
* 激活SQL查询，从数据库里获取相对应的查询结果。
* 对于每一个结果行实例化合适model的等价Ruby对象。
* 如果可以的话，运行after_find回调函数。

## 1.1 Retrieving a Single Object 获取单个对象

Active Record提供了多种不同的方式来获取单个对象。

### 1.1.1 Using a Primary Key 使用一个主键

使用Model.find(primary_key)，你能够获取所对应主键的对象，该方法可以匹配任何所提供的参数选项。例如：

{% highlight ruby %}
# Find the client with primary key (id) 10.
client = Client.find(10)
# => #<Client id: 10, first_name: "Ryan">
{% endhighlight %}

上述代码所等价的SQL是：

{% highlight sql %}
SELECT * FROM clients WHERE (clients.id = 10) LIMIT 1
{% endhighlight %}



