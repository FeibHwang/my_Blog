---
layout:     post
title:      "LeetCode 刷题笔记： 4.Longest Substring Without Repeating Characters"
subtitle:   "C++笔记"
date:       2017-05-29 14:00:00
author:     "飞白"
header-img: "img/post-bg-leetcode.jpg"
catalog: true
tags:
    - C++
    - LeetCode
---

There are two sorted arrays nums1 and nums2 of size m and n respectively.

Find the median of the two sorted arrays. The overall run time complexity should be O(log (m+n)).

Example 1:
nums1 = [1, 3]
nums2 = [2]

The median is 2.0
Example 2:
nums1 = [1, 2]
nums2 = [3, 4]

The median is (2 + 3)/2 = 2.5

Solution 1: Binary Search

The idea is simple but the real code is hard to make it right

First we need to generize the question: Finding median is equal to find kth smallest element of the array, k is (nums1.size() + nums2.size())/2. So we can implement a `find_kth` function which will find kth smallest element of 2 array.

The `find_kth` function looks like below:

{% highlight c++ %}

static int find_kth(std::vector<int>::const_iterator A, int m, std::vector<int>::const_iterator B, int n, int k)

{% endhighlight %}

`A` and `B` are the two iterator of the array. `m`,`n` stands for the length, `k` is the target index.

the whole function works like follows:

1. if(m > n): find_kth(B,n,A,m,k), swap the two array for simplicity.
2. if(m == 0): empty A array, the result is the kth element of B, return *(B+k-1)
3. if(k==1): the first element, return the smaller head element of A and B

use `ia` record min(k/2,m), `ib` record k - ia

4. if(*(A+ia-1) < *(B+ib-1)): compare the k/2th element of A and another k/2th element of B, if smaller, then the first half of A can be ruled out, find the (k-ia)th elemnt of the rest arraies

5. if(*(A+ia-1) > *(B+ib-1)): the half of the B array elements can be ruled out, find the rest of the iath smallest number

6. if equal: then A[ia-1] is the result.

Code:

 {% highlight c++ %}

double findMedianSortedArrays(vector<int>& nums1, vector<int>& nums2) {
        int l1 = nums1.size();
        int l2 = nums2.size();
        
        if((l1+l2) & 1) return find_kth(nums1.begin(),nums1.size(),nums2.begin(),nums2.size(),(l1+l2)/2+1);
        //odd total length
        
        return (find_kth(nums1.begin(),nums1.size(),nums2.begin(),nums2.size(),(l1+l2)/2) + 
                find_kth(nums1.begin(),nums1.size(),nums2.begin(),nums2.size(),(l1+l2)/2+1)
                )/2.0;
        //even total length
    }
    
    static int find_kth(std::vector<int>::const_iterator A, int m, std::vector<int>::const_iterator B, int n, int k)
    {
        if(m > n) return find_kth(B, n, A, m, k);
        if(m == 0) return *(B+k-1);
        if(k == 1) return min(*A, *B);
        
        int ia = min(k/2,m), ib = k - ia;
        
        if(*(A+ia-1) > *(B+ib-1)) return find_kth(A,m,B+ib,n-ib,ia);
        else if(*(A+ia-1) < *(B+ib-1)) return find_kth(A+ia,m-ia,B,n,ib);
        else return A[ia-1];
    }

{% endhighlight %}