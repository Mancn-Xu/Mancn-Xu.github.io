---
title: 集群日常中运行遇到的问题及解决方案
date: 2019-03-03 13:08:30
tags:
- Linux
- 集群
Categories:
- 集群里的那些坑坑洼洼
  
description: 本文主要介绍在集群中遇到的一些问题和解决方案


---

---

最近更新日期：2019-03-03

---

#### 1.使用Material Studio软件提交作业异常

问题时间：2019-03-02

问题描述：MS软件提交任务后，迅速报错，任务中止，没有反馈任何错误信息；尝试使用CASTEP、DMOL3模块均出现此异常。

问题解决：

1.MS软件内开启`Retain server files`，查看服务器内`stdouterr.txt`文件，发现如下错误

```sh
#DMOL3

/MS_dir/bin/dmol3.exe: error while loading shared libraries: libls_license64_g447.so: cannot open shared object file: No such file or directory

Warning: Permanently added 'cu39' (RSA) to the list of known hosts.
```



```sh
#CASTEP

Job started on host cu39

/MS_dir/bin/castepexe.exe: error while loading shared libraries: libls_license64_g447.so: cannot open shared object file: No such file or directory
```

2.怀疑cu39节点故障，尝试在其他节点运行程序。测试结果，运行正常

3.分析无法找到动态库的原因，怀疑cu39节点nfs服务启用异常

4.查询cu39节点硬盘挂载情况，并与cu40进行对比，发现cu39节点未成功挂载msi文件夹

```sh
$ df -h //列出所有挂载卷信息
$ showmount -e nodes01 //查询01号节点上的共享目录信息
```

5.手动挂载msi目录到mu39上

```sh
$ mount -t nfs -o vers=3 nodes01:/msi /msi //手动挂载01号节点上的msi文件夹到当前节点msi文件夹
```

**注意：挂载目录前后要有空格，挂在前要在当前节点下新建目录**

