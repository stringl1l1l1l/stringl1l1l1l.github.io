---
title: "Rust设计理念概览"
date: 2025-04-05
author: liwener
---

## **所有权与借用**

所有权是 Rust 最独特的特性，它让 Rust 无需垃圾回收（garbage collector）即可保障内存安全。



### **安全性与未定义行为 (UB)**

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

更多 UB 信息参考 Rust 手册维护的 [Behavior considered undefined](https://doc.rust-lang.org/reference/behavior-considered-undefined.html) 列表。



### **Rust 所有权**

Rust 设计的一个基本目标是**确保程序永远不会有未定义行为**，但 UB 通常都是在运行时出现的，而且并不是所有 UB 都会导致不良后果，不同于Java/C#/Python 等语言在**运行时**检查 UB 并抛出异常的策略，Rust 直接在**编译期**进行 UB 的检查，这样做有如下的好处：

- 在编译时捕获错误意味着避免在生产环境中出现这些错误，从而提高软件的**可靠性**。

- 在编译时捕获错误意味着对这些错误的运行时检查更少，从而提高软件的**性能**。

  

为了实现这点，Rust 提出了**所有权**机制，即 Rust 为了保证内存安全，避免 UB，在**编译器层级**对代码的一系列**约束和准则**，具体如下：

- **All heap data must be owned by exactly one variable.**  
  所有堆数据必须只由一个变量拥有。
- **Rust deallocates heap data once its owner goes out of scope. **  
  一旦堆数据的所有者超出范围，Rust 就会释放堆数据。
- **Ownership can be transferred by moves, which happen on assignments and function calls.**  
  所有权可以通过移动来转移，这发生在赋值和函数调用中。
- **Heap data can only be accessed through its current owner, not a previous owner.**  
  堆数据只能通过其当前所有者访问，而不能通过以前的所有者进行访问。

想要了解 Rust 所有权准则具体是如何工作的，我们需要深入了解 Rust 内存模型。此处我会假设你已经对**栈 (stack)**、**栈帧 (frame)**、**堆 (heap)**、**指针 (pointer)**和 **C++内存模型**这些概念有基本的了解。



#### Rust 自动释放堆内存

C/C++将堆区内存管理完全交给程序员，导致了无数的内存泄漏 bugs。Rust 从中吸取了教训，不允许在程序中调用类似`free()`的函数手动释放内存，而是自动释放堆内存。

但 Rust 如何做到这一点？不严谨但简洁地描述：**当 Rust 释放变量的 frame 时，Rust 会自动释放变量的堆内存**。

<img src="https://github.com/stringl1l1l1l/stringl1l1l1l.github.io/blob/master/_posts/assets/image-20250405181514331.png?raw=true" alt="image-20250405181514331" style="zoom: 50%;" />

我们用上面的示例代码解释 Rust 如何自动释放堆内存。在 L1 处， `make_and_drop` 尚未调用，此时内存中保存 `main` 的栈帧。执行到 L2 处，调用 `make_and_drop` 时，`a_box` 指向堆上的 `5`。 一旦 `make_and_drop` 完成，Rust 就会释放其栈帧。`make_and_drop` 包含变量 `a_box`，因此 Rust 也会释放 `a_box` 指向的堆内存数据。因此，堆在 L3 处为空。



#### 变量所有权

然而，如果 Rust 只像上面这样做，可能会导致一些额外的问题。如下，两个变量同时绑定到一块堆内存。当程序执行到 L1 处时，`main` 栈帧中拥有两个指向同一块堆内存 `Box::new([0; 1_000_000])` 的变量 `a` 和 `b`。执行到 L2 处时，Rust 会释放 `main` 堆栈，同时自动释放栈帧中所有变量的堆内存，导致了**双重释放（Double Free）**问题。

```rust
fn main() {
    let a = Box::new([0; 1_000_000]);
    let b = a; 	// L1
} // L2
```

为了避免该问题，Rust 最终提出了**所有权 (Owenership)**。当 `a` 绑定到 `Box::new([0; 1_000_000])` 时，我们说 `a` **拥有 (owns)** 这个 box。语句 `let b = a` 将堆内存的所有权从 `a` 移动 (move) 到 `b`。

综上所述，Rust 自动释放堆内存的策略的精确描述为：**如果一个变量拥有一个 box，当 Rust 释放变量的 frame 时，Rust 会释放 box 的堆内存。**

