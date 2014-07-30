---
layout: post
title:  "Active Record Callbacks 4 运行回调函数"
date:   2014-07-30 22:45:55
categories: Rails
tags: Rails Active Record Callbacks
---

下面的方法会触发回调函数：

*  create
*  create!
*  decrement!
*  destroy
*  destroy!
*  destroy_all
*  increment!
*  save
*  save!
*  save(validate: false)
*  toggle!
*  update_attribute
*  update
*  update!
*  valid?

另外，after_find回调函数会被下面的finder方法所触发：

*  all
*  first
*  find
*  find_by
*  find_by_*
*  find_by_*!
*  find_by_sql
*  last

每一次新的类对象被初始化时，after_initialize回调函数会被触发。

> Note
>
>find_by\_\*和find_by\_\*!方法是动态的finders，自动地产生每一个特性。更多的了解这些[Dynamic finders section](http://guides.rubyonrails.org/active_record_querying.html#dynamic-finders)
