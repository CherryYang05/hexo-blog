---
title: 实现C语言编译器：二、简单数学表达式的语法分析
mathjax: true
tags:
  - 编译器
  - 编译原理
  - C
  - Java
categories:
  - 编译原理
abbrlink: f73c25d7
date: 2022-01-06 16:14:27
---

# 二、简单数学表达式的语法分析

当前所实现的简易编译器是将一条或一组数学表达式编译成类似汇编语言的代码，因此就要给算术表达式设立一组语法规则，判断是否是合法的算数表达式，那么程序才能对输入的表达式进行分析，我们将一组带有分号的算数表达式称为 ``statements``，例如 ``1+2*3+4; 41*2;1+2+3+4;``，这三个表达式的集合称为 ``statements``，同时将一组表达式中的某一条带有分号的表达式称为 ``expression``，也就是说 ``statements`` 是由多个 ``expression`` 组成的。

statements 的语法规则可以写成：

$statements\rightarrow expression;|expression;statements$
<!-- more -->
观察上述表达式，我们发现箭头左边的被解析的对象在右边也出现了，这样便构成了一种循环，在编译原理中称为左循环(LR，Left recursive)

```cpp
statements(buffer) {
    expression(buffer);
    statements(buffer); //此处导致循环调用
}
```

其他的语法规则：

$$
expression \rightarrow expression + term | term \\
term \rightarrow term * factor | factor\\
factor \rightarrow NUMBER | (expression)
$$

现在我们先记住这些，根据代码先整体理解，在后面的内容中再详细介绍。

回到语法规则上，箭头右边有两种替换规则，那么我们应该选取哪一种进行替换呢？在编译原理中，有一种方法叫做 ``lookahead``，举个例子，对于语法：
$statements\rightarrow expression;|expression;statements$
我们按照顺序读取到第一个分号，若下一个字符有意义，那么就选取后者进行解析；如果下一个字符是 ``EOI``，那么说明该表达式已经读完，那么选取前者进行解析。

那么如何解决语法规则中的循环调用呢？我们更改语法规则如下：
$1. \quad statements \rightarrow 空|expression;statements$
$2. \quad expression \rightarrow term \,\, expression'$
$3. \quad expression \rightarrow +term \,\, expression' | 空$
$4. \quad term \rightarrow factor \,\, term'$
$5. \quad term' \rightarrow *factor \,\, term' | 空$
$6. \quad factor \rightarrow number | (expression)$

语法解析树如下：

![语法解析树](f73c25d7/语法解析树.png)
