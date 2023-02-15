---
title: 自动化的实现（包括makefile脚本的使用）
mathjax: true
tags:
  - 脚本
  - makefile
categories:
  - [生产力工具, 脚本]
abbrlink: ec9e8461
date: 2021-03-12 21:26:06
---

# makefile脚本的实现

在此之前，我们编译内核都是通过手动输入一条一条命令实现的，但是随着我们的模块越来越多，每次编译内核再手动链接成一个二进制文件，最后再手动反汇编是很麻烦的。作为一名合格的计算机专业的大学生，这种手动模式是很不专业的，一是手输代码很容易出错，倘若我们更新了某个模块，却忘了重新编译链接的话，那么最后的内核就会出现一些难以排除的诡异 bug；二是，作为计算机专业的大学生，将工作自动化是理所应当的，这也是和其他专业的毕业生的一个区别。因此，要想真正的提升自己的技术水平，就必须懂得将业务逻辑进行自动化实现。

那么从现在开始，我们将把以前手动处理的编译链接反编译等一系列工作，全面自动化。

我们之前在 Windows 上也有一个简单的 bat 脚本，如下所示：

<!-- more -->

```shell
@echo off
::cls
f:
cd F:\Code\Java\Java4LinuxOS
nasm boot.asm -o boot
::copy boot \src
nasm kernel.asm -o kernel
::copy kernel \src
cd src
javac OS.java
java OS
del OS.class
del Floppy.class
del Floppy$MAGNETIC_HEAD.class
::del boot
::del kernel
move system.img ../
cd F:\Code\Java\Java4LinuxOS
e:
cd E:\Virtual\SimpleOS
del system.img
echo 原映像文件已删除
copy F:\Code\Java\Java4LinuxOS\system.img E:\Virtual\SimpleOS
echo 新映像已移动至虚拟机根目录下
f:
move F:\Code\Java\Java4LinuxOS\system.img F:\bochs\Bochs-2.6.11
echo 新映像已移动至bochs根目录下
```

该脚本的作用是用 nasm 编译内核，运行 Java 类生成镜像文件，再将镜像文件传到虚拟机和 bochs 根目录下，这样一来，也能节省不少时间，而且不用启动 Java 编辑器了，也不用再一次次的点击复制粘贴了。
但是这样的话还是需要在 Linux 中手动输入很多冗长的编译指令，下面的 makefile 脚本文件能够实现这些编译，链接，反汇编，以及传输等操作。
首先，是内核 C 语言模块部分的编译，反汇编，以及从虚拟机传输到实体机。现在我们 C 内核有以下模块：write_vga_desktop.c,mem_util.c, mem_util.h, win_sheet.c, win_sheet.h。makefile内容如下：

```makefile
ckernel.asm : ckernel.o
    ./objconv -fnasm ckernel.o -o ckernel.asm
ckernel.o : write_vga_desktop_win.o win_sheet.o mem_util.o
    ld -m elf_i386 -r write_vga_desktop_win.o mem_util.o win_sheet.o -o ckernel.o
write_vga_desktop_win.o : write_vga_desktop_win.c win_sheet.c win_sheet.h mem_util.c mem_util.h
    gcc -m32 -fno-pic -fno-asynchronous-unwind-tables -s -c write_vga_desktop_win.c -o write_vga_desktop_win.o
win_sheet.o : win_sheet.c win_sheet.h
    gcc -m32 -fno-pic -fno-asynchronous-unwind-tables -s -c win_sheet.c -o win_sheet.o
mem_util.o : mem_util.c mem_util.h
    gcc -m32 -fno-pic -fno-asynchronous-unwind-tables -s -c mem_util.c -o mem_util.o
all : ckernel.asm
    @cd src/ && javac -encoding gbk OS.java
    @cd src/ && mv OS.class ../ && mv Floppy.class ../ && mv Floppy\$$MAGNETIC_HEAD.class ../ && mv ProcessASMFile.class ../
    @rm *.class
    java OS ckernel.asm
    @cp system.img /home/cherry/Virtual
    @echo "镜像已移动到虚拟机根目下"
    @mv system.img /home/cherry/Bochs
    @echo "镜像已移动到Bochs根目下"
    @rm ckernel.o write_vga_desktop_win.o win_sheet.o mem_util.o
clean:
    rm ckernel.o write_vga_desktop_win.o win_sheet.o mem_util.o
```

其中，我将OS.java和ProcessASMFile.java结合起来，并且增加了通过Java来执行命令行的功能，使用``Runtime.getRuntime().exec()``函数实现。

makefile脚本中转义字符为``\$``，在要转义字符的前面加上这个符号表示转义单个字符。

我们将不想显示的命令前面加``@``来隐藏，然后再将我们需要操作的文件夹在虚拟机上共享。

最终运行效果如下：

![001](ec9e8461/001.png)

![002](ec9e8461/002.png)

感觉makefile脚本中all指令执行有点繁琐，总是将字节码移来移去，最好的方式是在src源代码目录下运行Java，处理asm文件，运行nasm命令，然后在根目录下生成boot, kernel二进制文件并生成system.img镜像
