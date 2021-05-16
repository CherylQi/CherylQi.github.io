---
title: SPARQL 1.1 笔记(二)
date: 2021-03-21 11:11:49
updated: 2021-03-31 20:18:38
tags: 语义Web
---

## 12 子查询

子查询是一种将SPARQL查询嵌入其他查询中的方法，通常可以实现其他无法实现的结果，例如<u>限制查询中某些子表达式的结果数量。</u>

由于SPARQL查询评估的自下而上的性质，因此<u>先对子查询</u>进行逻辑评估，然后将结果投影到外部查询。

请注意，只有从子查询之外，或者在作用域内投影的变量对外部查询是可见的。

**数据**

```turtle
@prefix : <http://people.example/> .

:alice :name "Alice", "Alice Foo", "A. Foo" .
:alice :knows :bob, :carol .
:bob :name "Bob", "Bob Bar", "B. Bar" .
:carol :name "Carol", "Carol Baz", "C. Baz" .
```

为所有认识爱丽丝并有名字的人返回一个名字（排序最低的名字）。

<!--more-->

**查询**

```SPARQL
PREFIX : <http://people.example/>
PREFIX : <http://people.example/>
SELECT ?y ?minName
WHERE {
  :alice :knows ?y .
  {
    SELECT ?y (MIN(?name) AS ?minName)
    WHERE {
      ?y :name ?name .
    } GROUP BY ?y
  }
}
```

**结果**

| y      | minName  |
| ------ | -------- |
| :Bob   | "B. Bar" |
| :carol | "C. Baz" |

通过首先评估内部查询来获得此结果：

```SPARQL
SELECT ?y (MIN(?name) AS ?minName)
WHERE {
  ?y :name ?name .
} GROUP BY ?y
```

这将产生以下解决方案序列：

| y      | minName  |
| ------ | -------- |
| :alice | "A. Foo" |
| :Bob   | "B. Bar" |
| :carol | "C. Baz" |

与外部查询的结果结合在一起：

| y      |
| ------ |
| :Bob   |
| :carol |

## 13 RDF数据集

RDF数据模型将信息表示为包含主语，谓词和宾语的三元组的图形。许多RDF数据存储区包含多个RDF图并记录有关每个图的信息，从而使应用程序可以进行涉及多个图的信息的查询。

针对表示图集合的*RDF数据集*执行SPARQL查询。RDF数据集包括一个没有名称的默认图和零个或多个命名图，其中每个命名图由IRI标识。SPARQL查询可以将查询模式的不同部分与不同的图进行匹配，如13.3节“查询数据集”中所述。

RDF数据集可能包含零个命名图；RDF数据集始终包含一个默认图。查询不需要涉及与默认图的匹配。该查询仅涉及匹配命名图。

用于匹配基本图形模式的图形是***活动图形***。在前面的部分中，所有查询都显示为针对单个图执行，RDF数据集的默认图为活动图。该`GRAPH`关键字用于为部分查询得到所有数据集中的命名图之一的<u>活动图</u>。

### 13.1 RDF数据集示例

RDF数据集的定义不限制命名图和默认图的关系。信息可以在不同的图表中重复；图之间的关系可以被公开。两种有用的安排是：

- 在默认图中包含有关命名图的来源信息
- 将命名图中的信息也包括在默认图中。

**示例1**

```turtle
# Default graph
@prefix dc: <http://purl.org/dc/elements/1.1/> .

<http://example.org/bob>    dc:publisher  "Bob" .
<http://example.org/alice>  dc:publisher  "Alice" .
```

```turtle
# Named graph: http://example.org/bob
@prefix foaf: <http://xmlns.com/foaf/0.1/> .

_:a foaf:name "Bob" .
_:a foaf:mbox <mailto:bob@oldcorp.example.org> .
```

```turtle
# Named graph: http://example.org/alice
@prefix foaf: <http://xmlns.com/foaf/0.1/> .

_:a foaf:name "Alice" .
_:a foaf:mbox <mailto:alice@work.example.org> .
```

在此的示例默认图包含两个命名的图的发布者的名称。在此示例中，命名图中的三元组在默认图中不可见。

**示例2**

可以通过图的 RDF合并来合并数据。RDF数据集中图形的一种可能排列是使默认图形为命名图形中某些或所有信息的RDF合并。

在下一个示例中，命名图包含与以前相同的三元组。RDF数据集包括默认图中的已命名图的RDF合并，重新标记空节点以使它们保持不同。

```turtle
# Default graph
@prefix foaf: <http://xmlns.com/foaf/0.1/> .

_:x foaf:name "Bob" .
_:x foaf:mbox <mailto:bob@oldcorp.example.org> .

_:y foaf:name "Alice" .
_:y foaf:mbox <mailto:alice@work.example.org> .
```

```turtle
# Named graph: http://example.org/bob
@prefix foaf: <http://xmlns.com/foaf/0.1/> .

_:a foaf:name "Bob" .
_:a foaf:mbox <mailto:bob@oldcorp.example.org> .
```

```turtle
# Named graph: http://example.org/alice
@prefix foaf: <http://xmlns.com/foaf/0.1/> .

_:a foaf:name "Alice" .
_:a foaf:mbox <mailto:alice@work.example> .
```

在RDF合并中，合并图中的空节点不与正在合并图中的空节点共享。

### 13.2 指定RDF数据集

SPARQL查询可以通过使用`FROM`子句和`FROM NAMED`用于描述RDF数据集的子句来指定要用于匹配的数据集。如果查询提供了这样的数据集描述，则它将代替在查询中未提供任何数据集描述的情况下查询服务使用的任何数据集。还可以在SPARQL协议请求中指定RDF数据集 ，在这种情况下，协议描述会覆盖查询本身中的任何描述。如果数据集描述不被服务接受，则查询服务可以拒绝查询请求。

在`FROM`和`FROM NAMED`关键字允许查询指定引用一个RDF数据集; 它们指示数据集应包括从给定IRI标识的资源表示（即给定IRI参考的绝对形式）中获取的图形。由多个`FROM`and`FROM NAMED`子句得出的数据集为：

- 由RDF合并`FROM`子句中提到的图形组成的默认图形 
- 一组（IRI，图）对，来自每个`FROM NAMED`子句。

如果没有`FROM`子句，但是有一个或多个`FROM NAMED` 子句，则数据集将为默认图包括一个空图。

#### 13.2.1 指定默认图

每个`FROM`子句都包含一个IRI，该IRI指示要用于形成默认图的图。这不会将图作为命名图放入。

在此示例中，RDF数据集包含一个默认图，没有命名图：

```turtle
# Default graph (located at http://example.org/foaf/aliceFoaf)
@prefix  foaf:  <http://xmlns.com/foaf/0.1/> .

_:a  foaf:name     "Alice" .
_:a  foaf:mbox     <mailto:alice@work.example> .
```

```SPARQL
PREFIX foaf: <http://xmlns.com/foaf/0.1/>
SELECT  ?name
FROM    <http://example.org/foaf/aliceFoaf>
WHERE   { ?x foaf:name ?name }
```

| name    |
| ------- |
| "Alice" |

如果查询提供了多个`FROM`子句，并提供了多个IRI来指示默认图，则默认图是从给定IRI标识的资源表示中获得的图的 RDF合并。

#### 13.2.2 指定命名图

查询可以使用该`FROM NAMED`子句为RDF数据集中的命名图提供IRI 。每个IRI用于在RDF数据集中提供一个命名的图。在两个或多个`FROM NAMED`子句中使用相同的出现在数据集中的IRI会得到一个命名图。

```turtle
# Graph: http://example.org/bob
@prefix foaf: <http://xmlns.com/foaf/0.1/> .

_:a foaf:name "Bob" .
_:a foaf:mbox <mailto:bob@oldcorp.example.org> .
```

```turtle
# Graph: http://example.org/alice
@prefix foaf: <http://xmlns.com/foaf/0.1/> .

_:a foaf:name "Alice" .
_:a foaf:mbox <mailto:alice@work.example> .
```

```SPARQL
...
FROM NAMED <http://example.org/alice>
FROM NAMED <http://example.org/bob>
...
```

`FROM NAMED`语法表明IRI标识对应图表，但在一个RDF数据集中的IRI和图之间的关系是间接的。IRI标识资源，并且该资源由图形表示（或更准确地说，由序列化图形的文档表示）。

#### 13.2.3 合并FROM和FROM NAMED

该`FROM`子句和`FROM NAMED`子句可以在同一个查询中使用。

```turtle
# Default graph (located at http://example.org/dft.ttl)
@prefix dc: <http://purl.org/dc/elements/1.1/> .

<http://example.org/bob>    dc:publisher  "Bob Hacker" .
<http://example.org/alice>  dc:publisher  "Alice Hacker" .
```

```turtle
# Named graph: http://example.org/bob
@prefix foaf: <http://xmlns.com/foaf/0.1/> .

_:a foaf:name "Bob" .
_:a foaf:mbox <mailto:bob@oldcorp.example.org> .
```

```turtle
# Named graph: http://example.org/alice
@prefix foaf: <http://xmlns.com/foaf/0.1/> .

_:a foaf:name "Alice" .
_:a foaf:mbox <mailto:alice@work.example.org> .
```

```SPARQL
PREFIX foaf: <http://xmlns.com/foaf/0.1/>
PREFIX dc: <http://purl.org/dc/elements/1.1/>

SELECT ?who ?g ?mbox
FROM <http://example.org/dft.ttl>
FROM NAMED <http://example.org/alice>
FROM NAMED <http://example.org/bob>
WHERE
{
   ?g dc:publisher ?who .
   GRAPH ?g { ?x foaf:mbox ?mbox }
}
```

此查询的RDF数据集包含一个默认图和两个命名图。在`GRAPH`下面的关键字进行说明。

构造数据集所需的动作并非仅由数据集描述来确定。如果通过使用两个`FROM`子句或一个`FROM`子句和一个 `FROM NAMED`子句在数据集描述中两次给定IRI ，则它不会假定进行了一次或两次准确的尝试以获得与IRI关联的RDF图。因此，无法对从数据集描述中的两次出现而获得的三元组中的空节点身份进行任何假设。通常，不能对图的等价性做任何假设。

### 13.3 查询数据集

查询图形集合时，`GRAPH`关键字用于将模式与**命名图**进行匹配。`GRAPH`可以提供一个IRI来选择一个图或使用一个变量，该变量将在查询的RDF数据集中所有命名图的IRI范围内。

使用`GRAPH`更改活动图以在查询的该部分中匹配图模式。<u>除了使用`GRAPH`，还可以使用默认图进行匹配。</u>

示例中将使用以下两个图形：

```turtle
# Named graph: http://example.org/foaf/aliceFoaf
@prefix  foaf:     <http://xmlns.com/foaf/0.1/> .
@prefix  rdf:      <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix  rdfs:     <http://www.w3.org/2000/01/rdf-schema#> .

_:a  foaf:name     "Alice" .
_:a  foaf:mbox     <mailto:alice@work.example> .
_:a  foaf:knows    _:b .

_:b  foaf:name     "Bob" .
_:b  foaf:mbox     <mailto:bob@work.example> .
_:b  foaf:nick     "Bobby" .
_:b  rdfs:seeAlso  <http://example.org/foaf/bobFoaf> .

<http://example.org/foaf/bobFoaf>
     rdf:type      foaf:PersonalProfileDocument .
```

```turtle
# Named graph: http://example.org/foaf/bobFoaf
@prefix  foaf:     <http://xmlns.com/foaf/0.1/> .
@prefix  rdf:      <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix  rdfs:     <http://www.w3.org/2000/01/rdf-schema#> .

_:z  foaf:mbox     <mailto:bob@work.example> .
_:z  rdfs:seeAlso  <http://example.org/foaf/bobFoaf> .
_:z  foaf:nick     "Robert" .

<http://example.org/foaf/bobFoaf>
     rdf:type      foaf:PersonalProfileDocument .
```

#### 13.3.1 访问图名

下面的查询将图模式与数据集中的每个命名图进行匹配，并形成解决方案，该解决方案有一个绑定到要匹配图的IRI的变量`src`。图模式与活动图匹配，该活动图是数据集中的每个命名图。

```SPARQL
PREFIX foaf: <http://xmlns.com/foaf/0.1/>

SELECT ?src ?bobNick
FROM NAMED <http://example.org/foaf/aliceFoaf>
FROM NAMED <http://example.org/foaf/bobFoaf>
WHERE
  {
    GRAPH ?src
    { ?x foaf:mbox <mailto:bob@work.example> .
      ?x foaf:nick ?bobNick
    }
  }
```

查询结果给出了找到信息的图形的名称以及Bob昵称的值：

| src                                  | bobNick  |
| ------------------------------------ | -------- |
| \<http://example.org/foaf/aliceFoaf> | "Bobby"  |
| \<http://example.org/foaf/bobFoaf>   | "Robert" |

#### 13.3.2 通过图IRI限制

查询可以通过提供图IRI来限制应用于特定图的匹配。这会将活动图设置为<u>由IRI命名的图</u>。该查询查找给定图`http://example.org/foaf/bobFoaf`中的Bob的昵称。

```SPARQL
PREFIX foaf: <http://xmlns.com/foaf/0.1/>
PREFIX data: <http://example.org/foaf/>

SELECT ?nick
FROM NAMED <http://example.org/foaf/aliceFoaf>
FROM NAMED <http://example.org/foaf/bobFoaf>
WHERE
  {
     GRAPH data:bobFoaf {
         ?x foaf:mbox <mailto:bob@work.example> .
         ?x foaf:nick ?nick }
  }
```

得到一个单一解决方案

| nick     |
| -------- |
| "Robert" |

#### 13.3.3 限制可能的图IRIs

`GRAPH`子句中使用的变量也可以用在另一个`GRAPH`子句中，或用于与数据集中的默认图形匹配的图形模式中。

下面的查询使用带有IRI`http://example.org/foaf/aliceFoaf`的图形来查找Bob的个人文档；然后，它针对该图匹配另一个模式。第二个`GRAPH`子句中的模式查找与第一个GRAPH子句相同的邮箱（由变量mbox给定）的人（变量whom）的空节点（变量w），因为用于匹配Alice的FOAF文件中变量`whom`的空节点是与个人文件中的空白节点不同（它们在不同的图中）。

```SPARQL
PREFIX  data:  <http://example.org/foaf/>
PREFIX  foaf:  <http://xmlns.com/foaf/0.1/>
PREFIX  rdfs:  <http://www.w3.org/2000/01/rdf-schema#>

SELECT ?mbox ?nick ?ppd
FROM NAMED <http://example.org/foaf/aliceFoaf>
FROM NAMED <http://example.org/foaf/bobFoaf>
WHERE
{
  GRAPH data:aliceFoaf
  {
    ?alice foaf:mbox <mailto:alice@work.example> ;
           foaf:knows ?whom .
    ?whom  foaf:mbox ?mbox ;
           rdfs:seeAlso ?ppd .
    ?ppd  a foaf:PersonalProfileDocument .
  } .
  GRAPH ?ppd
  {
      ?w foaf:mbox ?mbox ;
         foaf:nick ?nick
  }
}
```

| mbox                       | nick     | ppd                                |
| -------------------------- | -------- | ---------------------------------- |
| \<mailto:bob@work.example> | "Robert" | \<http://example.org/foaf/bobFoaf> |

Alice的FOAF文件中给Bob的`nick`的任何三元组都不会为Bob提供一个昵称，因为涉及变量`nick`的模式受限于`ppd`特定的个人文档。

#### 13.3.4 命名和默认图

查询模式可以涉及默认图和命名图。在此示例中，聚合器在两种不同的情况下已读入Web资源。每次将图读入聚合器时，本地系统都会为其提供IRI。图表几乎相同，但“ Bob”的电子邮件地址已更改。

在此示例中，默认图用于记录来源信息，而实际读取的RDF数据保存在两个单独的图中，系统为每个图赋予不同的IRI。RDF数据集包含两个命名的图以及有关它们的信息。

RDF数据集：

```turtle
# Default graph
@prefix dc: <http://purl.org/dc/elements/1.1/> .
@prefix g:  <tag:example.org,2005-06-06:> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .

g:graph1 dc:publisher "Bob" .
g:graph1 dc:date "2004-12-06"^^xsd:date .

g:graph2 dc:publisher "Bob" .
g:graph2 dc:date "2005-01-10"^^xsd:date .
```

```turtle
# Graph: locally allocated IRI: tag:example.org,2005-06-06:graph1
@prefix foaf: <http://xmlns.com/foaf/0.1/> .

_:a foaf:name "Alice" .
_:a foaf:mbox <mailto:alice@work.example> .

_:b foaf:name "Bob" .
_:b foaf:mbox <mailto:bob@oldcorp.example.org> .
```

```turtle
# Graph: locally allocated IRI: tag:example.org,2005-06-06:graph2
@prefix foaf: <http://xmlns.com/foaf/0.1/> .

_:a foaf:name "Alice" .
_:a foaf:mbox <mailto:alice@work.example> .

_:b foaf:name "Bob" .
_:b foaf:mbox <mailto:bob@newcorp.example.org> .
```

该查询查找电子邮件地址，其中详细说明了该人的姓名和发现该信息的日期。

```SPARQL
PREFIX foaf: <http://xmlns.com/foaf/0.1/>
PREFIX dc:   <http://purl.org/dc/elements/1.1/>

SELECT ?name ?mbox ?date
WHERE
  {  ?g dc:publisher ?name ;
        dc:date ?date .
    
    GRAPH ?g
      { 
      	?person foaf:name ?name ; 
      			foaf:mbox ?mbox 
      }
  }
```

结果显示Bob的邮箱地址已经改变

| name  | mbox                              | date                   |
| ----- | --------------------------------- | ---------------------- |
| "Bob" | \<mailto:bob@oldcorp.example.org> | "2004-12-06"^^xsd:date |
| "Bob" | \<mailto:bob@newcorp.example.org> | "2005-01-10"^^xsd:date |

## 14 基本联合查询

本文档包含SPARQL联合身份验证扩展的语法。

在文档SPARQL 1.1联合查询中定义了此功能 。

## 15 解决方案序列和修饰符

查询模式生成无序的解决方案集合，每个解决方案都是从变量到RDF术语的部分函数。然后将这些解决方案视为一个序列（解决方案序列），最初没有特定的顺序；然后，可以应用任何序列修饰符来创建另一个序列。最后，后面的序列用于生成SPARQL查询表单的结果之一 。

一个**解决方案序列修饰符**是以下之一：

- Order修饰符：按顺序**排列**解决方案
- Projection修饰符：**选择**某些变量
- Distinct修饰符：确保序列中的解决方案是**唯一**的
- Reduced修饰语：允许**消除**一些不唯一的解决方案
- Offset修改器：控制解决方案从整体解决方案序列中的**位置**开始
- Limit修饰符：限制解决方案**数量**

修饰符按上面列出的顺序应用。

### 15.1 ORDER BY

该`ORDER BY`语句建立了解决方案序列的顺序。

该`ORDER BY`子句之后是一系列顺序比较器，由一个表达式和一个可选的顺序修饰符（`ASC()`或`DESC()`）组成。每个排序比较器可以是升序（由`ASC()`修饰符表示或没有修饰符表示）或降序（由`DESC()`修饰符表示）。

```SPARQL
PREFIX foaf:    <http://xmlns.com/foaf/0.1/>

SELECT ?name
WHERE { ?x foaf:name ?name }
ORDER BY ?name
```

```SPARQL
PREFIX     :    <http://example.org/ns#>
PREFIX foaf:    <http://xmlns.com/foaf/0.1/>

SELECT ?name
WHERE { ?x foaf:name ?name ; 
			:empId ?emp 
		}
ORDER BY DESC(?emp)
```

```SPARQL
PREFIX     :    <http://example.org/ns#>
PREFIX foaf:    <http://xmlns.com/foaf/0.1/>

SELECT ?name
WHERE { ?x foaf:name ?name ; 
			:empId ?emp 
		}
ORDER BY ?name DESC(?emp)
```

“<”操作符定义了`numerics`，`simple literals`，`xsd:strings`，`xsd:booleans` 和`xsd:dateTimes`的相对顺序。将他们作为`simple literals`比较IRIs，可以对IRI进行排序。

SPARQL还修复了某些RDF术语之间的顺序，否则这些顺序将不会被排序：

1. （最低）在此解决方案中未分配任何值给变量或表达式。
2. 空节点
3. IRIs
4. RDF文字

普通文字低于带有`xsd:string`相同词汇形式类型的RDF文字。

SPARQL并未定义所有可能的RDF术语的总顺序。以下是一些未定义相对顺序的术语对的示例：

- “ a”和“ a” @en_gb（简单的文字和带有语言标签的文字）
- “ a” @en_gb和“ b” @en_gb（两个带语言标签的文字）
- “ a”和“ 1” ^^ xsd：integer（简单的文字和具有受支持的数据类型的文字）
- “ 1” ^^ my：integer和“ 2” ^^ my：integer（两种不受支持的数据类型）
- “ 1” ^^ xsd：integer和“ 2” ^^ my：integer（受支持的数据类型和不受支持的数据类型）

变量绑定列表按升序排列：

|                  RDF Term                   |                            Reason                            |
| :-----------------------------------------: | :----------------------------------------------------------: |
|                                             |                Unbound results sort earliest.                |
|                    `_:z`                    |                 Blank nodes follow unbound.                  |
|                    `_:a`                    |        There is no relative ordering of blank nodes.         |
|       `<http://script.example/Latin>`       |                   IRIs follow blank nodes.                   |
|     `<http://script.example/Кириллица>`     | The character in the 23rd position, "К", has a unicode codepoint 0x41A, which is higher than 0x4C ("L"). |
|       `<http://script.example/漢字>`        | The character in the 23rd position, "漢", has a unicode codepoint 0x6F22, which is higher than 0x41A ("К"). |
|       `"http://script.example/Latin"`       |                 Simple literals follow IRIs.                 |
| `"http://script.example/Latin"^^xsd:string` |             xsd:strings follow simple literals.              |

通过将解决方案绑定代入表达式并将它们与“ <”运算符进行比较，可以建立两个解决方案相对于排序比较器的升序。降序与升序相反。

两个解的相对顺序是两个解相对于序列中第一个排序比较器的相对顺序。对于解决方案绑定的替换产生相同RDF项的解决方案，其顺序是两个解决方案相对于下一个排序比较器的相对顺序。如果两个解决方案未评估的顺序表达式产生不同的RDF项，则两个解决方案的相对顺序不确定。

排序解决方案序列始终会导致序列中包含相同数量的解决方案。

`ORDER BY`在`CONSTRUCT`或 `DESCRIBE`查询的解决方案序列上使用没有直接效果，因为仅`SELECT`返回结果序列。与`LIMIT`和结合使用`OFFSET`， `ORDER BY`可用于返回从解决方案序列的不同片段生成的结果。一个`ASK`查询不包括`ORDER BY`，`LIMIT`或`OFFSET`。

### 15.2 Projection

解决方案序列可以转换为仅包含变量子集的序列。对于序列中的每个解决方案，使用`SELECT`查询形式使用指定的变量选择来形成新的解决方案。

以下示例显示了一个查询，该查询仅使用FOAF属性提取RDF图中描述的人员的姓名。

```turtle
@prefix foaf:        <http://xmlns.com/foaf/0.1/> .

_:a  foaf:name       "Alice" .
_:a  foaf:mbox       <mailto:alice@work.example> .

_:b  foaf:name       "Bob" .
_:b  foaf:mbox       <mailto:bob@work.example> .
```

```SPARQL
PREFIX foaf:       <http://xmlns.com/foaf/0.1/>
SELECT ?name
WHERE
 { ?x foaf:name ?name }
```

| name    |
| ------- |
| "Bob"   |
| "Alice" |

### 15.3 重复的解决方案

没有`DISTINCT`或`REDUCED`修饰符的解决方案序列将保留重复的解决方案。

**数据**

```turtle
@prefix  foaf:  <http://xmlns.com/foaf/0.1/> .

_:x    foaf:name   "Alice" .
_:x    foaf:mbox   <mailto:alice@example.com> .

_:y    foaf:name   "Alice" .
_:y    foaf:mbox   <mailto:asmith@example.com> .

_:z    foaf:name   "Alice" .
_:z    foaf:mbox   <mailto:alice.smith@example.com> .
```

**查询**

```SPARQL
PREFIX foaf:    <http://xmlns.com/foaf/0.1/>
SELECT ?name WHERE { ?x foaf:name ?name }
```

**结果**

| name    |
| ------- |
| "Alice" |
| "Alice" |
| "Alice" |

修饰符`DISTINCT`和`REDUCED`影响是否在查询结果中包括重复项。

#### 15.3.1 DISTINCT

该`DISTINCT`解决方案修改消除重复的解决方案。该查询仅返回一个将相同变量绑定到相同RDF术语的解决方案。

**查询**

```SPARQL
PREFIX foaf:    <http://xmlns.com/foaf/0.1/>
SELECT DISTINCT ?name WHERE { ?x foaf:name ?name }
```

| name    |
| ------- |
| "Alice" |

请注意，按照解决方案序列修饰符的顺序，<u>在应用限制或偏移之前</u>，将消除重复项。

#### 15.3.2 REDUCED

尽管`DISTINCT`修饰符可确保从解决方案集中消除重复的解决方案，但`REDUCED`只需允许消除它们即可。`REDUCED`解决方案集中的变量绑定的任何集合的基数至少为一个，且不大于不包含`DISTINCT`或`REDUCED`修饰符的解决方案集的基数`使用DISTINCT返回的数量<= x <=不使用DISTINCT返回的属性`。例如，使用上面的数据，查询

```SPARQL
PREFIX foaf:    <http://xmlns.com/foaf/0.1/>
SELECT REDUCED ?name WHERE { ?x foaf:name ?name }
```

可能有一个，两个（如下所示）或三个解决方案：

| name    |
| ------- |
| "Alice" |
| "Alice" |

### 15.4 OFFSET

`OFFSET`使生成的解决方案在指定数量的解决方案之后开始。一个零`OFFSET`没有影响。

除非使用`ORDER BY`可以使顺序可预测，否则使用` LIMIT`和`OFFSET`选择查询解决方案的不同子集将无用。

```SPARQL
PREFIX foaf:    <http://xmlns.com/foaf/0.1/>

SELECT  ?name
WHERE   { ?x foaf:name ?name }
ORDER BY ?name
LIMIT   5
OFFSET  10								//从第10个解决方案开始
```

### 15.5 LIMIT

该`LIMIT`子句对返回的解决方案数设置了上限。如果`OFFSET`应用后实际解决方案的数量大于限制，那么最多将返回解决方案的限制数量。

```SPARQL
PREFIX foaf:    <http://xmlns.com/foaf/0.1/>

SELECT ?name
WHERE { ?x foaf:name ?name }
LIMIT 20
```

一个`LIMIT`为0将导致不返回任何结果。限制可能不为负。

## 16 查询形式

SPARQL有四种查询形式。这些查询表单使用从模式匹配到表单结果集或RDF图的解决方案。查询形式为：

> - SELECT
>
>   返回查询模式匹配中绑定的所有变量或变量的子集。
>
> - CONSTRUCT
>
>   返回通过将变量替换为一组三元组模板而构造的RDF**图**。
>
> - ASK
>
>   返回一个**布尔值**，指示查询模式是否匹配。
>
> - DESCRIBE
>
>   返回**描述**找到的资源的RDF图。

诸如 SPARQL 1.1查询结果JSON格式， SPARQL查询结果XML格式或 SPARQL 1.1查询结果CSV和TSV格式的格式可用于序列化` SELECT`查询的结果集或查询`ASK`的布尔结果。

### 16.1 SELECT

结果的`SELECT`形式直接返回变量及其绑定。它结合了将所需变量投影的操作与在查询解决方案中引入新的变量绑定的功能。

#### 16.1.1 投影

当在`SELECT`子句中给出变量名列表时，将返回特定的变量及其绑定。语法 `SELECT *`是一种缩写，它选择查询中该点作用域的所有变量。它排除了仅用`FILTER`于中的变量，在 `MINUS`的右侧的变量，并考虑了子查询。

<u>`SELECT *`仅当查询没有`GROUP BY`子句时才允许使用</u>。

```turtle
@prefix  foaf:  <http://xmlns.com/foaf/0.1/> .

_:a    foaf:name   "Alice" .
_:a    foaf:knows  _:b .
_:a    foaf:knows  _:c .

_:b    foaf:name   "Bob" .

_:c    foaf:name   "Clare" .
_:c    foaf:nick   "CT" .	 
```

```SPARQL
PREFIX foaf:    <http://xmlns.com/foaf/0.1/>
SELECT ?nameX ?nameY ?nickY
WHERE
  { ?x foaf:knows ?y ;
       foaf:name ?nameX .
    ?y foaf:name ?nameY .
    OPTIONAL { ?y foaf:nick ?nickY }
  }
```

| nameX   | nameY   | nickY |
| ------- | ------- | ----- |
| "Alice" | "Bob"   |       |
| "Alice" | "Clare" | "CT"  |

结果集可以通过本地API访问，也可以序列化为JSON，XML，CSV或TSV。

JSON格式

```json
{
  "head": {
    "vars": [ "nameX" , "nameY" , "nickY" ]
  } ,
  "results": {
    "bindings": [
      {
        "nameX": { "type": "literal" , "value": "Alice" } ,
        "nameY": { "type": "literal" , "value": "Bob" }
      } ,
      {
        "nameX": { "type": "literal" , "value": "Alice" } ,
        "nameY": { "type": "literal" , "value": "Clare" } ,
        "nickY": { "type": "literal" , "value": "CT" }
      }
    ]
  }
}
```

XML格式

```xml
<?xml version="1.0"?>
<sparql xmlns="http://www.w3.org/2005/sparql-results#">
  <head>
    <variable name="nameX"/>
    <variable name="nameY"/>
    <variable name="nickY"/>
  </head>
  <results>
    <result>
      <binding name="nameX">
        <literal>Alice</literal>
      </binding>
      <binding name="nameY">
        <literal>Bob</literal>
      </binding>
   </result>
    <result>
      <binding name="nameX">
        <literal>Alice</literal>
      </binding>
      <binding name="nameY">
        <literal>Clare</literal>
      </binding>
      <binding name="nickY">
        <literal>CT</literal>
      </binding>
    </result>
  </results>
</sparql>
```

#### 16.1.2 SELECT表达式

除了从模式匹配中选择哪些变量包括在结果中之外，SELECT子句还可以引入**新变量**。SELECT表达式中的分配规则与BIND中的分配规则相同。该表达式将查询解决方案中已经存在的或早先在SELECT子句中定义的变量绑定进行组合，以在查询解决方案中产生绑定。

`(expr AS v)`的作用域同样适用。在 `SELECT`表达式中，变量可以稍后在同一`SELECT`子句中的表达式中使用，并且可以在同一`SELECT`子句中不用再次分配。

**数据**

```turtle
@prefix dc:   <http://purl.org/dc/elements/1.1/> .
@prefix :     <http://example.org/book/> .
@prefix ns:   <http://example.org/ns#> .

:book1  dc:title  "SPARQL Tutorial" .
:book1  ns:price  42 .
:book1  ns:discount 0.2 .

:book2  dc:title  "The Semantic Web" .
:book2  ns:price  23 .
:book2  ns:discount 0.25 .
```

**查询**

```SPARQL
PREFIX  dc:  <http://purl.org/dc/elements/1.1/>
PREFIX  ns:  <http://example.org/ns#>
SELECT  ?title (?p*(1-?discount) AS ?price)
{ ?x ns:price ?p .
  ?x dc:title ?title . 
  ?x ns:discount ?discount 
}
```

| title              | price |
| ------------------ | ----- |
| "The Semantic Web" | 17.25 |
| "SPARQL Tutorial"  | 33.6  |

如果在语法上同一个SELECT子句中较早地引入了新变量，则它们也可以在表达式中使用：

```SPARQL
PREFIX  dc:  <http://purl.org/dc/elements/1.1/>
PREFIX  ns:  <http://example.org/ns#>
SELECT  ?title (?p AS ?fullPrice) (?fullPrice*(1-?discount) AS ?customerPrice)
{  ?x ns:price ?p .
   ?x dc:title ?title . 
   ?x ns:discount ?discount 
}
```

**结果**

| title              | fullPrice | customerPrice |
| ------------------ | --------- | ------------- |
| "The Semantic Web" | 23        | 17.25         |
| "SPARQL Tutorial"  | 42        | 33.6          |

### 16.2 CONSTRUCT

所述`CONSTRUCT`查询形式，返回由图形模板中指定一个单一的RDF**图**。结果是通过获取解决方案序列中的每个查询解决方案，替换图模板中的变量，然后通过设置**并集**将三元组合并为单个RDF图而形成的RDF图。

如果任何此类实例化产生包含未绑定变量或非法RDF构造的三元组（例如，位于主语或谓语位置的文字），则该三元组将不包含在输出RDF图中。图模板<u>可以包含</u>无变量的三元组（称为基础三元组或显式三元组），并且这些变量也会出现在由CONSTRUCT查询表单返回的输出RDF图形中。

```turtle
@prefix  foaf:  <http://xmlns.com/foaf/0.1/> .

_:a    foaf:name   "Alice" .
_:a    foaf:mbox   <mailto:alice@example.org> .
```

```SPARQL
PREFIX foaf:    <http://xmlns.com/foaf/0.1/>
PREFIX vcard:   <http://www.w3.org/2001/vcard-rdf/3.0#>
CONSTRUCT   { <http://example.org/person#Alice> vcard:FN ?name }
WHERE       { ?x foaf:name ?name }
```

根据FOAF信息创建vcard属性：

```turtle
@prefix vcard: <http://www.w3.org/2001/vcard-rdf/3.0#> .

<http://example.org/person#Alice> vcard:FN "Alice" .
```

#### 16.2.1 带有空节点的模板

<u>模板可以创建包含空节点的RDF图</u>。空节点标签的作用域是每种解决方案的模板。如果同一标签在模板中出现两次，则将为每个查询解决方案创建一个空节点，对于由不同查询解决方案生成的三元组，将存在不同的空节点。

```turtle
@prefix  foaf:  <http://xmlns.com/foaf/0.1/> .

_:a    foaf:givenname   "Alice" .
_:a    foaf:family_name "Hacker" .

_:b    foaf:firstname   "Bob" .
_:b    foaf:surname     "Hacker" .
```

```SPARQL
PREFIX foaf:    <http://xmlns.com/foaf/0.1/>
PREFIX vcard:   <http://www.w3.org/2001/vcard-rdf/3.0#>

CONSTRUCT { ?x  vcard:N _:v .
            _:v vcard:givenName ?gname .
            _:v vcard:familyName ?fname }
WHERE
 {
    { ?x foaf:firstname ?gname } UNION  { ?x foaf:givenname   ?gname } .
    { ?x foaf:surname   ?fname } UNION  { ?x foaf:family_name ?fname } .
 }
```

创建与FOAF信息相对应的vcard属性：

```turtle
@prefix vcard: <http://www.w3.org/2001/vcard-rdf/3.0#> .

_:v1 vcard:N         _:x .
_:x vcard:givenName  "Alice" .
_:x vcard:familyName "Hacker" .

_:v2 vcard:N         _:z .
_:z vcard:givenName  "Bob" .
_:z vcard:familyName "Hacker" .
```

在模板中使用变量`x`（此示例中该变量将绑定到数据中带有标签`_:a`和`_:b`的空节点），导致在所得RDF图形中使用不同的空白节点标签（`_:v1`和`_:v2`）。

#### 16.2.2 在RDF数据集中访问图

使用`CONSTRUCT`，可以从目标RDF数据集中提取部分或全部图形。第一个示例返回带有IRI标签`http://example.org/aGraph`的图（如果它在数据集中）；否则，将返回一个空图。

```SPARQL
CONSTRUCT { ?s ?p ?o } 
WHERE { GRAPH <http://example.org/aGraph> { ?s ?p ?o } . }
```

对图形的访问可以<u>以其他信息为条件</u>。例如，如果默认图包含有关数据集中命名图的元数据，则类似以下查询可以基于有关命名图的信息提取一个图：

```SPARQL
PREFIX  dc: <http://purl.org/dc/elements/1.1/>
PREFIX app: <http://example.org/ns#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

CONSTRUCT { ?s ?p ?o } WHERE
 {
   GRAPH ?g { ?s ?p ?o } .
   ?g dc:publisher <http://www.w3.org/> .
   ?g dc:date ?date .
   FILTER ( app:customDate(?date) > "2005-02-28T00:00:00Z"^^xsd:dateTime ) .
 }
```

其中`app:customDate`标识 扩展功能，以将日期格式转换为`xsd:dateTime` RDF术语。

#### 16.2.3 解决方案修饰符和CONSTRUCT

查询的解决方案修饰符会影响查询`CONSTRUCT`的结果 。在此示例中，`CONSTRUCT`模板的输出图仅由图模式匹配中的两个解决方案形成。该查询将输出一个图表，其中包含按点击数排名的前两个站点的人员名称。RDF图中的三元组不排序。

```turtle
@prefix foaf: <http://xmlns.com/foaf/0.1/> .
@prefix site: <http://example.org/stats#> .

_:a foaf:name "Alice" .
_:a site:hits 2349 .

_:b foaf:name "Bob" .
_:b site:hits 105 .

_:c foaf:name "Eve" .
_:c site:hits 181 .
```

```SPARQL
PREFIX foaf: <http://xmlns.com/foaf/0.1/>
PREFIX site: <http://example.org/stats#>

CONSTRUCT { [] foaf:name ?name }
WHERE
{ [] foaf:name ?name ;
     site:hits ?hits .
}
ORDER BY desc(?hits)
LIMIT 2
```

```turtle
@prefix foaf: <http://xmlns.com/foaf/0.1/> .
_:x foaf:name "Alice" .
_:y foaf:name "Eve" .
```

#### 16.2.3 CONSTRUCT WHERE

对于模板和模式相同且模式仅是基本图形模式（不允许`FILTER`和复杂图形模式）的情况，提供了CONSTRUCT查询格式的<u>缩写</u>形式。关键字`WHERE`是缩写形式。

以下两个查询是相同的；第一个是第二个的简短形式。

```SPARQL
PREFIX foaf: <http://xmlns.com/foaf/0.1/>
CONSTRUCT WHERE { ?x foaf:name ?name } 
```

```SPARQL
PREFIX foaf: <http://xmlns.com/foaf/0.1/>

CONSTRUCT { ?x foaf:name ?name } 
WHERE
{ ?x foaf:name ?name }
```

### 16.3 ASK

应用程序可以使用该`ASK`表单来测试查询模式是否具有解决方案。没有返回有关可能的查询解决方案的信息，无论是否存在解决方案。

```turtle
@prefix foaf:       <http://xmlns.com/foaf/0.1/> .

_:a  foaf:name       "Alice" .
_:a  foaf:homepage   <http://work.example.org/alice/> .

_:b  foaf:name       "Bob" .
_:b  foaf:mbox       <mailto:bob@work.example> .
```

```SPARQL
PREFIX foaf:    <http://xmlns.com/foaf/0.1/>
ASK  { ?x foaf:name  "Alice" }
```

```
true
```

结果集的XML格式

```xml
<?xml version="1.0"?>
<sparql xmlns="http://www.w3.org/2005/sparql-results#">
  <head></head>
  <boolean>true</boolean>
</sparql>
```

在相同的数据上，以下内容不返回匹配项，因为Alice的`mbox`没有被提到 。

```SPARQL
PREFIX foaf:    <http://xmlns.com/foaf/0.1/>
ASK  { ?x foaf:name  "Alice" ;
          foaf:mbox  <mailto:alice@work.example> }
```

```
false
```

### 16.4 DESCRIBE

该`DESCRIBE`表单返回一个包含RDF数据有关资源的单个结果RDF图。此数据不是由SPARQL查询指定的，查询客户端需要知道数据源中RDF的结构，而是由SPARQL查询处理器确定。查询模式用于创建结果集。该`DESCRIBE`形式采用了解决方案中标识的每个资源，以及由IRI直接命名的任何资源，并通过采用“描述”来组装单个RDF图，该“描述”可以来自任何可用信息，包括目标RDF数据集。该描述由查询服务确定。语法`DESCRIBE *` 是一种缩写，描述查询中的所有变量。

#### 16.4.1 显式IRI

该`DESCRIBE`语句本身可以采取IRIs识别资源。最简单的`DESCRIBE`查询只是该`DESCRIBE` 子句中的IRI ：

```turtle
DESCRIBE <http://example.org/>
```

#### 16.4.2 识别资源

还可以从绑定到结果集中查询变量的资源中获取要描述的资源。这使资源的描述无论是由IRI还是由数据集中的空白节点标识的：

```SPARQL
PREFIX foaf:   <http://xmlns.com/foaf/0.1/>
DESCRIBE ?x
WHERE    { ?x foaf:mbox <mailto:alice@org> }
```

该属性`foaf:mbox`被定义为FOAF词汇表中的逆函数属性。如果这样处理，此查询将返回有关最多一个人的信息。但是，如果查询模式具有多个解决方案，则每个解决方案的RDF数据都是所有RDF图描述的并集。

```SPARQL
PREFIX foaf:   <http://xmlns.com/foaf/0.1/>
DESCRIBE ?x
WHERE    { ?x foaf:name "Alice" }
```

可以给出多个IRI或变量：

```SPARQL
PREFIX foaf:   <http://xmlns.com/foaf/0.1/>
DESCRIBE ?x ?y <http://example.org/>
WHERE    {?x foaf:knows ?y}
```

#### 16.4.3 资源描述

返回的RDF由信息发布者确定。它可能是服务认为与正在描述的资源有关的信息。它可能包含有关其他资源的信息：例如，一本书的RDF数据可能还包含有关作者的详细信息。

一个简单的查询，例如

```SPARQL
PREFIX ent:  <http://org.example.com/employees#>
DESCRIBE ?x WHERE { ?x ent:employeeId "1234" }
```

可能会返回员工说明和其他一些可能有用的详细信息：

```turtle
@prefix foaf:   <http://xmlns.com/foaf/0.1/> .
@prefix vcard:  <http://www.w3.org/2001/vcard-rdf/3.0> .
@prefix exOrg:  <http://org.example.com/employees#> .
@prefix rdf:    <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix owl:    <http://www.w3.org/2002/07/owl#>

_:a     exOrg:employeeId    "1234" ;
       
        foaf:mbox_sha1sum   "bee135d3af1e418104bc42904596fe148e90f033" ;
        vcard:N
         [ vcard:Family       "Smith" ;
           vcard:Given        "John"  ] .

foaf:mbox_sha1sum  rdf:type  owl:InverseFunctionalProperty .
```

其中包括vcard词汇表vcard：N的闭合空节点 。决定返回什么信息的其他可能机制包括“简明描述” 。

对于诸如FOAF之类的词汇（其中资源通常是空节点），返回足够的信息来标识节点（例如InverseFunctionalProperty `foaf:mbox_sha1sum`）以及诸如名称和其他详细信息之类的信息将是适当的。在示例中，返回了与`WHERE`子句的匹配，但这不是必需的。

