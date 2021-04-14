---
title: Spatial论文笔记
date: 2021-04-13 08:14:50
tags: 空间数据
---

## A Spatial Logic based on Regions and Connection

David A. Randell, 1992

该理论假定了一种原始地二元关系，$C(x,y)$表示在区域内'x与y相连接'，对于区域内的**点**事件，表示'x和y共享同一个点'。同时理论还支持区域布尔组合的函数，以及允许显示表示特定区域的内部、闭包和外部的拓扑函数。

**连通区域**：当且仅当一个区域不能分成两个不相交的部分，则它是不连通的。如果一个区域分割后（这些部分都互不相连）的并集不是该区域，则该单独区域是连通的。 

**真子区域**：有非切向真子部分的区域。

**包含关系**：

![包含关系](https://tva1.sinaimg.cn/large/005SZbikgy1gpi653tbkwj30ic06jmyd.jpg)

**拓扑内部**： 必须穿过周围的物体，才能达到并接触包含的物体。

**几何内部**：可以构建一条线段，在不切断周围实体的情况下，既可以连接周围的实体，也可以连接包含的实体

<img src="https://tva1.sinaimg.cn/large/005SZbikgy1gpjgibn3jwj30ny0cc403.jpg" alt="RCC-8" style="zoom: 67%;" />

