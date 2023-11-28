---
title: Rust入门教程（十四）：异步编程
mathjax: true
date: 2023-11-02 15:54:33
tags:
  - Rust
categories:
  - 编程语言
  - Rust
---

>Rust 异步编程

<!-- more -->

## 一、异步编程介绍



1. 有两种方式调用异步函数，一种是 `block_on()`，另一种是 `await`。

`block_on()` 函数会阻塞，不会继续向下执行。

2. 在异步函数中执行异步操作需要用 `await` 来 “激活” 该函数，因为 `Future` 本身不会自己做任何事，它是一个惰性函数，需要被调用才能执行。
在异步函数或异步块中，通过 `await` 来执行，在非异步函数中通过执行器的 `block_on()` 来执行。

1. 用 `sleep()` 阻塞某个函数，但是线程并不会切换到另一个函数执行。因为使用的 `sleep()` 函数是标准库中的函数，我们应该使用异步包中的函数，例如 `tokio`、`async-std`、`futures` 的 `crate`。在这些库中，实现了 `executor`、`reactor` 等功能，也实现了 `sleep()` 等函数。

原因：

4. `Future` 会返回 `Ready` 或 `Pending` 两种状态，当一个函数执行完会通知 `Reactor`，然后 `Reactor` 通知 `executor`。



