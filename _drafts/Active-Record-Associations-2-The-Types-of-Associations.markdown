---
layout: post
title:  "Active Record Associations 2 关联性的类型"
date:   2014-08-05 23:12:55
categories: Rails
tags: Rails Active Record Associations
---

在Rails里，关联性是两个Active Record模型之间的一个连接。关联性是使用微类型调用来实现的，因此你可以直接在你的模型中增加属性。例如，通过声明一个模型belongs_to另一个，你可以通过Rails来维护两个模型实例之间的主键-外键信息，你也会得到一些实用方法，已被增加到你的模型中。Rails提供了六种类型的关联：

* belongs_to
* has_one
* has_many
* has_many :through
* has_one :through
* has_and_belongs_to_many

在接下来的教程里，你将会学习到如何去声明和使用各个关联性的特点。但首先，一个快速的介绍每一个关联类型适合的使用条件。

### 2.1 belongs_to关联

一个belongs_to关联是用来one_to_one联系另一个模型，因此每一个声明模型的实例是"belongs to"另一个模型的实例。例如，如果你的应用程序包括customers和orders，每一个order能够被归属到确切的一个customer，你可以这样子来声明order模型：

{% highlight ruby%}
class Order < ActiveRecord::Base
  belongs_to :customer
end
{% endhighlight %}

![belongs_to](/assets/2014/08/06/belongs_to.png)

>note:
>
>belongs_to关联必须使用单数的对象。如果在上面的例子里你使用了复数形式，对于在Order模型里关联了customer，你将会被告知，这里有一个“uninitialized constant Order::Customers”。这是因为Rails自动从关联的名称来推断类名。如果关联名称是错误地复数形式，那么推断出来的类也将是错误的复数形式。

所对应的migration可能是这个样子：

{% highlight ruby %}
class CreateOrders < ActiveRecord::Migration
  def change
    create_table :customers do |t|
      t.string :name
      t.timestamps
    end
 
    create_table :orders do |t|
      t.belongs_to :customer
      t.datetime :order_date
      t.timestamps
    end
  end
end
{% endhighlight %}

### 2.2 has_one关联

一个has_one关联同样是建立在与另一个模型的one-to-one联系，但有一些语义上面的不同(结论)。这个关联性隐含的是一个模型的每个实例包含或者处理另一个模型的实例。例如，如果在你应用程序里每一个supplier只有一个account，那么你将会是这个样子来声明supplier的：

{% highlight ruby %}
class Supplier < ActiveRecord::Base
  has_one :account
end
{% endhighlight %}

![has_one](/assets/2014/08/06/has_one.png)

所对应的migration可能是这个样子：

{% highlight ruby %}
class CreateSuppliers < ActiveRecord::Migration
  def change
    create_table :suppliers do |t|
      t.string :name
      t.timestamps
    end
 
    create_table :accounts do |t|
      t.belongs_to :supplier
      t.string :account_number
      t.timestamps
    end
  end
end
{% endhighlight %}

### 2.3 has_many关联

一个has_many关联指的是与另一个模型one-to-many的关系。你将会发现这个关联是belongs_to关联的"other side"。这个关联指的是模型的每一个实例


