---
title: "leetcode--Number of Digit One"
date: 2015-11-18T00:00:00+0800
lastmod: 2015-11-18T00:00:00+0800
draft: false
keywords: ["leetcode","1的个数"]
tags:  ["leetcode","1的个数"]
description: "计算1-n中所有数中1出现的个数"
categories: ["算法入门"]
tags: ["leetcode","1的个数"]
---

## Number of Digit One
Given an integer n, count the total number of digit 1 appearing in all non-negative integers less than or equal to n.

For example:

Given n = 13,Return 6, because digit 1 occurred in the following numbers: 1, 10, 11, 12, 13.

## 思路
1. 考虑134045百位上出现1的个数是多少？

    ```
    100-199        —>100
    1100-1199      ->100
    .
    .
    133100-133199  ->100
    总和是：134x100=13400
    ```

2. 考虑134145百位上出现1的个数是多少？

    ```
    100-199        —>100
    1100-1199      ->100
    .
    .
    133100-133199  ->100
    134100-134145  ->46
    总和是: 134x100+45+1=13446
    ```

3. 考虑134245百位上出现1的位数是多少？

    ```
    100-199        —>100
    1100-1199      ->100
    .
    .
    133100-133199  ->100
    134100-134199  ->100
    总和是: 134x100+1x100=13500
    ```

综合上面的分析可得：
c代表n的m位上的数，b代表这位前的数(在上面的例子就是134)，a代表的这位后的数(上面的例子中就是45)

* c=0,sum=ax10^m
* c=1,sum=ax10^m+b+1
* c\>1,sum=ax10^m+10^m

    ```c
    int countDigitOne(int n) {
        int i;
        int a,b;
        int ones=0;
        for(long long i=1;i<=n;i*=10)
        {
            a=n/i;
            b=n%i;
            switch(a%10)
            {
                case 0: ones+=a/10*i;break;
                case 1: ones+=a/10*i+b+1;break;
                default: ones+=(a/10+1)*i;
            }
        }
        return ones;
    }
    ```

