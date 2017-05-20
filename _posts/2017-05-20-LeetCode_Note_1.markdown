---
layout:     post
title:      "LeetCode 刷题笔记： 1.Two Sum"
subtitle:   "C++笔记"
date:       2017-05-20 12:00:00
author:     "飞白"
header-img: "post-bg-leetcode.jpg"
catalog: true
tags:
    - C++
    - LeetCode
---

# Introduction

Given an array of integers, return `indices` of the two numbers such that they add up to a specific target.

You may assume that each input would have `exactly` one solution, and you may not use the same element twice.

Example:


Given nums = [2, 7, 11, 15], target = 9,

Because nums[0] + nums[1] = 2 + 7 = 9,
return [0, 1].


# Solution 1: hash map

Using hash map for the solution, record the needed value and the index pair.

For Example: [2,7,11,15], target = 9

Then the algorithm goes like:
1. map->search 2: not find -> insert <7,0>  (9-2 = 7, index 0)
2. map->search 7: find -> return {map[7],1} -> {0,1} 

Space Complexity: O(n)

Time Complexity: O(n)

Code:

{% highlight c++ %}

vector<int> twoSum(vector<int>& nums, int target) {
    unordered_map<int,int> dict;
    for(int i = 0; i < nums.size(); ++i)
    {
        if(dict.find(nums[i])==dict.end())
        {
            dict[target-nums[i]] = i;
        }else{
            return {dict[nums[i]],i};
        }
    }
    return {};
}

{% endhighlight %}


P.S:

Two pointer is not suitable for this problem, because after sorting the array, the index is completely disorganized.

