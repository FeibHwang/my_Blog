---
layout:     post
title:      "函数式编程的并行化优势"
subtitle:   "函数式编程更加符合人的思维方式"
date:       2017-04-28 17:00:00
author:     "飞白"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 函数式编程
    - Haskell
    - 并行编程
---

> 图灵机不适合并行执行

## 前言

费了很大劲配置好了Jekyll + Gitblog，却不知道写些什么，正好前几天Parallel Computer System的课程刚刚结束，趁着还有印象把自己准备的Presentation总结一下。

## 过程式编程的流程

先看一下经典的快速排序的C++实现：

{% highlight c++ %}

int partition(vector<int> &nums, int p, int r)
{
	int x = nums[r];
	int i = p - 1;
	for (int j = p; j < r; ++j)
	{
		if (nums[j] <= x)
		{
			swap(nums[++i], nums[j]);
		}
	}
	swap(nums[++i], nums[r]);
	return i;
}

void quick_sort(vector<int> &nums, int p, int r)
{
	if (p < r)
	{
		int q = partition(nums, p, r);
		quick_sort(nums, p, q - 1); 
		quick_sort(nums, q + 1, r);
	}
}

{% endhighlight %}

其中`quick_sort`函数中`partition`函数必须先执行，之后的两个`quick_sort`函数可以并行执行。快速排序的原理很简单：随机选择数组中一个数，将数组分成两部分，一部分所有元素小于等于该数，另一部分所有元素大于该数；对两个数组递归调用快速排序，之后将整个数组整合起来。

从定义上也可以看出：快速排序中拆分数组必须先执行，之后的排序是不相关的。算法的定义决定了算法的并行方式，从人的角度来说，我们可以很快从快速排序的描述中定义并行方式，但是如果是从算法或代码的角度观察，并行性会变得十分隐蔽。

### 什么决定了算法并行性？
* 数据依赖性(Data Dependency)
从本质上来说，算法中的数据依赖性制约了并行化，快排中的两个子快排的对象是由`partition`函数产生的，因而必须等数组拆分完才能调用。另一个例子是归并排序(merge sort)，由于归并的必须是两个排好序的数组，所以归并排序的并行优化是“先并行，后顺序”。

* 变量修改(Variable Modification)
数据依赖性导致了算法并行极限。在代码中数据依赖体现在函数共享变量或内存地址，然而由于程序员的编程习惯，很多可以实现并行的算法由于共享变量等原因而无法并行执行，这导致了代码并行优化的复杂度提升。

### Sequencial Programming Parallelism常用技术与局限
为了解决数据依赖与共享内存缺陷，人们引入了Mutex, Semaphore, Locks, Synchronization等技术，使得过程式编程并行化变得十分复杂。很多介绍并行化编程的书有几乎一大半的篇幅被用于介绍同步，消息互锁等概念，由此可以看出并行化编程之复杂。

从本质上讲，这种复杂性源自于过程式编程的数学模型：[图灵机](https://en.wikipedia.org/wiki/Turing_machine)，图灵机简单来说包括一个写着0-1字符的纸带与可以读取纸带的机器，这个机器可以读取，修改纸带上的字符，移动到纸带的任意位置，从而实现各种运算。从定义上来说，图灵机是顺序式执行的设备，机器必须根据当前读取到的字符进行下一步的执行，这种执行方式导致了Data Dependency十分明显。那么问题来了：是否有另外一种模型没有那么严重的Data Dependency Issue?

## 函数式编程(Functional Programming)
接着上面的问题，那么究竟有没有这种模型？ 答案是有的，有一种数学模型天生适合做并行化，叫做[Lambda表达式](https://en.wikipedia.org/wiki/Lambda_calculus)。这个数学模型后来在计算机领域衍生成另外一种编程范式，称为函数式编程(Functional Programming)。

首先区分几个概念：
1. Lambda表达式与图灵机是两种数学模型，其中Lambda表达式已经证明是Turing Complete的，意味着Lambda表达式可以模拟图灵机的全部运算。
2. 过程式编程(Sequencial Programming)与函数式编程(Functional Programming)是两种编程范式，分别源自于图灵机与Lambda表达式两种数学模型
3. 编程语言： 不同的编程语言支持不同的编程范式，这也是有各种各样的编程语言的原因之一。过程式编程的代表语言有汇编，B, C；函数式编程的代表语言有Haskell, Scala, Lisp等。当然现代编程语言会支持多种编程范式，C++, Python, Javascript等就支持以上两种编程范式，但是它们又有不同的侧重，这里就不详谈了。

另外，本文章将使用Haskell作为函数式编程实例语言，一方面是因为Haskell是纯函数式编程语言，其内部实现严格遵循了Lambda表达式的演算方式，另一方面作者认为Haskell是一门逼格很高的语言......嗯。