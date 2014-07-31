---
layout: post
title:  "Active Record Callbacks 7 关联的回调函数"
date:   2014-07-31 23:05:55
categories: Rails
tags: Rails Active Record Callbacks
---

回调函数通过模型间的关联来执行，甚至能够被他们所定义。假设一个例子，一个用户写了许多博客。如果这个用户被销毁了，那么他的博客也应该被销毁。让我们在User模型里增加一个after_destroy回调函数，通过它来关联Post模型：

{% highlight ruby %}
class User < ActiveRecord::Base
  has_many :posts, dependent: :destroy
end
 
class Post < ActiveRecord::Base
  after_destroy :log_destroy_action
 
  def log_destroy_action
    puts 'Post destroyed'
  end
end
 
>> user = User.first
=> #<User id: 1>
>> user.posts.create!
=> #<Post id: 1, user_id: 1>
>> user.destroy
Post destroyed
=> #<User id: 1>
{% endhighlight%}