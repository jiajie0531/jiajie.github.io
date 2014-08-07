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

一个has_many关联指的是与另一个模型one-to-many的关系。你将会发现这个关联是belongs_to关联的"other side"。这个关联指的是模型的每一个实例有零个或者多个模型的实例。例如，在一个应用程序里包含customers和orders，customer模型能够这样来声明：

{% highlight ruby %}
class Customer < ActiveRecord::Base
  has_many :orders
end
{% endhighlight %}

>note:
>
>当声明一个has_many关联时，其他模型的命名是复数形式。

![has_many](/assets/2014/08/06/has_many.png)

所对应的migration可能是这个样子：

{% highlight ruby %}
class CreateCustomers < ActiveRecord::Migration
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

### 2.4 has_many :through关联

一个has_many :through关联是经常用来与另一个模型建立一个many-to-many关系。这个关联指的是声明模型能够被零个或多个模型实例来匹配，通过处理*through*一个第三方的实例。例如，考虑到一个看病的实际情况，patients被指定到physicians那里看病。涉及到的关联声明可以是这个样子的：

{% highlight ruby %}
class Physician < ActiveRecord::Base
  has_many :appointments
  has_many :patients, through: :appointments
end
 
class Appointment < ActiveRecord::Base
  belongs_to :physician
  belongs_to :patient
end
 
class Patient < ActiveRecord::Base
  has_many :appointments
  has_many :physicians, through: :appointments
end
{% endhighlight %}

![has_many_through](/assets/2014/08/06/has_many_through.png)

所对应的migration可能是这个样子：

{% highlight ruby %}
class CreateAppointments < ActiveRecord::Migration
  def change
    create_table :physicians do |t|
      t.string :name
      t.timestamps
    end
 
    create_table :patients do |t|
      t.string :name
      t.timestamps
    end
 
    create_table :appointments do |t|
      t.belongs_to :physician
      t.belongs_to :patient
      t.datetime :appointment_date
      t.timestamps
    end
  end
end
{% endhighlight %}

join模型的集合能够被API来管理。例如，如果你这样做：

{% highlight ruby %}
physician.patients = patients
{% endhighlight %}

新的联合模型被新关联的对象所创建，如果一些已经失去了，他们的行数据被删除了。

>warning
>
>联合模型的自动删除是直接的，没有销毁的回调函数会触发。

has_many :through 关联也是很有用处的，可以通过内嵌has_many关联来建立“shortcuts”。例如，如果一个文档有多个章节，每个章节有多个段落，有时候你可以想要文档里所有段落的一个简单的集合。你可以这样来建立：

{% highlight ruby %}
class Document < ActiveRecord::Base
  has_many :sections
  has_many :paragraphs, through: :sections
end
 
class Section < ActiveRecord::Base
  belongs_to :document
  has_many :paragraphs
end
 
class Paragraph < ActiveRecord::Base
  belongs_to :section
end
{% endhighlight %}

声明through: :sections，Rails会这样理解：

{% highlight ruby %}
@document.paragraphs
{% endhighlight %}

### 2.5 has_one :through关联

一个has_one :through 关联与另一个模型建立一个one-to-one。这个关联指的是，声明模型能够被对应另一个模型的一个实例，通过关联through一个第三方的模型。例如，如果每一个supplier有一个account，每一个account被关联到一个account的历史，那么supplier模型能够这个样子：

{% highlight ruby %}
class Supplier < ActiveRecord::Base
  has_one :account
  has_one :account_history, through: :account
end
 
class Account < ActiveRecord::Base
  belongs_to :supplier
  has_one :account_history
end
 
class AccountHistory < ActiveRecord::Base
  belongs_to :account
end
{% endhighlight %}

![has_one_through](/assets/2014/08/06/has_one_through.png)

所对应的migration可能是这个样子：

{% highlight ruby %}
class CreateAccountHistories < ActiveRecord::Migration
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
 
    create_table :account_histories do |t|
      t.belongs_to :account
      t.integer :credit_rating
      t.timestamps
    end
  end
end
{% endhighlight %}

