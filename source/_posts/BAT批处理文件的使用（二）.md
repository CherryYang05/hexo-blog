---
title: BAT批处理文件的使用（二）
subtitle: '批量删除指定文件以及相关简单的语法结构(if,for)，系统变量errorlevel，变量延迟等'
mathjax: true
tags:
  - BAT
  - 批处理
  - 脚本
categories:
  - [生产力工具, 脚本]
abbrlink: 58ced44c
date: 2020-05-09 18:44:22
---
## BAT批处理（二）：批量删除指定文件以及相关简单的语法结构(if,for)，系统变量errorlevel，变量延迟等
前两天在使用 VSCode 的时候，看到之前那么多 cpp 编译产生的 exe 可执行文件，于是心生一个念头，我要把它们全部删掉，但是又不想一个个地删，便想到了利用 ``bat`` 脚本批量删除 exe 文件。于是在搜集了大量资料后，开始了编写。
<!-- more -->
首先就是一条简单的 ``del``：
```shell
del F:\Code\C++\*.exe
```
但是发现只能删除掉 ``C++`` 那个文件夹根目录下的 exe 文件，然后发现，在后面加上 ``/s``，表示在当前目录递归删除指定文件，就是还要进入子目录寻找文件。

>/a 根据百属性选择要删除的文件
/f 强制删除只读度文件
/s 从所有子目录删除指定文件
/q 安静模式。删除全局通配符时，不要求确知

```shell
del F:\Code\C++\*.exe /s
```
运行之后，哗啦啦，全删掉了(在其他盘里测试一下)：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200509172606608.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjUwOTg4,size_16,color_FFFFFF,t_70)
本来其实这样就可以了，把这个批处理脚本放到 C++ 文件夹根目录就行了，但是如果多次执行的时候，本来已经没有 exe 文件了，还要执行，就会提示 ``找不到 G:\test\*.exe``，对于强迫症的我当然不能忍受起码提示一个 ``exe文件已经删完啦`` 之类的。要这样写势必要进行条件判断，然后，又去找了很多资料。
一开始搜到一个 ``errorlevel`` 的系统变量，会判断你当前这条语句是否执行，如果成功执行该变量的值为 ``0``， 否则是 ``1-255`` 之间的某一个值，这里的值主要是根据语句未能成功执行的原因（其实就是跟中断类型差不多）来决定的，一般是 ``1``
```shell
@echo off 
if ERRORLEVEL 1 goto fail
if ERRORLEVEL 0 goto success
goto done
:fail
echo exe文件已经没有啦，别删啦~
goto done
:success
echo 正在删除所有烦人的exe文件...
del F:\Code\C++\*.exe /s
:done
echo 执行完成
```
运行后发现了中文乱码，于是改一下编码为 ``ANSI``![在这里插入图片描述](https://img-blog.csdnimg.cn/20200509174008621.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjUwOTg4,size_16,color_FFFFFF,t_70)
然后再次执行，发现不管 ``exe`` 文件是否被删掉，都会执行 ``success`` 标号的语句，说明 ``del`` 语句都被执行了，只是找不到文件而已。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200509174405846.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjUwOTg4,size_16,color_FFFFFF,t_70)
这种方法不行那就试试其他方法，用 ``if exist`` 条件判断应该可以吧
```shell
@echo off 
::if ERRORLEVEL 1 goto fail
::if ERRORLEVEL 0 goto success
if exist F:\Code\C++\*.exe (goto success) else (goto fail)
goto done
:fail
echo exe文件已经没有啦，别删啦~
goto done
:success
echo 正在删除所有烦人的exe文件...
del F:\Code\C++\*.exe /s
:done
echo 执行完成
```
这里的 ``::``是注释，当然注释还有其他写法

>1、:: 注释内容（第一个冒号后也可以跟任何一个非字母数字的字符）
    2、rem 注释内容（不能出现重定向符号和管道符号）
    3、echo 注释内容（不能出现重定向符号和管道符号）〉nul
    4、if not exist nul 注释内容（不能出现重定向符号和管道符号）
    5、:注释内容（注释文本不能与已有标签重名）
    6、%注释内容%（可以用作行间注释，不能出现重定向符号和管道符号）
    7、goto 标签 注释内容（可以用作说明goto的条件和执行内容）
    8、:标签 注释内容（可以用作标签下方段的执行内容

完美执行。
然后在之后的其他测试中，若 ``C++`` 根目录下没有 ``exe`` 文件，而在其子文件夹下有 ``exe`` 文件的话，就出问题了，因为 ``if exist`` 那行语句只是判断当前文件夹根目录下是否存在 ``exe`` 文件。那该怎么办呢？
应该要对 ``C++`` 根目录进行递归查找，便是 ``for`` 语句：
```shell
@echo off
cls
set PATH=F:\Code\C++\
set FILE=*.exe	
set cnt=0
for /R %PATH% %%s in (%FILE%) do (
	::echo %%s
	set /a cnt=%cnt%+1
)
echo 删除了%cnt%个文件！
if %cnt% EQU 0 (goto fail) else (goto success)
goto done
:fail
echo exe文件已经没有啦，别删啦~
goto done
:success
echo 正在删除所有烦人的exe文件...
del *.exe /s
echo 删除了%cnt%个文件！
goto done
:done
echo 执行完成
```
``for`` 循环后面的 ``/R`` 便是对当前文件夹进行递归查找，若存在后缀名为 ``.exe`` 文件，便把计数值+1.然后运行，又出错了。。。
cnt的值始终为1，这又是为什么呢？
又查了许多资料，发现 批处理语法中还有叫什么 ``变量延迟`` 的语法，具体就是 **当我们准备执行一条命令的时候，命令解释器会先将命令读取，如果命令中有环境变量，那么就会将变量的值先读取来出，然后在运行这条命令**，如：``echo %cnt%``，当我们执行这条命令的时候，命令解释器会先读出 ``%cnt%`` 的值，然后执行echo，得到的结果是屏幕上显示出 ``cnt的值``。但是，有的时候，我们在执行一条命令的时候，命令解释器将环境变量的值读出来以后，我们的环境变量的值发生了改变，这时个再执行命令就是使用的变量改变前的值，这就不是我们想要的结果了。
也就是，当 ``for`` 语句执行时，命令解释器首先把它变成
```shell
for /R %s in (*.exe) do (set /a cnt=0+1 )
```
因此在这个循环里面 ``cnt`` 就全是0了，这里我们就需要用到 ``变量延迟`` 了，设置 ``setlocal enabledelayedexpansion``，然后将需要使用变量延迟的变量两边用 ``!`` 表示，即 ``set /a cnt=!cnt!+1``.
关闭变量延迟的话就是 ``setlocal disabledelayedexpansion``.
全部修改完了 ``bat`` 程序如下：

```shell
@echo off
cls
setlocal enabledelayedexpansion
set PATH=F:\Code\C++\
set FILE=*.exe	
set cnt=0
for /R %PATH% %%s in (%FILE%) do (
	::echo %%s
	set /a cnt=!cnt!+1
)
if %cnt% EQU 0 (goto fail) else (goto success)
goto done
:fail
echo exe文件已经没有啦，别删啦~
goto done
:success
echo 正在删除所有烦人的exe文件...
del %PATH%%FILE% /s
echo 删除了%cnt%个文件！
goto done
:done
echo 执行完成
endlocal
```
大功告成！！
