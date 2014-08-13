---
layout: post
title:  "Active Record Associations 3 提示，技巧，警告"
date:   2014-08-12 21:25:55
categories: Rails
tags: Rails Active Record Associations
---

![P1000759](/assets/2014/08/12/P1000759.gif)

### 3 提示，技巧，警告

这里一些内容你应该要知道，可以在你的Rails应用中更有效率地使用Active Record association（关联性）。

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

你不可以随便命名你的关联性。因为创建一个关系就是意味着增加一个方法，会给model一个名字，比较糟的办法是给一个关联性的名字就是已经被ActiveRecord::Base的方法实例所使用的。这个关系方法将会重写基础(base)方法，打破原有的方法逻辑。例如，对于关系命名attributes或者connection是不太好的名称。

### 3.3 更新模式(Schema)

关联性是非常有用的，但他们不是魔术。你有责任维护你的数据库模式匹配你的关联性。实际上，这意味着两个事情，依赖于你所创建关联性的顺序是什么。对于belongs_to关系，你需要去创建一个外键，对于has_and_belongs_to_many关系，你需要创建合适的联合（join）表。

#### 3.3.1 对于belongs_to关系创建外键

当你声明一个belongs_to关系时，你需要去创建一个合适的外键。例如，考虑一下这个model：

{% highlight ruby %}
class Order < ActiveRecord::Base
  belongs_to :customer
end
{% endhighlight %}

这个声明需要回退到在orders数据表里指明一个合适的外键：

{% highlight ruby %}
class CreateOrders < ActiveRecord::Migration
  def change
    create_table :orders do |t|
      t.datetime :order_date
      t.string   :order_number
      t.integer  :customer_id
    end
  end
end
{% endhighlight %}

如果哪天你建立了某个模型之后，你创建了一个关联性，那么你需要记得去创建一个add_column的数据迁移(migration)，用来提供所需要的外键。

#### 3.3.2 对于has_and_belongs_to_many关系创建联合(join)数据表

如果你创建了一个has_and_belongs_to_many关系，你需要明确地创建一个联合数据表。除非联合数据表的名称是被:join_table选项特别指定的，Active Record使用类名的字典式排序来创建名称.因此在customer和order models之间的一个联合将会得到默认的联合表名"customers_orders"，因为在字典排序上“c”要高于“o”。

>warning
>
>在model名称之间的优先权是通过使用String的\<操作符来计算出来的。这意味着，如果字符串是不同的长度，当字符串的最短长度进行比较时是相同的，那么比较长的字符串是被认为在字典序列上要高于较短的字符串。例如，某人希望数据表“paper_boxes”和“papers”会生成一个联合数据表，其名字为“papers_paper_boxes”，因为“paper_boxes”名字的长度，但实际上生成的联合数据表名字是“paper_boxes_papers”（因为下划线“_”在辞典编纂的通用编码里是小于‘s’的）。

无论名字怎样，你必须在一个合适的数据迁移文件（migration）里手动生成联合数据表。例如，考虑下面的关联性：

{% highlight ruby %}
class Assembly < ActiveRecord::Base
  has_and_belongs_to_many :parts
end
 
class Part < ActiveRecord::Base
  has_and_belongs_to_many :assemblies
end
{% endhighlight %}

这些需要回到一个数据迁移文件（migration）里来创建一个assemblies_parts数据表。这个数据表应该在创建的时候没有主键：

{% highlight ruby %}
class CreateAssembliesPartsJoinTable < ActiveRecord::Migration
  def change
    create_table :assemblies_parts, id: false do |t|
      t.integer :assembly_id
      t.integer :part_id
    end
  end
end
{% endhighlight %}

我们把id: false传入create_table的操作，因为这个数据表没有映射到一个model。那是必须的，为了让这关联性能正确地运行起来。如果你在has_and_belongs_to_many关系里观察到奇怪的行为，类似于砍掉了models IDs，或者IDs冲突异常，也有可能是你忘记写了。

### 3.4 控制关联性的范围

默认的情况下，关联性只会在当前模块的范围内查找对象。当你在一个模块里声明Active Record models时，这是很重要的。例如：

{% highlight ruby %}
module MyApplication
  module Business
    class Supplier < ActiveRecord::Base
       has_one :account
    end
 
    class Account < ActiveRecord::Base
       belongs_to :supplier
    end
  end
end
{% endhighlight %}

这个能很好地运行，因为Supplier和Account类都被定义在同一个范围内。但是下面的*将不会*执行，因为Supplier和Account被定义在不同的范围内：

{% highlight ruby %}
module MyApplication
  module Business
    class Supplier < ActiveRecord::Base
       has_one :account
    end
  end
 
  module Billing
    class Account < ActiveRecord::Base
       belongs_to :supplier
    end
  end
end
{% endhighlight %}

为了在不同的命名空间内关联一个model，你必须在关联声明里指定完整的类名：

{% highlight ruby %}
module MyApplication
  module Business
    class Supplier < ActiveRecord::Base
       has_one :account,
        class_name: "MyApplication::Billing::Account"
    end
  end
 
  module Billing
    class Account < ActiveRecord::Base
       belongs_to :supplier,
        class_name: "MyApplication::Business::Supplier"
    end
  end
end
{% endhighlight %}

### 3.5 双向关联

为了关联能在双向执行这是很正常的，需要声明两个不同的models：

{% highlight ruby %}
class Customer < ActiveRecord::Base
  has_many :orders
end
 
class Order < ActiveRecord::Base
  belongs_to :customer
end
{% endhighlight %}

默认情况下，Active Record不清楚关联性之间的联系。这会导致一个对象的两个拷贝，避免这个对象发生同步：

{% highlight ruby %}
c = Customer.first
o = c.orders.first
c.first_name == o.customer.first_name # => true
c.first_name = 'Manny'
c.first_name == o.customer.first_name # => false
{% endhighlight %}

这种情况的发生是因为c和o.customer是在内存里对于同一个数据有不同的表示，一个既不会从另一个改变中而导致其自动刷新。Active Record提供了:inverse_of选项，因此你可以从这些关系中获知：

{% highlight ruby %}
class Customer < ActiveRecord::Base
  has_many :orders, inverse_of: :customer
end
 
class Order < ActiveRecord::Base
  belongs_to :customer, inverse_of: :orders
end
{% endhighlight %}

随着这些改变，Active Record将会只导入customer对象的一个拷贝，防止不一致，使得你的应用变得更有效：

{% highlight ruby %}
c = Customer.first
o = c.orders.first
c.first_name == o.customer.first_name # => true
c.first_name = 'Manny'
c.first_name == o.customer.first_name # => true
{% endhighlight %}

对于inverse_of的支持有一些限制：

* 他们不能和 :through 关联共存
* 他们不能和 :polymorphic 关联共存
* 他们不能和 :as 关联共存
* 对于belongs_to关联，has_many的反向的(inverse)关联是被忽视的

每一个关联都试图去自动寻找反向的(inverse)关联，试探性地设置 :inverse_of选项（基于关联的名称）。大部分关联都支持标准命名。然而，包含下面选项的关联将不会自动设置他们的反向关联：


* :conditions
* :through
* :polymorphic
* :foreign_key