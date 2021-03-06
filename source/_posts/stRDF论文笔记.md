---
title: stRDF论文笔记
date: 2021-04-22 08:46:44
updated: 2021-07-05 10:38:45
tags: 时空推理
mathjax: true
---

## Modeling and Querying Metadata in the Semantic Sensor Web: The Model stRDF and the Query Language stSPARQL

Manolis Koubarakis and Kostis Kyzirakos ， 2010 有效时间+空间几何（点、线、线段、多边形）

本文扩展了语义传感器网络中常用的元数据模型RDF，可以表示传感器的空间位置、传感器的移动轨迹、传感器网络的覆盖范围、传感器捕获的有效时间。

本文采用n维空间$\mathbb Q^n$ 中的<u>**半线性**</u>点集表示空间几何，也就是线性方程的一阶逻辑和不等式中的**无量词公式**的集合。

半线性点集可以表示点、线、线段和多边形（不能表示高次多项式，如：圆）。

### 数据模型

#### 线性约束

约束以一阶语言$\mathcal L=\{\leq, \ +\} \cup \mathbb Q$表示，结构为$\mathcal Q=<\mathbb Q,\ \leq,\ (q)_{q\in\mathbb  Q }>$，由$\mathbb Q$表示的线性有序、稠密且无界的有理数集。

该语言的原子公式是线性方程和不等式:
$$
\sum_{i=1}^p a_ix_i\Theta a_0
$$
其中，$\Theta$ 是$=$ 或 $\leq$ 中的谓词，$x_i$ 表示变量，$a_i$是整型常量。

**定义1**：令$S$是$\mathbb Q^k$ 的子集，如果有$\mathcal L$的无量词公式$\emptyset(x_1,...x_k)$，其中$x_1,...x_k$是变量，使得当且仅当$\emptyset(x_1,...x_k)$ 在结构$\mathcal Q$为真，$(a_1,...a_k)\in S$，则$S$是半线性的。

#### sRDF数据模型

假设无限的集合序列$C_1,\ C_2,...$，与$I$，$B$和$L$都不相交。其中$C_k$是一阶语言$\mathcal L$的无量词公式中的第k个自由变量。表示$C$为$C_1\cup C_2\cup...$

**定义2**：sRDF三元组是集合元素为$(I\cup B)\times I\times (I\cup B\cup L\cup C)$ .

三元组的宾语可以是一个带有线性约束的无量词公式。

根据定义1，具有k个自由变量的无量词公式是半线性子集的有限表示。

#### stRDF数据模型

RDF已经支持用户定义时间，stRDF扩展了有效时间的表示。时序约束是使用单个变量的原子时态约束的布尔组合。

**定义3**：stRDF四元组是带有第四个组件时序约束$\tau$的RDF三元组，其中$\tau$定义为真实世界中有效**时间点**的集合。

### SPARQL的格式与语义

(i)  非空间变量的集合$V_{ns}$

(ii)  无限序列$V_s^1,\ V_s^2...$，用于表示集合$C_1,\ C_2$元素的变量的集合

(iii)  真实变量$V_r$

用$V_s$表示无限并集$V_s^1\cup V_s^2\cup ...$，$V$表示$V_{ns}\cup V_s \cup v_r$，$V$与$T$不相交。

**定义4**：令G为在时刻T的stRDF图，p是三元组模式，$P_1,\ P_2$是图模式。评估图G的图模式P，表示为$[[P]]_G$.

**定义5：** k元空间术语的表示如下：

(i)  $\mathcal L$的无量词公式，来自集合$C_k$

(ii)  来自集合$V_s^k$ 的空间变量

(iii)  交集、并集、差集、边界、最小边界盒以及buffer

(iv)   空间术语t的投影$t[i_1,...i_k']$ ，其中$i_1,...i_k'$是小于等于k的正整数

**定义6**：度量空间术语表示为$f(t)$，其中$f$是一个度量函数体积、面积、长度、最大值或最小值，$t$是空间术语。$AREA$要求$k\geq2$，$LEN,MAX,MIN$要求$k=1$.

**定义7**：空间术语是k元空间术语或度量空间术语

**定义8**：令$t$是空间术语。令$\mu$是使所有空间变量$t$为$dom(\mu)$中元素的映射。对于$\mu$，$t$ 的值表示为$\mu(t)$.

**定义9**：原子空间条件：拓扑关系；线性方程或不等式。

**定义10**：空间条件是原子空间条件的布尔组合。

**定义11**：映射满足空间条件R，表示为$\mu ╞ R$，R是原子的，且用$\mu(x)$替换R中的每个空间变量x得到的空间条件适用于$\mathbb Q^n$中的半线性集。

**定义12**：给定一个在时刻T的stRDF图G，图模式P，和空间条件R，则有$[[P\ FILTER\ R]]_G=\{\mu\in[[P]]_G\ |\ \mu ╞ R \}$，在$\mu$满足R的条件下，$\mu$属于评估图G的图模式P。

**定义13**：令t为空间术语，并且z是不在t中的空间变量，然后t as z称为目标变量z的空间扩展术语。

**定义14**：投影是由非空间变量，空间变量和扩展空间术语组成，使得所有扩展空间项的目标变量都彼此不同且与每个空间变量都不同。

**定义15**：令$\mu$是映射，$W$是具有空间和非空间变量的投影，以及扩展空间术语t as z。然后，$\pi_W(\mu)$是新的映射，使得：

(i)  $dom(\pi_W(\mu))=\{x_1,...x_l,z_1,...,z_m\}$

(ii) 对$1\leq i\leq l$ 有 $\pi_W(\mu)(x_i)=\mu(x_i)$；对$1\leq j \leq m$ 有 $\pi_W(\mu)(z_j)=\mu(t_j)$

**定义16**：stSPARQL查询是$(W,P)$对，其中$W$是投影，$P$是图模式。在图G上的查询$(W,P)$是映射集合${\pi_W(\mu)\ |\ \mu\in[[P]]}$.



## A data model and query language for an extension of RDF with time and space

Manolis Koubarakis, 2012 用户定义时间+OGC [WKT+GML]

本文关于空间的表示采用OGC标准表示几何信息，关于时间的表示只包含了用户定义时间，

简单特征既有空间属性也有非空间属性。其中，空间属性表示为几何值列，非空间属性表示为标准SQL数据类型列。

有两种类型的特征表的实现方式：

1. 仅使用SQL预定义数据类型

    基于经典的SQL关系表和存储在几何值列的几何对象(几何id);
    
    预定义类型：几何以坐标值存储

    二元类型：几何对象的well known binary表示

2. 具有几何类型的SQL

    SQL定义了新的几何数据类型

    没提供用于访问基于预定义SQL数据类型中的SQL函数

**利用stSPARQL实现几何函数**： 为每个函数定义一个URI，如`srdf: Contains`。在引入SPARQL时，可以采用`xsd:boolean srdf:Contains(srdf:geometry g1, srdf: geometry g2)`。函数的参数可以时变量，空间文字(WKT或GML)或其他扩展函数。其实就是将原函数定义一个返回值类型，并且改为`srdf: ____`形式，运算形式没有变化。空间术语包括空间文字，空间变量和复杂空间术语（空间谓词中拓扑、距离和方向，现在只有拓扑关系用于空间过滤表达式）。