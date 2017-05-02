---
layout:     post
title:      "CUDA学习笔记（二）"
subtitle:   "编程模型"
date:       2017-05-2 12:00:00
author:     "飞白"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - CUDA
    - 并行编程
    - 翻译
---

NVIDIA CUDA TOOLKIT DOCUMENTATION [第三章](http://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#programming-interface)笔记/翻译

CUDA C包括了一个C语言的扩展以及一个运行时的库，扩展已经在[Programming Model](http://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#programming-model)一节中展示，其中包括让程序员在kernel中定义C函数并用一些新的语法来声明`Grid`与`Block`维度。完整的C扩展可以在[这里](http://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#c-language-extensions)看到。所有的源代码必须使用`nvcc`编译。

运行时库提供了一系列C函数进行`host`到`device`的数据内存迁移，内存空间申请，`device`管理等函数。运行时库建立在底层C API与CUDA driver API之上，同时可以通过程序调取。运行库API通过调取CUDA上下文与CUDA模块提供了更高一层的控制权限，其中CUDA上下文(CUDA Context)是主进程对于kernel进程的抽象，CUDA模块(CUDA Module)是各种动态链接库对于`device`的抽象。对于大部分程序，这多余的一层控制是不需要的。

# [Compilation with NVCC](http://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#compilation-with-nvcc)