---
title: RDF 1.1 中文文档
date: 2021-03-15 19:40:13
tags: 语义Web
---

## 1. 引言

RDF（Resource Description Framework）资源描述框架，是一个表达关于资源的信息的框架。资源可以是任何事物，包括文档，人，物理对象，抽象内容。

RDF适用于处理需要由应用程序**处理**的Web信息，而不是仅将其显示给人们的情况。RDF提供了一个表达信息的通用框架，所以它可以在保证意义的条件下在应用之间交换信息。因为这是一个通用框架，所以应用程序设计者可以利用通用的RDF解析器和处理工具。<u>在不同应用之间交换信息的能力，意味着信息对于非初始创建它的应用程序也是可用的。</u>

特别是RDF可以用于Web发布和互联数据。例如，检索一个网址可以得到Bob的数据，包括他认识Alice的事实，通过定位她的IRI（国际资源标识符），检索Alice的IRI可以得到她更多的数据，包括与她的朋友、兴趣之间数据集的链接。一个人或一个自动化处理的过程可以追踪这样的链接，并且统计这些各种事物的数据。RDF的这些用法通常被称为链接数据。

<!--more-->

## 2. 为什么使用RDF？

下面阐明了RDF针对不同实践的不同用法：

- 将机器可读的信息，如热门的[schema.org](http://schema.org/)词汇表，添加到Web页面，使他们在**搜索引擎**上显示成一个更有利的格式，或者可以被第三方应用程序自动处理。
- 通过链接第三方数据集来**丰富数据集**。例如，一个关于绘画的数据集可以通过链接在维基百科中相关的画家来丰富，所以给出一个关于他们范围更广的信息和资源入口。
- 内联API，保证用户可以轻松的发现如何获取更多的信息。
- 使用当前发布的数据集作为**链接数据**，例如围绕特定主题建立数据集合。
- 通过内联RDF对跨多个Web网页的人的描述，建立**分布式社交网络**。
- 提供一个在数据库之间交换数据的标准方式。
- 内联组织内的不同数据集，可以使用**SPARQL执行跨数据集的查询**。

## 3. RDF数据模型

### 3.1 三元组

RDF让我们可以表示资源，格式如下：

```
<subject><predicate><object>         //主语 谓语 宾语
```

一个RDF语句可以表示两个资源之间的关系。**主语**和**宾语**表示两个资源是相关的，**谓语**表示他们之间的关系。关系以定向的方式表达（从主语到宾语），在RDF中称为**属性**。因为RDF语句包含三个元素，所以被称为三元组。

下面是一些三元组的例子：

```
<Bob> <is a> <person>.
<Bob> <is a friend of> <Alice>.
<Bob> <is born on> <the 4th of July 1990>. 
<Bob> <is interested in> <the Mona Lisa>.
<the Mona Lisa> <was created by> <Leonardo da Vinci>.
<the video 'La Joconde à Washington'> <is about> <the Mona Lisa>
```

相同的资源通常被多个三元组引用。在上面的例子中，Bob是四个三元组的主语，Mona Lisa是一个三元组的主语和两个三元组的宾语。<u>相同资源位于一个三元组的主语并且位于另一个三元组的宾语，可以找到两个三元组之间的关系</u>，这是RDF的重要部分。

我们可以将连接**图**可视化。图包含了节点和边。三元组的主语和宾语构成了图中的节点，谓语构成边。图1展示了样例三元组的结果。

![示例三元组的图](https://tva3.sinaimg.cn/large/005SZbikly1goktzh2b99j30nm0fsgqm.jpg)

​																					 图1

一旦有了一张这样的图，就可以使用SPARQL查询对达芬奇的画感兴趣的人。

在本节通过“抽象语法”描述了RDF数据模型，也就是说，数据模型是独立于特定的具体语法的（在文本文件中用于表示三元组存储的语法）。从抽象语法的角度看，不同的具体语法可能会产生完全相同的图。RDF图的语义是通过抽象语法确定的。具体的RDF语法在第五章节介绍。

### 3.2 国际资源标识符 IRIs (International Resource Identifier)

一个IRI定位一个资源。用作网址的URL（统一资源定位符）是IRI的一种。IRI的其他格式提供了一种不暗示其位置或访问方式的标识符。IRI的概念是URI（统一资源标识符）的概括，<u>允许在IRI字符串中使用非ASCⅡ字符</u>。

IRIs可以出现在三元组的**所有三个位置**中。

IRIs用于定位像文档，人，物理对象和抽象概念的资源。例如，在DBpedia中莱昂纳多·达芬奇的IRI为`http://dbpedia.org/resource/Leonardo_da_Vinci`

IRIs是**全局**定义，所以可以重用IRI定义相同的事物。比如很多人用`http://xmlns.com/foaf/0.1/knows`声明熟人的关系。

RDF不清楚IRI表示什么，然而，可以通过特定的词汇或规定赋予IRIs意义。例如，DBpedia 使用IRI `http://dbpedia.org/resource/Name`来表示相应的Wikipedia文章所描述的内容。

### 3.3 文字

文字是一个基本值，而不是IRIs。文字的例子包括像"La Joconde"的字符串，像"the 4th of July, 1990"的日期，还有像"3.14159"的数字。<u>文字与数据类型相关联</u>，可以使这样的值被正确的解析和解释。字符串文字可以**选择**与语言标签相关联。例如，"Léonardde Vinci"可以与"fr"语言标签关联，而“李奥纳多·达达·文西”可以与"zh"语言标签关联。

文字只能出现在三元组中的**宾语**位置。	

RDF Concepts文档提供了数据类型。这里包括了由XML Schema定义的很多数据类型，例如字符串，布尔值，整型，小数和日期。

### 3.4 空节点

IRIs和文字都为了RDF语句提供了基本的素材。除此之外，有时不必担心使用全局定位符就可以使用资源是很方便的。例如，我们可能像说明画作蒙娜丽莎的背景中有一个未被定义的树，我们可以知道那是柏树。可以使用空节点在RDF中表示没有全局标识符的资源。空节点就像代数中的简单变量，代表了一些东西，但不需要明确具体数值。

空节点可以出现在三元组的**主语和宾语**位置。可以用来表示资源，而无需使用IRI明确命名。

![空节点示例](https://tva2.sinaimg.cn/large/005SZbikly1goku0yyuysj30gq0dzjv8.jpg)

​																			 		图2

### 3.5 多图

RDF提供了一种将多个图中的RDF语句分组并且关联这些图的IRI的机制。多图是RDF数据模型的最新扩展。实际上，RDF工具创建者和数据的管理者需要一个探讨三元组集合子集的机制。多图最先在RDF查询语言SPARQL中被使用。因此，使用与SPARQL语言紧密关联的多图概念扩展了RDF数据模型。

在RDF文档中多图构成了RDF数据集。一个RDF数据集可以有多个命名的图，并且**至多**有一个未命名（默认）的图。

例如，样例1可以分为两个命名的图。第一个是Bob的社交网络图，由`http://example.org/bob`定义

```
<Bob> <is a> <person>.
<Bob> <is a friend of> <Alice>.
<Bob> <is born on> <the 4th of July 1990>.
<Bob> <is interested in> <the Mona Lisa>.
```

与图关联的IRI称为图名。

第二个图由维基百科提供`https://www.wikidata.org/wiki/Special:EntityData/Q12418`

```
<Leonardo da Vinci> <is the creator of> <the Mona Lisa>.
<The video 'La Joconde à Washington'> <is about> <the Mona Lisa>
```

未命名的图的例子，它包含了两个三元组，以图名`http://example.org/bob`作为主语。三元组将发布者和许可信息与该图IRI关联：

```
<http://example.org/bob> <is published by> <http://example.org>.
<http://example.org/bob> <has license> <http://creativecommons.org/licenses/by/3.0/>.
```

在样例数据集中，我们假设图名表示相应图的RDF数据来源，也就是说，检索<http://example.org/bob>

我们可以得到图中的四个三元组。

![示例数据集的图](https://tva3.sinaimg.cn/large/005SZbikly1goku1f280sj30nm0qsn41.jpg)

​																				 		图3

## 4. RDF词汇表

RDF数据模型提供了一种声明资源的方法。这个数据模型不对IRI代表什么作任何假设。实际上，RDF采用结合词汇或其他规定的方式，提供了关于资源的语义信息。

为了支持RDF词汇的定义，RDF提供了RDF Schema语言。这个语言可以定义RDF数据的语义特征。例如，可以声明IRI` http://www.example.org/friendOf `作为属性，`http://www.example.org/friendOf`三元组的主语和宾语必须是类`http://www.example.org/Person`的资源。

RDF Schema采用**类**的概念去指定可用于对资源进行分类的类别。实例和他所属的类之间的关系可以通过**类型**属性声明。使用RDF Schema可以创建类与子类和属性与子属性。通过**定义域**和**值域**限制可以定义特定三元组的主语和宾语的类型限制。上面的例子就是定义域限制："friendOf"三元组的主语应该是”Person“类。(RDF中没有定义类、属性等词汇，RDF只能是对具体的事物进行描述，缺乏抽象能力，无法对同一个类别的事物进行定义和描述。也就是说，RDF可以描述实体、实体的属性以及他们之间的关系，但是无法描述类与类之间的关系，类的属性等。<u>RDFS在数据层（data）的基础上引入了模式层（schema），模式层定义了一种约束规则，而数据层是在这种规则下的一个实例填充。</u>)

由RDF Schema提供的主要模型构造如下表所示：

| Construct                                                    | Syntactic form                     | Description                                                  |
| ------------------------------------------------------------ | ---------------------------------- | ------------------------------------------------------------ |
| [Class](http://www.w3.org/TR/rdf-schema/#ch_classes) (a class) | **C** `rdf:type rdfs:Class`        | **C** (a resource) is an RDF class                           |
| [Property](http://www.w3.org/TR/rdf-schema/#ch_property) (a class) | **P** `rdf:type rdf:Property`      | **P** (a resource) is an RDF property                        |
| [type](http://www.w3.org/TR/rdf-schema/#ch_type) (a property) | **I** `rdf:type` **C**             | **I** (a resource) is an instance of **C** (a class)         |
| [subClassOf](http://www.w3.org/TR/rdf-schema/#ch_subclassof) (a property) | **C1** `rdfs:subClassOf` **C2**    | **C1** (a class) is a subclass of **C2** (a class)           |
| [subPropertyOf](http://www.w3.org/TR/rdf-schema/#ch_subpropertyof) (a property) | **P1** `rdfs:subPropertyOf` **P2** | **P1** (a property) is a sub-property of **P2** (a property) |
| [domain](http://www.w3.org/TR/rdf-schema/#ch_domain) (a property) | **P** `rdfs:domain` **C**          | domain of **P** (a property) is **C** (a class)              |
| [range](http://www.w3.org/TR/rdf-schema/#ch_range) (a property) | **P** `rdfs:range` **C**           | range of **P** (a property) is **C** (a class)               |

在RDF Schema的帮助下可以构造RDF的数据模型。一个简单的例子：

```
<Person> <type> <Class>						 //Person是一个类
<is a friend of> <type> <Property>           //<is a friend of>的类型是属性
<is a friend of> <domain> <Person>			//<is a friend of>的定义域和值域都是人
<is a friend of> <range> <Person>
<is a good friend of> <subPropertyOf> <is a friend of>          //子属性
```

注意，虽然`<is a friend of>`通常被用作三元组的谓语属性，<u>但像这样的属性，本身就可以被描述为三元组的资源，或者为其他资源的描述提供值</u>。在这个例子中，`<is a friend of>`是三元组的主语，三元组为其分配了类型，定义域和值域，并且是描述`<is a good friend of>`属性的宾语。

最先被广泛应用的RDF词汇表是Friend of a Friend，用于描述社交网络。其他词汇表：

[Dublin Core]():保留了元数据元素，可以描述范围更广的资源。词汇表提供了像”创建者“，”发布者“和”标题“这样的属性。

[schema.org]()：是由一些搜索提供商开发的词汇表。网站管理员可以使用这些术语标记网页，让搜索引擎明白页面是关于什么的。

[SKOS]()：关于Web术语的分类方案词汇表。

词汇表从重用中获得价值：词汇表IRI被其他人重用的次数越多，使用IRI的价值就越高（所谓的网络效应）。这意味着您应该更喜欢重用别人的IRI，而不是发明一个新的IRI。

## 5. 编写RDF图

有很多不同的序列格式可以写RDF图。然而，采用不同的编写方式写相同的图会得到完全相同的三元组，在逻辑上是等效的。

### 5.1 RDF的turtle系列

在这个章节介绍了四个密切相关的RDF语言。从N-Triples开始，它为RDF三元组提供了基础语法。Turtle语法通过多种格式的语法糖扩展了这个基础语法，提高了可读性。随后讨论TriG和N-Quads，分别是Turtle和N-Triples的扩展。他们四个称为Turtle系列。

#### 5.1.1 N-Triples

N-Triples 为序列化RDF图提供了一种简单的基于行的纯文本方式。图1可以用N-Triples进行如下表示：

```
01    <http://example.org/bob#me> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://xmlns.com/foaf/0.1/Person> .
02    <http://example.org/bob#me> <http://xmlns.com/foaf/0.1/knows> <http://example.org/alice#me> .
03    <http://example.org/bob#me> <http://schema.org/birthDate> "1990-07-04"^^<http://www.w3.org/2001/XMLSchema#date> .
04    <http://example.org/bob#me> <http://xmlns.com/foaf/0.1/topic_interest> <http://www.wikidata.org/entity/Q12418> .
05    <http://www.wikidata.org/entity/Q12418> <http://purl.org/dc/terms/title> "Mona Lisa" .
06    <http://www.wikidata.org/entity/Q12418> <http://purl.org/dc/terms/creator> <http://dbpedia.org/resource/Leonardo_da_Vinci> .
07    <http://data.europeana.eu/item/04802/243FA8618938F4117025F17A8B813C5F9AA4D619> <http://purl.org/dc/terms/subject> <http://www.wikidata.org/entity/Q12418> .
```

每行代表一个三元组，尖括号<>中包含完整的IRI，行尾的句号表示三元组的结束。在第3行中，有一个文字，是日期。数据类型通过^^定界符附加到文字上。日期的表示符合XML Schema数据类型的规定。

因为字符串文字在N-Triples非常普遍，所以允许用户编写字符串文字时可以省略数据类型。比如第5行中的"Mona Lisa"等同于"Mona Lisa"^^。如果是带有语言标签的字符串，则标签会直接出现在字符串之后，并且用@符号分隔（例如"La Joconde"@fr）

![N-Triples RDF图](https://tvax3.sinaimg.cn/large/005SZbikly1goku1xj0edj30nm0g3dlh.jpg)

​																							图 4 

七行语句对应图中七条边。

N-Triples常用于交换大量RDF，使用面向行的文本处理工具处理大型RDF。

#### 5.1.2 Turtle

Turtle是N-Triples的扩展。除了N-Triples的基础语法之外，Turtle引入了许多语法的简便方式，例如，支持命名空间的前缀，数据类型文字的简写。Turtle权衡了易于编写，易于解析和可读性。图4可以用Turtle表示如下：

```turtle
01    BASE   <http://example.org/>
02    PREFIX foaf: <http://xmlns.com/foaf/0.1/>
03    PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
04    PREFIX schema: <http://schema.org/>
05    PREFIX dcterms: <http://purl.org/dc/terms/>
06    PREFIX wd: <http://www.wikidata.org/entity/>
07 
08    <bob#me>
09        a foaf:Person ;
10        foaf:knows <alice#me> ;
11        schema:birthDate "1990-07-04"^^xsd:date ;
12        foaf:topic_interest wd:Q12418 .
13   
14    wd:Q12418
15        dcterms:title "Mona Lisa" ;
16        dcterms:creator <http://dbpedia.org/resource/Leonardo_da_Vinci> .
17  
18    <http://data.europeana.eu/item/04802/243FA8618938F4117025F17A8B813C5F9AA4D619>
19        dcterms:subject wd:Q12418 .
```

第1-6行包含许多指令，这些指令为记下IRI提供了快捷方式。相对IRIs（例如bob#me第8行）是针对第一行在此处指定的base IRI进行解析的。第2-6行定义了IRI前缀（例如foaf:）,可以用于前缀的名字（例如foaf: Person）而不是全部的IRIs。通过将前缀替换成对应的IRI来构造相应的IRI（例如foaf: Person代表`<http://xmlns.com/foaf/0.1/Person>`）。第8-12行显示了Turtle如何为一组具有相同主语的三元组提供简写。第9-12行指定了以`<http://example.org/bob#me>`为主语的三元组的谓语-宾语部分。第9-11行末尾的分号表示，紧随其后的谓语-宾语对是新三元组的一部分，使用数据中显示最相近的主语，在本例中为`bob#me`。

第9行显示了一种特殊类型语法糖的示例，三元组应该被读作"Bob (is) a person"。<u>宾语a是模型实例关系中属性rdf: type的简写。</u>

**空节点的表示**

两个空节点的例子（蒙娜丽莎背景的柏树示例）

```
PREFIX lio: <http://purl.org/net/lio#> 

<http://dbpedia.org/resource/Mona_Lisa> lio:shows _:x .
_:x a <http://dbpedia.org/resource/Cypress> .
```

_:x是一个空节点。代表出现在蒙娜丽莎的背景的一个未命名资源；未命名资源是Cypress的实例。

Turtle还为空节点提供了另一种表示方法：

```turtle
@prefix foaf: <http://xmlns.com/foaf/0.1/> .

# Some resource (blank node) is interested in some other resource
# entitled "Mona Lisa" and created by Leonardo da Vinci.

[] foaf:topic_interest [
          dcterms:title "Mona Lisa" ;
          dcterms:creator <http://dbpedia.org/resource/Leonardo_da_Vinci> ] .
```

方括号在这里代表一个空节点。方括号内的谓词-宾语对被解释为空节点为主语的三元组。以“＃”开头的行表示注释。

#### 5.1.3 TriG

Turtle语法知识支持单个图的规范，但是没有方法去命名他们。TriG是Turtle的扩展，可以以RDF数据集的形式规范多个图。

多图示例如下

```
01    BASE   <http://example.org/> 
02    PREFIX foaf: <http://xmlns.com/foaf/0.1/> 
03    PREFIX xsd: <http://www.w3.org/2001/XMLSchema#> 
04    PREFIX schema: <http://schema.org/> 
05    PREFIX dcterms: <http://purl.org/dc/terms/> 
06    PREFIX wd: <http://www.wikidata.org/entity/> 
07    
08    GRAPH <http://example.org/bob>
09      {
10        <bob#me>
11            a foaf:Person ;
12            foaf:knows <alice#me> ;
13            schema:birthDate "1990-07-04"^^xsd:date ;
14            foaf:topic_interest wd:Q12418 .
15      }
16  
17    GRAPH <https://www.wikidata.org/wiki/Special:EntityData/Q12418>
18      {
19        wd:Q12418
20            dcterms:title "Mona Lisa" ;
21            dcterms:creator <http://dbpedia.org/resource/Leonardo_da_Vinci> .
22    
23        <http://data.europeana.eu/item/04802/243FA8618938F4117025F17A8B813C5F9AA4D619>
24           dcterms:subject wd:Q12418 .
25      }
26  
27    <http://example.org/bob>
28        dcterms:publisher <http://example.org> ;
29        dcterms:rights <http://creativecommons.org/licenses/by/3.0/> .
```

这个RDF数据集包含两个命名图。第8行和第17行给出了这两个图的名称。命名图中的三元组放置在匹配的花括号之间（第9-15，18-25行）。可以在图形名称前添加关键字Graph，可以提高可读性，但是主要是为了与SRARQL Update 匹配而引入的。

三元组和头部指令的语法符合Turtle语法

在第27-29行指定非命名图，来自RDF数据集的默认图。

![TriG 三元组](https://tvax1.sinaimg.cn/large/005SZbikly1goku3fcw7sj30nm0q4tgy.jpg)

​																						图5

#### 5.1.4 N-Quads

N-Quads是N-Triples简单的扩展，可以交换RDF数据集。N-Quads允许一行中有四个元素，捕捉该行三元组描述的图IRI。

```
01    <http://example.org/bob#me> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://xmlns.com/foaf/0.1/Person> <http://example.org/bob> .
02    <http://example.org/bob#me> <http://xmlns.com/foaf/0.1/knows> <http://example.org/alice#me> <http://example.org/bob> .
03    <http://example.org/bob#me> <http://schema.org/birthDate> "1990-07-04"^^<http://www.w3.org/2001/XMLSchema#date> <http://example.org/bob> .
04    <http://example.org/bob#me> <http://xmlns.com/foaf/0.1/topic_interest> <http://www.wikidata.org/entity/Q12418> <http://example.org/bob> .
05    <http://www.wikidata.org/entity/Q12418> <http://purl.org/dc/terms/title> "Mona Lisa" <https://www.wikidata.org/wiki/Special:EntityData/Q12418> .
06    <http://www.wikidata.org/entity/Q12418> <http://purl.org/dc/terms/creator> <http://dbpedia.org/resource/Leonardo_da_Vinci> <https://www.wikidata.org/wiki/Special:EntityData/Q12418> .
07    <http://data.europeana.eu/item/04802/243FA8618938F4117025F17A8B813C5F9AA4D619> <http://purl.org/dc/terms/subject> <http://www.wikidata.org/entity/Q12418> <https://www.wikidata.org/wiki/Special:EntityData/Q12418> .
08    <http://example.org/bob> <http://purl.org/dc/terms/publisher> <http://example.org> .
09    <http://example.org/bob> <http://purl.org/dc/terms/rights> <http://creativecommons.org/licenses/by/3.0/> .
```

第1-7行表示，第一个元素是图IRI。四边形中在图IRI之后制定了主谓宾语句，符合N-Triples语法的规定。第8行和第9行表示未命名图，因此缺少第四个元素，只包含常规三元组。

N-Quads常用于交换大量RDF，使用面向行的文本处理工具处理大型RDF。

### 5.2 JSON-LD

JSON-LD为RDF图和数据集提供了一个JSON语法。JSON-LD可以通过最小的变化将JSON文档转换为RDF。JSON-LD为JSON对象提供通用标识符，<u>这种机制可以使JSON文档指向一个位于Web其他位置的JSON文档描述的对象，也可以是数据类型和语言处理</u>。JSON-LD也通过图关键字的使用提供一种序列化RDF数据集的方法。

```json
01    {
02      "@context": "example-context.json",
03      "@id": "http://example.org/bob#me",
04      "@type": "Person",
05      "birthdate": "1990-07-04",
06      "knows": "http://example.org/alice#me",
07      "interest": {
08        "@id": "http://www.wikidata.org/entity/Q12418",
09        "title": "Mona Lisa",
10        "subject_of": "http://data.europeana.eu/item/04802/243FA8618938F4117025F17A8B813C5F9AA4D619",
11        "creator": "http://dbpedia.org/resource/Leonardo_da_Vinci"
12      }
13    }
```

第2行的键@context指向一个描述文档如何映射到RDF图的JSON文档（下方）。每个JSON对象对应一个RDF资源。在这个例子中，主要资源是`http://example.org/bob#me`，第3行通过键@id指定资源。当@id作为JSON文档的键时，指向一个IRI定位的资源对应的JSON对象。第4行描述了资源的类型，第5行描述了生日，第6行描述了他的一个朋友。第7-12行描述了兴趣和画作蒙娜丽莎。

为了描述这幅画，在第7行创建了一个新的JSON对象并且将它与第8行Mona Lisa的IRI相关联。在9-11行描述画作不同的属性。

```json
01    {
02      "@context": {
03        "foaf": "http://xmlns.com/foaf/0.1/", 
04        "Person": "foaf:Person",
05        "interest": "foaf:topic_interest",
06        "knows": {
07          "@id": "foaf:knows",
08          "@type": "@id"
09        },
10        "birthdate": {
11          "@id": "http://schema.org/birthDate",
12          "@type": "http://www.w3.org/2001/XMLSchema#date"
13        },
14        "dcterms": "http://purl.org/dc/terms/",
15        "title": "dcterms:title",
16        "creator": {
17          "@id": "dcterms:creator",
18          "@type": "@id"
19        },
20        "subject_of": {
21          "@reverse": "dcterms:subject",
22          "@type": "@id"
23        }
24      }
25    }
```

这段描述了一个JSON-LD文档如何映射到一个RDF图。第4-9行指定如何将Person,interest和knows映射到第3行命名空间定义的FOAF中的属性和类型。在第8行通过关键字@type和@id指定knows键含有可以作为IRI解析的值。

第10-12行将birthdate映射到一个schema.org的属性IRI，并且指定可以被映射到一个数据类型`xsd:date`的值。

第14-23行描述了如何映射title，creator和subject_of到Dublin Core的属性IRIs。当在文档中遇到`"subject_of": "x"`时，第21行关键字@reverse用于指定映射主语是x IRI，属性是`dcterms:subject`，**并且宾语与父JSON宾语对应。**

### 5.3 RDFa

RDFa是一种RDF语法，可以将RDF数据嵌入到HTML和XML文档。这使搜索引擎在爬Web时可以集成数据，并且可以丰富搜索结果。

```html
01  <body prefix="foaf: http://xmlns.com/foaf/0.1/
02                   schema: http://schema.org/
03                   dcterms: http://purl.org/dc/terms/">
04    <div resource="http://example.org/bob#me" typeof="foaf:Person">
05      <p>
06        Bob knows <a property="foaf:knows" href="http://example.org/alice#me">Alice</a>
07        and was born on the <time property="schema:birthDate" datatype="xsd:date">1990-07-04</time>.
08      </p>
09      <p>
10        Bob is interested in <span property="foaf:topic_interest"
11        resource="http://www.wikidata.org/entity/Q12418">the Mona Lisa</span>.
12      </p>
13    </div>
14    <div resource="http://www.wikidata.org/entity/Q12418">
15      <p>
16        The <span property="dcterms:title">Mona Lisa</span> was painted by
17        <a property="dcterms:creator" href="http://dbpedia.org/resource/Leonardo_da_Vinci">Leonardo da Vinci</a>
18        and is the subject of the video
19        <a href="http://data.europeana.eu/item/04802/243FA8618938F4117025F17A8B813C5F9AA4D619">'La Joconde à Washington'</a>.
20      </p>
21    </div>
22    <div resource="http://data.europeana.eu/item/04802/243FA8618938F4117025F17A8B813C5F9AA4D619">
23        <link property="dcterms:subject" href="http://www.wikidata.org/entity/Q12418"/>
24    </div>
25  </body>
```

上例包含四个特定的RDFa的属性，约束RDF三元组在HTML中的形式：`resource,property,typeof,prefix`

第1行的属性prefix像Turtle前缀一样指定简短IRI。严格来说，这种特定前缀可以被省略，因为RDFa具有一系列预定义的前缀，其中包括本示例中使用的前缀。

第4行和第14行的div元素含有resource属性，指定可以被写入HTML元素的RDF语句的IRI。第4行属性typeof与Turtle中的简写is a相似：主语`http://example.org/bob#me` 是类`foaf:Person`的一个实例 (`rdf:type`).

第6行有一个property属性，这个属性的值`foaf:knows`作为RDF属性IRI解析；属性href的值`http://example.org/alice#me`作为三元组的宾语解析。也就是说，第6行的RDF语句为：

```
<http://example.org/bob#me> <http://xmlns.com/foaf/0.1/knows> <http://example.org/alice#me> .
```

第7行的三元组中的宾语是一个文字。属性property在这里被指定在HTML中的time元素内。HTML请求time元素的内容应该是一个有效的时间值。通过使用HTML中time元素的语义，RDFa可以在没有确切的数据类型声明的情况下，视为`xsd:date`解析值。

第10-11行，属性resource也被用于指定三元组的宾语。这个方法用于宾语是一个IRI时，并且IRI本身不是HTML内容的一部分（例如属性href）。第16行包含了第二个文字的例子，在这里定义为属性span。如果RDFa不能推断文字的数据类型，就假设数据类型为`xsd:string`

通常不一定定义RDF语句作为HTML内容的一部分。在这种情况下，可以使用不呈现内容的HTML构造来指定三元组。第22-23行是一个例子。第23行HTML元素link在这里用于指定第22行Europeana video的宾语。

### 5.4 RDF/XML

RDF/XML为RDF图提供了一种XML语法。

```XML
01    <?xml version="1.0" encoding="utf-8"?>
02    <rdf:RDF
03             xmlns:dcterms="http://purl.org/dc/terms/"
04             xmlns:foaf="http://xmlns.com/foaf/0.1/"
05             xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#"
06             xmlns:schema="http://schema.org/">
07       <rdf:Description rdf:about="http://example.org/bob#me">
08          <rdf:type rdf:resource="http://xmlns.com/foaf/0.1/Person"/>
09          <schema:birthDate rdf:datatype="http://www.w3.org/2001/XMLSchema#date">1990-07-04</schema:birthDate>
10          <foaf:knows rdf:resource="http://example.org/alice#me"/>
11          <foaf:topic_interest rdf:resource="http://www.wikidata.org/entity/Q12418"/>
12       </rdf:Description>
13       <rdf:Description rdf:about="http://www.wikidata.org/entity/Q12418">
14          <dcterms:title>Mona Lisa</dcterms:title>
15          <dcterms:creator rdf:resource="http://dbpedia.org/resource/Leonardo_da_Vinci"/>
16       </rdf:Description>
17       <rdf:Description rdf:about="http://data.europeana.eu/item/04802/243FA8618938F4117025F17A8B813C5F9AA4D619">
18          <dcterms:subject rdf:resource="http://www.wikidata.org/entity/Q12418"/>
19       </rdf:Description>
20    </rdf:RDF>
```

在RDF/XML中RDF三元组被指定在XML元素`rdf:RDF`中。开始标签`rdf:RDF`提供了XML元素和属性名字的简写方式。XML元素`rdf:Description`(是`http://www.w3.org/1999/02/22-rdf-syntax-ns#Description`的简写)用于定义三元组的集合，通过属性about指定主语的IRI。第一个语句块（7-12）有四个子元素。子元素的名字是一个IRI表示一个RDF的property，例如: `rdf:type`(第8行)。每个子元素表示一个三元组。在这种情况下，三元组的宾语也是一个IRI，通过属性`rdf:resource`指定宾语。例如，第10行对应的三元组：

```
<http://example.org/bob#me> <http://xmlns.com/foaf/0.1/knows> <http://example.org/alice#me> .
```

当三元组的宾语是一个文字，文字值作为property元素的内容输入（9和14）。数据类型由property元素指定（9）。如果数据类型被省略，并且没有语言标签，则被认为数据类型`xsd:string`。

## 6. RDF图的语义

使用RDF的首要目标是能够自动合并来自多个来源的可用信息，来形成更大的集合。传递时，都采用同样简单的方式，主谓宾三元组。为了保证信息的连贯性，我们不仅需要标准的语法，还需要三元组的语义一致性。

RDF的语义：

1. IRIs用于命名主谓宾，是**全局**的，每次使用时都命名相同的事物。
2. 当主语和宾语**确实**存在谓语所描述的关系时，这个三元组就是true
3. 当RDF图中的三元组**都**为true时，该RDF图为true

具有这些声明性语义的RDF的好处之一是系统可以进行逻辑判断。也就是说，给定一组true的输入三元组，系统可以推断出其他三元组在逻辑上也必须是true的。第一个三元组的集合促成了其他的三元组。

鉴于RDF的灵活性，当创建了新的词汇表时，人们想要使用新的内容，可能想做许多不同类型的推理。当一种特定类型的推理在许多不同的应用中似乎有用时，可以将其记录为“蕴含”。RDF语义中包含了许多蕴含。

作为示例，请考虑以下两个语句：

```
ex:bob foaf:knows ex:alice .
foaf:knows rdfs:domain foaf:Person .
```

RDF语义文档告诉我们，从该图中得出以下三元组是合法的：

```
ex:bob rdf:type foaf:Person .
```

RDF的语义还告诉我们三元组：

```
ex:bob ex:age "forty"^^xsd:integer . 
```

导致逻辑上的不一致，因为文字不遵守为XML Schema数据类型整数定义的约束。

请注意，RDF工具可能无法识别所有数据类型。至少需要使用工具来支持字符串文字和带有语言标签的文字的数据类型。

与许多其他数据建模语言不同，RDF Schema允许相当大的建模自由。例如，同一实体可以同时用作类和属性。同样，“类”和“实例”之间也没有严格的区分。因此，RDF语义将以下图形视为有效：

```
   ex:Jumbo rdf:type ex:Elephant .
   ex:Elephant rdf:type ex:Species .
```

因此，大象既可以是一类（Jumbo为一个实例），又可以是一个实例（即动物物种类）。