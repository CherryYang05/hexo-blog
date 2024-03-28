---
title: 执行 yum 提示CRITICAL
mathjax: true
tags:
  - yum
  - 配置
  - Centos
categories:
  - - 运维
    - 配置
abbrlink: d74c62dc
date: 2024-03-21 16:52:56
---

# 执行 yum 命令提示 CRITICAL:yum.cli:Config Error: Error accessing file for config file:///etc/yum.conf

关于这个问题，网上可以找到很多解决方案，包括但不限于：

1. 重新安装 yum
2. 重新创建 /etc/yum.conf文件
3. 使用 rpm 卸载当前安装的包

但是最后仍然报同样的错误。

因为之前在更换 libc 版本，手动编译安装了 glibc，最后搜到这个[博客](https://www.silverdragon.cn/archives/7252/)，得知是 curl 出了问题，是 libcurl 版本过低，libcurl 是 c/cpp 使用时的链接库，常用的 http get，post 请求，http，ftp 下载等都支持

curl 的功能实现是依赖 libcurl.so 这个动态链接库的，通常位于 /lib64 或者 /usr/lib64 中，4.x 版本的链接库地址为 libcurl.so.4

最后发现 libcurl 版本为 4.3.0，可能不支持 glibc-2.30 高版本的运行时，因此重新下载了高版本的 libcurl，具体操作见[博客](https://www.silverdragon.cn/archives/7252/)
