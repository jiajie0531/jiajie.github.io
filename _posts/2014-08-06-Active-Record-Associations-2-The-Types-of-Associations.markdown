---
layout: post
title:  "Active Record Associations 2 关联性的类型"
date:   2014-08-06 17:12:55
categories: Rails
tags: Rails Active Record Associations
---

在Rails里，关联性是两个Active Record模型之间的一个连接。关联性是使用微类型(macro-style)调用来实现的，因此你可以直接在你的模型中增加属性。例如，通过声明belongs_to来关联两个models，你可以通过Rails来维护两个模型实例之间的主键-外键信息，你也会得到一些实用方法，已被增加到你的模型中。Rails提供了六种类型的关联：

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

### 2.6 has_and_belongs_to_many 关联

一个has_and_belongs_to_many 关联创建了一个与另一个model的直接many_to_many关系，没有中间串联的model。例如，如果你的应用系统包括assemlies和parts，每一个assembly拥有多个parts，每一个part含有多个assemblies，你可以这样来声明这些models：

{% highlight ruby %}
class Assembly < ActiveRecord::Base
  has_and_belongs_to_many :parts
end
 
class Part < ActiveRecord::Base
  has_and_belongs_to_many :assemblies
end
{% endhighlight %}

![habtm](/assets/2014/08/06/habtm.png)

所对应的migration可能是这个样子：

{% highlight ruby %}
class CreateAssembliesAndParts < ActiveRecord::Migration
  def change
    create_table :assemblies do |t|
      t.string :name
      t.timestamps
    end
 
    create_table :parts do |t|
      t.string :part_number
      t.timestamps
    end
 
    create_table :assemblies_parts, id: false do |t|
      t.belongs_to :assembly
      t.belongs_to :part
    end
  end
end
{% endhighlight %}

### 2.7 选择belongs_to和has_one

如果你想要在两个model之间建立one-to-one的关系，你将需要给其中一个增加belongs_to，给另外一个增加has_one。你如何来知道哪个是哪个呢？

区别在于你所设置的外键（它是作用于数据表上，为class声明belongs_to关联性），但你也应该对于一些实际的数据有一些想法。has_one关系意思是一些东西中的某一个是属于你的----也就是说，那些东西还是会指向到你。例如，一个supplier拥有一个account，一个account属于一个supplier。建议比较正确的关联性声明方式如下：

{% highlight ruby %}
class Supplier < ActiveRecord::Base
  has_one :account
end
 
class Account < ActiveRecord::Base
  belongs_to :supplier
end
{% endhighlight %}

所对应的migration可能是这个样子：

{% highlight ruby %}
class CreateSuppliers < ActiveRecord::Migration
  def change
    create_table :suppliers do |t|
      t.string  :name
      t.timestamps
    end
 
    create_table :accounts do |t|
      t.integer :supplier_id
      t.string  :account_number
      t.timestamps
    end
  end
end
{% endhighlight %}

> note:
>
> 使用t.integer :supplier_id使得外键命名直白又明确。在当前的Rails版本中，你可以抽象提炼这个实现细节，通过使用t.references :supplier来代替。

### 2.8 选择has_many :through和has_and_belongs_to_many

Rails提供了两种不同的方式来声明models之间的一个many-to-many关系。比较简单的方式是使用has_and_belongs_to_many，这个能允许你直接搞定这个关联：

{% highlight ruby %}
class Assembly < ActiveRecord::Base
  has_and_belongs_to_many :parts
end
 
class Part < ActiveRecord::Base
  has_and_belongs_to_many :assemblies
end
{% endhighlight %}

第二个方式来声明一个many-to-many关系是使用has_many :through。这使得关联性变得间接，通过一个关联model来实现：

{% highlight ruby %}
class Assembly < ActiveRecord::Base
  has_many :manifests
  has_many :parts, through: :manifests
end
 
class Manifest < ActiveRecord::Base
  belongs_to :assembly
  belongs_to :part
end
 
class Part < ActiveRecord::Base
  has_many :manifests
  has_many :assemblies, through: :manifests
end
{% endhighlight %}

如果你需要与关联的model产生联系作为一个独立的entity，最简单的规则就是你应该建立一个has_many :through关系。如果你不需要与所关联的model产生联系，比较简单的做法就是建立一个has_and_belongs_to_many关系（虽然你将需要去记得要在数据库里创建关联的数据表）。

如果你需要对于所join model做校验，回调，或者额外的属性，你应该使用has_many :through。

### 2.9 多态关联性

稍微再进一步探寻关联性，会是*多态*关联。有了多态关联性，在单一的关联上，一个model可以属于多个其他的model。例如，你可能有一个picture的model，其属于一个employee的model或者一个product的model。这里就是怎样来声明：

{% highlight ruby %}
class Picture < ActiveRecord::Base
  belongs_to :imageable, polymorphic: true
end
 
class Employee < ActiveRecord::Base
  has_many :pictures, as: :imageable
end
 
class Product < ActiveRecord::Base
  has_many :pictures, as: :imageable
end
{% endhighlight %}

你可以考虑吧一个多态belongs_to声明作为建立一个接口，可以让其他的model来使用。从一个Employee model的实例中，你可以获取一个pictures的集合：@employee.pictures。

相似地，你可以获取@product.pictures。

如果你有一个Picture model的实例，你可以得到他的父类通过@picture.imageable。为了执行这个，你需要在model里声明一个外键和一个类型的数据列，来声明多态的接口：

{% highlight ruby %}
class CreatePictures < ActiveRecord::Migration
  def change
    create_table :pictures do |t|
      t.string  :name
      t.integer :imageable_id
      t.string  :imageable_type
      t.timestamps
    end
  end
end
{% endhighlight %}

这个migration可以通过使用t.references来简化：

{% highlight ruby %}
class CreatePictures < ActiveRecord::Migration
  def change
    create_table :pictures do |t|
      t.string :name
      t.references :imageable, polymorphic: true
      t.timestamps
    end
  end
end
{% endhighlight %}

![polymorphic](/assets/2014/08/06/polymorphic.png)

### 2.10 Self Joins

设计一个数据模型，有时候你将会找到一个model，它只对于他本身有一个关联。例如，你可能想要保存所有雇员信息到一单个的数据库模型里，但是能够被跟踪其联系，就类似于manager和subordinates。这种情况能够用self-joining关联性来被模型化：

{% highlight ruby %}
class Employee < ActiveRecord::Base
  has_many :subordinates, class_name: "Employee",
                          foreign_key: "manager_id"
 
  belongs_to :manager, class_name: "Employee"
end
{% endhighlight %}

搞定这个步骤，你可以取得@employee.subordinates和@employee.manager。

在你的migration/schema，你将会在其model本身里增加一个references数据列。

{% highlight ruby %}
class CreateEmployees < ActiveRecord::Migration
  def change
    create_table :employees do |t|
      t.references :manager
      t.timestamps
    end
  end
end
{% endhighlight %}
