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

### Linux C下的文件操作权限

Linux系统中采用三位**八进制的数字**来表示文件的操作权限，为了表示方便（不确定是不是这个原因）通常用四位八进制数来表示，首位取0，也即0ABC的形式，其中A、B、C都是0~7的数字：
A表示的是文件主的权限；
B表示的是组用户的权限；
C表示的是其他用户的权限。

0~7各个数字代表的含义如下（r：Read读，w：Write写，x：eXecute执行）：

```
---     0   不可读写，不可执行
--x     1   可执行，不可读写
-w-     2   可写，不可读，不可执行
-wx     3   可写可执行，不可读
r--     4   可读，不可写，不可执行
r-x     5   可读，可执行，不可写
rw-     6   可读写，不可执行
rwx     7   可读写，可执行
```

因此，0644代表文件主可读写，组用户和其他用户有可读权限。