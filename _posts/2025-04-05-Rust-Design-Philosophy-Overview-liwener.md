---
title: "Rust设计理念概览"
date: 2024-09-16
author: liwener
---

## 所有权与借用

所有权是 Rust 最独特的特性，它让 Rust 无需垃圾回收（garbage collector）即可**保障内存安全**。



### 安全性与未定义行为 (UB)

程序的**安全性**，即程序不存在**未定义行为** (Undefined Behavior, **UB**)。未定义行为（以下简称UB）是指该行为或指令执行后发生的情况是不确定的，具体情况包括但不限于：

- UB 执行时不会任何不良影响
- UB 产生了 [Segmentation fault](https://en.wikipedia.org/wiki/Segmentation_fault)，导致代码崩溃
- UB 在执行不会立即崩溃，但会在特定的恶意输入下造成严重后果

对于可以直接访问内存的低级程序，UB 尤其危险。在低级系统中[报告的安全漏洞中，大约 70%](https://msrc.microsoft.com/blog/2019/07/a-proactive-approach-to-more-secure-code/) 是由内存 UB 引起的。常见的内存 UB 包括但不限于：

- **空指针解引用**
- **悬垂指针（Dangling Pointer）**
- **缓冲区溢出/数组越界访问**
- **数据竞争**
- **访问未初始化内存**
- **双重释放（Double Free）**
- **内存泄漏**

更多 UB 信息参考 Rust 手册维护的 [“Behavior considered undefined”](https://doc.rust-lang.org/reference/behavior-considered-undefined.html) 列表。



### Rust 所有权

Rust 设计的一个基本目标是**确保程序永远不会有未定义行为**，但 UB 通常都是在运行时出现的，而且并不是所有 UB 都会导致不良后果，不同于Java/C#/Python 等语言在**运行时**检查 UB 并抛出异常的策略，Rust 直接在**编译期**进行 UB 的检查，这样做有如下的好处：

- 在编译时捕获错误意味着避免在生产环境中出现这些错误，从而提高软件的**可靠性**。
- 在编译时捕获错误意味着对这些错误的运行时检查更少，从而提高软件的**性能**。

为了实现这点，Rust 提出了**所有权**机制，即 Rust 为了保证内存安全，避免 UB，在**编译器层级**对代码的一系列**约束和准则**，具体如下：

- **All heap data must be owned by exactly one variable.**
  所有堆数据必须只由一个变量拥有。
- **Rust deallocates heap data once its owner goes out of scope.**
  一旦堆数据的所有者超出范围，Rust 就会释放堆数据。
- **Ownership can be transferred by moves, which happen on assignments and function calls.**
  所有权可以通过移动来转移，这发生在赋值和函数调用中。
- **Heap data can only be accessed through its current owner, not a previous owner.**
  堆数据只能通过其当前所有者访问，而不能通过以前的所有者进行访问。



### **Rust 内存模型**

想要了解 Rust 所有权具体是如何工作的，我们需要深入了解 Rust 内存模型。



