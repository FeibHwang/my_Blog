---
layout:     post
title:      "LeetCode 刷题笔记"
subtitle:   "7. Reverse Integer"
date:       2017-05-30 09:00:00
author:     "飞白"
header-img: "img/post-bg-leetcode.jpg"
catalog: true
tags:
    - C++
    - LeetCode
---

Reverse digits of an integer.

Example1: x = 123, return 321
Example2: x = -123, return -321

# Solution 1: Direct Reverse

Construct the new number directly

1. take care of sign
2. using `long long` in case of overflow

Code:

{% highlight c++ %}

int reverse(int x) {
        if(x == 0) return 0;
        int sign = x>0 ? 1:-1;
        long long t = x;
        t = abs(t);
        
        long long res = 0;
        
        while(t != 0)
        {
            res = 10*res + t%10;
            t /= 10;
        }
        
        res = sign*res;
        
        return (res>INT_MAX || res<INT_MIN) ? 0:(int)res;
    }

{% endhighlight %}

