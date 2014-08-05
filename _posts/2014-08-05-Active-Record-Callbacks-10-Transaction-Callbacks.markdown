---
layout: post
title:  "Active Record Callbacks 10 回调函数事务"
date:   2014-08-05 22:19:55
categories: Rails
tags: Rails Active Record Callbacks
---

这里有两个额外的由一个数据库事务来完成的回调函数，after_commit和after_rollback。这些回调函数和after_save回调函数是非常相似的，他们不会执行，除非等到数据库完成修改包括已经被提交或者回滚。当你的Active Record Models需要和另外的系统交互时，所交互的不是数据库的事务，他们是非常有用的。

思考一下，例如，上一个例子，在PictureFile Model需要删除一个文件的地方所对应的数据记录也要被删除。在after_destroy回调函数被调用的时候以及事务回滚之后，如果任何动作抛出了一个异常，该文件会被删除，模型也会被置于一个不一致的状态。例如，假设在代码中picture_file_2是无效的，save！方法会抛出一个错误。

{% highlight ruby %}
PictureFile.transaction do
  picture_file_1.destroy
  picture_file_2.save!
end
{% endhighlight %}

通过使用after_commit回调函数，我们可以搞定这个例子。

{% highlight ruby %}
class PictureFile < ActiveRecord::Base
  after_commit :delete_picture_file_from_disk, on: [:destroy]
 
  def delete_picture_file_from_disk
    if File.exist?(filepath)
      File.delete(filepath)
    end
  end
end
{% endhighlight %}

>note:
>
>:on选项指的是这个回调函数何时会被调用。如果你没有指明:on选项，这个回调函数在每一次action时都会被调用。

>warning:
>
>after_commit和after_rollback回调函数是保证能被调用的，在所有模型在一个事务中创建，更新，或者销毁。如果这其中任一回调函数有异常被抛出，他们会被忽视，因此他们不会干扰到其他的回调函数。同样地，如果你的回调函数会抛出一个异常，你将需要去捕获它，在回滚中合适地处理它。


