---
title: Unix系统调用
date: 2018-09-16 22:37:10
categories: codenote
tags: [OS, Linux]
---

《Unix高级编程学习笔记》 1

<!--more-->

# 系统调用简介

1. 为用户空间提供了一种硬件的抽象接口  
2. 保证了系统的稳定和安全  
3. Linux中进程都运行在虚拟系统中


## POSIX、API、C库
POSIX、API、C库和syscall之间的关系
![关系图](img/1.png)

## 系统调用
- 系统调用通常但不绝对，以`long`类型的返回值（为与64位的硬件体系结构保持兼容）
- 表示对错：负数表示错误，0值表示成功，出错将错误码写入`errno`全局变量，通过perror()库函数翻译成用户理解的错误字符串。   
- 宏定义：  `SYSCALL_DEFINE0(###)  `
- 含义： 定义一个在内核中无参数的系统调用  
  用例：

```
//定义内核中的系统调用getpid()的实现
//展开前
SYSCALL_DEFINE0(getpid)
{
    return task_tgidvnr(current); //return current->tgid
}

//展开后
asmlinkage long sys_getpid(void){
    return task_tgidvnr(current); //return current->tgid
}
```
用例:
```
//定义内核中的系统调用bar()的实现
//展开前
SYSCALL_DEFINE0(bar)
//展开后
asmlinkage long sys_bar()
```
注:用户空间返回`int`,内核空间返回`long`
## 系统调用号
用于关联系统调用。用户进程空间的进程执行一个系统调用时，使用系统调用号指明到底执行哪个系统调用，且不会提及系统调用的名称。
### 特点：
1. 一旦分配便不会再有任何改变;  
2. 系统调用号一旦删除系统调用号也不会被回收;  
3. `sys_ni_syscall()`返回`-ENOSYS`,专门填补无效系统调用;
4. sys_call_table 记录所有已注册过的系统调用，在`x86-64`中，定义于`arch/i386/kernel/syscall_64.c`中，并为每个系统调用指定唯一的系统调用号


## 系统调用号的性能  
很高：上下文切换时间短，进出内核被优化得简洁高效，系统调用处理程序和系统调用简洁
# 系统调用处理程序
告诉内核需要切换到内核态，让内核代表应用程序在内核空间执行系统调用    
实现机制:  
系统调用处理程序：引发一个异常促使系统谢欢到内核态去执行异常处理程序,叫syscall_call()
## 指定恰当的系统调用
1. x86上将系统调用号通过eax传递给内核  
2. 再执行`system_call()  `
3. 系统调用号与`NR_syscalls`相比较，大于等于NR_syscalls则返回-ENOSYS,否则执行相应的系统调用


```
call *sys_call_table(,%eax, 8)
//8代表系统调用表中表项以64位类型存放，则结果等于 系统调用号×4 用于查询系统调用位置。
//x86-32则用4代替8
```
![调用系统系统调用处理程序以执行一个系统调用](img/2.png)
## 参数传递
1. `x86-32 `系统中前五个参数放置在`ebx, ecx, edx, esi和edi`中  
2. 六个及六个以上则用单独的寄存器存放指向所有这些参数在用户空间的指针


# 系统调用的实现
## 实现系统调用
1. 明确此系统函数应该做什么  
2. 不提倡采用多用途的系统调用，考虑参数、返回值、错误码  
3. 接口应尽量为未来多做考虑，并考虑可移植性


## 参数验证
1. 检查他们的参数是否合法有效
2. 检查用户提供的指针是否有效

# 系统调用上下文
## 绑定一个系统调用的最后步骤
编写完一个系统调用后，将其注册诚正式的系统调用
1. 在系统调用表的最后加入一个表项其中系统调用号即为该系统调用在所有系统调用中的次序
2. 对所有支持的各种体系结构，系统调用号必须定义于`<asm/unistd.h>`中
3. 系统调用必须被编译进系统内核，即放进kernel/下的一个相关文件


## 从用户空间访问系统调用
通常，系统调用靠C库支持，如果仅仅写出系统调用，glibc库并不支持，怎么办呢  
答：宏` _syscalln()`,其中n范围为0~6代表传递给系统调用的参数个数，功能为设置好寄存器并调用陷入指令  
例子：

```
//open()系统调用定义
long open(const char* filename, int flags, int mode)
```
不靠库支持，直接调用此系统调用的宏形式
```
#define NR_open 5
_syscall3(long, open, const char*, filename, int, flags, int, mode)
```

# 在Android系统中系统调用的应用
在Android系统中，`_syscall()`被`syscall()`替代，且每一种Android支持的系统调用的参数形式都在Android源码中被SYSCALLS.txt记录，我们可以根据系统调用的知识，并结合SYSCALLS.txt，可以很轻易地写出自己对应的的syscall()。