---
title: Linux下使用VASP的一些小需求和解决
date: 2019-02-28 08:52:19
categories:  
- Linux
- VASP
- 集群里的那些坑坑洼洼
tags: 
 - VASP
 - 集群
 - PBS
description: 本文主要根据Linux下进行计算时遇到的小需求，结合一些常用的Linux命令进行一些总结，以实现一些任务计算或任务管理的小需求。
---

最近更新日期:2019-02-28 

---

# 1.记录任务开始和完成时间

在实际使用过程中发现，对于PBS系统来说，任务提交会返回一个作业ID，任务结束后这个作业ID在用账号下就消失了。如果有不同时间提交的任务有的时候作业结束了或者停止了，就容易找不到任务的目录，甚至会忘记是什么时间提交的什么类型的计算任务。而且PBS中提供的out和err输出也仅能显示任务的错误或标准输出。所以在提交任务的同时创建一个自己的`任务记录日志`就十分有必要。

这里主要使用`echo`、`date`、`time`命令，结合PBS的环境变量`$PBS_JOBID`、`$PBS_JOBNAME`、`$PBS_O_WORKDIR`在PBS中写一行小代码来完成。

```sh
#输出作业开始时间、任务ID、作业名、作业目录
echo ''$PBS_JOBID'  named  '$PBS_JOBNAME' in '$PBS_O_WORKDIR'  started at  '`date`'  ' >> /your_dir.log

#记录vasp实际运行时间输出到`任务ID.out`文件，同时将vasp程序运行的实时消息传递到当前目录下的`VASP.out`文件
time mpirun -genv I_MPI_DEVICE rdssm -np 36 vasp_std > VASP.out

#输出作业结束时间、任务ID、作业名、作业目录
echo ''$PBS_JOBID'  named  '$PBS_JOBNAME' in '$PBS_O_WORKDIR'  finished at  '`date`'\n  ' >> your_dir/log


```

关于date命令：[Linux date命令 | 菜鸟教程](http://www.runoob.com/linux/linux-comm-date.html)

关于time命令：[Linux date命令 | 菜鸟教程](http://www.runoob.com/linux/linux-comm-time.html)