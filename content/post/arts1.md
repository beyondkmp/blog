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

# Review

[The junior developer’s guide to writing super clean and readable code](https://medium.freecodecamp.org/the-junior-developers-guide-to-writing-super-clean-and-readable-code-cd2568e08aae)

clean code要像文章一样, 结构清晰，有标题，小标题，章节，这样就能很快的看清楚整个架构。文章整个都是代码类比文章，写代码就像写文章一样，要有明确的标题，章节，格式统一。

## 怎么做

1. 格式统一
2. 使用清晰的变量和方法名
3. 在必要的时候使用注释
4. 记住DRY原则(Don't Repeat Yourself)
5. 不要过度，提前clean代码


# Tip

## golang tips

1. `` `raw-string` ``, 多行时可以直接使用，相当于python里面的`"""`

## markdown

1. backticks输出backticks

    ```
    `` `test` ``
    ```

# Share


# 参考

1. [Binary Tree Level Order Traversal II](https://leetcode.com/problems/binary-tree-level-order-traversal-ii/)
2. [How do I escape a backtick ` within in-line code in Markdown? `](https://meta.stackexchange.com/questions/82718/how-do-i-escape-a-backtick-within-in-line-code-in-markdown)

