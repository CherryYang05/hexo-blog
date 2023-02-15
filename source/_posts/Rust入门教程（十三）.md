---
title: Rust入门教程（十三）：智能指针
mathjax: true
tags:
  - Rust
categories:
  - 编程语言
  - Rust
abbrlink: e9389bfa
date: 2022-07-08 16:50:48
---

>指针是指向一个变量在内存中的地址，在 Rust 中最常见的指针就是引用 `&` 了，没有其他开销。
>智能指针：引用计数智能指针。该智能指针允许你同时拥有同一个数据的多个所有权，它会跟踪每一个所有者并进行计数，当所有的所有者都归还后，该智能指针及指向的数据将自动被清理释放。

<!-- more -->

## 一、智能指针介绍

**引用和智能指针的区别**

- 智能指针往往基于结构体实现
- 引用只借用数据，而智能指针很多时候拥有其指向的数据

**智能指针的例子**

- `String` 和 `Vec<T>`
- 都拥有一片内存区域，且允许用户对其操作
- 还拥有元数据（例如容量等）
- 提供额外的功能保障（String 保证其是合法的 UTF-8 数据）

**智能指针的实现**

- 智能指针通常使用 struct 实现，并且实现了 `Deref` 和 `Drop` 这两个 trait
- `Deref trait`：允许智能指针 struct 的实例像引用一样使用
- `Drop trait`：允许你自定义当智能指针实例走出作用域时的代码

**本章内容**

- 介绍标准库中常见的智能指针
  - `Box<T>`：在 heap内存上分配值
  - `Rc<T>`：启用多重所有权的引用计数类型
  - `Ref<T>` 和 `RefMut<T>`，通过 `RefCelk<T>` 访问：在运行时而不是编译时强制借用规则的类型
- 此外
- 内部可变模式（interior mutability pattern）：不可变类型暴露出可修改其内部值的 API
- 引用循环（reference cycles）：它们如何泄露内存，以及如何防止其发生

## 二、使用 Box\<T>

### 2.1 Box\<T>

`Box<T>` 是最简单的智能指针
- 允许你在 heap 上存储数据（而不是 stack）
- stack 上是指向 heap 数据的指针
- 没有性能开销
- 没有其它额外功能
- 实现了 `Deref trait` 和 `Drop trait`

### 2.2 使用场景

- 在编译时，某类型的大小无法确定。但使用该类型时，上下文却需要知道它的确切大小
- 当你有大量数据，想移交所有权，但需要确保在操作时数据不会被复制
- 使用某个值时，你只关心它是否实现了特定的 trait，而不关心它的具体类型

#### 2.2.1 Box\<T> 如何在 heap 上存储数据

来看一段简单的代码：

```rust
fn main() {
    let x = Box::new(5);
    println!("x = {}", 5);
}
```

如果不用 `Box`，那么就会在栈中创建一个变量 x，而使用了 `Box` 就会在 heap 上创建一个变量。在变量 x 走出作用域时，变量 x 在 stack 上的指针和在 heap 上的值都会被释放。

#### 2.2.2 使用 Box 赋能递归类型

比如有这样一个枚举

```rust
pub enum List {
    Cons(i32, List),
    Nil,
}
```

对于 Cons 变体，里面有一个他类型本身（List），这样就会一直递归下去。但是在编译时，Rust 需要知道一个类型所占的空间大小，而这样的递归类型无法确定其大小。在递归类型中使用 Box 就可以解决上述问题。这也是函数式语言中的 Cons List

**关于 Cons List**

Cons List 是来自 Lisp 语言的一种数据结构。Cons List里每个成员由两个元素组成：
- 当前项的值
- 下一个元素

Cons List 里最后一个成员只包含一个 Nil 值，没有下一个元素。**实际上就是 Rust 中的一种链表**，但他并不是 Rust 的常用集合。

**Rust 如何确定非递归类型所占用的大小的？**

实际上是取结构体或枚举下最大空间的变体的大小作为整个结构体或枚举的大小（非常类似于 C 语言中的联合体 Union）。

因此最终应将代码改成

```rust
use List::{Cons, Nil};

fn main() {
    let list = Cons(1, Box::new(Cons(2, 
        Box::new(Cons(3, 
            Box::new(Nil))))));
}

enum List {
    Cons(i32, Box<List>),
    Nil
}
```

**使用 Box 来获得确定大小的递归类型**

- `Box<T>` 是一个指针，Rust 知道它需要多少空间，因为
  - 指针的大小不会基于它指向的数据的大小变化而变化
- Box<T>:
  - 只提供了“间接”存储和 heap 内存分配的功能
  - 没有其它额外功能
  - 没有性能开销
  - 适用于需要“间接”存储的场景，例如 Cons List
  - 实现了 `Deref trait` 和 `Drop trait`
    - `Deref trait`：可以将 Box 的值当做引用来处理
    - `Drop trait`：定义了当 Box 值走出作用域时，清理掉栈上的指针和堆上的数据

## 三、Deref trait

`Deref` 就是 `dereference` 解引用的意思。

- 实现 Deref Trait 使我们可以**自定义解引用运算符 `*` 的行为**
- 通过实现 Deref，智能指针可像常规引用一样来处理

### 3.1 解引用运算符

常规引用也是指针。

```rust
fn test01() {
    let x = 3;
    let y = &x;
    assert_eq!(x, 3);
    assert_eq!(*y, 3);
}
```

这里没有什么好解释的，和 C 语言一样，`*y` 表示解引用变量 y。

### 3.2 定义自己的智能指针

`Box<T>` 被定义成拥有一个元素的 tuple struct。下面来定义自己的 `MyBox<T>`。

```rust
struct MyBox<T>(T);

impl<T> MyBox<T> {
    fn new(x: T) -> Self {
        MyBox(x)
    }
}

fn main() {
    let x = 5;
    let y = MyBox::new(x);
    assert_eq!(5, x);
    assert_eq!(5, *y);
}
```

先来看定义，`MyBox<T>` 实际上就是一个有名称的元组（tuple），这个元组里只有一个元素。

再看第二个断言，这里的 `*y` 会报错 `type 'MyBox<{integer}>' cannot be dereferenced`，表示 `MyBox` 类型不能被解引用，这是因为 `MyBox` 没有实现 `Deref trait`。

标准库中的 `Deref trait` 要求我们实现一个 `deref` 方法
- 该方法借用 self
- 返回一个指向内部数据的引用

因此我们为 `MyBox` 实现 `Deref`。

```rust
use std::ops::Deref;
...
impl<T> Deref for MyBox<T> {
    type Target = T;
    fn deref(&self) -> &T {
        &self.0
    }
}
```

这里 `type Target = T;` 类似于 C 语言中的 `typedef struct Person {} P`？

而 `*y` 实际上是 `*(y.deref())`，调用 `*` 之前，先执行 `deref` 取引用，然后通过 `*` 运算符解引用。

### 3.3 隐式解引用转化

- 隐式解引用转化（Deref Coercion）是为函数和方法提供的一种便捷特性。
- 假设 T 实现了 Deref trait：
  - Deref Coercion 可以把 T 的引用转化为 T 经过 Deref 操作后生成的引用
- 当把某类型的引用传递给函数或方法时，但它的类型与定义的参数类型不匹配
  - Deref Coercion 就会自动发生
  - 编译器会对 deref 进行一系列调用，来把它转为所需的参数类型
  - 在编译时完成，没有额外性能开销

在上面实现 `MyBox` 和 `Deref` 的前提下，增加以下代码：

```rust
fn hello(s: &str) {
    println!("Hello, {}", s);
}

fn main() {
    let m = MyBox::new(String::from("Cherry"));
    hello(&m);
}
```

我们主要分析 `hello` 函数中传入的参数类型，是一个字符串切片类型，而 `m` 是一个实现了 `Deref` 的结构体类型，但是为什么能传入 `&m` 呢？

首先 `m` 的类型是 `MyBox<String>`，那么 `&m` 的类型就是 `&MyBox<String>`，由于 `MyBox` 实现了 `Deref` 方法，因此 Rust 可以调用 `deref()` 方法，来将 `MyBox<String>` 的引用转化成 `String` 的引用。然而在标准库中，`String` 也实现了 `Deref` 这个 trait，它返回的是一个字符串切片 `&str`，因此 Rust 会继续调用 `deref()`，最终返回一个字符串切片的类型。

而如果 Rust 没有解引用转化功能，则参数应该这样传：`hello(&(*m)[..]);`，而这却相当繁琐。只要类型实现了 `Deref` 这个 trait，Rust 就会自动分析类型，并不断尝试调用 `deref()` 方法来让其与函数或方法签名中的参数类型匹配，而这一切都在编译时执行，因此运行时不会产生额外的性能开销。


**解引用与可变性**

- 可使用 `DerefMut trait` 重载可变引用的 `*` 运算符
- 在类型和 trait 在下列三种情况发生时，Rust 会执行 deref coercion
  - 当 `T: Deref<Target=U>`，允许 `&T` 转换为 `&U`（即类型 `T` 实现了 `Deref trait`，而 `deref` 方法返回的类型是 `U`，那么 `T` 的引用可以转化为 `U` 的引用）
  - 当 `T: DerefMut<Target=U>`，允许 `&mut T` 转换为 `&mut U`
  - 当 `T: Deref<Target=U>`，允许 `&mut T` 转换为 `&U`（反过来不成立，即不能将不可变引用转化为可变引用，违反借用规则）


## 四、Drop trait

- 实现 `Drop trait`，可以让我们自定义**当值将要离开作用域的时候发生的动作**
  - 例如文件、网络资源的释放等
  - 任何类型都可以实现 `Drop trait`
- `Drop trait` 只要求实现 `drop` 方法，其参数是对 `self` 的可变引用
- `Drop trait` 在预导入模块中，无需手动导入

（个人理解：和 C++ 中的析构函数有点类似）

```rust
struct CustomSmartPointer {
    data: String
}

impl Drop for CustomSmartPointer {
    fn drop(&mut self) {
        println!("Dropping CustomSmartPointer with data `{}`", self.data);
    }
}

fn test03() {
    let s1 = CustomSmartPointer {data: String::from("Rust")};
    let s2 = CustomSmartPointer {data: String::from("vscode")};
    println!("CustomSmartPointer created!");
}
```

输出结果为如下，注意输出顺序（变量声明顺序为 s1，s2，Drop 顺序为 s2，s1）

```rust
➜  ~/code/rust/my_box git:(master) ✗ cargo run
   Compiling my_box v0.1.0 (/home/cherry/code/rust/my_box)
    Finished dev [unoptimized + debuginfo] target(s) in 0.23s
     Running `target/debug/my_box`
CustomSmartPointer created!
Dropping CustomSmartPointer with data `vscode`
Dropping CustomSmartPointer with data `Rust`
```

使用 `std::mem::drop` 来提前 drop 值
- 很难直接禁用自动的 drop 功能，也没必要
- Drop trait 的目的就是进行自动的释放处理逻辑
- Rust 不允许手动调用 Drop trait 的 drop 方法
- 若要强行使用 `a.drop()` 这样来调用，会提示 `explicit destructor calls not allowed`，然后后面给的帮助是考虑使用 `drop(a)`
- 但可以调用标准库中的 `std::mem::drop` 函数提前 `drop` 值

我们手动 `drop` 一下：

```rust
fn test03() {
    let s1 = CustomSmartPointer {data: String::from("Rust")};
    drop(s1);
    let s2 = CustomSmartPointer {data: String::from("vscode")};
    println!("CustomSmartPointer created!");
}
```

输出结果为：

```rust
➜  ~/code/rust/my_box git:(master) ✗ cargo run
   Compiling my_box v0.1.0 (/home/cherry/code/rust/my_box)
    Finished dev [unoptimized + debuginfo] target(s) in 0.23s
     Running `target/debug/my_box`
Dropping CustomSmartPointer with data `Rust`
CustomSmartPointer created!
Dropping CustomSmartPointer with data `vscode`
```

尽管手动调用了 `drop` 函数，但是和 `drop` 方法并不会冲突，不会重复释放同一块内存，设计 Rust 语言的时候显然已经考虑到了这一点。

## 五、Rc\<T>：引用计数智能指针

通常情况下，Rust 的所有权都是很清晰的，但是在某些场景中，单个值可能同时被多个所有者持有。例如一个图的数据结构，一个结点有多条边相连，那么这个结点就应该属于所有与其相连的边，只有当所有指向它的边都释放掉，该结点才会被清理，这就是多重所有权。

在 Rust 中，为了支持多重所有权，便有了 `Rc<T>`，即 `reference count（引用计数）`，这个类型会在实例的内部维护一个用于记录引用次数的计数器，从而判断该值是否仍然被使用，可以追踪所有对其的引用。若引用个数为 0，那么该值就会被清理掉，不会发生引用失效的问题。

### 5.1 使用场景及实例

- 需要在 heap 上分配数据，这写数据被程序的多个部分读取（只读），但在编译时无法确定哪个部分最后使用完这些数据
- 若在编译时能够确定哪个部分最后使用完这些数据，那么直接将这个部分程序成为这些数据的所有者即可，这样就只需要靠编译时期所有权规则，就可以保证程序的正确性
- `Rc<T>` 只能用于单线程场景，后面会介绍如何在多线程场景中使用引用计数
- `Rc<T>` 不在预导入模块中，需要手动导入 `use sstd::rc::Rc`
- `Rc::clone(&a)` 函数：增加引用计数
- `Rc::strong_count(&a)`：获得引用计数
- 还有 `Rc::weak_count`函数

例子如下：

![两个List共享所有权](https://raw.githubusercontent.com/CherryYang05/PicGo-image/master/images/20220709214558.png)

要实现两个 List 共享一个 List 的所有权。先看下面的代码：

```rust
fn test04() {
    let a = Cons(3, Box::new(Cons(4, Box::new(Cons(5, Box::new(Nil))))));
    let b = Cons(1, Box::new(a));
    let c = Cons(2, Box::new(a));
}
```

在声明变量 c 处，会报错，提示不能使用被移动的值 `use of moved value: 'a'`。因为在声明 b 时，变量 a 的所有权已经移交给了 b。我们可以改变 `Cons` 的定义，让其持有 `List` 的引用而不是所有权，并为其指定声明周期参数，这个生命周期要求 `List` 中的所有元素的存活时间至少要和 `List` 本身一样。因此借用检查器会阻止我们编译这样的代码：`let a = Cons(1, &Nil)`，因为这里 `Nil` 这个变体值会在 `a` 取得其引用前就被丢弃。
 
另一种办法就是将 `Box` 换成 `Rc`。如下：

```rust
use std::rc::Rc;

enum List {
    Cons(i32, Rc<List>),
    Nil
}

fn test04() {
    let a = Rc::new(Cons(4, Rc::new(Cons(5, Rc::new(Nil)))));
    println!("创建 a 后的的强引用计数为 {}", Rc::strong_count(&a));
    let b = Cons(1, Rc::clone(&a));
    println!("创建 b 后的的强引用计数为 {}", Rc::strong_count(&a));
    {
        let c = Cons(2, Rc::clone(&a));
        println!("创建 c 后的的强引用计数为 {}", Rc::strong_count(&a));
    }
    println!("c 离开作用域后的的强引用计数为 {}", Rc::strong_count(&a));
}
```

最终输出结果为
```rust
➜  ~/code/rust/my_box git:(master) ✗ cargo run
   Compiling my_box v0.1.0 (/home/cherry/code/rust/my_box)
    Finished dev [unoptimized + debuginfo] target(s) in 0.28s
     Running `target/debug/my_box`
创建 a 后的的强引用计数为 1
创建 b 后的的强引用计数为 2
创建 c 后的的强引用计数为 3
c 离开作用域后的的强引用计数为 2
```

其实在 `Rc<T>` 这个类型上也有一个 `clone` 方法，和 `Rc::clone()` 的区别是，后者不会对数据进行深拷贝，只会增加引用计数，速度较快。而前者是类型上的 `clone` 方法，会进行深拷贝，拷贝对象本身，比较耗时。同时 `Rc<T>` 也实现了 `Drop` 这个 trait，因此当变量离开作用域时，引用计数会自动减少一。

在 `Rc<T>` 中，通过不可变引用，使你在程序不同部分之间共享只读数据，若共享的引用可变，将会违反 Rust 的借用规则，即多个指向同一区域的可变引用会导致数据竞争以及数据的不一致。但是在某些情况下，让其共享的数据可变也是非常重要的，这就需要使用 `RefCell<T>`。

### 5.2 RefCell\<T> 和内部可变性

内部可变性（interior mutability）是 Rust 的设计模式之一，它允许你在只持有不可变引用的前提下对数据进行修改，数据结构中使用了 `unsafe` 代码来绕过 Rust 正常的可变性和借用规则，使其可变的借用一个不可变的值。

先来回忆一下 Rust 的借用规则：在任何给定的时刻，要么只能拥有一个可变的引用，要么只能拥有任意数量的不可变引用，且引用总是有效的。

与 `Rc<T>` 不同，`RefCell<T>` 类型代表了其持有数据的唯一所有权。而 `RefCell<T>` 与 `Box<T>` 的区别是：前者只会在**运行时**检查借用规则，否则触发 panic；而后者在编译阶段就要强制代码遵守借用规则，否则出现错误，编译不通过。

**借用规则在不同阶段进行检查的比较**

- 编译阶段
  - 尽早暴露问题
  - 没有运行时的开销
  - 对大多数场景是最佳选择
  - 是 Rust 的默认行为
- 运行时
  - 问题暴露延迟，甚至到生产环境
  - 因为借用计数器而导致性能的损失
  - 实现某些特定的内存安全场景（不可变环境中修改自身数据）

实际上 Rust 编译器是比较保守的，有些代码并不是在编译阶段就能分析明白，针对这些代码，Rust 编译器是无法完成分析的，因此编译器就会简单的拒绝所有不符合所有权规则的代码，哪怕这些代码并没有任何问题，这就是 Rust 编译器的保守性。因为一旦 Rust 将某一段有问题的程序通过了，那么 Rust 对安全性的保证将直接破产，尽管拒绝掉某些正确的代码会对开发者带来不便，但是至少不会在生产中带来灾难性的后果。针对 Rust 编译器无法分析的代码，如果开发者能够确保代码能够满足借用规则，这时候就要用到 `RefCell<T>` 了。

和 `Rc<T>` 一样 `RefCell<T>` 只能用于单线程的场景。

- `Box<T>`
  - 同一个数据只有一个所有者
  - 可变和不可变借用，在编译时检查
- `Rc<T>`
  - 同一个数据可以有多个所有者
  - 不可变借用，在编译时检查
- `RefCell<T>`
  - 同一个数据只有一个所有者
  - 可变和不可变借用，在运行时检查
  - 其中，即使 `RefCell<T>` 本身不可变，但是仍能修改其中存储的值

对于内部可变性，可以可变的借用一个不可变的值。在某些情况下，我们需要一个值，它对外部是不可变的，但能在方法内部修改自身的值，除了这个值本身的方法，其他的方法都不能修改这个值，`RefCell<T>` 正是获得这种内部可变性的一种方法。但是这种方法并没有完全绕开借用规则，只是通过内部可变性通过了编译检查，但是借用检查也只是从编译阶段延后到运行阶段，如果运行阶段仍然违反了借用规则，那么将会 panic，而不是编译错误。

**使用 `RefCell<T>` 在运行时记录借用信息**

- `RefCell<T>` 会记录当前存在多少个活跃的 `Ref<T>` 和 `RefMut<T>` 智能指针：
  - 每次调用 borrow：不可变借用计数加1
  - 任何一个 `Ref<T>` 的值离开作用域被释放时：不可变借用计数减 1
  - 每次调用 `borrow_mut`：可变借用计数加1
  - 任何一个 `RefMut<T>` 的值离开作用域被释放时：可变借用计数减 1
- 以此技术来维护借用检查规则：
  - 任何一个给定时间里，只允许拥有多个不可变借用或一个可变借用

**其它可实现内部可变性的类型**

- `Cell<T>`：通过复制来访问数据
- `Mutex<T>`：用于实现跨线程情形下的内部可变性模式

## 六、循环引用导致内存泄露

### 6.1 Rust 可能发生内存泄漏

- Rust 的内存安全机制可以保证很难发生内存泄漏，但不是不可能
- 例如使用 `Rc<T>` 和 `RefCell<T>` 就可能创造出循环引用，从而发生内存泄漏：
  - 每个项的引用数量不会变成 0，值也不会被处理掉

例子如下：

```rust
use crate::List::{Cons, Nil};
use std::{rc::Rc, cell::RefCell};

#[derive(Debug)]
enum List {
    Cons(i32, RefCell<Rc<List>>),
    Nil
}

impl List {
    fn tail(&self) -> Option<&RefCell<Rc<List>>> {
        match self {
            Cons(_, item) => Some(item),
            Nil => None,
        }
    }
}

fn main() {
    test02();
}

fn test02() {
    let a = Rc::new(Cons(5, RefCell::new(Rc::new(Nil))));
    println!("a initial rc count is {}", Rc::strong_count(&a));
    println!("a next item is {:?}", a.tail());

    let b = Rc::new(Cons(10, RefCell::new(Rc::clone(&a))));
    println!("a rc count after b creation is {}", Rc::strong_count(&a));
    println!("b initial rc count is {}", Rc::strong_count(&b));
    println!("b next item is {:?}", b.tail());

    if let Some(link) = a.tail() {
        *link.borrow_mut() = Rc::clone(&b);
    }

    println!("a rc count after changing a is {}", Rc::strong_count(&a));
    println!("b rc count after changing a is {}", Rc::strong_count(&b));

    // println!("a next item is {:?}", a.tail());
}
```

先创建 `a`，`Cons` 第一个元素是 5，第二个元素是 `Nil`，然后输出 a 的强引用，输出 a 的 `Cons` 的第二个元素。同理创建 b，`Cons` 第一个元素是 10，第二个元素引用 a，然后输出此时 a 的强引用和 b 的强引用。

这个时候将 a 的 Cons 第二个元素改成 b 的引用，即这个 List 的 a 和 b 收尾相连了，形成了一个类似于循环链表的结构。这时我们观察输出结果为：

```rust
➜  smart_pointer git:(master) ✗ cargo run
   Compiling smart_pointer v0.1.0 (/Users/cherry/Code/Rust/learning/smart_pointer)
    Finished dev [unoptimized + debuginfo] target(s) in 0.25s
     Running `target/debug/smart_pointer`
a initial rc count is 1
a next item is Some(RefCell { value: Nil })
a rc count after b creation is 2
b initial rc count is 1
b next item is Some(RefCell { value: Cons(5, RefCell { value: Nil }) })
a rc count after changing a is 2
b rc count after changing a is 2
```

收尾相连后 a 和 b 的强引用计数都是 2（很显然）。但是这个时候如果要输出 a 的下一个元素，就会发生栈溢出。因为 a 和 b 形成了循环，a 的下一个元素是 b，但是 b 中又包含 a，a 中又包含 b... 如此往复，b 的大小其实是无穷大的，因此会导致**栈溢出**。取消上面的注释行，再次运行，会得到下面的结果：

```rust
...RefCell { value: Cons(5, RefCell { value: Cons(10, RefCell { value: Cons(5, RefCell { value: Cons(10, RefCell { value: Cons(5, RefCell { value: Cons(10, RefCell { value: Cons(5, RefCell { value: Cons(10, RefCell { value: Cons(5, RefCell { value: Cons(10, RefCell { value: Cons(5, RefCell { value: Cons(10, RefCell { value: Cons(5, RefCell { value: Cons(10, RefCell { value: Cons(5, RefCell { value: Cons(10, RefCell { value: Cons(5, RefCell { value: Cons(10, RefCell { value: Cons(5, RefCell { value: Cons(10, RefCell { 
thread 'main' has overflowed its stack
fatal runtime error: stack overflow
[1]    28287 abort      cargo run
```

### 6.2 防止内存泄漏的解决办法

- 依靠开发者来保证，不能依靠 Rust
- 重新组织数据结构：一些引用来表达所有权，一些引用不表达所有权
  - 循环引用中的一部分具有所有权关系，另一部分不涉及所有权关系
  - 而只有所有权关系才影响值的清理

**防止循环引用把 `Rc<T>` 换成 `Weak<T>`**

- `Rc::clone` 为 `Rc<T>` 实例的 strong_count 加 1，`Rc<T>` 的实例只有在 strong_count 为 0 的时候才会被清理
- `Rc<T>` 实例通过调用 `Rc::downgrade` 方法可以创建值的 Weak Reference（弱引用）
  - 返回类型是 `Weak<T>`（智能指针）
  - 调用 `Rc::downgrade` 会为 weak_count 加1
- `Rc<T>` 使用 weak_count 来追踪存在多少 `Weak<T>`
- weak_count 不为 0 并不影响 `Rc<T>` 实例的清理


**Strong vs Weak**

- Strong Reference（强引用）是关于如何分享 `Rc<T>` 实例的所有权
- Weak Reference（弱引用）并不表达上述意思
- 使用 Weak Reference 并不会创建循环引用：
  - 当 Strong Reference 数量为 0 的时候，Weak Reference 会自动断开
- 在使用 `Weak<T>` 前，需保证它指向的值仍然存在：
  - 在 `Weak<T>` 实例上调用 `upgrade` 方法，返回 `Option<Rc<T>>`

### 6.3 实现树的数据结构的例子

```rust
#[derive(Debug)]
struct Node {
    value: i32,
    children: RefCell<Vec<Rc<Node>>>,
}

fn test03() {
    let leaf = Rc::new(Node {
        value: 3,
        children: RefCell::new(vec![])
    });

    let branch = Rc::new(Node {
        value: 5,
        children: RefCell::new(vec![Rc::clone(&leaf)])
    });
}
```

新建一个叶子结点值为 3，该叶子结点没有孩子结点，再创建一个分支结点，作为叶节点 3 的父节点。此时该叶子结点拥有两个强引用，即叶子结点本身和父节点拥有该叶子结点的所有权。我们可以通过分支结点访问到叶节点，因为他拥有叶节点的引用（这里也是所有权），而叶子结点无法访问到父节点，因为他没有拥有父节点的所有权或引用。

上面这种双向的引用形成了循环引用，这个时候就可以使用 `Weak<T>` 来避免产生循环引用。

我们修改上面的代码：

```rust
#[derive(Debug)]
struct Node {
    value: i32,
    parent: RefCell<Weak<Node>>,
    children: RefCell<Vec<Rc<Node>>>,
}

fn test03() {
    let leaf = Rc::new(Node {
        value: 3,
        parent: RefCell::new(Weak::new()),
        children: RefCell::new(vec![])
    });

    println!("leaf parent is {:?}", leaf.parent.borrow().upgrade());
    let branch = Rc::new(Node {
        value: 5,
        parent: RefCell::new(Weak::new()),
        children: RefCell::new(vec![Rc::clone(&leaf)])
    });

    *leaf.parent.borrow_mut() = Rc::downgrade(&branch);
    println!("leaf parent is {:?}", leaf.parent.borrow().upgrade());
}
```

先调用 leaf 的 parent 字段的 `borrow` 方法获取其不可变引用，然后通过 `upgrade` 方法将 `Weak<T>` 转化成 `Rc<T>`，然后再调用 leaf 的 parent 字段的 `borrow_mut` 方法获取其可变引用，然后通过调用 `Rc::downgrade` 将 branch 里的 `Rc<Node>` 转化成 `Weak<Node>` 并存在 leaf.parent 里。

输出结果如下：

```rust
➜  smart_pointer git:(master) ✗ cargo run
   Compiling smart_pointer v0.1.0 (/Users/cherry/Code/Rust/learning/smart_pointer)
    Finished dev [unoptimized + debuginfo] target(s) in 0.14s
     Running `target/debug/smart_pointer`
leaf parent is None
leaf parent is Some(Node { value: 5, parent: RefCell { value: (Weak) }, children: RefCell { value: [Node { value: 3, parent: RefCell { value: (Weak) }, children: RefCell { value: [] } }] } })
```

我们通过分别打印 leaf 和 branch 的强弱引用来深入理解一下本小节内容：

```rust
fn test03() {
    let leaf = Rc::new(Node {
        value: 3,
        parent: RefCell::new(Weak::new()),
        children: RefCell::new(vec![]),
    });

    println!(
        "leaf strong = {}, weak = {}",
        Rc::strong_count(&leaf),
        Rc::weak_count(&leaf)
    );

    {
        let branch = Rc::new(Node {
            value: 5,
            parent: RefCell::new(Weak::new()),
            children: RefCell::new(vec![Rc::clone(&leaf)]),
        });

        *leaf.parent.borrow_mut() = Rc::downgrade(&branch);

        println!(
            "leaf strong = {}, weak = {}",
            Rc::strong_count(&leaf),
            Rc::weak_count(&leaf)
        );
        println!(
            "branch strong = {}, weak = {}",
            Rc::strong_count(&branch),
            Rc::weak_count(&branch)
        );
    }
    println!("leaf parent is {:?}", leaf.parent.borrow().upgrade());
    println!(
        "leaf strong = {}, weak = {}",
        Rc::strong_count(&leaf),
        Rc::weak_count(&leaf)
    );
}
```

输出结果如下：

```rust
➜  smart_pointer git:(master) ✗ cargo run
   Compiling smart_pointer v0.1.0 (/Users/cherry/Code/Rust/learning/smart_pointer)
    Finished dev [unoptimized + debuginfo] target(s) in 0.35s
     Running `target/debug/smart_pointer`
leaf strong = 1, weak = 0
leaf strong = 2, weak = 0
branch strong = 1, weak = 1
leaf parent is None
leaf strong = 1, weak = 0
```

刚创建 leaf 时，只有一个 leaf 的强引用，没有弱引用。然后在一个新的作用域内创建 branch，将 branch 和 leaf 关联起来，此时 branch 强引用只有一个（branch 自身），弱引用有一个（leaf）。leaf 强引用有两个，自身和 branch。然后 branch 走出了作用域，强引用计数减 1 变成 0，内存被释放，branch 便不存在了，此时通过 leaf 访问 branch 显然为 None，最后再输出 leaf 的强引用为 1，即 leaf 自身。
