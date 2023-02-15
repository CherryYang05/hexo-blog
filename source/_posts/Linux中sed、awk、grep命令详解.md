---
title: Linux中sed、awk、grep命令详解
mathjax: true
tags:
  - sed
  - awk
  - grep
  - Linux命令
categories:
  - Linux
abbrlink: 574c20f6
date: 2022-04-01 19:40:35
---

# Linux中sed、awk、grep命令详解

在开始介绍 `sed`、`awk`、`grep` 命令之前，先简单介绍一下 `bash`.

## 一、bash 及一些特性

### 1. 命令行展开
```bash
➜  ~ echo change1 change2 change3 change4 change5
change1 change2 change3 change4 change5
➜  ~ echo change{1..5}
change1 change2 change3 change4 change5
➜  ~ echo change{1..10..2}
change1 change3 change5 change7 change9
➜  ~ echo change{01..10..2}
change01 change03 change05 change07 change09
```

`echo` 命令后面加上 `{}` 表示命令行展开，例如上例中 `{1..5}` 表示从 `1` 展开到 `5`，如果后面再加上一个参数，则表示步长，`{1..10..2}` 表示输出 1 到 10 的奇数，若在参数前面加上 0，则表示用 0 补全。

### 2. alias

输出命令 `alias` 表示查看当前的命令别名，输入 `unalias` 可以取消已经设定的命令别名。例如：

```bash
alias rm="rm -i"
```

### 3. 历史命令

`history` 可以查看所有时间内该机器输入过的命令，可以通过 `!行号` 快捷输入曾经输过的命令，`!!` 表示上一次输入的命令。

### 4. 快捷键

```bash
ctrl + a 光标移动到行首
ctrl + e 光标移动到行尾
ctrl + u 删除光标之前的字符
ctrl + k 删除光标之后的字符
ctrl + l 清屏，相当于 clear
```

## 二、Linux 正则表达式

正则表达式 `REGEXP` 分为两类
- 基本正则表达式 `BRE`
- 扩展正则表达式 `ERE`

Linux 下只有 `sed`、`grep`、`awk` 三个命令可以使用正则表达式，其他命令都无法使用

**Linux 三剑客**

- `grep`：文本过滤工具，模式工具
- `sed`：stream editor，流编辑器，文本编辑工具
- `awk`：Linux 的文本报告生成器（格式化文本），实际上是链接到 `gawk` 上

### 正则表达式的分类

- 基本正则表达式对应元字符有：`^$.[]*`
- 扩展正则表达式在基本正则表达式基础上增加了 `(){}?+|` 等字符

基本功能：

| 符号   | 作用                                            |
| ------ | ----------------------------------------------- |
| ^      | 用于模式最左侧，如 `^abc` 表示以 `abc` 开头的行 |
| $      | 用于模式最右侧，如 `abc$` 表示以 `abc` 结尾的行 |
| ^$     | 组合符，表示空行                                |
| .      | 匹配任意单个字符，不包括空行                    |
| \      | 转义符                                          |
| ^.*    | 匹配任意多个字符开头的行                        |
| .*$    | 匹配任意多个字符结尾的行                        |
| [abc]  | 匹配集合内任意一个字符                          |
| [^abc] | 匹配除了集合中任意一个字符                      |

### 扩展正则表达式 ERE 集合

扩展正则必须用 `grep -E` 才能生效

| 符号   | 作用                               |
| ------ | ---------------------------------- |
| +      | 匹配前一个字符一次或多次           |
| [a]+   | 匹配集合中的符号至少一次或多次     |
| ?      | 匹配前一个字符 0 次或 1 次         |
| ()     | 在括号中的符号表示一个整体         |
| a{n,m} | 匹配前一个字符至少 n 次，至多 m 次 |
| a{n,}  | 匹配前一个字符至少 n 次            |
| a{n}   | 匹配前一个字符正好 n 次            |
| a{,m}  | 匹配前一个字符至多 m 次            |

### 一些正则表达式的例子





## 三、grep

- `grep` 英文全称是 `Global search REgular expression and Print out the line.`
- 作用：文本搜索工具，根据用户指定的模式过滤条件，对目标文本逐行进行匹配检查，打印匹配到的行

```bash
语法
grep [options] [pattern] file
命令   参数      匹配模式   文件数据
                -i: ignorecase，忽略字符的大小写
                -o: 仅显示匹配到的字符串本身
                -v, --invert-match: 显示不能被模式匹配到的行
                -E: 支持使用扩展的正则表达式元字符
                -q, --quiet, --silent: 静默模式，即不输出任何信息
```

| 参数选项     | 参数含义           |
| ------------ | ------------------ |
| -v           | 排除匹配结果       |
| -i           | 不区分大小写       |
| -n           | 显示匹配行与行号   |
| -c           | 只统计匹配的行数   |
| -E           | 使用拓展正则       |
| --color=auto | 为过滤结果添加颜色 |
| -w           | 只匹配过滤的单词   |
| -o           | 只输出匹配到的内容 |


**例子 1：找出文本文件 `bre` 中的空行**

```bash
➜  ~/code/awk grep -n -E '^$' bre
2:
5:
6:
➜  ~/code/awk grep -n -E '^$' bre -c
3
➜  ~/code/awk cat -n bre
     1  abc is a simple word.
     2
     3  What is the answer?
     4  I am 23 years old.
     5
     6
     7  I come from JiangSu...
```

**例子 2：在上面的文件中增加以 `#` 开头的注释行，并筛选出非空行和注释行**

```bash
➜  ~/code/awk cat -n bre
     1  abc is a simple word.
     2
     3  What is the answer?
     4  I am 23 years old.
     5
     6
     7  I come from JiangSu...
     8  #这是一个注释行
➜  ~/code/awk grep '^$' bre -v
abc is a simple word.
What is the answer?
I am 23 years old.
I come from JiangSu...
8:#这是一个注释行
➜  ~/code/awk grep '^$' bre -v | grep -E '#' -v
abc is a simple word.
What is the answer?
I am 23 years old.
I come from JiangSu...
```

用管道符将两个命令连接起来。

**例子3：匹配以 `.` 结尾的行**

```bash
➜  ~/code/awk grep '\.$' bre
abc is a simple word.
I am 23 years old.
I come from JiangSu...
```

`.` 前要用转义符 `\` 进行转义

**【注】：在 Linux 下，所有文件每一行的结尾最后都有一个 `$`，可以加上参数 `-E` 查看，如下所示：**

```bash
➜  ~/code/awk cat -En bre
     1  abc is a simple word.$
     2  $
     3  What is the answer?$
     4  I am 23 years old.$
     5  $
     6  $
     7  I come from JiangSu...$
     8  #这是一个注释行$
```

**例子4：在 `bre` 文件中找到以 `a` 或 `w` 开头的行，忽略大小写**

```bash
➜  ~/code/awk grep -E -n -i '^[a|w]' bre
1:abc is a simple word.
3:What is the answer?
```

`grep` 整体比较简单，主要是正则表达式的使用。

## 四、sed

`sed` 是 `Stream Editor`（字符流编辑器）的缩写，简称流编辑器。

`sed` 是操作、过滤和转换文本内容的强大工具，常用功能包括结合正则表达式对文件实现快速增删改查，较为重要的两个功能是提取富川和提取整行。

### 4.1 sed 语法

`sed` 语法如下：

```bash
sed [选项] [sed内置命令字符] [输入文件]
```

**选项**

| 参数选项 | 解释                                                     |
| -------- | -------------------------------------------------------- |
| -n       | 取消默认 `sed` 的输出，常与 `sed` 内置命令 `p` 一起用    |
| -i       | 直接将修改结果写入文件,不用 `-i`，`sed` 修改的是内存数据 |
| -e       | 多次编辑,不需要管道符了                                  |
| -r       | 支持正则扩展                                             |


**内置命令字符**

| 内置命令字符      | 解释                                                    |
| ----------------- | ------------------------------------------------------- |
| a                 | append，对文本追加，在指定行后面添加一行/多行文本       |
| d                 | Delete，删除匹配行                                      |
| i                 | insert，表示插入文本,在指定行前添加一行/多行文本        |
| p                 | Print，打印匹配行的内容,通常 p 与 -n 一起用             |
| s/正则/替换内容/g | 匹配正则内容，然后替换内容(支持正则)，结尾g代表全局匹配 |

**匹配范围**

| 匹配范围  | 解释                                                               |
| --------- | ------------------------------------------------------------------ |
| 空地址    | 全文处理                                                           |
| 单地址    | 指定文件某一行                                                     |
| /pattern/ | 被模式匹配到的每一行                                               |
| 范围区间  | 10, 20 十到二十行，10,+5 第10行向下5行, /pattern1/ ,/pattern2/     |
| 步长      | 1~2，表示1、3、5、7、9行，2~2两个步长，表示 2、4、6、8、10、偶数行 |

### 4.2 sed 例子

下列所有例子都将使用如下测试用例。

```bash
➜  ~/code/awk cat -n bre
     1  abc is a simple word.
     2
     3  What is the answer?
     4  I am 23 years old.
     5
     6  My tel is 12345678999
     7  My favirate food is hamburger.
     8  I come from JiangSu...
     9
    10
    11  #这是一个注释行
    12  xxxx
    13  xxxxxxx
```

#### 1. 输出文件第3、4行

```bash
➜  ~/code/awk sed "3,4p" -n  bre
What is the answer?
I am 23 years old.
```

`sed` 默认输出是所有的，因此要加上 `-n` 参数，`p` 表示打印。

#### 2. 过滤含有 My 的字符串行

```bash
➜  ~/code/awk sed "/My/p" -n  bre
My tel is 12345678999
My favirate food is hamburger.
```

这里要匹配字符串 `My`，因此要用 `/My/` 进行模式匹配，同样要用 `-n` 参数。

#### 3. 删除有 is 的行

```bash
➜  ~/code/awk sed "/is/d" bre

I am 23 years old.

I come from JiangSu...


#这是一个注释行
xxxx
xxxxxxx
```

我们这里用参数 `d`，表示删除匹配到的行，然后不用加 `-n` 参数，但是 `bre` 文件中的内容并不会被改变，因为 `sed` 是将文件内容读到内存中进行操作的。如果要对源文件进行修改，则要加上 `-i` 参数。


#### 4. 删除第 5 行之后的行

```bash
➜  ~/code/awk sed '5,$d' bre
abc is a simple word.

What is the answer?
I am 23 years old.
```

这里模式是 `5,$d`，`$` 表示文件末尾。

#### 5. 将文件中的 is 全部替换成 IS，并打印前 5 行

```bash
➜  ~/code/awk sed 's/is/IS/g' bre | sed '1,5p' -n    
abc IS a simple word.

What IS the answer?
I am 23 years old.
```

替换模式为 `s/is/IS/g`，这里 `/` 也可以换成 `#` 或 `@`。两个命令用管道连接，后面的 `sed` 无需指定文件名。


#### 6. 将文件中的 is 全替换成 IS，同时将 23 替换成 35

```bash
➜  ~/code/awk sed -e 's/is/IS/g' -e 's#23#35#g' bre
abc IS a simple word.

What IS the answer?
I am 35 years old.

My tel IS 13545678999
My favirate food IS hamburger.
I come from JiangSu...


#这是一个注释行
xxxx
xxxxxxx
```

5 中的例子中有两个操作，是用管道符进行操作的，但是 `sed` 参数中提供了 `-e` 进行多次编辑，因此就不需要使用管道了。5 也可以这样写：


```bash
➜  ~/code/awk sed -e 's/is/IS/g' -e '1,5p' -n bre  
abc IS a simple word.

What IS the answer?
I am 23 years old.
```

#### 7. 在文件第 4 行后追加一行 Linux is funny.. 

```bash
➜  ~/code/awk sed '4a ##Today is Wednesday##' bre   
abc is a simple word.

What is the answer?
I am 23 years old.
Linux is funny.. 

My tel is 12345678999
My favirate food is hamburger.
I come from JiangSu...


#这是一个注释行
xxxx
xxxxxxx
```

#### 8. 在文件第 1 行后追加一行 ##Today is Wednesday## 

```bash
➜  ~/code/awk sed '1i ##Today is Wednesday##' bre
##Today is Wednesday##
abc is a simple word.

What is the answer?
I am 23 years old.

My tel is 12345678999
My favirate food is hamburger.
I come from JiangSu...


#这是一个注释行
xxxx
xxxxxxx
```

加上 `-i` 参数将会直接修改文件，且不会输出。

若是不加地址，则表示全文范围，如要在每一行后面加上 `------------`，如下：

```bash
➜  ~/code/awk sed 'a ---------------' bre
abc is a simple word.
---------------

---------------
What is the answer?
---------------
I am 23 years old.
---------------

---------------
My tel is 12345678999
---------------
My favirate food is hamburger.
---------------
I come from JiangSu...
---------------

---------------

---------------
#这是一个注释行
---------------
xxxx
---------------
xxxxxxx
---------------
```

### 4.3 sed 进阶例子：取出 Linux 的 IP 地址

以下例子将用 `sed` 配合正则表达式进行处理文本。

首先输入 `ifconfig` 命令打印出网络信息，我们要提取出 `eth0` 网卡中的 `inet` 后的 IP 地址信息，然后保存到文件 `IP_eth0` 中。

我们首先打印 `eth0` 网卡信息：

```bash
➜  ~/code/awk ifconfig eth0              
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.19.194.137  netmask 255.255.240.0  broadcast 172.19.207.255
        inet6 fe80::215:5dff:feeb:1353  prefixlen 64  scopeid 0x20<link>
        ether 00:15:5d:eb:13:53  txqueuelen 1000  (Ethernet)
        RX packets 127  bytes 60337 (60.3 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 109  bytes 13475 (13.4 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

然后打印出第二行：

```bash
➜  ~/code/awk ifconfig eth0 | sed '2p' -n 
        inet 172.19.194.137  netmask 255.255.240.0  broadcast 172.19.207.255
```

然后用正则匹配将 IP 地址前后内容替换为空：

```bash
➜  ~/code/awk ifconfig eth0 | sed '2p' -n | sed 's/^.*inet //' | sed 's/ netmask.*$//'
172.19.194.137
```

这样便提取出了 IP 地址，然后再写入文件：

```bash
➜  ~/code/awk ifconfig eth0 | sed '2p' -n | sed 's/^.*inet //' | sed 's/ netmask.*$//' | tee IP_eth0
172.19.194.137
```

当然也可以用 `-e` 参数进行多次编辑：

```bash
➜  ~/code/awk ifconfig eth0 | sed -e '2s/^.*inet //' -e  '2s/ netmask.*$//p' -n | tee IP_eth0 
172.19.194.137
```

注意哪些地方有 `p` 参数，哪些地方没有，如果两次匹配加了 `p`，那么就会打印两次信息，一次是删掉前面之后的剩余部分，一次是删掉前后剩下的部分。

## 五、awk

awk 有强大的文本格式化功能，它更像是一门编程语言，支持条件判断、数组、循环等功能。

awk 语法如下：

```bash
awk [option] 'pattern[action]' file
     可选参数  模式     动作      文件/数据
```

action 指的是动作，awk 擅长文本格式化，且输出格式化之后的结果，因此最常见的动作就是 `pirnt` 和 `printf`。

若我们执行的命令是 `awk '{print $2}'`，没有使用参数和模式，`$2` 表示输出文本的第二列信息。`awk` 默认以空格为分隔符，且多个空格也识别为一个空格，作为分隔符。

`awk` 是按行处理文件，一行处理完毕，处理下一行，根据用户指定的分割符去工作，没有指定则默认空格。

`$0` 表示整行信息，`$1` 表示文本第一列信息，`$NF` 表示当前分割后的最后一列，倒数第二列可以写成 `$(NF-1)`。


### 1. awk 变量

`awk` 分为内置变量和自定义变量。

#### 1.1 内置变量

一些常用的内置变量如下表所示。

| 内置变量                    | 解释                                             |
| --------------------------- | ------------------------------------------------ |
| $n                          | 指定分隔符后，当前记录的第n个字段                |
| $0                          | 完整的输入记录                                   |
| FS                          | 输入字段分隔符，默认是空格                       |
| OFS                         | 输出字段分隔符，默认是空格                       |
| RS                          | 输入记录分隔符（输入换行符），指定输入时的换行符 |
| ORS                         | 输出记录分隔符（输入换行符），指定输入时的换行符 |
| NF(Number of fields)        | 分割后，当前行一共有多少个字段                   |
| NR(Number of records)       | 当前记录数，行数                                 |
| FNR                         | 各文件分别计数的行号                             |
| FILENAME                    | 当前文件名                                       |
| ARGC                        | 命令行参数的个数                                 |
| ARGV                        | 数组，存储命令行所给定的各个参数                 |  |
| 更多内置变量可以man手册查看 | man awk                                          |

例子，一次性输出多列信息：

```bash
awk '{print $1,$2}' file.txt
```

其中参数之间的逗号表示默认分隔符，在输出时每列之间会有一个空格。

再如，自定义输出内容：

```bash
awk '{print "第一列",$1,"第二列",$2}' file.txt
```

【注】：awk 必须外层单引号，内层双引号，并且内置变量 `$n` 不可以加引号，否则会被识别为字符串。

awk 的内置变量 `NR`、`NF` 是不用加 `$` 的，但是例如 `$0`、`$1` 等是需要加 `$` 的。

##### NR 和 NF

有如下测试文件 `awk1`：

```bash
Cherry 25 19990101
James 35 19850422
Anna 22 20000322
ChrisTim 48 19731225
Qiang 23 19990103
Sue 6 20160505
Paul 37 19850428
Steve 0 00000000
```

现在要输出第二行内容，命令如下：

```bash
➜  ~/code/awk awk 'NR==2' awk1 
James 35 19850422
```

注意这里是 `==`。如果要输出第二到第五行，则命令如下：

```bash
➜  ~/code/awk awk 'NR==2,NR==5' awk1
James 35 19850422
Anna 22 20000322
ChrisTim 48 19731225
Qiang 23 19990103
```

给每一行内容添加行号：

```bash
➜  ~/code/awk awk '{print NR,$0}' awk1
1 Cherry 25 19990101
2 James 35 19850422
3 Anna 22 20000322
4 ChrisTim 48 19731225
5 Qiang 23 19990103
6 Sue 6 20160505
7 Paul 37 19850428
8 Steve 0 00000000
```

输出第一列和最后一列：

```bash
➜  ~/code/awk awk '{print $1,$(NF)}' awk1
Cherry 19990101
James 19850422
Anna 20000322
ChrisTim 19731225
Qiang 19990103
Sue 20160505
Paul 19850428
Steve 00000000
```

##### 处理多个文件显示行号

若我们想对多个文件使用 `awk` 进行处理并输出每一行的行号，`awk` 会将所有文件放在一起显示行号，如下：

```bash
➜  ~/code/awk awk -v FS=':' '{print NR,$1}' passwd awk1 
1 root
2 daemon
3 bin
4 sys
5 sync
6 games
7 root
8 lp
9 root
10 news
11 Cherry 25 19990101
12 James 35 19850422
13 Anna 22 20000322
14 ChrisTim 48 19731225
15 Qiang 23 19990103
16 Sue 6 20160505
17 Paul 37 19850428
18 Steve 0 00000000
```

但是我们希望对每个文件分别打印行号，则可以将变量 `NR` 改为 `FNR`，如下：

```bash
➜  ~/code/awk awk -v FS=':' '{print FNR,$1}' passwd awk1
1 root
2 daemon
3 bin
4 sys
5 sync
6 games
7 root
8 lp
9 root
10 news
1 Cherry 25 19990101
2 James 35 19850422
3 Anna 22 20000322
4 ChrisTim 48 19731225
5 Qiang 23 19990103
6 Sue 6 20160505
7 Paul 37 19850428
8 Steve 0 00000000
```

##### RS 和 ORS

我们看这个文件内容：

```bash
➜  ~/code/awk awk '{print NR,$1}' awk1 
1 Cherry
2 James
3 Anna
4 ChrisTim
5 Qiang
6 Sue
7 Paul
8 Steve
```

这个文件一共有 8 行，默认是以换行符为每一行的分隔，若我们要修改其默认的输入换行符，则可以修改参数 `RS`。

```bash
➜  ~/code/awk awk -v RS=' ' '{print NR,$0}' awk1
1 Cherry
2 25
3 19990101
James
4 35
5 19850422
Anna
6 22
7 20000322
ChrisTim
8 48
9 19731225
Qiang
10 23
11 19990103
Sue
12 6
13 20160505
Paul
14 37
15 19850428
Steve
16 0
17 00000000
```

我们看到最终将以空格进行每一行的分隔，而不是换行符。我们还可以修改 `ORS` 修改默认的输出分隔符。

```bash
➜  ~/code/awk awk -v ORS=' ' '{print NR,$1}' awk1
1 Cherry 2 James 3 Anna 4 ChrisTim 5 Qiang 6 Sue 7 Paul 8 Steve
```

##### FILENAME

该变量显示正在处理文件的名字

```bash
➜  ~/code/awk awk '{print FILENAME,$1}' awk1
awk1 Cherry
awk1 James
awk1 Anna
awk1 ChrisTim
awk1 Qiang
awk1 Sue
awk1 Paul
awk1 Steve
```

##### 变量 ARGV、ARGC

```bash
➜  ~/code/awk awk 'BEGIN{print "AWK Start!"} {print "一共有",ARGC,"个参数"}' awk1
AWK Start!
一共有 2 个参数
一共有 2 个参数
一共有 2 个参数
一共有 2 个参数
一共有 2 个参数
一共有 2 个参数
一共有 2 个参数
一共有 2 个参数

➜  ~/code/awk awk 'BEGIN{print "AWK Start!"} {print "第一个awk参数为:",ARGV[0]}' awk1
AWK Start!
第一个awk参数为: awk
第一个awk参数为: awk
第一个awk参数为: awk
第一个awk参数为: awk
第一个awk参数为: awk
第一个awk参数为: awk
第一个awk参数为: awk
第一个awk参数为: awk

➜  ~/code/awk awk 'BEGIN{print "AWK Start!"} {print "第一个awk参数为:",ARGV[1]}' awk1
AWK Start!
第一个awk参数为: awk1
第一个awk参数为: awk1
第一个awk参数为: awk1
第一个awk参数为: awk1
第一个awk参数为: awk1
第一个awk参数为: awk1
第一个awk参数为: awk1
第一个awk参数为: awk1
```

我们发现该命令一共有 2 个参数，分别输出为 `awk` 和 `awk1`。

#### 1.2 自定义变量


















### 2. awk 参数

| 参数 | 解释                        |
| ---- | --------------------------- |
| -F   | 指定分割字段符              |
| -v   | 定义或修改一个awk内部的变量 |
| -f   | 从脚本文件中读取awk命令     |

更改分隔符见下一节内容。

### 3. awk 分隔符

在 `awk` 中，默认是以空格进行分隔的，在 `sed` 命令中，我们要获取网卡 `eth0` 的 IP 地址信息，采用正则表达式将前后部分删掉来实现。但是我们发现 `ifconfig` 信息都是用空格进行分隔的，因此在 `awk` 中，可以利用其特点，直接取出对应行的对应列，从而直接取出 IP 地址。

```bash
➜  ~/code/awk ifconfig eth0 | awk 'NR==2{print $2}'
192.168.202.82
```

`awk` 的分隔符有两种
- 一种是输入分隔符，默认是空格，叫 `field separator`，变量名为 `FS`
- 一种是输出分隔符，`output field separator`，简称 `OFS`

#### 3.1 FS 输入分隔符

`awk` 逐行处理文本的时候，以输入分割符为准，把文本切成多个片段，默认符号是空格。

例如下面的 `passwd` 文件，输出其第一列和最后一列：

```bash
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
root:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
root:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
```

```bash
➜  ~/code/awk awk -F ':' '{print $1,$NF}' passwd 
root /bin/bash
daemon /usr/sbin/nologin
bin /usr/sbin/nologin
sys /usr/sbin/nologin
sync /bin/sync
games /usr/sbin/nologin
root /usr/sbin/nologin
lp /usr/sbin/nologin
root /usr/sbin/nologin
news /usr/sbin/nologin
```

这里使用了 `-F` 变量修改了默认分隔符，除了使用这种方法之外，还可以使用变量的形式，指定分隔符，使用 `-v` 选项搭配，修改 `FS` 变量：

```bash
➜  ~/code/awk awk -v FS=':' '{print $1,$NF}' passwd  
root /bin/bash
daemon /usr/sbin/nologin
bin /usr/sbin/nologin
sys /usr/sbin/nologin
sync /bin/sync
games /usr/sbin/nologin
root /usr/sbin/nologin
lp /usr/sbin/nologin
root /usr/sbin/nologin
news /usr/sbin/nologin
```

#### 3.2 输出分隔符

上面的例子我们发现，第一列和最后一列之间默认输出是用空格分隔，若我们想自定义该输出分隔符，可以更改 `OFS` 变量修改默认输出分隔符，如下：

```bash
➜  ~/code/awk awk -v FS=':' -v OFS='==>' '{print $1,$NF}' passwd 
root==>/bin/bash
daemon==>/usr/sbin/nologin
bin==>/usr/sbin/nologin
sys==>/usr/sbin/nologin
sync==>/bin/sync
games==>/usr/sbin/nologin
root==>/usr/sbin/nologin
lp==>/usr/sbin/nologin
root==>/usr/sbin/nologin
news==>/usr/sbin/nologin
```

这样就修改了默认的输出分隔符。

### 









