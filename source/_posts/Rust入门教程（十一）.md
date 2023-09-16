---
title: Rust入门教程（十一）：闭包和迭代器
mathjax: true
tags:
  - Rust
categories:
  - 编程语言
  - Rust
abbrlink: 1b9cba25
date: 2022-07-08 16:49:31
---

> 闭包（closures）是可以保存在一个变量中或作为参数传递给其他函数的匿名函数，即可以捕获其所在环境的匿名函数。 可以在一个地方创建闭包，然后在不同的上下文中执行闭包运算。不同于函数，闭包允许捕获调用者作用域中的值。我们将展示闭包的这些功能如何复用代码和自定义行为。
> 函数式编程的特点有：将函数作为参数或者作为其他函数的返回值，以及将函数赋值给一个变量，这些都是函数式编程的常见特点

<!-- more -->

## 一、闭包

### 1.1 什么是闭包

- 是匿名函数
- 保存为变量、作为参数
- 可在一个地方创建闭包，然后在另一个上下文中调用闭包来完成运算
- 可从其定义的作用域捕获值

### 1.2 例子：生成自定义运动计划的程序

- 该算法的逻辑并不是重点，重点是算法中的计算过程需要几秒钟时间。
- 目标：不让用户发生不必要的等待
  - 仅在必要时调用该算法

例子如下：

```rust
fn simulated_expensive_calculation(intensity: u32) -> u32{
    println!("calculating slowly...");
    thread::sleep(Duration::from_secs(3));
    intensity
}

fn generate_workout(intensity: u32, random_number: u32) {
    if intensity < 25 {
        println!(
            "Today, do {} pushups!",
            simulated_expensive_calculation(intensity)
        );
        println!(
            "Next, do {} situps!",
            simulated_expensive_calculation(intensity)
        );
    } else {
        if random_number == 3 {
            println!("Take a break today!");
        } else {
            println!(
                "Today, run for {} minutes!",
                simulated_expensive_calculation(intensity)
            );
        }
    }
}
```

我们用函数 `simulated_expensive_calculation` 模拟复杂的计算，我们不希望调用多次该函数，因为非常耗时，对用户不友好，因此首先想出的优化方案就是用一个变量接收该函数值，当 `generate_workout` 进入条件语句时，便只需要执行一次即可，如下：

```rust
fn generate_workout(intensity: u32, random_number: u32) {
    let res = simulated_expensive_calculation(intensity);
    if intensity < 25 {
        println!("Today, do {} pushups!", res);
        println!("Next, do {} situps!", res);
    } else {
        if random_number == 3 {
            println!("Take a break today!");
        } else {
            println!(
                "Today, run for {} minutes!",
                simulated_expensive_calculation(intensity)
            );
        }
    }
}
```

然而这样又会产生一个新的问题：当进入 `else` 时，随机数值为 3 的时候，是无需执行复杂计算的，这时候用一个变量接收该复杂计算的函数值便会显得浪费。我们真正希望的是，函数定义单独在一个地方，等到函数真正被用到时再被执行，这就是闭包的功能。代码如下：


```rust
fn generate_workout(intensity: u32, random_number: u32) {
    let expensive_closure = |num| {
        println!("calculating slowly...");
        thread::sleep(Duration::from_secs(3));
        num
    };

    if intensity < 25 {
        println!("Today, do {} pushups!", expensive_closure(intensity));
        println!("Next, do {} situps!", expensive_closure(intensity));
    } else {
        if random_number == 3 {
            println!("Take a break today!");
        } else {
            println!("Today, run for {} minutes!", expensive_closure(intensity));
        }
    }
}
```

其中 `expensive_closure` 只是定义了一个匿名函数，并没有执行。当然在条件语句中，该闭包还是执行了两次，对于这里的优化，后面会讲到。

### 1.3 闭包的类型推断和标注

- 闭包不要求标注参数和返回值的类型，和函数不同，无需对外暴露接口
- 闭包通常很短小，只在狭小的上下文中工作，编译器通常能推断出类型
- 可以手动添加类型标注
- 注意：闭包的定义最终只会为参数/返回值推断出唯一具体的类型
 
例子：

```rust
let example_closure = |x| x;
let s = example_closure(String::from(Hello));
let a = example_closure(3);
```

变量 `s` 传给该闭包一个字符串类型，编译器便推断出来闭包中参数 `x` 是字符串类型，便与其绑定，因此变量 `a` 再传入一个整型便会报错。

### 1.4 使用泛型参数和 Fn Trait 来存储闭包

#### 1.4.1 继续解决 1.2 中的例子

除了创建局部变量存储闭包的值，还有另一种解决方案：

创建一个 struct，它持有闭包及其调用结果，只会在需要结果时才执行该闭包，可缓存结果。
这个模式通常叫做**记忆化（memoization）**或**延迟计算（lazy evaluation）**

**如何让 struct 持有闭包**

- struct 的定义需要知道所有字段的类型
  - 需要指明闭包的类型
- 每个闭包实例都有自己唯一的匿名类型，即使两个闭包签名完全一样
- 所以需要使用：泛型和 Trait Bound（第10章）

**Fn Trait**

- Fn traits 由标准库提供
- 所有的闭包都至少实现了以下 trait 之一：
  - Fn
  - FnMut
  - FnOnce

代码如下：

```rust
struct Cacher<F>
where 
    F: Fn(u32) -> u32
{
    calculation: F,
    value: Option<u32>
}

impl<F> Cacher<F>
where
    F: Fn(u32) -> u32
{
    fn new(calculation: F) -> Cacher<F> {
        Cacher {
            calculation,
            value: None
        }
    }

    fn value(&mut self, arg: u32) -> u32 {
        match self.value {
            Some(v) => v,
            None => {
                let v = (self.calculation)(arg);
                self.value = Some(v);
                v
            }
        }
    }
}
```

首先定义了一个结构体，该结构体泛型参数要实现 Fn trait，然后为该结构体实现 `new` 和 `value` 函数（方法），如果已经执行过该闭包，则返回值，若没有执行过则执行闭包，将值存进结构体变量中。`generate_workout` 函数实现如下：


```rust
fn generate_workout(intensity: u32, random_number: u32) {
    let mut expensive_closure = Cacher::new(|num| {
        println!("calculating slowly...");
        thread::sleep(Duration::from_secs(3));
        num
    });

    if intensity < 25 {
        println!("Today, do {} pushups!", expensive_closure.value(intensity));
        println!("Next, do {} situps!", expensive_closure.value(intensity));
    } else {
        if random_number == 3 {
            println!("Take a break today!");
        } else {
            println!("Today, run for {} minutes!", expensive_closure.value(intensity));
        }
    }
}
```

#### 1.4.2 使用缓存器（Cacher）实现的限制

1. Cacher 实例假定针对不同的参数 arg，value 方法总会得到同样的值
   - 可以使用 HashMap 代替单个值：
     - key: arg 参数
     - value：执行闭包的结果
2. 只能接收一个 u32 类型的参数和 u32 类型的返回值
   - 引入两个或多个泛型参数

### 1.5 使用闭包捕获环境

#### 1.5.1 利用闭包捕获环境中的变量

闭包可以捕获他们所在的环境

- 闭包可以访问定义它的作用域内的变量，而普通函数则不能
- 会产生额外内存开销

例子：

```rust
let x = 3;
let equal_to_x = |z| z == x;
let y = 4;
assert!(equal_to_x(y));
```

上述代码中，闭包内的变量 `x` 并不是在闭包内定义的，但是却可以访问，因为闭包可以捕获和其在**同一作用域内**的其他变量，而函数却没有这样的作用。

**闭包从所在环境捕获值的方式**

与函数获得参数的三种方式一样：
- 取得所有权：`FnOnce`
- 可变借用：`FnMut`
- 不可变借用： `Fn`

创建闭包时，通过闭包对环境值的使用，Rust 推断出具体使用哪个 frait：
- 所有的闭包都实现了 FnOnce
- 没有移动捕获变量的实现了 FnMut
- 无需可变访问捕获变量的闭包实现了 Fn

注：实现了 `Fn trait` 的闭包一定实现了 `Fn Mut`，实现了 `Fn Mut` 一定实现了 `Fn Once`。

#### 1.5.2 move 关键字

在参数列表前使用 move 关键字，可以强制闭包取得它所使用的环境值的所有权。当将闭包传递给新线程以移动数据使其归新线程所有时，此技术最为有用。

例子如下：

```rust
let x = vec![1, 2, 3];
let equal_to_x = move |z| z == x;
println!("can't use x here：{:?}", x);
let y = vec![1, 2, 3];
assert!(equal_to_x(y));
```

这里便不能使用 `x` 变量了。

**最佳实践**
当指定 Fn trait bound 之一时，首先用 Fn，基于闭包体里的情况，如果需要 FnOnce 或 FnMut，编译器会再告诉你。
（面向编译器编程实锤 o_O）

## 二、迭代器

**什么是迭代器**

- 迭代器模式：对一系列项执行某些任务
- 迭代器负责：
  - 遍历每个项
  - 确定序列（遍历）何时完成

**Rust 的迭代器：**

- 懒惰的：除非调用消费迭代器的方法，否则迭代器本身没有任何效果。

先用一个最简单的迭代器的例子来进入本节的学习：

```rust
fn main() {
    let v1 = vec![3, 9, 100];
    let v1_iter = v1.iter();

    for val in v1_iter {
        println!("Got: {}", val);
    }
}
```

output：

```rust
➜  it git:(master) ✗ cargo run   
   Compiling it v0.1.0 (/Users/cherry/Code/Rust/learning/it)
    Finished dev [unoptimized + debuginfo] target(s) in 0.33s
     Running `target/debug/it`
Got: 3
Got: 9
Got: 100
```

### 2.1 Iterator trait 和 next 方法

- 所有迭代器都实现了 Iterator trait
- Iterator trait 定义于标准库，定义大致如下

```rust
pub trait Iterator {
    type ltem;
    fn next(&mut self) -> Option<Self::tem>;
        // methods with default implementations elided
}
```

- type Item 和 Self::Item 定义了与此该 trait 关联的类型
  - 实现 Iterator trait 需要你定义一个 Item 类型，它用于 next 方法的返回类型（迭代器的返回类型）

**Iterator trait 仅要求实现一个方法：next**
- next:
  - 每次返回迭代器中的一项
  - 返回结果包裹在 Some 里
  - 迭代结束，返回 None

```rust
let mut v1_iter_mut = v1.iter();
assert_eq!(v1_iter_mut.next(), Some(&3));
assert_eq!(v1_iter_mut.next(), Some(&9));
assert_eq!(v1_iter_mut.next(), Some(&100));
println!("{}", v1_iter_mut.len());
```

上述的例子要定义一个可变的迭代器，因为 `next` 方法会更改迭代器内部的用来标示顺序的某些值，而上面的 for 之所以不用定义迭代器为可变，是因为用 for 来进行循环，实际上是取得了该迭代器的所有权，在其内部已经将其变成可变的了。

需要注意的是，next 方法是一种消耗型行为，我们最后输出了迭代器 `v1_iter_mut` 的长度，结果为 0。

**几个迭代方法**

- iter 方法：在不可变引用上创建迭代器
- into_iter 方法：创建的迭代器会获得所有权
- iter_mut 方法：迭代可变的引用

### 2.2 消耗/产生迭代器

#### 2.2.1 消耗迭代器的方法

- 在标准库中，Iterator trait 有一些带默认实现的方法
- 其中有一些方法会调用 next 方法
  - 实现 Iterator frait 时必须实现 nex† 方法的原因之一
- 调用 next 的方法叫做“消耗型适配器”
  - 因为调用它们会把迭代器消耗尽
- 例如：sum 方法（就会耗尽迭代器）
  - 取得迭代器的所有权
  - 通过反复调用 next，遍历所有元素
  - 每次迭代，把当前元素添加到一个总和里，迭代结束，返回总和

```rust
fn test02() {
    let v1 = vec![4, 5, 8];
    let sum: i32 = v1.iter().sum();
    assert_eq!(sum, 17);
}
```

使用 `sum` 的时候要注意显示声明类型。


#### 2.2.2 产生其它迭代器的方法(map)

- 定义在 Iterator trait 上的另外一些方法叫做“迭代器适配器”
  - 把迭代器转换为不同种类的迭代器
- 可以通过链式调用使用多个迭代器适配器来执行复杂的操作，这种调用可读性较高。
- 例如：map
  - 接收一个闭包，闭包作用于每个元素
  - 产生一个新的迭代器
- collect 方法：消耗型适配器，把结果收集到一个集合类型中

例子如下：

```rust
fn test03() {
    let v1 = vec![2, 4, 6];
    let v2: Vec<_> = v1.iter().map(|x| x + 1).collect();
    assert_eq!(v2, vec![3, 5, 7]);
}
```

### 2.3 使用闭包捕获环境(filter)

- filter 方法：
  - 接收一个闭包
  - 这个闭包在遍历迭代器的每个元素时，返回 bool 类型
  - 如果闭包返回 true：当前元素将会包含在 filter 产生的迭代器中
  - 如果闭包返回 false：当前元素将不会包含在 filter 产生的迭代器中

我们现在实现一个功能，取出一个迭代器中所有为偶数的元素，将取出的元素再放入一个新的迭代器，例子如下：

```rust
fn test04() {
    let v1 = vec![1, 2, 3, 4, 5, 6];
    let v2: Vec<_> = v1.into_iter().filter(|x| x % 2 == 0).collect();
    assert_eq!(v2, vec![2, 4, 6]);
}
```

这里有一点要注意：`iter` 方法里的元素都是引用类型，且不可变，因此若要进行 `x % 2 == 0` 操作的话，需要解引用 `*x`。上述代码中使用 `into_iter` 方法获得了迭代器中元素的所有权。

### 2.4 创建自定义迭代器

**使用 Iterator frait 来创建自定义迭代器**

- 实现 next 方法

```rust
struct Counter {
    count: u32,
}

impl Counter {
    fn new() -> Counter {
        Counter {count: 0}
    }
}

impl Iterator for Counter {
    type Item = u32;
    fn next(&mut self) -> Option<Self::Item> {
        if self.count < 5 {
            self.count += 1;
            Some(self.count)
        } else {
            None
        }
    }
}

fn test05() {
    let mut counter = Counter::new();
    assert_eq!(counter.next(), Some(1));
    assert_eq!(counter.next(), Some(2));
    assert_eq!(counter.next(), Some(3));
    assert_eq!(counter.next(), Some(4));
    assert_eq!(counter.next(), Some(5));
    assert_eq!(counter.next(), None);
}
```

这个例子很容易理解，就是通过 `next` 方法不断迭代，范围是 1-5，至于为 Counter 实现 Iterator 特征，将来会介绍。

下面要对迭代器的需求进行改进，有两个迭代器，第一个迭代器就是上面所说的，第二个迭代器的元素是 `[2, 3, 4, 5]`，现在要求将两个迭代器中的元素按顺序相乘，然后将结果存入一个新的迭代器，然后过滤出能被 3 整除的数，并求和。

代码如下：

```rust
fn test06() {
    let sum: u32 = Counter::new()
        .zip(Counter::new().skip(1))
        .map(|(a, b)| a * b)
        .filter(|x| x % 3 == 0)
        .sum();
    assert_eq!(sum, 18);
}
```

介绍一下 `zip` 方法，这个单词本意是“拉链”，这里表示将两个迭代器“捏到一起”，形成一个新的迭代器，里面的每个元素就是一个元组 ，这个元组里有两个元素，这两个元素分别来自原来的两个迭代器。这里第一个迭代器就是通过 `Counter::new()` 得到的，第二个迭代器就是 `zip()` 方法内的参数 `Counter::new().skip(1)`，表示跳过第一个元素后剩下的元素组成的迭代器。

为了更好展示每一个方法的实现过程，我们运行

```rust
let v1 = Counter::new().zip(Counter::new().skip(1));
println!("{:?}", v1);
```

对于上例中的两个初始化后的迭代器，输出结果为：

```rust
➜  it git:(master) ✗ cargo run
   Compiling it v0.1.0 (/Users/cherry/Code/Rust/learning/it)
    Finished dev [unoptimized + debuginfo] target(s) in 0.20s
     Running `target/debug/it`
Zip { a: Counter { count: 0 }, b: Skip { iter: Counter { count: 0 }, n: 1 } }
```

再来看官方文档中的实例就更清楚了：

```rust
let a1 = [1, 2, 3];
let a2 = [4, 5, 6];

let mut iter = a1.iter().zip(a2.iter());

assert_eq!(iter.next(), Some((&1, &4)));
assert_eq!(iter.next(), Some((&2, &5)));
assert_eq!(iter.next(), Some((&3, &6)));
assert_eq!(iter.next(), None);
```

### 2.5 优化第十章的 I/O 项目

项目具体内容请参考 [Rust入门教程（十）](Rust入门教程（十）.md)

#### 2.5.1 利用迭代器优化 new 函数

我们来看一下 `minigrep` 项目中 `Config` 函数的 `new()` 函数：

```rust
impl Config {
    pub fn new(args: &[String]) -> Result<Config, &str> {
        if args.len() < 3 {
            return Err("输入参数错误，请输入两个参数。");
        }
        let search_string = args[1].clone();
        let filename = args[2].clone();
        let case_sensitive = env::var("CASE_INSENSITIVE").is_err();
        Ok(Config {
            search_string,
            filename,
            case_sensitive
        })
    }
}
```

在 `new()` 函数中，传入的参数是字符串切片，为了返回 `Config` 结构体，需要获得命令行参数这两个变量的所有权，之前的解决方法是将那两个参数进行克隆，但是这样会对性能带来一定的损耗。现在我们学习完了闭包和迭代器，便可以通过迭代器获取其实例，并且可以使用迭代器所带的一些方法进行长度检查和索引。通过迭代器的 `next` 方法，便将读取具体值的功能分离了出去。

原来的 `main` 函数中，通过 `env::args().collect()` 将参数列表转化成 vector，然后将这个 vector 传到 `new()` 函数中，其实 `env::args()` 返回的就是迭代器，我们直接把它当做 `new()` 的参数即可。

```rust
fn main() {
    let args: Vec<String> = env::args().collect();
    let config = Config::new(&args).unwrap_or_else(|err| {
        ...
    });
    ...
}
```

更改过后的 `main` 函数：

```rust
fn main() {
    let config = Config::new(env::args()).unwrap_or_else(|err| {
        ...
    });
    ...
}
```

修改过后的 `new()` 函数：

```rust
impl Config {
    pub fn new(mut args: std::env::Args) -> Result<Config, &'static str> {
        if args.len() < 3 {
            return Err("输入参数错误，请输入两个参数。");
        }
        args.next();
        let search_string = match args.next() {
            Some(args) => args,
            None => return Err("无法读取要查询的字符串参数")
        };
        let filename = match args.next() {
            Some(args) => args,
            None => return Err("无法读取文件名参数")
        };
        let case_sensitive = env::var("CASE_INSENSITIVE").is_err();
        Ok(Config {
            search_string,
            filename,
            case_sensitive
        })
    }
}
```

#### 2.5.2 利用迭代器优化 search 函数

原来的 `search()` 函数实现如下：

```rust
fn search<'a>(query: &str, content: &'a str) -> Vec<&'a str> {
    let mut vec = Vec::new();
    for lines in content.lines() {
        if lines.contains(query) {
            vec.push(lines);
        }
    }
    vec
}
```

实现思路是先将文本内容每一行取出来，然后手动遍历，判断当前行是否包含所要查询的字符串，将结果放入新的 vector 中，最后返回这个 vector。但是学完了迭代器适配器的知识点后，应该很容易想到 `filter` 这个方法。实现如下：

```rust
fn search<'a>(query: &str, content: &'a str) -> Vec<&'a str> {
    content.lines().filter(|line| line.contains(query)).collect()
}
```

同理 `search_case_insensitive` 修改如下：

```rust
fn search_case_insensitive<'a>(query: &str, content: &'a str) -> Vec<&'a str> {
    content.lines()
        .filter(|line| line.to_uppercase().contains(&query.to_uppercase()))
        .collect()
}
```

现在我们将原来的七行代码简化成了一行，这一行代码和之前的七行实现的功能是相同的，但是显然利用迭代器实现的这一行代码更加易读（熟悉了迭代器的使用之后，这种写法会非常简单），不仅从代码，减少了临时变量，同时消除了可变状态 `result`，这样可以使得将来通过并行来提升搜索效率，因为并行时不用再考虑并发访问 `result` 这个变量时会出现的安全问题了。

实际上，对于大多数 Rust 程序员会更喜欢使用迭代器这样的方式来实现，因为这样可以更加专注于实现逻辑本身，而不是总是浪费时间在大量的循环和维护临时变量这些细节工作上。至于两者的效率问题，并非像表面上那样，使用迭代器效率会降低，具体的我们下节再介绍。

### 2.6 性能比较：循环 vs 迭代器

**零开销抽象 Zero-Cost Abstraction**

- 使用抽象时不会引入额外的运行时开销

对于迭代器，编译器会自行判断底层代码展开策略，对于某些特定次数的循环，编译器底层会手动将迭代器展开特定的次数，这样对于流水线 CPU 来说，会减少因跳转或延迟槽产生的停顿周期，使得流水线的吞吐量增大，从而使得效率提高。

因此在 Rust 中，尽量使用迭代器实现。
