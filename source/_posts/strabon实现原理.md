---
title: strabon实现原理
date: 2021-07-07 10:06:38
updated: 2021-07-07 10:06:38
tags: 时空关系
---

Strabon以透明的方式包含在RDF4J软件堆栈的层中，不会影响RDF4J的功能，同时可以兼容RDF4J的新版本。Strabon可以看作是RDF4J的SAIL api，可以在不影响当前功能的情况下安装在RDF4J之上。

**查询引擎**：扩展了RDF4J的评估器和优化器用于处理stSPARQL的查询，解析器和事务管理器没有修改。

**存储管理器**：该模块处理存储在PostGIS RDBMS层中的数据，为了存储空间文字，扩展了RDF4J的组件。

**PostGIS**：既用于stRDF数据的存储，也用于stsPARQL查询的评估。





