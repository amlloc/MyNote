---
title: baksmali源码解析
date: 2019-04-21 17:37:55
categories: codenote
tags: Android, smali, reverse
---

bakesmali源码编译及其解析笔记

<!--more-->

## 源码编译

- 源码clone:

  `git clone git@github.com:JesusFreke/smali.git`

- 导入idea中，并编译，发现出现如下错误

  ![1555840082723](baksmali源码解析/1555840082723.png)

- 此时应该下载如下工具进行转化：

  1. [antlr-3.5.2-complete.jar](https://www.antlr3.org/download/)，注意，必须使用antlr3版本的

     一个开源的词法分析器的jar包，主要用于生成smali和baksmali的语法解释器

  2. [jflex-1.7.0.tar.gz](https://jflex.de/download.html),其中我们需要用到的jflex-full-1.7.0.jar文件在解压缩包下面的`lib`目录下

  接下来就是利用这两个jar包分别对三个文件进行转换:

  - ` java -jar antlr-3.5.2-complete.jar '/home/amlloc/Projects/github/smali/smali/src/main/antlr/smaliTreeWalker.g'`

  - ` java -jar antlr-3.5.2-complete.jar '/home/amlloc/Projects/github/smali/smali/src/main/antlr/smaliParser.g'`
  - `java -jar jflex-full-1.7.0.jar '/home/amlloc/Projects/github/smali/smali/src/main/jflex/smaliLexer.jflex' `

  之后得到三个java源文件(标红的三个文件):

  ![1555840720906](baksmali源码解析/1555840720906.png)

- 将这三个文件移动到`org/jf/smali`文件夹下，然后进行编译，依旧报错

  ![1555841916679](baksmali源码解析/1555841916679.png)

  发现提示涉及的文件夹下有关键.g文件，`smali/smali/src/test/antlr/org/jf/smali/expectedTokensTestGrammar.g`

  如法炮制，

  `java -jar antlr-3.5.2-complete.jar '/home/amlloc/Projects/github/smali/smali/src/test/antlr/org/jf/smali/expectedTokensTestGrammar.g'`

- 编译成功

参考文献:

https://blog.51cto.com/sunzeduo/1540085