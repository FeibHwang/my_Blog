---
layout:     post
title:      "LeetCode 刷题笔记"
subtitle:   "2.Add Two Numbers"
date:       2017-05-26 13:00:00
author:     "飞白"
header-img: "img/post-bg-leetcode.jpg"
catalog: true
tags:
    - C++
    - LeetCode
---

# Introduction

You are given two `non-empty` linked lists representing two non-negative integers. The digits are stored in reverse order and each of their nodes contain a single digit. Add the two numbers and return it as a linked list.

You may assume the two numbers do not contain any leading zero, except the number 0 itself.

`Input`: (2 -> 4 -> 3) + (5 -> 6 -> 4)
`Output`: 7 -> 0 -> 8

# Solution 1: Direct Calculation with full adder

The idea is pretty simple: since the order is allready reversed, the lowest bit is automatically matched, using a Cin bit to preserve carray.

Pay attention that it's possible the two number has different length, so in the while loop we need to consider 3 situation:

1. l1 ended, l2 continue
2. l2 ended, l1 continue
3. l1,l2 both continue 

Code:

{% highlight c++ %}

ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        int c = 0;
        ListNode* head = new ListNode(0);
        ListNode* cnt = head;
        
        while(l1 || l2)
        {
            if(!l1)
            {
                cnt->val = (l2->val + c)%10;
                c = (l2->val + c)/10;
                l2 = l2->next;
            }
            else if(!l2)
            {
                cnt->val = (l1->val + c)%10;
                c = (l1->val + c)/10;
                l1 = l1->next;
            }
            else
            {
                cnt->val = (l1->val + l2->val + c)%10;
                c = (l1->val + l2->val + c)/10;
                l1 = l1->next;
                l2 = l2->next;
            }
            
            if(l1 || l2){
                cnt->next = new ListNode(0);
                cnt = cnt->next;
            }
        }
        
        if(c!=0) cnt->next = new ListNode(c);
        
        return head;
    }

{% endhighlight %}