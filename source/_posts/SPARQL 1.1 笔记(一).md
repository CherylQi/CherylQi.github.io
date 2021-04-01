---
title: SPARQL 1.1 笔记(一)
date: 2021-03-21 11:09:52
tags: 语义Web
---

# SPARQL 1.1

## 1 引言

RDF是一种有向的、带标签的图形数据格式，用于表示Web中的信息。RDF通常用于表示个人信息、社交网络、有关数字工件的元数据，以及提供一种跨不同信息源的集成方式。本规范定义了RDF的SPARQL查询语言的语法和语义。

用于RDF的SPARQL查询语言旨在满足RDF数据访问用例和需求[UCNR]和SPARQL新功能和原理[UCNR2]中RDF数据访问工作组确定的用例和需求。

<!--more-->

### 1.1 文档大纲

### 1.2 文档约定

#### 1.2.1 命名空间

在本文档中，除非另有说明，否则示例假定以下命名空间前缀绑定：

| Prefix  | IRI                                           |
| ------- | --------------------------------------------- |
| `rdf:`  | `http://www.w3.org/1999/02/22-rdf-syntax-ns#` |
| `rdfs:` | `http://www.w3.org/2000/01/rdf-schema#`       |
| `xsd:`  | `http://www.w3.org/2001/XMLSchema#`           |
| `fn:`   | `http://www.w3.org/2005/xpath-functions#`     |
| `sfn:`  | `http://www.w3.org/ns/sparql#`                |

#### 1.2.2 数据描述

本文档使用Turtle[Turtle]数据格式显示每个三元组。Turtle允许使用前缀缩写IRIs：

```turtle
@prefix dc:   <http://purl.org/dc/elements/1.1/> .
@prefix :     <http://example.org/book/> .
:book1  dc:title  "SPARQL Tutorial" .
```

#### 1.2.3 结果描述

结果集以表格形式显示。

| x       | y                    | z    |
| ------- | -------------------- | ---- |
| "Alice" | `<http://example/a>` |      |

“绑定”是一对（变量， RDF术语(IRIs, 文字和空节点)）。在该结果集中，有三个变量： `x`，`y`和`z`（显示为列标题）。每个结果在表格主体中显示为一行。在这里，有一个解决方案，其中将变量`x`绑定到`"Alice"`，将变量 `y`绑定到`<http://example/a>`，而变量`z` 不被绑定。在解决方案中不需要绑定变量。

#### 1.2.4 术语

SPARQL语言包括IRI，IRI是RDF URI引用的子集，它省略了空格。请注意，SPARQL查询中的所有IRI都是绝对的。它们可能包含也可能不包含片段标识符。IRI包括URI和URL。SPARQL语法中的缩写形式（相对IRI和前缀名称）解析以生成绝对IRI。

以下术语在RDF概念和抽象语法中定义， 并在SPARQL中使用：

- IRI（对应于概念和抽象语法术语“ `RDF URI reference`”）
- 文字
- 词汇形式
- 普通文字(带有可选的语言标签)
- 语言标签
- 类型文字(带有数据类型)
- 数据类型IRI（对应于概念和抽象语法术语“ `datatype URI`”）
- 空节点

另外，我们定义以下术语：

- RDF术语，包括IRI，空节点和文字
- 简单文字，涵盖了没有语言标签或数据类型IRI的文字

## 2 简单的查询

大多数形式的SPARQL查询都包含一组称为*基本图模式*的三元组模式。三元组模式类似于RDF三元组，不同之处在于<u>主语，谓语和宾语中的每一个都可以是变量。</u>**当来自该子图的RDF术语可以代替变量并且结果是与该子图等效的RDF图时，基本图模式会*匹配*该RDF数据的子图。**

### 2.1 写一个简单的查询

下面的示例显示了一个SPARQL查询，该查询从给定的数据图中查找一本书的书名。查询由两部分组成：`SELECT`子句标识要在查询结果中显示的变量，并且该`WHERE`子句提供了与数据图匹配的基本图模式。在此示例中，基本图形模式由带有宾语位置为单个变量`?title`的单个三元组模式组成。

**数据**

```turtle
<http://example.org/book/book1><http://purl.org/dc/elements/1.1/title>"SPARQL Tutorial" .
```

**查询**

```SPARQL
SELECT ?title
WHERE
{
   //基本图模式
  <http://example.org/book/book1> <http://purl.org/dc/elements/1.1/title> ?title .
}
```

这个查询在数据中返回一个解决方案：

**结果集**

| "title"           |
| ----------------- |
| "SPARQL Tutorial" |

### 2.2  多匹配

查询的结果是一个解决方案序列，对应于查询的图模式与数据匹配的方式。一个查询可能有零个，一个或多个解决方案。

**数据**

```turtle
@prefix foaf:  <http://xmlns.com/foaf/0.1/> .

_:a  foaf:name   "Johnny Lee Outlaw" .
_:a  foaf:mbox   <mailto:jlow@example.com> .
_:b  foaf:name   "Peter Goodguy" .
_:b  foaf:mbox   <mailto:peter@example.org> .
_:c  foaf:mbox   <mailto:carol@example.org> .
```

**查询**

```SPARQL
PREFIX foaf:   <http://xmlns.com/foaf/0.1/>
SELECT ?name ?mbox
WHERE
  { ?x foaf:name ?name .
    ?x foaf:mbox ?mbox }
```

**结果集**

| name                | mbox                        |
| ------------------- | --------------------------- |
| "Johnny Lee Outlaw" | \<mailto:jlow@example.com\> |
| "Peter Goodguy"     | \<mailto.peter@example.com> |

每种解决方案都提供了一种可以将所选变量绑定到RDF术语的方法，以使查询模式与数据匹配。结果集提供了所有可能的解决方案。在上面的示例中，数据的以下两个子集提供了两个匹配项。

```turtle
_:a foaf:name  "Johnny Lee Outlaw" .
_:a foaf:box   <mailto:jlow@example.com> .
```

```turtle
_:b foaf:name  "Peter Goodguy" .
_:b foaf:box   <mailto:peter@example.org> .
```

这是一个基本图模式匹配；**查询模式中使用的所有变量必须在每个解决方案中都绑定。**

### 2.3 匹配RDF文字

下列数据包含三个RDF文字

```turtle
@prefix dt:   <http://example.org/datatype#> .
@prefix ns:   <http://example.org/ns#> .
@prefix :     <http://example.org/ns#> .
@prefix xsd:  <http://www.w3.org/2001/XMLSchema#> .

:x   ns:p     "cat"@en .
:y   ns:p     "42"^^xsd:integer .
:z   ns:p     "abc"^^dt:specialDatatype .
```

注意，在Turtle中，"cat"@en是一个文字，带有"cat"和一个语法标签"en"的语法格式；"42"^^xsd:integer是一个类型文字，带有数据类型`http://www.w3.org/2001/XMLSchema#integer`；"abc"^^dt:specialDatatype"是一个带有数据类型``http://example.org/datatype#specialDatatype`.的类型文字。

#### 2.3.1 匹配带有语言标签的文字

语言标签在SPARQL中使用@表示。

下面的查询没有返回结果，因为"cat"与"cat@en"不同

```SPARQL
SELECT ?v WHERE { ?v ?p "cat" }
```

但是下列查询可以得到一个结果，变量v与:x绑定，因为语言标签是指定的，并且匹配给出的数据：

```SPARQL
SELECT ?v WHERE { ?v ?p "cat"@en }
```

得到

| v                          |
| -------------------------- |
| \<http://example.org/ns#x> |

#### 2.3.2 匹配带有数字类型的文字

整型在SPARQL中表示为带有数据类型`xsd:integer`的RDF类型文字。例如：42是`"42"^^<http://www.w3.org/2001/XMLSchema#integer>`.的一个简写格式。

以下查询模式得到一个带有变量v绑定y的结果：

```SPARQL
SELECT ?v WHERE { ?v ?p 42 }
```

| v                          |
| -------------------------- |
| \<http://example.org/ns#y> |

#### 2.3.3 匹配任意数据类型的文字

下面的查询返回一个带有变量v绑定:z的解决方案。查询处理器对数据类型空间中的值没有任何理解。因为词法格式和数据类型IRI都匹配，则文字也匹配。

```SPARQL
SELECT ?v WHERE { ?v ?p "abc"^^<http://example.org/datatype#specialDatatype> }
```

| v                           |
| --------------------------- |
| `<http://example.org/ns#z>` |

### 2.4 空节点的查询结果

<u>查询结果可以包含空节点。</u>空节点在本文档的样例结果集中写作"_:"跟随一个空节点标签的格式。

空节点标签的作用域为结果集，或者对于`CONSTRUCT`查询表单，为结果图。在结果集中用相同的标签表示相同的空节点。

**数据**

```turtle
@prefix foaf:  <http://xmlns.com/foaf/0.1/> .

_:a  foaf:name   "Alice" .
_:b  foaf:name   "Bob" .
```

**查询**

```SPARQL
PREFIX foaf:   <http://xmlns.com/foaf/0.1/>
SELECT ?x ?name
WHERE  { ?x foaf:name ?name }
```

| x    | name    |
| ---- | ------- |
| _:c  | "Alice" |
| _:d  | "Bob"   |

同样可以使用不同的空节点标签来给出上述结果，因为结果中的标签仅指示解决方案中的RDF术语相同还是不同。

| x    | name    |
| ---- | ------- |
| _:r  | "Alice" |
| _:s  | "Bob"   |

两个结果有相同的信息：<u>在两个解决方案用于匹配查询的空节点是不同的。结果集中的标签_:a与具有相同标签的数据图中的空节点之间不必存在任何关系。</u>

应用程序编写者不应期望查询中的空节点标签引用数据中的特定空节点。

### 2.5 用表达式创建值

SPARQL1.1 允许从复杂的表达式创建值。下面的查询显示如何使用CONCAT函数连接foaf数据中的名字和姓氏，然后使用SELECT语句分配值，以及也使用BIND形式分配值。

**数据**

```turtle
@prefix foaf:  <http://xmlns.com/foaf/0.1/> .
          
_:a  foaf:givenName   "John" .
_:a  foaf:surname  "Doe" .
```

**查询**

```SPARQL
PREFIX foaf:   <http://xmlns.com/foaf/0.1/>
SELECT ( CONCAT(?G, " ", ?S) AS ?name )
WHERE  { ?P foaf:givenName ?G ; foaf:surname ?S }
```

```SPARQL
PREFIX foaf:   <http://xmlns.com/foaf/0.1/>
SELECT ?name
WHERE  { 
   ?P foaf:givenName ?G ; 
      foaf:surname ?S 
   BIND(CONCAT(?G, " ", ?S) AS ?name)
}
```

| name       |
| ---------- |
| "John Doe" |

### 2.6 构建RDF图

SPARQL有很多查询形式。`SELECT`查询形式返回绑定的变量。`CONSTRUCT`查询形式返回一个RDF图。

**数据**

```turtle
@prefix org:    <http://example.com/ns#> .

_:a  org:employeeName   "Alice" .
_:a  org:employeeId     12345 .

_:b  org:employeeName   "Bob" .
_:b  org:employeeId     67890 .
```

**查询**

```SPARQL
PREFIX foaf:   <http://xmlns.com/foaf/0.1/>
PREFIX org:    <http://example.com/ns#>

CONSTRUCT { ?x foaf:name ?name }
WHERE  { ?x org:employeeName ?name }
```

**结果**

```
@prefix foaf: <http://xmlns.com/foaf/0.1/> .
      
_:x foaf:name "Alice" .
_:y foaf:name "Bob" .
```

可以用RDF/XML写作：

```xml
<rdf:RDF
    xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#"
    xmlns:foaf="http://xmlns.com/foaf/0.1/"
    >
  <rdf:Description>
    <foaf:name>Alice</foaf:name>
  </rdf:Description>
  <rdf:Description>
    <foaf:name>Bob</foaf:name>
  </rdf:Description>
</rdf:RDF>
```

## 3 RDF术语约束条件

图模式匹配生成一个解决方案序列，每一个解决方案有一个由变量绑定RDF术语的集合。SPARQL `FILTER` 限制解决方案为那些过滤表达式为true的。

本章提供了一个对SPARQL FILTER的信息简介。本节中的示例共享一个输入图：

**数据**

```turtle
@prefix dc:   <http://purl.org/dc/elements/1.1/> .
@prefix :     <http://example.org/book/> .
@prefix ns:   <http://example.org/ns#> .

:book1  dc:title  "SPARQL Tutorial" .
:book1  ns:price  42 .
:book2  dc:title  "The Semantic Web" .
:book2  ns:price  23 .
```

### 3.1 限制字符串的值

SPARQL `FILTER`函数类似正则，可以匹配RDF文字。正则匹配只有字符串文字。通过使用**str**函数，可以使用regex匹配其他文字的词法形式。

**查询**

```SPARQL
PREFIX  dc:  <http://purl.org/dc/elements/1.1/>
SELECT  ?title
WHERE   { ?x dc:title ?title
          FILTER regex(?title, "^SPARQL") 
        }
```

**查询结果**

| title             |
| ----------------- |
| "SPARQL Tutorial" |

可以使用“ i”标志使正则表达式匹配不区分大小写。

**查询**

```SPARQL
PREFIX  dc:  <http://purl.org/dc/elements/1.1/>
SELECT  ?title
WHERE   { ?x dc:title ?title
          FILTER regex(?title, "web", "i" ) 
        }
```

**结果**

| title              |
| ------------------ |
| "The Semantic Web" |

### 3.2 限制数字的值

SPARQL `FILTER`可以限制算术表达式。

**查询**

```SPARQL
PREFIX  dc:  <http://purl.org/dc/elements/1.1/>
PREFIX  ns:  <http://example.org/ns#>
SELECT  ?title ?price
WHERE   { ?x ns:price ?price .
          FILTER (?price < 30.5)
          ?x dc:title ?title . }
```

| title              | price |
| ------------------ | ----- |
| "The Semantic Web" | 23    |

通过约束价格变量，只有:book2匹配查询，因为只有:book2的价格低于30.5(filter条件)

### 3.3 其他格式的约束

除了数字类型，SPARQL支持类型`xsd:string,xsd:boolean,xsd:datetime`。

## 4 SPARQL语法(Syntax)

本节介绍SPARQL用于RDF术语和三元组模式的语法。

### 4.1 RDF术语语法

#### 4.1.1 IRIs的语法

所述IRI指定IRI的集合; IRI是URI的概括，并且与URI和URL完全兼容。该PrefixedName指定一个前缀名。下面描述从前缀名称到IRI的映射。IRI引用（相对或绝对IRI）是由IRIREF产生的，其中<u>“ <”和“>”定界符</u>不构成IRI引用的一部分。

RDF概念和抽象语法中定义的RDF术语集包括RDF URI引用，而SPARQL术语包括IRI。RDF URI引用包含“ `<`”，“ `>`”，“ `"`”（双引号），空格，“{”，“ }”，“ |”，“ \”，“ ^”和“ `”，不是IRI。未定义针对由此类RDF URI引用组成的RDF语句的SPARQL查询的行为。

##### 4.1.1.1 前缀名

该`PREFIX`关键字与IRI前缀标签相关联。前缀名称是前缀标签和局部部分，以冒号“ `:`”分隔。通过前缀和部分关联的IRI串联在一起，可以将前缀名称映射到IRI。前缀标签或局部部分可能为空。请注意，SPARQL本地名称允许前导数字，而XML本地名称则不允许。 <u>SPARQL本地名称还允许通过反斜杠字符转义符（例如`ns:id\=123`）在IRI中允许使用非字母数字字符。</u> SPARQL本地名称具有更多的语法限制。

##### 4.1.1.2 相对IRIs

根据统一资源标识符（URI），相对IRI与基本IRI结合在一起。不执行基于语法的规范化和基于方案的规范化。根据[国际化资源标识符（IRI）]的6.5节，以与URI引用中未保留的字符相同的方式对待IRI引用中另外允许的字符 。

`BASE`关键字定义base IRI用于解析相对的IRI，“base URI嵌入在内容中”。在5.1.3中基于“检索URI中的URI”标识的“检索URI”是从中检索特定SPARQL查询的URL。如果以上均未指定基本URI，则使用默认基本URI（第5.1.4节“默认基本URI”）。

以下是写同一个IRI的不同方式：

```turtle
<http://example.org/book/book1>
```

```turtle
BASE <http://example.org/book/>
<book1>
```

```turtle
PREFIX book: <http://example.org/book/>
book:book1
```

#### 4.1.2 文字语法

文字的常规语法是一个字符串（用双引号`"..."`或单引号引起来`'...'`），带有可选的语言标记（通过引入`@`）或可选的数据类型IRI或前缀名称（通过引入`^^`）。

为方便起见，可以直接写入整数（不带引号和显式数据类型IRI），并将其解释为数据类型的类型化文字`xsd:integer`。没有指数的小数没有被解释为`xsd:decimal`; 带指数的小数解释为`xsd:double`。类型为`xsd:boolean`的值也可以写为`true`或` false`。

为了便于编写<u>本身包含引号</u>或<u>长且包含换行符的文字值</u>，SPARQL提供了附加的引号构造，其中文字被用三个单引号或双引号引起来。

SPARQL中的文字语法示例包括：

- `"chat"`
- `'chat'@fr` 带有语言标签 "fr"
- `"xyz"^^<http://example.org/ns/userDatatype>`
- `"abc"^^appNS:appDataType`
- `'''The librarian said, "Perhaps you would enjoy 'War and Peace'."'''`
- `1`, 等同于 `"1"^^xsd:integer`
- `1.3`, 等同于 `"1.3"^^xsd:decimal`
- `1.300`, 等同于`"1.300"^^xsd:decimal`
- `1.0e6`, 等同于 `"1.0e6"^^xsd:double`
- `true`, 等同于`"true"^^xsd:boolean`
- `false`, 等同于 `"false"^^xsd:boolean`

与INTEGER，DECIMAL，DOUBLE和BooleanLiteral匹配的令牌，等效于带有令牌的词法值和相应数据类型（xsd：integer，xsd：decimal，xsd：double，xsd：boolean）的类型文字。

#### 4.1.3 查询变量的语法

查询变量通过使用“？”或“”; “？”或“”;“？”标记不是变量名的一部分。在查询中，$abc和?abc标识相同的变量。变量的可能名称在SPARQL语法中给出。

#### 4.1.4 空节点的语法

图模式中的<u>空节点</u>作为**<u>变量</u>**，而不是指向要查询的数据中特定空白节点。

空节点由标签形式（例如“ `_:abc`”）或缩写形式“ `[]`”表示。<u>可以用`[]`表示在查询语法中仅在一个位置使用的空白节点。唯一的空节点将用于形成三元组模式。对于带有标签“ `abc`”的空节点，空白节点标签被写为“`_:abc` ”。相同的空节点标签不能在同一查询</u>的<u>两个不同的基本图模式</u>中使用。

`[:p :v]`构建体可以在三元组模式中使用。它创建一个空节点标签，用作所有包含的谓词-宾语对的主语。创建的空节点还可以在主语和宾语位置的其他三元组模式中使用。

以下两种格式

```
[ :p "v" ] .
```

```
[] :p "v" .
```

分配唯一的空节点标签（此处为“ b57”），等效于编写：

```
_:b57 :p "v" .
```

此分配的空节点标签可以用作其他三元模式的主语或宾语。例如，作为主语：

```
[ :p "v" ] :q "w" .
```

相当于两个三元组：

```
_:b57 :p "v" .
_:b57 :q "w" .
```

作为宾语

```
:x :q [ :p "v" ] .
```

相当于两个三元组：

```
:x  :q _:b57 .
_:b57 :p "v" .
```

缩写的空节点语法可以与常见主语和谓语的其他缩写组合使用。

```turtle
[ 	foaf:name  ?name ;
    foaf:mbox  <mailto:alice@example.org> ]
```

这与为某些唯一分配的空节点标签“ b18”，编写以下基本图模式相同：

```turtle
  _:b18  foaf:name  ?name .
  _:b18  foaf:mbox  <mailto:alice@example.org> .
```

### 4.2 三元组模式语法

三元组模式被写为主语，谓语和宾语；有一些简单的方法可以编写一些常见的三元组模式构造。

以下示例表示相同的查询：

```turtle
PREFIX  dc: <http://purl.org/dc/elements/1.1/>
SELECT  ?title
WHERE   { <http://example.org/book/book1> dc:title ?title } 
```

```turtle
PREFIX  dc: <http://purl.org/dc/elements/1.1/>
PREFIX  : <http://example.org/book/>

SELECT  $title
WHERE   { :book1  dc:title  $title }
```

```turtle
BASE    <http://example.org/book/>
PREFIX  dc: <http://purl.org/dc/elements/1.1/>

SELECT  $title
WHERE   { <book1>  dc:title  ?title }
```

#### 4.2.1 谓语-宾语列表

可以写入具有共同主语的三元组模式，以便该主语仅被写入一次，并且通过使用符号**“;”**可以用于多个以上的三元组模式。

```turtle
	?x  foaf:name  ?name ;
        foaf:mbox  ?mbox .
```

这与编写三元模式相同：

```turtle
    ?x  foaf:name  ?name .
    ?x  foaf:mbox  ?mbox .
```

#### 4.2.2 宾语列表

如果三元模式同时共享主语和谓语，则宾语可以用**“,”**分隔。

```turtle
    ?x foaf:nick  "Alice" , "Alice_" .
```

与编写三元组模式相同：

```turtle
   ?x  foaf:nick  "Alice" .
   ?x  foaf:nick  "Alice_" .
```

宾语列表可以与谓词-宾语列表结合使用：

```turtle
   ?x  foaf:name ?name ; 
   foaf:nick  "Alice" , "Alice_" .
```

等价于

```turtle
   ?x  foaf:name  ?name .
   ?x  foaf:nick  "Alice" .
   ?x  foaf:nick  "Alice_" .
```

#### 4.2.3 RDF集合

可以使用语法“（element1 element2 ...）”以三元组模式编写 RDF集合。形式“ `()`”是IRI` http://www.w3.org/1999/02/22-rdf-syntax-ns#nil`的替代形式。当与集合元素一起使用时，例如`(1 ?x 3 4)`，会为集合分配带有空节点的三元组模式。集合头部的空节点可以用作其他三元组模式的主语或宾语。集合语法分配的空节点不会出现在查询中的其他位置。

```
(1 ?x 3 4) :p "w" .
```

是下面写法的语法糖（注意b0，b1，b2和b3不会在查询中的其他位置出现）

```turtle
    _:b0  rdf:first  1 ;
          rdf:rest   _:b1 .
    _:b1  rdf:first  ?x ;
          rdf:rest   _:b2 .
    _:b2  rdf:first  3 ;
          rdf:rest   _:b3 .
    _:b3  rdf:first  4 ;
          rdf:rest   rdf:nil .
    _:b0  :p         "w" . 
```

RDF集合可以嵌套，并且可以涉及其他语法形式：

```turtle
(1 [:p :q] ( 2 ) ) .
```

是下面形式的语法糖

```turtle
    _:b0  rdf:first  1 ;
          rdf:rest   _:b1 .
    _:b1  rdf:first  _:b2 .
    _:b2  :p         :q .
    _:b1  rdf:rest   _:b3 .
    _:b3  rdf:first  _:b4 .
    _:b4  rdf:first  2 ;
          rdf:rest   rdf:nil .
    _:b3  rdf:rest   rdf:nil .
```

#### 4.2.4 rdf:type

关键字`a`可以用作三元组的谓语，并且是IRI http://www.w3.org/1999/02/22-rdf-syntax-ns#type的替代形式。此关键字区分大小写。

```turtle
  ?x  a  :Class1 .
  [ a :appClass ] :p "v" .
```

是下面形式的语法糖：

```turtle
  ?x    rdf:type  :Class1 .
  _:b0  rdf:type  :appClass .
  _:b0  :p        "v" .
```

## 5 图模式

SPARQL基于图模式匹配。通过以各种方式组合较小的模式可以形成更复杂的图形模式：

- **基本图模式**，其中一组**三元组**模式必须匹配
- **组图模式**，其中一组**图**模式必须全部匹配
- **可选的图模式**，其中附加的模式可以扩展解决方案
- **替代图模式**，尝试两个或多个可能的模式
- **命名图上的模式**，其中模式与命名图匹配

在本节中，我们描述两种通过结合来组合模式的形式：基本图形模式（组合三元组模式）和组图模式（组合所有其他图模式）。

查询中最外层的**图**模式称为**查询模式**。它是通过语法确定`GroupGraphPattern`在

```SPARQL
[17]  	WhereClause	  ::=  	'WHERE'? GroupGraphPattern
```

### 5.1 基本图模式

<u>基本图模式是三元组模式的集合</u>。SPARQL图模式匹配是根据对基本图模式匹配的结果进行组合来定义的。

带有可选过滤器的三元组模式序列包含一个单一的基本图模式。任何其他图模式都会终止基本图模式。

#### 5.1.1 空节点标签

使用形式`_:abc`为的空节点时，空节点的标签的作用域为基本图模式。在任何查询中，标签只能在单个基本形模式中使用。

#### 5.1.2 扩展的图模式匹配

SPARQL使用子图匹配来评估基本图模式，该子图匹配是为简单起见而定义的。 如下所述，在给定某些条件的情况下，SPARQL可以扩展到其他形式的蕴含。

### 5.2 组图模式

在SPARQL查询字符串中，组图模式用花括号`{}`定界 。例如，此查询的查询模式是一个基本图形模式的组图形模式。

```SPARQL
PREFIX foaf:    <http://xmlns.com/foaf/0.1/>
SELECT ?name ?mbox
WHERE  {
          ?x foaf:name ?name .
          ?x foaf:mbox ?mbox .
       }
```

从将三元组模式分组为两个基本图形模式的查询中，可以获得相同的解决方案。例如，下面的查询具有不同的结构，但将产生与上一个查询相同的解决方案：

```SPARQL
PREFIX foaf:    <http://xmlns.com/foaf/0.1/>
SELECT ?name ?mbox
WHERE  { { ?x foaf:name ?name . }
         { ?x foaf:mbox ?mbox . }
       }
```

#### 5.2.1 空组模式

组模式

```
{ }
```

与任何图（包括空图）的不绑定任何变量的一种解决方案匹配。例如：

```SPARQL
SELECT ?x
WHERE {}
```

与其中未绑定变量x的一种解决方案匹配。

#### 5.2.2 过滤范围

用关键字`FILTER`表示的约束是对出现过滤器的整个**组**的解决方案的限制。以下模式均具有相同的解决方案：

```SPARQL
 {  ?x foaf:name ?name .
    ?x foaf:mbox ?mbox .
    FILTER regex(?name, "Smith")
 }
```

```SPARQL
 {  FILTER regex(?name, "Smith")
    ?x foaf:name ?name .
    ?x foaf:mbox ?mbox .
 }
```

```SPARQL
 {  ?x foaf:name ?name .
    FILTER regex(?name, "Smith")
    ?x foaf:mbox ?mbox .
 }
```

#### 5.2.3 组图模式示例

```SPARQL
  {
    ?x foaf:name ?name .
    ?x foaf:mbox ?mbox .
  }
```

是一组一个基本图模式，该基本图形模式由两个三元组模式组成。

```SPARQL
  {
    ?x foaf:name ?name . FILTER regex(?name, "Smith")
    ?x foaf:mbox ?mbox .
  }
```

是一组由**一个**基本图形模式和一个过滤器组成的组，该基本图形模式由两个三元组模式组成；过滤器不会将基本图形模式分为两个基本图形模式。

```SPARQL
  {
    ?x foaf:name ?name .
    {}
    ?x foaf:mbox ?mbox .
  }
```

是由三个元素组成的组

- 一个三元组模式的基本图形模式
- 一个空组
- 一个三元组模式的另一个基本图形模式。

## 6 可选值

基本图形模式允许应用程序进行查询，其中必须匹配整个查询模式才能找到解决方案。对于仅包含至少一个基本图模式的组图模式查询的每个解决方案，每个变量都绑定到解决方案中的RDF术语。但是，不能在所有RDF图中假定规则的完整结构。能够进行查询以允许将信息添加到解决方案中（如果有信息可用），但是不拒绝解决方案，这是有用的，因为查询模式的某些部分不匹配。可选匹配提供了此功能：<u>如果可选部分不匹配，则不创建绑定，但不会消除解决方案。</u>

### 6.1 可选模式匹配

可以使用应用于图形模式的`OPTIONAL`关键字在语法上指定图形模式的可选部分：

```SPARQL
pattern OPTIONAL { pattern }
```

句法形式：

```SPARQL
{ OPTIONAL { pattern } }
```

等价于

```SPARQL
{ { } OPTIONAL { pattern } }
```

`OPTIONAL`关键字是左关联的：

```SPARQL
pattern OPTIONAL { pattern } OPTIONAL { pattern }
```

等同于

```SPARQL
{ pattern OPTIONAL { pattern } } OPTIONAL { pattern }
```

在可选匹配中，可选图模式与图匹配，从而定义并添加了对一个或多个解决方案的绑定，或者使解决方案保持不变而未添加任何其他绑定。

**数据**

```turtle
@prefix foaf:       <http://xmlns.com/foaf/0.1/> .
@prefix rdf:        <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .

_:a  rdf:type        foaf:Person .
_:a  foaf:name       "Alice" .
_:a  foaf:mbox       <mailto:alice@example.com> .
_:a  foaf:mbox       <mailto:alice@work.example> .

_:b  rdf:type        foaf:Person .
_:b  foaf:name       "Bob" .
```

**查询**

```SPARQL
PREFIX foaf: <http://xmlns.com/foaf/0.1/>
SELECT ?name ?mbox
WHERE  { ?x foaf:name  ?name .
         OPTIONAL { ?x  foaf:mbox  ?mbox }
       }
```

上述数据的查询结果为：

| name    | mbox                         |
| ------- | ---------------------------- |
| "Alice" | \<mailto:alice@example.com>  |
| "Alice" | \<mailto:alice@work.example> |
| "Bob"   |                              |

解决方案中名字为"Bob"的没有mbox的值

此查询在数据中查找人员的姓名。如果存在带有谓词`mbox`且主语相同的三元组 ，则解决方案也将包含该三元组的宾语。在此示例中，查询的可选匹配部分仅给出了一个三元组模式，但通常，<u>可选部分可以是任何图模式</u>。整个可选图模式必须与可选图模式匹配才能影响查询到解决方案。

### 6.2 可选模式匹配中的约束

约束可以由可选图模式给出。例如：

```turtle
@prefix dc:   <http://purl.org/dc/elements/1.1/> .
@prefix :     <http://example.org/book/> .
@prefix ns:   <http://example.org/ns#> .

:book1  dc:title  "SPARQL Tutorial" .
:book1  ns:price  42 .
:book2  dc:title  "The Semantic Web" .
:book2  ns:price  23 .
```

```SPARQL
PREFIX  dc:  <http://purl.org/dc/elements/1.1/>
PREFIX  ns:  <http://example.org/ns#>
SELECT  ?title ?price
WHERE   { ?x dc:title ?title .
          OPTIONAL { ?x ns:price ?price . FILTER (?price < 30) }
        }
```

| title              | price |
| ------------------ | ----- |
| "SPARQL Tutorial"  |       |
| "The Semantic Web" | 23    |

标题为“ SPARQL教程”的书没有价格出现，因为可选图形模式没有涉及变量“ price”的解决方案。

### 6.3 多个可选图模式

图形模式是递归定义的。图模式可以具有零个或多个可选图模式，查询模式的任何部分都可以具有可选部分。在此示例中，有两个可选的图形模式。

**数据**

```turtle
@prefix foaf:       <http://xmlns.com/foaf/0.1/> .

_:a  foaf:name       "Alice" .
_:a  foaf:homepage   <http://work.example.org/alice/> .

_:b  foaf:name       "Bob" .
_:b  foaf:mbox       <mailto:bob@work.example> .
```

**查询**

```SPARQL
PREFIX foaf: <http://xmlns.com/foaf/0.1/>
SELECT ?name ?mbox ?hpage
WHERE  { ?x foaf:name  ?name .
         OPTIONAL { ?x foaf:mbox ?mbox } .
         OPTIONAL { ?x foaf:homepage ?hpage }
       }
```

**查询结果**

| name    | mbox                             | hpage                            |
| ------- | -------------------------------- | -------------------------------- |
| "Alice" |                                  | <http://work.example.org/alice/> |
| "Bob"   | <http://work.example.org/alice/> |                                  |

## 7 替代匹配

SPARQL提供了一种组合图形模式的方法，以便可以匹配多个替代图形模式之一。<u>如果多个备选方案匹配，那么将找到所有可能的模式解决方案。</u>

模式选择用`UNION`关键字在语法上指定。

**数据**

```turtle
@prefix dc10:  <http://purl.org/dc/elements/1.0/> .
@prefix dc11:  <http://purl.org/dc/elements/1.1/> .

_:a  dc10:title     "SPARQL Query Language Tutorial" .
_:a  dc10:creator   "Alice" .

_:b  dc11:title     "SPARQL Protocol Tutorial" .
_:b  dc11:creator   "Bob" .

_:c  dc10:title     "SPARQL" .
_:c  dc11:title     "SPARQL (updated)" .
```

**查询**

```SPARQL
PREFIX dc10:  <http://purl.org/dc/elements/1.0/>
PREFIX dc11:  <http://purl.org/dc/elements/1.1/>

SELECT ?title
WHERE  { { ?book dc10:title  ?title } UNION { ?book dc11:title  ?title } }
```

**查询结果**

| title                            |
| -------------------------------- |
| "SPARQL Protocol Tutorial"       |
| "SPARQL"                         |
| "SPARQL (updated)"               |
| "SPARQL Query Language Tutorial" |

无论使用1.0版还是1.1版的Dublin Core属性记录标题，此查询都会在数据中查找书名。为了准确确定信息的记录方式，可以为两个替代方案使用不同的变量进行查询：

```SPARQL
PREFIX dc10:  <http://purl.org/dc/elements/1.0/>
PREFIX dc11:  <http://purl.org/dc/elements/1.1/>

SELECT ?x ?y
WHERE  { { ?book dc10:title ?x } UNION { ?book dc11:title  ?y } }
```

| x                                | y                          |
| -------------------------------- | -------------------------- |
|                                  | "SPARQL (updated)"         |
|                                  | "SPARQL Protocol Tutorial" |
| "SPARQL"                         |                            |
| "SPARQL Query Language Tutorial" |                            |

这将返回结果，该结果的变量`x`绑定到的左分支的解决方案`UNION`，`y`绑定到右边的分支的解决方案。如果`UNION` 模式的任何部分都不匹配，则图形模式将不匹配。

该`UNION`模式结合了图形模式；<u>每个替代可能性都可以包含多个三元组模式</u>：

```SPARQL
PREFIX dc10:  <http://purl.org/dc/elements/1.0/>
PREFIX dc11:  <http://purl.org/dc/elements/1.1/>

SELECT ?title ?author
WHERE  { { ?book dc10:title ?title .  ?book dc10:creator ?author }
         UNION
         { ?book dc11:title ?title .  ?book dc11:creator ?author }
       }
```

| title                            | author  |
| -------------------------------- | ------- |
| "SPARQL Query Language Tutorial" | "Alice" |
| "SPARQL Protocol Tutorial"       | "Bob"   |

如果该查询同时具有相同版本的Dublin Core的书名和创建者谓语，则该查询仅与书匹配。

## 8 否定

SPARQL查询语言合并了两种否定样式，一种基于过滤结果，该结果取决于图形模式是否在要过滤的查询解决方案的内容中**匹配**，另一种基于**删除**与另一种模式相关的解决方案。

### 8.1 使用图模式过滤

使用`NOT NOTISTS`和`EXISTS`在`FILTER`表达式中完成查询解决方案的过滤。请注意，过滤器作用域规则适用于出现过滤器的**整个组**。

#### 8.1.1 测试图的缺失

给定出现过滤器的组图形模式中的变量值，`NOT EXISTS`过滤器表达式将测试图模式是否与数据集**不**匹配。它不会生成任何其他绑定。（等价于在查询模式中已经绑定的模式上检验而已）

**数据**

```turtle
@prefix  :       <http://example/> .
@prefix  rdf:    <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix  foaf:   <http://xmlns.com/foaf/0.1/> .

:alice  rdf:type   foaf:Person .
:alice  foaf:name  "Alice" .
:bob    rdf:type   foaf:Person . 
```

**查询**

```SPARQL
PREFIX  rdf:    <http://www.w3.org/1999/02/22-rdf-syntax-ns#> 
PREFIX  foaf:   <http://xmlns.com/foaf/0.1/> 

SELECT ?person
WHERE 
{
    ?person rdf:type  foaf:Person .
    FILTER NOT EXISTS { ?person foaf:name ?name }
}     
```

**结果**

| person                |
| --------------------- |
| \<http://example/bob> |

#### 8.1.2 测试图的存在

还提供了过滤器表达式`EXISTS`。它测试是否可以在数据中找到模式。它不会生成任何其他绑定。

**查询**

```SPARQL
PREFIX  rdf:    <http://www.w3.org/1999/02/22-rdf-syntax-ns#> 
PREFIX  foaf:   <http://xmlns.com/foaf/0.1/> 

SELECT ?person
WHERE 
{
    ?person rdf:type  foaf:Person .
    FILTER EXISTS { ?person foaf:name ?name }
}
```

**结果**

| person                  |
| ----------------------- |
| \<http://example/alice> |

### 8.2 删除可能的解决方案

SPARQL中提供的另一种否定方式是`MINUS`，它会评估两个参数，然后在左侧计算出与右侧解决方案不兼容的解决方案。

**数据**

```turtle
@prefix :       <http://example/> .
@prefix foaf:   <http://xmlns.com/foaf/0.1/> .

:alice  foaf:givenName "Alice" ;
        foaf:familyName "Smith" .

:bob    foaf:givenName "Bob" ;
        foaf:familyName "Jones" .

:carol  foaf:givenName "Carol" ;
        foaf:familyName "Smith" .
```

**查询**

```SPARQL
PREFIX :       <http://example/>
PREFIX foaf:   <http://xmlns.com/foaf/0.1/>

SELECT DISTINCT ?s
WHERE {
   ?s ?p ?o .
   MINUS {
      ?s foaf:givenName "Bob" .
   }
}
```

**结果**

| s                       |
| ----------------------- |
| \<http://example/carol> |
| \<http://example/alice> |

### 8.3 NOT EXISTS和MINUS之间的关系

`NOT EXISTS`并`MINUS`代表两种关于否定的思考方式，一种是基于**给定**查询模式已确定的绑定的情况下，**测试**数据中是否存在某种模式，另一种是基于对**两种**模式的评估来**消除**匹配。在某些情况下，他们可能会得出不同的答案。

#### 8.3.1 示例：共享变量

```turtle
@prefix : <http://example/> .
:a :b :c .
```

```SPARQL
SELECT *
{ 
  ?s ?p ?o
  FILTER NOT EXISTS { ?x ?y ?z }
}
```

计算结果为无解的结果集，因为`{ ?x ?y ?z }` 匹配任何给定的`?s ?p ?o`，因此`NOT EXISTS { ?x ?y ?z }` 消除了所有解。<u>内外共享</u>

**结果**

| s    | p    | o    |
| ---- | ---- | ---- |
|      |      |      |

而使用时`MINUS`，在第一部分（`?s ?p ?o`）和第二部分（`?x ?y ?z`）之间没有共享变量，因此不会消除任何绑定。<u>内外不共享，?s和?x代表了两个独立的模式，是相互独立的两部分</u>

```SPARQL
SELECT *
{ 
   ?s ?p ?o 
   MINUS 
     { ?x ?y ?z }
}
```

**结果**

| s                   | p                   | o                   |
| ------------------- | ------------------- | ------------------- |
| \<http://example/a> | \<http://example/b> | \<http://example/c> |

#### 8.3.2 示例：固定模式

？另一种情况是示例中存在具体模式（无变量）：

```SPARQL
PREFIX : <http://example/>
SELECT * 
{ 
  ?s ?p ?o 
  FILTER NOT EXISTS { :a :b :c }
}
```

评估为没有查询解决方案的结果集：

**结果**

| s    | p    | o    |
| ---- | ---- | ---- |
|      |      |      |

然而

```SPARQL
PREFIX : <http://example/>
SELECT * 
{ 
  ?s ?p ?o 
  MINUS { :a :b :c }
}
```

**结果**

| s                   | p                   | o                   |
| ------------------- | ------------------- | ------------------- |
| \<http://example/a> | \<http://example/b> | \<http://example/c> |

因为没有绑定的匹配，所以没有解决方案被消除。

#### 8.3.3 示例：内部FILTERs

之所以会产生差异，是因为在过滤器中，该组中的变量在作用域内。在此示例中，`NOT EXISTS`内部的`FILTER`可以访问解决方案中的`?n`值。`FILTER NOT EXISTS`的条件是对于每一个查询模式的解决方案都会去测试，并且可以引用每一个解决方案里面的变量，而`MINUS`里的条件里是没办法引用查询模式里面的变量的，因此如果它里面的条件用到了查询模式里面的，那么将是无效的。

```turtle
@prefix : <http://example.com/> .
:a :p 1 .
:a :q 1 .
:a :q 2 .

:b :p 3.0 .
:b :q 4.0 .
:b :q 5.0 .
```

当使用`FILTER NOT EXISTS`，测试每一个`?x :p ?n `

```SPARQL
PREFIX : <http://example.com/>
SELECT * WHERE {
        ?x :p ?n
        FILTER NOT EXISTS {
                ?x :q ?m .
                FILTER(?n = ?m)
        }
}
```

| x                       | n    |
| ----------------------- | ---- |
| \<http://example.com/b> | 3.0  |

而使用`MINUS`时，`FILTER`模式的内部没有?n的值，并且始终是未绑定的：

```SPARQL
PREFIX : <http://example/>
SELECT * WHERE {
        ?x :p ?n
        MINUS {
                ?x :q ?m .
                FILTER(?n = ?m)
        }
}
```

| x                       | n    |
| ----------------------- | ---- |
| \<http://example.com/b> | 3.0  |
| \<http://example.com/a> | 1    |

## 9 属性路径

属性路径是通过两个图**节点**之间的图的可能路径。平凡的情况是长度恰好为1的属性路径，它是一个三元组模式。路径的末端可以是RDF术语或变量。变量不能用作路径本身的一部分，只能用作端点。

属性路径允许某些SPARQL基本图形模式的表达式更简洁，并且还增加了通过任意长度路径匹配两个资源的连通性的功能。

### 9.1 属性路径语法

在下方的描述中，iri是一个由完整的或前缀名缩写的IRI，或者关键字a。elt是一个路径元素，它本身可能由路径构造组成。

| Syntax Form                            | Property Path Expression Name | Matches                                                      |
| -------------------------------------- | ----------------------------- | ------------------------------------------------------------ |
| `iri`                                  | PredicatePath                 | An IRI. 路径长度为1.                                         |
| `^elt`                                 | InversePath                   | 逆路径 (宾语到主语).                                         |
| `elt1/elt2`                            | SequencePath                  | `elt1` 后跟随 `elt2`的序列路径.                              |
| `elt1 | elt2`                          | AlternativePath               | `elt1` 或`elt2`的替换路径 (尝试全部可能性).                  |
| `elt*`                                 | ZeroOrMorePath                | 通过`elt`的零个或多个匹配关系连接路径的主语和宾语的路径。    |
| `elt+`                                 | OneOrMorePath                 | 通过`elt`的一个或多个匹配关系连接路径的主语和宾语的路径。    |
| `elt?`                                 | ZeroOrOnePath                 | 通过`elt`的零个或一个匹配关系连接路径的主语和宾语的路径。    |
| `!iri` or `!(iri1| ...|irin)`          | NegatedPropertySet            | 否定属性集. 不属于 `irii. `的IRI。`!iri` is short for `!(iri)`. |
| `!^iri` or `!(^iri1| ...|^irin)`       | NegatedPropertySet            | 基于逆路径的否定属性集. 也就是不在 *iri1*...*irin* 中的逆路径. `!^iri` is short for `!(^iri)`. |
| `!(iri1| ...|irij|^irij+1| ...|^irin)` | NegatedPropertySet            | 否定属性集中的正向路径和逆路径的组合.                        |
| `(elt)`                                |                               | `elt`路径组, 括号控制优先级.                                 |

否定的属性集中的IRI和反向IRI的顺序并不重要，它们可以混合顺序出现。

语法形式的优先级从高到低：

- IRI, prefixed names
- Negated property sets
- Groups
- Unary operators `*`, `?` and `+`
- Unary ^ inverse links
- Binary operator `/`
- Binary operator `|`

优先级在组内从左到右。

### 9.2 示例

备选方案：匹配一种或两种可能性

```SPARQL
  { :book1 dc:title|rdfs:label ?displayString }
```

可以写成

```SPARQL
  { :book1 <http://purl.org/dc/elements/1.1/title> | <http://www.w3.org/2000/01/rdf-schema#label> ?displayString }
```

顺序：找到爱丽丝认识的任何人的名字。

```SPARQL
  {
    ?x foaf:mbox <mailto:alice@example> .
    ?x foaf:knows/foaf:name ?name .
  }
```

顺序：查找2个“ foaf：knows”链接的人名。

```SPARQL
  { 
    ?x foaf:mbox <mailto:alice@example> .
    ?x foaf:knows/foaf:knows/foaf:name ?name .
  }
```

与这个SPARQL查询相同：

```SPARQL
  SELECT ?x ?name 
  {
     ?x  foaf:mbox <mailto:alice@example> .
     ?x  foaf:knows [ foaf:knows [ foaf:name ?name ]]. 
  }
```

或者，使用显式变量：

```SPARQL
  SELECT ?x ?name
  {
    ?x  foaf:mbox <mailto:alice@example> .
    ?x  foaf:knows ?a1 .
    ?a1 foaf:knows ?a2 .
    ?a2 foaf:name ?name .
  }
```

过滤重复项：由于Alice认识的某人可能很了解Alice，因此上面的示例可能包括Alice自己。可以通过以下方法避免这种情况：

```SPARQL
  { ?x foaf:mbox <mailto:alice@example> .
    ?x foaf:knows/foaf:knows ?y .
    FILTER ( ?x != ?y )
    ?y foaf:name ?name 
  }
```

反向属性路径：这两个查询是相同的：第二个只是反转属性方向，从而互换了主语和宾语的角色。

```SPARQL
  { ?x foaf:mbox <mailto:alice@example> }
```

```SPARQL
  { <mailto:alice@example> ^foaf:mbox ?x }
```

逆路径序列：找到所有认识?x认识的人。

```SPARQL
  {
    ?x foaf:knows/^foaf:knows ?y .  
    FILTER(?x != ?y)
  }
```

等效于（?gen1是系统生成的变量）：

```SPARQL
  {
    ?x foaf:knows ?gen1 .
    ?y foaf:knows ?gen1 .  
    FILTER(?x != ?y)
  }
```

任意长度的匹配：通过`foaf:knows`查找可以从Alice联系到的所有人员的姓名：

```SPARQL
  {
    ?x foaf:mbox <mailto:alice@example> .
    ?x foaf:knows+/foaf:name ?name .
  }
```

任意长度路径中的替代方法：

```SPARQL
  { ?ancestor (ex:motherOf|ex:fatherOf)+ <#me> }
```

任意长度路径匹配：有限推理的某些形式也是可能的。例如，对于RDFS，资源的所有类型和超类型：

```SPARQL
  { <http://example/thing> rdf:type/rdfs:subClassOf* ?type }
```

所有资源及其所有推断类型：

```SPARQL
  { ?x rdf:type/rdfs:subClassOf* ?type }
```

子属性

```SPARQL
  { ?x ?p ?v . 
  ?p rdfs:subPropertyOf* :property }
```

否定的属性路径：查找不是通过rdf：type连接的节点（无论哪种方式）：

```SPARQL
  { ?x !(rdf:type|^rdf:type) ?y }
```

RDF集合中的元素：？

```SPARQL
  { :list rdf:rest*/rdf:first ?element }
```

注意：此路径表达式不能保证结果的**顺序**。

### 9.3 属性路径和等效模式

SPARQL属性路径将RDF三元组视为<u>具有命名边的有向图</u>（可能是循环图）。一些属性路径等效于三元组模式中的转换和SPARQL UNION图形模式。对属性路径表达式的求值可能会导致**重复**，因为以等效模式<u>引入的任何变量</u>都不是结果的一部分，并且尚未在其他地方使用。通过将结果<u>隐式投影</u>到查询中给定的变量，可以将它们隐藏起来。

**数据**

```turtle
@prefix :       <http://example/> .

:order  :item :z1 .
:order  :item :z2 .

:z1 :name "Small" .
:z1 :price 5 .

:z2 :name "Large" .
:z2 :price 5 .
```

**查询**

```SPARQL
PREFIX :   <http://example/>
SELECT * 
{  ?s :item/:price ?x . }
```

**结果**

| s                       | x    |
| ----------------------- | ---- |
| \<http://example/order> | 5    |
| \<http://example/order> | 5    |

反之，如果查询被写为包含中间变量（?_a），则结果中是没有重复行的：

```SPARQL
PREFIX :   <http://example/>
SELECT * 
{  ?s :item ?_a .
   ?_a :price ?x . }
```

**结果**

| s                       | _a                   | x    |
| ----------------------- | -------------------- | ---- |
| \<http://example/order> | \<http://example/z1> | 5    |
| \<http://example/order> | \<http://example/z2> | 5    |

当查询还涉及聚合操作时，图形模式的等效性尤其重要。订单的总成本可以在以下位置找到

```SPARQL
  PREFIX :   <http://example/>
  SELECT (sum(?x) AS ?total)
  { 
    :order :item/:price ?x
  }
```

| total |
| ----- |
| 10    |

### 9.4 任意长度路径匹配

可以使用“零个或多个”属性路径运算符`*`和“一个或多个”属性路径运算符`+`，找到任意长度的属性路径在主语和主语之间的连通性。还有一个“零或一个”连接属性路径运算符 `?`。

这些运算符中的每一个都使用属性路径表达式来尝试查找主语和宾语之间的连接，并多次使用路径步长，这受到运算符的限制。

例如，使用以下方法可以找到资源的所有可能类型，包括资源的超类型：

```SPARQL
  PREFIX  rdfs:   <http://www.w3.org/2000/01/rdf-schema#> . 
  PREFIX  rdf:    <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
  SELECT ?x ?type
  { 
    ?x rdf:type/rdfs:subClassOf* ?type
  }
```

同样，找到所有的人:x通过foaf:knows关系进行连接，

```SPARQL
  PREFIX foaf: <http://xmlns.com/foaf/0.1/>
  PREFIX :     <http://example/>
  SELECT ?person
  { 
    :x foaf:knows+ ?person
  }
```

即使重复路径本身否则会导致重复，这种连接匹配也不会引入重复（它不包含进行连接的方式数量的任何计数）。

匹配的图可以包括循环。定义了连通性匹配，以便匹配循环不会导致不确定或无限的结果。

## 10 重命名

通过将新变量绑定到表达式的值（RDF术语），可以将表达式的值添加到解决方案映射中。然后，该变量可以在查询中使用，也可以在结果中返回。

三个语法形式允许：`BIND`关键字，  `SELECT`和`GROUP BY`中的表达式。重命名形式为`(expression AS ?var)`

如果表达式的求值产生错误，则该解决方案的变量将保持未绑定状态，但查询求值将继续。

数据也可以用 `VALUES`用于内联数据直接包含在查询中。

### 10.1 BIND：分配变量

该`BIND`形式允许将值分配给<u>基本图形模式</u>或<u>属性路径表达式</u>中的变量。使用`BIND`作为前面基本图形模式的结束。直到在`BIND`中使用，在组图模式中都不得使用`BIND`子句引入的变量。由`BIND`引入的变量必须在其引入之后才能使用。

**数据**

```turtle
@prefix dc:   <http://purl.org/dc/elements/1.1/> .
@prefix :     <http://example.org/book/> .
@prefix ns:   <http://example.org/ns#> .

:book1  dc:title     "SPARQL Tutorial" .
:book1  ns:price     42 .
:book1  ns:discount  0.2 .

:book2  dc:title     "The Semantic Web" .
:book2  ns:price     23 .
:book2  ns:discount  0.25 .
```

**查询**

```SPARQL
PREFIX  dc:  <http://purl.org/dc/elements/1.1/>
PREFIX  ns:  <http://example.org/ns#>

SELECT  ?title ?price
{  ?x ns:price ?p .
   ?x ns:discount ?discount
   BIND (?p*(1-?discount) AS ?price)
   FILTER(?price < 20)
   ?x dc:title ?title . 
}
```

等效查询（`BIND`结束基本图形模式；`FILTER`应用于整个组图形模式）：

```SPARQL
PREFIX  dc:  <http://purl.org/dc/elements/1.1/>
PREFIX  ns:  <http://example.org/ns#>

SELECT  ?title ?price
{  { ?x ns:price ?p .
     ?x ns:discount ?discount
     BIND (?p*(1-?discount) AS ?price)
   }
   {?x dc:title ?title . }
   FILTER(?price < 20)
}
```

**结果**

| title              | price |
| ------------------ | ----- |
| "The Semantic Web" | 17.25 |

### 10.2 值：提供内联数据

数据可以直接以图形模式写入，也可以使用 `VALUES`添加到查询中。 `VALUES`提供内联数据作为解决方案序列，并通过**联接**操作将其与查询评估的结果组合在一起 。应用程序可以使用它来提供对查询结果的特定要求，SPARQL查询引擎实现也可以使用它来通过关键字提供联合查询，`SERVICE`将以更受限的查询发送到远程查询服务。

#### 10.2.1 值语法

`VALUES`允许在数据块中指定多个变量；对于仅指定一个变量和一些值的常见情况，有一种特殊的语法。

在以下示例中，有两个变量 `?x`和`?y`的表。第二行没有为 `?y`赋值。

```turtle
VALUES (?x ?y) {
  (:uri1 1)
  (:uri2 UNDEF)
}
```

（可选）当有一个变量和一些值时：

```turtle
VALUES ?z { "abc" "def" }
```

与使用一般形式相同：

```turtle
VALUES (?z) { ("abc") ("def") }
```

#### 10.2.2 值示例

`VALUES`数据块可以出现在查询模式中或`SELECT`查询的末尾，包括子查询

**数据**

```turtle
@prefix dc:   <http://purl.org/dc/elements/1.1/> .
@prefix :     <http://example.org/book/> .
@prefix ns:   <http://example.org/ns#> .

:book1  dc:title  "SPARQL Tutorial" .
:book1  ns:price  42 .
:book2  dc:title  "The Semantic Web" .
:book2  ns:price  23 .
```

**查询**

```SPARQL
PREFIX dc:   <http://purl.org/dc/elements/1.1/> 
PREFIX :     <http://example.org/book/> 
PREFIX ns:   <http://example.org/ns#> 

SELECT ?book ?title ?price
{
   VALUES ?book { :book1 :book3 }
   ?book dc:title ?title ;
         ns:price ?price .
}
```

| book                             | title             | price |
| -------------------------------- | ----------------- | ----- |
| \<http://example.org/book/book1> | "SPARQL Tutorial" | 42    |

如果变量在`VALUES`子句中没有指定解决方案的值，则使用关键字`UNDEF`代替RDF术语。

```SPARQL
PREFIX dc:   <http://purl.org/dc/elements/1.1/> 
PREFIX :     <http://example.org/book/> 
PREFIX ns:   <http://example.org/ns#> 

SELECT ?book ?title ?price
{
   ?book dc:title ?title ;
         ns:price ?price .
   VALUES (?book ?title)
   { (UNDEF "SPARQL Tutorial")
     (:book2 UNDEF)
   }
}
```

| book                             | title              | price |
| -------------------------------- | ------------------ | ----- |
| \<http://example.org/book/book1> | "SPARQL Tutorial"  | 42    |
| \<http://example.org/book/book2> | "The Semantic Web" | 23    |

在此示例中，`VALUES`可能已经指定了对`SELECT`查询结果的执行（先有结果，再指定）：

```SPARQL
PREFIX dc:   <http://purl.org/dc/elements/1.1/> 
PREFIX :     <http://example.org/book/> 
PREFIX ns:   <http://example.org/ns#> 

SELECT ?book ?title ?price
{
   ?book dc:title ?title ;
         ns:price ?price .
}
VALUES (?book ?title)
{ (UNDEF "SPARQL Tutorial")
  (:book2 UNDEF)
}
```

这是一个不同的查询，但在示例情况下，结果相同。

## 11 聚合

聚合将表达式应用于解决方案组。默认情况下，解决方案集由一个包含所有解决方案的组组成。

可以使用`GROUP BY`语法指定分组。

在SPARQL的1.1版本中定义的聚集体 `COUNT`，`SUM`，`MIN`，`MAX`，`AVG`，`GROUP_CONCAT`，和`SAMPLE`。

查询器希望查看通过一组解决方案而不是单个解决方案计算出的结果时，使用聚合。例如，特定变量采用的最大值，而不是单独使用每个值。

### 11.1 聚合示例

**数据**

```turtle
@prefix : <http://books.example/> .

:org1 :affiliates :auth1, :auth2 .
:auth1 :writesBook :book1, :book2 .
:book1 :price 9 .
:book2 :price 5 .
:auth2 :writesBook :book3 .
:book3 :price 7 .
:org2 :affiliates :auth3 .
:auth3 :writesBook :book4 .
:book4 :price 7 .
```

**查询**

```SPARQL
PREFIX : <http://books.example/>
SELECT (SUM(?lprice) AS ?totalPrice)
WHERE {
  ?org :affiliates ?auth .
  ?auth :writesBook ?book .
  ?book :price ?lprice .
}
GROUP BY ?org
HAVING (SUM(?lprice) > 10)
```

**结果**

| totalPrice |
| ---------- |
| 21         |

此示例演示了聚合的两个功能：`GROUP BY`，该属性根据一个或多个表达式（在本例中为`?org`）对查询解决方案进行分组 ；以及`HAVING`，类似于`FILTER` 表达式，是在**组**而不是单个解决方案上运行。

该示例是通过根据`GROUP BY` 表达式对解决方案进行分组（即，`?org`具有特定值的所有解决方案都出现在同一组中），然后评估该组中的设置函数`SUM`来产生的。然后用`HAVING`表达式过滤组，该表达式将删除所有`SUM(?lprice)`不大于10的组。

在聚合查询和子查询中，出现在查询模式中但不在`GROUP BY`子句中的变量，如果它们被聚合，则只能在选择表达式中进行投影或使用。该 `SAMPLE`聚合可用于这一目的。有关详细信息，请参见“投影限制”部分。

应该注意的是，对于每个函数，都需要对聚合表达式进行别名化（同样，类似于`BIND`子句，使用关键字`AS`），以便从查询或子查询中对其进行投影。在上面的示例中，这是使用变量完成的`?totalPrice`。聚合使用其他聚合投影或`WHERE`子句中**已经使用的名称**来投影变量是**错误**的。

### 11.2 分组

为了计算解决方案的聚合值，首先将解决方案分为一个或多个组，然后为每个组计算合计值。

如果聚集体在查询级别使用`SELECT`， `HAVING`或`ORDER BY`但`GROUP BY`没有使用，则这被认为是一个单一的隐性组，属于所有的解决方案。

在`GROUP BY`子句中绑定关键字`AS`，可以使用诸如`GROUP BY (?x + ?y AS ?z)`。等同于`{ ... BIND (?x + ?y AS ?z) } GROUP BY ?z`。

例如，给定一个求解序列S`（{?x→2，?y→3}，{?x→2，?y→5}，{?x→6，?y→7}）`，我们可能希望根据?x的值对解进行分组，并计算每组?y的平均值。

可以这样写：

```SPARQL
SELECT (AVG(?y) AS ?avg)
WHERE {
  ?a :x ?x ;
     :y ?y .
}
GROUP BY ?x
```

### 11.3 拥有

`HAVING`对分组解决方案集进行操作，与`FILTER`对未分组解决方案集进行操作的方式相同。

`HAVING` 表达式具有与分组查询的投影相同的求值规则，如以下部分所述。

`HAVING`下面提供了使用示例。

```SPARQL
PREFIX : <http://data.example/>
SELECT (AVG(?size) AS ?asize)
WHERE {
  ?x :size ?size
}
GROUP BY ?x
HAVING(AVG(?size) > 10)
```

这将返回按主语分组的平均大小，但仅在平均大小大于10的情况下。

### 11.4 聚合投影限制

在使用聚合的查询级别中，只有一个由聚合和常量组成的表达式可以被投影，只有一个例外。当`GROUP BY`给出仅由<u>一个变量组成的一个或多个简单表达式</u>时，这些变量可以从级别投影。

例如，以下查询是合法的，因为?x作为`GROUP BY`术语给出。

```SPARQL
PREFIX : <http://example.com/data/#>
SELECT ?x (MIN(?y) * 2 AS ?min)
WHERE {
  ?x :p ?y .
  ?x :q ?z .
} GROUP BY ?x (STR(?z))
```

注意，投影`STR(?z)`不是合法的，因为这不是一个简单的变量表达式。但是，有`GROUP BY (STR(?z) AS ?strZ)`可能对`?strZ`进行投影。

其他不使用`GROUP BY`变量的表达式或集合可能具有使用`SAMPLE`集合从其组中投影的不确定性值。

### 11.5 带有错误的聚合示例

本节显示了一个使用聚合的示例查询，该示例演示了在存在聚合的情况下如何处理结果中的错误。

**数据**

```turtle
@prefix : <http://example.com/data/#> .

:x :p 1, 2, 3, 4 .
:y :p 1, _:b2, 3, 4 .
:z :p 1.0, 2.0, 3.0, 4 .
```

**查询**

```SPARQL
PREFIX : <http://example.com/data/#>
SELECT ?g (AVG(?p) AS ?avg) ((MIN(?p) + MAX(?p)) / 2 AS ?c)
WHERE {
  ?g :p ?p .
}
GROUP BY ?g
```

**结果**

|               g               | avg  |  c   |
| :---------------------------: | :--: | :--: |
| \<http://example.com/data/#x> | 2.5  | 2.5  |
| \<http://example.com/data/#y> |      |      |
| \<http://example.com/data/#z> | 2.5  | 2.5  |

请注意，:y组的绑定不包括在结果中，因为对`Avg（{1，_:b2，3，4}）`的求值，并且`（_：b2 + 4）/ 2`是错误的，因此删除了解决方案中的绑定。