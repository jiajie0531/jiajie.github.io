---
layout: post
title:  "Writing posts"
date:   2014-07-25 21:14:29
categories: jekyll
---

### 写博客


Jekyll最核心的一面就是“专注博客”。这意味着什么呢？简单的来说，它意味着博客就是Jekyll的核心功能。如果你撰写文章，然后在网上发布出来，这也就是说，你可以简单地来发布和维护一个博客，只要在你的电脑上管理一个文本文件夹即可。相比较配置的繁琐，维护数据库和机遇web的CMS系统，这将会是一个受欢迎的改变！

### 博客的文件夹

正如在[目录结构](http://jekyllrb.com/docs/structure/)页面所解释的，**_posts**文件夹就是你的博客文章所存放的地方。这些文件可以是[Makedown](http://daringfireball.net/projects/markdown/)或者[Textile](http://redcloth.org/textile)格式的文本文件，只要他们有[YAML front-matter](http://jekyllrb.com/docs/frontmatter/)，他们将会被从他们代码格式转换成HTML页面，既是你的静态站点的一部分。

### 创建博客文件

为了创建一个新的博客，你所要做的就是创建一个新的文件在**_posts**目录里。在这个文件夹里你如何命名文件名是很重要的。Jekyll需要被命名的博客文件名需要遵守下面的格式：
{% highlight ruby %} 
YEAR-MONTH-DAY-title.MARKUP
{% endhighlight %}

这里**YEAR**是四位数字，**MONTH**和**DAY**都是两位数字，以及**MARKUP**是文件扩展名，表示在文件里所使用的格式。例如，下面的例子就是有效的博客文件名：
{% highlight ruby %} 
2011-12-31-new-years-eve-is-awesome.md
2012-09-12-how-to-write-a-blog.textile
{% endhighlight %}

>protip: 关联其他的博客
>
>使用**post_url**标签来关联其他的博客，不需要担心URL的失联，在当网站的永久链接改变的时候。

### 文件内容格式
所有博客文件必须以[YAML front-matter](http://jekyllrb.com/docs/frontmatter/)为开头。然后，就很简单地选择你所需要的格式。Jekyll提供了两种比较流行的标记格式：[Markdown](http://daringfireball.net/projects/markdown/)和[Textile](http://redcloth.org/textile)。每一个格式有他们自己的标记方式，属于各自不同的内容格式，因此你应该选择你自己熟悉的格式，决定哪一个格式更适合你的需求。

>Be aware of character sets
>
>处理内容能够使得某些字符变得更美观。例如，在Redcarpet里**smart**扩展是转换标准，ASCII引用字符变化为Unicode字符。为了合理地浏览来显示那些字符，定义字符集的值为**<meta charset="utf-8">**，在你的布局文件**<head>**中。

### 包含图片和资源
在某一时刻，你将会想包含图片，下载文件，或则其他的数码资源在你的文本内容中，这些都是可以实现的。当涉及到关联资源的语法在Markdown和Textile两者之间是不一样的，问题在于找出存放这些文件在你网站里的位置，这是一些每个人都会碰到的事儿。

因为Jekyll的灵活性，关于如何来实现它是由很多解决方案。一个最通常的方案是在项目的根目录下创建一个文件夹，取名为**assets**或则**downloads**类似这样的，里面有图片，下载的内容或者其他放置的资源。然后，对于所有的博客，他们能够被关联，通过使用网站的root，其路径就是包含这些资源。这会再一次可依赖的方式，你网站的（子）域名和路径是可被设置的，但这里一些例子（用Markdown）是你如何能够用博客里的**site.url**变量来实现这个。

在一篇博客里包含一个图片资源：
{% highlight ruby %}
… which is shown in the screenshot below:
![My helpful screenshot]({{ site.url }}/assets/screenshot.jpg)
{% endhighlight %}

链接一个PDF让读者来下载：
{% highlight ruby %}
… you can [get the PDF]({{ site.url }}/assets/mydoc.pdf) directly.
{% endhighlight %}

>ProTip: Link using just site root URL
>链接使用的是网站的根URL
>
>你能够跳过**{{site.url}}**变量，如果你知道你网站不会丹丹只显示域名的主路径。在这个例子里，你可以直接引用资源通过**/path/file.jpg**。

### 显示博客的索引
把拥有的博客放在一个文件夹里是比较好的，但一个博客没有使用除非你有一个关于博客的列表。在另外的页面上（或者在一个[template](http://jekyllrb.com/docs/templates/)）创建一个博客的索引是比较简单的，
 

[writing posts](http://jekyllrb.com/docs/posts/)