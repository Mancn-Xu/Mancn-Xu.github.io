---
title: 简单介绍InfiniBand
date: 2019-02-26 19:00:09
categories:  
- VASP自学成才
- 集群里的那些坑坑洼洼
tags: 
 - VASP
 - 集群

description: 本文简单介绍集群中的另一种不为人所知的高速数据交换通道——IB网。具体内容不会深入，只是简单记录一些基础知识。
---

## 一、InfiniBand（IB网）简介

我们都知道对于集群计算而言，其最大的优势在于并行计算。但在大规模并行计算的同时，如何在节点之间快速传输和交换数据成为了集群所需要面对的严峻考验。

一个集群中需要支持多种类型的数据流，所以，我们可以选择在同一集群中同时采用不同类型的互联网络，这些不同的网络将各自支持不同的网络协议，同时，这些不同的网络也拥有不同的网络性能和特性。例如，基于以太网的网络，其带宽一般可以达到千兆（1Gb/s）、万兆（10Gb/s），可以通过TCP/IP通道来传输信息，但缺点是需要占用大量CPU资源来处理网络通信，导致整体处理效率的下降。

Infiniband是一种新型的总线结构，它可以消除目前阻碍服务器和存储系统的瓶颈问题，是一种将服务器、网络设备和存储设备连接在一起的交换结构的I/O技术。 它是一种致力于服务器端而不是PC端的高性能I/O技术。

现阶段其理论带宽最高可达44Gb/s，是常见千兆以太网的44倍。对于需要大量进行并行计算的集群来说，无疑是最为优秀的信号传输方案。
 
经过测试其可以大大减小多节点并行时数据在节点交换所需的时间。


## 二、集群中遇到的问题


上一篇文章[《PBS学习笔记》](https://myvivi.life/2019/02/21/PBS_Notes/)中说到，在节尝试并行任务的时候，集群一直报错：

任务提交脚本：
```
#PBS -l nodes=2:ppn=20
mpiexec.hydra -n 30 vasp_std > VASP.out
```
错误提示：
```
#PBS 输出文件
[18:cu40] unexpected DAPL event 0x4008
Assertion failed in file ../../src/mpid/ch3/channels/nemesis/netmod/dapl/dapl_init_rc.c at line 1525: 0
internal ABORT - process 0
[19:cu40] unexpected DAPL event 0x4008
Assertion failed in file ../../src/mpid/ch3/channels/nemesis/netmod/dapl/dapl_init_rc.c at line 1525: 0
internal ABORT - process 0

#VASP 输出文件
cu34:CMA:65dd:a1ebf700: 60167511 us(7 us): dapl_cma_active: CM ADDR ERROR: -> DST 10.0.0.32 retry (1)..
cu34:CMA:65dd:a1ebf700: 60167516 us(5 us): dapl_cma_active: ARP_ERR, retries(15) exhausted -> DST 10.0.0.32,25925
cu34:CMA:65df:d8134700: 60167529 us(4 us): dapl_cma_active: ARP_ERR, retries(15) exhausted -> DST 10.0.0.32,25925
cu34:CMA:65d7:bac99700: 60163599 us(4004212 us!!!): dapl_cma_active: CM ADDR ERROR: -> DST 10.0.0.32 retry (1)..
cu34:CMA:65d7:bac99700: 60163613 us(14 us): dapl_cma_active: ARP_ERR, retries(15) exhausted -> DST 10.0.0.32,25916
```

后来经过个人查阅，虽然通过将通信协议强制改为`shm:tcp`可以解决问题，但是发现并行效率相比单节点没有任何优势，这是因为集群使用千兆以太网，本身效率就不高。

后来通过百度、谷歌等各种查询，隐隐约约发觉是因为IB网卡的驱动没有装好，导致整个IB网的连接是断开的。根据网络的指示在user用户下找了半天也没找到相应的驱动那个程序和服务开启方法。遂暂时作罢。

这几天刚好有工程师过来日常维护，便向工程师询问了相关IB网络的问题。后来经过了解是因为IB网络的服务在配置好后一直没有开启，导致以默认的网络拓扑进行通信是，IB网不通发生错误。

经过工程师介绍，现在主流的IB网硬件供应商为：Intel/英特尔和Mellanox/迈络斯，这两家分别提供了不同的IB交换机驱动，如果需要自行安装的话一定要认准集群的网卡型号。当然最好的还是请工程师来排查、部署。


## 三、MPI通信结构表

本部分内容来自[Selecting a Network Fabric](https://scc.ustc.edu.cn/zlsc/chinagrid/intel/Getting_Started/2_6_Selecting_a_Network_Fabric.htm)部分集群应该可以直接使用默认设置。

这部分的设置内容主要是决定集群节点之间的通信方式。详见链接内容。

如果有更希望更进一步了解集群中MPI设置的同学可以看[《Running MPI applications on Linux over Infiniband cluster with Intel MPI》](https://www.mir.wustl.edu/Portals/0/Documents/Uploads/CHPC/WashU_6_intelmpi.pdf)，写的十分详细。

>The Intel® MPI Library dynamically selects the most appropriate fabric for communication between MPI processes. To select a specific fabric combination, set I_MPI_FABRICS or the deprecated I_MPI_DEVICE environment variable.

### 3.1 I_MPI_FABRICS

Select a particular network fabric to be used for communication.

#### 3.1.1 Syntax

`I_MPI_FABRICS=<fabric>|<intra-node fabric>:<inter-nodes fabric>`

```
<fabric> := {shm, dapl, tcp, tmi, ofa}
<intra-node fabric> := {shm, dapl, tcp, tmi, ofa}
<inter-nodes fabric> := {shm, tcp, tmi, ofa}
```
Deprecated Syntax

`I_MPI_DEVICE=<device>[:<provider>]`

#### 3.1.2 Arguments

|Argument|Definition|
|--|--|
|fabric|Define a network fabric|
|shm|Shared memory transport (used for intra-node)|
|ofi|OpenFabrics Interfaces* (OFI)-capable network fabrics, such as Intel® True Scale Fabric, Intel® Omni-Path Architecture, InfiniBand\*, and Ethernet (through OFI API).|
|dapl|Direct Access Programming Library\* (DAPL)-capable network fabrics, such as InfiniBand\* and iWarp\* (through DAPL). 
|tcp|TCP/IP-capable network fabrics, such as Ethernet and InfiniBand\* (through IPoIB\*).|
|tmi|Tag Matching Interface (TMI)-capable network fabrics, such as Intel® True Scale Fabric, Intel® Omni-Path Architecture and Myrinet\* (through TMI).
|ofa|OpenFabrics Alliance* (OFA)-capable network fabrics, such as InfiniBand\* (through OFED* verbs).
|ofi|OpenFabrics Interfaces\* (OFI)-capable network fabrics, such as Intel® True Scale Fabric, Intel® Omni-Path Architecture, InfiniBand\* and Ethernet (through OFI API). 

### 3.2 Correspondence with I_MPI_DEVICE

**Switch interconnection fabrics support without re-linking**

|device|Equivalent notation for the I_MPI_FABRICS variable|
|---|---|
|sock|TCP/IP sockets – ethernet,  IPoIB|
|shm|shared memory for large SMPs|
|ssm|shared memory + sockets|
|rdma|InfiniBand,  Myrinet,  Quadrix|
|rdssm|rdma + shm + Sock|

rdssm:
- Shared memory for intra-node processes
- RDMA for inter-node processes
- Fails over to sockets if RDMA device is not available (default)


## 四、心得
日后集群上计算的大家如果遇到类似的错误提示，建议大家直接致电集群工程师进行维护和排查。自己排查宛如大海捞针，辛苦不说还容易误入歧途。
不过话又说回来，自己排错的过程也是自己学习的过程，这种排查会让自己有更多对硬件、软件、参数之间关联的理解。推荐时间较为充裕，又比较爱瞎折腾的同学可以尝试。

