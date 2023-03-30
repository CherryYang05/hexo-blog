---
title: Git的使用
mathjax: true
tags:
  - Git
  - Linux
categories:
  - 生产力工具
abbrlink: 3645f6a9
date: 2022-03-09 22:26:24
---

# Git 的使用

<!-- more -->

## 1. 分支命令

```shell
git checkout -b <new-branch-name>   //新建分支并切换到该分支
git branch <new branch name>        //新建分支
git branch -D <branch name>         //删除分支
git branch -m oldName newName       //修改分支名称
git branch -vv                      //参数 `vv` 表示显示更多冗余信息

git rebase
```

## 2. git config 配置

```shell
git config [--global] user.name <"Your-Name">   //设置git的用户名和邮箱密码等
git config --global core.pager "less -FRSX"     //设置不要将结果输出到新窗口，并且进行less分页显示
```
`less` 的功能比 `more` 更多，前者可以上下滚屏，后者不可以。`core-pager` 默认参数为 `less`，会在新的窗口中输出信息，按 `q` 键退出后终端将不显示刚刚的信息，这样有点不太方便。或者是添加参数 `--no-pager`.


```shell
git log --all --graph --decorate --oneline
```
展示git的树形提交记录，`oneline` 表示展示的信息在一行中表示

```shell
git diff <commit1> <commit2> <file-name>
```
显示指定的两次 commit 中的 file-name 文件的不同，默认 commit 为最新 commit

## 3. remote操作

```shell
git init --bare

git remote add <name> <url>     // url可以是本地地址，也可以是远程仓库地址(例如github)
git remote add origin ../remote

git push <remote> <local branch>:<remote branch>
git push origin master:master

git clone <url> <folder name>
git clone ./gitDemo demo2

//创建本地和远端分支的关联关系
git branch --set-upstream-to <remote>/<remote branch>

git pull = git fetch; git merge
git pull --all                          // 拉取远程所有分支

git pull origin <远程分支名>:<本地分支名>   // 将远程指定分支拉取到本地指定分支上
git pull origin <远程分支名>              // 将远程指定分支拉取到本地当前分支上

git push origin <本地分支名>:<远程分支名>   // 将本地当前分支推送到远程指定分支上（注意：pull 是远程在前本地在后，push 相反）
git push origin <本地分支名>              // 将本地当前分支推送到与本地当前分支同名的远程分支上

git push -u origin <本地分支名>           // 将本地分支与远程同名分支相关联
```

注意远程仓库的初始化一定是“裸”的，即 `git init --bare`，因为远程仓库中不允许进行 git 操作，不包含工作区。

`git fetch` 是将远程主机的最新内容拉到本地，用户在检查了以后决定是否合并到工作本机分支中，而 `git pull` 是直接合并。

## 4. 其他

```shell
git add -p <filename>

git clone --shallow         //浅克隆，不会保留仓库原有的提交记录

git show <commit id>        //展示某次提交的具体内容(通过diff展示)

git stash                   //暂时移除工作目录下的修改内容
git stash pop               //移除隐藏
```

参数 `-p` 表示交互式暂存，可以自定义某些修改是否要提交


|命令|功能|
|---|---|
|git init                   |在本地的当前目录里初始化git仓库|
|git status                 |查看当前仓库的状态|
|git add -A                 |增加目录中所有的文件到缓存区|
|git add file               |增加相应文件到缓存区|
|git commit -m "msg"        |将缓存区中更改提交到本地仓库|
|git log                    |查看当前版本之前的提交记录|
|git reflog                 |查看HEAD的变更记录，包括回退|
|git branch -b branch_name  |建立一个新的分支|
|git diff                   |查看当前文件与缓存区文件的差异|
|git checkout --file        |取消更改，将缓存区的文件提取覆盖当前文件|
|git reset --hard ver_num   |回退到相应版本号，同样也可以回退到未来的版本号|
|git clean -xf              |删除当前目录中所有未追踪的文件|
|git config --global core.quotepath false|处理中文文件名|


## 5. git reset 命令及用 amend 提交合并

当我们提交一版代码后，发现有一个小问题需要重新修改一下，但是又不想重新再提交一次，因为这样会多一次无用的提交记录，不方便整体的管理和日志的查看。我们首先使用 `git reset` 进行回退。

`git reset` 命令有几个参数
- `--soft` 参数表示撤销到某次提交，保留**暂存区**（`add` 操作之后保存到的地方）里的内容，如果要再提交只要再执行 `commit` 即可
- `--mixed` 会撤销某次提交，但是工作目录下修改的内容不会被存到暂存区中，需要重新执行 `add` 命令才能存入暂存区
- `--hard` 会直接回退到某一提交，任何修改都不会被保存，这个参数要谨慎使用，很可能会造成本地文件丢失。

若是不小心使用 `git reset --hard` 误操作了可以先使用 `git reflog` 查看所有操作记录，找到要回退的版本号（ `HEAD` 指针或 `commitId`），然后指定具体的版本号进行回退即可。

另一种避免重复提交的方法就是 `git --amend` 命令，在第二次提交时，若我们想合并上一次提交，可以使用 `git commit --amend -m "xxx"`，这样新一次的提交会覆盖上一次的提交。

但是在使用 `--amend` 时候不能进行 `push` 提交，否则会提

```shell
提示：更新被拒绝，因为您当前分支的最新提交落后于其对应的远程分支。
提示：再次推送前，先与远程变更合并（如 'git pull ...'）。详见
提示：'git push --help' 中的 'Note about fast-forwards' 小节`
```

若 `push` 了之后应该使用 `--force-with-lease` 参数。

## 6. 

```shell
git ls-files                // 查看暂存区中的文件
git restore --staged        // 撤销在暂存区提交的文件
```









