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

基于约定，Rails假设在这个model里该列用来作为外键，关联性的命名是加上后缀_id。:foreign_key选项是让你直接设置外键名称：

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

where方法让你指定条件，其关联对象是必须满足的。

{% highlight ruby %}
class Order < ActiveRecord::Base
  belongs_to :customer, -> { where active: true }
end
{% endhighlight %}

#### 4.1.3.2 includes

你可以使用includes方法来指定第二顺位的(second-order)关联，当这个关联被使用时而被期望加载。例如，考虑一下这些方法：

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



