---
title: C 语言预处理器和宏的高级用法
mathjax: true
tags:
  - C
  - 宏
categories:
  - [编程语言, C/C++]
abbrlink: aaaac292
date: 2023-01-22 22:27:45
---

>本篇文章介绍 C 语言中类似 `#define`, `#if`, `#ifdef` 等预处理指令以及宏的高级用法，最后整理出项目中一些常用的宏，例如打印调试信息等。
>本篇文章将不会介绍简单的宏用法，例如 `#define ADD(a, b) ((a)+(b))`
>本篇文章大部分参考《C Primer Plus 第六版》第 16 章

<!-- more -->

## 一、预处理及宏

### 1.1 “#” 运算符

`#` 是一个预处理运算符，可以将记号转化成字符串。例如 `#define TYPE(x) #x`，若使用宏 `TYPE(int)`，则将其替换成**字符串** `"int"`，`#x` 就是转换为 `x` 的形参名。

下面是一个例子。

```c
#include<stdio.h>
#define TYPE(x) #x

int main() {
    printf("The num 3 is an "TYPE(int)" type");
    return 0;
}
```

输出结果为：`The num 3 is an int type`

### 1.2 "##" 运算符

与 `#` 运算符类似，`##` 运算符可以用于类函数宏的替换部分，而且还可以用于对象宏的替换部分。`##` 运算符将两个记号组合成一个记号，例如 `#define TEST(n) TEST_##n`，然后宏 `TEST1` 将其展开为 `TEST_1`。

下面是一个具体的例子。

```c
#include<stdio.h>
#define XNAME(n) x##n
#define PRINT_XN(n) printf("x"#n" = %d\n", x##n) 
int main() {
    int XNAME(1) = 14;      // 展开成 int x1 = 14;
    int XNAME(2) = 20;      // 展开成 int x2 = 20;
    int x3 = 30;
    PRINT_XN(1);            // 展开成 printf("x1 = %d\n", x1);
    PRINT_XN(2);            // 展开成 printf("x2 = %d\n", x2);
    PRINT_XN(3);
    return 0;
}
```

>注意，`#` 运算符组合成**字符串**，而 `##` 运算符组合成为一个新的**标识符**。

### 1.3 #undef 指令

`#undef` 指令用于取消已定义的 `#define` 指令。若之前没有定义某个宏，取消对其的定义也是有效的，如果想使用一个名称，但不确定之前是否已经用过，使用 `#undef` 先取消定义是一个安全的方法。

### 1.4 条件编译指令

#### 1.4.1 #ifdef、#else 和 #endif 指令

先用一个简单的例子来说明这三个条件编译指令。

```c
#ifdef MAVIS
    #include "horse.h"
    #define STABLES 5
#else
    #include "cow.h"
    #define STABLES 15
#endif
```

上述代码理解起来应该挺简单，若用 `#define` 定义了 `MAVIS`，就引入 `horse.h` 头文件，若没有定义 `MAVIS` 就引入头文件 `cow.h`。

`#ifdef` 测试的宏可以是对象宏，也可以是函数宏。

#### 1.4.2 #ifndef

`#ifndef` 用法和 `#ifdef` 类似，但是意思相反。除此之外 `#ifndef` 还可以防止相同的宏被重复定义，例如下面的例子。

```c
#ifndef _MATH_H_
#define _MATH_H_
```

通过 `#ifndef` 也可以避免头文件被引入多次。

#### 1.4.3 #if、#elif

`#if` 和 `#elif` 后面跟一个常量表达式，如果表达式的值为非零，则表达式为真，类似于 C 语言中的 if else，可以使用关系运算符和逻辑运算符。

`#if` 和 `#elif` 后面的宏只能是对象宏，不能是函数宏。

#### 1.4.4 #defined

`#defined` 用于判断宏是否已经被定义，可以是对象宏，也可以是函数宏，可以和 `#elif` 嵌套使用。

条件编译可以让程序更容易移植，改变文件开头的几个关键定义，可以根据不同的架构或系统设置不同的值和包含不同的文件。

### 1.5 预定义宏

C 标准规定了一些预定义宏，如下列表格所示。

| 宏 | 含义 |
|---|---|
| __DATE__  | 预处理的日期（“Mmm dd yyyy”形式的字面量，如 Nov 12 2023）|
| __FILE__  | 表示当前源代码文件名的字符串字面量 |
| __LINE__  | 表示当前源代码文件中行号的整型量 |
| __STDC__  | 设置为 1 时表示遵循 C 标准 |
| __STDC_HOSTED__  | 本机环境设置为 1，否则设置为 0 |
| __STDC_VERSION__  | 支持 C99 标准，设置为 199901L；支持 C11标准，设置为 201112L |
| __TIME__  | 翻译代码的时间，格式为 “hh:mm:ss” |

### 1.6 #line 和 #error

`#line` 指令重置 `__LINE__` 和 `__FILE__` 宏报告的行号和文件名，用法如下。

```c
#line 1000          // 将当前行号重置为 1000
#line 10 cool.c     // 将当前行号重置为 10，文件名重置为 cool.c
```

`#error` 指令让预处理器发出一条错误信息，该消息包含指令中的文本，用法如下。

```c
#if __STDC_VERSION__ != 201112L
#error Not C11
#endif
```

编译上述代码将会产生 error，并且提示 `Not C11`。

### 1.7 变参宏 ... 和 \_\_VA_ARGS__

一些函数可以接受数量可变的参数，例如 `printf`，在头文件 `stdvar.h` 中提供了相关操作。

同样，宏定义中也可以实现可变参数，通过将宏列表中最后的参数写成 `...` 来实现这一功能。这样，预定义宏 `__VA_ARGS__` 可用在替换部分中，用来表示省略号代表什么。例如定义 `#define PRINT(...) printf(__VA_ARGS__)`，调用宏 `PRINT("Hello")`，`__VA_ARGS__` 展开为一个参数 `Hello`，调用宏 `PRINT("My name is %s", name)`，`__VA_ARGS__` 展开为两个参数 `"My name is %s"` 和 `name`。

### 1.8 __attribute__

GNU C 的一大特色就是 `__attribute__` 机制。`__attribute__` 可以设置函数属性（Function Attribute ）、变量属性（Variable Attribute ）和类型属性（Type Attribute）。

具体内容请参见链接 [C语言__attribute__的使用](https://blog.csdn.net/qlexcel/article/details/92656797)、[__attribute__ 机制详解](https://blog.csdn.net/weaiken/article/details/88085360)

## 二、宏模板

由于 C 语言中库比较少，而一些比较基础的操作又无需通过函数实现，因此可以将一些基础功能写成宏进行展开，并集成到头文件中，在今后的项目中可以很方便的进行调用。

在这里我自己总结并整理了若干个常用的宏。

| 宏名称 | 功能 |
|-----|--------------------|
| LOG          | 打印调试信息（带颜色） |
| UPPERCASE    | 转化为大写字母 |
| LOWERCASE    | 转化为小写字母 |
| FPOS         | 获取结构体成员偏移量 |
| FSIZ         | 获取结构体成员所占用字节数 |
| container_of | 根据成员指针、结构体类型、结构体成员名称获取结构体起始地址 |
| offsetof     | 获取结构体成员偏移量 |
|  |  |
|  |  |
|  |  |
|  |  |
|  |  |
|  |  |


### 2.1 打印调试信息

调试信息是任何项目必不可少的内容，下面的宏可以在终端中输出带颜色的调试标签，方便观察错误和警告信息。

```c
#include<stdio.h>

#define _LOG_
#ifdef _LOG_
#define LOG_ERROR_STYLE "\x1b[31m"
#define LOG_INFO_STYLE "\x1b[32m"
#define LOG_WARNING_STYLE "\x1b[33m"
#define LOG_STYLE_CLEAR "\x1b[0m "
#define LOG_ERROR(...) printf(LOG_ERROR_STYLE"[ERROR]"LOG_STYLE_CLEAR __VA_ARGS__)
#define LOG_INFO(...) printf(LOG_INFO_STYLE"[INFO]"LOG_STYLE_CLEAR __VA_ARGS__)
#define LOG_WARNING(...) printf(LOG_WARNING_STYLE"[WARN]"LOG_STYLE_CLEAR __VA_ARGS__)
#define LOG(TYPE, ...) LOG_##TYPE(__VA_ARGS__)
#endif
```

调用上面的 `LOG` 宏，可以看到结果如下。

```c
int main() {
    LOG(ERROR, "%s", "This is an error msg...\n");
    LOG(INFO, "%s", "This is an info msg...\n");
    LOG(WARNING, "%s", "This is a warning msg...\n");
}
```

![LOG宏实现效果](https://raw.githubusercontent.com/CherryYang05/PicGoImage/master/images/20230122232752.png)

### 2.2 大小写转化

```c
#define UPPERCASE(c) (c & 0xdf)
#define LOWERCASE(c) (c | 0x20)
```

### 2.3 得到一个结构体成员 member 在结构体 struct 中的偏移量

```c
#define FPOS(type, member) (&((type*)0)->member)
```

### 2.4 得到一个结构体中某个成员字段 member 所占用的字节数

```c
#define FSIZ(type, member) sizeof(((type*)0)->member)
```

### 2.5 container_of

```c
#define container_of(ptr, type, member) ({              \
const typeof(((type *)0)->member) *__mptr = (ptr);    \
(type *)((char *)__mptr - __offsetof(type,member)); })
```

`container_of` 宏函数的作用是 **已知结构体 type 的成员 member 的地址 ptr，得到结构体 type 的起始地址**。

第一行用于“类型检查”。它确保 type 有一个名为 member 的成员（不过我认为这也是由 offsetof 宏完成的），并且如果 ptr 不是指向正确类型（成员的类型）的指针，编译器将打印警告，这对调试很有用。

在上述宏的第三行，用了 `char *` 进行指针转化，这是因为 `offsetof` 指针偏移量是按照字节计算的，同时 `char *` 的指针也是以字节计算的，若转化为例如 `int *` 等类型，则 C 的指针算法将会计算 `sizeof(int) * offsetof` 作为最终的结果，也就是 4 字节乘以偏移量。

具体说明参考链接 [container of()函数简介](https://blog.csdn.net/s2603898260/article/details/79371024) 和 [linux 内核宏container_of剖析](https://zhuanlan.zhihu.com/p/54932270)

### 2.6 offsetof

```c
#define offsetof(type, member) ((size_t) & ((type *)0)->member)
```

`offsetof` 宏函数的作用是 **得到结构体 type 的成员 member 所在的内存偏移量**

对于 `container of` 以及 `offsetof` 我会单独用一篇博客进行详细讲解。

### 2.7



### 2.8


### 2.9


### 2.10


### 2.11


### 2.12


### 2.13