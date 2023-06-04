---
title: 龙书习题8.2
date: 2023-03-16 23:35:07
categories: codenote
tags: [Compilers]
---

龙书习题8.2

<!--more-->

## 8.2.1

Generate code for the following three-address statements assuming all variables are stored in memory locations.
1. x = 1
2. x = a
3. x = a + 1
4. x = a + b
5. The two statements
    - x = b * c
    - y = a + x

#### Anwser

```
1. LD R1, #1
   ST x, R1
2. LD R1, a
   ST x, R1
3. LD R1, a
   ADD R1, R1, 1
   ST x, R1
4. LD R1, a
   LD R2, b
   ADD R1, R1, R2
   ST x, R1
5. LD R1, b
   LD R2, c
   MUL R1, R1, R2
   ST x, R1
   LD R2, a
   ADD R1, R1, R2
   ST y, R1
```

## 8.2.2

Generate code for the following three-address statements assuming a and b are arrays whose elements are 4-byte values.

1. The four-statement sequence

   ```
    x = a[i]
    y = b[j]
    a[i] = y
    b[j] = x
   ```

2. The three-statement sequence

   ```
    x = a[i]
    y = b[i]
    z = x * y
   ```

3. The three-statement sequence

   ```
    x = a[i]
    y = b[x]
    a[i] = y
   ```

### Anwser

```
1. LD R1, i
   MUL R1, R1, 4
   LD R2, a(R1)
   ST x, R2
   LD R3, j
   MUL R3, R3, 4
   LD R4, b(R3)
   ST y, R4
   ST a(R1), R4
   ST b(R3), R2
2. LD R1, i
   MUL R1, R1, 4
   LD R2, a(R1)
   LD R3, b(R1)
   MUL R2, R2, R3
   ST z, R2
3. LD R1, i
   MUL R1, R1, 4
   LD R2, a(R1)
   ST x, R2
   LD R3, b(R2)
   ST y, R3
   ST a(R1), R3
```

## 8.2.3

Generate code for the following three-address sequence assuming that p and q are in memory locations:

```
y = *q
q = q + 4
*p = y
p = p + 4
```

### Anwser

```
LD R1, q
LD R2, 0(R1)
ADD R1, R1, #4
ST q, R1
LD R3, p
ST 0(R3), R2
ADD R3, R3, #4
ST q, R3
```

## 8.2.4

Generate code for the following sequence assuming that x, y, and z are in memory locations:

```
    if x < y goto L1
    z = 0
    goto L2
L1: z = 1
```

### Anwser

```
   LD R1, x
   LD R2, y
   SUB R1, R2
   BLTZ L1
   LD R1, #0
   ST z, R1
   BR L2
L1:LD R1, #1
   ST z, R1
L2:
```

## 8.2.5

Generate code for the following sequence assuming that n is in a memory location:

```
    s = 0
    i = 0
L1: if i > n goto L2
    s = s + i
    i = i + 1
    goto L1
L2:
```

### Anwser

```
   LD R1, #0
   ST s, R1
   ST i, R1
L1:LD R2, i
   LD R3, n
   SUB R4, R2, R3
   BGTZ L2
   LD R4, s
   ADD R4, R4, R2
   ST s, R4
   ADD R2, R2, #1
   ST i, R2
   BR L1
L2:
```
## 8.2.6
Determine the costs of the following instruction sequences:

```
1.  LD R0, y
    LD R1, z
    ADD R0, R0, R1
    ST x, R0

2.  LD R0, i
    MUL R0, R0, 8
    LD R1, a(R0)
    ST b, R1

3.  LD R0, c
    LD R1, i
    MUL R1, R1, 8
    ST a(R1),R0

4.  LD R0, p
    LD R1, 0(R0)
    ST x, R1

5.  LD R0, p
    LD R1, x
    ST 0(R0), R1

6.  LD R0, x
    LD R1, y
    SUB R0, R0, R1
    BLTZ *R3, R0
```

### Answer

1. 2 + 2 + 1 + 2 = 7
2. 2 + 2 + 2 + 2 = 8
3. 2 + 2 + 2 + 2 = 8
4. 2 + 2 + 2 = 6
5. 2 + 2 + 2 = 6
6. 2 + 2 + 1 + 1 = 6

