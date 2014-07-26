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
所有博客文件必须以[YAML front-matter](http://jekyllrb.com/docs/frontmatter/)为开头。然后，

[writing posts](http://jekyllrb.com/docs/posts/)