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

NVIDIA CUDA TOOLKIT DOCUMENTATION [第二章](http://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#programming-model)笔记/翻译

# [Kernels](http://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#kernels)

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

# [Thread Hierachy](http://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#thread-hierarchy)

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

注意，Block是被设计为可以独立的，意味着每一个Block是可以以任意顺序执行的，并行或顺序。进程调度程序会自动分配这些Block, 如下图所示：

![Block Schedule](http://docs.nvidia.com/cuda/cuda-c-programming-guide/graphics/automatic-scalability.png).

Block中的线程通过共享内存空间实现线程通信，这就需要进行线程同步。通过在kernel中调用`__syncthreads()`实现同步。`__syncthreads()`就像是一道屏障，当所有的线程都越过以后才会继续运行。共享内存的例子可以看[这里](http://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#shared-memory).

共享内存要求低延时（L1 cache）, `__syncthreads()`要求程序轻量执行。

# [Memory Hierachy](http://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#memory-hierarchy)

![Memory Hierarchy](http://docs.nvidia.com/cuda/cuda-c-programming-guide/graphics/memory-hierarchy.png).

上图展示了CUDA线程可访问的内存地址空间，每一个线程都有一个自己的私有内存空间。每一个Block又有一个自己的内存空间共自己的所有线程共享，其存在周期与Block自己一致。同时所有线程又共享一段全局内存空间(global memory).

同时我们又有两个只读内存可供所有线程访问：Constant Memory与Texture Memory.两种内存对于不同的内存使用[策略](http://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#device-memory-accesses)进行了优化。Texture Memory同时提供了不同的地址访问方式，以及数据筛选。详细内容看[这里](http://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#texture-and-surface-memory)。

对于同一个kernel或应用, 其全局内存，constant memory, texture memory是预先分配且确定的。

# [Heterogeneous Programming](http://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#heterogeneous-programming)

![Heterogeneous Programming](http://docs.nvidia.com/cuda/cuda-c-programming-guide/graphics/heterogeneous-programming.png).

如上图所示，CUDA编程模型假设CUDA线程是运行在一个协处理器中的，这个协处理器是与一个运行C主程序的处理器分离的。对应于电脑中CPU与GPU的分离。

CUDA编程模型同时还假设主设备与协处理器有其自己的DRAM, 分别称为`host memory`和`device memory`.因此，一个程序通过kernel调用控制着global,constant,texture memory的访问权限（不太确定本意），这部分内收收录在[这里](http://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#programming-interface)。这包括了设备内存管理与host/devise设备切换。

# [Compute Capability](http://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#compute-capability)

一个设备(device)的版本号(version number)表示了这个设备的计算性能(compute capability), 有时版本号又称`SM version`.这个版本号指明了GPU硬件所支持的功能，程序在执行时可以获取这个号码已确定哪些指令是可以使用的。

版本号由一个主版本号X和一个副版本号Y组成，设备的主版本号X相同意味着两者的`core architecture`是一样的： 
* 5： 核心基于Maxwell架构
* 3： 核心基于Kepler架构
* 2： 核心基于Fermi架构
* 1： 核心基于Tesla架构

设备的副版本号表示核心架构的增强版本，意味着一些新的feature的加入

详细的版本号在NVIDA官网查询到：[CUDA-Enabled GPUs](http://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#cuda-enabled-gpus),[Compute Capabilities](http://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#compute-capabilities).
