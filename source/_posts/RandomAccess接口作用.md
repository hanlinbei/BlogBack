---
title: RandomAccess接口作用
categories:
  - 面试
tags:
  - Java
abbrlink: 36690
date: 2020-07-14 21:51:57
---

众所周知，在List集合中，我们经常会用到ArrayList以及LinkedList集合，但是通过查看源码，就会发现ArrayList实现RandomAccess接口，但是RandomAccess接口里面是空的！Linked并没有实现RandomAccess接口。
这是为什么呢？

<!--more-->
这是ArrayList实现RandomAccess接口的源码
{% asset_img 2020-08-20-19-06-10.png %}
这是LinkedList的源码，并没实现RandomAccess接口
{% asset_img 2020-08-20-19-06-33.png %}
这是RandomAccess接口的源码
{% asset_img 2020-08-20-19-06-47.png %}
原来RandomAccess接口是一个标志接口（Marker），然而实现这个接口有什么作用呢？

解答：只要List集合实现这个接口，就能支持快速随机访问，然而又有人问，快速随机访问是什么东西？有什么作用？

通过查看Collections类中的binarySearch（）方法，源码如下：
{% asset_img 2020-08-20-19-07-10.png %}
由此可以看出，判断list是否实现RandomAccess接口来实行indexedBinarySerach(list,key)或iteratorBinarySerach(list,key)方法。ps（instanceof其作用是用来判断某对象是否为某个类或接口类型）

那么，又有人疑问，执行这两个方法有什么不同？
查看下indexedBinarySerach(list,key)方法源码：
{% asset_img 2020-08-20-19-07-44.png %}
查看下iteratorBinarySerach(list,key)方法源码：
{% asset_img 2020-08-20-19-08-06.png %}
通过查看源代码，发现实现RandomAccess接口的List集合采用一般的for循环遍历，而未实现这接口则采用迭代器。

接下来，我们将进行下测试ArrayList以及LinkedList采用这两种方法各自的性能是如何！

main方法：
{% asset_img 2020-08-20-19-08-44.png %}
for循环遍历ArrayList
{% asset_img 2020-08-20-19-08-55.png %}
iterator迭代器遍历ArrayList
{% asset_img 2020-08-20-19-09-07.png %}
for循环遍历LinkedList
{% asset_img 2020-08-20-19-09-20.png %}
iterator迭代器遍历LinkedList
{% asset_img 2020-08-20-19-09-33.png %}
运行结果：
{% asset_img 2020-08-20-19-09-45.png %}
从上面数据可以看出，ArrayList用for循环遍历比iterator迭代器遍历快，LinkedList用iterator迭代器遍历比for循环遍历快，

所以说，当我们在做项目时，应该考虑到List集合的不同子类采用不同的遍历方式，能够提高性能！

然而有人发出疑问了，那怎么判断出接收的List子类是ArrayList还是LinkedList呢？

这时就需要用instanceof来判断List集合子类是否实现RandomAccess接口！
代码如下：
{% asset_img 2020-08-20-19-11-09.png %}
main方法：
{% asset_img 2020-08-20-19-11-22.png %}
运行结果：
{% asset_img 2020-08-20-19-11-34.png %}

总结：RandomAccess接口这个空架子的存在，是为了能够更好地判断集合是否ArrayList或者LinkedList，从而能够更好选择更优的遍历方式，提高性能！