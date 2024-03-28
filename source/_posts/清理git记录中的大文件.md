---
title: 清理git记录中的大文件
mathjax: true
tags:
  - git
  - Linux
categories:
  - 生产力工具
  - git
abbrlink: f6c73689
date: 2024-03-27 15:26:55
---

前几天我想把一个本地仓库推送到 github 仓库中时提示：

```shell
remote: error: Trace: 9222dcf96227a19766567c1da4183cfef7b9ec21689a2ee21f6c1c3b2f1fab7a
remote: error: See https://gh.io/lfs for more information.
remote: error: File 18T.trace is 771.88 MB; this exceeds GitHub's file size limit of 100.00 MB
remote: error: GH001: Large files detected. You may want to try Git Large File Storage - https://git-lfs.github.com.
To https://github.com/CherryYang05/DiskPine.git
 ! [remote rejected] master -> master (pre-receive hook declined)
错误：无法推送一些引用到 'https://github.com/CherryYang05/DiskPine.git'
```

发现在之前的 git 记录中有一个大文件（18T.trace 771.88MB），但是在最新的提交记录中，我已经将其删除了，却依然无法推送。

经过查询资料，找到了解决方法：

1. 若文件还没有删除，并且需要将其推送到远程仓库，使用 LFS(Large File Storage) 修改文件大小的限制；
2. 若文件已经删除，使用 BFG Repo-Cleaner 清理 git 历史记录中对大文件的引用。

<!-- more -->

## 1. LFS

Git LFS 是一个命令行扩展，用于使用 Git 管理大文件。git-lfs [Github 链接](https://github.com/git-lfs/git-lfs)

### 1.1 安装

```shell
curl -s https://packagecloud.io/install/repositories/github/git-lfs/script.rpm.sh | sudo bash
```

### 1.2 使用

1. 初始化 lfs：

```shell
git lfs install
```

2. 添加需要上传的文件：

```shell
git lfs track "your_big_file"
```

3. 将文件添加到暂存区：

```shell
git add "your_big_file"
```

4. 提交文件：

```shell
git commit -m "Add big file"
```

5. 推送到远程仓库：

```shell
git push
```

### 1.3 注意

- LFS 仅支持新的提交记录，无法修改历史记录中的引用；
- 若需要清理历史记录中的引用，需要使用 BFG Repo-Cleaner。

其他使用方式见 [Example Usage](https://github.com/git-lfs/git-lfs?tab=readme-ov-file#example-usage)。

## 2. BFG

BFG Repo-Cleaner 是一个由 Scala 编写的命令行工具，用于快速、精确地从 Git 仓库中删除不需要的数据。相比于 git-filter-branch 命令，BFG 的速度更快，使用也更简单。无论是大文件还是敏感数据，只需简单的命令，就能将其从所有提交历史中移除。

### 2.1 安装

在 [这里](https://rtyley.github.io/bfg-repo-cleaner/) 下载 bfg-1.14.0.jar 包，或者使用 curl 下载：

```shell
curl -o bfg.jar -L https://repo1.maven.org/maven2/com/madgag/bfg/1.14.0/bfg-1.14.0.jar
```

### 2.2 使用

1. 首先，将要清理的文件添加到 `.gitignore` 文件中，以免再次出现在提交中；
2. 然后，使用 BFG 清理大文件：

```shell
java -jar bfg.jar --strip-blobs-bigger-than 100M <repo>.git
```

其中，`--strip-blobs-bigger-than 100M` 表示删除大于 100M 的文件。

3. 接着，清理 git 历史记录中的引用：

```shell
cd <repo>
git reflog expire --expire=now --all && git gc --prune=now --aggressive
```

4. 最后，推送到远程仓库：

```shell
git push
```

### 2.3 注意

- BFG Repo-Cleaner 只会修改本地仓库，不会修改远程仓库；
- BFG Repo-Cleaner 会修改 commit 的 hash 值，因此不要使用 `git pull` 同步远程仓库，而是使用 `git fetch`；
- BFG Repo-Cleaner 会修改提交记录，因此可能会导致冲突，需要手动解决。

### 2.4 参考链接

1. [BFG Repo-Cleaner](https://rtyley.github.io/bfg-repo-cleaner/)
2. [BFG Repo-Cleaner - GitHub](

### 2.1 安装

jar 包[下载链接](https://rtyley.github.io/bfg-repo-cleaner/)


