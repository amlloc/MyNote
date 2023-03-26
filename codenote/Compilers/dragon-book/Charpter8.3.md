---
title: 龙书习题
date: 2023-03-16 23:35:07
categories: codenote
tags: [Compilers]

---

龙书习题8.3

<!--more-->

## 8.3.1

Generate code for the following three-address statements assuming stack allocation where register SP points to the top of the stack.

```
call p
call q
return
call r
return
return
```

### Anwser

```assembly
100:LD sp, #600
108:ADD sp, sp, #psize
116:ST 0(sp),#132
124:BR pStart
132:SUB sp, sp, #psize
140:ADD sp, sp, #qsize
148:ST (0)sp, #164
156:BR qStart
164:SUB sp, sp, #qsize
172:BR (0)sp

```

