---
layout: post
title:  "Active Record Callbacks 10 回调函数事务"
date:   2014-08-04 23:25:55
categories: Rails
tags: Rails Active Record Callbacks
---

这里有两个额外的由一个数据库事务来完成的回调函数，after_commit和after_rollback。这些回调函数和after_save回调函数是非常相似的，他们不会执行，除非等到数据库完成修改包括已经被提交或者回滚。当你的Active Record Models需要和另外的系统交互时，所交互的不是数据库的事务，他们是非常有用的。

思考一下，例如，上一个例子，在PictureFile Model需要删除一个文件的地方所对应的数据记录也要被删除。如果任何事务抛出了一个异常，