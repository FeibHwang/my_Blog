---
layout:     post
title:      "Ubuntu+CUDA Installation Guide"
subtitle:   "A basic memo when I install CUDA8.0 on my Ubuntu"
date:       2017-09-12 22:00:00
author:     "飞白"
header-img: "img/post-bg-2015.jpg"
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



# First Thing First
PLEASE PLEASE read the official [CUDA installation guide](http://developer.download.nvidia.com/compute/cuda/7.5/Prod/docs/sidebar/CUDA_Installation_Guide_Linux.pdf)
Check every details including the system requirements and installation status


# GPU requirement Check

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

# Choose instalation methods
There are several methods to install CUDA, I really recommand using runfile
Download the runfile at [Here](https://developer.nvidia.com/cuda-downloads)

# runfile installation
1. Disable nouveau
run command: `$ lsmod | grep nouveau` if something show up, then nouveau is currently loading
To disable it:
Creat file named `blacklist-nouveau.conf` under `/etc/modprobe.d`, adding following content

blacklist nouveau 
options nouveau modeset=0

Then run: `$ sudo update-initramfs –u`
Finally use command `$ lsmod | grep nouveau` again, if no output then you can proceed

2. Text Mode Installation
Reboot your computer, when you enter login, use `alt+ctrl+F1` to enter text mode, then log in.

run `$ sudo service lightdm stop` to close GUI interface

run `sudo init 3`

go to your runfile location, run `$ sudo sh cuda_<your_version>_linux.run --override`

Follow the option, be sure to choose `no` when asking whether to install OpenGL if you have dual graphics card

After Installation, restart lightdm again by running `$ sudo service lightdm start`

Reboot

Env Setting: `$ sudo gedit /etc/profile`, add following content at end of the profile:

export PATH=/usr/local/cuda-8.0/bin:$PATH 
export LD_LIBRARY_PATH=/usr/local/cuda-8.0/lib64


The installation part has finished

# Post Check
run `$ cat /proc/driver/nvidia/version` to print NVIDIA Driver version

run `$ nvcc –V` to print CUDA version

Try to compile CUDA Example
change location to ~/NVIDIA_CUDA-8.0_Samples
run `$ make`

Run CUDA program
change location to ~/NVIDIA_CUDA-8.0_Samples/bin
run `$ sudo ./deviceQuery` to print GPU info

It is possible you get error like “cuda driver version is insufficient for cuda runtime”, this is possibally because your CUDA version and Driver version are not compatiable, use the following command:
run `sudo apt-get purge nvidia*`
run `sudo apt-get install nvidia-current`
run `sudo reboot`
