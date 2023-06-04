---
title: 龙书习题8.6
date: 2023-03-16 23:35:07
categories: codenote
tags: [Compilers]
---

龙书习题8.6

<!--more-->

## 8.6.1

 For each of the following C assignment statements 

```
a) x = a + b*c;
b) x = a/(b+c) - d*(e+f);
c) x = a[i] + 1;
d) a [i] = b [c[i]];
e) a[i][j] = b[i][k] + c[k][j];
f) *p++ = *q++;
```

generate three-address code, assuming that all array elements are integers taking four bytes each. In parts (d) and (e), assume that a, b, and c are constants giving the location of the first (Oth) elements of the arrays with those names, as in all previous examples of array accesses in this chapter.

### Answer

a)  x = a + b*c;

```
t1 = b * c
t2 = a + t1
x = t2
```

b) x = a/(b+c) - d*(e+f);

```
t1 = b + c
t2 = a / t1
t3 = e + f
t4 = d * t3
t5 = t2 - t4
x = t5
```

c) x = a[i] + 1;

```
t1 = i * 4
t2 = a[t1]
t3 = t2 + 1
x = t3
```

d) `a [i] = b [c[i]] `

```
t1 = i * 4
t2 = c[t1]
t3 = b[t2]
t4 = t3 * 4
a[t1] = t4
```

e) `a[i][j] = b[i][k] + c[k][j];`

假设所有二维数组都是n * n的大小

```
t1 = i * n
t2 = t1 + k
t3 = t2 * 4
t4 = b[t3]
t5 = k * n
t6 = t5 + j
t7 = t6 * 4
t8 = c[t7]
t9 = t4 + t8
t10 = i * n
t11 = t10 + j
t12 = t11 * 4
a[t12] = t9
```

f)`*p++ = *q++;`

```
t1 = *q
t2 = q + 4
q = t3
t3 = *p
t4 = p + 4
p = t4
*p = t1
```

## 8.6.2

Repeat Exercise 8.6.1 parts (d) and (e), assuming that the arrays a, b, and c are located via pointers, pa, pb, and pc, respectively, pointing to the locations of their respective first elements.
d) `a [i] = b [c[i]] `

```
t1 = pa
t2 = pb
t3 = pc
t4 = i * 4
t5 = t3 + t4
t6 = *t5 //c[i]
t7 = t6 * 4
t8 = t2 + t7
t9 = *t8 //b[c[i]]
t10 = pa + t4
*t10 = t9
```

## 8.6.3

Convert your three-address code from Exercise 8.6.1 into machine code for the machine model of this section. You may use as many registers as you need.

a)  x = a + b*c;

```
LD R0, a
LD R1, b
LD R2, c
MUL R3, R1, R2
ADD R4, R0, R3
ST X, R4
```

b) x = a/(b+c) - d*(e+f);

```
LD R0, a
LD R1, b
LD R2, c
LD R3, d
LD R4, e
LD R5, f
ADD R6, R1, R2
DIV R7, R0, R6
ADD R8, R4, R5
MUL R9, R3, R8
SUB R10, R7, R9
ST x, R10
```

c) x = a[i] + 1;

```
LD R0, i
LD R1, #4
MUL R2, R0, R1
LD R3, a(R2)
LD R4, #1
ADD R5, R3, R4
ST x, R5
```

d) `a [i] = b [c[i]] `

```
LD R0, i
LD R1, #4
MUL R2, R0, R1
LD R3, c(R2)
LD R4, b(R3)
ST a(R2), R4
```

e) `a[i][j] = b[i][k] + c[k][j]`

```
LD R0, i
LD R1, j
LD R2, k
LD R3, n
LD R4, #4
MUL R5, R0, R3
ADD R6, R5, R2
MUL R7, R6, R4 
LD R8, b(R7)  //b[i][k]
MUL R5, R2, R3
ADD R6, R5, R1
MUL R7, R6, R4 
LD R9, c(R7)  //c[k][j]
ADD R10, R8, R9
MUL R5, R0, R3
ADD R6, R5, R1 //b[i][k] + c[k][j]
MUL R7, R6, R4
ST a(R7), R10 //a[i][j]定值
```

## 8.6.4

Convert your three-address code from Exercise 8.6.1 into machine code, using the simple code-generation algorithm of this section, assuming three registers are available. Show the register and address descriptors after each step.

a)  x = a + b*c;

优化后三地址语句

```
t1 = b * c
t1 = a + t1
x = t1
```

| R0   | R1   | a    | b    | c    | x    | t1   |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- |
|      |      | a    | b    | c    |      |      |

`LD R0, b`

| R0   | R1   | a    | b    | c    | x    | t1   |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| b    |      | a    | b,R0 | c    |      |      |

`LD R1, c`

| R0   | R1   | a    | b    | c    | x    | t1   |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| b    | c    | a    | b,R0 | c,R1 |      |      |

`MUL R0, R0, R1`

| R0   | R1   | a    | b    | c    | x    | t1   |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| t1   | c    | a    | b    | c,R1 |      | R0   |

`LD R1,a`

| R0   | R1   | a     | b    | c    | x    | t1   |
| ---- | ---- | ----- | ---- | ---- | ---- | ---- |
| t1   | a    | a, R1 | b    | c    |      | R0   |

`ADD R0, R0, R1`

| R0   | R1   | a     | b    | c    | x    | t1   |
| ---- | ---- | ----- | ---- | ---- | ---- | ---- |
| t1   | a    | a, R1 | b    | c    |      | R0   |

`ST X, R0`

| R0   | R1   | a     | b    | c    | x    | t1   |
| ---- | ---- | ----- | ---- | ---- | ---- | ---- |
| t1,x | a    | a, R1 | b    | c    | R0   | R0   |

b) x = a/(b+c) - d*(e+f);

优化后三地址语句

```
t1 = b + c
t1 = a / t1
t2 = e + f
t2 = d * t2
t3 = t1 - t2
x = t3
```


化简后汇编代码

```
LD R0, b
LD R1, c
ADD R0, R0, R1
LD R1, a
DIV R0, R1, R0
LD R1, e
LD R2, f
ADD R1, R1, R2
LD R2, d
MUL R1, R2, R1
SUB R2, R0, R1
ST x, R2
```

| R0   | R1   | R2   | a    | b    | c    | d    | e    | f    | t1   | t2   | t3   | x    |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
|      |      |      | a    | b    | c    | d    | e    | f    |      |      |      |      |

`LD R0, b`

| R0   | R1   | R2   | a    | b    | c    | d    | e    | f    | t1   | t2   | t3   | x    |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| b    |      |      | a    | b,R0 | c    | d    | e    | f    |      |      |      |      |

`LD R1, c`

| R0   | R1   | R2   | a    | b    | c    | d    | e    | f    | t1   | t2   | t3   | x    |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| b    | c    |      | a    | b,R0 | c,R1 | d    | e    | f    |      |      |      |      |

`ADD R0, R0, R1`

| R0   | R1   | R2   | a    | b    | c    | d    | e    | f    | t1   | t2   | t3   | x    |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| t1   | c    |      | a    | b    | c,R1 | d    | e    | f    | R0   |      |      |      |

`LD R1, a`

| R0   | R1   | R2   | a    | b    | c    | d    | e    | f    | t1   | t2   | t3   | x    |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| t1   | a    |      | a,R1 | b    | c    | d    | e    | f    | R0   |      |      |      |

`DIV R0, R1, R0`

| R0   | R1   | R2   | a    | b    | c    | d    | e    | f    | t1   | t2   | t3   | x    |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| t1   | a    |      | a,R1 | b    | c    | d    | e    | f    | R0   |      |      |      |

`LD R1, e`

| R0   | R1   | R2   | a    | b    | c    | d    | e    | f    | t1   | t2   | t3   | x    |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| t1   | e    |      | a    | b    | c    | d    | e,R1 | f    | R0   |      |      |      |

`LD R2, f`

| R0   | R1   | R2   | a    | b    | c    | d    | e    | f    | t1   | t2   | t3   | x    |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| t1   | e    | f    | a    | b    | c    | d    | e,R1 | f,R2 | R0   |      |      |      |

`ADD R1, R1, R2`

| R0   | R1   | R2   | a    | b    | c    | d    | e    | f    | t1   | t2   | t3   | x    |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| t1   | t2   | f    | a    | b    | c    | d    | e    | f,R2 | R0   | R1   |      |      |

`LD R2, d`

| R0   | R1   | R2   | a    | b    | c    | d    | e    | f    | t1   | t2   | t3   | x    |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| t1   | t2   | d    | a    | b    | c,R2 | d,R2 | e    | f    | R0   | R1   |      |      |

`MUL R1, R2, R1`

| R0   | R1   | R2   | a    | b    | c    | d    | e    | f    | t1   | t2   | t3   | x    |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| t1   | t2   | d    | a    | b    | c,R2 | d,R2 | e    | f    | R0   | R1   |      |      |

`SUB R2, R0, R1`

| R0   | R1   | R2   | a    | b    | c    | d    | e    | f    | t1   | t2   | t3   | x    |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| t1   | t2   | t3   | a    | b    | c,R2 | d    | e    | f    | R0   | R1   | R2   |      |

`ST x, R2`

| R0   | R1   | R2   | a    | b    | c    | d    | e    | f    | t1   | t2   | t3   | x    |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| t1   | t2   | t3,x | a    | b    | c,R2 | d    | e    | f    | R0   | R1   | R2   | R2   |

## 8.6.5

Repeat Exercise 8.6.4, but assuming only two registers are available.
