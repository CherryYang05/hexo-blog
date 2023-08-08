---
title: 用 gdb 调试程序
mathjax: true
tags:
  - GDB
categories:
  - [生产力工具, GDB]
abbrlink: 1ea0c606
date: 2023-07-08 22:29:49
---

>学计算机的人不能不会用命令行 GDB 进行调试，就像西方不能没有耶路撒冷～～

<!-- more -->

## 1. gdb 调试 run 和 start 的区别

一般来说，在启动 gdb 之后，执行 `r` 或者 `run` 之后，就开始执行程序了，直到遇到第一个断点。

`start` 指令会执行程序至 `main()` 主函数的起始位置，即在主函数的第一行停止执行（改行代码还没有执行）。

另外，程序执行过程中使用 `run` 或 `start` 指令，表示重新启动程序。

## 2. gdb tui：在 gdb 中显示程序源码

我们使用 gdb 的时候，想要看源码，需要输入 `list` 命令查看断点前后的代码，但是 `list` 没有代码高亮，也无法实时跟踪。用 gdb tui 自带的界面可以方便的查看并跟踪源码。输入命令 `gdb -tui` 打开窗口。

gdb 还有其他窗口类型，输入如下命令可以打开相应的窗口。

```shell
layout cmd  // 命令窗口，可以输入调试命令
layout src  // 源代码窗口，显示当前行、断点等信息
layout asm  // 汇编代码窗口
layout reg  // 寄存器窗口
```

要想使用这些窗口，需要通过源码编译 gdb，在编译时添加参数 `--enable-tui`。

```shell
./configure --prefix=/usr/local/gdb-11.2
make -j32
make install
```

在编译安装时可能会出现如下错误：

```shell
configure: error: no enhanced curses library found; disable TUI
```

在 CentOS 下需要安装 `ncurses-devel` 包，Ubuntu 下安装 `libncurses5-dev`




























# Log 的使用

```bash
➜  ~ logger "Hello World"
➜  ~ log show --last 1m | grep Hello
2022-10-04 22:42:30.438584+0800 0x81a171   Default     0x0                  47131  0    logger: Hello World
```

