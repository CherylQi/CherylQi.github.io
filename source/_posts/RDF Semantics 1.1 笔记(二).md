---
title: RDF 1.1 Semantics 笔记(二)
date: 2021-03-19 16:21:57
updated: 2021-04-03 09:21:00
tags: 语义Web
---

## 8. RDF 解释

RDF解释对`xsd:string`和部分带有命名空间前缀IRIs的无限集合的施加了额外的语义条件。

**RDF词汇表**

```
rdf:type 
rdf:subject 
rdf:predicate 
rdf:object 
rdf:first 
rdf:rest 
rdf:value 
rdf:nil 
rdf:List 
rdf:langString 
rdf:Property 
rdf:_1 
rdf:_2 ...
```

<!--more-->

识别D的RDF解释是D-interpretation ，其中D包括`rdf：langString`和`xsd：string`，并且满足：

RDF语义条件

```
当且仅当<x, I(rdf:Property)>属于IEXT(I(rdf:type))，x属于IP
对于在D中的每个IRI aaa，当且仅当x属于值空间I(aaa)，都有<x,I(aaa)>属于IEXT(I(rdf:type))
```

并且以下无限集合满足每个三元组

**RDF公理**

```
rdf:type rdf:type rdf:Property .
rdf:subject rdf:type rdf:Property .
rdf:predicate rdf:type rdf:Property .
rdf:object rdf:type rdf:Property .
rdf:first rdf:type rdf:Property .
rdf:rest rdf:type rdf:Property .
rdf:value rdf:type rdf:Property .
rdf:nil rdf:type rdf:List .
rdf:_1 rdf:type rdf:Property .
rdf:_2 rdf:type rdf:Property .
...
```

RDF对RDF词汇表中剩余的部分没有特殊的规范意义。附录D描述了其中一些词汇的预期用途。

<u>数据类型IRIs `rdf:langString`和`xsd:string`必须被所有的RDF解释识别。</u>

`rdf:XMLLiteral`和`rdf:HTML`定义在[RDF11-CONCEPTS].RDF-D interpretations 可能识别不出这些数据类型。

### 8.1 RDF蕴含

当每个满足S的RDF interpretation识别D 也满足E时，S RDF(识别D) 蕴含E。当D是{`rdf:langString, xsd:string`}时，可以说S RDF蕴含E。没有满足RDF interpretation时，E是RDF不可满足。

之前描述的简单蕴含的属性并不都适用于RDF蕴含。例如：所有RDF公理在RDF interpretation中都为true，因此空图RDF蕴含RDF公理，这与RDF蕴含的插值矛盾。

#### 8.1.1 RDF蕴含模式

上表中的最后一个语义条件给出了对于识别数据类型IRIs的以下蕴含模式：（文字只能做宾语）

|       | if S contains                                 | **then S RDF entails, recognizing D**       |
| ----- | --------------------------------------------- | ------------------------------------------- |
| rdfD1 | `xxx aaa `"`sss`"^^`ddd .`<br>` for ddd in D` | `xxx aaa _:nnn .`<br>`_:nnn rdf:type ddd .` |

<u>注意，当文字是错误的类型时，这是有效的，因为一个不可满足的图蕴含所有三元组。</u>

例如：

```
ex:a ex:p "123"^^xsd:integer .
```

RDF识别`xsd:integer`蕴含

```
ex:a ex:p _:x .
_:x rdf:type xsd:integer .
```

除此之外，第一个RDF语义条件证明了以下蕴含模式：

|       | **if S contains** | **then S RDF entails, recognizing D** |
| ----- | ----------------- | ------------------------------------- |
| rdfD2 | `xxx aaa yyy `.   | `aaa rdf:type rdf:Property .`         |

所以上例也RDF识别`xsd:integer`蕴含

```
ex:p rdf:type rdf:Property .
```

某些数据类型支持特有的包含模式，而其他数据类型则不适用。（穷举的方式推理出来）例如：

```
ex:a ex:p "true"^^xsd:boolean .
ex:a ex:p "false"^^xsd:boolean .
ex:v rdf:type xsd:boolean .
```

一起RDF 蕴含

```
ex:a ex:p ex:v .
```

识别`{xsd:boolean}`

除此之外，在值空间中的语义条件可能产生其他不可满足的图。例如，当D包含`xsd:integer`和`xsd:boolean`时，以下是RDF不可满足识别D：（相同空节点代表不同类型）

```
_:x rdf:type xsd:boolean .
_:x rdf:type xsd:integer .
```

## 9. RDFS 解释

RDF Schema 将RDF扩展了一个较大的带有更复杂语义约束的词汇表：

**RDFS词汇表：**

```
rdfs:domain 
rdfs:range 
rdfs:Resource 
rdfs:Literal 
rdfs:Datatype 
rdfs:Class 
rdfs:subClassOf 
rdfs:subPropertyOf 
rdfs:member 
rdfs:Container 
rdfs:ContainerMembershipProperty 
rdfs:comment 
rdfs:seeAlso 
rdfs:isDefinedBy 
rdfs:label
```

（这里包含`rdfs:comment`,` rdfs:seeAlso`, `rdfs:isDefinedBy`和`rdfs:label`，因为可以使用`rdfs:domain`,` rdfs:range`和`rdfs:subPropertyOf`声明的一些限制。除此之外，形式语义不限制他的含义。

用新的语义结构(**类**)，陈述RDFS语义很方便，也是表示全集中的该类且具有属性`rdf:type`值的一个集合。

类被定义为属性`rdfs：class`，并且在一个解释中所有类的集合都被称为**IC**。语义条件是根据从**IC**到**IR**的子集的映射**ICEXT**（**I**中的类扩展）陈述的。

<u>一个类可以有一个空的类扩展。两个不同的类可以有相同的类扩展</u>。`rdfs:Class`的类扩展包含类`rdf:Class`

一个RDFS 解释（识别D)是一个满足下表中的语义条件的RDF 解释（识别D）**I**，以及随后的RDFS定理三元组表中的所有三元组。

RDFS语义条件

```
ICEXT(y)定义为{x:<x,y>属于IEXT(I(rdf:type))}              //所有type是y的x，x的type是y
IC 定义为ICEXT(I(rdfs:Class))
LV 定义为ICEXT(I(rdfs:Literal))
ICEXT(I(rdfs:Resource)) = IR
ICEXT(I(rdfs:langString))是集合{I(E):E是一个语言标签字符串}
对于在D中的每个其他IRI aaa,ICEXT(I(aaa))是I(aaa)的值空间
对于在D中的每个IRI aaa,I(aaa)在ICEXT(I(rdfs:Datatype))中

如果<x,y>在IEXT(I(rdfs:domain))中并且<u,v>属于IEXT(x)中,则u属于ICEXT(y)    //u是类型y
如果<x,y>在IEXT(I(rdfs:range))中并且<u,v>属于IEXT(x),则v属于ICEXT(y)		//v是类型y

IEXT(I(rdfs:subPropertyOf))在IP上具有传递性和自反性

如果<x,y>属于IEXT(I(rdfs:subPropertyOf))，则x和y属于IP，且IEXT(x)是IEXT(y)的一个子集

如果x属于IC,则<x,I(rdfs:Resourcce)>属于IEXT(I(rdfs:subClassOf))

IEXT(I(rdfs:subClassOf))在IC上具有传递性和自反性

如果<x,y>属于IEXT(I(rdfs:subClassOf))，则x和y属于IC，且IEXT(x)是IEXT(y)的一个子集

如果x属于ICEXT(I(rdfs:ContainerMembershipProperty))则<x,I(rdfs:member)>属于 IEXT(I(rdfs:subPropertyOf))

如果x属于ICEXT(I(rdfs:Datatype))则<x,I(rdfs:Literal)>属于IEXT(I(rdfs:subClassOf))
```

**RDFS 公理三元组**

```
rdf:type rdfs:domain rdfs:Resource .
rdfs:domain rdfs:domain rdf:Property .
rdfs:range rdfs:domain rdf:Property .
rdfs:subPropertyOf rdfs:domain rdf:Property .
rdfs:subClassOf rdfs:domain rdfs:Class .
rdf:subject rdfs:domain rdf:Statement .
rdf:predicate rdfs:domain rdf:Statement .
rdf:object rdfs:domain rdf:Statement .
rdfs:member rdfs:domain rdfs:Resource .
rdf:first rdfs:domain rdf:List .
rdf:rest rdfs:domain rdf:List .
rdfs:seeAlso rdfs:domain rdfs:Resource .
rdfs:isDefinedBy rdfs:domain rdfs:Resource .
rdfs:comment rdfs:domain rdfs:Resource .
rdfs:label rdfs:domain rdfs:Resource .
rdf:value rdfs:domain rdfs:Resource .

rdf:type rdfs:range rdfs:Class .
rdfs:domain rdfs:range rdfs:Class .
rdfs:range rdfs:range rdfs:Class .
rdfs:subPropertyOf rdfs:range rdf:Property .
rdfs:subClassOf rdfs:range rdfs:Class .
rdf:subject rdfs:range rdfs:Resource .
rdf:predicate rdfs:range rdfs:Resource .
rdf:object rdfs:range rdfs:Resource .
rdfs:member rdfs:range rdfs:Resource .
rdf:first rdfs:range rdfs:Resource .
rdf:rest rdfs:range rdf:List .
rdfs:seeAlso rdfs:range rdfs:Resource .
rdfs:isDefinedBy rdfs:range rdfs:Resource .
rdfs:comment rdfs:range rdfs:Literal .
rdfs:label rdfs:range rdfs:Literal .
rdf:value rdfs:range rdfs:Resource .

rdf:Alt rdfs:subClassOf rdfs:Container .
rdf:Bag rdfs:subClassOf rdfs:Container .
rdf:Seq rdfs:subClassOf rdfs:Container .
rdfs:ContainerMembershipProperty rdfs:subClassOf rdf:Property .

rdfs:isDefinedBy rdfs:subPropertyOf rdfs:seeAlso .

rdfs:Datatype rdfs:subClassOf rdfs:Class .

rdf:_1 rdf:type rdfs:ContainerMembershipProperty .
rdf:_1 rdfs:domain rdfs:Resource .
rdf:_1 rdfs:range rdfs:Resource .
rdf:_2 rdf:type rdfs:ContainerMembershipProperty .
rdf:_2 rdfs:domain rdfs:Resource .
rdf:_2 rdfs:range rdfs:Resource .
```

因为 **I** 是RDF解释，所以第一个条件意味着IP=ICEXT(I(rdf:Property))

RDF解释中的语义条件，以及ICEXT的RDFS条件，意味着可以将每个已识别的数据类型都视为一个类，其扩展名是该数据类型的值空间，每个具有该数据类型的文字要么无法引用，要么引用该类中的值。

当使用RDFS语义时，所有公认的数据类型IRI的引用都可以认为时这类`rdfs:Datatype`

上面列出的公理和条件有一些冗余。例如，除了一个RDF公理三元组外，其他所有子集都可以从RDFS公理三元组和ICEXT上的语义条件，`rdfs:domain`和`rdfs:range`中得出。

在所有RDFS解释中必须正确的其他三元组包括以下内容。这是一个不完全的集合。

rdfs有效的三元组

```
rdfs:Resource rdf:type rdfs:Class .
rdfs:Class rdf:type rdfs:Class .
rdfs:Literal rdf:type rdfs:Class .
rdf:XMLLiteral rdf:type rdfs:Class .
rdf:HTML rdf:type rdfs:Class .
rdfs:Datatype rdf:type rdfs:Class .
rdf:Seq rdf:type rdfs:Class .
rdf:Bag rdf:type rdfs:Class .
rdf:Alt rdf:type rdfs:Class .
rdfs:Container rdf:type rdfs:Class .
rdf:List rdf:type rdfs:Class .
rdfs:ContainerMembershipProperty rdf:type rdfs:Class .
rdf:Property rdf:type rdfs:Class .
rdf:Statement rdf:type rdfs:Class .

rdfs:domain rdf:type rdf:Property .
rdfs:range rdf:type rdf:Property .
rdfs:subPropertyOf rdf:type rdf:Property .
rdfs:subClassOf rdf:type rdf:Property .
rdfs:member rdf:type rdf:Property .
rdfs:seeAlso rdf:type rdf:Property .
rdfs:isDefinedBy rdf:type rdf:Property .
rdfs:comment rdf:type rdf:Property .
rdfs:label rdf:type rdf:Property .
```

RDFS不会将全集划分为类别，属性和个人的不相交的类别。全集中的任何事物都可以用作类或属性，或两者兼有，同时保留其作为个体的状态（可能在类中并具有属性）。因此，RDFS允许包含其他类，属性类，类属性等的类。如上面的公理三元组所示，它还允许包含自身的类和适用于其的属性。类的属性不一定是其成员的属性，反之亦然。

### 9.1 rdfs:Literal笔记

类`rdfs:Literal`不是文字的类，而是文字的**值**的类，IRIs也可能会引用。例如：LV不包含文字"foodle"^^xsd:string，但是包含字符串"foodle"。

一个三元组`ex:a rdf:type rdfs:Literal .`即使主语是IRI而不是文字也一致。这说明IRI 'ex:a'指的是文字值，因为文字值在全集中，所以这是有可能的。同样，空节点的范围可能超出文字值。

### 9.2 RDFS蕴含

当每个满足S的RDF interpretation识别D时，S RDFS 蕴含E识别D。

因为每个RDFS解释都是一个RDF解释，所以如果S RDFS蕴含E，则S也RDF蕴含E，但是RDFS蕴含强于RDF蕴含。甚至一些空图有更多的RDFS蕴含，而不是RDF蕴含，例如：

```
aaa rdf:type rdfs:Resource .
```

当aaa是一个IRI，在所有RDFS解释中为true

#### 9.2.1 RDFS蕴含模式

RDFS蕴涵适用于以下所有模式，这些模式与RDFS语义条件紧密对应：

|        | If S contains:                                               | then S RDFS entails recognizing D:     |
| ------ | ------------------------------------------------------------ | -------------------------------------- |
| rdfs1  | any IRI aaa in D                                             | aaa `rdf:type rdfs:Datatype .`         |
| rdfs2  | aaa `rdfs:domain` xxx .<br> yyy aaa zzz  .                   | yyy `rdf:type` xxx .                   |
| rdfs3  | aaa `rdfs:range` xxx  .<br> yyy aaa zzz  .                   | zzz `rdf:type` xxx .                   |
| rdfs4a | xxx aaa yyy .                                                | xxx `rdf:type rdfs:Resource .`         |
| rdfs4b | xxx aaa yyy.                                                 | yyy `rdf:type rdfs:Resource .`         |
| rdfs5  | xxx `rdfs:subPropertyOf` yyy .<br> yyy `rdfs:subPropertyOf` zzz . | xxx `rdfs:subPropertyOf` zzz .         |
| rdfs6  | xxx `rdf:type rdf:Property .`                                | xxx `rdfs:subPropertyOf` xxx .         |
| rdfs7  | aaa `rdfs:subPropertyOf` bbb .<br> xxx aaa yyy  .            | xxx bbb yyy .                          |
| rdfs8  | xxx `rdf:type rdfs:Class .`                                  | xxx `rdfs:subClassOf rdfs:Resource .`  |
| rdfs9  | xxx `rdfs:subClassOf` yyy `.` zzz `rdf:type` xxx `.`         | zzz `rdf:type` yyy `.`                 |
| rdfs10 | xxx `rdf:type rdfs:Class .`                                  | xxx `rdfs:subClassOf` xxx `.`          |
| rdfs11 | xxx `rdfs:subClassOf` yyy .<br> yyy `rdfs:subClassOf` zzz  . | xxx `rdfs:subClassOf` zzz `.`          |
| rdfs12 | xxx `rdf:type rdfs:ContainerMembershipProperty .`            | xxx `rdfs:subPropertyOf rdfs:member .` |
| rdfs13 | xxx `rdf:type rdfs:Datatype .`                               | xxx `rdfs:subClassOf rdfs:Literal .`   |

RDFS提供了几种不可满足的识别D的新方法。例如，下图是RDFS不可满足的识别{`xsd:integer`, `xsd:boolean`}:

```
ex:p rdfs:domain xsd:boolean .
ex:a rdf:type xsd:integer .
ex:a ex:p ex:c .
```

## 10. RDF数据集

RDF数据集，定义在RDF Concepts[RDF11-CONCEPTS]中，打包零个或多个已命名的RDF图以及一个未命名的默认RDF图。单个数据集中的图可以共享空节点 。SPARQL [SPARQL11-QUERY]将图名IRI与图关联，以允许针对特定图进行查询。

数据集中的图名可能引用与其配对的图以外的其他名称。这允许在数据集中使用引用其他类型的实体（例如人）的IRI来标识与图名IRI表示的实体相关信息的图。

当在数据集中的RDF三元组内使用图形名称时，它可能引用或可能不引用其名称的图。在没有外部原因的情况下，RDF引擎不需要语义，也不需要RDF引擎假定，RDF三元组中使用的图名称引用它们命名的图。

RDF数据集可用于表达RDF内容。当以这种方式使用时，应将数据集理解为其默认图至少具有相同的内容。但是请注意，用逻辑上等效的图替换数据集的默认图通常不会产生结构上相似的数据集，因为它可能会破坏默认图与数据集中其他图之间空节点的共现，除了数据集中图的语义之外，其他原因可能很重要。除了数据集中图形的语义之外，其他原因可能很重要。

就像RDF图一样，其他语义扩展和包含方式也可能对RDF数据集设置进一步的语义条件和限制。例如，一个此类扩展可以建立一种类似模型的解释结构，以便数据集之间的蕴含将要求具有相同名字的图之间的RDF图蕴含（根据需要添加空图）。

## 附录

### A. 蕴含规则

上表中列出的RDF和RDFS蕴含模式可以视为从左到右的规则，这些规则将所需的结论添加到图中。通过以下操作序列，这些规则集可用于检查图S和E之间是否RDF（或RDFS）蕴含：

1. 将所有RDF（或RDF和RDFS）公理三元组添加到S中，除了那些包含容器成员属性IRIs的`rdf:_1, rdf:_2, ...`。
2. 对于E中出现的每个容器成员属性IRI，添加包含该IRI的RDF（或RDF和RDFS）公理三元组到S中。
3. 将RDF（或RDF和RDFS）推理模式作为规则，穷举每个结论添加到图中；也就是说，直到它们不生成新的三元组。
4. 确定E是否有一个实例是集合的子集，即大集合是否蕴含E。

此过程显然是正确的，因为如果给出肯定的结果，则S确实RDF（RDFS）蕴含E。但是，它并不完整：有些情况下S蕴含E的情况无法通过此过程检测到。示例包括：

|                                                              | RDF entails                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `ex:a ex:p "string"^^xsd:string .`<br>`ex:b ex:q "string"^^xsd:string .` | `ex:a ex:p _:b .`<br>`ex:b ex:q _:b .`<br>`_:b rdf:type xsd:string .` |
|                                                              | **RDFS entails**                                             |
| `ex:a rdfs:subPropertyOf _:b .`<br>`_:b rdfs:domain ex:c .`<br>`ex:d ex:a ex:e .` | `ex:d rdf:type ex:c .`                                       |

这两种情况可以通过允许将规则应用于RDF广义语法处理，其中文字可能出现在主语位置，而空节点可能出现在谓语位置。

考虑广义RDF三元组，图和数据集，而不是RDF三元组，图和数据集（扩展了[HORST04]中使用的泛化，并严格遵循[ OWL2-PROFILES]中使用的术语）。本文档中描述的语义适用于广义而无需更改，因此可以自由使用解释，可满足性和必要性的概念。然后，我们可以用更简单，更直接的方式替换第一个RDF蕴含模式。

广义RDF蕴含模式

|        | **if S contains**                   | **then S RDF entails, recognizing D** |
| ------ | ----------------------------------- | ------------------------------------- |
| GrdfD1 | `xxx aaa "sss"^^ddd . for ddd in D` | `"sss"^^ddd rdf:type ddd .`           |

给出蕴含

```
ex:a ex:p "string"^^xsd:string .
ex:b ex:q "string"^^xsd:string .
"string"^^xsd:string rdf:type xsd:string .
```

这是上面结论的一个实例（广义RDF）。

第二个示例由RDFS规则得出：

```
ex:a rdfs:subPropertyOf _:b .
_:b rdfs:domain ex:c .
ex:d ex:a ex:e .
ex:d _:b ex:c . 								//by rdfs7
ex:d rdf:type ex:c .							//by rdfs2
```

在将蕴含模式应用于广义RDF语法，得出最终结论即合法RDF。

根据广义语法，这些规则对于RDF和RDFS蕴含是完备的。确切地说：

S和E为RDF图。定义从S到E的广义RDF(RDFS)闭包，通过下述过程得到一个集合。

1. 将所有RDF不包含容器成员属性IRI的公理三元组添加到S中。
2. 对每个在E中出现的容器成员属性IRI，将包含这些IRI RDF(RDFS)公理三元组添加到S中。
3. 如果在第二步中没有添加三元组，则添加包含`rdf:_1`的RDF(RDFS)公理三元组。
4. 应用规则GrdfD1和rdfD2(通过rdfs13的规则rdfs1)，带有D={`rdf:langString, xsd:string`}，到集合中，以进行穷举。

然后得出完整性结果：

```
如果S是RDF(RDFS)一致，则当从S到E的广义RDF闭包蕴含E时，S RDF(RDFS)蕴含E。
```

闭包是有限的。生成过程是可确定的并且具有多项式复杂性。通常，检测简单蕴含是NP完全的，但是当E不包含空节点时，其多项式阶数较低。

每个RDF（S）闭包，甚至从空图开始，都将包含所有RDF（S）永真式，可以使用原始图的词汇表以及RDF和RDFS词汇表来表达。在实践中，重新推导这些规则几乎没有用处，并且可以使用规则的子集来建立大多数具有实际意义的内容。

如果保持合法的RDF语法很重要，则可以使用规则[rdfD1](https://www.w3.org/TR/rdf11-mt/#dfn-rdfd1)代替[GrdfD1](https://www.w3.org/TR/rdf11-mt/#dfn-grdfd1)，并且可以将引入的空节点用作后续派生文字的替代物。但是，最终的规则集将不完整。

如前所述，检测较大数据类型IRI集的数据类型需求需要注意特定数据类型的特质属性。

### B. 有限的解释

为了使说明简单，对RDF语义的措辞要求解释的范围大于绝对必要的范围。例如，解释全部IRI词汇需要所有解释，并且D所包含 `xsd:string`的所有D解释的全集都必须包含所有可能的字符串，因此是无限的。本附录在没有证明的情况下给出了如何使用较小的语义结构重述语义而又不更改任何内容的概述。

基本上，仅需要一个解释结构来解释实际在考虑其含义的图中实际使用的名字，并考虑其全集至多与图中的名字和空节点数量一样大的解释。更正式地，我们可以定义一个预先解释过的词汇表V到结构**I**类似简单的解释，但只有从V到它的全集IR的映射。然后，在确定G是否蕴含E时，仅考虑对G联合E中实际使用的名称的有限词汇的预解释。此类预解释的范围可以限于基数<u>N+B+1</u>，其中N是词汇量，B是图中空节点的数量。任何这样的预解释都可以扩展为简单的解释s，所有这些都将为G或E中的任何三元组提供相同的真值。然后，可以针对这些有限的预解释来定义可满足性，蕴含度等，并且显示与规范正文中定义的思想相同。

在考虑D蕴含时，可以通过削弱数据类型文字的语义条件来使预解释保持有限，以使IR仅包含实际出现在G或E中的文字的文字值，并且全集的大小限制为<u>（N+B）×（D+1）</u>，其中D是已识别数据类型的数量。（可能会有更严格的界限。）对于RDF蕴含而言，只有RDF词汇表的有限部分（包括那些实际在图中出现的容器成员属性）才需要解释，第二个RDF语义条件被弱化以仅应用于值，这是实际出现在词汇表中的文字值。对于RDFS解释，同样，只需要解释在所考虑的图中实际出现的无限容器成员属性词汇表的有限部分。在所有这些情况下，可以将图词汇的预解释扩展为对适当类型的完整解释，而无需更改图中任何三元组的真值。

如果认为*有限模型属性*很重要，则可以按照预解释的方式描述整个语义，产生相同的含义，并允许在有限的结构中解释有限的RDF图。

### C. 证明

```
任何图都简单蕴含空图，但是空图不简单蕴含任何图，除了空图本身。
```

在简单的解释中，空图为真，所以可以被任何图蕴含。

如果G包含一个三元组<a,b,c>，则任何带有IEXT(I(b)) = {}的简单解释**I**，都使G为false；所以空图不蕴含G。

```
一个图简单蕴含它的所有子图。
```

如果**I**满足G，则它满足G中的所有三元组，因此满足G中任何子集的每个三元组。

```
一个图的所有实例都简单蕴含这个图。
```

假设H是G经过M实例化映射的一个实例，并且**I**满足H。对于不在H中，但在G中的空节点n，定义`A(n)=I(M(n))`；则`I+A`满足G，所以**I**满足G。

```
每个图都是简单可满足的。
```

考虑到带有全集{x}的简单解释，IEXT(x)=<x,x>，并且对于任何IRI aaa有I(aaa) = x。这个解释满足每个RDF图。

```
当且仅当G的子图是E的一个实例时，G简单蕴含图E
```

如果G的子图E'是E的一个实例，则G蕴含E'，该E'蕴含E，因此G蕴含E。现在假设G蕴含E，并且考虑Herbrand解释 **I**。IR包含图中出现的名称和空白节点，每个名称n的I(n)= n；当三元组\<a n b>在图中时，n在IP中，<a,b>在IEXT（n）中。（对于图中未出现的IRI，请在IR中为它们随机分配值。）**I**满足E中所有的三元组<s,p,o>；也就是说，对一些从E中的空节点到词汇表G的映射A，三元组<[I+A]\(s), I(p), [I+A]\(o)>出现在G中。但是这些是在实例映射A下的\<s p o>实例，所以E的一个实例时G的子图。

```
如果E是精简的，并且E'是E的一个真实例，则E不简单蕴含E'
```

假设E蕴含E'，则E的子图是E'的一个实例，同时E'是E的真实例，所以E的子图是E的真实例，所以E不是精简的。

```
如果E包含一个在S中没有出现的IRI，则S不简单蕴含E。
```

如果S蕴含E，则S的子图是E的一个实例，所以每个E中的IRI必须出现在该子图中，因此必须出现在S中。

```
对每个图H，如果sk(G)简单蕴含H，则存在图H',使得G蕴含H',并且H=sk(H')
```

skolemization映射sk为每一个空节点替换一个新的IRI，因为它是1：1的，所以有可逆性。定义ks是替换每个空节点的skolem IRI的逆映射。因为sk(G)蕴含H，一个sk(G)的子图是H的一个实例，对H中的空节点的实例映射A，可以称为A(H)。ks(A(H))是G的一个子图，并且ks(A(H))=A(ks(H))，因为A和ks的定义域是不相交的。所以ks(H)有一个实例是G的子图，所以被G蕴含，并且H=sk(ks(H))。

```
对任何不包含sk(G)中的"新"IRIs的图H，当且仅当G简单蕴含H时，sk(G)简单蕴含H
```

如果H不包含任何skolem IRIs，则H=ks(H)。所以如果sk(G)蕴含H，则G蕴含ks(H)=(H)，并且如果G蕴含H，则sk(G)蕴含G蕴含H，所以sk(G)蕴含H。

### D. RDF具体化，容器和集合

RDF语义条件并未对要用于描述容器和集合的RDF词汇表的含义或任何旨在使RDF图描述RDF三元组的说明化词汇表的形式施加正式的约束。本附录简要回顾了该词汇表的预期含义。

从形式语义上省略这些条件是一项设计决策，以适应现有RDF使用情况的变化，并使其更易于实施检查形式RDF蕴含的过程。例如，实现可以决定使用特殊的程序技术来实现RDF集合词汇表。

#### D.1 具体化

| **RDF reification vocabulary**                               |
| ------------------------------------------------------------ |
| `rdf:Statement`<br>`rdf:subject`<br>`rdf:predicate` <br>`rdf:object` |

该词汇表的预期含义是允许RDF图充当描述其他RDF三元组的元数据。

考虑一个包含单个三元组的示例图：

```
ex:a ex:b ex:c .
```

并且假设使用IRI`ex:graph1`定义这个图。RDF模型的确切实现方式是在RDF模型的外部进行的，但这可能是通过IRI解析为描述该图的具体语法文档，或者是通过IRI作为数据集中命名图的关联名称而实现的。假设可以使用IRI来指代三元组，那么通过具体化词汇表可以使我们在**另一张图中描述第一个图**：

```
ex:graph1 rdf:type rdf:Statement .
ex:graph1 rdf:subject ex:a .
ex:graph1 rdf:predicate ex:b .
ex:graph1 rdf:object ex:c .
```

第二张图在第一张图中称为三元组的具体化。

具体化不是一种引述形式。确切地说，这种具体化描述了三元组的令牌与该三元组所引用的资源之间的关系。`rdf:subject`属性的值不是主语IRI本身，而是它所表示的事物，并且对于`rdf:predicate`和`rdf:object`也相似。例如，如果`ex:a`引用的是珠穆朗玛峰，则具体化三元组的主语也是山峰，而不是它引用的IRI。

<u>具体化可以被写作以空节点为主语，或者一个IRI主语没有被任何具体化实现的三元组所定义，在这两种情况下，他们只是断言所描述的三元组存在。</u>

具体化的主语旨在指代RDF三元组的具体实现，例如表面语法中的文档，而不是一个被认为是抽象宾语的三元组。这支持了将例如日期或出处信息应用于具体化三元组，这些用例仅在认为是指三元组的特殊实例或令牌时才有意义。

<u>一个三元组的具体化不蕴含三元组，并且也不被它蕴含。</u>具体化只是说三元组的令牌存在，以及它是关于什么，并不是说它是true，因此它不蕴含三元组。另一方面，断言一个三元组不会自动地意味着任何一个三元组的令牌存在一个由三元组描述的全集中。？例如，三元组可能成为描述动物的本体的一部分，这个本体满足一个仅包含动物的全集的解释，因此对它的具体化是错误的。

因为三元组和具体化三元组之间的关系不需要是一对一的，断言一个关于一些由具体化描述的属性不一定意味着另一个实体拥有相同的属性，即使它有相同的组成部分。例如：

```
_:xxx rdf:type rdf:Statement .
_:xxx rdf:subject ex:subject .
_:xxx rdf:predicate ex:predicate .
_:xxx rdf:object ex:object .
_:yyy rdf:type rdf:Statement .
_:yyy rdf:subject ex:subject .
_:yyy rdf:predicate ex:predicate .
_:yyy rdf:object ex:object .
_:xxx ex:property ex:foo .
```

不蕴含

```
_:yyy ex:property ex:foo .
```

#### D.2 RDF容器

| **RDF(S) Container Vocabulary**                              |
| ------------------------------------------------------------ |
| `rdf:Seq`<br>`rdf:Bag `<br>`rdf:Alt`<br>` rdf:_1 `<br>`rdf:_2` ...<br>` rdfs:member`<br>`rdfs:Container`<br>`rdfs:ContainerMembershipProperty` |

RDF提供用于描述三类容器的词汇。容器具有类型，可以通过使用一组固定的*容器成员属性*来枚举其成员。这些属性由整数索引，以提供一种区分成员的方法，但是这些索引不必一定被认为是定义容器本身的顺序。一些容器被认为是无序的。

所述RDFS词汇增加，通用的不受位置限制的通用成员属性，以及包含所有的容器类和所有成员的属性。

人们应该将此词汇表理解为*描述* 容器，而不是像通常由编程语言提供的那样作为构建容器的工具。实际的容器是语义世界中的实体，使用词汇表的RDF图仅提供有关这些实体的非常基本的信息，从而使RDF图能够表征容器的类型并提供有关容器成员的部分信息。由于RDF容器词汇如此有限，因此RDF形式语义不能正式认可与RDF容器有关的许多自然假设。这不应被视为意味着这些假设是错误的，而仅仅是RDF并不正式意味着它们必须是正确的。

容器词汇表上没有特殊的语义条件：<u>RDF假定其容器具有的唯一结构是可以从使用该词汇表和一般RDF语义条件中推断出的结构。</u>这相当于知道容器的类型，并部分枚举了容器中的物品。预期的使用方式是类型`rdf:Bag` 被认为是无序的，但允许重复；类型`rdf:Seq`被认为是有序的，类型`rdf:Alt`被视为代表备选方案的集合，可能具有优先顺序。如果容器是有序类型，则打算通过容器成员属性的数字顺序来指示容器中项目的顺序，这些成员属性被假定为单值。但是，这些非正式条件并未反映在任何正式的RDF要求中。

RDF语义不支持`rdf:Bag`以不同顺序枚举无序元素可能引起的任何问题。例如，

```
_:xxx rdf:type rdf:Bag .
_:xxx rdf:_1 ex:a .
_:xxx rdf:_2 ex:b .
```

不蕴含

```
_:xxx rdf:_1 ex:b .
_:xxx rdf:_2 ex:a .
```

（如果这个结论是有效，则将它添加到原始图的结果被图蕴含，并且这将断言两个元素都是在两个位置，这是以下事实的一个结果是RDF是一种纯粹的assertional语言。）

没有假定容器的属性适用于容器的任何元素，反之亦然。

没有正式的要求，这三个容器类是不相交的，因此例如可以断言某物既是`rdf:Bag`又是 `rdf:Seq`。没有假设容器是无间隙的，因此例如

```
_:xxx rdf:type rdf:Seq.
_:xxx rdf:_1 ex:a .
_:xxx rdf:_3 ex:c .
```

不蕴含

```
_:xxx rdf:_2 _:yyy .
```

RDF中无法断言容器仅包含固定数量的成员。这反映了一个事实，即向图添加三元组以声明任何容器的隶属关系属性始终是一致的。最后，没有内置的假设认为RDF容器只有有限的许多成员。

#### D.3 RDF集合

| **RDF Collection Vocabulary**                        |
| ---------------------------------------------------- |
| `rdf:List`<br>`rdf:first`<br>`rdf:rest`<br>`rdf:nil` |

RDF提供了一个用头尾链接来描述集合的词汇，即“列表结构”。集合与容器的不同之处在于允许分支结构和具有显式终止符，从而允许应用程序确定集合中项目的确切集合。

至于容器，没有特殊的语义条件强加给这个词汇的其他类型`rdf:nil`是`rdf:List`。它通常用于在使用空节点描述容器以连接“格式正确”的项目序列的情况下使用，每个项目均由这样的两个三元组描述：

```
_:c1 rdf:first aaa .
_:c1 rdf:rest _:c2 .
```

通过使用`rdf:nil`用作`rdf:rest`的值来指示最后一项。按照惯例，`rdf:nil` 可以将其视为空集合。任何这样的图都等于断言该集合存在，并且由于可以通过检查来确定集合的成员，因此这通常足以使应用程序确定含义。除了图中明确提到的那些集合（以及空集合）之外，语义不需要任何集合存在。例如，包含两个项目的集合的存在并不能自动保证排列了项目的相似集合也存在：

```
_:c1 rdf:first ex:aaa .
_:c1 rdf:rest _:c2 .
_:c2 rdf:first ex:bbb .
_:c2 rdf:rest rdf:nil .
```

不蕴含

```
_:c3 rdf:first ex:bbb .
_:c3 rdf:rest _:c4 .
_:c4 rdf:first ex:aaa .
_:c4 rdf:rest rdf:nil .
```

同样，RDF在使用此词汇表时不施加“格式正确”的条件，因此可以编写RDF图来断言存在高度特殊的宾语，例如<u>具有分叉或非列表尾部的列表或多个头的</u>列表：

```
_:666 rdf:first ex:aaa .
_:666 rdf:first ex:bbb .
_:666 rdf:rest ex:ccc .
_:666 rdf:rest rdf:nil .
```

也有可能编写一组三元组，这些三元组由于未指定其`rdf:rest`属性值而未充分指定一个集合。

语义扩展可能会对该词汇的使用施加额外的语法良好格式限制，以排除此类图。它们可能排除对集合词汇的解释，这些解释违反了约定，即上述形式两个的三元组的“链接”集合的主语（以`rdf:nil`结尾的项结尾），表示一个完全有序的序列，其成员是这些项目的rdf：first值的表示形式，从主语到`rdf:nil`，通过跟踪属性`rdf:rest`的顺序。这允许序列包含其他序列。

RDFS语义条件要求该`rdf:first`属性的任何主语以及该属性的任何主语或宾语`rdf:rest`都应为`rdf:type rdf:List`。