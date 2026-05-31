---
title: 并发2
published: 2026-05-18
tags: [ 操作系统, 多线程, 锁 ]
category: 操作系统

draft: false
---
作业：

使用C语言编写peterson算法，并且编写model checker程序检查

使用原子交换实现自旋锁(Compare and swap, CAS)

# 互斥

### Amdahl's Law
T<sub>∞</sub> > T<sub>1</sub> / k

不能并行的代码，占总代码的1/k；T<sub>n</sub>为有n个处理器的时候，处理的时间

### Gustafson’s Law

T<sub>p</sub>  < (T<sub>∞</sub> + T<sub>1</sub> / p)

### 局部性原理
只能和相邻的事物进行交互->相邻的边占事物的很小的一部分

T<sub>∞</sub> << T<sub>1</sub>

## 实现互斥
### 1. 单处理器实现互斥
关中断   stop the world

NMI：不可屏蔽中断

### 2. Peterson算法（没啥用）
**实现假设**
- 任何时候可以执行一条load/store指令
- 读写本身是原子的

![peterson算法的实现](images/dxc-1.png)

只能支持2个线程的互斥
### model checker
利用程序节省复杂的人工

## 多处理器系统的互斥
问题：load和store是分开的，你要么看，要么写

一个解法：同时load和store（软件不够，硬件来凑）

### 原子指令
原子指令：一小段时间的 stop the world

**自旋锁**
使用原子交换实现自旋锁(Compare and swap, CAS)
```C
extern atomic_xchg(int *a, int *b) // swap values of a and b atomatically
extern atomic_cmp_and_xchg(int *a, int *b, int value) // swap values of a and b atomatically when a equals to value


```

发生中断了怎么办？
所有需要锁的线程空转
在中断处理程序中尝试获取锁会怎么样

# 操作系统内核中的互斥
## 正确性准则
**正确实现互斥**
- 关中断+自旋可以保证实现互斥
**上锁/解锁前后中断状态不变**
- 不得在关中断时随意打开中断
- 不得随意关闭中断
- 需要保存中断状态

阅读XV6实现的自旋锁的代码
github.com/mit-pdos/xv6-riscv

## Read-Copy-Update
针对读写不对称的情况
需要写的时候，持有写锁的情况下，复制一份，写完之后，将指针指向新的版本：牺牲了读写一致性（同一时刻不同线程读到的可能是不一样的）

什么时候回收旧对象？
当所有的CPU都完成一次线程切换后，所有的线程都指向新的版本，此时可以回收（graceful period）


# 应用程序和互斥锁
syscall(SYSCALL_lock, &lk); // 试图获得`lk`，但如果失败，就切换其他线程
syscall(SYSCALL_unlock, &lk); // 释放`lk`，并唤醒（通知）其他线程

应用程序如何进一步减少用户态和内核态的切换？
- fastPath: 在用户态获取锁，若成功直接进入临界区
- slowPath: 使用系统调用，让操作系统获取锁