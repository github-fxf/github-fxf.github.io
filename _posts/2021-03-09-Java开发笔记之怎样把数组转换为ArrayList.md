---
layout:     post
title:      Java开发笔记之怎样把数组转换为ArrayList
subtitle:   
date:       2021-03-09
author:     fxf
header-img: img/post-web.jpg
catalog: true
tags:
    - java开发笔记
---


# Java开发笔记之怎样把数组转换为ArrayList

本文分析了Stack Overflow上最热门的的一个问题的答案，提问者获得了很多声望点，使得他得到了在Stack Overflow上做很多事情的权限。这跟我没什么关系，我们还是先看看这个问题吧。 
这个问题是”在Java中怎样把数组转换为ArrayList?”

```
Element[] array = {new Element(1),new Element(2),new Element(3)};
```

### 1.最流行也是被最多人接受的答案

最普遍也是被最多人接受的答案如下：

```
ArrayList<Element> arrayList = new ArrayList<Element>(Arrays.asList(array));
```

首先，我们来看下ArrayList的构造方法的文档。 
ArrayList(Collection < ? extends E > c) : 构造一个包含特定容器的元素的列表，并且根据容器迭代器的顺序返回。 
所以构造方法所做的事情如下：

1. 将容器c转换为一个数组
2. 将数组拷贝到ArrayList中称为”elementData”的数组中

ArrayList的构造方法的源码如下：

```java
public ArrayList(Collection<? extends E> c) {
       elementData = c.toArray();
       size = elementData.length;
 
       if (elementData.getClass() != Object[].class)
             elementData = Arrays.copyOf(elementData, size, Object[].class);
}
```

### 2.另外一个流行的答案

另外一个流行的答案是：

```
List<Element> list = Arrays.asList(array);
```

这不是最好的，因为asList()返回的列表的大小是固定的。事实上，返回的列表不是java.util.ArrayList，而是定义在java.util.Arrays中一个私有静态类。我们知道ArrayList的实现本质上是一个数组，而asList()返回的列表是由原始数组支持的固定大小的列表。这种情况下，如果添加或删除列表中的元素，程序会抛出异常UnsupportedOperationException。

```
list.add(new Element(4));
Exception in thread "main" java.lang.ClassCastException: java.util.Arrays$ArrayList cannot be cast to java.util.ArrayList
    at collection.ConvertArray.main(ConvertArray.java:22)
```

### 3.又一个解决方案

这个解决方案由Otto提供

```
Element[] array = {new Element(1), new Element(2)};
List<element> list = new ArrayList<element>(array.length);
Collections.addAll(list, array);
```

### 4.该问题的表明的东西

这个问题不难，但却很有趣。每个Java程序员都知道ArrayList，但却很容易犯下这样的错误。我想这就是这个问题很火的原因吧。如果是一个特定领域的Java库的相似的问题，就远不会这样火热了。

这个问题有好多答案提供了相同的解决方案，对于StackOverflow的其他问题也是这样。我猜想当人们想要回答一个问题时，是不会管别人说了什么的

 摘自：[在Java中怎样把数组转换为ArrayList?](https://www.cnblogs.com/liushaobo/p/4423102.html) 