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

![图灵机](http://3.bp.blogspot.com/-O8pU1eLknqg/Uf_2LlOsGWI/AAAAAAAAHoU/COz3NOtX-Q8/s200/turingmachine.gif)

从本质上讲，这种复杂性源自于过程式编程的数学模型：[图灵机](https://en.wikipedia.org/wiki/Turing_machine)，图灵机简单来说包括一个写着0-1字符的纸带与可以读取纸带的机器，这个机器可以读取，修改纸带上的字符，移动到纸带的任意位置，从而实现各种运算。从定义上来说，图灵机是顺序式执行的设备，机器必须根据当前读取到的字符进行下一步的执行，这种执行方式导致了Data Dependency十分明显。那么问题来了：是否有另外一种模型没有那么严重的Data Dependency Issue?

## 函数式编程(Functional Programming)
接着上面的问题，那么究竟有没有这种模型？ 答案是有的，有一种数学模型天生适合做并行化，叫做[Lambda表达式](https://en.wikipedia.org/wiki/Lambda_calculus)。这个数学模型后来在计算机领域衍生成另外一种编程范式，称为函数式编程(Functional Programming)。

首先区分几个概念：
1. Lambda表达式与图灵机是两种数学模型，其中Lambda表达式已经证明是Turing Complete的，意味着Lambda表达式可以模拟图灵机的全部运算。
2. 过程式编程(Sequencial Programming)与函数式编程(Functional Programming)是两种编程范式，分别源自于图灵机与Lambda表达式两种数学模型
3. 编程语言： 不同的编程语言支持不同的编程范式，这也是有各种各样的编程语言的原因之一。过程式编程的代表语言有汇编，B, C；函数式编程的代表语言有Haskell, Scala, Lisp等。当然现代编程语言会支持多种编程范式，C++, Python, Javascript等就支持以上两种编程范式，但是它们又有不同的侧重，这里就不详谈了。

另外，本文章将使用Haskell作为函数式编程实例语言，一方面是因为Haskell是纯函数式编程语言，其内部实现严格遵循了Lambda表达式的演算方式，另一方面作者认为Haskell是一门逼格很高的语言......嗯。

### 继续看快速排序

回顾一下Quick Sort的表述：随机选择数组中一个数，将数组分成两部分，一部分所有元素小于等于该数，另一部分所有元素大于该数；对两个数组递归调用快速排序，之后将整个数组整合起来。

我们可以换一种表述：
1.Quick Sort是一个函数，这个函数接受一个数组，输出一个排好序的数组
2.Quick Sort (array)= array1 ++ a ++ array2
                      a: array的第一个元素
                      array1: Quick Sort [array中所有小于a的元素组成的数组]
                      array2: Quick Sort [array中所有大于等于a的元素组成的数组]

我们可以看出两种表述方式很相近，第二种表述方式描述了快速排序作为一个函数的行为。

让我们看一下Haskell表示的快排：

{% highlight hs %}

qsort [] = []
qsort (x:xs) = qsort (filter (< x) xs) ++ [x] ++ qsort (filter (>= x) xs)

{% endhighlight %}

Bang!就是这么简单!

稍微解释一下：
第一句表示如果传入一个空数组，那么就直接输出，这是coner case
第二句描述了整个快排，filter是一个高阶函数，`filter (< x) xs`表示将数组xs中小于x的数保留，其他删除。

可以看出Haskell版本的Quick Sort基本上就是第二种表述的翻版，这表示相比起C++的版本，Haskell版快排的可读性更加友好。更重要的是我们可以很快发现并行性：
`qsort (filter (< x) xs)`, `[x]`与`qsort (filter (>= x) xs)`作为级联关系是可以并行的，不能并行的数组拆分与排序在代码中变成了函数嵌套，这是很重要的特性！在后面我会仔细解释。

### 万物皆是函数
在函数式编程中，函数是一等公民，一切都可以用函数表示，这里的函数已经不仅仅局限于我们所理解的函数，而是一种映射关系。再看看Haskell版本的快速排序，你能识别出多少函数？

* qsort: 主函数，输入一个数组，输出一个数组
* (<x), (>=x): 判断大小函数，输入一个数字，将其与x比大小，输出Bool值 
* filter: 之前已经说明，是一个高阶函数，输入一个函数与一个数组，输出一个数组
* (++): 嗯，也是函数，输入两个数组，输出一个数组(级联)
* ([]): 。。。也是函数，或确切的说是一个Monad，输入一堆数，将他们打包(wapped)成数组

从这里我们可以大概感受到函数式编程中“万物皆是函数”的定义，从Lambda表达式的角度来说，常量，符号，数字等都是函数，在这里我不准备详谈，有机会另写。

## 正题：函数式编程的并行化优势

### 优势一：引用透明(Referential transparency)

引用透明是纯函数(Pure Function)的特性之一，纯函数源自于数学上对于函数的定义：函数的输出仅取决于其输入。或是我们常用的公式定义：`y=f(x)`。
相比于数学上的函数，程序中的函数更像是过程(procedure)。对与C++来说，观察一下下面3种函数：

{% highlight c++ %}

int f1(int x)
{
	x = x + 1;
	return 10;
}

int f2(int & x)
{
	x = x + 1;
	return 10;
}

int f3(int * x)
{
	*x = *x + 1;
	return 10;
}

{% endhighlight %}

3种函数从返回值上来说，都做了一样的事：返回常量10。此外，3种函数还做了另外一件事：将传入的参数+1，其中参数传入方式分别是值传递，引用传递，指针传递。效果也是十分明显的，如果分别执行完这三种函数后我们继续打印x的值得话，可以发现：`f1`中的x没变，而`f2``f3`函数将x的值+1了，换句话说，`f2``f3`函数除了返回一个常量，还做了其它行为，修改了一些“外部”状态。这种现象称为函数的“副作用”。

对于编程来说，函数的副作用有很多种，比如全局变量，引用传递，IO操作等。现在我们再重新定义一下纯函数，纯函数具有如下特征：

1. 函数如果传入相同参数，返回值不变
2. 函数不会产生副作用

举个例子，`sqrt(x)`是一个纯函数，`launch_nuclear_missile()`是一个有很大副作用的函数(╯-╰)

从定义上看纯函数是有很多优点的，其中最重要的是：纯函数没有状态(state)，意味着纯函数是“线程安全的”，因而不需要进行同步。当然其缺点也很明显：没有什么卵用!=_=, 我们仔细想一想，大部分编程活动是需要副作用的，比如我们要把数据打印到屏幕上的，进程之间是需要进行通讯的，等等等等。那么有什么解决方法？很简单，只要将算法中需要副作用的部分分离开就可以了。

#### 传统数据结构的“副作用”

过程式编程有很多经典的数据结构，现在我们从纯函数的角度出发，看看哪些行为是有副作用的：

| 数据结构 | List:链表 | Stack:栈 | Queue:队列 | Graph:图 | Tree:树 |
|---------|-----------|---------|------------|----------|---------|
|无副作用：Safe|构建，搜索|构建，Push,Top|构建，Push,Front|构建|构建，搜索|
|有副作用：unsafe|删除，插入，反向|Pop|Pop|删除，搜索|删除，Invert,平衡|

数据结构是算法的基础，人们设计这些数据结构本身是为了优化一些带有副作用的行为的，这些行为优化了算法复杂度，但也由于引入复杂度而导致并行复杂化。
大部分函数式编程采用最原始的数组为核心数据对象，当然函数式编程也是有自己的数据结构的，比如说Functional Programming领域中近似于“算法导论”存在的论文[Pure Functional Data Structures](https://www.cs.cmu.edu/~rwh/theses/okasaki.pdf)，建议所有对函数式编程感兴趣的人读一下。

### 优势二：类型抽象 Abstraction Hierarchy

### 优势三：惰性求值 (Haskell 独有)

### 优势四：Lambda演算的并行优势(以后有机会再写。。。)