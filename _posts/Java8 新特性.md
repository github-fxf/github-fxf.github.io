---
layout:     post
title:      java8新特性学习
subtitle:   
date:       2020-12-02
author:     fxf
header-img: img/post-web.jpg
catalog: true
tags:
    - SQL
---


# java8新特性简介

+ 速度更快

+ 代码更少（增加了新的语法lambda表达式）

- 强大的stream API
- 便于并行
- 最大化减少空指针异常optional

**其中最核心的为lambda表达式和Stream API**

# lambda表达式

## 1. 为什么使用lambda表达式

lambda表达式是一个匿名函数，我们可以把lambda表达式理解为是一段可以传递的代码（将代码像数据一样进行传递）。可以写出更简洁、更灵活的代码 。

## 2. lambda表达式

- 从匿名类到lambda的转换

```
//匿名内部类
Runnable r1 = new Runnable(){
	@Override
	public void run(){
		system.out.println("Hello World");
	}
}

//lambda 表达式
Runnable r1 = ()->System.out.println("Hello Lambda!");
```

##  3. lambda表达式语法

sslambda表达式在java语言中引入了一个新的语法元素和操作符。这个操作符为“->”,该操作符被称为lambda操作符或箭头操作符。它将lambda分为两部分：

**左侧：**指定了lambda表达式需要的所有参数

**右侧：**指定了lambda体，即lambda表达式要执行的功能。

