---
layout: post
title:  "table view"
date:   2015-05-28 13:05:55
categories: ios
tags: ios swift tableView
---

## Index paths

你已经看到table view为了一个cell去获取**data source**，使用的方法是tableView(cellForRowAtIndexPath)。

因此这个index-path到底是什么呢？

NSIndexPath简而言之就是一个object，表示的是一个table中的一行。它是一个行和分组的结合。当一个table view为了一个cell而需要data source，在indexPath.row属性里，你能够找到所属的哪一行，想要的哪个cell。

它也能够把table的那些行都进行分组。在地址簿的app里，你可能想要把联系人按照姓做一下排序。所有姓以“A”开头的联系人都被单独排在一块儿，所有姓以“B”开头的联系人被放在另外一块儿，等等。

为了寻找哪一行属于哪个分组，你得在indexPath.section的属性里来操作。清单app还没有分组的需求，因此你可以在NSIndexPath里忽视`section`属性。

也就是说，"NS"前缀，NSIndexPath意味着这个object是由Foundation framework所提供的。NS是NextStep，一个1990's后期的操作系统，也就是后来的Mac OS X，同样也是iOS的源头。
 
## the delegation pattern 
#### 委托模式

委托的概念在iOS里非常普通的。一个object会经常性地依赖另一个object来帮助它来处理某些任务。松耦合*separation of concerns*使得系统变得简单，每个object只做它关注的事，让其他object去做剩余的事情。这个table view就提供了关于委托的例子。

因为每个app有它自己的需求，有着它自己的数据内容，table view必须能够处理大量的不同类型的数据。而不是使得table view更为复杂，或者需要你去修改它来适合你自己的app，UIKit设计者决定使用委托来负责填充这些cell到另一个object，即data source。

table view真的不关心它的data source是谁的，或者你的app处理何种类型的数据，仅仅是能够向cellForRowAtIndexPath发送消息，以及它将接收一个所返回的cell。这使得table view的组件变得简单，转移用来处理数据的功能，这个功能原本是属于你的代码里。

同样地，当用户去点击row的时候，table view知道如何去识别，但在response里应该做什么则完全取决于app。在这个app里，你将会触发这个checkmark；其他的app可能做的是完全不一样的事情。

使用委托系统，table view能够简单地发送一个消息，就是触发了一个tap的动作，然后让委托机制来处理后续的。



