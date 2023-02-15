---
title: Rust入门教程（十二）：cargo 和 crates.io
mathjax: true
tags:
  - Rust
categories:
  - 编程语言
  - Rust
abbrlink: af9cf945
date: 2022-07-08 16:50:09
---

>本章内容如下：
>- 通过 release profle 来自定义构建
>- 在 https://crates.io/ 上发布库
>- 通过 workspaces 组织大工程
>- 从 https://crates.io/ 来安装库
>- 使用自定义命令扩展 cargo

<!-- more -->

## 一、通过 release profle 来自定义构建

**release profile（发布配置）**

- 是预定义的
- 可自定义：可使用不同的配置，对代码编译拥有更多的控制
- 每个 profile 的配置都独立于其它的 profile
- Cargo 主要的两个 profile
  - dev profile：适用于开发，cargo build
  - release profile：适用于发布，cargo build --release

**自定义 profile**

- 针对每个 profle，Cargo 都提供了默认的配置
- 如果想自定义 xxxx profile 的配置
  - 可以在 Cargo.toml 里添加 `[profile.xxxx]` 区域，在里面覆盖默认配置的子集

例：

```rust
[profile. dev]
opt-level = 0
[profile.release]
opt-level = 3
```

`opt-level` 表示编译优化等级，在开发模式下，一般希望有较快的编译速度，因此编译优化等级较低。而在发布模式下，需要多次运行程序，因此希望有较高等级的优化，从而用较长的编译时间换取较短的运行时间。

对于每个配置的默认值和完整选项，请参见：https://doc.rustlang.ora/cargo/

## 二、发布 crate 到 crates.io

### 2.1 crates.io

- 可以通过发布包来共享你的代码
- crate 的注册表在 https://crates.io/
- 它会分发己注册的包的源代码
- 主要托管开源的代码

### 2.2 文档注释

#### 2.2.1 文档注释的使用

- 文档注释：用于生成文档
  - 生成 HTML 文档
  - 显式公共 API 的文档注释：如何使用 API
  - 使用 `///`
  - 支持 Markdown
  - 放置在被说明条目之前

例如：

```rust
/// 这个函数是将传入的 u32 类型的值加一，然后返回这个值的所有权
/// # 实例
/// let a = 3;
/// let b = add_one(a);
/// assert_eq!(4, b);
fn add_one(num: u32) -> u32 {
    num + 1
}
```

#### 2.2.2 生成 HTML 文档的命令

`cargo doc`
- 它会运行 rustdoc 工具（Rust 安装包自带）
- 把生成的 HTML 文档放在 target/doc 目录下

`cargo doc --open`
- 构建当前 crate 的文档（也包含 crate 依赖项的文档）
- 在浏览器打开文档

![Rust Doc](https://raw.githubusercontent.com/CherryYang05/PicGo-image/master/images/20220808150051.png)

#### 2.2.3 常用章节

- ＃ Examples
- 其它常用的章节
  - Panics：函数可能发生 panic 的场景
  - Errors：如果函数返回 Result，描述可能的错误种类，以及可导致错误的条件
  - Safety：如果函数处于 unsafe 调用，就应该解释函数 unsafe 的原因，以及调用者确保的使用前提

#### 2.2.4 文档注释作为测试

- 示例代码块的附加值：
  - 运行 cargo test：将把文档注释中的示例代码作为测试来运行 

```rust
➜  it git:(master) ✗ cargo test
   Compiling it v0.1.0 (/Users/cherry/Code/Rust/learning/it)
    Finished test [unoptimized + debuginfo] target(s) in 0.18s
     Running unittests src/lib.rs (target/debug/deps/it-d31f5191c1eed090)

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running unittests src/main.rs (target/debug/deps/it-8c2dc60df354a71a)

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests it

running 1 test
test src/lib.rs - add_one (line 3) ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.16s
```

#### 2.2.5 为包含注释的项添加文档注释

- 符号：`//!`
- 这类注释通常用描述 crate 和模块：
  - crate root（按惯例 src/lib.rs)
  - 一个模块内，将 crate 或模块作为一个整体进行记录

![Rust Doc](https://raw.githubusercontent.com/CherryYang05/PicGo-image/master/images/20220808151624.png)

### 2.3 pub use

**使用 pub use 导出方便使用的公共 API**

- 问题：crate 的程序结构在开发时对于开发者很合理，但对于它的使用者不够方便
  - 开发者会把程序结构分为很多层，使用者想找到这种深层结构中的某个类型很费劲
- 例如
  - 麻烦：my_crate::some_module::another_module::UsefulType；
  - 方便：my_crate::UsefulType
- 解决办法
  - 不需要重新组织内部代码结构
  - 使用 pub use：可以重新导出，创建一个与内部私有结构不同的对外公共结构

`main.rs:`

```rust
use pubuse::PrimaryColor;
use pubuse::mix;

fn main() {
    let c1 = PrimaryColor::Blue;
    let c2 = PrimaryColor::Yellow;
    let c3 = mix(c1, c2);
    println!("The mixed color is {:?}", c3);
}
```

`lib.rs:`

```rust
//! 这是测试 pub use 使用的 crate

pub use kinds::PrimaryColor;
pub use kinds::SecondaryColor;
pub use utils::mix;

pub mod kinds {
    /// 色彩三原色：红黄蓝
    #[derive(Debug)]
    pub enum PrimaryColor {
        Red,
        Yellow,
        Blue,
    }
    
    /// 两种颜色混合后的颜色
    #[derive(Debug)]
    pub enum SecondaryColor {
        Orange,
        Green,
        Purple,
    }
}

pub mod utils {
    use crate::kinds::*;

    /// 将红黄蓝中任意两种颜色混合
    pub fn mix(c1: PrimaryColor, c2: PrimaryColor) -> SecondaryColor {
        SecondaryColor::Green
    }
}
```

![pub use Doc](https://raw.githubusercontent.com/CherryYang05/PicGo-image/master/images/20220808163422.png)

我们从生成的文档看出，就将很深层的函数提到了 crate 的外层，方便用户查看和调用。

### 2.4 发布 crate

- 创建并设置 crates.io 账号
- 发布 crate 前，需要在 crates.io 创建账号并获得 API token
- 运行命令：`cargo login` [你的API token]
  - 通知 cargo，你的 API token 存储在本地 `~/.cargo/credentials`
- API token 可以在 https://crates.io/ 进行撤销

**为新的 crate 添加元数据**
- 在发布 crate 之前，需要在 Cargo.toml 的 Ipackage] 区域为 crate 添加一些元数据：
  - crate 需要唯一的名称：name
  - description：一两句话即可，会出现在 crate 搜索的结果里
  - license：需提供许可证标识值（可到 http://spdk.org/licenses/ 查找）
  - 可指定多个 license：用 OR
  - version
  - author
- 发布：`cargo publish` 命令

**发布到 crates.io**

- crate一旦发布，就是永久性的：该版本无法覆盖，代码无法删除
- 目的：依赖于该版本的项目可继续正常工作

**发布己存在 crate 的新版本**

- 修改 crate 后，需要先修改 Cargo.toml 里面的 version 值，再进行重新发布
- 参照 htto://semver.orgl 来使用你的语义版本

**使用 cargo yank 从 Crates.io 撤回版本**

- 不可以删除 crate 之前的版本
- 但可以防止其它项目把它作为新的依赖：yank（撤回）一个 crate 版本
  - 防止新项目依赖于该版本
  - 己经存在项目可继续将其作为依赖（并可下载）
- yank 意味着：
  - 所有己经产生 Cargo.lock 的项目都不会中断
  - 任何将来生成的 Cargo.lock 文件都不会使用被 yank 的版本。
- 命令：
  - yank 一个版本（不会删除任何代码）：`cargo yank -vers 1.0.1`


### 2.5 cargo 工作空间

#### 2.5.1 创建工作空间

随着项目越来越大，代码也会越来越臃肿，这时就需要将 crate 拆分成多个包，cargo 便提供了**工作空间**这个功能。

- cargo 工作空间：帮助管理多个相互关联且需要协同开发的 crate
- cargo 工作空间是一套共享同一个 Cargo.lock 和输出文件夹的包

**创建工作空间**

- 有多种方式来组建工作空间例：1 个二进制 crate，2 个库 crate
  - 二进制 crate：main 函数，依赖于其它2个库 crate
  - 其中1个库 crate 提供 addLone 函数
  - 另外1个库 crate 提供 add_two 函数

**在工作空间中依赖外部 crate**

- 工作空间只有一个 Cargo.lock 文件，在工作空间的顶层目录
  - 保证工作空间内所有 crate 使用的依赖的版本都相同
  - 工作空间内所有 crate 相互兼容

参数 `-p` 可以指定某个 crate 构建、运行或测试。例如 `cargo test -p add-one`。
发布的时候需要手动进入每个 crate 逐个发布。

以上内容参考[视频](https://www.bilibili.com/video/BV1hp4y1k7SV?p=83&spm_id_from=pageDriver&vd_source=cd979f5b9cea5f03bd91461762cdd74c)，未做具体记录。

### 2.6 安装二进制 crate

**从 CRATES.IO 安装二进制 crate**

- 命令：cargo install
- 来源： https://crates.io
- 限制：只能安装具有二进制目标 (binary target）的 crate
- 二进制目标 binary target：是一个可运行程序
  - 由拥有 src/main.rs 或其它被指定为二进制文件的 crate 生成
- 通常：README 里有关于 crate 的描述：
  - 拥有 library target
  - 拥有 binary target
  - 两者兼备

**cargo install**

- cargo install 安装的二进制存放在根目录的 bin 文件夹
- 如果你用 rustup 安装的 Rust，没有任何自定义配置，那么二进制存放目录是 `$HOME/.cargo/bin`
  - 要确保该目录在环境变量 ＄PATH 中

**使用自定义命令扩展 cargo**

- cargo 被设计成可以使用子命令来扩展
- 例：如果＄PATH 中的某个二进制是 cargo-something，你可以像子命令一样运行：
  - cargo something
- 类似这样的自定义命令可以通过该命令列出：cargo --list
- 优点：可使用 cargo install 来安装扩展，像内置工具一样来运行
