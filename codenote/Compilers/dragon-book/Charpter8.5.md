---
title: 龙书习题8.5
date: 2023-03-16 23:35:07
categories: codenote
tags: [Compilers]



---

龙书习题8.5

<!--more-->

## 8.5.1

Construct the DAG for the basic block

```
d = b * c
e = a + b
b = b * c
a = e - d
```
### Answer
```mermaid
flowchart LR
subgraph a
c1(("-"))
end
subgraph e
c2(("+"))
end
subgraph d ["d,b1"]
c3(("*"))
end
c1----c2
c1----c3
c2----a1((a0))
c2----b1((b0))
c3----b1
c3----b2((c))
```

## 8.5.2

Simplify the three-address code of Exercise 8.5.1, assuming

1. Only a is live on exit from the block.
2. a, b, and c are live on exit from the block.

### Answer

1. ```
   e = a + b
   d = b * c
   a = e - d
   ```

2. ```
   e = a + b
   b = b * c
   a = e - b
   ```

## 8.5.3

Construct the basic block for the code in block B6 of Fig. 8.9. Do not forget to include the comparison i <= 10.






### Answer

```mermaid
graph TB
node1(("[]="))----node2((a))
subgraph t6
node3(("*"))
end
node1(("[]="))---node3
node1(("[]="))---node4((1.0))
node3---node5((88))
subgraph t5
node6(("-"))
end
node3---node6
node6---node7(("i0"))
node6---node8(("1"))
subgraph i
node9(("+"))
end
node9---node7
node9---node8
node10(("<="))---node9
node10(("<="))---node11((11))
node10(("<="))---node12((B6))
```

## 8.5.8

Suppose a basic block is formed from the C assignment state­ments

```
x = a + b + c + d + e + f;
y = a + c + e;
```

1. Give the three-address statements (only one addition per statement) for this block.
2. Use the associative and commutative laws to modify the block to use the fewest possible number of instructions, assuming both x and y are live on exit from the block.

### Answer

1. three-address statements

   ```
   t1 = a + b
   t2 = t1 + c
   t3 = t2 + d
   t4 = t3 + e
   x = t4 + f
   t5 = a + c
   y = t5 + e
   ```

2. optimized statments

   ```
   t1 = a + c
   t2 = t1 + e
   y = t2
   t3 = t2 + b
   t4 = t3 + d
   t5 = t4 + f
   x = t5
   ```

   