---
layout:     post
title:      "复杂的C/C++ 变量声明读取"
subtitle:   "识别复杂C/C++变量的基本步骤"
date:       2017-05-05 07:00:00
author:     "飞白"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - C/C++
    - 编程
    - 翻译
    - 学习笔记
---

C style 的变量声明有时候会很难读取，比如下面几种：

{% highlight c %}

int (*p)[ 5 ];   
char *x[ 3 ][ 4 ];
int *(*a[ 10 ])();
int (*t[])();

{% endhighlight %}

有一种比较简单的识别方式：

从变量名（或标识符）开始。在不跳过右括号的前提下，看看右边是什么。

在不跳过左括号的前提下，看看左边有什么。

如果是在括号内，跳出括号。

再看看右边和左边

注意： 始终记得函数与数组是从左到右读，且是优先于指针的，指针是从右向左读。

回到之前的例子：

`int (*p)[ 5 ]` 
先找到标识符`p`，右边是括号，看左边。左边是指针符，所以p是一个指针。
跳出括号看右边，我们看到数组index 5，所以`p`是一个`指向一个size为5的数组的指针(a pointer to an array of 5 ints)` 

`char *x[ 3 ][ 4 ]` 
先找到标识符`x`，向右看，我们看到了`[ 3 ][ 4 ]`，表示3个size为4的数组，注意数组的优先值大于指针，所以我们现实别出二维数组。接着看左边，是一个指针符，因此变量`x`是一个`3x4的指向字符的指针数组(x is an array composed of 3 arrays of 4 pointers to char)`

`int *(*a[ 10 ])()` 
先找到标识符`x`,向右看，得到数组index,向左看，是指针，所以是指针数组。跳出括号，左边是一个function,右边是指针，表示指向函数的指针。
因此，`a`是`一个size为10的指针数组，指向返回int的函数(x is an array composed of 3 arrays of 4 pointers to char)`


`int (*t[])()` 
先找到标识符`t`，向右看，表示不定长数组（指针），左边是指针，还是指针数组。跳出括号，右边是function,左边是返回值int。所以`t`是`一个指针数组，指向返回int的函数(an array of pointers to functions returning an int)`
