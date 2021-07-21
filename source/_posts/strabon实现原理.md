---
title: strabon实现原理
date: 2021-07-07 10:06:38
updated: 2021-07-21 20:38:31
tags: 时空关系
---

Strabon以透明的方式包含在RDF4J软件堆栈的层中，不会影响RDF4J的功能，同时可以兼容RDF4J的新版本。Strabon可以看作是RDF4J的SAIL api，可以在不影响当前功能的情况下安装在RDF4J之上。

### RDF4J

**SAIL(Sorage And Inference Layer)** 是为对RDF数据进行低级失误访问而设计的接口集合，充当特定数据库实现和rdf4j框架的功能模块（解析器、查询引擎、终端用户API访问等）之间的耦合点。

[![WtNRrn.png](https://z3.ax1x.com/2021/07/20/WtNRrn.png)](https://imgtu.com/i/WtNRrn)

**Sail**接口是对RDF存储的主要访问点，可以说就是数据库，每个Sail对象由零个或多个SailConnection对象组成，集中了所有实际数据库的访问功能。**SailConnection**提供了执行查询，检索和修改三元组，以及管理事务的方法。

#### AbstractSail 和 AbstractSailConnection

rdf4j提供了为大多数SAIL功能提供了默认（抽象）接口，可以为任何具体的实现重载。

[![Wty2f1.png](https://z3.ax1x.com/2021/07/20/Wty2f1.png)](https://imgtu.com/i/Wty2f1)

AbstractSail类提供了Sail接口所有方法的基本实现，提供了以下关于具体实现Sail的好处：

- 实现了所有要求的基本getter/setter方法。
- 存储关闭管理，包括活动连接的宽限期和存储关闭时活动连接的最终强制关闭。
- 线程安全，处理关于打开多个连接的基本并发问题。
- 持续兼容性：未来rdf4j的发布在Sail提供的由AbstractSail默认实现中引进新功能。

同样的，AbstraciSailConnection提供了SailConnection接口所有方法的基本实现，提供了一下关于具体实现SailConnection的好处：

- 处理与启动/执行该事务相关的所有基本并发问题。
- （可配置）缓冲任何事务中的活动更改。
- 持续兼容性：未来rdf4j的发布在SailConnection提供的由AbstractSailConnection默认实现中引进新功能。

抽象基类使用命名规范`方法名**Internal**`，表示该方法是有关实现的具体子类，基本原理是在抽象类的公有方法中实现了基本并发处理以及其他记录，和他们相关的(protected)...Internal方法，可以由提供了**实际业务逻辑**的具体子类代码实现。

#### Stacking SAILS

SAIL API提供了StackableSail接口，允许SAIL在其他上层的stack实现，每个在stack中的SAIL实现了指定特征(推理，访问控制，数据过滤， 查询扩展)

















**查询引擎**：扩展了RDF4J的评估器(Evaluator)和优化器(Optimizer)用于处理stSPARQL的查询，解析器(parse)和事务管理器(transaction manager)没有修改。

**存储管理器**：该模块处理存储在PostGIS RDBMS层中的数据，为了存储空间文字，扩展了RDF4J的组件。

**PostGIS**：既用于stRDF数据的存储，也用于stSPARQL查询的评估。

strabon源码环境：jdk 1.6, tomcat 6.0.53, maven 3.2.3(解决https错误)




