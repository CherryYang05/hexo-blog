---
title: Linux服务器配置
mathjax: true
tags:
  - Linux服务器
categories:
  - Linux
abbrlink: 3472be8b
date: 2020-05-26 22:10:55
---
### 实验内容介绍

Linux 操作系统在服务器领域具有广泛的应用。Web 服务是服务器领域中应用最广泛的服务，常见 Web 服务有 ``WAMP(Windows+Apache+MySQL+PHP)`` 和 ``LAMP(Linux+Apache+MySQL+PHP)``架构，其中 Apache 是全世界最流行的 Web 服务软件，此外，Web 服务软件 nginx，因其具有优秀的性能也受到越来越多的用户欢迎。本实验中，演示在 Linux 环境下搭建支持 PHP 等网页的 Web 服务平台，即LAMP。
Linux 环境下搭建 Web 服务器有三种方式，一是在安装操作系统时勾选相应服务组件；二是从网上下载或从 ISO 系统镜像包中拷贝安装包手动安装；三是在在连接网络的的情况下在线安装。
本实验演示以 kali 64 位操作系统为例。因 kali 系统已经自带 Apache、MySQL 和 PHP，为了解安装流程，请先卸载系统自带软件。

<!-- more -->

### 卸载系统原有Apache、MySQL、PHP

sudo apt-get remove apache2
sudo apt-get remove mysql-*
sudo apt-get remove php

### 在线安装Apache 服务器

输入命令：``sudo apt-get install apache2``

![在线安装Apache服务器](3472be8b/在线安装Apache服务器.png)

输入命令：``sudo /etc/init.d/apache2 start`` 手动启动服务

![启动Apache服务](3472be8b/启动Apache服务.png)

以上页面详细介绍了 Debian 发行版 Linux 中 Apache 基本信息，其中：

**Apache 根目录：**
``/var/www/html``

**Apache 配置目录和文件：**
``/etc/apache2``: Apache 主配置目录， Apache 所有配置文件均在此目录下；
``/etc/apache2/apache2.conf``: 主配置文件，可配置 Apache 全局配置；
``/etc/apache2/ports.conf``: 端口配置文件。默认情况下，当启用提供 SSL功能的模块时，Apache 监听端口 80，并在端口 443 上进行监听。
其它文件略。

**Apache 日志文件：**
``/var/log/apache2/access.log``: 服务请求日志
``/var/log/apache2/error.log``: 服务错误日志

**Apache 基本操作：**
服务启动：``/etc/init.d/apache2 start``
服务停止：``/etc/init.d/apache2 stop``
服务重启：``/etc/init.d/apache2 restart``
部分 Linux 安装 Apache 后可能出现服务正常运行，相应端口正常打开，但其它主机无法访问情况，此现象一般是因为系统防火墙未开放相应服务端口。在浏览器找那个输入虚拟机的IP，看到：

![Apache服务启动成功](3472be8b/Apache服务启动成功.png)

说明服务器配置成功！
### 安装 MySQL/MariaDB
本示例采用 MariaDB 代替 MySQL。MariaDB 是 MySQL 的一个分支，现由开源社区维护，采用 GPL 授权许可，其大部分语法与 MySQL 都相同。因 MySQL 被 Oracle收购后有闭源的风险，大部分 Linux 组织均从其套件清单删除了 MySQL，并以MariaDB 代替 MySQL，若一定要使用 MySQL，可通过下载安装包的方式手动安装MySQL。

安装 MariaDB 服务：
``sudo apt-get install mariadb-server``
``sudo apt-get install mariadb-client``

![安装mariadb](3472be8b/安装mariadb.png)

![安装mariadb2](3472be8b/安装mariadb2.png)

#### 遇到的问题：
##### 首先就是提示没有``mariadb-server``软件包，类似于这样

![提示没有mariadb软件包](3472be8b/提示没有mariadb软件包.png)

这个可能是apt版本不够，需要执行``sudo apt-get update`` 进行更新，但是更新实在太慢，我更新了33分钟才更新好，这时候已经快下课了。

![apt-get-update](3472be8b/apt-get-update.png)

##### 然后就是出现域名无法解析的错误
这个错误困扰了我好久，按照网上的方法试了很多，但是还是不行。最后发现是``/etc/network/interfaces``文件中网关写错了，写成了``192.168.1.0``，正确的应该是``192.168.1.1``
然后修改``/etc/resolv.conf``文件，增加字段：``nameserver 8.8.8.8（好像是谷歌的DNS服务器）``，然后执行``ifdown -a``关闭网卡，``ifup -a``启动网卡，然后就可以了。

至于为什么会出现DNS解析错误，一开始我的虚拟机的网络是用的NAT（虚拟地址转换），在NAT模式中，主机网卡直接与虚拟NAT设备相连，然后虚拟NAT设备与虚拟DHCP服务器一起连接在虚拟交换机VMnet8上，这样就实现了虚拟机联网。在NAT模式下，宿主计算机相当于一台开启了DHCP功能的路由器，而虚拟机则是内网中的一台真实主机，通过路由器(宿主计算机)DHCP动态获得网络参数。因此在NAT模式下，虚拟机可以访问外部网络，反之则不行，因为虚拟机属于内网。

而改成了桥接模式后，虚拟机和宿主计算机处于同等地位（同处一个局域网），虚拟机就像是一台真实主机一样存在于局域网中。因此在桥接模式下，我们就要像对待其他真实计算机一样为其配置IP、网关、子网掩码等等。
但是修改``/etc/resolv.conf``，重启过后就没有了，我们需要安装``resolvconfig``应用组件：``sudo apt-get install resolvconf``，在``/etc/resolvconf/resolv.conf.d/base``文件中添加DNS信息``（nameserver 8.8.8.8）``，就可以了。

然后再次重新启动，刚刚的问题解决！

![resolvconfig应用](3472be8b/resolvconfig应用.png)

![apt-get更新成功](3472be8b/apt-get更新成功.png)

**配置 Mariadb 的安全选项:**
``sudo mysql_secure_installation``

此时系统会提示输入数据库 root 用户密码，因系统并未设置相应密码，此时直接按提示回车即可。若提示错误，则 sudo /etc/init.d/mysql restart 命令重启 mysql 服务并重新执行安全选项命令。随后数据库会提示以下安全设置信息，一般设置数据库 root 用户密码即可，其它选项按回车选择默认：

1. Enter current password for root (enter for none): 输入当前 root的密码(因新数据库无密码，回车即可)；
2. Set root password? [Y/n] 回车，默认为输入 Y；
3. New password: 输入新密码；
4. Re-enter new password 确认密码；
5. Remove anonymous users? [Y/n] 移除匿名用户；
6. Disallow root login remotely? [Y/n] 禁止 root 远程登录；
7. Remove test database and access to it? [Y/n] 移除测试数据库；
8. Reload privilege tables now? [Y/n] 重新加载权限表。


**测试数据库:**
如图所示，若进入数据库则表示数据库安装成功，`sudo mysql -u root -p`

![MySQL](3472be8b/MySQL.png)

### 安装PHP

安装 PHP 除了 PHP 应用程序外，还需安装 PHP 与 Apache、MySQL/MariaDB相关扩展包，扩展包需与软件对应，本例中安装 php7.2 版本，对应扩展包可通过以下命令模糊查询：
``sudo apt-cache search php7``
从查询结果可知，php7.3对应Apache、MySQL扩展包分别为 ：``libapahe2-mod-php7.2，php7.2-mysql``.如下图所示为安装 PHP 相关软件包。
``sudo apt-get install php7.2 libapahe2-mod-php7.2 php7.2-mysql
PHP`` 安装完毕后需重启 Apache

### 测试PHP页面

编辑测试文件，如下图所示，在 ``/var/www/html`` 目录下新建 ``test.php`` 文件，并输入如下所示的测试代码。

```php
<?php echo phpinfo();?>
```

在浏览器中访问如下：

![安装PHP](3472be8b/安装PHP.png)

**注意：我修改了Apache的端口为8080，因此访问时要在URL后面加上8080端口。**

### 创建文件上传页面

文件上传功能由上传文件的 HTML 表单和文件上传脚本构成。在根目录下创建``upload.html``文件，编辑表单

![upload.html](3472be8b/upload.html.png)

在根目录下创建 ``upload.php`` 脚本文件，编写文件上传功能代码：

![upload.php](3472be8b/upload.php.png)

在根目录下创建“upload”目录，用于保存上传的图片
然后访问``192.168.1.120:8080/upload.html``

![访问html](3472be8b/访问html.png)

选择一个不超过200K的图片文件：

![链接php](3472be8b/链接php.png)

先修改upload文件夹的权限，上传之后查看服务器中upload文件夹：

![upload文件夹](3472be8b/upload文件夹.png)

或者查看Apache的属主：``ps -ef | grep apache``，发现是``www-data``

![Apache属主](3472be8b/Apache属主.png)

然后更改修改目录的所有者：

![chown更改属主](3472be8b/chown更改属主.png)

结果和上述一样，大功告成！
