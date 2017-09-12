---
layout:     post
title:      "Ubuntu+CUDA Installation Guide"
subtitle:   "7. Reverse Integer"
date:       2017-09-12 22:00:00
author:     "飞白"
header-img: "img/post-bg-leetcode.jpg"
catalog: true
tags:
    - Ubuntu
    - CUDA
    - Guide
---

Here are the basic step of installing the CUDA-8.0 driver and toolkit on Ubuntu 16.04

My Device: DELL XPS 15 9550
OS: Windows 10 Home + Ubuntu 16.04
Graphics: GTX 960M + Intel HD something
CUDA Version: 8.0

P.S: when your computer has dual graphics card installed like me and you are using non-NVIDIA card for display, DO NOT INSTALL `openGL Library` when you install CUDA



#First Thing First
PLEASE PLEASE read the official [CUDA installation guide](http://developer.download.nvidia.com/compute/cuda/7.5/Prod/docs/sidebar/CUDA_Installation_Guide_Linux.pdf)
Check every details including the system requirements and installation status


#GPU requirement Check

1.CUDA-capable check
Use command: 
`$ lspci | grep -i nvidia`
This command will list your GPU version, compare it with the CUDA support list in NVIDIA CUDA official site

2.Check Whether your Linux version support CUDA

3.Check gcc installation using `$gcc –-version`

4.Check kernel header and pacjage development
Use command:
`$uname –r`
`$ sudo apt-get install linux-headers-$(uname -r)`
to install the correct kernel header

#Choose instalation methods
There are several methods to install CUDA, I really recommand using runfile
Download the runfile at [Here](https://developer.nvidia.com/cuda-downloads)

#runfile installation
