---
layout: post
title:  "Active Record Query Interface 1 从数据库获取对象"
date:   2014-08-26 22:05:55
categories: Rails
tags: Rails Active Record Query Interface
---

如果你过去使用原生SQL语句来查找数据库记录，那么你将会在Rails里发现有其他的方式来操作原来相同的操作。Active Record可以让你在大部分情况下不需要使用原生的SQL。

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

为了从数据库里获取对象，Active Record 提供了多个查询方法。每个查找方法允许你把参数代入，用来执行对于数据库的某些查询而不用写原生SQL。

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

Model.find(primary_key)如果没匹配记录没有找到的话，将会抛出一个ActiveRecord::RecordNotFound异常。

### 1.1.2 take

Model.take获得一条记录，不带有任何隐藏的排序。例如：

{% highlight ruby %}
client = Client.take
# => #<Client id: 1, first_name: "Lifo">
{% endhighlight %}

上述代码所等价的SQL是：

{% highlight sql %}
SELECT * FROM clients LIMIT 1
{% endhighlight %}

如果没有数据记录被发现，那么Model.take返回nil，也没有异常被抛出。

>information:
>
>所获取的数据记录可能非常依赖数据库引擎。

### 1.1.3 first

Model.first按照主键的顺序查找第一条数据记录。例如：

{% highlight ruby %}
client = Client.first
# => #<Client id: 1, first_name: "Lifo">
{% endhighlight %}

上述代码所等价的SQL是：

{% highlight sql %}
SELECT * FROM clients ORDER BY clients.id ASC LIMIT 1
{% endhighlight %}

如果没有匹配的数据记录被发现，那么Model.first返回nil，也没有异常被抛出。

### 1.1.4 last

Model.last按照主键的顺序，查找最后一条数据记录。例如：

{% highlight ruby %}
client = Client.last
# => #<Client id: 221, first_name: "Russel">
{% endhighlight %}

上述代码所等价的SQL是：

{% highlight sql %}
SELECT * FROM clients ORDER BY clients.id DESC LIMIT 1
{% endhighlight %}

如果没有匹配的数据记录被发现，那么Model.last返回nil，也没有异常被抛出。

### 1.1.5 find_by

Model.find_by基于一些条件来查找第一条数据记录。例如：

{% highlight ruby %}
Client.find_by first_name: 'Lifo'
# => #<Client id: 1, first_name: "Lifo">
 
Client.find_by first_name: 'Jon'
# => nil
{% endhighlight %}

等价于这样来写：

{% highlight ruby %}
Client.where(first_name: 'Lifo').take
{% endhighlight %}

### 1.1.6 take!

Model.take!获得一条数据记录，而没有隐含的排序。例如：

{% highlight ruby %}
client = Client.take!
# => #<Client id: 1, first_name: "Lifo">
{% endhighlight %}

上述代码所等价的SQL是：

{% highlight sql %}
SELECT * FROM clients LIMIT 1
{% endhighlight %}

如果没有没有匹配的数据记录被查找到的话，Model.take!会抛出异常ActiveRecord::RecordNotFound。

### 1.1.7 first!

Model.first!依据主键顺序查找到第一条数据记录。例如：

{% highlight ruby %}
client = Client.first!
# => #<Client id: 1, first_name: "Lifo">
{% endhighlight %}

上述代码所等价的SQL是：

{% highlight sql %}
SELECT * FROM clients ORDER BY clients.id ASC LIMIT 1
{% endhighlight %}

如果没有没有匹配的数据记录被查找到的话，Model.first!会抛出异常ActiveRecord::RecordNotFound。

### 1.1.8 last!

Model.last!依据主键顺序查找到最后一条数据记录。例如：

{% highlight ruby %}
client = Client.last!
# => #<Client id: 221, first_name: "Russel">
{% endhighlight %}

上述代码所等价的SQL是：

{% highlight sql %}
SELECT * FROM clients ORDER BY clients.id DESC LIMIT 1
{% endhighlight %}

如果没有没有匹配的数据记录被查找到的话，Model.last!会抛出异常ActiveRecord::RecordNotFound。

### 1.1.9 find_by!

Model.find_by!依据匹配的条件查询到第一条数据记录。果没有没有匹配的数据记录被查找到的话，Model.last!会抛出异常ActiveRecord::RecordNotFound。例如：

{% highlight ruby %}
Client.find_by! first_name: 'Lifo'
# => #<Client id: 1, first_name: "Lifo">
 
Client.find_by! first_name: 'Jon'
# => ActiveRecord::RecordNotFound
{% endhighlight %}

上述代码所等价是：

{% highlight ruby %}
Client.where(first_name: 'Lifo').take!
{% endhighlight %}

## 1.2 Retrieving Multiple Objects 获取多个对象

### 1.2.1 Using Multiple Primary Keys 使用多个主键

Model.find(array_of_primary_key)接受一组主键，返回一组包含所提供主键的匹配数据记录。例如：

{% highlight ruby %}
# Find the clients with primary keys 1 and 10.
client = Client.find([1, 10]) # Or even Client.find(1, 10)
# => [#<Client id: 1, first_name: "Lifo">, #<Client id: 10, first_name: "Ryan">]
{% endhighlight %}

上述代码所等价的SQL是：

{% highlight sql %}
SELECT * FROM clients WHERE (clients.id IN (1,10))
{% endhighlight %}

>warning
>
>Model.find(array_of_primary_key)将会抛出一个ActiveRecord::RecordNotFound异常，除非对于所提供的所有主键值其一个匹配的数据记录被找到。

### 1.2.2 take

Model.take(limit)获取到所指定数量的记录行数，而不带有显式的排序：

{% highlight ruby %}
Client.take(2)
# => [#<Client id: 1, first_name: "Lifo">,
      #<Client id: 2, first_name: "Raf">]
{% endhighlight %}

上述代码所等价的SQL是：

{% highlight sql %}
SELECT * FROM clients LIMIT 2
{% endhighlight %}

### 1.2.3 first

Model.first(limit)依据主键顺序查询所指定数量的记录数：

{% highlight ruby %}
Client.first(2)
# => [#<Client id: 1, first_name: "Lifo">,
      #<Client id: 2, first_name: "Raf">]
{% endhighlight %}

上述代码所等价的SQL是：

{% highlight sql %}
SELECT * FROM clients ORDER BY id ASC LIMIT 2
{% endhighlight %}

### 1.2.4 last

Model.last(limit)依据主键倒序查询所指定数量的记录数：

{% highlight ruby %}
Client.last(2)
# => [#<Client id: 10, first_name: "Ryan">,
      #<Client id: 9, first_name: "John">]
{% endhighlight %}

上述代码所等价的SQL是：

{% highlight sql %}
SELECT * FROM clients ORDER BY id DESC LIMIT 2
{% endhighlight %}

## 1.3 Retrieving Multiple Objects in Batches 批量获取多个对象

我们经常需要去迭代一个大型集合的数据记录，特别是在我们发送一个实时通讯给一个大型集合的用户，或者当我们想导出数据时。

这可能看起来很简单：

{% highlight ruby %}
# This is very inefficient when the users table has thousands of rows.
User.all.each do |user|
  NewsLetter.weekly_deliver(user)
end
{% endhighlight %}

但随着数据表越来越大，这个方法也变得不可实现，既然User.all.each让Active Record在单个过程里取回整个数据表，给每行建立一个model对象，然后在内存里保存整个model对象的数组。事实上，如果我们有大数据量的数据记录，整个集合可能会超过可用内存的容量。

Rails提供了两种方法，通过在memory-friendly批处理执行的运算来划分数据记录而可以用来定位这个问题。第一个方法，find_each，获取一组数据记录
然后把每一条记录分别转变为一个model。第二个方法，find_in_batches，获取一组数据记录，然后把整个一组数据整个转变为一组models。

>information
>
>find_each和find_in_batches方法趋向于在大数据里的批处理中来使用，不会一下子占据全部内存。如果你只是需要循环一千条数据记录，那么find方法是更合适的选择。

### 1.3.1 find_each

find_each方法获取一组数据记录，然后把每一条记录分别转变为一个model。在下面的例子里，find_each将会获取1000条数据记录(这是find_each和find_in_batches当前的默认值)，然后每一个数据记录分别变为一个model。这个过程是重复的直到所有数据记录被处理完：

{% highlight ruby %}
User.find_each do |user|
  NewsLetter.weekly_deliver(user)
end
{% endhighlight %}

#### 1.3.1.1 Options for find_each 

find_each方法接受find方法大部分的选项，除去:order和:limit，这两个是被find_each保留为内部使用的。

额外的两个选项，:batch_size和:start也是可以使用的。

:batch_size

:batch_size选项允许你去指定每一次所获取的数据记录数，在被单独地传入至block里。例如，批量所获取的数据记录数为5000：

{% highlight ruby %}
User.find_each(batch_size: 5000) do |user|
  NewsLetter.weekly_deliver(user)
end
{% endhighlight %}

:start

默认情况下，数据记录按照主键升序被抓取，主键必须是integer类型。:start选项允许你去配置第一个ID顺序，无论何时最小的ID不是你所需要的。这将是有用处的，例如，如果你想要继续一个中断的批处理过程，提供你保存的最后一个ID作为校验点即可。

例如，为了发送newsletter给users，其主键是从2000开始，获取到的数据块大小为5000：

{% highlight ruby %}
User.find_each(start: 2000, batch_size: 5000) do |user|
  NewsLetter.weekly_deliver(user)
end
{% endhighlight %}

另外一个例子将是，如果你想要多个处理同一个流程。你可以让每一个worker处理10000条数据记录，只要通过设置每一个worker的:start选项即可。

### 1.3.2 find_in_batches

find_in_batches方法是类似于find_each方法，两者都是获取数据记录块。不同之处在于find_in_batches把整块数据记录变为一个models数组，而不是分开来分别转换。下面的例子，将会把所提供的数据块同时处理1000条invoices，最后的数据块包含剩下来的invoices：

{% highlight ruby %}
# Give add_invoices an array of 1000 invoices at a time
Invoice.find_in_batches(include: :invoice_lines) do |invoices|
  export.add_invoices(invoices)
end
{% endhighlight %}

>note
>
>:include选项允许你去命名关联性，应该和models并排载入。

#### 1.3.2.1 Options for find_in_batches

find_in_batches方法接受find_each方法相同的选项:batch_size和:start选项，同样也和find方法大致一样的选项，除了:order和:limit，这两个选项被保留find_in_batches内部使用。
