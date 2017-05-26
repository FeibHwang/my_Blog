---
layout:     post
title:      "LeetCode 刷题笔记： 3.Longest Substring Without Repeating Characters"
subtitle:   "C++笔记"
date:       2017-05-26 14:00:00
author:     "飞白"
header-img: "img/post-bg-leetcode.jpg"
catalog: true
tags:
    - C++
    - LeetCode
---

# Introduction:

Given a string, find the length of the longest substring without repeating characters.

Examples:

Given `"abcabcbb"`, the answer is `"abc"`, which the length is 3.

Given `"bbbbb"`, the answer is `"b"`, with the length of 1.

Given `"pwwkew"`, the answer is `"wke"`, with the length of 3. Note that the answer must be a substring, `"pwke"` is a subsequence and not a substring.

# Solution 1: Two Pointer

Idea:

using two pointer `fast` and `slow`, and a length-256 array. 

The array record how many times the char show up in the string from index `slow` to `fast`, the `fast` ponter will keep going, the char is record, once the record char number is larger than 1, which means the character show up twice, then we should update the max length.

Then the slow pointer will keep going until the duplicated character is removed from the record.

Pay attention that after the loop, the result should update once more, in case of the whole string satisfy the requirement.

Code:

{% highlight c++ %}

int lengthOfLongestSubstring(string s) {
        if(s.empty()) return 0;
        
        vector<int>rec(256,0);
        
        int slow = 0, fast = 0, res = 0;
        
        while(fast < s.length())
        {
            rec[s[fast]]++;
            
            if(rec[s[fast]]>1)
            {
                res = max(res,fast-slow);
                while(rec[s[fast]]>1)
                {
                    rec[s[slow++]]--;
                }
            }
            
            fast++;
        }
        res = max(res,fast-slow);
        return res;
    }

{% endhighlight %}