---
layout:     post
title:      "使用Haskell实现Lisp解释器"
subtitle:   "编译原理学习笔记"
date:       2017-05-16 18:00:00
author:     "飞白"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - Haskell
    - 编译原理
    - 翻译
---

在翻了几章SICP之后，一直想试着实现一个Lisp解释器，在自己读master的第一年曾经试着实现了一个Lisp的子集编译器（没有看过编译原理，纯粹是凭感觉实现的，代码惨不忍睹=_=!），在学完Haskell以后，一直想找一个可以练手的项目，看到了这篇[文章](http://www.defmacro.org/ramblings/lisp-in-haskell.html)，就顺便翻译一下吧。

# 简介

不久前，在看了一堆有关Haskell的文章之后，我终于越过了理论安装了一个Haskell编译器。我决定通过两种方式探寻这门语言的演化方式。我准备先使用Haskell解决一个它本身擅长的问题，之后解决一个源自真实世界的问题。选择一个问题很简单，有很多“传言”说Haskell很适合实现编译器或解释器，因此我决定使用Haskell实现一个Lisp方言的解释器。对于第二个问题我选择实现一个网络应用，因为Haskell并没有大规模应用于这个领域。

我完成了第一个测试例子，结果也是验证有效的。我准备使用188行Haskell实现一个解释器来解析我的这篇[有关Lisp的文章](http://www.defmacro.org/ramblings/lisp.html)里的例子。我并没有试图压缩程序的体积，作为一个Haskell菜鸟这一步一般有花费更多的时间。

