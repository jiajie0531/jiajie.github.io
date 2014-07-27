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
在某一时刻，你将会想包含图片，下载文件，或则其他的数码资源在你的文本内容中，这些都是可以实现的。


[writing posts](http://jekyllrb.com/docs/posts/)