---
title: "leetcode数组题目"
date: 2015-07-23T00:00:00+0800
lastmod: 2015-07-23T00:00:00+0800
draft: false
keywords: ["leetcode","数组"]
tags:  ["leetcode","数组"]
description: "leetcode上面数组题目的解决方法"
categories: ["算法入门"]
tags: ["leetcode","数组"]
---

## Remove Element

Given an array and a value, remove all instances of that value in place and
return the new length.

The order of elements can be changed. It doesn't matter what you leave beyond the new length.

主要思路如下图：

![remove_element](/imgs/leetcode/array/1.png)

code:

```c
int removeElement(int* nums, int numsSize, int val) {
    int i,j;
    i=-1;
    for(j=0;j<numsSize;j++)
        if(j-i>0)
            if(nums[j]!=val)
                nums[++i]=nums[j];
    return i+1;
}
```

## Single Number

Given an array of integers, every element appears twice except for one. Find that single one.

思路：A^A=0,A^0=A,所以所有的数都异或一遍，最后得到的结果就是那个出现一次的数字了！

```c
int singleNumber(int* nums, int numsSize) {
    int single=0;
    for(int i=0;i<numsSize;i++)
        single^=nums[i];
    return single;
}
```

##  Single Number II

Given an array of integers, every element appears three times except for one. Find that single one.

思路：这个可以从每个数字的二进制位来看，每一个数二进制位上出现的数相同的位必出现三次，所以相加再对3取模最后得到的那个位就是那个single number的二进制位上的数了。这其实就是对三进制数进行运算，而上面的题就是对二进制数进行运算。

```c
int singleNumber(int* nums, int numsSize) {
    int sum[32];
    int i,j;
    int single=0;
    for(i=0;i<32;i++)
        sum[i]=0;
    for(i=0;i<numsSize;i++)
        for(j=0;j<32;j++)
            sum[j]+=(nums[i]>>j &1);
    for(j=0;j<32;j++)
        if(sum[j]%3)
            single|= 1<<j;
    return single;
}
```

## Majority Elements

Given an array of size n, find the majority element. The majority element is the element that appears more than ⌊ n/2 ⌋ times.

You may assume that the array is non-empty and the majority element always exist in the array.

思路：用major记录最大数(初始为nums[0]),开始循环，如果等于major则出现的次数加1，如果不等则减1，最后如果nums==0,则将major重新指向nums[i].由于出现的次数大于1/2,所以最后得到的数一定是那个超过一半的数。

```c
int majorityElement(int* nums, int numsSize) {
    int major,i,num=0;
    for (i=0;i<numsSize;i++)
    {
        if(num==0)
            major=nums[i];
        (major==nums[i])?num++:num--;
    }
    return major;
}
```


