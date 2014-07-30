---
layout: post
title:  "Active Record Callbacks 3 可用的回调函数"
date:   2014-07-30 14:54
categories: Rails
tags: Rails Active Record Callbacks
---

这里罗列的是所有可用的Active Record的回调函数，罗列在同一个规则中，在各自的操作中他们会被调用：

### 3.1 创建一个对象

*  before_validation
*  after_validation
*  before_save
*  around_save
*  before_create
*  around_create
*  after_create
*  after_save

### 3.2 更新一个对象

*  before_validation
*  after_validation
*  before_save
*  around_save
*  before_update
*  around_update
*  after_update
*  after_save

### 3.3 销毁一个对象

*  before_destroy
*  around_destroy
*  after_destroy

>warning
>
>after_save在create和update中都有运行，但通常来说after比较经典的回调函数是after_create和after_update，无论在哪个规则中宏的调用都能被执行

### 3.4 after_initialize和after_find

只要一个Active Record对象被实例化时，after_initialize回调函数将会被调用，或者直接使用new，也可以当一条记录被从数据库取出来载入时。这是很有用处的，可以避免需要直接重写你的Active Record initialize 方法。

当Active Record从数据库里载入一条记录时，after_find回调函数将会被调用。如果两者都被定义的话，after_find会在after_initialize之前被调用。

after_initialize和after_find回调函数没有before_*配对的函数，但他们能够被注册，就如同其他的Active Record回调函数一样。

{% highlight ruby %}
class User < ActiveRecord::Base
  after_initialize do |user|
    puts "You have initialized an object!"
  end
 
  after_find do |user|
    puts "You have found an object!"
  end
end
 
>> User.new
You have initialized an object!
=> #<User id: nil>
 
>> User.first
You have found an object!
You have initialized an object!
=> #<User id: 1>
{% endhighlight %}

### 3.5 after_touch

只要一个Active Record对象被touched时，after_touch会被调用。

{% highlight ruby %}
class User < ActiveRecord::Base
  after_touch do |user|
    puts "You have touched an object"
  end
end
 
>> u = User.create(name: 'Kuldeep')
=> #<User id: 1, name: "Kuldeep", created_at: "2013-11-25 12:17:49", updated_at: "2013-11-25 12:17:49">
 
>> u.touch
You have touched an object
=> true
{% endhighlight %}

它也能够伴随着belongs_to一起使用:

{% highlight ruby %}
class Employee < ActiveRecord::Base
  belongs_to :company, touch: true
  after_touch do
    puts 'An Employee was touched'
  end
end
 
class Company < ActiveRecord::Base
  has_many :employees
  after_touch :log_when_employees_or_company_touched
 
  private
  def log_when_employees_or_company_touched
    puts 'Employee/Company was touched'
  end
end
 
>> @employee = Employee.last
=> #<Employee id: 1, company_id: 1, created_at: "2013-11-25 17:04:22", updated_at: "2013-11-25 17:05:05">
 
# triggers @employee.company.touch
>> @employee.touch
Employee/Company was touched
An Employee was touched
=> true
{% endhighlight%}