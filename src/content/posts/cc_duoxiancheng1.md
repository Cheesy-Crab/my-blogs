---
title: 并发1
published: 2026-05-12
tags: [ 操作系统, 多线程 ]
category: 操作系统

draft: false
---

# 多线程入门

1. 最简单的多线程程序
2. 多线程程序的基础性质
 - 共享内存：证明
 - 独立堆栈：证明and求堆栈大小

作业：使用多线程实现最长公共子串


## 线程模型
### 简化的线程API

```phython
# spawn 创建入口函数为fn的线程
# fn：给定函数指针 void fn(int tid) {...}
spawn(fn)
# join 等待所有的线程返回
join()
```
### 例子
多线程例一
```
def T_print(name):
    heap.n += 1
    sys_sched() # 主动随机切换线程
    sys_write(f'{name}{heap.n}')
    sys_sched()

def main():
    heap.n = 0
    sys_spawn(T_print, 'A')
    sys_spawn(T_print, 'B')
```
多线程例二
```
#inclue "thread.h"
void T_a() {
    while(1) {
        printf("a");
    }
}
void T_b() {
    while(1) {
        printf("b");
    }
}

int main() {
    create(T_a);
    create(T_b);
}
```

# 并发编程打破的假设

## NP完全？

## 原子性
- **任何时候，load读到的值都可能是别的线程写入的**
```C
// 案例一：支付
unsiged long balance = 100;
void Alipay_withdraw(int amout) {
    if (balance >= amout) {
        usleep(1);
        balance -= amout;
    }
}

void T_alipay() {
    Alipay_witchdraw(100);
}

int main() {
    create(T_alipay);
    create(T_alipay);
    join();
    printf("balance:%d\n", amout);
}
// 案例二：并发求和
// 足够多的线程求和，最后可能得到的最小值为2
```

## 程序顺序执行（汇编层面）

- 编译器也做了同样的假设（编译器会试图优化状态迁移，改变执行流，
  - `while(!flag)` 最大优化下，只读一次flag，（flag是共享内存）

## 全局指令顺序（处理器层面）

处理器在运行时进行动态分析 **Memory Models by Russ Cox**

- cache miss：等待刷新cache，再执行下一条；直接执行下一条，反正下一条没有影响，等cache执行完再执行这条

**Linux:  | read -n 10000 | sort | uniq -c**

**内存模型**