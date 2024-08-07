---
title: TFTP简单文本传输协议的基本原理
mathjax: true
tags:
  - 计算机网络
  - TFTP协议
categories:
  - 计算机网络
  - TCP/IP协议
abbrlink: 2c942dc5
date: 2020-06-10 19:32:31
---

## TFTP简单文本传输协议的介绍

随着互联网发展，文件传输效率越来越快，相应的传输协议也越来越复杂。早年有很多文件传输协议如今已经很少再用，所谓老兵不死，只是慢慢凋零。这些协议尽管现在使用不多，但它们的设计思想依然值得我们好好研究和掌握。

例如 FTP 以及它的 UDP 版本 TFTP，它们实现文件传输的协议设计思想依然非常值得研究，它对我们设计新协议依然很有启发性。本节开始，我们研究 TFTP 协议的原理以及相关代码实现。

TFTP 原名叫``Trivial File Transport Protocol``.对互联网早期存有记忆的同学对 FTP 协议一定非常了解，当时局域网乃至整个网络上很多大文件，例如电影的传输依靠的就是该协议。FTP 协议运行在 TCP 协议之上，它的内容很复杂，除了文件传输外，它还支持很多文件相关操作，例如远程实现文件建立，删除等。TFTP 是 FTP 协议的简化版，它运行在 UDP 协议上，同时简化了很多 FTP 操作，只支持文件的传输功能。
<!-- more -->
TFTP协议是基于服务器和客户端之间的传输协议。一开始客户端向服务器发出连接请求，服务器应答后两者连接建立。然后客户端向服务器发出文件传输请求，服务器将客户端需要的文件分割成多个小块，依次传递给客户端，客户端每收到一个小块后向服务器发出应答，收到应答后服务器再发送下一个小块。当所有文件块传输完毕后，两者连接断开。

TFTP 服务器程序通常在端口 69 监听客户端请求。值得注意的是，当服务器与客户端进行数据块传输时，服务器会使用一个随机端口而不是用于监听请求的 69 端口，这是为了服务器能同时相应多个客户端的连接。服务器与不同客户端使用不同端口进行数据通信，这样就保证不同客户端所需要的数据块不会发生混淆。

服务器与客户端在发送文件数据时，会按照一种名为"锁定步骤"的方式进行数据传输。也就是服务器向客户端发送一个数据块，再接收到客户端发回的应答数据包前什么都不做，直到收到客户端确定数据块已经收到的应答后，它才发送下一个数据块，这种方式使得数据传输效率不高，但确保数据传输流程足够简单，同时能保证传输出错时，数据重传很方便，同时客户端也不用考虑数据块不按次序抵达时，如何将数据块进行正确组装。

TFTP 协议的简单附带的代价是效率不高。由于它走的是 UDP 协议，因此一次发送数据块不能超过 512 字节，这也是服务器必须把文件切成小块反复传输的原因。还有一点值得注意的是 TFTP 协议没有任何安全措施，它不需要注册或登录，任何客户端都可以连接然后下载文件。

我们首先在网络上下载一个 TFTP 的服务器客户端：[Tftpd64](http://tftpd32.jounin.net/tftpd32_download.html) 具体界面如下：

![tftpd64](2c942dc5/tftpd64.png)

然后在 Windows 中的设置的程序与功能中打开 TFTP Client:

![tftp_Windows](2c942dc5/tftp_Windows.png)

## TFTP抓包演示

我们在 TFTP 服务器客户端上选择我们的 IP 地址：192.168.1.101，然后这台电脑就可以作为我们的 TFTP 服务器了，然后我们在命令行中输入：``tftp 192.168.1.101 get test.txt``(test.txt 是我特地新建的用来测试的文件，该文件必须存放在 tftp 服务器客户端软件中 Current Directory 所显示的目录中)：

![tftp抓包1](2c942dc5/tftp抓包1.png)

![tftp抓包2](2c942dc5/tftp抓包2.png)

然后就可以在当前磁盘（G盘）中看到我们传送的文件了

需要注意的几个问题:

1. 文件传送成功与否，你朋友也可以在 Tftpd32 的 "Tftp Server" 和 "Current Action"这两项中看到。
2. 如果想把文件传给你朋友，那么只要把命令换成 "Tftp -i朋友IP put pictures.rar" 即可。关于Tftp命令的更多参数，你可以在 CMD 下输入 Tftp 进行查看。不过此时你朋友不能进行上传和下载工作，因为他此时是 Tftp 的服务端，只有客户端才能进行这些操作。如果他想把西传给你，那就需要你做服务端了。
3. 用 Tftp 传送文件时，服务端需有确定的公网 IP，如果你朋友在局域网中通过网关上网的话，那就无法传送了。当然，如果两个人在同一局域网中，内网的 IP 也可以传送文件,只是有些多此一举。
4. Windows 98 系统可以当服务端，但客户端一定要是 Windows 2000 或是 Windows XP 等有 Tftp 命令的系统。
以后如果你遇到因为防火墙等原因不能通过QQ传送文件时，不妨试试 Tftp.

![tftp抓包3](2c942dc5/tftp抓包3.png)

我们再通过 Wireshark 抓包来观察 TFTP 协议(由于在 Windows上自己传给自己抓不到包，因此我在 Linux虚拟机上进行测试)，但是出现了 ``Destination unreachable (Host administratively prohibited)``，查询资料得知，是 Linux 的防火墙没有设置，我们设置一下 iptables:

```bash
iptables -P OUTPUT ACCEPT       #允许所有本机向外的访问
iptables -P FORWARD ACCEPT      #允许所有转发
iptables -P INPUT ACCEPT        #允许所有本机向外的访问
iptables -F                     #清除所有规则
```

![tftp抓包4](2c942dc5/tftp抓包4.png)

![tftp抓包5](2c942dc5/tftp抓包5.png)

![tftp抓包6](2c942dc5/tftp抓包6.png)

【注】:我的电脑不知道为何只能抓到单向的数据包，无法抓到主机向虚拟机传送的数据包，查了很多资料之后未果，先接着学习吧。

抓包显示两种数据包，分别是 Acknowledgement 和 Data block，其中前者是客户端收到数据块后对服务器的应答，后者是服务器向客户端发送的数据块。数据包的具体格式我们会在后面进行详细分析。

## TFTP交互细节及数据包解读

tftp 主要分为三步，首先是连接，然后是数据传输，最后是连接中断。所有这些步骤都通过发送相关数据包完成。最开始由客户端发送一个数据读取或写入请求，这个请求发出的同时连接自动建立，在这个过程中双方会协议要传输什么格式的文件。TFTP协议支持两个格式文件的传输，分别是 ASCII 文本，另一种是二进制数据，FTP 协议支持的文本格式比 TFTP 要复杂得多。

如果客户端请求的文件存在，服务器会直接将第一个数据块发送给客户端。如果是客户端想上传文件，服务器会发送一个 ACK 数据包表示确认。在这个过程中如果出现错误，其中一方就向另一方发送错误信息数据包，然后文件传输终止。由于使用 UDP 作为底层协议，因此一次数据发送最大不超过 ``512`` 字节。因此为了保证数据顺序正确性，每个数据包必须对应相应编号，编号根据数据块的顺序从1开始。

由于每次数据块最大是 512 字节，只要文件传输没有结束，那么一次数据块就是 512 字节，如果有数据包中数据少于 512 字节，那意味着这是文件最后一个数据包，最后一个数据块发送完后，连接自动中断。我们通过一个具体实例来掌握数据发送流程，假设客户端想从服务器读取一个 1200 字节的文件，以下是相关步骤：

1. 客户端发送一个数据包给服务器，其中包含了要读取的文件名。
2. 服务器发回第一个 512 字节数据块，并对其标号为 1.
3. 客户端返回服务器一个标号为 1 的确认数据包
4. 服务器发送标号为 2 包含 512 字节的数据块
5. 客户端收到 2 号数据块后发生确认数据包
6. 服务器发送标号为 3 的包含 176 字节的数据块
7. 客户端收到后回发标号为 3 的确认数据包
8. 服务器收到确认数据包后，确认文件发送完毕

![传输示例](2c942dc5/传输示例.png)

我们再看看客户端上传文件的流程：

1. 客户端发送一个写请求数据包，里面包含了要写的文件名称
2. 服务器发送确认数据包，在数据包中它使用编号 0
3. 客户端发送一个含有 512 字节，编号为1的数据包
4. 服务器返回编号为 1 的确认数据包
5. 客户端发送编号为 2，包含 512 字节的数据包
6. 服务器返回编号为 2 的确认数据包
7. 客户端发送编号为 3，包含 176 字节的数据包，等待服务器返回确认数据包。
8. 服务器接受 3 号数据包后，返回确认数据包，由于该数据包数据少于 512 字节，服务器知道是最后一个数据包。
9. 客户端收到 3 号确认数据包后，知道文件传输完毕，中断连接。

![传输示例2](2c942dc5/传输示例2.png)

TFTP 协议后来又经过一次扩展，增加一些控制命令。如果客户端或服务器想使用扩展命令时，它必须向对方确认是否也能支持相应命令。它会发送一个数据包，里面包含扩展命令对应的数值列表，对方也会返回一个列表，把它支持的扩展命令对应的数值放在列表中，不支持的则不在列表里。TFTP 协议一个特点是，它不允许任何一方连续发送 2 个数据包，必须是``一来一回``。具体的扩展功能在协议实现时我们再详细研究。

## 探究数据包具体格式

我们看看 TFTP 数据包的组装方式，为我们代码实现该协议奠定基础。TFTP 协议总共有5种不同数据包，分别对应读请求，写请求，数据块，接收回应(ACK)，以及错误。前两种数据包格式一样，只不过某些值域设置有差别，剩下的三种数据包格式各不相同。但无论哪一种数据包，他们都包含一个值域叫操作码，用来定义该数据包属于那种类型。

我们先看读请求和写请求数据包的格式，首先是2字节表示操作码，它用来表示当前数据包的类型，取值 1 表示该数据包是个读请求，2 表示该数据包是；接下来是可变长字段，它用来表示要读取或上传的文件名，它使用 ASCII 码并以 0 表示结尾；第三个字段叫 Mode，也是可变长字段，用来表示传输文件的数据类型，如果传输的是字符串文件，那么它填写字符串 "netascii"，如果传输的是二进制文件，那么它填写字符串 "octet"，这些字符串都以 0 结尾，其结构用下图表示：

![读写请求数据包](2c942dc5/读写请求数据包.png)

接着我们看看传输数据块的数据包，它的前 2 字节也是操作码，取值 3 用于表示数据包用于数据块传输，接下来是 2 字节，用于表示数据块编号，最后是可变长字段 Data，用于装载数据块，该数据包的格式如下：

![传输数据块数据包](2c942dc5/传输数据块数据包.png)

然后是应答数据包，它开始 2 字节也是操作码，取值 4，接下来2字节表示接收到的数据块编号，相应结构如下图：

![应答数据包](2c942dc5/应答数据包.png)

最后一个是错误数据报，它前 2 字节表示操作码，取值 5；接下来 2 字节表示错误码，0 表示未知错误，1 表示文件不存在，2 表示权限不足，3 表示磁盘已满，具体的错误码我们在实践时再具体分析；接下来是可变长字段，它用字符串的形式描述具体错误，该数据包的结构如下图：

![错误数据包](2c942dc5/错误数据包.png)

## 具体代码实现
