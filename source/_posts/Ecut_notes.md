---
title: ENCUT到底是个什么东东
date: 2019-02-26 19:09:09
categories:  
- VASP自学成才
- 集群里的那些坑坑洼洼
tags: 
 - VASP
 - 第一性原理

description: 本文由一次VASP收敛性测试遇到的实际问题出发，记述了出现问题的折腾经历和解决方案，以及对第一性原理一些概念的理解。

mathjax: true
---

预计文章可能会写的比较长。时间跨度也比较久。因为如果要讲清楚ENCUT究竟是个什么东西，可能会涉及到第一性原理计算所需要的各种理论、计算手段和近似方法，包括平面波方法、贗势的选取等等。

文章主要参考资料：
- `ChemiAndy`老师的[《关于平面波PW ——从波的角度理解平面波密度泛函》](http://chemiandy.lofter.com/post/1d254b22_69a16c6)。其中介绍了关于平面波、K点采样、动能截断等相关问题。
- 华中科技大学单斌老师等人编著的《材料学的纳米尺度计算模拟：从基本原理到算法实现》

---

# 0 写作动机

本文写作的主要动机主要还是源于一次实践中遇到的问题。而也正是这个问题促使了前面两篇文章[《简单介绍InfiniBand》](https://myvivi.life/2019/02/26/InfinitiBand_Notes/)和[《PBS学习笔记》的诞生](https://myvivi.life/2019/02/21/PBS_Notes/)。

问题来源：对PZT单胞进行收敛性测试，发现在截断能ENCUT超过900后系统终止。

```
终止位置信息：
 k-point 74 :   0.5000 0.3750 0.5000  plane waves:   13692
 k-point 75 :   0.5000 0.5000 0.5000  plane waves:   13696

 maximum and minimum number of plane-waves per node :     13724    13604

 maximum number of plane-waves:     13724
 maximum index in each direction:
   IXMAX=   11   IYMAX=   11   IZMAX=   26
   IXMIN=  -11   IYMIN=  -11   IZMIN=  -26

```

```
out输出信息：

running on    1 nodes
distr:  one band on    1 nodes,    1 groups
vasp.4.6.31 08Feb07 complex
POSCAR found :  4 types and   80 ions
WARNING: PSMAXN for non-local potential too small
LDA part: xc-table for Pade appr. of Perdew
POSCAR, INCAR and KPOINTS ok, starting setup
WARNING: wrap around errors must be expected
REAL_OPTLAY: internal error (1)     6851168     6851456

```

#### PSMAXN:
来自官网的解释：
> PSMAXN (the cutoff for the projection operators) is read form POTCAR. NTYP=1 is the forst of the potentials read from POTCAR.

>The warning means that PSMAXN is too small for the required cutoff energy (ENMAX) the first of the atoms given in POTCAR.please either use a harder potential or decrease ENMAX


排除各种硬件及软件设置因素后，发现是ENCUT和元素贗势之间可能有某种关联关系，导致VASP在正空间进行运算时ENCUT不能超过元素贗势所给定的EMAX太多。

解决方案为：
- 使用更低的ENCUT
- 使用更“硬”的元素贗势
- INCAR加入：`LREAL = .FALSE`

更多详细内容请参考《The VASP Manual》 ——[#6.39 LREAL-tag](https://cms.mpi.univie.ac.at/vasp/vasp/LREAL_tag_and_ROPT_tag.html)


## 0.1 采坑经历：

### （1）怀疑内存不够

1.考虑跨节点并行程序以获取更多内存

- 遭遇跨节点问题，遂先解决了跨节点问题详见《简单介绍InfiniBand》和《PBS学习笔记》

- 在多节点并行后，发现问题依旧。转而进行内存测试。

2.使用能使软件正常运行的较低ENCUT，并减少内存资源申请；

- 计算终止有一个波函数展开上限，8500左右。超过上限无论几个节点并行都会报错；低于8500，发现仅需非常低的内存使用量就可以完成计算；

遂排除内存问题，用时2天。

### （2）怀疑元素或结构问题

1.较重的原子在展开成波函数的时候会更加吃力，或者是它们的贗势有什么缺陷，不能使用太大的 ENCUT

- 单独提取了Pb、Zr、Ti、O的原子，并用最简单的INCAR进行了测试。

测试结果：
发现对于单个原子，ENCUT调整到2000eV都不会造成内存不足的问题。

2.是否体系中有些原子之间因为有强烈的相互作用，导致出错

- 将POSCAR中的10个原子（O6、Pb2、Zr1、Ti1）进行缩减。首先是从（O1、Pb1、Zr1、Ti1）其次是（O1、Pb1），再次是（Zr1、Ti1）


测试结果：
问题出现依旧。进而排除是原子间相互作用的原因。

接下来怀疑INCAR的配置问题。之所以最后尝试这种可能是因为，印象中并没有什么INCAR的参数会导致类似的错误，当然最终的结果告诉我，我还是太天真了。 

### （3）怀疑INCAR问题

1.尽量使用默认设置

- 发现仅设置`ENCUT`、`IBRION`、`ISIF`、`ISMEAR`时，vasp可以正常运行

遂怀疑可能是某个INCAR参数导致了计算错误。经过搜索和自己测试，发现起决定性作用的INCAR参数为`LREAL`

修改后VASP恢复正常。用时1天。


从问题出现到解决估计用了3个工作日加上1个周末。通过该问题的解决也顺道解决了一个集群上的软件问题并且加深了自己对于波函数、贗势、截断能的了解。

这也让我深刻的认识到，有很多错误混杂在硬件和软件之间，要积极探索，敢于尝试，同时也要在明确问题大概方向后尽快找专业人员解决。

---

以下是正文部分

---

# 一、基组是什么及基组的选择

## 1.1 计算化学中的基组

量子化学中有2大计算流派，基于分子轨道的的计算(HF理论)，和基于密度泛函的计算(DFT)。

- HF使用“轨道-轨道”和“轨道-核”的作用来计算能量。

- 密度泛函使用“密度-密度”，和“密度-外势(原子核)“以及“密度->交换相关能”来计算能量

因此，对于DFT来说，只要密度计算正确，你就别管我用了什么基函数来对密度进行展开表示。因此，有几种不同的基函数类型。其中由固体电子理论发展而来的平面波表示应用非常广泛，因为其中平面波基函数的使用数量主要由体积决定，而与电子数目无关，因而可以应用到较大体系。