---
title: c-cplus-mode
date: 2018-09-16 22:35:59
categories: codenote
tags: [C/C++]
---

一张C/C++打开文件mode对应表
<!--more--> 
某一天在Android studio下浏览Android源码下C++源码时发现下图，还是很有意思而且实用的hhhhh  
熟悉C或C++中一门语言的同学可参照下表来进行文件的打开，可获得等价的打开效果
如

```c
fopen("/sdcard/test.txt",w+)
//相当于
if.open("/sdcard/test.txt", ios::in || ios:: out || trunc);
```
表如下：
```c++
/**
       *  @brief  Opens an external file.
       *  @param  __s  The name of the file.
       *  @param  __mode  The open mode flags.
       *  @return  @c this on success, NULL on failure
       *
       *  If a file is already open, this function immediately fails.
       *  Otherwise it tries to open the file named @a __s using the flags
       *  given in @a __mode.
       *
       *  Table 92, adapted here, gives the relation between openmode
       *  combinations and the equivalent @c fopen() flags.
       *  (NB: lines app, in|out|app, in|app, binary|app, binary|in|out|app,
       *  and binary|in|app per DR 596)
       *  <pre>
       *  +---------------------------------------------------------+
       *  | ios_base Flag combination            stdio equivalent   |
       *  |binary  in  out  trunc  app                              |
       *  +---------------------------------------------------------+
       *  |             +                        w                  |
       *  |             +           +            a                  |
       *  |                         +            a                  |
       *  |             +     +                  w                  |
       *  |         +                            r                  |
       *  |         +   +                        r+                 |
       *  |         +   +     +                  w+                 |
       *  |         +   +           +            a+                 |
       *  |         +               +            a+                 |
       *  +---------------------------------------------------------+
       *  |   +         +                        wb                 |
       *  |   +         +           +            ab                 |
       *  |   +                     +            ab                 |
       *  |   +         +     +                  wb                 |
       *  |   +     +                            rb                 |
       *  |   +     +   +                        r+b                |
       *  |   +     +   +     +                  w+b                |
       *  |   +     +   +           +            a+b                |
       *  |   +     +               +            a+b                |
       *  +---------------------------------------------------------+
       *  </pre>
       */
```