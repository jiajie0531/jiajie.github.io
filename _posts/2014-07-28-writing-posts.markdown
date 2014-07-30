---
layout: post
title:  "撰写博客"
date:   2014-07-28 21:14:29
categories: jekyll
tags: jekyll
---
 
Jekyll最核心的一点就是“专注博客”。这意味着什么呢？简单的来说，它意味着博客就是Jekyll的核心功能。如果你撰写文章，然后在网上发布出来，这也就是说，你可以简单地来发布和维护一个博客，只要在你的电脑上管理一个文本文件夹即可。相比较配置的繁琐，维护数据库和基于web的CMS系统，这将会是一个受欢迎的改变！

### 博客的文件夹

正如在[目录结构](http://jekyllrb.com/docs/structure/)页面所解释的，**_posts**文件夹就是你的博客文章所存放的地方。这些文件可以是[Makedown](http://daringfireball.net/projects/markdown/)或者[Textile](http://redcloth.org/textile)格式的文本文件，只要他们有[YAML front-matter](http://jekyllrb.com/docs/frontmatter/)，他们将会从他们代码格式被转换成HTML页面，作为你的静态网站的一部分。

### 创建博客文件

为了创建一个新的博客，你所要做的就是创建一个新的文件在**_posts**目录里。在这个文件夹里你如何命名文件名是很重要的。Jekyll需要被命名的博客文件名需要遵守下面的格式：
{% highlight html %} 
{% raw %}
YEAR-MONTH-DAY-title.MARKUP
{% endraw %}
{% endhighlight %}

这里**YEAR**是四位数字，**MONTH**和**DAY**都是两位数字，以及**MARKUP**是文件扩展名，表示在文件里所使用的格式。例如，下面的例子就是有效的博客文件名：
{% highlight html %} 
{% raw %}
2011-12-31-new-years-eve-is-awesome.md
2012-09-12-how-to-write-a-blog.textile
{% endraw %}
{% endhighlight %}

>ProTip<sup>TM</sup>: 关联其他的博客
>
>使用**post_url**标签来链接其他的博客，不需要担心URL的失联，在当网站的永久链接改变的时候。

### 文件内容格式
所有博客文件必须以[YAML front-matter](http://jekyllrb.com/docs/frontmatter/)为开头。然后，就很简单地选择你所需要的格式。Jekyll提供了两种比较流行的标记格式：[Markdown](http://daringfireball.net/projects/markdown/)和[Textile](http://redcloth.org/textile)。每一个格式有他们自己的标记方式，属于各自不同的内容格式，因此你应该选择你自己熟悉的格式，决定哪一个格式更适合你的需求。

>Be aware of character sets 注意字符集的设置
>
>处理内容能够使得某些字符变得更美观。例如，在Redcarpet里**smart**扩展是转换标准，ASCII引用字符变化为Unicode字符。为了合理地浏览来显示那些字符，在你的布局文件**<head>**中定义字符集的值为 \<meta charset="utf-8"> 

### 包含图片和资源
在某一时刻，你将会想包含图片，下载文件，或则其他的数码资源在你的文本内容中，这些都是可以实现的。当涉及到关联资源的语法在Markdown和Textile两者之间是不一样的，问题在于找出存放这些文件在你网站里的位置，这是一些每个人都会碰到的事儿。

因为Jekyll的灵活性，关于如何来实现它是由很多解决方案。一个最通常的方案是在项目的根目录下创建一个文件夹，取名为**assets**或则**downloads**类似这样的，里面有图片，下载的内容或者其他放置的资源。然后，对于所有的博客，他们能够通过使用网站的root被关联，其路径就是包含这些资源。这种方式再一次被证明是可依赖的，你网站的（子）域名和路径是可被设置的，但这下面一些例子（用Markdown）是你如何可以用博客里的**site.url**变量来实现这个。

在一篇博客里包含一个图片资源：
{% highlight html %} 
{% raw %}
… which is shown in the screenshot below:
![My helpful screenshot]({{ site.url }}/assets/screenshot.jpg)
{% endraw %}
{% endhighlight %}

链接一个PDF让读者来下载：
{% highlight html %} 
{% raw %}
… you can [get the PDF]({{ site.url }}/assets/mydoc.pdf) directly. 
{% endraw %}
{% endhighlight %}

>ProTip<sup>TM</sup>: Link using just site root URL
>链接使用的是网站的根URL
>
>你能够跳过**\{\{site.url\}\}**变量，如果你知道你网站不会单单只显示域名的主路径。在这个例子里，你可以直接通过**/path/file.jpg**来引用资源。

### 显示博客的索引
把拥有的博客放在一个文件夹里是比较好的，但一个博客没有使用除非你有一个关于博客的列表。在另外的页面上（或者在一个[template](http://jekyllrb.com/docs/templates/)）创建一个博客的索引是比较简单的，这得归结于[Liquid template language](http://docs.shopify.com/themes/liquid-documentation/basics)和它的标签。这里有一个基本的例子，关于如何来创建一系列的博客文章的链接：
{% highlight html %} 
{% raw %}
<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>
{% endraw %}
{% endhighlight %}
当然，你可以完全控制如何（何处）来显示你的博客，以及如何你网站的结构。如果你想知道更多的话，你可以了解更多关于[how templates work](http://jekyllrb.com/docs/templates/)如何用Jekyll来执行网站模板化。

### 文章摘录
每一篇博客自动得到其文本的第一段，从内容的开始处到**excerpt_separator**的首个出现的位置，把它设置为**post.excerpt**。获取上面例子中博客的索引。可能你想要包含一小段关于博客内容的提示，可以通过增加你每一篇博客的第一段落：
{% highlight html %} 
{% raw %}
<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
      {{ post.excerpt }}
    </li>
  {% endfor %}
</ul>
{% endraw %}
{% endhighlight %}
因为Jekyll抓取的第一段落，你将不会需要把摘录打包到**p**标签里，但事实是已经帮你打包进去了。如果你纠结的话，这些标签能够被移出：
{% highlight html %} 
{% raw %}
{{ post.excerpt | remove: '<p>' | remove: '</p>' }}
{% endraw %}
{% endhighlight %}
如果你不喜欢自动生成的博客索引，他能够通过增加**excerpt**到你博客的YAML front-matter里用来被重写。通过把你的**excerpt_separator**设置为**""**来完全屏蔽。

同样，随着Liquid标签生成的输出内容，你可以传入**\| strip_html**标志来移出输出内容中的任何html标签。这是相当有用的，如果你希望输出一个博客的摘录，那么用一个**meta="description"**标签放进博客的**head**，或者任何位置，其他的在内容中有html标签是没什么吸引力的。

### 高亮代码片段
对于代码块的语法高亮Jekyll也有内建支持，可以使用Pygments或者Rouge，在任何博客中包含一个代码片段是容易的。只要使用对应的Liquid标签： 
{% highlight text %} 
{% raw %}
{% highlight ruby %}
def show
  @widget = Widget(params[:id])
  respond_to do |format|
    format.html # show.html.erb
    format.json { render json: @widget }
  end
end
{% endhighlight %}
{% endraw %} 
{% endhighlight %}
然后输出的结果是:
{% highlight ruby linenos %}
def show
  @widget = Widget(params[:id])
  respond_to do |format|
    format.html # show.html.erb
    format.json { render json: @widget }
  end
end
{% endhighlight %}

>ProTip<sup>TM</sup>: Show line numbers
>
>通过增加参数**linenos**，你可以使得代码块显示行数，在打开高亮标签的结尾，类似于：{% raw %}{% highlight ruby linenos %}{% endraw %} 

这些基本的能够满足你开始撰写你的第一篇博客。当你准备好进一步挖掘细节，你可以有兴趣来做[customizing post permalinks](http://jekyllrb.com/docs/permalinks/)或者在你博客中或者你的网站里使用[custom variables](http://jekyllrb.com/docs/variables/)。


[writing posts](http://jekyllrb.com/docs/posts/)