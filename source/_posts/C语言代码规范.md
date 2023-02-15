---
title: C语言代码规范
mathjax: true
tags:
  - 代码规范
categories:
  - [编程语言, C/C++]
abbrlink: d5e60d4a
date: 2022-05-30 12:57:42
---

## C 语言代码规范

<!-- more -->

### 一、命名规范

**1. 变量命名**

小驼峰式：`firstName`，`hexToBinary`
下划线式：`first_name`，`hex_to_binary`
不要使用非常见单词的缩写，类似 FTL 这种可以缩写，但是 hexToBinary 不要写成 htb

**2. 常量命名**

全部大写，单词之间用下划线分隔，`OUT_OF_BOUND_EXCEPTION_CODE`，也不要使用缩写

**3. 函数及结构体命名**

下划线式？小驼峰式？

小驼峰式：`firstName`，`hexToBinary`
下划线式：`first_name`，`hex_to_binary`

### 二、注释

#### 1. 文件开头注释

类似这样的注释

```c
/* 
 * Date: *** 
 * Author: *** 
 * Version: ***
 * Description: ***
 */
```

#### 2. 变量声明注释

和变量声明同一行，用 `//` 注释，例如：

```c
int cacheCapacity = 1024;   // cache 容量
bool isEndState = false;    // 是否到了终止状态
```

注释尽量用 tab 全对齐

#### 3. 函数功能注释

函数体前对函数进行注释，注明函数功能，参数类型及含义，返回值类型及含义，例如：

```c
/**
 * 函数功能描述
 *
 * @param parm1 该参数的含义
 * @return 若有返回值，则注明返回值含义或其他说明
 */
int func(int parm1) {
    // ...
    return OK;
}
```

#### 4. 代码块注释

在函数内对某一个代码块进行注释，使用 `/*` 注释，例如：

```c
void func() {
    
    /*
     * 代码块功能说明
     */
    if (statement) {
        // ...
    }

    foo();
    ...
}
```

当循环或条件判断嵌套过多时，在代码块的结束部分加上注释标记，例如：

```c
if (...) {
    while (index < MAX_INDEX) {
        // TODO
    } // 结束 while (index < MAX_INDEX)
} // 结束if (...)
```

### 三、排版

#### 1. 空格和换行

if, while 等和后面的括号之间要有一个空格，当有多个参数时，逗号后面也要有一个空格。
所有的二元运算符，除了"."，应该使用空格将之与操作数分开，一元操作符和操作数之间不要加空格。
注释和注释标记之间需要一个空格，空行中不要有空格。

```c
struct ListNode {
    int num;
} LinkList;             // 注意这里 } 后面有一个空格 

if (num == 1) {         // 注意 if 与 '(' 之间有一个空格，元素 '1' 和 num 与操作符 '==' 之间有一个空格
    // TODO
}

void func(int a, int b, int c) {
    // TODO
}

// 注释与双斜杠中间需要一个空格

/* 这种类型的注释与 '/*' 中间也需要一个空格 */
```

另外，函数体内逻辑相关的功能的某些代码中间可以加上空行进行区分，例如：

```c
int func() {
    char hex[10];
    char bin[40];

    hex = getHex();
    hexToBinary(hex, bin);

    return OK;
}
```

一行代码太长时需要换行，尽量在运算符前换行，并且缩进是八个空格（两个 tab），比如：

```C
if ((condition1 && condition2) 
        || (condition3 && condition4) 
        || !(condition5 && condition6)) { 
    doSomethingAboutIt(); 
}
```

#### 2. 花括号形式

花括号在函数名在同一行

```c
void func() {
    // TODO
}
```

不在同一行

```c
void func()
{
    // TODO
}
```

### 四、其他

1. 尽量不使用全局变量，尽量将变量作用域限定到最小，例如局部变量或者静态全局变量，重要全局变量尽量使用 getter 及 setter 调用。局部变量最好要初始化。
2. 将指针释放后要将指针置空，否则可能会导致野指针，例如

```c
int *p = (int*)malloc(sizeof(int) * 5);

if (p != NULL) {
    free(p);
    p = NULL;
}
```
3. 