---
title: "leetcode: Summary Ranges"
date: 2016-05-08T00:00:00+0800
lastmod: 2016-05-08T00:00:00+0800
draft: false
keywords: ["leetcode","summary ranges"]
tags:  ["leetcode","summary ranges"]
description: "每天一题-leetcode"
categories: ["算法入门"]
tags: ["leetcode","summary ranges"]
---

## 描述

    Given a sorted integer array without duplicates, return the summary of its ranges.
    
    For example, given [0,1,2,4,5,7], return ["0->2","4->5","7"].
    
    **Credits:**
    Special thanks to @jianchao.li.fighter for adding this problem and creating all test cases.
    
    Subscribe to see which companies asked this question


### go语言

```go

func summaryRanges(nums []int) []string {
    if len(nums)==0{
        return []string{}
    }
    s := 0
    e := 0
    result := []string{}
    for i:=1;i<len(nums);i++{
        if nums[i]==nums[i-1]+1{
            e = i
        }else{
            if s ==e{
                result = append(result,fmt.Sprintf("%v",nums[s]))
            }else{
                result = append(result,fmt.Sprintf("%v->%v",nums[s],nums[e]))
            }
            s = i
            e = i
        }
    }
    if s ==e{
        result = append(result,fmt.Sprintf("%v",nums[s]))
    }else{
        result = append(result,fmt.Sprintf("%v->%v",nums[s],nums[e]))
    }

    return result
}
```
