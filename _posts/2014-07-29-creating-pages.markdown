---
layout: post
title:  "创建页面"
date:   2014-07-28 22:32:27
categories: jekyll
tags: jekyll
---

这是[writing posts](http://jekyllrb.com/docs/posts/)的加强版，你可能想要处理Jekyll网站的另外一个东西，那就是创建静态页面。Jekyll的简单之处就是只要拷贝文件和目录即可。

### 主页面

关于每一个你设置的web服务器配置，只要在网站的主目录下去查找一个HTML文件，名字叫**index.html**(这是约定俗成的)，然后显示出来，这就是主页面。除非你使用的web服务器是被默认配置为其他不同的文件名，这个文件将会是你的Jekyll-generated网站的主页面。

>ProTip<sup>TM</sup>: 在你主页面上使用布局
> 
>在你网站里的任何HTML文件能够使用布局文件或者includes，甚至是主页面。通常的内容，类似header和footers，非常适合抽象出来形成一个布局文件。

### 额外的页面存在于何处

你把页面的HTML文件放在何处，依赖于你想让页面如何去工作。这里有两种主要的方式用来创建页面：

*    把每一个页面所命名的HTML文件放置在你网站的根目录下。
*    在网站的根目录下，为每一个页面创建一个文件夹，然后在每一个页面文件夹下放置一个**index.html**文件。

两种方法都可以（两个也可以结合起来使用），两者的不同之处是在于URLs的显示。

### 命名HTML文件

增加一个页面最简单的方式是在根目录里增加一个HTML文件，然后给你想创建的一面赋予一个合适的名字。对于一个网站有一个主页面，一个about页面和一个联系页面，下面是根目录和相关联的URLs的例子：
{% highlight text %}  
.
|-- _config.yml
|-- _includes/
|-- _layouts/
|-- _posts/
|-- _site/
|-- about.html    # => http://example.com/about.html
|-- index.html    # => http://example.com/
└── contact.html  # => http://example.com/contact.html
{% endhighlight %}

### 命名包含索引HTML文件的文件夹

上面的方法是可行的，然而一些人喜欢让他们的URLs搞得类似于文件名的扩展。为了获得清晰的URLS，可以使用Jekyll，你只要简单的在每一个页面的上一级创建一个文件夹，然后在每一个页面的文件夹里放置一个**index.html**。这个方式里的页面URL结尾处是一个文件夹名，这样的话web服务会调用各自的**index.html**文件。下面就是一个例子，涉及到这种目录结构的：

{% highlight text %} 
.
├── _config.yml
├── _includes/
├── _layouts/
├── _posts/
├── _site/
├── about/
|   └── index.html  # => http://example.com/about/
├── contact/
|   └── index.html  # => http://example.com/contact/
└── index.html      # => http://example.com/
{% endhighlight %}

这个方式可能不适合每个人，但对于那些喜欢清晰URLs的人们而言，它是简单又实用。最终的决定权还是取决于你！