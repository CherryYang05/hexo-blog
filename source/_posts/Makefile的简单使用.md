---
title: Makefile的简单使用
mathjax: true
tags:
  - 脚本
  - Makefile
categories:
  - [生产力工具, 脚本]
abbrlink: 31dce7e5
date: 2023-01-05 15:50:03
---

## Makefile 的简单使用

>makefile 脚本是类 Unix 上常用的脚本文件，通常用来自动化地构建项目。本文介绍构建一个简单的 makefile 脚本，并能够阅读和修改常见的 makefile 脚本文件。

<!-- more -->

### 一、环境及样例源代码

本机环境： MacBook M1
测试用源代码：

- main.cpp
- print.cpp
- add.cpp
- func.h
- makefile

**main.cpp**:

```cpp
#include "func.h"

int main() {
    printHappyNewYear();
    cout << add_one(3) << endl;
    return 0;
}
```

**print.cpp**:

```cpp
#include "func.h"

void printHappyNewYear() {
    cout << "Happy New Year!!" << endl;
}
```

**add.cpp**:

```cpp
#include "func.h"

int add_one(int a) {
    return a + 1;
}
```

**func.h**:

```cpp
#define _FUNC_H_
#ifdef _FUNC_H_

#include<iostream>
using namespace std;

void printHappyNewYear();
int add_one(int a);

#endif
```

### 二、编译方式

#### 2.1 手动编译

这种方式很简单，直接输入下面的命令。

```bash
g++ main.cpp add.cpp print.cpp -o main
```

但这种显然不是我们想要的。

#### 2.2 makefile 脚本编译

如果我们只让他编译但是不链接，可以使用 `-c` 参数。

```bash
g++ main.cpp -c
```

结果可以看到生成一个 `main.o` 文件，`.o` 文件便是 Unix 下的中间目标文件（Objective File）

然后我们逐个编译每个文件。

```bash
g++ add.cpp -c
g++ print.cpp -c
```

然后将所有 `.o` 文件链接到一起，生成可执行文件 `main`。

```bash
g++ *.o -o main
```

当我们只修改某个源文件时，只需要单独编译某个文件而不需要重新编译所有文件。最后重新链接即可。但是当源文件太多的时候，这样也是不方便的，于是使用 makefile 脚本实现自动化。

#### Makefile（Version 1）

```bash
## VERSION 1
main: main.cpp print.cpp add.cpp
	@g++ main.cpp add.cpp print.cpp -o main

clean:
	rm *.o main
```

语法格式：main 这个文件依赖于后面的三个 cpp 文件，下一行的命令前面必须是一个 tab，否则语法错误。

运行 `make` 运行 `makefile` 脚本，或 `make -f Makefile` 根据指定文件名运行脚本。

首先脚本先去找 main，如果 main 不存在，则尝试生成 main。若已经生成了 main，则根据后面的依赖项判断该 main 是不是最新的，若不是最新的，则重新生成，否则不做任何操作。

第一个版本的缺点是若源文件太多，则命令显得很冗长。

#### Makefile（Version 2）

```bash
## VERSION 2
CXX = g++
TARGET = main
OBJ = main.o print.o add.o

$(TARGET): $(OBJ)
	$(CXX) $(OBJ) -o $(TARGET)

main.o: main.cpp
	$(CXX) -c main.cpp
print.o: print.cpp
	$(CXX) -c print.cpp
add.o: add.cpp
	$(CXX) -c add.cpp
```

第二个版本中使用了 `CXX`，`TARGET`，`OBJ` 变量，依次查找依赖，只编译已经修改过的文件，而不会编译所有文件。

#### Makefile（Version 3）

```bash
## VERSION 3
CXX = g++
TARGET = main
OBJ = main.o print.o add.o

CXXFLAGS = -c -Wall

$(TARGET): $(OBJ)
	$(CXX) $^ -o $@

%.o: %.cpp
	$(CXX) $(CXXFLAGS) $< -o $@

.PHONY: clean

clean:
	rm -f *.o $(TARGET)
```

符号说明：

`$@`: 目标文件，`$^`: 所有的依赖文件，`$<`: 第一个依赖文件

`.PHONY` 表示的意思：若在该目录下有一个名叫 `clean` 的文件，那么脚本便无需生成该文件，也就不会执行相应的命令，但是这跟我们期望的不一致。加上 `.PHONY` 之后，依赖 `clean`，因此就会去执行 `clean`。

`.PHONY` 是一个伪目标，可以有效防止在 Makefile 文件中定义的可执行命令的目标规则和工作目录下的实际文件出现名称冲突的问题。

第三个版本中，若将来有其他新的源文件加入之后，只需要在 `OBJ` 变量后面加入新的源文件即可。

#### Makefile（Version 4）

```bash
## VERSION 4
CXX = g++
TARGET = main
SRC = $(wildcard *.cpp)
OBJ = $(patsubst %.cpp, %.o, $(SRC))

CXXFLAGS = -c -Wall

$(TARGET): $(OBJ)
	$(CXX) $^ -o $@

%.o: %.cpp
	$(CXX) $(CXXFLAGS) $< -o $@

.PHONY: clean

clean:
	rm -f *.o $(TARGET)
```

在 Makefile 规则中，通配符会被自动展开。但在变量的定义和函数引用时，通配符将失效。这种情况下如果需要通配符有效，就需要使用函数 `wildcard`，它的用法是：`$(wildcard PATTERN...)`

`wildcard`: 扩展通配符
`patsubst`：替换通配符
`notdir`：去除路径

`SRC = $(wildcard *.cpp)` 表示获得工作目录下所有 `.cpp` 文件并生成列表 `SRC`。
`OBJ = $(patsubst %.cpp, %.o, $(SRC))` 表示将所有 `.cpp` 后缀替换为 `.o` 并生成文件列表 `OBJ`。

这样下来，makefile 文件就相对比较智能化了，新增加文件之后也无需修改脚本了，

总之，脚本使你越懒惰，这个脚本的功能就越强大。