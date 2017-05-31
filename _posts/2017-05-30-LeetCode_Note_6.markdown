---
layout:     post
title:      "LeetCode 刷题笔记"
subtitle:   "6. ZigZag Conversion"
date:       2017-05-30 09:00:00
author:     "飞白"
header-img: "img/post-bg-leetcode.jpg"
catalog: true
tags:
    - C++
    - LeetCode
---

The string `"PAYPALISHIRING"` is written in a zigzag pattern on a given number of rows like this: (you may want to display this pattern in a fixed font for better legibility)

P   A   H   N
A P L S I I G
Y   I   R
And then read line by line: `"PAHNAPLSIIGYIR"`
Write the code that will take a string and make this conversion given a number of rows:

string convert(string text, int nRows);
convert("PAYPALISHIRING", 3) should return `"PAHNAPLSIIGYIR"`.

Solution 1: Direct Calculation

This solution follows direct definition of the question, when we align the string in ZigZag format:

first and last line: char distance is `2 x numRows - 2`
rest lines: char distance is `2 x numRows - 2 - 2 x i`

Simply Calculate the result string char position will do

Complexity: O(n)

Code:

{% highlight c++ %}

string convert(string s, int numRows) {
        if(numRows==1 || s.empty()) return s;
        string res = "";
        int distance = 2 * numRows-2;
        for(int i = 0; i < numRows; ++i)
        {
            int dis = distance - 2*i;
            int init = i;
            while(init < s.length())
            {
                res.push_back(s[init]);
                if(i==0||i==numRows-1) init += distance;
                else{
                    init+=dis;
                    dis = distance - dis;
                }
            }
        }
        return res;
    }

{% endhighlight %}

