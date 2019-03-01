---
title: 使用standalone模式（命令行）运行CASTEP
date: 2019-02-27 22:09:09
categories:  
 - CASTEP
 - 集群里的那些坑坑洼洼
 - Linux
tags: 
 - CASTEP
 - Material Studio
 - Linux
description: 本文主要介绍在standalone模式下使用MS，也就是使用命令行向集群提交CASTEP作业。
---

# 一、简介

Material Studio作为一款强大的材料计算模拟软件，其主要的优势在于有强大的图形界面支持，极大地方便了材料结构模型的建立。

但相信也会有一部分同学更喜欢使用Linux下的命令行进行交互，这样可以通过脚本实现一些列的批量操作。

MS本身提供了两种方式进行批量化计算，其一是通过其内置的API使用perl语言在MS软件内进行批量操作；除此之外，MS还提供了一种`standalone`模式，可以直接调用各模块的执行程序配合MPI以及PBS作业系统使用。

关于MS Linux并行版的集群配置问题在这里不会详细展开，本文仅适用于组内已经配置好MS Linux并行版、MPI并行套件和PBS作业系统的同学。

**文章的所有内容均出自Material Studio 内置Tutorials。如果有疑问或者是希望详细了解，还望各位查阅手册。**



# 二、CASTEP输入文件准备

与VASP类似，如果我们想要在命令行中运行CASTEP模块需要先了解CASTEP模块所需要的输入文件。
根据手册介绍:


File type	|	Input or Output	|	Brief description
---	|	---	|	---
.param * |	**Input**	|	General input file (text)
.cell *	|	**Input**	|	Structure input file (text)
.castep	|	Output	|	Report on the calculation (text)
.check, .castep_bin *	|	Output	|	Charge density, wavefunctions, structure, and so on (binary)
.bands *	|	Output	|	Eigenvalues (text)
.cst_esp *	|	Output	|	Electrostatic potentials (binary)
.geom *	|	Output	|	Geometry optimization trajectory data (text)
.md *	|	Output	|	Molecular dynamics trajectory data (text)
.ts *	|	Output	|	Transition state search trajectory data (text)
.phonon *	|	Output	|	Phonon data (text)
.pdos_weights *	|	Output	|	Weights required to calculate PDOS (binary)
.cst_ome *	|	Output	|	Matrix elements required to calculate optical properties (binary)
.elf *	|	Output	|	Electron localization function, ELF (binary)
.bib	|	Output	|	Bibliography file in BibTeX format (text)
.otfg *	|	N/A	|	Information required for generating pseudopotential files on the fly stored on the client (text)

也就是说我们主要需要准备两个文件：

- `.param`文件：该文件即是CASTEP calculattion图形化界面所对应的内容

- `.cell`文件：该文件在win操作系统下是一个隐藏文件，其对应于可视化界面中的.xsd文件

也就是说最简单的单点能计算，我们只需要导出`.param`文件和`.cell`文件并发送至服务器即可。

除此之外当然是必不可少的`.pbs`文件。

## 2.1 获取CASTEP输入文件

1.在图形界面的Calculation界面中设置好参数，点击底部的`Save Files`按钮，可以导出计算参数。
![-w595](https://img.myvividlife.cn/15512740034694.jpg)

2.在左侧文件夹中我们已经可以看到以.param为后缀的文件。
![-w679](https://img.myvividlife.cn/15512741300635.jpg)

3.在对应地文件夹中可以看到`.cell`为后缀的结构文件。
![-w599](https://img.myvividlife.cn/15512742386388.jpg)

4.现在我们已经准备好了两个输入文件，通过FTP软件或者是MS中的Tools-File Transfer工具可以将准备好的文件上传至集群中。

```sh
[shenjie@mu01 dapl]$ cd test
[shenjie@mu01 test]$ ls
BaTiO3.castep  BaTiO3.param
[shenjie@mu01 test]$
```

## 2.2 准备PBS文件

关于PBS的使用可以详见[《PBS学习笔记》](https://myvivi.life/2019/02/21/PBS_Notes/)

这里直接贴出PBS脚本

```sh
#!/bin/bash -l
#
# Job: Task

#
#PBS -m n
#PBS -l nodes=1:ppn=20
#PBS -l walltime=1696:00:00
#PBS -e /your_dir/PBS_JOBID.err
#PBS -o ./PBS_JOBID.out
#PBS -j oe
#PBS -N Mancn //修改为你的任务名
#PBS -q batch
#


#添加MS安装路径的环境变量
export PATH=PATH:.:/home/msi/BIOVIA172/MaterialsStudio17.2/etc/CASTEP/bin

#添加MPI并行和MKL数学库的环境变量
source /opt/intel/composer_xe_2015/bin/compilervars.sh intel64
source /opt/intel/mkl/bin/intel64/mklvars_intel64.sh
source /opt/intel/impi/5.0.2.044/bin64/mpivars.sh

#cd到任务目录
cd PBS_O_WORKDIR

#执行CASTEP脚本
RunCASTEP.sh -np 20 BaTiO3 //这里务必注意要省略.param

```

直接使用`qsub`命令运行即可


## 2.3 CASTEP执行脚本

根据《Material Studio Tutorials》的介绍，MS在standalone模式下执行模块的命令如下：

```
Usage:
RunCASTEP.sh [-h] [-np number of cores] [-q queue name] seedname (Linux)
or
RunCASTEP [-h] [-np number of cores] [-q queue name] seedname (Windows)
```

|Argument|Description|
|---|---|
|-h  |Displays the help text. 
|-np  |Specifies the number of cores on which to run CASTEP. When this option is not specified a single core is used. 
|number of cores  |The number of cores to use. 
|-q  |Submits the job to the specified queue. 
|queue name  |The name of the queue on which to run the job. 
|seedname  |The seed used to identify the set of CASTEP input and output files. The input files should be present in the directory in which the CASTEP script is started. 

If you wish to calculate properties, you should execute the script for the main run first:

```sh
RunCASTEP.sh -np 4 seedname
```

When that run is complete, you should make copies of the .check file that will be used in all subsequent properties runs:

```sh
cp seedname.check seedname_BandStr.check
cp seedname.check seedname_DOS.check
cp seedname.check seedname_Optics.check
```
When the appropriate copies have been made, execute RunCASTEP for each properties run:

```sh
RunCASTEP.sh -np 2 seedname_DOS
RunCASTEP.sh -np 2 seedname_BandStr
RunCASTEP.sh -np 4 seedname_Optics
```
These jobs can be run independently because they do not share input or output files


至此，我们就可以愉快的使用脚本运行CASTEP程序啦。其他模块的使用与CASTEP大同小异，有需求的通知自己尝试一下即可。)
