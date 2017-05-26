---
layout:     post
title:      "Kotlin学习笔记(一)：安装"
subtitle:   "Kotlin笔记"
date:       2017-05-26 12:00:00
author:     "飞白"
header-img: "img/post-bg-leetcode.jpg"
catalog: true
tags:
    - Kotlin
---

最近Kotlin语言突然变得很火，主要原因是因为Google在I/O 2017上宣布Kotlin正式成为Android开发语言。因此对这一们新兴语言很是好奇，这一系列学习笔记是作者边学边写的。

# 安装

Kotlin依然是基于JVM的，因此首先需要安装JDK，安装地址在[这里](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)，安装完之后可能需要在Path中配置环境变量信息，可以参考这篇[文章](http://jingyan.baidu.com/article/6dad5075d1dc40a123e36ea3.html)。

之后是安装Kotlin的IDE,这里我使用了[IntelliJ IDEA](https://www.jetbrains.com/idea/?fromMenu#chooseYourEdition)，使用的是免费的Community version.


# Hello World

安装完之后打开IDEA,一路next，最后来到了主界面，依次执行下面几步：

1. 点击`Create New Project`

2. 在左边的栏目中选中`Kotlin`, 右边选中`Kotlin(JVM)`，点击Next

3. 自己设定Project name, Project location. 之后设定Project SDK, 选中Java虚拟机。这一步有可能因为前面JDK安装的问题，系统无法找到Java虚拟机，这时不用担心，点击右边的`New`，找到之前的Java安装目录，选中JDK的安装文件夹就好。比如如果Java安装在默认路径，选中`C:\Program Files\Java\jdk1.8.0_131`

点击`Finish`建立项目，这时一个Kotlin空项目已经建立。

4. 在空项目界面右边有几个文件，右键点击`src文件夹`->`New`->`Kotlin File/Class`，自己随便编辑一个名字生成程序文件。

5. 文件创建后会自动打开，在里面敲下以下代码：

{% highlight kotlin %}

package demo

fun main(args : Array<String>){
    println("Hello, world!")
}

{% endhighlight %}

保存代码。之后依次选择Build Project并执行(Run)，如果`Hello, world`出现在下面的命令行窗口中，说明环境配置成功！
