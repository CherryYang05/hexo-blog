---
title: C语言进阶学习
mathjax: true
tags:
  - C
categories:
    - [编程语言, C/C++]
abbrlink: 1d66df69
date: 2022-04-13 14:39:25
---

# C语言进阶学习

<!-- more -->
## 1. typedef、void 和 sizeof 的使用

### typedef
1. 简化 struct 关键字

```cpp
struct Student {
    string name;
    int age;
};
//将struct Student类型简化成Stu，就是少写一个struct
typedef struct Student Stu;

//更简单的写法就是
typedef struct Student {
    string name;
    int age;
} Stu;

void test() {
    struct Student s1 = {Cherry, 20};
    //可以直接用Stu定义一个结构体变量
    Stu s2 = {Alice, 23};
}
```

2. 区分数据类型

```cpp
void test2() {
    //这里p1是char*类型，而p2是char类型
    char *p1, p2;
    //为了做更好的区分，可以使用typedef定义char*
    typedef char* PCHAR;
    //这样两个变量都是char指针类型，这样等价于 char *p1, *p2;
    PCHAR p1, p2;
}
```

3. 提高移植性

```cpp
typedef long long MYTYPE
long long a = 10;
long long b = 20;

//可以写成
MYTYPE a = 10;
MYTYPE b = 10;
```

### void

1. 无类型，不能通过无类型创建变量，因为不知道分配多少内存空间

```cpp
//这样是错误的，编译器会报错
void a = 2;
```

2. 可以限定函数的返回值，限定函数参数

```cpp
#include <stdio.h>

func() {
    return 10;
}

void test2() {
    func(10, 20);
}

int main() {
    test2();
    return 0;
}
```

上面这段 `C` 代码，看似有很多问题：func 函数没有返回类型，返回了 10，并且函数没有传入参数，调用函数时还传入了参数，但是该 C 语言代码依旧可以运行，可以看出 C 语言并不严谨。

这时将 func 函数改成下面这样，编译器就会报出警告

```cpp
func(void) {
    return 10;
}
```
3. void* 万能指针

```cpp
void *p = NULL;

int *pInt = NULL;
char *pChar = NULL;

//不同类型指针之间需要强制转换才不会报警告
pChar = (char*)pInt;

//通过 void* 万能指针，就不会报警告了
pChar = p;
```

### sizeof

1. 首先 `sizeof` 本质上不是一个函数，而是一个运算符，类似于 `+-*/`

```cpp
void test01() {
    double d = 3.14;
    printf("sizeof int = %d\n", sizeof(int));
    //对于统计变量的时候，可以不加小括号，所以他不是函数
    printf("sizeof d = %d\n", sizeof d);
}
```

2. `sizeof` 返回一个无符号整型

```cpp
void test02() {
    unsigned int a = 10;
    if (a - 20 > 0) {
        printf("大于0");
    } else {
        printf("小于0");
    }
}
```

这段代码输出结果为 `大于0`，因为一个无符号整数和有符号整数做运算，最后统一转换为无符号整数。

同理可以验证 sizeof 返回的也是无符号整数：

```cpp
void test02() {
    if (sizeof(int) - 5 > 0) {
        printf("大于0");
    } else {
        printf("小于0");
    }
}
```

最后得到的结果也是 `大于0`。

3. `sizeof` 其他用法

统计数组占用内存空间大小

```cpp
#include <stdio.h>

void test(int arr[]) {
    printf("##sizeof arr = %d\n", sizeof(arr));
}

int main() {
    int arr[] = {1, 2, 3, 4};
    printf("sizeof arr = %d\n", sizeof(arr));
    test(arr);
    return 0;
}
```

最后输出为：

```
sizeof arr = 16
##sizeof arr = 8
```

这是因为当数组以参数的形式传递时，将得到的是数组的指针，即数组的第一个元素所在的位置，这时打印出的大小便是指针的大小。

## 2. C语言 %d 等输出格式意义

1. %d整型输出，%ld长整型输出。

2. **%p指针变量地址，如果输出数据不够8位数，则左边补零。**

3. %o以八进制数形式输出整数。

4. %x以十六进制数形式输出整数。

5. %u以十进制数输出unsigned型数据(无符号数)。

6. %c用来输出一个字符。

7. %s用来输出一个字符串。

8. %f用来输出实数，以小数形式输出。

9. %e以指数形式输出实数。

10. %g根据大小自动选f格式或e格式，且不输出无意义的零。

## 3. 内存的四个区域

- 代码区
  - 共享
  - 只读
- 数据区
  - 静态变量，全局变量，常量
  - 已初始化（data段）
  - 未初始化（bss段）
- 栈区
  - 编译器自动分配释放，存放函数的参数值、返回值、局部变量等
  - 局部变量的生存周期为函数内部申请到释放该段栈空间
- 堆区
  -  容量远大于栈
  -  用于动态内存分配
  -  堆在内存中位于 bss 段和栈区之间，一般由程序员分配和释放，程序结束后由 OS 释放

**局部变量在栈空间创建空间举例**

```cpp
char *getString() {
    char str[] = "Hello World";
    return str;
}

void test02() {
    char *str = getString();
    printf("%s\n", str);
}
```

这段代码是无法正确输出 `Hello World` 的，因为在 `getString` 中创建的是局部变量 `str`，是在栈空间中创建空间，而在函数 `getString` 运行结束后，栈空间将被释放，因此获得的 `str` 指针为空。

*即不要返回局部变量的地址*

**堆区举例**

```cpp
//堆区
int *getSpace() {
    //开辟4*5=20B空间，开辟到了堆区
    int *p = (int*)malloc(sizeof(int) * 5);
    if (p == NULL) {
        return NULL;
    }
    for (int i = 0; i < 5; i++) {
        p[i] = i + 100;
    }
    return p;
}

void test03() {
    int *p = getSpace();
    for (int i = 0; i < 5; i++) {
        printf("%d ", p[i]);
    }
    free(p);        //释放掉p指向的内存区域
    p = NULL;       //释放掉之后将p指向空，避免p成为野指针
    for (int i = 0; i < 5; i++) {
        printf("%d ", p[i]);
    }
}
```

**堆区分配内存注意点**

```cpp
//堆区注意点
void allocateSpace(char *pp) {
    char *temp = malloc(100);
    memset(temp, 0, 100);
    strcpy(temp, "Hello World");
    pp = temp;
}

void test04() {
    char *p = NULL;
    allocateSpace(p);
    printf("p = %s", p);
}
```

上面的代码输出结果实际上是 null，并不会输出字符串 `Hello World`

因为在 `test04` 中，传入的参数是 `char` 指针，是一个局部变量，在栈中分配内存。函数 `allocateSpace` 中的参数也是在栈中分配内存，只不过 `p` 是在 `test04` 的栈中，而 `pp` 是在 `allocateSpace` 的栈中。而用 `malloc` 在堆中分配内存时，返回的首地址指针赋值给了一个局部变量 `temp`，然后在这个堆空间中复制字符串 `Hello World`，最后这个首地址指针又赋值给了 `pp`。因此自始至终都未修改 `test04` 栈中的 `p` 指针的值，所以最后输出的是 `null`。

要想修改 `p` 指针的值，需要用更高级别的指针进行修改，这时传入函数 `allocateSpace` 的参数应该是 `p` 指针的地址，即 `p` 指针的指针（二级指针），这样才能完成对 `p` 指针的赋值操作。

将代码改成下面这样，就可以正确输出结果了：

```cpp
void allocateSpace2(char **pp) {
    char *temp = malloc(100);
    memset(temp, 0, 100);
    strcpy(temp, "Hello World");
    *pp = temp;
}

void test04() {
    char *p = NULL;
    allocateSpace2(&p);
    printf("p = %s", p);
    
}
```

## 4. 静态变量和全局变量

数据区中存放**全局变量、静态变量、常量**

**静态变量**

静态变量在程序运行前分配内存，生命周期在程序结束时死亡。默认属于内部链接属性，只能在该文件内部使用。

**全局变量**

默认在 C 语言下，全局变量前加了关键字 `extern`。属于外部链接属性。

## 5. 常量

**const修饰的常量**

```cpp
const int a = 2022;     //放在数据区的常量区中，受到保护，无法修改

void test01() {
    //无法直接修改一个 const 修饰的常量
    // a = 333;

    //利用指针修改，实际上也是无法成功，在解引用的时候会出现异常
    int *p = &a;
    *p = 100;
    printf("%d", *p);
}
```

全局常量是无法被直接或间接修改的。

```cpp
void test02() {
    //局部常量也无法被直接修改
    const int b = 421;
    // b = 2022;
    //但是可以被间接修改，因为局部变量存放在栈中，不受常量区保护
    int *p = &b;
    *p = 145;
    printf("%d\n", b);
}
```

但是局部常量可以被间接修改，因为局部变量存放在栈中，不受常量区保护。

**字符串常量**

```cpp
void test03() {
    char *p1 = "Hello World";
    char *p2 = "Hello World";
    char *p3 = "Hello World";

    printf("%p\n", p1);
    printf("%p\n", p2);
    printf("%p\n", p3);
    printf("%p\n", &"Hello World");
}
```

它们输出的指针是同一个，说明同样的字符串常量在内存中只保留一份，节省了内存。

```cpp
//字符串常量也是不可以修改的
void test04() {
    char *p1 = "Hello World";
    p1[0] = 'B';
    printf("%s", p1);
}
```

但是有些编译器支持修改字符串常量。而且对于相同的字符串常量是否共享，不同的编译器会有不同的结果。

## 6. 函数调用流程

### 宏函数

将一些频繁使用且短小的函数定义成宏函数，定义宏函数时要注意函数的完整性。

优点：**以空间换时间**，这里的时间就是将宏函数当做普通函数时参数的入栈出栈操作

```cpp
#define ADD(x, y) ((x) + (y))

void test() {
    int a = 2017;
    int b = 5;
    printf("%d", ADD(a, b));
}
```

### 函数调用惯例

对于下面这一段代码，分析程序调用的流程。

```cpp
int sum(int a, int b) {
    int _a = a;
    int _b = b;
    return _a + _b;
}

int main() {
    int res = 0;
    res = sum(10, 20);
    return 0;
}
```

首先将主函数中 `ret` 进栈，然后调用 `sum` 函数，将改行的返回地址进栈。接着进入到 `sum` 函数中，将参数 `b` 和 `a` 依次进栈，再在栈中为 `_a` 和 `_b` 开辟空间，最后返回两者的和存到一个临时变量中，这个变量也是在栈中，然后函数执行完毕，释放掉栈中的变量，回到返回地址处继续执行代码。

注意点：
1. 参数入栈是从右到左
2. 由主调函数负责出栈

|调用惯例|出栈方|参数传递|名字修饰|
|-------|-----|------|-------|
|cdecl（C和C++默认）|函数调用方（主调函数）|从右到左参数入栈|下划线+函数名|
|stdcall|函数本身（被调函数）|从右到左参数入栈|下划线+函数名+@+参数字节数|
|fastcall|函数本身（被调函数）|前两个参数由寄存器传递，其余参数通过栈传递|@+函数名+@+参数字节数|
|pascal|函数本身（被调函数）|从左到右参数入栈|较为复杂，略|

### 变量传递

main -> 子函数1 -> 子函数2

- main 函数在栈区开辟的内存，所有子函数都可以使用
- main 函数在堆区开辟的内存，所有子函数都可以使用
- 子函数 1 在栈区开辟的内存，子函数 1 和 2 都可以使用
- 子函数 1 在堆区开辟的内存，所有函数都可以使用
- 子函数 2 在全局区开辟的内存，子函数 1 和 main 函数都可以使用


### 栈扩展方向

通过下面的代码可以得到栈顶在低地址，栈底在高地址，是从高地址向低地址扩展的。

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <string.h>


void test(int a, int b, int c) {
    printf("0x%p\n", &a);
    printf("0x%p\n", &b);
    printf("0x%p\n", &c);
}

int main() {
    test(1, 2, 3);
    return 0;
}
```

输出结果为：

```
0x000000001041FE00
0x000000001041FE08
0x000000001041FE10
```

## 7. 野指针和空指针

**空指针**指向 `NULL`，而**野指针**指向一个已经删除的对象或未申请访问受限区域的指针。野指针只能通过良好的代码习惯来避免，而空指针可以通过判断语句来避免。

**野指针的三种情况**

- 指针变量未初始化
  - 指针创建时不会自动设置为 NULL，需要手动置为空或指向有效内存
- 指针释放后未置空
  - 指针释放后只是释放掉了当前指针指向的内存，并没有修改指针的值
- 指针变量超出了作用域
  - 即在函数中返回局部变量的地址


对于指针 `free` 后未置空的情况，下面有个例子可以说明：

```cpp
void test02() {
    char *p = malloc(20);
    strcpy(p, "I want to swim...");
    printf("%s\n", p);
    free(p);
    printf("%s\n", p);
}
```

第一个 `printf` 输出的是正确的字符串，而第二个 `printf` 输出却是不正确的值，因为 `free` 操作释放掉了原来指针指向的内存，那片内存可以被计算机中其他程序使用，因此数据成了脏数据（尽管可能那段内存没有被使用，数据还和原来一样，但是从逻辑上指针已经失去了对那片内存的控制权）。对于野指针，可以读取（编译器不会报错），但是要写入的话就会报错。

对于函数中返回局部变量的地址，因为在函数中的局部变量是在栈中开辟的，函数调用完后将会清空栈中的变量，这样指针指向的地址也就失去了意义，成为了野指针。

```cpp
int *test() {
    int a = 10;
    return &a;
}

int main() {
    int *p = test();
    printf("%p", p);
    printf("%p", p);
    return 0;
}
```

上述代码两句输出理论上都是无法输出正确结果的，但是代码运行结果确是第一个能正确输出，第二个无法正确输出。这是因为编译器的缘故，编译器会保留一次栈中的变量供之后访问。（不同的编译器可能会有不同的结果，但是尽量不要这样写）

另外：空指针可以重复释放，而野指针不能重复释放

```cpp
void test() {
    //这样不会报错
    int *p = 10;
    free(p);
    p = NULL;
    free(p);

    //这样会报错
    int *pp = 10;
    free(pp);
    free(pp);
}
```

## 8. 指针的步长

指针步长指的是指针变量 +1 后跳跃的字节数，也是指指针解引用的时候取出的字节数。下面的例子很好的帮助理解指针的步长：

```cpp
void test05() {
    char buf[10] = {0};
    int a = 2022;
    memcpy(buf + 1, &a, sizeof(int));
    printf("%d\n", *(int*)(buf + 1));
}
```

这样可以成功输出 2022。

下面以一个结构体作为例子，进一步说明指针步长：

```cpp
struct Pointer {
    char a;         //0~3
    int b;          //4~7
    char buf[64];   //8~71
    int d;          //72~76
};

void test06() {
    struct Pointer p = {'B', 2022, "I will go to Beijing soon...", 8013};
    //p中d字段的偏移量？(包含在 stddef.h 头文件中)
    /* 
     *  #define offsetof(s, m) (size_t)&(((s*)0)->m)
     */
    printf("d's offset =  %d\n", offsetof(struct Pointer, d));
    printf("%d\n", *(int*)((char*)(&p) + offsetof(struct Pointer, d)));
}
```

上述代码要输出结构体中字段 `d`，我们首先获取 `d` 的偏移量：`offsetof(struct Pointer, d)`，然后取得结构体首地址，将指针类型转成 `char*`，然后加上偏移量之后再转成 `int*`，最后再解引用，输出结果 `8013`。

## 9. 指针做函数参数

指针做函数参数，具有**输入**和**输出**两种特性

**输入特性**：在主调函数分配内存，被调函数中使用
**输出特性**：在被调函数分配内存，主调函数中使用（被调函数中要用比主调函数更高级的指针去修改）

## 10. 字符串基本操作

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main() {
    //当未定义字符数组的长度时，会输出乱码，因为输出直到出现 '\0' 为止
    char str1[] = {'B', 'e', 'i', 'j', 'i', 'n', 'g'};
    printf("str1 = %s\n\n", str1);

    //当定义了字符数组的长度时，对字符数组初始化后会在后面默认用 0 填充
    char str2[20] = {'B', 'e', 'i', 'j', 'i', 'n', 'g'};
    printf("str2 = %s\n\n", str2);

    //若以字符串初始化，编译器会默认在字符串最后添加 '\0'
    char str3[] = "Beijing";
    printf("str3 = %s\n", str3);
    printf("sizeof str3 = %d\n", sizeof(str3));      // 8 sizeof 计算 '\0'
    printf("strlen str3 = %d\n\n", strlen(str3));    // 7 strlen 不计算 '\0'

    char str4[20] = "Beijing";
    printf("str4 = %s\n", str4);
    printf("sizeof str4 = %d\n", sizeof(str4));      // 20
    printf("strlen str4 = %d\n\n", strlen(str4));    // 7

    char str5[] = "Beijing\0Welcome";
    printf("str5 = %s\n", str5);
    printf("sizeof str5 = %d\n", sizeof(str5));      // 16
    printf("strlen str5 = %d\n\n", strlen(str5));    // 7

    char str6[] = "Beijing\012Welcome";
    printf("str6 = %s\n", str6);
    printf("sizeof str6 = %d\n", sizeof(str6));      // 16 \012 是用八进制表示的转义字符，表示换行，再加上末尾的 '\0'
    printf("strlen str6 = %d\n\n", strlen(str6));    // 15  
    return 0;
}
```

最终输出结果为：

```cpp
str1 = Beijing爅€       //后面为乱码

str2 = Beijing

str3 = Beijing
sizeof str3 = 8
strlen str3 = 7

str4 = Beijing
sizeof str4 = 20
strlen str4 = 7

str5 = Beijing
sizeof str5 = 16
strlen str5 = 7

str6 = Beijing
Welcome
sizeof str6 = 16
strlen str6 = 15
```

## 11. 字符串拷贝

这里实现了字符串拷贝的三种实现。

```cpp
//第一种实现
void copyString01(char *dest, char *src) {
    //利用下标方式拷贝
    int len = strlen(src);
    for (int i = 0; i < len; i++) {
        dest[i] = src[i];
    }
    dest[len] = '\0';
}

//第二种实现
void copyString02(char *dest, char *src) {
    int i = 0;
    while (*(src + i) != '\0') {
        *(dest + i) = *(src + i);
        i++;
    }
    *(dest + i) = '\0';
}

//第三种实现
void copyString03(char *dest, char *src) {
    while (*dest++ = *src++);
}
```

其中第三种实现只有一行代码 `while (*dest++ = *src++);`，在 `while` 循环中，当 `*src` 取到 `\0` 时，就把它当做 `0` 看待，因此 `while` 循环条件中出现 `0` 便不会继续循环下去，因此会一直拷贝 `src` 直到遇到 `\0`，结束循环时，`src` 指向 `\0` 的下一个位置。

字符串翻转实现同理。

## 12. 字符串格式化

1. **sprintf 函数**

函数定义：

```cpp
int sprintf(char *str, const char *format, ...)
```

这个函数的参数是可变参数。若成功则返回实际格式化的字符个数，否则返回 -1

具体用法参考 **《C++常用函数整理》** 这篇博客。

2. **sscanf 函数**

函数定义：

```cpp
int sscanf(const char *str, const char *format, ...);
```

功能：
从 str 指定的字符串读取数据，并根据参数 format 字符串来转换并格式化数据

返回值：
成功返回参数数目，失败返回 -1

| 格式 | 作用 |
|---|---|
|%*d 或 %*s|跳过数据|
|%[width]s|读取指定宽度的数据|
|%[a-z]|匹配所有小写字母|
|%[^a]|匹配非字符a的任意字符|

例子：

```cpp
char *str = "http://192.168.3.107:8080";

void test01() {
    // char *str = "1234ABCD";
    char buf[30] = {0};
    sscanf(str, "%*[^/]//%[^:]%*s", buf);
    printf("%s\n", buf);
}


void test02() {
    // char *str = "ABCD2022";
    char buf[30] = {0};
    // sscanf(str, "%*s%d", buf);      //这样写的话在 %*s 的时候就已经忽略掉整个 str 了
    // printf("%s\n", buf);

    // 应该像下面这样
    char *str = "ABCD 2022";        //加上一个空格
    sscanf(str, "%*s%s", buf);
    printf("%s\n", buf);
}

void test03() {
    char buf[30] = {0};
    sscanf(str, "%15s", buf);
    printf("%s\n", buf);

}

//如果第一次匹配失败，后续都不再进行匹配
void test04() {
    char *str = "ccherry20221314";
    char buf[30] = {0};
    sscanf(str, "%[chery]", buf);
    printf("%s\n", buf);
}

void test05() {
    char *str = "abcde12345";
    char buf[30] = {0};
    sscanf(str, "%[^e]", buf);
    printf("%s\n", buf);
}

void test06() {
    char s[30] = {"http://192.168.3.107:8080"}, ss[10];
    char protocol[20], host[20], port[20];
    sscanf(s,"%[^:]://%[^:]:%[1-9]", protocol, host, port);
    /*这样可以把协议名称，IP名称和端口号分别取出*/
}
```

输出结果为：

```cpp
192.168.3.107
2022
http://192.168.
ccherry
abcd
```

要注意的一点是：**如果第一次匹配失败，后续将不再匹配。**

**sscanf 的练习**

```cpp
//sscanf练习
void test06() {
    char *ip = "192.168.1.109";
    int num1 = 0, num2 = 0, num3 = 0, num4 = 0;
    sscanf(ip, "%d.%d.%d.%d", &num1, &num2, &num3, &num4);
    printf("num1 = %d\n", num1);
    printf("num2 = %d\n", num2);
    printf("num3 = %d\n", num3);
    printf("num4 = %d\n", num4);
}

void test07() {
    char *str = "GMY#baominyang@hrsoft.net";
    char buf[50] = {0};
    sscanf(str, "%*[^#]#%[^@]", buf);
    printf("My name is %s\n", buf);
}
```

## 13. malloc、calloc、realloc

**calloc**

```cpp
void *calloc(size_t nmemb, size_t size);
```

功能：在内存动态存储区中分配 `nmemb` 块长度为 `size` 字节的连续区域，**`calloc` 自动将分配的内存置零。**

参数：
- nmenb：所需内存单元数量
- size：每个内存单元的大小（字节）

返回值：内存起始地址，错误则返回 NULL.

**realloc**

```cpp
void *realloc(void *ptr, size_t size);
```

功能：

重新分配用 `malloc` 和 `calloc` 分配的空间。
realloc不会自动清理增加的内存（即新增加的内存不会赋值为0），需要手动清理，如果指定的地址后面有连续的空间，那么就会在已有地址基础上增加内存，否则就会重新分配新的连续内存，把旧内存的值拷贝到新内存，同时释放旧内存。

参数：

- ptr: 为之前分配的内存的起始地址，若为 NULL，则功能和 `malloc` 一致
- size: 重新分配的内存的大小

## 14. const

全局 const 无法被修改，局部 const 当做局部变量使用，可以被间接修改。

