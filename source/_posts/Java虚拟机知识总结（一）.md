---
title: Java虚拟机知识总结(一)
mathjax: true
tags:
  - JVM
categories:
  - Java虚拟机
abbrlink: bc4fb6eb
date: 2020-05-24 14:42:02
---

## Java虚拟机介绍

Java虚拟机(Java Virtual Machine，JVM)，一种能够运行Java字节码的虚拟机。作为一种编程语言的虚拟机，实际上
不只是专用于Java语言，只要生成的编译文件匹配JVM对加载编译文件格式要求，任何语言都可以由JVM编译运行。
<!-- more -->

## JVM基本结构

![JVM基本结构](47ec812d/JVM基本结构.png)

JVM由三个主要的子系统构成

- 类加载子系统
- 运行时数据区（内存结构）
- 执行引擎

## 类加载机制

![类的生命周期](47ec812d/类加载机制.png)

1. 加载
将``.class``文件从磁盘读到内存
2. 链接
   1. 验证
验证字节码文件的正确性
   2. 准备
给类的静态变量分配内存
   3. 解析
类加载器装入类所引用的其他所有类(静态链接)
静态链接：解析阶段，由符号引用转化为直接引用
动态链接：程序运行期间，由符号引用转化为直接引用
3. 初始化
为类的静态变量赋予正确的初始值，上述的准备阶段为静态变量赋予的是虚拟机默认的初始值，此处赋予的才是程序
编写者为变量分配的真正的初始值，执行静态代码块
4. 使用
5. 卸载

### 类加载器的种类

- 启动类加载器(Bootstrap ClassLoader)
负责加载JRE的核心类库，如JRE目标下的``rt.jar``,``charsets.jar``等
- 扩展类加载器(Extension ClassLoader)
负责加载JRE扩展目录``ext``中jar类包
- 系统类加载器(Application ClassLoader)
负责加载``ClassPath``路径下的类包
- 用户自定义加载器(User ClassLoader)
负责加载用户自定义路径下的类包

![加载器](47ec812d/加载器.png)

### 双亲委派机制

#### 全盘负责委托机制

当一个``ClassLoader``加载一个类的时候，除非显式的使用另一个``ClassLoader``,该类所依赖和引用的类也由这个``ClassLoader``载入

#### 双亲委派机制

指先委托父类加载器寻找目标类，在找不到的情况下载自己的路径中查找并载入目标类。实际上``双亲委派机制``实则``父类委派机制``。

##### 双亲委派模式的优势

- 沙箱安全机制：比如自己写的``String.class``类不会被加载，这样可以防止核心库被随意篡改
- 避免类的重复加载：当父``ClassLoader``已经加载了该类的时候，就不需要子``ClassLoader``再加载一次

*要确定一个类的唯一性，要获得该类的类加载器实例以及类的全限定名。*

不同的类加载器加载同一个``class``文件是不同的类模板信息

为什么要打破双亲委派机制？
Tomcat为了做wap包隔离

### 运行时数据区

![运行时数据区](47ec812d/运行时数据区.png)

堆：用来放类的实例对象
栈：栈帧，用来存放方法，线程

#### 方法区（Method Area）(永久代/持久代jdk1.8以前，元空间)

类的所有字段和方法字节码，以及一些特殊方法如构造函数，接口代码也在这里定义。简单来说，所有定义的方法的信息都保存在该区域，静态变量+常量+类信息（构造方法/接口定义）+运行时常量池都存在方法区中，虽然Java虚拟机规范把方法区描述为堆的一个逻辑部分，但是它却有一个别名叫做Non-Heap（非堆），目的应该是为了和Java的堆区分开(jdk1.8以前hotspot虚拟机叫永久代、持久代，jdk1.8时叫元空间)

#### 堆（Heap）

- YoungGC/MinorGC
- CMS OldGC
- MajorGC/FullGC

虚拟机启动时自动分配创建，用于存放对象的实例，几乎所有对象都在堆上分配内存，当对象无法在该空间申请到内存是将抛出``OutOfMemoryError(OOM)``异常。同时也是垃圾收集器管理的主要区域。

![堆](47ec812d/堆.png)

当年龄到15时，转入老年代

##### 新生代（Young Generation）

类出生、成长、消亡的区域，一个类在这里产生，应用，最后被垃圾回收器收集，结束生命。
新生代分为两部分：伊甸区``（Eden space）``和幸存者区``（Survivor space）``，所有的类都是在伊甸区被new出来的。
幸存区又分为``From``和``To``区。当``Eden``区的空间用完是，程序又需要创建对象，JVM的垃圾回收器将``Eden``区进行垃圾回
收``（Minor GC）``，将``Eden``区中的不再被其它对象应用的对象进行销毁。然后将``Eden``区中剩余的对象移到``From Survivor``区。若``From Survivor``区也满了，再对该区进行垃圾回收，然后移动到``To Survivor``区。

##### 老年代（Old Generation）

新生代经过多次GC仍然存货的对象移动到老年区。若老年代也满了，这时候将发生``Major GC``（也可以叫``Full GC``），进行老年区的内存清理。若老年区执行了``Full GC``之后发现依然无法进行对象的保存，就会抛出``OOM（OutOfMemoryError）``异常

##### 元空间（Meta Space）

在``JDK1.8``之后，元空间替代了永久代，它是对JVM规范中方法区的实现，区别在于元数据区不在虚拟机当中，而是用的本地内存，永久代在虚拟机当中，永久代逻辑结构上也属于堆，但是物理上不属于。

**为什么移除了永久代？**
参考官方解释 **http://openjdk.java.net/jeps/122**

大概意思是移除永久代是为融合``HotSpot``与``JRockit``而做出的努力，因为``JRockit``没有永久代，不需要配置永久代。

![新生代和老年代转化](47ec812d/堆和GC.png)

#### 栈(Stack)

Java线程执行方法的内存模型，一个线程对应一个栈，每个方法在执行的同时都会创建一个栈帧（用于存储局部变量表，操作数栈，动态链接，方法出口等信息）不存在垃圾回收问题，只要线程一结束该栈就释放，生命周期和线程一致.

#### 本地方法栈(Native Method Stack)

和栈作用很相似，区别不过是Java栈为JVM执行Java方法服务，而本地方法栈为JVM执行native方法服务。登记``native``方法，在``Execution Engine``执行时加载本地方法库

#### 程序计数器(Program Counter Register)

就是一个指针，指向方法区中的方法字节码（用来存储指向下一跳指令的地址，也即将要执行的指令代码），由执行引擎读取下一条指令，是一个非常小的内存空间，几乎可以忽略不计

---

###### 将下面代码编译生成``Class``字节码并反汇编：

```java
public class JVM {

   public int math() {
      int a = 1;
      int b = 5;
      int c = (a + b) * 10;
      System.out.println(a);
      return c;
   }

   public static void main(String args[]) {
      JVM jvm = new JVM();
      System.out.println(jvm.math());
   }
}
```

反汇编后：``(javap -c JVM.class > JVM.txt)``

```java
Compiled from "JVM.java"
public class JVM {
  public JVM();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public int math();
    Code:
       0: iconst_1
       1: istore_1
       2: iconst_5
       3: istore_2
       4: iload_1
       5: iload_2
       6: iadd
       7: bipush        10
       9: imul
      10: istore_3
      11: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
      14: iload_1
      15: invokevirtual #3                  // Method java/io/PrintStream.println:(I)V
      18: iload_3
      19: ireturn

  public static void main(java.lang.String[]);
    Code:
       0: new           #4                  // class JVM
       3: dup
       4: invokespecial #5                  // Method "<init>":()V
       7: astore_1
       8: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
      11: aload_1
      12: invokevirtual #6                  // Method math:()I
      15: invokevirtual #3                  // Method java/io/PrintStream.println:(I)V
      18: return
}
```

很多其实都是编译原理的知识，下面是数据区：

![数据区](47ec812d/数据区.png)

反汇编之后前面的数字序号便是程序计数器``PC``

目前市面上大部分虚拟机都是用``C/C++``实现的，实际上最后启动线程都是通过``C/C++``库来调用操作系统内核函数实现。

利用``javap -v JVM.class < dynamicLink.txt``查看动态链接详细信息：

```java
Classfile /G:/JVM.class
  Last modified 2020-5-24; size 493 bytes
  MD5 checksum 5cdaa7b4d5fb74c45c86c10186ae5c8c
  Compiled from "JVM.java"
public class JVM
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #7.#18         // java/lang/Object."<init>":()V
   #2 = Fieldref           #19.#20        // java/lang/System.out:Ljava/io/PrintStream;
   #3 = Methodref          #21.#22        // java/io/PrintStream.println:(I)V
   #4 = Class              #23            // JVM
   #5 = Methodref          #4.#18         // JVM."<init>":()V
   #6 = Methodref          #4.#24         // JVM.math:()I
   #7 = Class              #25            // java/lang/Object
   #8 = Utf8               <init>
   #9 = Utf8               ()V
  #10 = Utf8               Code
  #11 = Utf8               LineNumberTable
  #12 = Utf8               math
  #13 = Utf8               ()I
  #14 = Utf8               main
  #15 = Utf8               ([Ljava/lang/String;)V
  #16 = Utf8               SourceFile
  #17 = Utf8               JVM.java
  #18 = NameAndType        #8:#9          // "<init>":()V
  #19 = Class              #26            // java/lang/System
  #20 = NameAndType        #27:#28        // out:Ljava/io/PrintStream;
  #21 = Class              #29            // java/io/PrintStream
  #22 = NameAndType        #30:#31        // println:(I)V
  #23 = Utf8               JVM
  #24 = NameAndType        #12:#13        // math:()I
  #25 = Utf8               java/lang/Object
  #26 = Utf8               java/lang/System
  #27 = Utf8               out
  #28 = Utf8               Ljava/io/PrintStream;
  #29 = Utf8               java/io/PrintStream
  #30 = Utf8               println
  #31 = Utf8               (I)V
{
  public JVM();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 1: 0

  public int math();
    descriptor: ()I
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=4, args_size=1
         0: iconst_1
         1: istore_1
         2: iconst_5
         3: istore_2
         4: iload_1
         5: iload_2
         6: iadd
         7: bipush        10
         9: imul
        10: istore_3
        11: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
        14: iload_1
        15: invokevirtual #3                  // Method java/io/PrintStream.println:(I)V
        18: iload_3
        19: ireturn
      LineNumberTable:
        line 4: 0
        line 5: 2
        line 6: 4
        line 7: 11
        line 8: 18

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=2, args_size=1
         0: new           #4                  // class JVM
         3: dup
         4: invokespecial #5                  // Method "<init>":()V
         7: astore_1
         8: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
        11: aload_1
        12: invokevirtual #6                  // Method math:()I
        15: invokevirtual #3                  // Method java/io/PrintStream.println:(I)V
        18: return
      LineNumberTable:
        line 12: 0
        line 13: 8
        line 14: 18
}
SourceFile: "JVM.java"
```

第一段的``Constant pool``是符号引用
