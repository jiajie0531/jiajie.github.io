---
layout: post
title:  "Active Record Associations 1 为什么用关联性？"
date:   2014-08-05 23:00:55
categories: Rails
tags: Rails Active Record Associations
---

为什么在模型之间我们需要关联性？因为他们可以在你的代码中更简单更方便的执行普通操作。例如，想象一个简单的Rails应用程序，包含了一个customers的模型和一个orders模型。每一个customer可以拥有多个orders。没有关联性，模型的声明就这样子：

{% highlight ruby %}
class Customer < ActiveRecord::Base
end
 
class Order < ActiveRecord::Base
end
{% endhighlight %}

现在，假设我们想要给已存在的一个customer增加一个新的order。我们需要这样子做：

{% highlight ruby %}
@order = Order.create(order_date: Time.now, customer_id: @customer.id)
{% endhighlight %}

或者考虑删除一个customer，确保属于他的orders也要被删除：

{% highlight ruby %}
@orders = Order.where(customer_id: @customer.id)
@orders.each do |order|
  order.destroy
end
@customer.destroy
{% endhighlight %}

要是用Active Record的关联性，我们可以把这些操作变得合理化，告知Rails在两个模型之间有关联。这是修改过的代码，关于customers和orders：

{% highlight ruby %}
class Customer < ActiveRecord::Base
  has_many :orders, dependent: :destroy
end
 
class Order < ActiveRecord::Base
  belongs_to :customer
end
{% endhighlight %}

随着这个改变，为特定的customer创建一个新的order是比较简单的：

{% highlight ruby %}
@order = @customer.orders.create(order_date: Time.now)
{% endhighlight %}

删除一个customer以及所属的orders也是比较简单的：

{% highlight ruby %}
@customer.destroy
{% endhighlight %}

为了了解关联性的不同的类型，读下去即可。下文会有一些使用关联性的窍门和技巧，通过在Rails里关于方法的一个完整引用和关联性的选项。
