---
layout: post
title:  "Active Record Callbacks 8 条件回调函数"
date:   2014-08-04 23:05:55
categories: Rails
tags: Rails Active Record Callbacks
---

如同校验一样，我们也可以调用一个有条件的回调函数，在满足给定的判断条件下。我们能够使用:if和:unless属性来实现这个，后面可以跟一个标志，一个字符串，一段代码或者一个数组。当你想要指定回调函数应该什么条件下被调用，你可以使用:if属性。如果你想要指定回调函数在什么条件下不应该被调用，那么你可以使用:unless属性。

### 8.1 使用:if和:unless和标志

你能够把:if和:unless属性和标志关联起来，这个标志对应的是一个断言方法的名称，这个方法在回调函数之前被调用。当使用:if选项时，如果断言方法返回false，回调函数将不会被执行；当使用:unless选项时，如果断言方法返回true，回调函数将不会被执行。

{% highlight ruby %}
class Order < ActiveRecord::Base
  before_save :normalize_card_number, if: :paid_with_card?
end
{% endhighlight %}

### 8.2 使用:if和:unless和一字符串

你也可以使用一个字符串，将会被转换为运算式，因此需要包含可验证的代码。当字符串描述的真正地短条件时，你应该只使用这个选项：

{% highlight ruby %}
class Order < ActiveRecord::Base
  before_save :normalize_card_number, if: "paid_with_card?"
end
{% endhighlight %}

### 8.3 使用:if和:unless和一段代码

最终，还是能够关联到:if和:unless后面跟着一段代码。当用来写简短的验证方法时，这个选项是最合适的，通常都是一行：

{% highlight ruby %}
class Order < ActiveRecord::Base
  before_save :normalize_card_number,
    if: Proc.new { |order| order.paid_with_card? }
end
{% endhighlight %}

### 8.4 多个条件的回调函数

当编写有条件的回调函数时，有可能在同一个回调函数声明中混合使用:if和:unless：
{% highlight ruby %}
class Comment < ActiveRecord::Base
  after_create :send_email_to_author, if: :author_wants_emails?,
    unless: Proc.new { |comment| comment.post.ignore_comments? }
end
{% endhighlight%}

