---
layout:     post
title:      "CUDA学习笔记（一）"
subtitle:   "编程模型"
date:       2017-04-30 17:00:00
author:     "飞白"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - CUDA
    - 并行编程
    - 翻译
---

# Kernels

`kernels`是CUDA C对C的延伸内容之一，当kernel函数被调用时，它会在N个CUDA线程中执行N次。

kernel函数通过`__global__`声明标识，CUDA的线程数分配则通过一种新的语法结构`<<<...>>>`标识，可以参考[C Language Extensions](http://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#c-language-extensions). 每一个执行kernel函数的线程会被分配一个独一无二的`thread ID`用以区分，这个ID可以通过内置的`threadIdx`变量访问获得。

以下代码展示了相加两个长度为N的数组A和B,并将数组存储到数组C中的实现：

{% highlight c %}

// Kernel definition __global__ void 
VecAdd(float* A, float* B, float* C) 
{ 
	int i = threadIdx.x; 
	C[i] = A[i] + B[i]; 
} 

int main() 
{ 
	... 
	// Kernel invocation with N threads 
	VecAdd<<<1, N>>>(A, B, C); 
	... 
}

{% endhighlight %}

# Thread Hierachy

为了方便，`threadIdx`被写成了一个3维数组，这样线程可以被识别为1~3维的线程架构，称为`thread block`。这样就提供了一个有利于进行数值，数组，矩阵运算的抽象。