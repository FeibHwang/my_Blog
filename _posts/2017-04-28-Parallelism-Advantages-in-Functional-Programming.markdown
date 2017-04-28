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

{% highlight C++ %}
if __name__ =='__main__':
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

###什么决定了算法并行性？
* 数据依赖性(Data Dependency)
从本质上来说，算法中的数据依赖性制约了并行化，快排中的两个子快排的对象是由`partition`函数产生的，因而必须等数组拆分完才能调用。另一个例子是归并排序(merge sort)，由于归并的必须是两个排好序的数组，所以归并排序的并行优化是“先并行，后顺序”。

* 变量修改(Variable Modification)
数据依赖性导致了算法并行极限。在代码中数据依赖体现在函数共享变量或内存地址，然而由于程序员的编程习惯，很多可以实现并行的算法由于共享变量等原因而无法并行执行，这导致了代码并行优化的复杂度提升。

###并行化常用技术
为了解决数据依赖与共享内存缺陷，人们引入了Mutex, Semaphore, Locks, Synchronization等技术，使得过程式编程并行化变得十分复杂。很多介绍并行化编程的书有几乎一大半的篇幅被用于介绍同步，消息互锁等概念，由此可以看出并行化编程之复杂。