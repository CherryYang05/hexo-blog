---
title: 'C语言中的可变参数详解(va_list, va_start...)'
mathjax: true
tags:
  - C
  - 可变参数
categories:
  - 编程语言
  - C/C++
abbrlink: 9c76e862
date: 2021-03-16 21:27:13
---

# 详解C语言中的可变参数(头文件stdarg.h)

## 1. 获取函数的变长参数(va_list, va_start, va_arg, va_end)

[详细介绍](https://www.cnblogs.com/pengdonglin137/p/3345911.html)

例：求若干个数的和

```cpp
int func(int num, ...) {        //省略号前面有无逗号都可以
    int sum;
    //在stdarg.h 头文件中
    va_list list;
    //为 num 个参数初始化 valist
    va_start(list, num);
    for (int i = 0; i < num; ++i) {
        sum += va_arg(list, int);
    }
    return sum;
}

int main() {
    cout << func(5, 1, 2, 3, 4, 5) << endl;
    return 0;
}
```
<!-- more -->
步骤如下：

- 定义一个函数，最后一个参数为省略号，省略号前面可以设置自定义参数。
- 在函数定义中创建一个 va_list 类型变量，该类型是在 stdarg.h 头文件中定义的。
- 使用 int 参数和 va_start 宏来初始化 va_list 变量为一个参数列表。宏 va_start 是在 stdarg.h 头文件中定义的。
- 使用 va_arg 宏和 va_list 变量来访问参数列表中的每个项。
- 使用宏 va_end 来清理赋予 va_list 变量的内存。

va_start 宏，获取可变参数列表的第一个参数的地址（list 是类型为 va_list 的指针，param1 是最后一个显式声明的参数）：

```cpp
#define va_start(list, param1) (list = (va_list)&param1 + sizeof(param1))
```

va_arg 宏，返回变长参数的值，第二个参数是该变长参数的类型，返回指定类型并将指针指向下一参数（mode 参数描述了当前参数的类型）：

```cpp
#define va_arg(list, mode) ((mode *) (list += sizeof(mode)))[-1]
```

va_end宏，清空va_list可变参数列表：

```cpp
#define va_end(list) (list = (va_list)0)
```

注：以上 sizeof() 只是为了说明工作原理，实际实现中，增加的字节数需保证为为 int 的整数倍

**注意事项：**

a）他们都是宏，因此不能做运算和求地址等操作；

b）变长参数的类型和数目不能通过宏来获取，只能通过自己写程序控制；

c）编译器对变长参数函数的原型检查不够严格，会影响代码质量。

C中的 `printf` 函数实际上就使用了变长参数，实现原理如下所示(参考前文博客链接中代码)：

```cpp
#include <iostream>
#include <stdarg.h>
using namespace std;

void myprintf(const char *format...){
    va_list argptr;
    va_start(argptr, format);        //va_start
      
    char ch;
    while (ch = *(format++)) {       //逐个遍历format字符串
        if (ch == '%') {
            ch = *(format++);
            if (ch == 's')
            {
                char *name = va_arg(argptr, char *);    //va_arg
                cout<<name;
            }
            else if (ch == 'd')
            {
                int age = va_arg(argptr, int);    //va_arg
             cout<<age;
            }
        }
        else {
            cout<<ch;
        }
    }
    cout << endl;
    va_end(argptr);        //va_end
}

int main() {
    myprintf("My name is %s, age %d.", "AnnieKim", 24);
    return 0;
}
```

## 2. va_list 的用法

C语言的 printf, scanf 函数不同于我们写的那种只能接受固定参数个数的函数，他们可以接受任意多个参数。C 语言允许定义这样的接受变参的函数, 它的机制就是 va_list , 使用它 , 我们也可以定义自己的变参个数的函数.

首先, 看下 printf 函数的声明:

```cpp
int printf(char * format, ... );
```

1. 变参处的定义或声明, 用 `...` 代替参数类型.
2. 变参 `...` 只能放在参数列表最末尾.

这里我们写一个小程序, 来演示 va_list 的用法, 定义一个 barycentre 函数, 计算 n 个点的重心并返回, 声明如下:

```cpp
#include <iostream>
#include <algorithm>
#include <stdarg.h>

using namespace std;

struct Point {
    Point() {}
    Point(double _x, double _y) : x(_x), y(_y) {}
    double x, y;
};

struct Point barycentre(int n...) {
    struct Point p;
    struct Point sum = {0, 0};
    va_list list;
    va_start(list, n);
    for (int i = 0; i < n; ++i) {
        p = va_arg(list, Point);
        sum.x += p.x;
        sum.y += p.y;
    }
    sum.x /= n;
    sum.y /= n;
    va_end(list);
    return sum;

}

int main() {
    Point P1 = Point(1.25, 3.0);
    Point P2 = Point(3.0, 5.2);
    cout << barycentre(2, P1, P2).x;
    return 0;
}
```

从 va 的实现可以看出，指针的合理运用，把C语言简洁、灵活的特性表现得淋漓尽致，叫人不得不佩服 C 的强大和高效。不可否认的是，给编程人员太多自由空间必然使程序的安全性降低。va 中，为了得到所有传递给函数的参数，需要用va_arg依次遍历。其中存在两个隐患：

1）如何确定参数的类型。
 
va_arg 在类型检查方面与其说非常灵活，不如说是很不负责，因为是强制类型转换，va_arg 都把当前指针所指向的内容强制转换到指定类型；
 
2） 结束标志。如果没有结束标志的判断，va将按默认类型依次返回内存中的内容，直到访问到非法内存而出错退出。例2中 SqSum() 求的是自然数的平方和，所以我把负数和0作为它的结束标志。例如 scanf 把接收到的回车符作为结束标志，`大家熟知的 printf() 对字符串的处理用 '\0' 作为结束标志`，无法想象C中的字符串如果没有`'\0'`，代码将会是怎样一番情景，估计那时最流行的可能是字符数组，或者是 malloc/free。

允许对内存的随意访问，会留给不怀好意者留下攻击的可能。当处理 cracker 精心设计好的一串字符串后，程序将跳转到一些恶意代码区域执行，以使 cracker 达到其攻击目的。(常见的 exploit 攻击)所以，必需禁止对内存的随意访问和严格控制内存访问边界。

