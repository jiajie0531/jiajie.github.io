---
layout: post
title:  "创建页面"
date:   2014-07-28 22:32:27
categories: jekyll
---

这是[writing posts](http://jekyllrb.com/docs/posts/)的加强版，你可能想要处理Jekyll网站的另外一个东西，那就是创建静态页面。Jekyll的简单之处就是只要拷贝文件和目录即可。

### 主页面

就是关于每一个你设置的web服务配置，就是在网站的主目录下去查找一个HTML文件，名字叫**index.html**(这是约定俗成的)，然后显示出来，这就是主页面。除非你使用的web服务是被默认配置为其他不同的文件名，这个文件将会是你的Jekyll-generated网站的主页面。

>ProTip<sup>TM</sup>: Use layouts on your homepage
>
>在你主页面上使用布局
>在你网站里的任何HTML文件能够使用布局文件或者includes，甚至是主页面。通常的内容，类似header和footers，非常适合抽象出来形成一个布局文件。

### 额外的页面存在于何处

你把页面的HTML文件放在何处，依赖于你像让页面如何去工作。这里有两种主要的方式用来创建页面：
* 把每一个页面所命名的HTML文件放置在你网站的根目录下。
* 在网站的根目录下，为每一个页面创建一个文件夹，然后在每一个页面文件夹下放置一个index.html文件。
两种方法都可以（两个也可以结合起来使用），两者的不同之处是在于URLs的显示。

### 命名HTML文件
