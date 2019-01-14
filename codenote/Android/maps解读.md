---
title: /proc/{pid}/maps解读
date: 2018-09-16 22:37:38
categories: codenote
tags: [Android, 填坑记录]

---
非常常用的系统文件
<!--more-->

## /proc/{pid}/maps解读

总共6列，如

`76093000-76096000 r-xp 00000000 b3:19 941        /system/lib/libmemalloc.so`

1. 所处虚拟内存地址(VMA)范围：``76093000-76096000`

   - 在Linux中将进程虚拟空间中的一个段叫做虚拟内存区域VMA（Virtual Memory Area)。
   - VMA对应ELF文件中的segment。
   - ELF文件有section和segment的概念。从链接的角度看，ELF是按照section存储的，事实也的确如此；从装载的角度看，ELF文件又按照segment进行划分，这是为了防止按照section装载时造成的内部碎片。segment相当与是将多个属性（读写执行）相同的section合并在一起进行。program headers 存放segment的信息;section table存放section的信息.

2. VMA权限：`r-xp`

   `r=读，w=写,x=,s=共享,p=私有`

3. 偏移量：`00000000`

   表示VMA对应的segment在映像文件中的偏移。

4. 主设备号和次设备号（大雾）：`b3:19`

5. 映像文件的节点号inode：`941`

6. 映像文件的路径：`/system/lib/libmemalloc.so`
