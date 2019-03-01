---
title: PBS学习笔记
date: 2019-02-21 19:37:09
categories:  
- VASP自学成才
- 集群里的那些坑坑洼洼
tags: 
 - VASP
 - 集群
 - PBS
 - Linux

description: 本文主要总结了PBS系统的一些使用方法以及在组内使用集群遇到的一些问题
---


---
个人的PBS学习笔记
- 整理了网络上对于PBE文档的介绍，并结合实验室集群的相关信息进行整理。
- 包括一些在集群任务遇到的坑以及解决方案。

最近更新时间：2019-02-21

---


# 一、PBS简介

PBS(Portable Batch System)最初由NASA的Ames研究中心开发，主要为了提供一个能满足异构计算网络需要的软件包，用于灵活的批处理，特别是满足高性能计算的需要，如集群系统、超级计算机和大规模并行系统。

PBS的主要特点有：代码开放，免费获取；支持批处理、交互式作业和串行、多种并行作业。

PBS的目前包括openPBS, PBS Pro和Torque三个主要分支，本集群主要用的是torque+maui。作业系统有三个主要组成模块，SCHEDULER、PBS_SERVER  和PBS_MOM。

作业管理系统通过这三个模块间的相互工作来完成对节点的监控和分配任务。 
- Scheduler一直在检测节点状态，主要检测节点的CPU占用率,查看节点是否空闲。 并把结果每10S传送给PBS_SERVER,当有一个作业被提交后， 
- PBS_SERVER就会根据Scheduler返回的结果将作业分配到空闲的节点上,
- PBS_mom接收到运算命令就会告知节点开始计算。如果没有空闲节点，作业就一直在排队

作业管理软件可以自动且合理地分配资源，避免各个用户之间抢占CPU资源，造成机器死机。

# 二、PBS常用命令

在终端界面，可以使用`pbsnodes`命令查看节点的pbs状态。
PBS常用的命令有：

#### 作业控制

- `qsub`：用于提交作业脚本，所采用的选项一般放在pbs脚本中提交，详见pbs脚本选项；
- `qdel`：用于删除已经提交的作业；

- `qhold`:挂起作业；
- `qrls`：释放挂起的作业
- `qrerun`：重新运行作业
- `qmove`：将作业移动到另一个队列
- `qalter`： 更改作业资源属性

#### 作业检测

- `qstat`：显示作业状态

- `showq`： 查看所有作业

#### 节点状态

- `pbsnodes`：列出集群中所有节点的状态和属性

    - -a 列出所有节点及其属性
    - -l free 以行的方式列出空余节点


## 2.1 qsub命令
```
命令格式：qsub [-x]

命令作用：用于提交作业脚本

全部命令：[-a][-c][-C][-e][-I][-j][-k][-l][-m][-M][-N][-o][-p][-q][-r][-S][-u][-v][-V][-W][-z]  

常见命令：[-l][-j][-e][-o][-q][-N][-m]
```
|命令|作用|
|---|---|
|[-a date_time] |表示经过date_time 时间后作业才可以运行,格式为：[[[[CC]YY]MM]DD]hhmm[.SS]|
|[-A user_name] | 使用不同用户名提交作业，缺省使用当前用户名
|[-c interval] |定义作业的检查点间隔，如果机器不支持检查点，则忽略此选项
|[-C directive_prefix] |在脚本文件中以directive_prefix 开头的行解释为qsub 的命令选项。（若无此选项，则默认为’#PBS’ ）
|[-e path] |将标准错误信息重定向到path
|[-I] |以交互方式运行
|[-j oe\eo]|将标准输出信息与标准错误信息.oe合并到输出。eo合并到错误。
|[-k keep]|定义在执行结点上保留标准输出和标准错误信息中的哪个文件.keep 为o 表示保留前者，e 表示后者，oe 或eo 表示二者都保留
|[-l nodes=N:ppn=M]|请求N个结点，每个结点M个处理器|
|[-l mem=N[K\M\G][B\W]]|请求N {kilo\mega\giga}{bytes\words} 大小的内存
|[-m mail_options]|mail_option为a：作业abort 时给用户发信；为b：作业开始运行发信；为e：作业结束运行时发信。默认为a。 
|[-M user_list]|定义有关此作业的mail 发给哪些用户|
|[-N name]|作业名，限15 个字符，首字符为字母，无空格。
|[-o path]|重定向标准输出到path
|[-p priority]| 任务优先级，整数，[-1024，1023]，若无定义则为0
|[-q destination]|destination 有三种形式： queue , @server, queue@server
|[-r y\n]|指明作业是否可运行，y 为可运行，n 为不可运行
|[-S path_list] |指明执行运行脚本所用的shell，须包含全路径
|[-u user_list]|定义作业将在运行结点上以哪个用户名来运行
|[-v variable_list]|定义export 到本作业的环境变量的扩展列表
|[-V]|表明qsub 命令的所有环境变量都export 到此作业
|[-W additional_attributes]|additional_attributes ： 作业的其它属性
|[-z]|指明qsub 命令提交作业后，不在终端显示作业号


```
qsub代码示例及详解：

# Job: Task
#
#PBS -N //作业名
#PBS -m e //作业结束运行时发信
#PBS -l nodes=1:ppn=20 //调用系统资源，nodes=1:ppn=20，表示调用1个节点，每节点调用24核心
#PBS -l walltime=1696:00:00 //申请计算时间，此处表示要求计算1696小时
#PBS -l mem=120000mb //调用系统内存
#PBS -e /your_folder/$PBS_JOBID.err //错误输出重定向到某目录并命名，使用绝对路径
#PBS -o /your_folder/$PBS_JOBID.out //标准输出重定向到某目录并命名，使用绝对路径
#PBS -q batch //申请队列类型
#

```

### 更多设置：

如果需要指定节点进行计算：

`#PBE -l nodes=NODES1:ppn=20+NODES2:ppn=20+……`

其中`NODES1`、`NODES2`、`20`需要改成节点的名称和核心数量。




## 2.2 qstat命令

```
命令格式：qstat [-x]
命令作用：用于查询作业状态信息
全部命令：[-f][-a][-i] [-n][-s] [-R] [-Q][-q][-B][-u]
常见命令：
```

参数说明：

|命令|内容|
|---|---|
|-f  jobid|  列出指定作业的信息 
|-a        |列出系统所有作业
|-i         |列出不在运行的作业
|-n        |列出分配给此作业的结点
|-s		  |列出队列管理员与scheduler所提供的建议
|-R		  |列出磁盘预留信息
|-Q       |操作符是destination id，指明请求的是队列状态	  
|-q        |列出队列状态，并以alternative形式显示
|-au userid  |列出指定用户的所有作业
|-B       |列出PBS Server信息
|-r        |列出所有正在运行的作业
|-Qf queue |列出指定队列的信息
|-u        |若操作符为作业号，则列出其状态。


## 2.3 qdel命令
```
命令格式：qdel  [-W 间隔时间] 作业号
命令作用：用于删除已提交的作业
```
示例：

`qdel 211`：删除作业号为211的作业

---


## 2.4 PBS中常用的环境变量

|变量名|说明|
|---|---|
|登录SHELL继承来的变量|包括#PATH、$SHELL等|
|$PBS_O_HOST|qsub提交的节点名称|
|$PBS_O_QUEUE|qsub提交的作业的最初队列名|
|$PBS_JOBID|qsub提交的作业的绝对路径
|$PBS_JOBNAME|用户指定的作业名，#PBS -N|
|$PBS_NODEFILE|PBS系统指定的作业运行的节点名|
|$PBS_QUEUE|PBS脚本执行时所处的对列名|


# 三、MPI运行部分

## 3.1 mpi简介

MPI全称是message passing interface，即信息传递接口，是用于跨节点通讯的基础软件环境。它提供让相关进程之间进行通信，同步等操作的API，可以说是并行计算居家出游必备的基础库。

一个 MPI 程序包含若干个进程。每个 mpi 进程都运行一份相同的代码，进程的行为由通讯域(communication world)和该通讯域下的 id(rank id)所决定。

MPI的编程方式，是“一处代码，多处执行”。编写过多线程的人应该能够理解，就是同样的代码，不同的进程执行不同的路径。


mpi补充阅读：
>它是一个库，不是一门语言。可以被fortran，c，c\++等调用。MPI允许静态任务调度，显示并行提供了良好的性能和移植性，用MPI编写的程序可直接在多核集群上运行。在集群系统中，集群的各节点之间可以采用 MPI 编程模型进行程序设计，每个节点都有自己的内存，可以对本地的指令和数据直接进行访问，各节点之间通过互联网络进行消息传递，这样设计具有很好的可移植性，完备的异步通信功能，较强的可扩展性等优点。

## 3.2 MPI常用命令

对于登录集群计算而言，我们只需要掌握两种mpi运行指令即可，它们分别是：`mpirun`、`mpiexec`。

简而言之`mpirun`和`mpiexec`均是MPI程序运行命令：
- `mpirun` ：MPI程序快速执行命令，运行前不必运行mpdboot开启守护进程。
- `mpiexec`：MPI程序运行命令，运行前必须开启mpd守护进程。 

> 守护进程(daemon)是一类在后台运行的特殊进程，用于执行特定的系统任务。很多守护进程在系统引导的时候启动，并且一直运行直到系统关闭。另一些只在需要的时候才启动，完成任务后就自动结束。


简单来说，在服务器配置良好的情况下，绝大多数情况不需要考虑选项的配置，基本语句如下：

`mpirun -n 20 vasp_std` //20核心并行vasp_std程序，vasp_std需要加入环境变量

`mpiexec.hydra -n 20 vasp_std` //20核心并行vasp_std程序，vasp_std需要加入环境变量

**此时节点和核心的分配完全由PBS系统进行分配，跨节点运行时也会自动进行核心和节点的匹配**

另外，在实际操作过程中，我发现部分同学对于PBS和`mpirun -np`的作用可能不太理解，在这里想简单解释如下：
1. PBS是作业分配系统，通过.pbs脚本像PBS作业管理软件申请相应的核心数量和节点数量。
2. 但要注意，这并不意味着通过分配的作业会完全按照申请的资源运行。下面看一个小小的例子：


```
#PBS -l nodes=2:ppn=20 //请求分配2个节点，20个核心

mpirun -np 30 vsap_std //30核心并行vsap_std程序

```
这意味着`#PBS -l`命令部分决定问集群申请多少计算资源。
`mpirun -np`部分决定vasp程序实际使用多少个节点并行。
---


### mpirun:
> Use this command to launch an MPI job. The mpirun command uses Hydra as the underlying process manager.
The mpirun command detects if the MPI job is submitted from within a session allocated using a job scheduler like Torque*, PBS Pro*, LSF*, Parallelnavi* NQS*, SLURM*, Univa* Grid Engine*, or LoadLeveler*. The mpirun command extracts the host list from the respective environment and uses these nodes automatically according to the above scheme.
In this case, you do not need to create a host file. Allocate the session using a job scheduler installed on your system, and use the mpirun command inside this session to run your MPI job.

---

## 3.3* MPI进阶

MPI执行命令的详细格式如下：

```
常用形式： 
mpiexec <g-options> <l-options> <executable> 

mpiexec –configfile <file> 
其中，
    <g-options>  全局选项运用于所有MPI进程。 
    <l-options>  本地选项应用于部分MPI进程集合。 
    <executable> 可执行文件的路径。 
    <file>       包含命令行选项的文件。 

全局选项中常用参数：
    -gdb 调试运行 
    -machinefile <file> MPI进程分配文件 

本地选项中常用参数：
    -n num 设置执行MPI程序的进程总数 

注意：全局选项和本地选项顺序不要弄错。

```


具体内容详见[Intel MPI手册](https://software.intel.com/sites/default/files/intelmpi-2019u1-developer-guide-linux.pdf)：

补充阅读：[mpi常用命令](https://blog.csdn.net/shijinupc/article/details/6788990)


## 3.4* MPI通讯结构

之所以会出现这一节是因为在集群的使用中，当进行跨节点并行计算时，集群疯狂报错，查阅了关于MPI的很多资料，最终解决了在本地集群上的报错问题。最终发现问题的出现于MPI在节点之间的通讯结构（Communication Fabrics）有关，特别记录如下：

任务提交脚本：
```sh
#PBS -l nodes=2:ppn=20
mpiexec.hydra -n 30 vasp_std > VASP.out
```
错误提示：
```sh
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

经过查阅，怀疑是节点之间的通讯问题。遂查阅相关手册文档。

通讯结构如下：

|结构|介绍|
|---|---|
|shm|Shared memory transport (used for intra-node)|
|ofi|OpenFabrics Interfaces* (OFI)-capable network fabrics, such as Intel® True Scale Fabric, Intel® Omni-Path Architecture, InfiniBand\*, and Ethernet (through OFI API).|
|dapl|Direct Access Programming Library\* (DAPL)-capable network fabrics, such as InfiniBand\* and iWarp\* (through DAPL). 
|tcp|TCP/IP-capable network fabrics, such as Ethernet and InfiniBand\* (through IPoIB\*).|
|tmi|Tag Matching Interface (TMI)-capable network fabrics, such as Intel® True Scale Fabric, Intel® Omni-Path Architecture and Myrinet\* (through TMI).
|ofa|OpenFabrics Alliance* (OFA)-capable network fabrics, such as InfiniBand\* (through OFED* verbs).
|ofi|OpenFabrics Interfaces\* (OFI)-capable network fabrics, such as Intel® True Scale Fabric, Intel® Omni-Path Architecture, InfiniBand\* and Ethernet (through OFI API). 

通讯结构由变量 `I_MPI_FIBRIC` 控制，默认的行为：使用共享内存，并从结构表中选择最前面且可用的结构方式，结构表一般为（dapl,ofa,tcp,tmi,ofi），也可以看I_MPI_FABRICS_LIST。

但不知为何组里的集群在节点间通讯会有问题，所以需要手动设置通信结构。这里经过测试发现组里应该使用以太网来进行节点之间的数据交换。
``` sh
方法一：
mpiexec.hydra -genv I_MPI_FIBRIC=sch:tcp //节点内shm，节点外tcp

mpiexec.hydra -genv I_MPI_FIBRIC=tcp //节点内外均为tcp


方法二；
export I_MPI_FIBRIC=tcp //节点内外均为tcp

方法三：
设置结构列表，使用I_MPI_FABRICS_LIST变量，例如：
export I_MPI_FABRICS_LIST=ofi,tcp
```

参考文档：[linux下intel使用intel mpi](https://blog.csdn.net/qq_35571432/article/details/78501612)


# 四、vasp作业提交脚本

```sh
sub.pbs代码示例及解析：

# Job: Task
#
#PBS -N //作业名
#PBS -m n //是否发送邮件
#PBS -l nodes=1:ppn=20 //调用系统资源，nodes=1:ppn=20，表示调用1个节点，每节点调用24核心
#PBS -l walltime=1696:00:00 //申请计算时间，此处表示要求计算1696小时
#PBS -l mem=120000mb //\调用系统内存
#PBS -e /your_folder/$PBS_JOBID.err //错误输出重定向及命名，使用绝对路径
#PBS -o /your_folder/$PBS_JOBID.out //标准输出重定向及命名，使用绝对路径
#PBS -q batch //申请队列类型
#

export PATH=$PATH:.:/your_direction/vasp.5.4.4_vtst/bin //将vasp可执行文件的目录临时添加进环境变量

source /opt/intel/composer_xe_2015/bin/compilervars.sh intel64 //intel编译器的环境变量
source /opt/intel/mkl/bin/intel64/mklvars_intel64.sh //intel_mkl数学库的环境变量
source /opt/intel/impi/5.0.2.044/bin64/mpivars.sh //mpi并行的环境变量

cd /your_VASP_Cal_Folder/  //进入VASP计算目录

mpiexec.hydra -n 20 vasp_std //并行命令，调用1个节点，每个节点20个核心，运行vasp_std程序

```

# 五*、其它配置文件（部分协议已经过时）
 
![mpirun-intel mpi](https://img.myvividlife.cn/2D5B2FAC48992298DAC9EDAF6DADBC09.jpg)

![mpirun-open mpi](https://img.myvividlife.cn/8566911AEF0D986C75F97FAD9B33ECBF.jpg)

![PBS_script](https://img.myvividlife.cn/31EB7748213717F8D63B715576C8168C.jpg)
