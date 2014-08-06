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
>belongs_to关联必须使用单数的对象。