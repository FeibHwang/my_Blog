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

// Kernel definition 

__global__ void VecAdd(float* A, float* B, float* C) 
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

线程的index与其ID通过一直十分直接的方式关联： 
* 一维block: index == ID 两者相同
* 二维block, size为(Dx,Dy): (x,y)的`thread ID` = x + y*Dx
* 三维block, size为(Dx,Dy,Dz): (x,y,z)的`thread ID` = x + y*Dx + z*Dx*Dy

Example: 两个NxN的矩阵加法与存储

{% highlight c %}

// Kernel definition 

__global__ void MatAdd(float A[N][N], float B[N][N], float C[N][N]) 
{ 
	int i = threadIdx.x; 
	int j = threadIdx.y; 
	C[i][j] = A[i][j] + B[i][j]; 
} 

int main() 
{ 
	... 
	// Kernel invocation with one block of N * N * 1 threads 
	int numBlocks = 1; 
	dim3 threadsPerBlock(N, N); 
	MatAdd<<<numBlocks, threadsPerBlock>>>(A, B, C);
	... 
}


{% endhighlight %}

每一个`block`的大小是有限制的，因为block中所有线程理论上是共享处理器与内存的，现在的GPU中，一个block的内存数为1024.

由于一个kernel中可能有多个相同维度的block,因此kernel的总线程数等于block数乘以每个block中的线程数。

同时，`Block`也被划分为1~3维的`grid`，如下图所示

![Grid-Block](http://docs.nvidia.com/cuda/cuda-c-programming-guide/graphics/grid-of-thread-blocks.png)

一个Grid中的Block数取决于问题的数据复杂度，是可以改变的。

`Block`中的线程数与`Grid`中的Block数都是通过`<<<...>>>`语法定义的`int`或`dim3`声明的。上图声明了一个2-D的Block或Grid.

Block在Grid中的Index通过`blockIdx`变量获取，Block的线程维度通过`blockDim`获取。

现在通过扩展之前的`MatAdd()`例子，我们可以处理多个Block:

{% highlight c %}

// Kernel definition 

__global__ void MatAdd(float A[N][N], float B[N][N], float C[N][N]) 
{ 
	int i = blockIdx.x * blockDim.x + threadIdx.x; 
	int j = blockIdx.y * blockDim.y + threadIdx.y; 
	if (i < N && j < N) C[i][j] = A[i][j] + B[i][j]; 
} 

int main() 
{ 
	... 
	// Kernel invocation 

	dim3 threadsPerBlock(16, 16); 
	dim3 numBlocks(N / threadsPerBlock.x, N / threadsPerBlock.y); 
	MatAdd<<<numBlocks, threadsPerBlock>>>(A, B, C); 
	...
}


{% endhighlight %}