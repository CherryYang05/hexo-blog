---
title: Java虚拟机知识总结（二）
mathjax: true
tags:
  - JVM
categories:
  - Java虚拟机
abbrlink: 7a3f40e8
date: 2020-05-24 17:28:44
---

## 三、GC算法和收集器

本文参考：
Oracle Java JVM Standard Options：https://docs.oracle.com/javase/8/docs/technotes/tools/windows/java.html
HotSpot Glossary of Terms：http://openjdk.java.net/groups/hotspot/docs/HotSpotGlossary.html
周志明《深入理解Java虚拟机》第二版

### 如何判断对象可以被回收

堆中几乎放着所有的对象实例，对堆垃圾回收前的第一步就是要判断哪些对象已经死亡（即不能再被任何途径使用的对象）

#### 引用计数法

给对象添加一个引用计数器，每当有一个地方引用，计数器就加1。当引用失效，计数器就减1。任何时候计数器为0的对象就是不可能再被使用的。
这个方法实现简单，效率高，但是目前主流的虚拟机中没有选择这个算法来管理内存，最主要的原因是它很难解决对象之前相互循环引用的问题。所谓对象之间的相互引用问题，通过下面代码所示：除了对象a和b相互引用着对方之外，这两个对象之间再无任何引用。但是它们因为互相引用对方，导致它们的引用计数器都不为0，于是引用计数器法无法通知GC回收器回收它们。
