---
layout: post
title:  "Active Record Associations 4 关联性的细节"
date:   2014-08-13 15:55:55
categories: Rails
tags: Rails Active Record Associations
---

![P1000804](/assets/2014/08/13/P1000804.gif)

下面的部分会给出每一个关联类型的细节，包括他们增加的方法和你可以用来声明关系的操作。

### 4.1 belongs_to关联

belongs_to关联创建了一个one-to-one映射到另一个model。在数据库里，这个关联就是说这个类包含了一个外键。如果另外一个类包含了外键，那么你应该使用has_one来替代。

#### 4.1.1 belongs_to增加的方法
