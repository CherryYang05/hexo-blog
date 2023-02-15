---
title: DNS协议原理实现
mathjax: true
tags:
  - 计算机网络
  - TCP/IP协议
  - DNS协议
categories:
  - 计算机网络
  - TCP/IP协议
abbrlink: 47ec812d
date: 2020-05-27 15:21:23
---
## DNS(域名解析协议)基本原理介绍

我们这章开始研究和实现一个体系较为复杂的协议，也就是域名解析协议，简写为DNS。该协议几乎也是我们"日用而不知"的幕后英雄，没有它肯定就没有现在的互联网繁荣。

当我们在浏览器上输入网址，例如``www.baidu.com``时，浏览器先通过DNS协议找到与该网址对应的IP地址，然后再使用IP去向服务器获取网页信息。也就是说互联网上的设备其实有两种辨认方法，一种是IP，一种是域名。就如同人的身份证，人有名字，同时也有几十位数字组成的身份证号。

人与人相互识别时，使用的都是名字，几乎没有人使用身份证号来识别他人的，即使身份证号相对于名字而言更加唯一和准确。说到底是因为人对数字识别很费劲，而记住名字很容易。
<!-- more -->
对计算机的访问也是如此。在互联网发展早期，计算机只是在局域网内互联，并且联网的机器非常有限，因此当时使用IP直接定位不同的机器。但是随着网络的发展，联网的机器越来越多，使用数字辨别每一台计算机变得越来越困难，于是人们开始想用更方便的记忆方式，于是自然就想到用字符串来替代难以记忆的数字。

然而对程序而言，它只能识别数字，于是字符串仅仅用作于方便记忆，在运行机制上，程序就得把字符串与IP数字进行转换。最早使用的转换机制很简单，甚至到现在还在使用，那就是hosts文件，它使用文本的方式将IP与字符串名字对应起来，如下图：

![host](47ec812d/host.png)

程序在运行时，先将该文件内容读入内存，当用户输入网址时，它先从里面的对应关系中，将网址直接转换成对应的IP地址。这种方法在主机数量少时适用，但现在网络上的主机数量数以千万计，如此我们得在文件中维护几万个对应关系，这显然不合情理，随着联网的设备越来越多，适用这种静态配置的方式越来越不合时宜，因此全网使用一种统一的IP字符串映射方式是势在必行。

如今能满足这种域名转换成IP需要机制，就是我们要研究和实现的DNS协议，它是极佳的分布式系统设计案例，互联网发展几十年来，接入网络的设备呈指数级增长，需要进行域名解析的请求自然也指数级增长，DNS自设计完成以来就具备了极佳的扩展性，因此它在没有大变动的情况下，满足日益增长的需求，可见其设计思路之巧妙。

### DNS协议系统

相比于其他网络协议，DNS协议本身更像是一个系统。它主要包含以下三种系统功能：

![DNS协议](47ec812d/DNS协议.png)

显然我们不可能实现全部功能，但我们会选择一些重要模块进行研究和实现，事实上抓住局部原理，对整体功能的把握也就能做到心中有数。DNS分为三大块，紫色部分是名字空间，它规定了域名的层级构造标准，第二部分是名字注册，它负责添加新设备的名称并防止名字冲突，第三部分是名字解析，它负责将域名转换为对应IP。

在了解DNS协议时，我们需要掌握一个很重要的概念叫域名。"域名"其实是对一个特定领域的统一称呼。例如我的大学名叫"北京化工大学"，于是对于我母校而言，它是一级域名，在下面又分很多个学院，例如理学院，化工学院，文法学院等，这些学院名称就是二级"域名"，学院内又有很多个系，像我所在的理学院有：数学系，物理系，化工系，这些系的名称就是三级域名，系下面又分班，比如我所在的数学系分为1，2，3，4班，这四个班名称对应的是四级域名，每个班内的人都有自己的名字，于是就对应五级域名。

### DNS域名结构

DNS中的域名以拓扑树的方式存在，如下图：

![DNS域名](47ec812d/DNS域名.png)

它以一个根节点开始，派生出一级域名，一级域名下面是二级域名，二级域名之后全都叫子域名。每一级域名可以使用数字和字符组合成的字符串来表示，其中字符串不区分大小写，同时一个域名下面的子域名不能相同，假设有一个一级域名叫``Black``,那么它下层的子域名可以是``white``，但必须只有一个，如果有另一个子域名叫``WHITE``，由于域名不区分大小写，因此两个域名被认为相同，这是不允许的。

由此域名设定时可以从根到叶子节点，中间以符号``.``隔开。上图中的Root始终对应空字符串，所以``www.baidu.com``对应于上图而言，第一级域名就是``com``，第二级域名就是``baidu``，第三级域名就是``www``，在解析的时候，该字符串域名就要倒转变成``www.baidu.com``。如果是百度知道，那么域名就是``zhidao.baidu.com``.

一级域名由特定机构控制例如IANA这类互联网管理机构，而二级域名往往对应一个特定组织或团体，例如baidu对应百度公司，三级域名则由组织自己控制。域名对公司而言是非常重要的信息资产，早期有很多聪明人通过域名碰瓷，也就曾大公司或机构不注意，用他们的名字注册域名，结果这些公司想用时只能从他们手中高价购买，很多人就靠这种手段发了大财。

### 域名解析流程

接下来我们看看域名解析的基本流程。首先要解析域名，我们先找到含有相关域名信息的服务器，然后向该服务器发送信息请求。问题在于，域名信息不是存储在固定服务器中，为了系统鲁棒性和扩展性，域名信息以分布式的形式存储在不同服务器里，因此第一步要查询哪个服务器包含了域名对应的信息。

假设我们要解析域名``C.B.A``，首先我们将请求发送给所谓的根域名服务器，该服务器会把拥有域名A的服务器地址返回给我们，返回的服务器可能知道域名B.A的信息或者它把关于A的信息返回后，再给我们一个知道域名B的服务器地址，返回的服务器可能知道域名C的信息，或者返回域名B的信息后，再告诉我们哪个服务器知道域名C的信息，因此我们在解析过程中要根据服务器返回信息进行选择。

## DNS协议系统运行流程及数据包解析

### 域名信息存储

DNS协议的运转需要客户端和服务器进行交互。由于服务器端需要存储大量的域名信息，同时每天需要应答海量的解析请求，因此它的设计必须遵循``分布式系统``。客户端向一台服务器请求解析服务时，对方可能没有相应的域名信息，于是它会向上一层查询，获得拥有给定域名信息的服务器，然后把对应服务器的信息归还给客户端，然后客户端再重新发起请求。

我们还需要关注域名信息如何在服务器上存储。在域名服务器上，信息存储有两种方式，一种是域名信息以二进制格式存储，这种格式对应的名称叫``Resource Record Filed Format``，同时为了方便管理员管理，这些信息又通过文本形式展现出来，对应的格式称为``Master File Representation``,管理员通过修改后者就能使得对应的二进制信息进行相应变换：

![域名信息存储](47ec812d/域名信息存储.png)

Resource Record 是一种特定数据结构，专门用于存储域名解析相关信息，例如域名对应的服务器IP，域名解析服务器地址等，在后面我们解析数据包时再深入探讨。

### 域名解析

域名解析其实有三种形式：

- 第一种是我们熟悉的，将域名发给服务器然后获得域名对应IP；
- 第二种叫反向解析，将IP发给服务器然后获得对应域名；
- 第三种叫电子邮件解析，将邮件地址发给服务器然后获得邮件的接收对象IP.

我们将主要关注第一种形式的原理和实现。

当我们执行第一种域名解析时，首先要做的是获得域名服务器地址。这个过程并非一撮而就，有可能我们查询第一个服务器时，它给我们返回另一个服务器的地址，然后我们继续查询；第二步是确定服务器后，我们要解析它返回来的数据内容。在这个过程中，第二步相对容易，而第一步则比较棘手。

在查询对应域名服务器时有两种方式，一种是``循环式``，第一个域名没有对应信息，但返回另一个它认为有对应信息的服务器，接着客户端向第二个服务器请求，第二个服务器又返回另一个服务器信息，该过程依次循环直到找到对应服务器为止：

![域名解析：循环式](47ec812d/域名解析：循环式.png)

第二种叫``递归式``，它与一种的区别在于，服务器承担起客户端查找对应服务器的职责，服务器会反复向其他服务器查询，直到拿到对应域名信息后，直接返回给客户端：

![域名解析：递归式](47ec812d/域名解析：递归式.png)

一般担任root服务器的角色的是路由器

### 数据包格式

接下来我们看看DNS数据包的基本格式，首先第一部分叫``Header``，用于描述消息类型，以及后续数据结构的相关信息；第二部分叫``Question``，它用来包含客户端想向服务器查询的信息；第三部分叫``Answer``，是服务器用于回复客户端查询；第四部分叫``Authority``，如果请求没有得到全部答复，这部分内容告诉客户端向哪个服务器进行查询；第五部分叫``Additional``，这部分包含客户端查询信息的附加说明，它并非必须，所以数据包的基本结构如下：

![DNS数据包](47ec812d/DNS数据包.png)

我们用wireshark抓取DNS有关的消息包后，对照上面描述的条目进行解析。启动wireshark，然后使用关键词DNS过滤，然后在浏览器里输入一个你以前没有访问过的网址，如果输入已经访问过的，浏览器会有缓存，因此不会走dns协议。以下是我抓取到的一个DNS解析请求包：

![DNS数据包具体格式(查询)](47ec812d/DNS数据包格式.png)

![DNS数据包格式（回复）](47ec812d/DNS数据包格式（回复）.png)

首先是头部，它包含12字节，从``Transaction ID`` 到 ``Additional RRs``，每个字段2字节。ID用来标志一次会话，一个会话内的数据包拥有相同ID。Flags分为两部分，第一部分一字节叫做``QR(query & response)``，用来表示该数据包是查询还是回答，如果是查询就设置为0，如果是回答就设置为1.如果是查询，那么第二个字节就是OpCode，进一步表明具体查询，它分为若干部分，前四个比特位用于表明查询类型，0表示查询域名对应IP，1不再使用；2表示查询域名服务器状态；3目前不使用，4用于服务器之间的交互；5也是用于服务器之间的交互。

第五个比特位叫``AA(Authoritative)``,它只在回复包中设置，用于表明回复的权威性，只有在最后能够获得完整的IP的服务器上，才会返回1，其他中间的服务器只是起到代理作用，返回0.第六个比特位叫``TC(Truncated)``，它用于表明数据是否被截断，用于DNS支持UDP和TCP，但使用UDP时数据包不能超过512字节，如果超过数据包就得截断成多个小数据包，如果该位设置成1，它表明双方需要通过TCP来建立连接。第八位叫``RD(Recursion desired)``，如果设置成1，它意味着客户端``请求递归式查询``，也就是让服务器帮忙向其他服务器询问，得到最终消息后再返还给客户端。**根据此比特位可以判断下面的信息是不是最终信息，如果该比特位为0，那么还需要根据下面的信息解析其他服务器传递过来的内容。**

接下来字节的比特位是``RA``，如果设置为1表示服务器支持递归式查询，也就是服务器把所有累活都承担了，0则是不支持。接下来三个比特位必须设置为0，接着4个比特位表示返回码，如果值为0表示返回数据正常，非0表示出现错误，其中取值1表示查询数据包格式错误；2表示服务器自身故障；3表示解析错误；4表示不支持所要求的查询；5表示拒绝查询请求；其他值我们暂时忽略。

接下来用于表示相应条目的数量，``Questions``表示有几个查询条目，``Answer RRs``表示有几个回复条目，``Authority RRs``表示有几个权威信息条目，所谓"权威"是指真正能够解析域名的服务器，如果当前服务器不能解析域名请求，它需要把请求转发给其他服务器时，它自己就不是Authoritive，我们家用路由器其实承担域名解析服务器的职责，但是它本身不可能包含所需要的域名信息，它会把请求转发给上一层服务器，因此路由器就不是"权威"域名解析服务器。由此一个DNS域名解析数据包的轮廓如下：

![DNS域名解析数据包](47ec812d/DNS域名解析数据包.png)

返回的问题段的数据结构：

![问题段数据结构](47ec812d/Answer段数据结构.png)

数据包：

![答案段数据包](47ec812d/Answer段数据包.png)

首先是答案名字，这个字段长度可变，存储的是要查询的域名，以00作为结尾。第二个是问题类型(Type)，它是2字节，用于表明查询的类型，取值1表示查询域名对应IP，取值2查询服务器名称，具体类型在后面我们用到时再详细讨论。最后是问题类别，一般而言写死为1(IN,表明查询媒介类型为因特网)。

这里我们讲解一下查询的域名(Queries Name)对应的字符串结构，例如对于字符串：www.baidu.com，它的对应格式为``[3]www[5]baidu[3]com``,其中[]内表示接下来字符个数，例如[3]表示后面跟着3个字符www,[5]表示接下来跟着5个字符，**注意到这些数字所在位置正好对应字符串中符号点所在位置。** 最后以``00``结尾。

接下来我们看``Answer Resource Records``的结构，服务器收到客户端请求，完成解析工作后，把解析信息存储在该结构里发回给客户端。它的结构如下，第一个是名字字符串，可变长，它对应要解析的域名或服务器名称。接着是资源类型，2字节，表明资源的类型，如果取值是5，那么接下来对应着域名服务器对应的字符串名称，接着是资源类别，2字节，一般设置成1；接着是TTL(Time To Live)，4字节，表明这些信息能在缓存中存储多久；接着是RDLength，2字节，用于表明接下来内容的长度；最后是相应内容，如果资源类型是5，那么内容就是字符串，如果是1，那么内容就是4字节的IP地址，该数据类型对应的格式轮廓如下：

![答案段完整数据结构](47ec812d/Answer段完整数据结构.png)

这里值得提到的是，如果资源类型5，那么对应的字符串才是"真正"域名，例如下面显示内容：

![资源类型5](47ec812d/资源类型5.png)

它显示的是，一开始我们使用域名``pan.baidu.com``去进行域名解析，此时解析服务器没有直接返回该域名对应的IP，而是返回另一个域名``yiyun.n.shifen.com``，前面``pan.baidu.com``其实是一个别名，打个比方，一个人可以使用假名和真名，假名可以随时变，真名则要跟身份证绑定。同样的道理，``pan.baidu.com``这个域名可以根据需要随时变化，例如以后它可以变成``pen.baidu.com``，但是第二个域名就唯一绑定一台服务器，我们只有拿这个域名去查询才能找到对应的IP。

为了简单起见，其他两种资源的数据格式我们暂时放一放，以后需要的时候才研究，在下一节我们将使用代码实现本节描述的DNS域名解析流程。

## 代码实现

```java
package Application;

import protocol.IProtocol;
import protocol.ProtocolManager;
import protocol.UDPProtocolLayer;

import java.net.InetAddress;
import java.nio.ByteBuffer;
import java.util.HashMap;
import java.util.Random;

/**
 * @Author Cherry
 * @Date 2020/5/29
 * @Time 16:53
 * @Brief DNS 域名解析协议
 */

public class DNSApplication extends Application {
    private byte[] resolve_server_ip = null;
    private String domainName = "";
    private byte[] dnsHeader = null;
    private byte[] dnsQuestion = null;
    private short transition_id = 0;
    private static int QUESTION_TYPE_LENGTH = 2;
    private static int QUESTION_CLASS_LENGTH = 2;
    private static short QUESTION_TYPE_A = 1;
    private static short QUESTION_CLASS = 1;
    private static char DNS_SERVER_PORT = 53;

    //该类型表示服务器返回域名的类型号
    private static short DNS_ANSWER_CANONICAL_NAME_FOR_ALIAS = 5;
    private static short DNS_ANSWER_HOST_ADDRESS = 1;

    /**
     * 初始化，一般将域名交给路由器让其查询IP
     *
     * @param destIP     要查询域名的服务器IP，一般为该局域网的路由器
     * @param domainName 域名
     */
    public DNSApplication(byte[] destIP, String domainName) {
        resolve_server_ip = destIP;
        this.domainName = domainName;
        Random random = new Random();
        transition_id = (short) random.nextInt();
        //随机选择一个发送端口，这个没有具体要求
        this.port = (short) random.nextInt();
        constructDNSPacketHeader();
        constructDNSPacketQuestion();
    }

    /**
     * 组装 DNS 首部，12B
     */
    private void constructDNSPacketHeader() {
        byte[] header = new byte[12];
        ByteBuffer buffer = ByteBuffer.wrap(header);
        //会话Id,2B
        buffer.putShort(transition_id);
        //2字节操作码
        short opCode = 0;
        /*
         * 如果是查询数据包，第16个比特位要将最低位设置为0,接下来的4个比特位表示查询类型，如果是查询ip则设置为0，
         * 第11个比特位由服务器在回复数据包中设置，用于表明信息是它拥有的还是从其他服务器查询而来，
         * 第10个比特位表示消息是否有分割，有的话设置为1，由于我们使用UDP，因此消息不会有分割。
         * 第9个比特位表示是否使用递归式查询请求，我们设置成1表示使用递归式查询,
         * 第8个比特位由服务器返回时设置，表示它是否接受递归式查询
         * 第7,6,5，3个比特位必须保留为0，
         * 最后四个比特由服务器回复数据包设置，0表示正常返回数据，1表示请求数据格式错误，2表示服务器出问题，3表示不存在给定域名等等
         * 我们发送数据包时只要将第9个比特位设置成1即可(从1开始向左数第9位)
         */
        opCode = (short) (opCode | (1 << 8));
        buffer.putShort(opCode);
        //接下来是2字节的question count,由于我们只有1个请求，因此它设置成1
        short questionCount = 1;
        buffer.putShort(questionCount);
        //剩下的默认设置成0
        short answerRRCount = 0;
        buffer.putShort(answerRRCount);
        short authorityRRCount = 0;
        buffer.putShort(authorityRRCount);
        short additionalRRCount = 0;
        buffer.putShort(additionalRRCount);
        this.dnsHeader = buffer.array();
    }

    /**
     * 构造DNS数据包中包含域名的查询数据结构
     * 首先是要查询的域名，它的结构是是：字符个数+是对应字符，
     * 例如域名字符串pan.baidu.com对应的内容为
     * [3]pan[5]baidu[3]com也就是把 '.'换成它后面跟着的字母个数
     */
    private void constructDNSPacketQuestion() {
        //解析域名结构，按照 '.' 进行分解,第一个1用于记录"pan"的长度，第二个1用0表示字符串结束
        dnsQuestion = new byte[1 + 1 + domainName.length() + QUESTION_TYPE_LENGTH + QUESTION_CLASS_LENGTH];
        String[] domain = domainName.split("\\.");
        ByteBuffer buffer = ByteBuffer.wrap(dnsQuestion);
        for (String str : domain) {
            //先填写字符个数
            buffer.put((byte) str.length());
            //填写字符
            for (int i = 0; i < str.length(); ++i) {
                buffer.put((byte) str.charAt(i));
            }
        }
        //用0表示域名表示结束
        byte end = 0;
        buffer.put(end);
        //填写查询问题的类型和级别
        buffer.putShort(QUESTION_TYPE_A);
        buffer.putShort(QUESTION_CLASS);
    }


    /**
     * 向服务器发送查询请求
     */
    public void queryDomain() {
        byte[] dnsPacketBuffer = new byte[dnsHeader.length + dnsQuestion.length];
        ByteBuffer buffer = ByteBuffer.wrap(dnsPacketBuffer);
        buffer.put(dnsHeader);
        buffer.put(dnsQuestion);

        byte[] udpHeader = createUDPHeader(dnsPacketBuffer);
        byte[] ipHeader = createIP4Header(udpHeader.length);
        byte[] dnsPacket = new byte[ipHeader.length + udpHeader.length];
        buffer.clear();
        buffer = ByteBuffer.wrap(dnsPacket);
        buffer.put(ipHeader);
        buffer.put(udpHeader);
        //将消息发给路由器
        try {
            ProtocolManager.getInstance().sendData(dnsPacket, resolve_server_ip);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 组装 UDP 包头
     *
     * @param data
     * @return
     */
    private byte[] createUDPHeader(byte[] data) {
        IProtocol udpProtocol = ProtocolManager.getInstance().getProtocol("udp");
        if (udpProtocol == null) {
            return null;
        }
        HashMap<String, Object> headerInfo = new HashMap<>();
        char udpPort = (char) this.port;
        headerInfo.put("source_port", udpPort);
        headerInfo.put("dest_port", DNS_SERVER_PORT);
        headerInfo.put("data", data);
        return udpProtocol.createHeader(headerInfo);
    }

    /**
     * 组装 IP 包头
     *
     * @param length
     * @return
     */
    private byte[] createIP4Header(int length) {
        IProtocol ipPrtocol = ProtocolManager.getInstance().getProtocol("ip");
        if (ipPrtocol == null || length <= 0) {
            return null;
        }
        HashMap<String, Object> headerInfo = new HashMap<>();
        headerInfo.put("data_length", length);
        ByteBuffer destIP = ByteBuffer.wrap(resolve_server_ip);
        headerInfo.put("destination_ip", destIP.getInt());
        byte protocol = UDPProtocolLayer.PROTOCOL_UDP;
        headerInfo.put("protocol", protocol);
        headerInfo.put("identification", transition_id);
        return ipPrtocol.createHeader(headerInfo);
    }

    /**
     * 解析服务器回发的数据包，首先读取头2字节判断 transition_id 是否与我们发送时使用的一致
     *
     * @param headerInfo data
     */
    @Override
    public void handleData(HashMap<String, Object> headerInfo) {
        System.out.println("\n==================== DNS START ====================");
        byte[] data = (byte[]) headerInfo.get("data");
        if (data == null) {
            System.out.println("Empty data...");
            return;
        }
        ByteBuffer buffer = ByteBuffer.wrap(data);
        short transitionID = buffer.getShort();
        if (transitionID != transition_id) {
            System.out.println("TransitionID is different!!!");
            return;
        }
        //读取2字节flag各个比特位的含义
        short flag = buffer.getShort();
        readFlags(flag);
        //下面两个字节表示请求数量(Questions)
        short questionCount = buffer.getShort();
        System.out.println("Client send " + questionCount + " requests.");
        //两字节表示服务器回复信息的数量
        short answerCount = buffer.getShort();
        System.out.println("Server return " + answerCount + " answers.");
        //两字节表示数据拥有属性信息的数量
        short authorityCount = buffer.getShort();
        System.out.println("Server return " + authorityCount + " authority resources.");
        //两字节表示附加信息的数量
        short additionalInfoCount = buffer.getShort();
        System.out.println("Server return " + additionalInfoCount + " additional info.");

        //处理回复包中的Question部分，这部分湖人查询包中的内容一模一样
        readQuestions(questionCount, buffer);
        //处理服务器返回的信息
        readAnswers(answerCount, buffer);
    }

    /**
     * 分析 flag 字段各个比特位的含义
     *
     * @param flag
     */
    private void readFlags(short flag) {
        //最高字节为1表示该数据包为回复数据包
        if ((flag & (1 << 15)) != 0) {
            System.out.println("This is packet returned from server...");
        }
        //如果第9个比特位为1表示客户端请求递归式查询
        if ((flag & (1 << 8)) != 0) {
            System.out.println("Client requests recursive query!(客户端请求递归查询)");
        }
        //第8个比特位为1表示服务器接受递归式查询请求
        if ((flag & (1 << 7)) != 0) {
            System.out.println("Server accept recursive query request!(服务器接受递归查询)");
        }
        //第6个比特位表示服务器是否拥有解析信息
        if ((flag & (1 << 5)) != 0) {
            System.out.println("Sever own the domain info!(拥有解析信息)");
        } else {
            System.out.println("Server query domain info from other servers!(无解析信息)");
        }
    }

    /**
     * 处理 Question 部分
     *
     * @param questionCount question 部分的数量
     * @param data          buffer
     */
    private void readQuestions(int questionCount, ByteBuffer data) {
        System.out.println("\n=============== Queries ===============");
        for (int i = 0; i < questionCount; i++) {
            readStringContent(data);
            System.out.println();
            //查询问题的类型
            short type = data.getShort();
            if (type == QUESTION_TYPE_A) {
                System.out.println("Request IP for given domain name");
            }
            //查询问题的级别
            short clasz = data.getShort();
            System.out.println("The class of the request is " + clasz);
        }
    }

    /**
     * 处理 Answer 部分
     * 回复信息的格式如下：
     * 第一个字段是 name，它的格式如同请求数据中的域名字符串
     * 第二个字段是类型，2字节
     * 第三字段是级别，2字节
     * 第4个字段是 Time to live, 4字节，表示该信息可以缓存多久
     * 第5个字段是数据内容长度，2字节
     * 第6个字段是内如数组，长度如同第5个字段所示
     *
     * @param answerCount 服务器返回的 answer 部分的数量
     * @param data        buffer
     */
    private void readAnswers(int answerCount, ByteBuffer data) {
        System.out.println("\n=============== Answers ===============");
        /*
         * 在读取name字段时，要注意它是否使用了压缩方式，如果是那么该字段的第一个字节就一定大于等于192，
         * 也就是它会把第一个字节的最高2比特设置成11，接下来的1字节表示数据在dns数据段中的偏移，
         * 即从DNS报文段开头开始偏移。
         * 因为规定字符串长度不能超过63，即6位，因此若发现字符串的长度超过(或等于)192，就是采用了压缩
         */
        for (int i = 0; i < answerCount; i++) {
            System.out.println(i + 1 + ": Name content in answer filed is:");
            if (isNameCompression(data.get())) {
                int offset = (int) data.get();
                byte[] array = data.array();
                ByteBuffer dup_buffer = ByteBuffer.wrap(array);
                //从指定偏移处读取
                dup_buffer.position(offset);
                readStringContent(dup_buffer);
                System.out.println();

            } else {
                readStringContent(data);
                System.out.println();
            }
            //类型
            short type = data.getShort();
            System.out.println("Answer type is : " + type);
            if (type == DNS_ANSWER_CANONICAL_NAME_FOR_ALIAS) {
                System.out.println("This answer contains server string name..." +
                        "(该答复中包含了服务器的字符串名称)");
            }
            //级别
            short clasz = data.getShort();
            System.out.println("Answer class: " + clasz);
            //接下来4个字节是TTL存活时间
            int ttl = data.getInt();
            System.out.println("This content can cache (该域名生存时间为):" + ttl + " seconds(秒)...");
            //接下来2字节表示数据长度，长度为4表示IP，其他长度为服务器字符串名称
            short length = data.getShort();
            if (type == DNS_ANSWER_CANONICAL_NAME_FOR_ALIAS) {
                readStringContent(data);
                System.out.println();
            } else if (type == DNS_ANSWER_HOST_ADDRESS) {
                //打印服务器的IP
                byte[] ip = new byte[4];
                for (int j = 0; j < 4; ++j) {
                    ip[j] = data.get();
                }
                try {
                    InetAddress inetAddress = InetAddress.getByAddress(ip);
                    System.out.println("IP for domain name is(域名解析得到的IP为): " +
                            inetAddress.getHostAddress());
                    System.out.println("==================== DNS END ====================");

                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
            System.out.println();
        }
    }

    /**
     * 解析域名字符串
     *
     * @param buffer buffer
     */
    private void readStringContent(ByteBuffer buffer) {
        byte charCount = buffer.get();
        //如果字符第一个数正确或者使用压缩方式，输出字符串内容
        while (charCount != 0 || isNameCompression(charCount)) {
            if (isNameCompression(charCount)) {
                int offset = buffer.get();
                byte[] array = buffer.array();
                ByteBuffer dup_buffer = ByteBuffer.wrap(array);
                dup_buffer.position(offset);
                readStringContent(dup_buffer);
                break;
            }
            //输出字符
            for (int i = 0; i < charCount; ++i) {
                System.out.print((char) buffer.get());
            }
            charCount = buffer.get();
            if (charCount != 0) {
                System.out.print(".");
            }
        }
    }

    /**
     * 判断字符串是否使用压缩
     * 若 7.8位 为 1，则采用压缩，因为允许的字符串长度最长为 64，即 6位
     *
     * @param b 字符串本串
     * @return
     */
    private boolean isNameCompression(byte b) {
        return (b & (1 << 7)) != 0 && (b & (1 << 6)) != 0;
    }
}

```

稍后进行代码讲解...
