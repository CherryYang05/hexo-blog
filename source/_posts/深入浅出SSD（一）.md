---
title: 深入浅出SSD（一）
mathjax: true
tags:
  - SSD
categories:
  - [OpenSSD, SSD]
abbrlink: 11b7e709
date: 2022-05-12 16:07:36
---

>本文是《深入浅出SSD 固态存储核心技术、原理与实战》的学习笔记

<!-- more -->

## 一、SSD 介绍

SSD(Solid State Drive) 固态硬盘，主要以半导体闪存为介质的存储设备，属于 ROM 的一种。

SSD 硬件包括几大组成部分:主控、闪存、缓存芯片DRAM（可选，有些SSD上可能只有 SRAM，并没有配置 DRAM）、PCB（电源芯片、电阻、电容等）、接口（SATA、SAS、PCle等），其主体就是一块 PCB。

软件角度，SSD 内部运行固件（Firmware, FW）负责调度数据从接口端到介质端的读写，还包括嵌入核心的闪存介质寿命和可靠性管理调度算法，以及其他 SSD 的算法。

SSD 控制器，闪存和固件是 SSD 三大技术核心。

SSD 的容量，性能，可靠性，寿命，尺寸，功耗，噪声，重量，抗震，温度都优于机械硬盘，目前价格也逐渐接近机械硬盘。

## 二、SSD 基本工作原理

SSD 的输入是命令(Command)，输出是数据(Data)和命令状态(Command Status)。SSD 前端(Front End)接收用户命令请求，经过内部计算和处理逻辑，输出用户所需要的数据或状态。


![SSD系统调用](https://raw.githubusercontent.com/CherryYang05/PicGo-image/master/images/20220512163939.png)

SSD 的三大功能模块：
- 前端接口和相关的协议模块;
- 中间的 FTL 层（Flash Translation Layer，闪存转换层）模块;
- 后端和闪存通信模块

![接口协议](https://raw.githubusercontent.com/CherryYang05/PicGo-image/master/images/20220512164826.png)

FTL 算法决定了性能、可靠性、功耗等 SSD 的核心参数。


![SSD的垃圾回收（GC）](https://raw.githubusercontent.com/CherryYang05/PicGo-image/master/images/20220512165605.png)

## 三、SSD 系统架构

SSD 作为数据存储设备，其实是一种典型的(System on Chip)单机系统，有主控CPU、RAM、操作加速器、总线、数据编码译码等模块(见图2-1)，操作对象为协议、数据命令、介质，操作目的是写入和读取用户数据。

![SSD主控模块硬件图](https://raw.githubusercontent.com/CherryYang05/PicGo-image/master/images/20220512170135.png)

SSD 系统架构分为前端（Host Interface Controller，主机接口控制器）和后端（Flash Controller，闪存控制器），前端和主机接口打交道，包含 SATA、PCIe、SAS 接口等，后端和闪存打交道，完成数据编解码和 ECC。

### 1. 前端

SATA 的全称是 Serial Advanced Technology Attachment（串行高级技术附件），是一种串行硬件驱动器接口。

SAS（Serial Attached SCSI）即串行连接 SCSI，是新一代的 SCSI 技术。

PCIe（Peripheral Component Interconnect Express）是一种高速串行计算机扩展总线标准，原来被称为 3GIO

### 2. 主控 CPU

### 3. 后端

后端两大模块分别为 ECC 和闪存控制器。

ECC 模块是数据编解码单元，由于闪存存储天生的误码率，要在数据写入是加上 ECC 检验保护，同样读取的时候要通过 ECC 进行解码。

SSD 中的 ECC 算法主要有 BCH 和 LDPC，后者逐渐成为主流。

## 四、闪存物理结构

现在固态硬盘采用 NAND 闪存作为存储介质。闪存在写之前必须先擦除，不能覆盖写，因此 SSD 需要有垃圾回收，一个块读写次数超过一定限度就会损坏，因此要有磨损均衡。

闪存是一种非易失性存储器，基本存储单元是一种类 NMOS 的双层浮栅 MOS 管，电子被上下绝缘层包围，因此掉电不会消失。

施敏发明浮栅晶体管的来源是奶酪蛋糕~~

### SLC、MLC、TCL

一个存储单元存储 1bit 的数据的闪存，叫做 SLC（Single Level Cell），存储 2bit 数据的闪存叫 MLC（Multiple Level Cell），存储 3bit 数据的闪存为 TLC（Triple Level Cell）。

存储单元存储的值是按照存储单元里的电子相对个数来判断的，电子相对个数通过电压的区间范围来判断。

但同时，一个存储单元电子划分得越多，那么在写入的时候，控制进入浮栅极的电子个数就要越精细，所以写耗费的时间就越长;同样的，读的时候，需要尝试用不同的参考电压去读取，一定程度上加长了读取时间。所以我们会看到在性能上，TLC 不如 MLC，MLC 不如 SLC。

![SLC、MLC和TCL参数比较](https://raw.githubusercontent.com/CherryYang05/PicGo-image/master/images/20220513194602.png)

一个闪存内部的存储组织结构如下图所示:
一个闪存芯片有若干个 DIE（或者叫LUN），每个 DIE 有若干个 Plane，每个 Plane 有若干个 Block，每个 Block 有若干个 Page,每个 Page 对应着一个 Wordline，Wordline 由成千上万个存储单元构成。

![闪存内部组织架构](https://raw.githubusercontent.com/CherryYang05/PicGo-image/master/images/20220513194929.png)

DIE/LUN 是接收和执行闪存命令的最基本单元，在一个 LUN 中，不能对某个 Page 写的同时，又对其他 Page 进行读访问。

一个 LUN 又分为若干个 Plane，市面上常见的是 1 个或者 2 个 Plane，现在也有 4 个 Plane 的闪存了。每个 Plane 都有自己独立的 Cache Register 和 Page Register，其大小等于一个 Page 的大小。

固态硬盘主控在写某个 Page 的时候，先把数据从主控传输到该 Page 所对应 Plane 的 Cache Register 中，然后把整个 Cache Register 写到闪存阵列，读的时候相反，先把这个 Page 中的数据从闪存介质读取到 Cache Register 中，然后**按需**传给主控。

同时有 Cache Register 和 Page Register 的目的是优化闪存的访问速度（类似于二级缓存）。

我们通常所说的闪存读写时间，并不包含数据从闪存到主控之间的数据传输时间，也不包括数据在 Cache Register 和 Page Register 之间的传输时间。闪存写人时间是指一个 Page 的数据从 Page Register 当中写人闪存介质的时间，闪存读取时间是指一个 Page 的数据从闪存介质读取到 Page Register 的时间。

闪存一般都支持 Multi-Plane（或者Dual-Plane）操作，就是将同一个 LUN 中多个 Plane 中 Cache Register 写入数据后同时写入闪存介质，这样可以节省 Page 写入时间。

闪存的擦除是以 block 为单位的，因为在物理结构上，一个 block 中的存储单元共用一个衬底（Substrate）。

### 读、写、擦原理

量子隧道效应

### 三维闪存

### 3D XPoint

相变存储器：PRAM，PCM

PCM 通过一种微小的电阻作用使得玻璃融化，相变为晶体，现在 PCM 读取时间已经可以到皮秒级别。

PCM 特点：
- 掉电数据不丢失
- 可以按照字节访问
- 软件简单
- 写之前不需要擦除操作
- 功耗低，和闪存差不多
- 读写速度快
- 寿命远长于闪存

![几种存储器特性对比](https://raw.githubusercontent.com/CherryYang05/PicGo-image/master/images/20220513213919.png)
