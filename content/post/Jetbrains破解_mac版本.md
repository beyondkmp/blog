---
title: "Jetbrainds破解(mac和linux版本)"
date: 2015-05-07T00:00:00+0800
lastmod: 2015-05-07T00:00:00+0800
draft: false
keywords: ["jetbraind","破解"]
tags:  ["jetbraind","破解"]
description: "在linux和mac下Jetbrainds ide的破解方法"
categories: ["常用技巧"]
tags: ["jetbraind","破解"]
---

1. 从官网下载Clion,使用[Rover12421](http://www.rover12421.com)的一个破解程序，从github[下载](https://github.com/rover12421/JetbrainsPatchKeygen)。

2. 执行gradle fatjar，但是没有gradle，记得以前下载过一个gradle的下载包，运行下面的可以生成相应的jar,如果有失败的情况，要升级你的java的版本要1.7以上。

    ```
    ⇒  ~/code/software/gradle-2.2.1/bin/gradle fatjar
    Download
    https://jcenter.bintray.com/eu/appsatori/gradle-fatjar-plugin/0.3/gradle-fatjar-plugin-0.3.pom
    Download
    https://jcenter.bintray.com/eu/appsatori/gradle-fatjar-plugin/0.3/gradle-fatjar-plugin-0.3.jar
    Incremental java compilation is an incubating feature.
    Download
    https://repo1.maven.org/maven2/commons-io/commons-io/2.4/commons-io-2.4.pom
    Download
    https://repo1.maven.org/maven2/org/apache/commons/commons-parent/25/commons-parent-25.pom
    Download
    https://repo1.maven.org/maven2/commons-io/commons-io/2.4/commons-io-2.4.jar
    :compileJava
    :compileJava - is not incremental (e.g. outputs have changed, no previous
    execution, etc.).
    警告: [options] 未与 -source 1.7 一起设置引导类路径
    1 个警告
    :processResources UP-TO-DATE
    :classes
    :fatJarPrepareFiles
    :fatJar

    BUILD SUCCESSFUL

    Total time: 17.517 secs

    ```
3. 打补丁并且生成注册序列号，如下：

    ```
    ⇒  java -jar build/libs/JetbrainsPatchKeygen-1.0.jar
    Jetbrains's Product Patch or Keygen.
    Crack by Rover12421.
    Http://Www.Rover12421.Com
    Please support genuine(https://www.jetbrains.com).

    Please select product:
    1. RubyMine
    2. PyCharm
    3. WebStorm
    4. PhpStorm
    5. AppCode
    6. Clion
    7. Idea
    6
    Please enter your username :
    birdcat
    Clion need patch
    Please enter your Clion install path or clion.jar path :
    /Applications/CLion.app/Contents/lib/
    Find Key. Patch...
    ---------------------------------------
    ===== LICENSE BEGIN =====
    4495-D14624T
    00002RGu86EMk0Nd5YVUoM0nAMcogC
    P"rtzULz7CK08rhViSFA3Q8WUro7k7
    wkYCdjNdc4axIlrgO!ZroIi2evBZ21
    ===== LICENSE END =====
    ```

4. 注册成功的图如下：

    ![注册成功](/imgs/jetbrainds破解/2.png)

#### 参考
1. [CLion 1.0 破解 (MAC 版)](http://www.jianshu.com/p/3ebdf488e32a)
2. [Jetbrains Product Patch or Keygen](http://www.rover12421.com/2015/04/07/jetbrains-product-patch-or-keygen.html)
3. [rover12421/JetbrainsPatchKeygen](https://github.com/rover12421/JetbrainsPatchKeygen)



