---
title: 龙书习题8.9
date: 2023-03-16 23:35:07
categories: codenote
tags: [Compilers]

---

龙书习题8.9

<!--more-->

一些目标机指令的树重写规则

![image-20230330230406061](Charpter8.9.assets/1.png)

## 8.9.1

Construct syntax trees for each of the following statements assuming all nonconstant operands are in memory locations:

a) x = a * b + c * d;

b) x[i] = y[j] * z[k];

c) x = x + 1;

Use the tree-rewriting scheme in Fig. 8.20 to generate code for each statement.

a)x = a * b + c * d;

![image-20230330231617095](Charpter8.9.assets/2.png)

```
LD R0, a
LD R1, b
MUL R0, R0, R1
LD R2, c
LD R3, d
MUL R2, R2, R3
ADD R0, R0, R2
ST x, R0
```

b) x[i] = y[j] * z[k];

![image-20230330234940423](Charpter8.9.assets/3.png)

```
LD R0, Cy
ADD R0, R0, Rsp
ADD R0, R0, j(Rsp)
LD R1, Cz
ADD R1, R1, Rsp
ADD R1, R1, k(Rsp)
MUL R0, R0, R1
LD R2, Cx
ADD R2, R2, Rsp
ADD R2, R2, i(Rsp)
ST *R2, R0
```

c) x = x + 1;

![image-20230330235236237](Charpter8.9.assets/4.png)

```
LD R0, x
ADD R0, R0, 1
ST x, R0
```

