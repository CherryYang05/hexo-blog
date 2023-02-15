---
title: Rust入门教程（九）：Rust 宏编程
mathjax: true
tags:
  - Rust
categories:
  - 编程语言
  - Rust
abbrlink: c3f8af28
date: 2022-07-02 11:26:35
---

>C/C++、Rust 等语言中，宏编程一直是书本上讲解很少但是在实际开发中却及其重要的内容。宏展开在编译期发生，并没有运行期的性能损耗。Rust 宏分为声明宏和过程宏。

【未完】

<!-- more -->

## 

## 过程宏

过程宏必须定义在一个独立的 crate 中。

【解释】：过程宏是在编译一个 crate 之前，对 crate 的代码进行加工的一段程序，这段程序也是需要编译后执行的。如果定义过程宏和使用过程宏的代码写在一个 crate 中，那么就会陷入死锁：
- 要编译的代码首先需要运行过程宏来展开，否则代码就是不完整的，没法编译 crate
- 不能编译 crate，那么其中的过程宏代码就没法执行，就不能展开被过程宏装饰的代码

要开发 rust 过程宏，需要在 `Cargo.toml` 文件中添加必备的三个依赖包：

```rust
[dependencies]
proc-macro2 = "1.0.7"
quote = "1"
syn = { version = "1.0.56", features = {"full"} }
```

