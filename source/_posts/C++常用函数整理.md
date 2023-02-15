---
title: C/C++常用函数整理
mathjax: true
tags:
  - STL函数
  - C/C++
categories:
    - [编程语言, C/C++]
abbrlink: 37079a34
date: 2021-03-08 19:29:56
---

## C/C++ 中常用函数

<!-- toc --> <!-- more -->
### 1. time

```cpp
clock_t clock(void);
```

简单而言，就是该程序从启动到函数调用占用 CPU 的时间。这个函数返回从“开启这个程序进程”到“程序中调用 clock()函数”时之间的 CPU 时钟计时单元(clock tick)数，在 MSDN 中称之为挂钟时间(wal-clock)；若挂钟时间不可取，则返回-1。其中 clock_t 是用来保存时间的数据类型。
在 time.h 文件中，我们可以找到对 clock_t()的定义：

```cpp
#ifndef _CLOCK_T_DEFINED
typedef long clock_t;
#define _CLOCK_T_DEFINED
#endif
```

例：

```cpp
int time_begin = clock();
func();
int time_end = clock();
cout << (time_end - time_begin) / CLOCKS_PER_SEC << endl;
```

### 2. fill

```cpp
fill(v.begin(), v.end(), val); //将容器中每个元素被重置为val
```

fill 函数作用是将一个区间的元素赋值为 val，该 val 可以非零，包含在 `algorithm` 头文件里。

**和 memset 函数区别：**
我们定义数组时通常喜欢用 `memset` 函数赋初值 `0`，但是实际上 `memset` 也只能赋值为 `0`。函数原型如下：

`void *memset(void *s, int ch, size_t n);`

函数功能是：将 `s` 所指向的某一块内存中的前 n 个字节的内容全部设置为 `ch` 指定的 ASCII 值， 第一个值为指定的内存地址，块的大小由第三个参数指定，这个函数通常为新申请的内存做初始化工作，其返回值为指向 s 的指针，它是对较大的结构体或数组进行清零操作的一种最快方法。

但是要注意的是， `memset` 是按照一个一个字节进行初始化的，如果初始化的值为 0，则结果是对的，但是如果将一个 `int` 型变量初始化为 1，则不能使用 memset，因为初始化后该变量会变成 `00000001 00000001 00000001 00000001`，这样的值便是错误的。

这是和 `fill` 最根本的区别。

[参考博客](https://blog.csdn.net/Xiao2018428/article/details/101268398)

### 3. next_permutation

```cpp
bool next_permutation(iterator start,iterator end)
```

该函数是获得容器内序列的下一个排列组合，包含在头文件 `algorithm` 中。


示例：

```cpp
string str;
sort(str.begin(), str.end());
do {
	cout << str << endl;
} while(next_permutation(str.begin(), str.end()));
```

此外，next_permutation(v, v + n, cmp)可以对结构体 num 按照自定义的排序方式 cmp 进行排序

若要求的字典序是 `'A'<'a'<'B'<'b'<...<'Z'<'z'`，则可以自定义 cmp 函数：

```cpp

#include <iostream>                 //poj 1256 Anagram
#include <string>
#include <algorithm>
using namespace std;
int cmp(char a, char b) {
   if(tolower(a) != tolower(b))     //tolower 是将大写字母转化为小写字母.
       return tolower(a) < tolower(b);
   else
       return a < b;
}
int main() {
    char ch[20];
    int n;
    cin >> n;
    while(n--) {
        scanf("%s", ch);
        sort(ch, ch + strlen(ch), cmp);
        do {
            printf("%s\n", ch);
        } while(next_permutation(ch, ch + strlen(ch), cmp));
    }
    return 0;
}

```

[参考博客](https://blog.csdn.net/qq_43488547/article/details/100032724)

### 4. stoi

```cpp
stoi(string, pos, n进制)
```

stoi(string to int)，表示将 n 进制的字符串转化为十进制

和 atoi 的比较：

相同点：

① 都是 C++的字符处理函数，把数字字符串转换成 int 输出
② 头文件都是 `#include<cstring>`

不同点：

① atoi()的参数是 const char*，因此对于一个字符串 str 我们必须调用 c_str() 的方法把这个 string 转换成 const char* 类型的,而 stoi()的参数是 const string*，不需要转化为 const char*；

② stoi()会做范围检查，默认范围是在 int 的范围内的，如果超出范围的话则会 runtime error.
而 atoi()不会做范围检查，如果超出范围的话，超出上界，则输出上界，超出下界，则输出下界。

### 5. GetTickCount

GetTickCount 是一种函数。GetTickCount 返回(retrieve)从操作系统启动所经过(elapsed)的毫秒数，它的返回值是 DWORD。
函数原型：

```cpp
DWORD GetTickCount(void);
```

头文件：
C/C++头文件：winbase.h
windows 程序设计中可以使用头文件 windows.h

例：

```cpp
#include<iostream>
#include<Windows.h>
using namespace std;
int main() {
    DWORD startTime = GetTickCount();//计时开始
    for (int i = 0; i < 2147483640; i++) {
        i++;
    }
    DWORD endTime = GetTickCount();//计时结束
    cout << "The run time is:" << endTime - startTime << "ms" << endl;
    system("pause");
    return 0;
}
```

GetTickcount 函数：它返回从操作系统启动到当前所经过的毫秒数，常常用来判断某个方法执行的时间，其函数原型是 DWORD GetTickCount(void)，返回值以 32 位的双字类型 DWORD 存储，因此可以存储的最大值是(2^32-1) ms 约为 49.71 天，因此若系统运行时间超过 49.71 天时，这个数就会归 0，MSDN 中也明确的提到了:"Retrieves the number of milliseconds that have elapsed since the system was started, up to 49.7 days."。因此，如果是编写服务器端程序，此处一定要万分注意，避免引起意外的状况。

特别注意：这个函数并非实时发送，而是由系统每 18ms 发送一次，因此其最小精度为 18ms。当需要有小于 18ms 的精度计算时，应使用 StopWatch 方法进行。
用 clock()函数计算运行时间，表示范围一定大于 GetTickCount()函数，所以，建议使用 clock()函数。

[参考网站](https://www.cnblogs.com/didiaodidiao/p/9194702.html)

### 6. C 中的可变参数

包含在头文件 `stdarg.h` 中

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
    cout << func(4, 1, 2, 3, 4) << endl;
    return 0;
}
```

步骤如下：

- 定义一个函数，最后一个参数为省略号，省略号前面可以设置自定义参数。
- 在函数定义中创建一个 va_list 类型变量，该类型是在 stdarg.h 头文件中定义的。
- 使用 int 参数和 va_start 宏来初始化 va_list 变量为一个参数列表。宏 va_start 是在 stdarg.h 头文件中定义的。
- 使用 va_arg 宏和 va_list 变量来访问参数列表中的每个项。
- 使用宏 va_end 来清理赋予 va_list 变量的内存。

[详细介绍](https://www.cnblogs.com/pengdonglin137/p/3345911.html)

### 7. sprintf

```cpp
int sprintf(char *buffer, const char *format, ...)
```

**1. 把整数 123 打印成一个字符串保存在 s 中**
```cpp
sprintf(s, "%d", 123); //产生"123"
```

**2. 格式对齐**
```cpp
sprintf(s, "%08X", 4567);//产生：000011D7
sprintf(s, "%-8x", 4567);//产生：11D7，小写16进制，左对齐
sprintf(s, "%8X", 4567);//产生：11D7，大写十六进制，右对齐
```

**3. 取对应个数的字符**
```cpp
char a1[] = {'A', 'B', 'C', 'D', 'E', 'F', 'G'};
char a2[] = {'H', 'I', 'J', 'K', 'L', 'M', 'N'};
sprintf(s, "%.7s%.7s", a1, a2);
//产生："ABCDEFGHIJKLMN"
```

这可以类比打印浮点数的 `"%m.nf"`，在 `"%m.ns"` 中，`m` 表示占用宽度（字符串长度不足时补空格，超出了则按照实际宽度打印），`n` 才表示从相应的字符串中最多取用的字符数。通常在打印字符串时 `m` 没什么大用，还是点号后面的 `n` 用的多。自然，也可以前后都只取部分字符：

```cpp
sprintf(s, "%.6s%.5s", a1, a2);//产生："ABCDEFHIJKL"
sprintf(s, "%.*s%.*s", 7, a1, 7, a2);
//或者
sprintf(s, "%.*s%.*s", sizeof(a1), a1, sizeof(a2), a2);
```

**4. 实际上，前面介绍的打印字符、整数、浮点数等都可以动态指定那些常量值**
比如：
```cpp
sprintf(s, "%-*d", 4, 'A'); //产生"65 "
sprintf(s, "%#0*X", 8, 128); //产生"0X000080"，"#"产生0X
sprintf(s, "%*.*f", 10, 2, 3.1415926); //产生" 3.14"
//这里 (%*.*f) 相当于 (%10.2f), 表示该浮点数保留 2 位小数，并且向右靠齐占位 10 个字符
```

**5. sprintf(s, "%p", &i);**

**6. strlen 便已经知道了结果字符串的长度**
```cpp
int len = sprintf(s, "%d", i);
```
对于正整数来说，`len` 便等于整数 `i` 的 `10` 进制位数。

`sprintf` 函数属于 C 库函数，功能是将标准输出到字符串 `buffer` 中(C 语言的中的字符数组)，若写入成功则返回写入的字符数量，否则返回负值，除了第一个参数外，其他参数同 `printf`

### 8. sscanf

```cpp
int sscanf(const char *s, const char *format, ...);
```

`sscanf` 与 `scanf` 等价，所不同的是，前者的输入字符来源于字符串 `s`，而 `scanf` 以 `stdin` 作为输入源

**1. %*d, %*s: * 号表示此数据不读入，忽略掉**

比如:

```cpp
ch = "MemTotal:2028248 kB"
sscanf(ch, "%*s%d", &total);
printf("%d\n", total);
输出结果：2028248
```
**[注意] sprintf 中的 `*` 号表示占位符，类似于 JDBC 中的 ?，而 sscanf 中的 `*` 号表示跳过对应格式的数据不读入**

**2. 取指定长度的字符串**
```cpp
sscanf("123456 ", "%4s", buf);
printf("%s\n", buf);
结果为：1234
```

**3. 取到指定字符为止的字符串**
在下例中，取遇到空格为止的字符串
```cpp
sscanf("123456 abcdedf", "%[^ ]", buf);
printf("%s\n", buf);
结果为：123456
```

**4. sscanf(ch, "%*[^e]%[^2]", ch1)**
寻找e到2之间的数，包括e但不包括2
如果中间有空格，包括空格。

**5. 给定一个字符串"hello, world"，仅保留world（注意："，"之后有一空格）**
```cpp
sscanf(“hello, world”, "%*s%s", buf);
printf("%s\n", buf);
结果为：world
```
`%*s` 表示第一个匹配到的 `%s` 被过滤掉，即 `hello` 被过滤了
如果没有空格则结果为`NULL`


**6. 分割字符串**
```cpp
sscanf("2006:03:18", "%d:%d:%d", a, b, c);
sscanf("2006:03:18 - 2006:04:18", "%s - %s", sztime1, sztime2);
```
如果 `2006:03:18 - 2006:04:18` 间没有空格
```cpp
sscanf("2006:03:18 - 2006:04:18", "%[0-9,:] - %[0-9,:]", sztime1, sztime2);
```

### 9. c_str

```cpp
const char *c_str();
```

c_str() 函数返回一个指向正规 C 字符串的指针常量，内容与本 string 串相同。
这是为了与 c 语言兼容，在 c 语言中没有 string 类型，故必须通过 string 类对象的成员函数 c_str()把 string 对象转换成 c 中的字符串样式。

1. c_str 是一个内容为字符串指向字符数组的临时指针；
2. c_str 返回的是一个可读不可改的常指针；

注意：一定要使用 strcpy() 函数等来操作方法 c_str() 返回的指针 。

比如：最好不要这样：

```cpp
char* c;
string s = "1234";
c = s.c_str();
```

c 最后指向的内容是垃圾，因为 s 对象被析构，其内容被处理，同时，编译器也将报错——将一个 const char 赋与一个 char 。

应该这样用：

```cpp
char c[20];
string s = "1234";
strcpy(c, s.c_str());
```

这样才不会出错，c_str()返回的是一个临时指针，不能对其进行操作。

### 10. 用 scanf 读入 string

```cpp
string a;
a.resize(100);          //需要预先分配空间
scanf("%s", &a[0]);
puts(a.c_str());
return 0;
```

### 11. 5 种读入整行字符串的操作

**1. scanf()读入 char[]**

```cpp
char str[1024];
scanf("%[^\n]", &str);
getchar();
```

说明：在 scanf 函数中，可以使用 `%c`来读取一个字符，使用 `%s` 读取一个字符串，但是读取字符串时不忽略空格，读字符串时忽略开始的空格，并且读到空格为止，因此只能读取一个单词，而不是整行字符串。

其实 scanf 函数也可完成这样的功能，而且还更强大。这里主要介绍一个参数:`%[]`，这个参数的意义是读入一个字符集合。`[]` 是个集合的标志，因此 `%[]` 特指读入此集合所限定的那些字符，比如 `%[A-Z]` 是输入大写字母，一旦遇到不在此集合的字符便停止。如果集合的第一个字符是 `"^"`，这说明读取不在 `"^"` 后面集合的字符，即遇到 `"^"` 后面集合的字符便停止。注意此时读入的字符串是可以含有空格的，而且会把开头的空格也读进来。

注意：如果要循环的多次从屏幕上读取一行的话，就要在读取一行后，在用 `%c` 读取一个字符，将输入缓冲区中的换行符给读出来。否则的话，在下一次读取一行的时候，第一个就遇到`'\n'`，匹配不成功就直接返回了。这里可以用 `scanf()` 或者 `getchar()` 函数读取换行符。

**2. getchar() 读入 char[]**

```cpp
char str[1024];
int i = 0;
while((str[i] = getchar()) != '\n')
    i++;
getchar();
```

说明：这样一个一个读也可以，也会把开头的空格读进来。最后也需要考虑换行符，使用 `getchar()` 读出来。

**3. gets()读入 char[]**

```cpp
char str[1024];
gets(str);
```

说明：感觉这个就是多个 `getchar` 的集合函数，很好用。功能是从标准输入键盘上读入一个完整的行（从标准输入读，一直读到遇到换行符），把读到的内容存入括号中指定的字符数组里，并用空字符 `'\0'` 取代行尾的换行符 `'\n'`。读入时不需要考虑换行符。

**4. getline() 读入 string 或 char[]**

```cpp
string str;
getline(cin, str);          //读入string

char str2[1024];
cin.getline(str2, 1024);    //读入char数组
```

说明：这是比较常用的方法，`cin.getline` 第三个参数表示间隔符，默认为换行符`'\n'`。读入不需要考虑最后的换行符。

**5. get() 读入 char[]**

```cpp
char str3[1024];
cin.get(str3, 1024);         //读入char数组
```

说明：get 函数读入时需要考虑最后的换行符，也就是说，如果用 `get` 读入多行数据，要把 `'\n'` 另外读出来，一般使用 `cin.get(str, 1024).get();` 来读入多组数据。

### 12. reverse

reverse 这个函数功能非常强大，作用是将序列[first,last)的元素在原容器中翻转，包含在`algorithm`库中

- reverse()函数无返回值，时间复杂度`O(n)`
- 注意区间是 `[begin, end)`，左闭右开
- 可以作用于 `vector`，`string`，普通数组

```cpp
vector<int> v = {1, 2, 3};
reverse(begin(v), end(v));
reverse(v.begin(), v.end());

int a[] = {4, 5, 6, 7};
reverse(begin(a), end(a));

string str = "My name is Cherry.";
reverse(str.begin(), str.end());
```

### 13. sort(lamda表达式)

这个函数应该很熟悉了，这里我们主要介绍他的[lamda 表达式的用法](https://www.cnblogs.com/ye-ming/p/9310997.html)。

```cpp
struct stu {
    string name;
    int id;
    stu() {}
    stu(string _name, int _id) : name(_name), id(_id) {}
}
stu s[10];
sort(s, s + 10, [](stu s1, stu s2) {
    if (s1.name == s2.name) return s1.id < s2.id;
    else return s1.name < s2.name;
});

//也可以写成下面这样，因为根据return可清楚的知道返回类型为bool，便不用显式声明了
sort(s, s + 10, [](stu s1, stu s2) -> bool {
    if (s1.name == s2.name) return s1.id < s2.id;
    else return s1.name < s2.name;
});
```

这里 sort 函数的第三个参数，`[]`叫做捕获说明符，表示一个 lamda 表达式的开始，接下来是参数列表，即这个 lamda 函数的参数。

`[]` 表示该 lambda 表达式是不能访问任何外部变量的，即表达式的函数体内无法访问当前作用域下的变量。
`[=]` 表示按值访问， `[&]` 表示按引用访问变量之间用逗号分隔，比如 `[=factor, &total]` 表示按值访问变量 `factor`，而按引用访问 `total`。

示例：

```cpp
int test = 100;
auto fl = [test](int x, int y) {    //这里要传入test才能在lamda表达式中访问
    return test + x + y;
};
```