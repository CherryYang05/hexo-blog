---
title: Linux常用命令
mathjax: true
tags:
  - Linux命令
categories:
  - Linux
abbrlink: d0edc1ed
date: 2021-03-14 17:12:19
---

# Linux常用的命令总结

## 1. du

查看文件大小
```shell
du -sh
``` 

`du` 是 `disk usage` 磁盘使用的缩写，`-s` 是查看总用量，`-h` 是以 `M` 为单位

![du](d0edc1ed/du%20-sh.png)

## 2. tail

输出文件的最后若干行内容
```shell
ls -l / | tail -n2
```

表示查看根目录列表的最后两行信息

## 3. 管道

```shell
stdin | stdout
```
管道的表示是一条竖杠 `|`，将竖杠的左边的输出作为右边的输入，**他们是两个程序**。

例如
```shell
curl --head www.baidu.com | grep -i content-length
```
就是将请求百度的 HTTP 报头，传送到 `grep` 程序中，计算字符个数，最终显示：

```shell
Content-Length: 277
```

再例如
```shell
curl --head www.baidu.com | grep -i content-length | cut --delimiter=' ' -f2
```
表示将上述结果以空格进行切分，输出第二个参数，显然最终应该输出

```shell
277
```

## 4. tee

```shell
echo "welcome ysyx" | tee ysyx.txt
```
从标准输入设备读取数据，将其内容输出到标准输出设备，同时保存成文件。
上述命令将字符串 `welcome ysyx` 输入到 `ysyx.txt` 文件中，并在命令行中输出该字符串

参数：
`-a`： append，not overwrite

## 5. xdg-open

```shell
xdg-open {file | URL}
```
用用户默认的程序打开对应的文件和链接，比如链接就用网页打开，txt文件就用文本编辑器打开等。

## 6. find
```shell
sudo find /sys -name capacity -exec cat {} \;
```
`\`表转义，在不同的脚本中 `;` 有不同的含义，因此要转义


## 7. shell脚本

### 创建自定义的命令：
```shell
vim mcd.sh
```
```bash
mcd() {
    mkdir -p "$1"
    cd $1
}
```
```shell
source mcd.sh
mcd a/b/c
```
- $0 - 脚本名
- \$1 到 \$9 - 脚本的参数。$1 是第一个参数，依此类推
- $@ - 所有参数
- $# - 参数个数
- $? - 前一个命令的返回值
- $$ - 当前脚本的进程识别码
- !! - 完整的上一条命令，包括参数。常见应用：当你因为权限不足执行命令失败时，可以使用 sudo !!再尝试一次。
- $_ - 上一条命令的最后一个参数。如果你正在使用的是交互式shell，你可以通过按下 Esc 之后键入 . 来获取这个值。

命令通常使用 STDOUT来返回输出值，使用STDERR 来返回错误及错误码，便于脚本以更加友好的方式报告错误。 返回码或退出状态是脚本/命令之间交流执行状态的方式。返回值0表示正常执行，其他所有非0的返回值都表示有错误发生

### 使用 `!!` 表明复用上一个执行的命令
```shell
mkdir /sys/new
sudo !!
```

### <(CMD)

进程替换（process substitution）， `<(CMD)` 会执行 CMD 并将结果输出到一个临时文件中，并将 `<(CMD)` 替换成临时文件名。这在我们希望返回值通过文件而不是STDIN传递时很有用。例如， `diff <(ls foo) <(ls bar)` 会显示文件夹 `foo` 和 `bar` 中文件的区别。

### bash脚本编写
```shell
#!/bin/bash

echo "Starting program at $(date)" # date会被替换成日期和时间

echo "Running program $0 with $# arguments with pid $$"

for file in "$@"; do
    grep foobar "$file" > /dev/null 2> /dev/null
    # 如果模式没有找到，则grep退出状态为1
    # 我们将标准输出流和标准错误流重定向到Null，因为我们并不关心这些信息
    if [[ $? -ne 0 ]]; then
        echo "File $file does not have any foobar, adding one"
        echo "# foobar" >> "$file"
    fi
done
```

### shellcheck
shell脚本分析工具

### htop
能够可视化查看 CPU 占用和内存占用情况，并显示所有进程信息

## 8. alias

```shell
alias ll="ls -alh"
```

该命令将 `ll` 映射为 `ls -alh`

注意后面的命令 `=` 前后没有空格，因为它只接收一个参数，加上空格便会解析成多个参数

但是，但是这样的映射在 `shell` 关闭之后将失效，要想让他们每次都生效，将命令写入 `~/.bashrc` 中。

例 `.bashrc`:
```shell
alias ll="ls -alh"  //列表显示所有文件，大小以单位显示
alias mv="mv -i"    //让 mv 操作有提示信息
PS1="> "            //让 bash 命令行以 "> "开头
```
## 9. rsync
```shell
rsync -avP . name@ip:folder
```

`rsync` 支持断点下载，并保留原文件的权限，是一种比 `scp` 更有效的拷贝文件的方式

