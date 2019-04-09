---
title: "arts第一周"
date: 2019-04-09T15:12:14+0800
lastmod: 2019-04-09T15:12:14+0800
draft: false
keywords: ["arts第一周"]
description: "arts第一周"
tags: ["arts"]
categories: ["arts"]
author: "beyondkmp"

---

# Algorithm 

## Binary Tree Level Order Traversal II

### description

Given a binary tree, return the bottom-up level order traversal of its nodes' values. (ie, from left to right, level by level from leaf to root).

For example:
Given binary tree [3,9,20,null,null,15,7],

```
    3
   / \
  9  20
    /  \
   15   7
```

return its bottom-up level order traversal as:

```
[
  [15,7],
  [9,20],
  [3]
]
```

<!--more-->

### solution

```go
package main

/**
 * Definition for a binary tree node.
 * type TreeNode struct {
 *     Val int
 *     Left *TreeNode
 *     Right *TreeNode
 * }
 */
func levelOrderBottom(root *TreeNode) [][]int {
    var result [][]int
    var traverse func(head *TreeNode, level int)

    traverse = func(head *TreeNode, level int) {
        if head == nil {
            return
        }

        if level >= len(result) {
            result = append(result, []int{})
        }
        result[level] = append(result[level], head.Val)

        traverse(head.Left, level+1)
        traverse(head.Right, level+1)
    }
    traverse(root, 0)

    //reverse  result
    for left, right := 0, len(result)-1; left < right; left, right = left+1, right-1 {
        result[left], result[right] = result[right], result[left]
    }
    return result
}

```


# 参考

1. [Binary Tree Level Order Traversal II](https://leetcode.com/problems/binary-tree-level-order-traversal-ii/)

