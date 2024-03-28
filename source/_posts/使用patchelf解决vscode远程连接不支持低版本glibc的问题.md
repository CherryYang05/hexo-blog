---
title: 使用patchelf解决vscode远程连接不支持低版本glibc的问题
mathjax: true
tags:
  - glibc
  - Linux
categories:
  - Linux
abbrlink: 2cbef437
date: 2024-03-27 16:52:56
---

VScode 1.86 版本的 remote 要求 glibc 2.28 及以上，于是在各种旧版本服务器上就不支持了。

![不支持的 OS 版本](https://raw.githubusercontent.com/CherryYang05/PicGoImage/master/images/image.png)

新版本刚发布就在 github 上的 [issue](https://github.com/microsoft/vscode/issues/203375) 上讨论起来了，[VScode 官方文档](https://code.visualstudio.com/docs/remote/faq#_can-i-run-vs-code-server-on-older-linux-distributions) 中也说明了，从 VS Code 版本 1.86.1（2024 年 1 月）开始提高了远程服务器构建工具链的最低要求。VS Code 分发的预构建服务器与基于 glibc 2.28 或更高版本的 Linux 发行版兼容。

当然给服务器升级 glibc 是一个极其危险的操作，本人曾经就瞎捣鼓升级了 glibc，结果导致系统崩溃，不得已重装了系统。

**我们可以利用 patchelf 手动指定动态库，避免了重新编译系统的 glibc。**

<!-- more -->

## 1. 动态链接库下载

github 上有较为方便的下载 glibc 的仓库 [glibc-all-in-one](https://github.com/matrix1001/glibc-all-in-one)

根据仓库的 README，查看支持的版本：`cat list`

我选择的是 `2.31-0ubuntu9.14_amd64`，然后执行 `./download 2.31-0ubuntu9.14_amd64`

在当前文件夹下会生成 `libs` 文件夹，就是刚刚下载的 `2.31-0ubuntu9.14_amd64` 的动态库。

然后执行 `./build 2.31 arm64`，这一步会在根目录下编译生成 `/glibc` 文件夹，可以将其移动到 `glibc-all-in-one` 文件夹中。

## 2. 用 patchelf 修改 vscode-server 依赖的 glibc 版本

在执行命令前，先删除 `.vscode-server` 文件夹，用 vscode 连接服务器，让它自动重新下载 vscode-server 相关文件，这个时候在 `~/.vscode-server/bin` 中应该只有一个由数字和字母组成随机字符串的文件夹（我的服务器上是 `863d2581ecda6849923a2118d93a088b0745d9d6`，不同人应该不一样），进入这个文件夹，有一个 `node` 二进制文件，我们要重新 patch 的就是这个文件。

执行命令：

`patchelf --set-interpreter ~/pack/glibc-all-in-one/libs/2.31-0ubuntu9.14_amd64/ld-linux-x86-64.so.2 --set-rpath ~/pack/glibc-all-in-one/libs/2.31-0ubuntu9.14_amd64/:~/pack/glibc-all-in-one/glibc/2.31/amd64/lib --force-rpath ~/.vscode-server/bin/863d2581ecda6849923a2118d93a088b0745d9d6/node`

`--set-interpreter` 后面跟的是可执行文件的解释器路径，需要指定动态链接器的路径，路径为 `./glibc-all-in-one/libs/[your downloaded glibc version]/ld-linux-x86-64.so.2`，动态链接器负责在程序运行时加载所需的共享库。

`--set-rpath` 这部分设置了可执行文件的运行时搜索路径（Runtime PATH），指定了程序在运行时搜索共享库时应该查找的路径。这里指定了两个路径包含了 glibc 核心库和其他库。这里的路径为 `glibc-all-in-one/libs` 文件夹下的你下载的不同版本的 glibc 文件夹和编译生成的 `glibc/[version]/[arch]/lib` 文件夹。

`--force-rpath` 后面跟的就是要修改的 vscode-server 的 node 文件。

如果执行命令时提示：`patchelf: open: Text file busy`，将本机上 vscode 运行的远程连接关掉再执行就可以了。

重新打开 vscode，就不会有操作系统版本不支持的提示了~~

[参考链接](https://zhangjk98.xyz/patchelf/)