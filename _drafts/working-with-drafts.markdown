---
layout: post
title:  "Active Record Associations 8 条件回调函数"
date:   2014-08-04 23:05:55
categories: Rails
tags: Rails Active Record Associations
---


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