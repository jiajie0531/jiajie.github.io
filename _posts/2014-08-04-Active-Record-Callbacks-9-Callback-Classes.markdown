---
layout: post
title:  "Active Record Callbacks 9 回调函数的类"
date:   2014-08-04 23:15:55
categories: Rails
tags: Rails Active Record Callbacks
---

有时候，你所编写的回调函数是很有用的，能够被用来让其他模型重用。Active Record能够创建类，用来封装回调函数，因此就变得很容易地重用他们。

这里有一个例子，在PictureFile类中创建了一个类里面有一个回调函数after_destroy：

{% highlight ruby %}
class PictureFileCallbacks
  def after_destroy(picture_file)
    if File.exist?(picture_file.filepath)
      File.delete(picture_file.filepath)
    end
  end
end
{% endhighlight %}

当声明在类的内部时，就如同上面的例子，回调函数将会接收一个模型对象，作为一个参数。现在我们就能够在模型里来使用这个回调函数类了：

{% highlight ruby %}
class PictureFile < ActiveRecord::Base
  after_destroy PictureFileCallbacks.new
end
{% endhighlight %}

注意，我们需要实例化一个 new PictureFileCallbacks 对象，既然我们声明了我们的回调函数作为一个实例变量。如果回调函数使用了这个已实例化对象的状态的话，这就是一种特殊的用处。然而，经常性地用法是把回调函数声明为类的方法：

{% highlight ruby %}
class PictureFileCallbacks
  def self.after_destroy(picture_file)
    if File.exist?(picture_file.filepath)
      File.delete(picture_file.filepath)
    end
  end
end
{% endhighlight %}

如果回调函数被这样声明的话，将不需要去实例化一个 PictureFileCallbacks 对象。

{% highlight ruby %}
class PictureFile < ActiveRecord::Base
  after_destroy PictureFileCallbacks
end
{% endhighlight %}

只要你想要在回调函数里声明，你就能够声明多个回调函数。
