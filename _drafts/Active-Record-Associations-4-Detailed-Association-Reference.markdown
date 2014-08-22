---
layout: post
title:  "Active Record Associations 4 关联性的细节"
date:   2014-08-13 15:55:55
categories: Rails
tags: Rails Active Record Associations
---

![P1000804](/assets/2014/08/13/P1000804.gif)

下面的部分会给出每一个关联类型的细节，包括他们增加的方法和你可以用来声明关系的操作。

## 4.1 belongs_to关联

belongs_to关联创建了一个one-to-one映射到另一个model。在数据库里，这个关联就是说这个类包含了一个外键。如果另外一个类包含了外键，那么你应该使用has_one来替代。

### 4.1.1 belongs_to增加的方法

当你声明一个belongs_to关联时，声明的类自动获取五个相关的方法：

* association(force_reload = fasle)
* association=(associate)
* build_association(attributes = {})
* create_association(attributes = {})
* create_association!(attributes = {})

在所有的这些方法里，对于belongs_to，association被传入的第一个参数，即symbol所代替。例如，下面的声明：

{% highlight ruby %}
class Order < ActiveRecord::Base
  belongs_to :customer
end
{% endhighlight %}

该order model的每一个实例将会有这些方法：

{% highlight ruby %}
customer
customer=
build_customer
create_customer
create_customer!
{% endhighlight %}

>note
>
>当初始化一个新的has_one或者belongs_to关联时，你必须使用build_prefix来建立这个关联，而不是用association.build方法，association.build是用在has_many或者has_and_belongs_to_many关联的。为了创建一个，可以使用create_prefix。

#### 4.1.1.1 association(force_reload = false)

这个association方法如果有任意的被关联对象被发现，则返回的是一个已被关联的对象。如果没有被关联对象被发现，它返回的是nil。

{% highlight ruby %}
@customer = @order.customer
{% endhighlight %}

如果在这个对象中从数据库里以及得到了一个被关联的对象，其被缓存的版本将会被返回。为了重写这个行为（强迫数据库可查询），传入true作为force_rolad的参数。

#### 4.1.1.2 association=(associate)

association=方法赋值了一个被关联的对象。在这个背后，也就是说从关联对象中抓取主键，然后该这个对象的外键设置为相同的值。

{% highlight ruby %}
@order.customer = @customer
{% endhighlight %}

#### 4.1.1.3 build_association(attributes = {})

build_association方法返回一个新的被关联类型对象。这个对象将会从已传入的参数里实例化，通过这个对象的外键关联也会被设置，但被关联的对象将不会被保存。

{% highlight ruby %}
@customer = @order.build_customer(customer_number: 123,
                                  customer_name: "John Doe")
{% endhighlight %}

#### 4.1.1.4 create_association(attributes = {})

create_association方法返回一个新的被关联类型对象。这个对象将会从已传入的参数里实例化，通过这个对象的外键关联也会被设置，一旦它传入的所有校验被指定到被关联的model，被关联的对象将会被保存。

{% highlight ruby %}
@customer = @order.create_customer(customer_number: 123,
                                   customer_name: "John Doe")
{% endhighlight %}

#### 4.1.1.5 create_association!(attributes = {})

和上面的create_association方法是一样的，但是如果记录是无效的话,会抛出ActiveRecord::RecordInvalid。

### 4.1.2 belongs_to选项

当Rails使用智能化的默认，在大部分情况下也能完成工作，当你想要定制化belongs_to关联的行为时，可能就需要花费一些时间。当你创建关联时，一般性的定制化通过传入选项和部分代码块能够很容易地完成。例如，这个关联使用了这两个选项：

{% highlight ruby %}
class Order < ActiveRecord::Base
  belongs_to :customer, dependent: :destroy,
    counter_cache: true
end
{% endhighlight %}

belongs_to关联提供了这些选项：

* :autosave
* :class_name
* :counter_cache
* :dependent
* :foreign_key
* :inverse_of
* :polymorphic
* :touch
* :validate

#### 4.1.2.1 :autosave

如果你把:autosave选项设置为true，无论何时你在保存父类对象时，Rails将会保存所导入的部分，以及销毁的部分会被标记为已毁灭。

#### 4.1.2.2 :class_name


如果另一个model名字不能够从关联命名那里推导出来，你可以使用:class_name选项来替代mdel名字。例如，如果一个order属于一个customer，但实际包含customers的model名字是Parton，那么你可以这样来设置：

{% highlight ruby %}
class Order < ActiveRecord::Base
  belongs_to :customer, class_name: "Patron"
end
{% endhighlight %}

#### 4.1.2.3 :counter_cache

:counter_cache选项能够被用来更有效率地查找所属对象的计数。考虑一下这些mdels：

{% highlight ruby %}
class Order < ActiveRecord::Base
  belongs_to :customer
end
class Customer < ActiveRecord::Base
  has_many :orders
end
{% endhighlight %}

基于这些声明，找到@customer.orders.size的值，要求能够调用数据库来执行count(*)的查询。为了避免这个调用，你可以增加一个计数器缓存到*所属(belonging)*的model中：

{% highlight ruby %}
class Order < ActiveRecord::Base
  belongs_to :customer, counter_cache: true
end
class Customer < ActiveRecord::Base
  has_many :orders
end
{% endhighlight %}

有了这个声明，Rails将保持缓存一直是最新的，然后返回的那个值是size方法。

虽然:counter_cache选项被指定到这个model上，还包含了belongs_to声明，实际的数据列必须被增加到*被关联的(associated)*model。在上面的例子里，你需要在Customer的model里增加一列orders_count。如果你需要，你可以重写默认的列名：

{% highlight ruby %}
class Order < ActiveRecord::Base
  belongs_to :customer, counter_cache: :count_of_orders
end
class Customer < ActiveRecord::Base
  has_many :orders
end
{% endhighlight %}

计数缓存的列通过attr_readonly被增加到包含的model的只读属性列表里。

#### 4.1.2.4 :dependent

如果你设置了:dependent选项了：

* :destroy，当对象被销毁的时候，destroy将会被它的关联对象所调用。
* :delete，当对象被销毁的时候，它所关联的所有对象都会从数据里被直接删除，而不会调用它们的destroy方法。

>warning
>
>你不应该在一个belongs_to关联上使用这个选项，它是和另一个类的has_many关联有联系的。这样做会导致在你数据库里孤立其记录行的。

#### 4.1.2.5 :foreign_key

基于约定，Rails假设在其他model里用来作为外键的数据列是这个model名字加上后缀_id。:foreign_key选项是让你直接设置外键名称：

{% highlight ruby %}
class Order < ActiveRecord::Base
  belongs_to :customer, class_name: "Patron",
                        foreign_key: "patron_id"
end
{% endhighlight %}

>information
>
>在任何情况下，Rails不会为你创建外键的列。你需要在你的数据迁移文件（migration）里明确定义他们。

#### 4.1.2.6 :inverse_of

:inverse_of选项指的是has_many或者has_one关联性名称的互逆性。不要结合:polymorphic选项一起使用。

{% highlight ruby %}
class Customer < ActiveRecord::Base
  has_many :orders, inverse_of: :customer
end
 
class Order < ActiveRecord::Base
  belongs_to :customer, inverse_of: :orders
end
{% endhighlight %}

#### 4.1.2.7 :polymorphic

在:polymorphic选项中设置为true，指的是这是一个多态性的关联。关于多态性关联更多的细节在[earlier in this guide](http://guides.rubyonrails.org/association_basics.html#polymorphic-associations)

#### 4.1.2.8 :touch

如果你设置:touch选项的值为:true，那么在关联的对象里updated_at或者updated_on的时间戳会被设置为当前的时间，无论是这个对象被保存时还是被销毁时：

{% highlight ruby %}
class Order < ActiveRecord::Base
  belongs_to :customer, touch: true
end
 
class Customer < ActiveRecord::Base
  has_many :orders
end
{% endhighlight %}

在这个例子里，保存或销毁order将会更新关联的customer的时间戳。你也可以指定一个特定的时间戳属性来更新：

{% highlight ruby %}

class Order < ActiveRecord::Base
  belongs_to :customer, touch: :orders_updated_at
end
{% endhighlight %}

#### 4.1.2.9 :validate

如果你设置:validate选项的值为true，那么当你保存关联的对象时，它会被校验。默认情况下，是为:false,当这个关联对象被保存时，它不会被校验。

### 4.1.3 belongs_to的范围

当你希望通过belongs_to来定制化查询，可能需要一些时间。类似于下面的例子通过一段代码块来定制化获取数据：

{% highlight ruby %}
class Order < ActiveRecord::Base
  belongs_to :customer, -> { where active: true },
                        dependent: :destroy
end
{% endhighlight %}

你可以在代码块内使用一些的标准的查询方法[querying methods](http://guides.rubyonrails.org/active_record_querying.html)，将在下面的内容里讨论这些：

* where
* includes
* readonly
* select

#### 4.1.3.1 where

where方法让你指定条件，所取得的关联对象必须要满足的条件。

{% highlight ruby %}
class Order < ActiveRecord::Base
  belongs_to :customer, -> { where active: true }
end
{% endhighlight %}

#### 4.1.3.2 includes

你可以使用includes方法来指定第二顺位的(second-order)关联，当这个关联被使用时而被及时加载(eager-loaded)。例如，考虑一下这些方法：

{% highlight ruby %}
class LineItem < ActiveRecord::Base
  belongs_to :order
end
 
class Order < ActiveRecord::Base
  belongs_to :customer
  has_many :line_items
end
 
class Customer < ActiveRecord::Base
  has_many :orders
end
{% endhighlight %}

如果你经常性地从line items中直接获取customers(@line_item.order.customer)，那么你可以使用includes :customer使得你的代码更有效率，从line items到orders：

{% highlight ruby %}
class LineItem < ActiveRecord::Base
  belongs_to :order, -> { includes :customer }
end
 
class Order < ActiveRecord::Base
  belongs_to :customer
  has_many :line_items
end
 
class Customer < ActiveRecord::Base
  has_many :orders
end
{% endhighlight %}

>note
>
>对于直接的关联不是必须得用includes----也就是说，如果你让Order belongs_to :customer，那么当customer需要的时候，它会自动地及时加载（eager-loaded）。

#### 4.1.3.3 readonly

如果你使用readonly，那么当通过关联被接收时，关联对象将会是只读的（read-only）。

#### 4.1.3.4 select

select方法可以让你重写SQL SELECT语句，被用来获得关于关联对象的数据。默认情况下，Rails获得是所有列。

>information
>
>如果你在一个belongs_to关联上使用select方法，你也应该可以设置:foreign_key选项来保证正确的结果。

### 4.1.4 已关联对象是否存在？

你可以通过使用association.nil?方法看到已关联的对象是否存在：

{% highlight ruby %}
if @order.customer.nil?
  @msg = "No customer found for this order"
end
{% endhighlight %}

### 4.1.5 对象何时被保存？

把一个对象赋值给一个belongs_to的关联上是不会自动保存该对象的。也不会保存被关联的对象。

## 4.2 has_one关联

has_one关联创建了一个one-to-one匹配另一个model。在数据库里，这个关联意味着其他类包含其外键。如果这个类包含了外键，那么你应该使用belongs_to来替代。

### 4.2.1 has_one所新增的方法

当你声明一个has_one关联时，对于这个关联声明的类自动获取五个相关的方法：

* association(force_reload = false)
* association=(associate)
* build_association(attributes = {})
* create_association(attributes = {})
* create_association!(attributes = {})

在所有的这些方法里，has_one关联后面跟着symbol作为第一个参数。例如，可以这样声明：

{% highlight ruby %}
class Supplier < ActiveRecord::Base
  has_one :account
end
{% endhighlight %}

每一个Supplier model的实例将会有这些方法：

{% highlight ruby %}
account
account=
build_account
create_account
create_account!
{% endhighlight %}

>note
>
>当初始化一个新的has_one或者belongs_to关联时，你必须使用build_前缀构建这个关联，而不是association.build方法，association.build是被用在has_many或者has_and_belongs_to_many关联上。为了创建一个，可以使用create_prefix。

#### 4.2.1.1 association(force_reload = false)

如果一般情况下，association方法返回一个已关联的对象。如果没有发现关联的对象，返回nil。

{% highlight ruby %}
@account = @supplier.account
{% endhighlight %}

如果已关联对象已经从数据库里获取，其缓存的版本会被返回。为了重写这个行为（针对数据库的读取），代入true作为force_reload参数的值。

#### 4.2.1.2 association=(associate)

association=方法是给这个对象赋值一个关联对象。在本质上，这意味着从这个对象里抓取主键，设置关联对象的外键为相同的值。

{% highlight ruby %}
@supplier.account = @account
{% endhighlight %}

#### 4.2.1.3 build_association(attributes = {})

build_association方法返回一个新关联类型的对象。这个对象将会从传入的属性里被初始化，设置联系是通过它的外键，但是已关联的对象将不会被保存。

{% highlight ruby %}
@account = @supplier.build_account(terms: "Net 30")
{% endhighlight %}

#### 4.2.1.4 create_association(attributes = {})

create_association方法返回一个新关联类型的对象。这个对象将会从传入的属性里被初始化，设置联系是通过它的外键，一旦它传入所有的针对关联model的指定校验，已关联对象将会被保存。

{% highlight ruby %}
@account = @supplier.create_account(terms: "Net 30")
{% endhighlight %}

#### create_association!(attributes = {})

和上面的create_association是一样的，但如果数据记录无效的话，会抛出ActiveRecord::RecordInvalid。

### 4.2.2 has_one的选项

当Rails使用智能化的默认值，在大部分情况下将可以很好地执行，可能有些时候，当你想要定制化has_one关联的行为需要用到选项。类似于这样的定制化可以简单地通过传入选项来完成，在你创建关联的时候。例如，这个关联用到两个这样的选项：

{% highlight ruby %}
class Supplier < ActiveRecord::Base
  has_one :account, class_name: "Billing", dependent: :nullify
end
{% endhighlight %}

has_one关联提供这些选项：

* :as
* :autosave
* :class_name
* :dependent
* :foreign_key
* :inverse_of
* :primary_key
* :source
* :source_type
* :through
* :validate

#### 4.2.2.1 :as

设置:as选项表明这是一个多态关联。关于多态关联的细节在于[earlier in this guide](http://guides.rubyonrails.org/association_basics.html#polymorphic-associations)。

#### 4.2.2.2 :autosave

如果你把:autosave选项设置为true，只要在你保存父类对象时，Rails将会保存任何载入的事物和销毁已被标记为可删除的事物。

#### 4.2.2.3 :class_name

如果其他model的名字不能够从关联名称推导出来的话，你可以使用:class_name选项来支持model名字。例如，如果一个supplier有一个account，但包含accounts的model其实际名字是Billing，你将会这样来建立：

{% highlight ruby %}
class Supplier < ActiveRecord::Base
  has_one :account, class_name: "Billing"
end
{% endhighlight %}

#### 4.2.2.4 :dependent

当关联对象被销毁时，可以控制它所发生的情况是什么：

* :destroy引起已关联对象被销毁
* :delete引起已关联对象被直接从数据库里删除（因此回调函数将不会执行）
* :nullify引起外键被设置为NULL。回调函数不会被执行。
* :restrict_with_exception如果有一个已关联的记录的话，会引起一个异常被抛出
* :restrict_with_error如果有一个已关联对象，会引起一个错误被增加到owner

为了那些数据库的限制条件为NOT NULL的关联性不需要去设置或者委托:nullify选项。如果你不去设置依赖来销毁这样的关联性，你将不能够来改变其关联的对象，因为初始化的关联对象外键将被设置为不允许的NULL值。

#### 4.2.2.5 :foreign_key

基于约定，Rails假设在其他model里用来作为外键的数据列是这个model名字加上后缀_id。:foreign_key选项是让你直接设置外键名称：

{% highlight ruby %}
class Supplier < ActiveRecord::Base
  has_one :account, foreign_key: "supp_id"
end
{% endhighlight %}

>information
>
>在任何情况下，Rails不会为你创建外键的列。你需要显示地在你的数据迁移文件里(migration)定义它们

#### 4.2.2.6 :inverse_of

:inverse_of选项指定的是belongs_to关联的名字，它是这个关联的反向。不要结合:through或者:as选项一起使用。

{% highlight ruby %}
class Supplier < ActiveRecord::Base
  has_one :account, inverse_of: :supplier
end
 
class Account < ActiveRecord::Base
  belongs_to :supplier, inverse_of: :account
end
{% endhighlight %}

#### 4.2.2.7 :primary_key

按照约定，Rails假设一个列被用来作为model的主键是id。你可以重写它，显示地声明主键，用:primary_key选项。

#### 4.2.2.8 :source

:source选项指定的是source关联名通过一个has_one :through关联。

#### 4.2.2.9 :source_type

:source_type选项指的是source关联类型用来一个has_one :through关联，用来处理一个多态的关联。

#### 4.2.2.10 :through

:through选项指的是一个join model通过哪个model来执行查询。has_one :through关联细节已经在前面讲过[earlier in this guide](http://guides.rubyonrails.org/association_basics.html#the-has-one-through-association)。

#### 4.2.2.11 :validate

如果你设置:validate选项值为true，那么被关联的对象无论你何时保存它都将会被验证。默认的情况下，这个选项的值是false，当这个对象被保存的时候将不会校验。

### 4.2.3 has_one的范围

当你想要用has_one定制化查询时，可能需要一些时间。类似于这样的定制化能够通过一个代码块来获取。例如：

{% highlight ruby %}
class Supplier < ActiveRecord::Base
  has_one :account, -> { where active: true }
end
{% endhighlight %}

你可以在代码块里使用标准的[查询方法](http://guides.rubyonrails.org/active_record_querying.html)。接下来下面这些将会被讨论：

* where
* includes
* readonly
* select

#### 4.2.3.1 where

where方法让你指定条件，所取得的关联对象必须要满足的条件。

{% highlight ruby %}
class Supplier < ActiveRecord::Base
  has_one :account, -> { where "confirmed = 1" }
end
{% endhighlight %}

#### 4.2.3.2 includes

你可以使用includes方法来指定second-order关联，当这个关联被使用时，它会被及时导入。例如，考虑这些models：

{% highlight ruby %}
class Supplier < ActiveRecord::Base
  has_one :account
end
 
class Account < ActiveRecord::Base
  belongs_to :supplier
  belongs_to :representative
end
 
class Representative < ActiveRecord::Base
  has_many :accounts
end
{% endhighlight %}

如果你经常性地从suppliers直接获取representatives(@supplier.account.representative)，那么你可以使你的代码变得更有效率，通过在关联中从suupliers到accounts里来包括representatives：

{% highlight ruby %}
class Supplier < ActiveRecord::Base
  has_one :account, -> { includes :representative }
end
 
class Account < ActiveRecord::Base
  belongs_to :supplier
  belongs_to :representative
end
 
class Representative < ActiveRecord::Base
  has_many :accounts
end
{% endhighlight %}

#### 4.2.3.3 readonly

如果你使用readonly方法，那么当通过关联性来获取的被关联的对象将会是read-only。

#### 4.2.3.4 select

select方法让你重写SQL SELECT语句，用来获取关于关联对象的数据。默认情况下，Rails获取所有的列。

### 4.2.4 Do Any Associated Objects Exist?

你可以看到如果任何关联对象是否存在，可以通过使用association.nil?方法：

{% highlight ruby %}
if @supplier.account.nil?
  @msg = "No account found for this supplier"
end
{% endhighlight %}

### 4.2.5 对象何时被保存？

当你给一个has_one关联赋值一个对象时，对象是自动地被保存（为了更新它的外键）。另外，被替代的任何对象同样也会被自动地保存，因为它的外键也会被改变。

如果这些保存因为校验出错而失败了，那么赋值语句返回的是false，赋值本身会被取消。

如果父类对象（声明has_one的关联）不能被保存（就是new_record? 返回的是true），那么子对象不会被保存。当父类被保存时，他们也会被自动化地保存。

如果你想要给一个has_one关联性赋值一个对象，没有保存对象，通过使用association.build方法。

## 4.3 has_many关联性

has_many关联创建了与另一个model的一个one-to-many关系。在数据库方面，这个关联性意味着其他的类将会拥有一个外键来映射这个类的实例。

### has_many的方法

当你声明一个has_many的关联时，声明类自动地获得其相关联的16个方法：

* collection(force_reload = false)
* collection<<(object, ...)
* collection.delete(object, ...)
* collection.destroy(object, ...)
* collection=objects
* collection_singular_ids
* collection_singular_ids=ids
* collection.clear
* collection.empty?
* collection.size
* collection.find(...)
* collection.where(...)
* collection.exists?(...)
* collection.build(attributes = {}, ...)
* collection.create(attributes = {})
* collection.create!(attributes = {})

在所有的这些方法里，collection是被symbol所替代的，作为has_many的第一个参数，collection_singular是被symbol的单数版本所替代。例如，给予这样的声明：

{% highlight ruby %}
class Customer < ActiveRecord::Base
  has_many :orders
end
{% endhighlight %}

每一个customer model的实例都会有这些方法：

{% highlight ruby %}
orders(force_reload = false)
orders<<(object, ...)
orders.delete(object, ...)
orders.destroy(object, ...)
orders=objects
order_ids
order_ids=ids
orders.clear
orders.empty?
orders.size
orders.find(...)
orders.where(...)
orders.exists?(...)
orders.build(attributes = {}, ...)
orders.create(attributes = {})
orders.create!(attributes = {})
{% endhighlight %}

#### 4.3.1.1 collection(force_reload = false)

collection方法返回是一个关于所关联对象的数组。如果没有所关联的对象，它将返回一个空的数组。

{% highlight ruby %}
@orders = @customer.orders
{% endhighlight %}

#### 4.3.1.2 collection<<(object, ...)

collection<<方法增加一个或者多个对象到这个collection里，通过设置他们的外键关联到所调用model的主键上。

{% highlight ruby %}
@customer.orders << @order1
{% endhighlight %}

#### 4.3.1.3 collection.delete(object, ...)

collection.delete方法从这个collection里移除一个或者多个方法，通过设置他们的外键为NULL。

{% highlight ruby %}
@customer.orders.delete(@order1)
{% endhighlight %}

>warning
>
>另外，如果那些对象的关联是dependent: :destroy，那么他们将会被销毁，如果他们的关联是dependent: :delete_all，那么他么将会被删除。

#### 4.3.1.4 collection.destroy(object, ...)

collection.destroy方法从collection里移除一个或者多个对象，通过执行每一个对象的销毁。

{% highlight ruby %}
@customer.orders.destroy(@order1)
{% endhighlight %}

>warning
>
>对象将总是从数据库里被移除，那么应忽视:dependent选项。

#### 4.3.1.5 collection=objects

collection=方法使得collection只包含所提供的对象，用来新增和删除。

#### 4.3.1.6 collection_singular_ids

collection_singular_ids方法返回的是在collection里的一个对象ids的数组。

{% highlight ruby %}
@order_ids = @customer.order_ids
{% endhighlight %}

#### 4.3.1.7 collection_singular_ids=ids

collection_singular_ids=方法使得collection只包含所指定的对象，是通过所提供的主键值，用来新增和删除。

#### 4.3.1.8 collection.clear

collection.clear方法移除collection里每一个对象。如果他们的关联里有dependent: :destroy，那么这会销毁已关联的对象，如果是dependent: :delete_all，那么直接从数据库里删除他们，另外的就把它们的外键设置为NULL。

#### 4.3.1.9 collection.empty?

如果collection没有包含任何关联对象，collection.empty?方法返回的是true。

{% highlight ruby %}
<% if @customer.orders.empty? %>
  No Orders Found
<% end %>
{% endhighlight %}

#### 4.3.1.10 collection.size

collection.size方法返回的是在collection里对象的数量。

{% highlight ruby %}
@order_count = @customer.orders.size
{% endhighlight %}

#### 4.3.1.11 collection.find(...)

collection.find方法是在collection里查找objects。它用的是和ActiveRecord::Base.find相同的语法和选项。

{% highlight ruby %}
@open_orders = @customer.orders.find(1)
{% endhighlight %}

#### 4.3.1.12 collection.where(...)

collection.where方法是基于所提供的查询条件在collection里查找objects，但这些objects是延时导入，意味着数据库被查询只是在object(s)能被读取时。

{% highlight ruby %}
@open_orders = @customer.orders.where(open: true) # No query yet
@open_order = @open_orders.first # Now the database will be queried
{% endhighlight %}

#### 4.3.1.13 collection.exists?(...)

collection.exist?方法检查一个object是否符合在collection里所提供的查询条件。它使用的是和ActiveRecord::Base.exist?相同的语法和选项。

#### 4.3.1.14 collection.build(attributes = {}, ...)

collection.build返回的是一个或者多个新的关联类型objects。这些objects将会从传入的属性里被实例化，然后通过他们的外键的关联将会被创建，但已关联的对象*将不会被保存*。

{% highlight ruby %}
@order = @customer.orders.build(order_date: Time.now,
                                order_number: "A12345")
{% endhighlight %}

#### 4.3.1.15 collection.create(attributes = {})

collection.create方法返回的一个新的关联类型object。这些objects将会从传入的属性里被实例化，然后通过他们的外键的关联将会被创建，一旦它通过针对已关联model所有的校验，那么已关联object*将会被保存*。

#### 4.3.1.16 collection.create!(attributes = {})

类似于上面的collection.create，但如果数据记录无效时，会抛出ActiveRecord::RecordInvalid。

### 4.3.2 has_many选项

当Rails使用智能化的默认值，就能完成大部分的情况，当你想要定制化关于has_many关联的行为时，可能需要一些时间。类似于这样的定制化当你创建该关联性时，可以很简单地通过设定选项值来完成。例如，这个关联性使用了两个这样的选项：

{% highlight ruby %}
class Customer < ActiveRecord::Base
  has_many :orders, dependent: :delete_all, validate: :false
end
{% endhighlight %}

has_many关联支持这些选项：

* :as
* :autosave
* :class_name
* :dependent
* :foreign_key
* :inverse_of
* :primary_key
* :source
* :source_type
* :through
* :validate

#### 4.3.2.1 :as

设置:as选项指的是，这是一个多态关联，详细细节在[earlier in this guide](http://guides.rubyonrails.org/association_basics.html#polymorphic-associations)。

#### 4.3.2.2 :autosave

如果你设置:autosave选项值为true，无论何时当你想保存父类object时，Rails将会保存任何载入的成员和销毁那些被标记为可毁灭的成员。

#### 4.3.2.3 :class_name

如果其他model的名字不能够从关联名字哪里推导出来，你可以使用:class_name选项来指定model名字。例如，如果一个customer有多个orders，但是包含orders的model实际名字是Transaction，你将会用这个来设置：

{% highlight ruby %}
class Customer < ActiveRecord::Base
  has_many :orders, class_name: "Transaction"
end
{% endhighlight %}

#### 4.3.2.4 :dependent

当关联对象的拥有者被销毁时，可以控制它的执行情况：

* :destroy引起已关联对象被销毁
* :delete引起已关联对象被直接从数据库里删除（因此回调函数将不会执行）
* :nullify引起外键被设置为NULL。回调函数不会被执行。
* :restrict_with_exception如果有一个已关联的记录的话，会引起一个异常被抛出
* :restrict_with_error如果有一个已关联对象，会引起一个错误被增加到owner

>note:
>
>当你使用:through选项时，这个选项会被忽视。

#### 4.3.2.5 :foreign_key

基于约定，Rails假设在其他model里用来作为外键的数据列是这个model名字加上后缀_id。:foreign_key选项是让你直接设置外键名称：

{% highlight ruby %}
class Customer < ActiveRecord::Base
  has_many :orders, foreign_key: "cust_id"
end
{% endhighlight %}

>information
>
>在任何情况下，Rails不会为你创建外键的列。你需要在你的数据迁移文件（migration）里明确定义他们。

#### 4.3.2.6 :inverse_of

:inverse_of选项指定的是belongs_to关联的名字，它是这个关联的反向。不要结合:through或者:as选项一起使用。

{% highlight ruby %}
class Customer < ActiveRecord::Base
  has_many :orders, inverse_of: :customer
end
 
class Order < ActiveRecord::Base
  belongs_to :customer, inverse_of: :orders
end
{% endhighlight %}

#### 4.3.2.7 :primary_key

按照约定，Rails假设一个列被用来作为model的主键是id。你可以重写它，显示地声明主键，用:primary_key选项。

让我们以users数据表为例，它有一个id主键，但它也有guid数据列。需求就是todos数据表应该有guid数据列而不是id的值。可以这样子实现：

{% highlight ruby %}
class User < ActiveRecord::Base
  has_many :todos, primary_key: :guid
end
{% endhighlight %}

现在如果我们执行@user.todos.create，那么@todo数据记录会有user_id的值来作为@user的guid的值。

#### 4.3.2.8 :source

:source选项指定的是source关联名通过一个has_many :through关联。

如果source关联名字不能够从关联名字里被自动推导出来，你只需要使用这个选项。

#### 4.3.2.9 :source_type

:source_type选项指的是source关联类型用来一个has_many :through关联，用来处理一个多态的关联。

#### 4.3.2.10 :through

:through选项指的是一个join model通过哪个model来执行查询。has_many :through关联提供了一个方式实现many-to-many关系，细节已经在前面讲过[earlier in this guide](http://guides.rubyonrails.org/association_basics.html#the-has-many-through-association)。

#### 4.3.2.11 :validate

如果你设置:validate选项值为false，那么被关联的对象无论你何时保存它都将不会被验证。默认的情况下，这个选项的值是true，当这个对象被保存的时候将会被校验。

### 4.3.3 has_many范围

当你想要用has_many定制化查询时，可能需要一些时间。类似于这样的定制化能够通过一个代码块来获取。例如：

{% highlight ruby %}
class Customer < ActiveRecord::Base
  has_many :orders, -> { where processed: true }
end
{% endhighlight %}

你可以在代码块内使用一些的标准的查询方法[querying methods](http://guides.rubyonrails.org/active_record_querying.html)，将在下面的内容里讨论这些：

* where
* extending
* group
* includes
* limit
* offset
* order
* readonly
* select
* uniq

#### 4.3.3.1 where

where方法让你指定条件，所取得的关联对象必须要满足的条件。

{% highlight ruby %}
class Customer < ActiveRecord::Base
  has_many :confirmed_orders, -> { where "confirmed = 1" },
    class_name: "Order"
end
{% endhighlight %}

你也可以通过一个hash来设置多个条件：

{% highlight ruby %}
class Customer < ActiveRecord::Base
  has_many :confirmed_orders, -> { where confirmed: true },
                              class_name: "Order"
end
{% endhighlight %}

如果你使用一个hash-style的where选项，那么通过这个关联性创建的数据记录行将会自动地来使用hash。在这个例子里，使用@customer.confirmed_orders.create或者@customer.confirmed_orders.build将会创建orders是那些已被确认为true的数据。

#### 4.3.3.2 extending

extending方法是指的一个命名的module来扩展关联代理。关联代理将被在下面讨论其细节。[later in this guide](http://guides.rubyonrails.org/association_basics.html#association-extensions)。

#### 4.3.3.3 group

gourp方法提供一个特性名称用来分组结果集，在SQL查找中使用一个GROUP BY语句。

{% highlight ruby %}
class Customer < ActiveRecord::Base
  has_many :line_items, -> { group 'orders.id' },
                        through: :orders
end
{% endhighlight %}

#### 4.3.3.4 includes

你可以使用includes方法来指定第二顺位的(second-order)关联，当这个关联被使用时而被及时加载(eager-loaded)。例如，考虑一下这些方法：

{% highlight ruby %}
class Customer < ActiveRecord::Base
  has_many :orders
end
 
class Order < ActiveRecord::Base
  belongs_to :customer
  has_many :line_items
end
 
class LineItem < ActiveRecord::Base
  belongs_to :order
end
{% endhighlight %}

如果你经常性地从line items中直接获取customers(@customer.orders.line_items)，那么你可以使得你的代码更有效率，在关联中包括line items从customers到orders：

{% highlight ruby %}
class Customer < ActiveRecord::Base
  has_many :orders, -> { includes :line_items }
end
 
class Order < ActiveRecord::Base
  belongs_to :customer
  has_many :line_items
end
 
class LineItem < ActiveRecord::Base
  belongs_to :order
end
{% endhighlight %}

#### 4.3.3.5 limit

limit方法让你限制objects的合计数量，通过一个关联来获取。

{% highlight ruby %}
class Customer < ActiveRecord::Base
  has_many :recent_orders,
    -> { order('order_date desc').limit(100) },
    class_name: "Order",
end
{% endhighlight %}

#### 4.3.3.6 offset

offset让你指定一个开始的offset通过一个关联来获取objects。例如->{ offset(11) }将会跳过前11个数据记录行。

#### 4.3.3.7 order

order方法指的是所获取的关联对象的顺序（在SQL语法上使用的是ORDER BY语句）。

{% highlight ruby %}
class Customer < ActiveRecord::Base
  has_many :orders, -> { order "date_confirmed DESC" }
end
{% endhighlight %}

#### 4.3.3.8 readonly

如果你使用readonly，那么当通过关联被接收时，关联对象将会是只读的（read-only）。

#### 4.3.3.9 select

select方法可以让你重写SQL SELECT语句，被用来获得关于关联对象的数据。默认情况下，Rails获得是所有列。

>warning
>
>如果你声明你自己的select，确认包含已关联model的主键和外键列。如果你没有这样做，那么Rails将会抛出一个error。

#### 4.3.3.10 distinct

使用distinct方法可以保持collection复制的自由。和:through选项一起使用这是比较有用的。

{% highlight ruby %}
class Person < ActiveRecord::Base
  has_many :readings
  has_many :posts, through: :readings
end
 
person = Person.create(name: 'John')
post   = Post.create(name: 'a1')
person.posts << post
person.posts << post
person.posts.inspect # => [#<Post id: 5, name: "a1">, #<Post id: 5, name: "a1">]
Reading.all.inspect  # => [#<Reading id: 12, person_id: 5, post_id: 5>, #<Reading id: 13, person_id: 5, post_id: 5>]
{% endhighlight %}

在上面的例子里是两个readings，person.posts都关联了这两个，甚至这些数据记录行指向的是同一个post。

现在让我们来设置distinct：

{% highlight ruby %}
class Person
  has_many :readings
  has_many :posts, -> { distinct }, through: :readings
end
 
person = Person.create(name: 'Honda')
post   = Post.create(name: 'a1')
person.posts << post
person.posts << post
person.posts.inspect # => [#<Post id: 7, name: "a1">]
Reading.all.inspect  # => [#<Reading id: 16, person_id: 7, post_id: 7>, #<Reading id: 17, person_id: 7, post_id: 7>]
{% endhighlight %}

在上面例子里依然有两个readings。然而person.posts只显示一个post，因为collection只载入唯一的数据记录。

如果你想要确认的是，对于insertion，在persisted关联里的所有数据记录是distinct的（因此你可以确定的是，当你显示关联性时，你将不会发现重复的记录行），对于数据表本身你应该增加一个唯一索引。例如，如果你有一个数据表名字是person_posts，你想要确定的是所有的posts都是唯一的，你应该在数据迁移文件（migration）里增加下面的：

{% highlight ruby %}
add_index :person_posts, :post, unique: true
{% endhighlight %}

注意，校验唯一性使用的是类似于include?，这是执行条件的一部分内容。不要想要在一个关联里通过使用include?来执行distinctness。例如，使用上个post的例子，下面的代码将会是非常好用的，因为多个用户可能同时间来尝试：

{% highlight ruby %}
person.posts << post unless person.posts.include?(post)
{% endhighlight %}

### 4.3.4 When are Objects Saved?

当你给一个has_many关联赋值一个对象时，对象是自动地被保存（为了更新它的外键）。如果你在一个语句中赋值了多个对象，那么他们都会被保存。

如果这些保存因为校验出错而失败了，那么赋值语句返回的是false，赋值本身会被取消。

如果父类对象（声明has_many的关联）不能被保存（就是new_record? 返回的是true），那么子对象在被新增时也不会被保存。当父类被保存时，所有没有保存的关联成员也会被自动化地保存。

如果你想要给一个has_many关联性赋值一个对象，没有保存对象，通过使用association.build方法。

## 4.4 has_and_belongs_to_many关联性

has_and_belongs_to_many关联创建了与另一个model的一个many-to-many关系。在数据库层面，这个关联的两个类通过一个中间联合表，该表包含了联系各自类的外键。

### 4.4.1 has_and_belongs_to_many方法

当你声明一个has_and_belongs_to_many关联时，所声明的类自动获得16个与之关联的方法：

* collection(force_reload = false)
* collection<<(object, ...)
* collection.delete(object, ...)
* collection.destroy(object, ...)
* collection=objects
* collection_singular_ids
* collection_singular_ids=ids
* collection.clear
* collection.empty?
* collection.size
* collection.find(...)
* collection.where(...)
* collection.exists?(...)
* collection.build(attributes = {})
* collection.create(attributes = {})
* collection.create!(attributes = {})

在所有的这些方法里，collection是被symbol所替代的，作为has_and_belongs_to_many的第一个参数，collection_singular是被symbol的单数版本所替代。例如，给予这样的声明：

{% highlight ruby %}
class Part < ActiveRecord::Base
  has_and_belongs_to_many :assemblies
end
{% endhighlight %}

每一个part model实例将会有这些方法啊:

* assemblies(force_reload = false)
* assemblies<<(object, ...)
* assemblies.delete(object, ...)
* assemblies.destroy(object, ...)
* assemblies=objects
* assembly_ids
* assembly_ids=ids
* assemblies.clear
* assemblies.empty?
* assemblies.size
* assemblies.find(...)
* assemblies.where(...)
* assemblies.exists?(...)
* assemblies.build(attributes = {}, ...)
* assemblies.create(attributes = {})
* assemblies.create!(attributes = {})

#### 4.4.1.1 额外的列方法

如果有一个has_and_belongs_to_many关联的联合表有额外的列，不单单是两个外键，这些列将会通过那个关联性被增加为所接收数据记录的属性。返回额外属性的数据记录一般都是只读的（read-only），因为Rails不能够保存那些属性的改变。

>warning
>
>对于联合表里has_and_belongs_to_many关联性额外属性的用法是不推荐的。如果在一个many-to-many关系里有联合两个model的数据表，你需要这个复杂行为的类型，你应该使用一个has_many :through关联性来替代has_and_belongs_to_many。

#### 4.4.1.2 collection(force_reload = false)

collection方法返回是一个关于所关联对象的数组。如果没有所关联的对象，它将返回一个空的数组。

{% highlight ruby %}
@assemblies = @part.assemblies
{% endhighlight %}

#### 4.4.1.3 collection<<(object, ...)

collection<<方法增加一个或者多个对象到这个collection里，通过在联合表里创建数据记录。

{% highlight ruby %}
@part.assemblies << @assembly1
{% endhighlight %}

>note
>
>这个方法是collection.concat和collection.push的别名

#### 4.4.1.4 collection.delete(object, ...)

collection.delete方法从这个collection里移除一个或者多个方法，通过删除联合表里的数据记录。这不会销毁objects。

{% highlight ruby %}
@part.assemblies.delete(@assembly1)
{% endhighlight %}

>warning
>
>这不会触发联合数据记录的回调函数。

#### 4.4.1.5 collection.destroy(object, ...)

collection.destroy方法从collection里移除一个或者多个对象，通过执行在联合表里每一个对象的销毁，包括运行时的回调函数。这不会销毁objects。

{% highlight ruby %}
@part.assemblies.destroy(@assembly1)
{% endhighlight %}

#### 4.4.1.6 collection=objects

collection=方法使得collection只包含所提供的对象，用来新增和删除。

#### 4.4.1.7 collection_singular_ids

collection_singular_ids方法返回的是在collection里的一个对象ids的数组。

{% highlight ruby %}
@assembly_ids = @part.assembly_ids
{% endhighlight %}

#### 4.4.1.8 collection_singular_ids=ids

collection_singular_ids=方法使得collection只包含所指定的对象，是通过所提供的主键值，用来新增和删除。

#### 4.4.1.9 collection.clear

collection.clear方法通过删除联合表里的数据行，来移除collection里每一个对象。这不会销毁已关联对象。

#### 4.4.1.10 collection.empty?

如果collection没有包含任何关联对象，collection.empty?方法返回的是true。

{% highlight ruby %}
<% if @part.assemblies.empty? %>
  This part is not used in any assemblies
<% end %>
{% endhighlight %}

#### 4.4.1.11 collection.size

collection.size方法返回的是在collection里对象的数量。

{% highlight ruby %}
@assembly_count = @part.assemblies.size
{% endhighlight %}

#### 4.4.1.12 collection.find(...)

collection.find方法是在collection里查找objects。它用的是和ActiveRecord::Base.find相同的语法和选项。它也增加了额外的条件，就是该object必须在collection里。

{% highlight ruby %}
@assembly = @part.assemblies.find(1)
{% endhighlight %}

#### 4.4.1.13 collection.where(...)

collection.where方法是基于所提供的查询条件在collection里查找objects，但这些objects是延时导入，意味着数据库被查询只是在object(s)能被读取时。它也增加了额外的条件，就是该object必须在collection里。

{% highlight ruby %}
@new_assemblies = @part.assemblies.where("created_at > ?", 2.days.ago)
{% endhighlight %}

#### 4.4.1.14 collection.exists?(...)

collection.exist?方法检查一个object是否符合在collection里所提供的查询条件。它使用的是和ActiveRecord::Base.exist?相同的语法和选项。

#### 4.4.1.15 collection.build(attributes = {})

collection.build返回的是一个或新的关联类型object。这object将会从传入的属性里被实例化，然后通过联合表的关联将会被创建，但已关联的对象*将不会被保存*。

{% highlight ruby %}
@assembly = @part.assemblies.build({assembly_name: "Transmission housing"})
{% endhighlight %}

#### 4.4.1.16 collection.create(attributes = {})

collection.create方法返回的一个新的关联类型object。这object将会从传入的属性里被实例化，然后通过联合表的关联将会被创建，一旦它通过针对已关联model所有的校验，那么已关联object*将会被保存*。

{% highlight ruby %}
@assembly = @part.assemblies.create({assembly_name: "Transmission housing"})
{% endhighlight %}

#### 4.4.1.17 collection.create!(attributes = {})

类似于上面的collection.create，但如果数据记录无效时，会抛出ActiveRecord::RecordInvalid。

### 4.4.2 has_and_belongs_to_many选项

当Rails使用智能化的默认值，就能完成大部分的情况，当你想要定制化关于has_and_belongs_to_many关联的行为时，可能需要一些时间。类似于这样的定制化当你创建该关联性时，可以很简单地通过设定选项值来完成。例如，这个关联性使用了两个这样的选项：

{% highlight ruby %}
class Parts < ActiveRecord::Base
  has_and_belongs_to_many :assemblies, autosave: true,
                                       readonly: true
end
{% endhighlight %}

has_and_belongs_to_many关联支持这些选项：

* :association_foreign_key
* :autosave
* :class_name
* :foreign_key
* :join_table
* :validate
* :readonly

#### 4.4.2.1 :association_foreign_key

基于约定，Rails假设在联合表里的列被用来作为外键指向另一个model，该列是那个model的名字加上后缀_id。:association_foreign_key选项让你直接设置外键的名称：

>information
>
>当设置一个many-to-many self-join时，:foreign_key和:association_foreign_key选项是有用的。例如：

{% highlight ruby %}
class User < ActiveRecord::Base
  has_and_belongs_to_many :friends,
      class_name: "User",
      foreign_key: "this_user_id",
      association_foreign_key: "other_user_id"
end
{% endhighlight %}

#### 4.4.2.2 :autosave

如果你把:autosave选项设置为true，只要在你保存父类对象时，Rails将会保存任何载入的事物和销毁已被标记为可删除的事物。

#### 4.4.2.3 :class_name

如果其他model的名字不能够从关联名字哪里推导出来，你可以使用:class_name选项来指定model名字。例如，如果一个part有多个assemblies，但是包含assemblies的model实际名字是Gadget，你将会用这个来设置：

{% highlight ruby %}
class Parts < ActiveRecord::Base
  has_and_belongs_to_many :assemblies, class_name: "Gadget"
end
{% endhighlight %}

#### 4.4.2.4 :foreign_key

基于约定，Rails假设在联合表里用来作为外键的数据列指向这个model，该列名字是这个model名字加上后缀_id。:foreign_key选项是让你直接设置外键名称：

{% highlight ruby %}
class User < ActiveRecord::Base
  has_and_belongs_to_many :friends,
      class_name: "User",
      foreign_key: "this_user_id",
      association_foreign_key: "other_user_id"
end
{% endhighlight %}

#### 4.4.2.5 :join_table

如果联合表的默认名字，是基于辞典编纂的规则，而不是你想要的，那么你能够使用:join_table选项来重写其默认值。

#### 4.4.2.6 :validate

如果你设置:validate选项值为false，那么被关联的对象无论你何时保存它都将不会被验证。默认的情况下，这个选项的值是true，当这个对象被保存的时候将会被校验。

### 4.4.3 has_and_belongs_to_many范围

当你想要用has_and_belongs_to_many定制化查询时，可能需要一些时间。类似于这样的定制化能够通过一个代码块来获取。例如：

{% highlight ruby %}
class Parts < ActiveRecord::Base
  has_and_belongs_to_many :assemblies, -> { where active: true }
end
{% endhighlight %}

你可以在代码块内使用一些的标准的查询方法[querying methods](http://guides.rubyonrails.org/active_record_querying.html)，将在下面的内容里讨论这些：

* where
* extending
* group
* includes
* limit
* offset
* order
* readonly
* select
* uniq

#### 4.4.3.1 where

where方法让你指定条件，所取得的关联对象必须要满足的条件。

{% highlight ruby %}
class Parts < ActiveRecord::Base
  has_and_belongs_to_many :assemblies,
    -> { where "factory = 'Seattle'" }
end
{% endhighlight %}

你也能够通过一个hash来设置条件:

{% highlight ruby %}
class Parts < ActiveRecord::Base
  has_and_belongs_to_many :assemblies,
    -> { where factory: 'Seattle' }
end
{% endhighlight %}

如果你使用一个hash-style的where选项，那么通过这个关联性创建的数据记录行将会自动地来使用hash。在这个例子里，使用@parts.assemblies.create或者@parts.assemblies.build将会创建orders是那些factory列的值是“Seattle”的数据。

#### 4.4.3.2 extending

extending方法是指的一个命名的module来扩展关联代理。关联代理将被在下面讨论其细节。[later in this guide](http://guides.rubyonrails.org/association_basics.html#association-extensions)。

#### 4.4.3.3 group

gourp方法提供一个特性名称用来分组结果集，在SQL查找中使用一个GROUP BY语句。

{% highlight ruby %}
class Parts < ActiveRecord::Base
  has_and_belongs_to_many :assemblies, -> { group "factory" }
end
{% endhighlight %}

#### 4.4.3.4 includes

你可以使用includes方法来指定第二顺位的(second-order)关联，当这个关联被使用时而被及时加载(eager-loaded)。

#### 4.4.3.5 limit

limit方法让你限制objects的合计数量，通过一个关联来获取。

{% highlight ruby %}
class Parts < ActiveRecord::Base
  has_and_belongs_to_many :assemblies,
    -> { order("created_at DESC").limit(50) }
end
{% endhighlight %}

#### 4.4.3.6 offset

offset让你指定一个开始的offset通过一个关联来获取objects。例如->{ offset(11) }将会跳过前11个数据记录行。

#### 4.4.3.7 order

order方法指的是所获取的关联对象的顺序（在SQL语法上使用的是ORDER BY语句）。

{% highlight ruby %}
class Parts < ActiveRecord::Base
  has_and_belongs_to_many :assemblies,
    -> { order "assembly_name ASC" }
end
{% endhighlight %}

#### 4.4.3.8 readonly

如果你使用readonly，那么当通过关联被接收时，关联对象将会是只读的（read-only）。

#### 4.4.3.9 select

select方法可以让你重写SQL SELECT语句，被用来获得关于关联对象的数据。默认情况下，Rails获得是所有列。

#### 4.4.3.10 uniq

使用uniq方法来移除collection中重复的记录。

### 4.4.4 Objects何时被保存?

当你给一个has_and_belong_to_many关联赋值一个对象时，对象是自动地被保存（为了更新联合表）。如果你在一个语句里赋值了多个objects，那么他们都会被保存。

如果这些保存因为校验出错而失败了，那么赋值语句返回的是false，赋值本身会被取消。

如果父类对象（声明has_and_belongs_to_many的关联）不能被保存（就是new_record? 返回的是true），那么子对象被新增时他们不会被保存。当父类被保存时，所有没有保存的关联成员也会被自动化地保存。

如果你想要给一个has_and_belongs_to_many关联性赋值一个对象，没有保存对象，通过使用association.build方法。

## 4.5 关联性回调函数

一般的回调函数是存在于Active Record objects的生命周期里，允许你在多个方面可以和那些objects一起执行。例如，你能够使用一个:before_save回调函数来引发一些事情的发生，只在一个object被保存前。

关联性回调函数和一般的回调函数是类似的，但他们是在一个collection的生命周期里被事件触发的。这里有四个可用的关联性回调函数：

* before_add
* after_add
* before_remove
* after_remove

你通过在关联性声明里增加选项来定义一个关联性回调函数。例如：

{% highlight ruby %}
class Customer < ActiveRecord::Base
  has_many :orders, before_add: :check_credit_limit
 
  def check_credit_limit(order)
    ...
  end
end
{% endhighlight %}


## 4.6 关联性扩展


