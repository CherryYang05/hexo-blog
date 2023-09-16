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

## 声明宏

先来看几个例子。

```rust
macro_rules! test {
    () => {
        10
    };
}
let a = test!();    // or test![] or test!{}
assert_eq!(10, a);
```

这是最简单的一个宏，执行的时候，从左侧小括号匹配规则，从右侧进行，这里是直接返回一个数字。使用起来也很容易，宏的使用，用小括号，中括号，大括号都可以。

### 给宏传递参数

宏的匹配能力非常强，还可以匹配变量，表达式，函数等。

```rust
macro_rules! test {
    ($var: expr) => {
        $var
    };
}
let a = test!(20);
assert_eq!(20, a);
```

这里的 `$e` 是自己定的，写成 `$a`, `$b`, `$foo` 之类的都可以，不过还是推荐写得语义化一些。`expr` 是表达式 `expression` 的缩写，此外还有其他类型。

- item: 结构体，函数，mod 之类的
- block: 用大括号包起来的语句或者表达式，也就是代码块
- stmt: 一段 statement
- pat: 一段 pattern
- ty: 一个类型
- ident: 标识符
- path: 类似 foo::bar 这种路径
- meta: 元类型，譬如 #[...], #![...] 内的东西
- tt: 一个 token tree

以 `pat` 为例。

```rust
macro_rules! test {
    ($var: expr, $pattern: pat) => {
        match $var {
            $pattern => true,
            _ => false
        }
    }
}

let c = Some(true);
let d = test!(c, Some(true));
println!("{}", d);
```

上述的三个宏可以合到一起。

```rust
macro_rules! test {
    () => {
        10
    };
    ($var: expr) => {
        $var
    };
    ($var: expr, $pattern: pat) => {
        match $var {
            $pattern => true,
            _ => false
        }
    }
}
```

### 重复

这是 Rust Book 的宏那一节的例子，`*` 表示重复使用 `$()` 包裹的内容来处理传进来的值，`*` 前的 `,` 是参数的分隔符，`*` 可以换成正则表达式中的 `+` 或 `?`。这个例子中可以处理传入的多个数字。

```rust
macro_rules! vec {
    ( $( $x:expr ),* ) => {
        {
            let mut temp_vec = Vec::new();
            $(
                temp_vec.push($x);
            )*
            temp_vec
        }
    };
}
fn main() {
    let v = vec![1, 2, 3];
    println!("{:?}", v);
}
```

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
// TODO
