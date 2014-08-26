---
layout: post
title:  "Active Record Query Interface 1 从数据库获取对象"
date:   2014-08-26 17:05:55
categories: Rails
tags: Rails Active Record Query Interface
---

![P1000759](/assets/2014/08/12/P1000759.gif)

![belongs_to](/assets/2014/08/06/belongs_to.png)

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