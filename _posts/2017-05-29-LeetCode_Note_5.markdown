---
layout:     post
title:      "LeetCode 刷题笔记： 5. Longest Palindromic Substring"
subtitle:   "C++笔记"
date:       2017-05-29 16:00:00
author:     "飞白"
header-img: "img/post-bg-leetcode.jpg"
catalog: true
tags:
    - C++
    - LeetCode
---

Given a string s, find the longest palindromic substring in s. You may assume that the maximum length of s is 1000.

Example:

Input: "babad"

Output: "bab"

Note: "aba" is also a valid answer.
Example:

Input: "cbbd"

Output: "bb"

# Solution 1: DP

The basic implementation of DP, using a nxn bool matrix `rec`, n is the length of the string, `rec[i][j]` defines whether s substring i to j is palindrom

initiation:
1. All rec[i][i] is true
2. rec[i][i+1] = s[i] == s[i+1]: record even palindrom

transfer function:
rec[line][row] = ((s[line] == s[row]) && rec[line+1][row-1])

find solution:
finally using loop find the result;

Code:

{% highlight c++ %}

string longestPalindrome(string s) {
        vector<vector<bool>> rec(s.length(),vector<bool>(s.length(),false));
        for(int i = 0; i < s.length(); ++i) rec[i][i] = true;
        for(int i = 0; i < s.length()-1; ++i) rec[i][i+1] = s[i]==s[i+1];
        
        for(int j = 2; j < s.length(); ++j)
        {
            int line = 0, row = j;
            while(line < s.length() && row < s.length())
            {
                rec[line][row] = ((s[line] == s[row]) && rec[line+1][row-1]);
                line++;
                row++;
            }
        }
        
        int len = 1, first = 0;
        for(int i = 0; i < rec.size(); ++i)
        {
            for(int j = i; j < rec.size(); ++j)
            {
                if(rec[i][j] && len < j-i+1)
                {
                    len = j-i+1;
                    first = i;
                }
            }
        }
        return s.substr(first,len);
    }

{% endhighlight %}

# Solution 2: Manacher Algorithm

Really Cool O(n) complexity algorithm

I will update the introduction later, there is a better [illustration](https://www.youtube.com/watch?v=V-sEwsca1ak) 

Code:


{% highlight c++ %}

string longestPalindrome(string s) {
    	vector<int> lb(s.size()*2-1, 0);
    	int rmax = -1, cmax = -1, maxlen = 0, first = 0, l, r;
     
    	for (int c = 0; c < s.size()*2-1; ++c) {
    		// initialize current expansion with reflection of c wrt. cmax if possible
    		if (c > rmax) 	r = (c+1)/2;
    		else r = min(rmax, cmax - lb[cmax - c]);
    	
    		for (l = c-r; l>=0 && r<s.size() && s[r]==s[l]; --l, ++r);
    		
    		// update palindrome substring of max length and rightest boundary
    		if (maxlen < r-l-1) {maxlen = r-l-1; first = l+1;}
    		if (rmax < r-1) { rmax = r-1; cmax = c;}
    		
    		lb[c] = l+1;
    	}
    	return s.substr(first, maxlen);
    }

{% endhighlight %}