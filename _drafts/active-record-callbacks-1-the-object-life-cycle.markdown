---
layout: post
title:  "Active Record Callbacks 1 对象生命周期"
date:   2014-07-29 16:02
categories: Rails
tags: Rails Active Record Callbacks
---

### 1 对象生命周期

在一个Rails应用的普通操作中，对象可以被创建，更新，和被销毁。在这个*对象生命周期*里Active Record 提供了hooks，因此你能够控制你的应用和它的数据。

回调是允许你出发逻辑部分的，在修改一个对象的状态之前或者之后。

### 2 回调概要

回调是方法，可以在一个对象生命周期的某一个时刻被调用。可以在回调函数中编写代码，无论一个Active Record对象在被创建，保存，更新，删除，验证，或者从数据库里导入时，都可以运行。

## 2.1 回调函数的注册

为了能够使用回调函数，你需要先注册他们。你可以像实现普通函数般来实现回调函数，使用一个宏类型的方法来注册他们实现回调。

{% highlight ruby %} 
class User < ActiveRecord::Base
  validates :login, :email, presence: true
 
  before_validation :ensure_login_has_a_value
 
  protected
    def ensure_login_has_a_value
      if login.nil?
        self.login = email unless email.blank?
      end
    end
end
{% endhighlight %}

