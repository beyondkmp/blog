---
title: "Excel实现下拉列表并管理键值对"
date: 2015-04-14T00:00:00+0800
lastmod: 2015-04-14T00:00:00+0800
draft: false
keywords: ["Excel","下拉列表","键值对"]
tags:  ["Excel","下拉列表","键值对"]
description: "Excel实现下拉列表并管理键值对"
categories: ["常用技巧"]
tags: ["Excel","下拉列表","键值对"]
---

今天做.net程序，要实现excel列做成下拉框且键值对的效果。在网上查了很多资料，最后终于找到实现方法，怕以后会忘记，现在赶紧记录下来。

**使用excel自带的VLOOPUP函数实现下拉框管理键值对的效果。**

## VLOOKUP()详解

*=VLOOPUP(查找值,数据表,列序数,[匹配条件])*函数，四个参数，依次是：

* 判断的条件（我这里指的是下拉框的那列值）
* 跟踪数据的区域（key-value的list区域）
* 返回第几列的数据（第几列是相对跟踪数据的区域而言，跟踪数据从1开始列数）
* 是否精确匹配（false：否，即数据库的like条件；true：是，即数据库的=。说明，key-value模式下此参数不起作用，可随意）

## 具体实现方法：

1. 在sheet1中输入键值对key-value，实际中我用到了两组。

    ![1]({{IMAGE_PATH}}/excel/1.png)

2. sheet2中:

    (1) 实现列的下拉框效果：选中D列，数据->有效性，选“序列”，输入来源，为下拉框的值，值之间用英文逗号隔开。

    ![2]({{IMAGE_PATH}}/excel/2.png)

    (2) 在F2框输入公式`=VLOOKUP(D2,Sheet1!$A$2:$B$4,2,FALSE)`，D2为要查找的值，直接选中要sheet1中要跟踪的数据区域，2表示(1)截图中的B列

    ![3]({{IMAGE_PATH}}/excel/3.png)

    (3) 但是F2输入框中显示的是`#N/A`，传到后台程序会读取成`#N/A`，但是若想让空值显示为空，则使用IF函数和ISNA函数搭配，最终的函数为

        =IF(ISNA(VLOOKUP(D2,Sheet1!$A$2:$B$4,2,FALSE)),"",VLOOKUP(D2,Sheet1!$A$2:$B$4,2,FALSE))

    ![4]({{IMAGE_PATH}}/excel/4.png)


    (4) 再者就是要实现多行都使用这个公式了。直接从F2输入框往下拖动，就可以实现D列选值后value直接写到F列

    (5) 最后实现的效果是D列为下拉框，选择一个值后，F列跟着相应变化

    ![5]({{IMAGE_PATH}}/excel/5.png)



