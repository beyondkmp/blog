---
title: "pandoc常用技巧"
date: 2015-12-22T00:00:00+0800
lastmod: 2015-12-22T00:00:00+0800
draft: false
keywords: ["pandoc","github html"]
tags:  ["pandoc","github html"]
description: "收集了常用的pandoc将markdown文件生成对应的html和pdf文件"
categories: ["常用技巧"]
tags: ["pandoc","github html"]
---

在mac os x下面安装pandoc：`brew install pandoc`

1. 直接将md文件转化成html文件,没有css样式：

    ```
    pandoc test.md -o test.html
    ```
2. 使用指定的css文件样式，可以生成github模板的网页

    ```
    pandoc -c github.css test.md -o test.html
    ```
3. 生成独立的html，即将css样式包含到html中
    * 方法一：在github.css样式文件的头部和尾部加上,参考文献[[1]](http://devilgate.org/blog/2012/07/02/tip-using-pandoc-to-create-truly-standalone-html-files/)

        ```
        <style type="text/css">
        github.css中的原内容
        </style>

        pandoc -s -H github.css test.md -o test.html
        ```
    * 方法二：--self-contained会把css，图片，视频等都包含到html中，将其编码为base64,用data://协议读取。
        ```
        pandoc --self-contained -c github.css test.md -o test.html
        ```

## 参考
1. [Tip: using Pandoc to create truly standalone HTML files](http://devilgate.org/blog/2012/07/02/tip-using-pandoc-to-create-truly-standalone-html-files/)
2. [pandoc demos](http://pandoc.org/demos.html)
3. [Pandoc User’s Guide](http://pandoc.org/README.html)
