---
title: RDF 1.1 Semantics 笔记(一)
date: 2021-03-19 16:20:49
updated: 2021-04-03 09:20:53
tags: 语义Web
---

## 1. 引言

这篇文档为RDF图和RDF及RDFS词汇表定义了一个模型-理论语义，为转换RDF或者从其他RDF派生RDF内容的操作时，提供了正式的确切规定。

## 2. 标准

除了标记为非规范之外，所有的指南，图表，示例和注释都是非规范。其他内容均为规范。

这篇文档包含了RDF的语义规范以及RDF推理过程的有效性。对于RDF含义的许多方面而言，这不是规范的，这些语义并未对此进行描述或指定，包括社会问题，这些问题涉及如何分配IRI使用中的含义，以及IRI的引用对象如何与其他媒体（如自然语言文本）中表达的Web内容相关联。

<!--more-->

## 3. 语义扩展以及蕴含

RDF旨在作为多种OWL和RIF的扩展概念的基本概念，表达式可以编码为RDF图，该图使用了给定含义的词汇表。同样，特定的IRI词汇表可以通过其他规定赋予含义。如果假设了这些额外的含义，则给定的RDF图可能支持比基本RDF语义所确定的范围更大。通常，在RDF图中对IRI的含义所做的假设越多，这些假设带来的蕴含就越多。

这样一组特定的语义假设称为**语义扩展**。每一个语义扩展定义了一个在该扩展下有效的**蕴含规则**。RDFs，就是这样的语义扩展。可以像RDFs蕴含和D蕴含等命名。

语义扩展可能在RDF图上强加一些特殊的语法条件或限制，例如要求存在某些三元组，或者禁止将IRI进行三元组特殊组合，并且可以将不符合条件的RDF图视为错误。例如，RDF语句

```rdf
ex:a rdfs:subClassOf "Thing"^^xsd:string .
```

在基于描述逻辑的OWL语义扩展中是禁用的。在这种情况下，基本的RDF操作（例如获取三元组的子集或组合RDF图）可能会在识别扩展条件的解析器中引起语法错误。本文档中规范定义的语义扩展都没有对RDF图加入这样的语法限制。

所有的蕴含必须是本篇文档中描述的简单蕴含的单调扩展，在这个意义上，如果A蕴含B，则在满足所有语法条件的蕴含扩展的概念中，A都是蕴含B。换句话说，尽管语义扩展可以将结果视为语法错误，但不能取消由弱蕴含推导的蕴含。

## 4. 符号及术语

本篇文档使用如下的术语描述RDF图，所有内容均在RDF Concepts规范中定义：**IRI, RDF triple graph,subject predicate, object, RDF source, node, blank node, literal, isomorphic(同构) and RDF dataset.**本篇文档中的定义均不变地适用于广义RDF三元组，RDF图和数据集。

<u>一个**解释**是从IRI和文字到一个集合的**映射**，以及对该集合和映射的一些约束。</u>这篇文档定义了各种解释的概念，每个概念都标准的对应蕴含规则。有一些通过前缀来标识，例如*simple interpretation*等。一般而言，不合格的术语“解释”通常用于指代任何兼容的解释类型，但是， 如果在文本中是清晰的，可能指的是特定类型的解释。

词语**denotes**和**refers to**可以互换使用，二者是同义词，<u>作为IRI或文字与它在给定解释中所指的事物之间的关系</u>，该解释本身称为“**denotation**”或“**referent**”。IRI的含义也可以由RDF语义外部的其他约束来确定，当我们希望引用这样额外定义的命名关系时，使用"**identify**"。例如，事实IRI`http://www.w3.org/2001/XMLSchema#decimal`被广泛的用作XML模式文档的数据类型的名字，可以通过IRI “*identifies*”  数据类型来描述。如果IRI identifies事物，则在给定的解释中可能会也可能不会引用它，具体取决于如何指定语义。例如，用作图名的IRI，定义了RDF数据集中的命名图，可能引用与其标识的图不同的事物。

本篇文档中，=表示严格标识。语句`A=B`的意思是，A和B都涉及的一个实体。尖括号`< x, y >`用作表示一个有序对x和y。

在整个文档中，使用Turtle语法[TURTLE]的符号约定编写RDF图和RDF抽象语法的其他片段。命名空间前缀`rdf: rdfs: xsd: `  。当确切的IRI不重要时,使用`ex`(example)作为前缀。在陈述一般规则或条件时，我们使用三个字符的变量例如`aaa,xxx,sss`表明任意的IRIs，文字或其他RDF语法成分。可以由节点-弧图直接显示图结构。

**name**可以是任何IRI或者文字。一个类型文字包含两个name：一个是它本身，另一个是它内部类型的IRI。一个**词汇表**是name的集合。

**empty graph**是空三元组的集合。

**subgraph**一个RDF图的子图，是图中三元组的子集。一个三元组通过包含它的单例集来标识，所以图中的每个三元组都被视为一个子图。一个**proper subgraph**是图中三元组真子集。

**ground**是不包含空节点的图。

假设M是从一组**空节点**到一组**文字**、**空节点**和**IRIs**的函数<u>映射</u>。<u>通过用M(N)替换图G中的部分或全部空节点N，得到的任何图都是G的实例</u>。任何图都是它本身的实例，G实例的实例也是G的实例，并且如果H是G的实例，则H的所有三元组都是G中至少一个三元组的实例。

**instance with respect to** 词汇表V的实例，实例中所有用来替换原始词汇中空节点的names都是来自V的。

**proper instance** 是一个空节点已经被name替换的实例，或者图中的两个空节点已经被映射到实例中的同一个节点。（不可以映射到空节点）

当两个图中空节点1：1**互相**映射时，则两个图是<u>同构</u>的。同构图是具有可逆实例映射的**相互实例**。由于空节点除了在图中的位置外没有其他特定的标识，<u>因此我们常将同构图视为相同的。</u>

如果一个RDF图，没有真子图是它的实例，则它是*lean*的（**不包含空节点**）。<u>没有实例作为其自身的**真**子图。</u>	非lean的图内部冗余，并且表达和他们的lean子图内容相同。例如：	

```
ex:a ex:p _:x .           //not lean
_:y ex:p _:x .
```

```
ex:a ex:p _:x .				//lean
_:x ex:p _:x . 	
```

ground 图都是lean

### 4.1 共享空节点，联合与合并

图只有在它从由文档或其他结构（例如RDF数据集）描述的图派生而来时才共享空节点，这些文件或其他结构明确规定了在不同RDF之间共享空节点。仅仅下载Web文档并不意味着结果RDF图中的空节点，与来自相同文件的其他下载或相同RDF源的空节点相同。

处理使用空节点标识符的RDF的具体语法的RDF应用程序应注意追踪他们标识的空节点的身份。空节点标识符通常有局部作用域，所以当组合来自不同的RDF源时，可能必须更改标识符，以避免不同空节点的意外合并。

例如，两个文档可能都使用空节点标识符“:\_x”定义空节点，但是除非这些文档在共享标识符范围内或从共同的来源派生，一个中出现的":\_x"会标识与另一个文档所描述的图中不同的空节点。当通过组合来自多个来源的RDF形成图时，可能有必要通过用其他文档中未出现的其他节点替换空节点标识符来**标准化**空白节点标识符。例如：

```
ex:a ex:p _:x .
```

![example1](https://tva3.sinaimg.cn/large/005SZbikly1gp4jsrv4rrj309101zdfr.jpg)

```
ex:b ex:q _:x .
```

![example2](https://tvax4.sinaimg.cn/large/005SZbikly1gp4jtct4nxj309201za9z.jpg)

联合后应该包含四个节点。

![union1](https://tva3.sinaimg.cn/large/005SZbikly1gp4jto8c8fj309203qjrg.jpg)

然而，将这些表面展示的文本简单的联接：

```
ex:a ex:p _:x .
ex:b ex:q _:x .
```

描述的图包含三个节点

![union2](https://tvax1.sinaimg.cn/large/005SZbikly1gp4jtzcng8j309203fmx7.jpg)

因为在公共标识符范围中两次出现的空节点标识符"\_:x"标识了相同的空白节点。这两个图的四个节点的联合可以用这种形式更恰当的描述：

```
ex:a ex:p _:x1 .
ex:b ex:q _:x2 .
```

其中，空节点标识符已经<u>标准化</u>，以避免混淆不同的空节点。使用特定的空节点标识符没有意义，因为他们是不同的。

<u>两个或更多的图共享一个空节点是可能的，比如如果他们是一个单一较大图的子图或者从同一个来源派生。</u>在这种情况下，一组图的联合保留了图之间共享空节点的标识。通常，无论他们是否共享空节点，图集的联合可以准确的表示与图本身相同的语义内容。

一个相关的运算，称为merging，是在共享多个图中出现并且都不同的空节点后，得到联合。结果图称为**merge**。一个图的子图merge可能比原图大。比如，合并两个三节点图的单一子图。（note:空节点经过重新命名）

![merge1](https://tva2.sinaimg.cn/large/005SZbikly1gp4ju7w9tkj309203fmx7.jpg)

是一个四节点图

![merge2](https://tva2.sinaimg.cn/large/005SZbikly1gp4juh5a7jj309203qjrg.jpg)

联合永远是merge的实例(两个节点已经映射到同一个空节点)。如果图没有共同的空节点，则他们的union和merge是相同的。

## 5. 简单的解释

本章定义RDF图解释的基本概念。RDF编码中的任何词汇表或更高级别表示法的所有语义扩展都必须符合最小的条件。其他语义扩展可以扩展并**添加**到这些扩展中，但不能修改或者否定他们。比如，因为简单的解释是适用于IRI的映射，所以一个语义扩展不能以不同的方式解释单个IRI不同的情况。

全部语义应用于RDF**图**，而不是RDF源。一个RDF源仅仅在给定时间或者给定状态下的图中才有意义。图不能随时间改变语义。

一个简单的解释 **I** 的结构：

**定义**

1. 一个资源的非空集合**IR**，称为**I**的定义域或全集
2. 一个集合**IP**，称为**I**的属性集合
3. 从**IP**到**IR**的幂集的映射**IEXT**，即，IR中关于x和y的<x,y>的集合
4. 从**IRIs**到**IR**与**IP**并集的映射，**IS**
5. 从文字到**IR**的部分映射**IL**

Note

```
简单的解释需要解释所有name，因此是无限的。这简化了说明。但是，可以使用已确定算法的有限结构来解释RDF。
```

**IEXT(x)**称为x的扩展，是用于标识属性为true的参数的集合，也就是说，是一个二值型的关系扩展。

当定义了数据类型的语义时，**IR**和**IL**之间的区别是很重要的。因为一些文字没有指向，所以**IL**可以是一部分。

Note

```
通常将关系名直接映射到关系扩展。但是这样是假定了词汇表被分为关系名和个体名，RDF没有这样的假设。此外，RDF允许将IRI用作关系名，作为自身的一个参数使用。RDFs使用了这种自我应用的结构。？IEXT映射用于从满足两个要求的作为宾语的关系与其关系扩展区别开。它还提供了RDFs"类"的概念，可以将其与集合理论扩展区分开。
```

在**I** 中 ground RDF图的表示，可以看作是从表达式（names, triples and graphs）到全集和真值元素的函数。

ground 图的语义条件

```
如果E是文字，则I(E) = IL(E)
如果E是IRI，则I(E) = IS(E)
如果E是ground三元组s p o.则当I(p)属于IP且<I(s),I(o)>属于IEXT(I(p))时，I(E) = true,否则为false
如果E是一个ground RDF图则当一些三元组E'属于E，且I(E')=false，则I(E)=false;否则I(E) = true
```

如果某些文字E没有定义IL(E)，则E没有语义值，所以任何包含它的三元组都为false，任何包含这个三元组的图也都为false。

最后的条件显示空图（空三元组的集合）永远为true

集合IP和IR可能会有重叠，IP确实可以是IR的子集。因为IEXT的定义域条件，任何为true的三元组的主语和宾语都会在IR内；所以在一个图中出现的任何IRI都作为谓语和主语或宾语会表示为IP和IR的交集内的事物。

语义扩展通过要求某些IRI以特定的方式引用，可能会对解释映射施加进一步的约束。例如，以下所述的D解释要求某些IRI（具有识别和引用数据类型的功能）具有固定的符号。

### 5.1 空节点

空节点被视为简单指示事物的存在，无需使用IRI来识别任何特定事物。这与空节点表示未知IRI是不同的。

假设 **I** 是一个简单的解释，<u>而A是从一组空节点到IR的映射</u>。定义names上的 **I** 为映射[I+A]，当x是一个name时，`[I+A](x)=I(x)`；当x是一个空节点时，`[I+A](x)=A(x)`；可以用以上规则扩展这个映射到三元组和RDF图。RDF图中空节点的语义条件为：

```
如果E是RDF图，且对于一些从E中的空节点集合到IR的映射A,[I+A](E)=true,则I(E)=true,否则I(E)=false
```

从空节点到引用对象的映射不是简单解释的一部分，因为真实条件仅涉及某些此类映射。空节点本身与其他节点的不同之处在于，没有简单的解释为其分配符号，直观的反映为它们没有全局的含义。

#### 5.1.1 共享空节点

空节点的语义是根据图的真实性来陈述的。然而，当多个图共享一个空节点时，并不能通过单独处理的方式获取他们的语义。例如，这两个重叠的图

​	                            ![shareBlankNode](https://tvax1.sinaimg.cn/large/005SZbikly1gp4jv2v9eaj30cq05l74i.jpg)

一个简单的解释**I** `{Alice, Bob, Monica, Ruth}`

```
I(ex:Alice)=Alice, I(ex:Bob)=Bob, IEXT(I(ex:hasChild))={<Alice,Monica>,<Bob,Ruth> }
```

在这个解释下每个图都是true，但是放在一起就是不对的，因为三个节点的图表明Alice和Bob一起有一个孩子。为了得到共享空节点的图的含义，有必要考虑所有包含空节点的三元组的联合。

Note

```
RDF图可以看作是一阶逻辑中简单原子语句的结合，其中空节点是自由变量，被认为是存在的。然后，将两个图进行联合就类似于此语法中的语法合取。RDF语法没有显式的变量绑定量词，因此任何RDF图的真实条件都将该图中的自由变量视为该图中存在的量化值。选取共享空节点的图的联合会更改隐含的量词范围。
```

### 5.2 简单蕴含

标准的术语是，当I(E) = true时，我们说 I 满足E，当满足它一个简单的解释存在时，称E是可满足的，否则为不可满足的，**并且当满足G的解释也满足E时，称为图G蕴含图E。**如果两个图E和F互相蕴含，则称他们是逻辑上等效的。

Note

```
我们没有定义图集之间的蕴含的概念。为了确定一个图集是否蕴含一个图，集合中的图首先应该采用union或merge的方式结合成一个图。联合共享空节点的通用含义，而merge忽略了共享空节点。
```

**如果在任何情况下，S都简单蕴含E，则任何从其他图S构建图E的过程都是有效的，否则为无效的。**

推理有效的事实不应该被理解为以为这任何RDF应用程序都有义务或被要求做出推理。同样，某些RDF转换或过程的逻辑无效并不意味着该过程是不正确的或被禁止的。本规范中的任何内容均不要求或禁止在RDF图形或源上进行任何特定的操作。必然性和有效性仅与确定此类操作的条件有关，这些条件保证了真理。虽然不禁止逻辑上无效的过程，这些过程不会遵循有效的要求，用户应注意，他们可能会冒充将错误信息引入真实的RDF数据的风险。然而，在可以通过其他方式确保真相的情况下，逻辑上无效的过程的特殊使用可能是合理的，并且适合于数据处理。

蕴含仅是指RDF图的真实性，而不是指它们对任何其他目的的适用性。RDF图有可能适合于给定的目的，而有效地蕴含另一个不适用于同一目的的图。一个RDF测试样例清单，为方便用户，将其作为RDF文档提供。本文档通过描述其前提和结论列出了正确蕴含的示例。作为RDF图，清单简单蕴含一个子图，省略了前提，因此如果用作测试用例清单，则将是不正确的。（子图没有前提 ） 这并不违反RDF语义规则，但是它表明“成为正确的RDF测试用例清单”的属性在RDF蕴含下并未保留，因此不能描述为RDF语义扩展。RDF的这种蕴含风险的用法应仅限于以下情况：在这种情况下，对于所有各方来说，明确的预期的特定限制是什么，与使用RDF在Web上公开发布数据的更普通情况相反。

### 5.3 简单蕴含的属性

这里描述的属性仅适用于简单蕴含，而不是蕴含的扩展概念。

```
每个图都是可满足的
```

这并不总是适用于解释的扩展概念。例如，一个包含错误类型文字的图是D不满足的。

**interpolation lemma**

```
当且仅当G的子图是E的一个实例时，G简单蕴含图E             
//E中的空节点经过映射可以得到G的子图。（满足G的都满足E）
```

<u>要检测一个RDF图是否简单蕴含另一个图，检查一个图是否存在一些实例，这个实例是第一个图的子集。</u>

Note

```
任何图都蕴含空图，但是空图只蕴含它本身。
一个图蕴含它的所有子图。
一个图的所有实例蕴含图的本身。
如果E是lean图，并且E'是E的一个proper实例，则E不蕴含E'
如果S是S'的一个子图，并且S蕴含E，则S'蕴含E
如果S蕴含有限图E，则S的一些有限子集S'蕴含E
如果E包含一个在S中没有出现过的IRI，则S不蕴含E
```

## 6. Skolemization

Skolemization是一个RDF图的转换，它通过将空节点替换为“新” IRI来消除空节点，这意味着为此目的而创造IRI，因此可以保证在任何其他RDF图中都不会出现（创建时）。

假设G是一个包含空节点的图，而<u>sk</u>是从G中的空节点到替代它们的skolem IRIs的skolenization<u>映射</u>，所以sk(G)是G的一个skolemization。他们之间的语义关系可以总结如下：

```
sk(G)蕴含G（因为sk(G)是G的一个实例）
G不蕴含sk(G)（因为sk(G)包含G中没有的IRIs）
对于任何图H，如果sk(G)蕴含H，则存在一个图H'，使得G蕴含H'并且H=sk(H') 		//sk(G)蕴含sk(H')
对于任何图H，不包含sk(G)引入的任何新IRI，当且仅当G蕴含H时，sk(G)蕴含H
```

第二个属性表示图在逻辑上不等效于它的skolemization。但是，从强烈的意义上讲，它们几乎可以互换，如以下两个属性所示。第三个特性意味着，即使从skolemized图中得出的结论确实包含新词汇，这些将精确地反映在原位置为空节点的情况下从原图获得的结果。用IRIs替换空节点并不能有效地改变从图中导出的内容，除了给以前的匿名实体提供新的名称之外。第四个属性是第三个属性的结果，它清楚地表明，从某种意义上说，就蕴含所涉及的问题而言，G的skolemized可以“代表”G。使用sk（G）代替G不会影响任何不涉及新skolem词汇的条件。

## 7. 文字与数据类型

数据类型由IRIs定义。根据哪些IRIs被识别为表示数据类型，解释会有所不同。我们在简单的解释中使用参数D对此进行描述，<u>其中D是公认的数据类型IRIs的集合。</u>

IRI识别数据类型的确切机制被认为是语义外部的，但是语义假定已识别的IRI标识唯一的数据类型，无论它在哪里出现。无法确定IRI标识哪种数据类型的RDF处理器无法识别该IRI，因此应将具有该IRI作为其数据类型IRI的所有文字视为未知名称。

<u>总结：RDF文字将字符串和标识数据类型的IRI结合在一起。</u>带语言标记的字符串是一个例外，它具有两个语法成分：字符串和语言标记，并且被分配为`rdf：langString`类型。数据类型应理解为定义从词法空间（一组字符串）到值的部分映射，称为词法到值映射。<u>函数**L2V**将数据类型映射到它们的词法到值映射</u>。数据类型为d的文字表示是通过将此映射应用于字符串sss获得的值：L2V(d)(sss)。如果文字字符串不在词法空间中，那么从词法到值的映射就不会为文字字符串提供任何值，那么该文字就没有引用对象。<u>数据类型的值空间是词法到值映射的范围。</u>每个具有该类型的文字要么引用该类型的值空间中的一个值，要么根本无法引用。<u>类型错误的文字是可以识别其数据类型IRI，但其字符串未由该数据类型的词汇到值的映射分配任何值。</u>

RDF处理器不需要识别`rdf：langString`和`xsd：string`以外的任何数据类型IRI，但是如果识别出[RDF11-CONCEPTS]第5节中列出的IRI，它们必须按照此处所述进行解释，并且在识别IRI `rdf：PlainLiteral`时，必须将其解释为引用[RDF-PLAIN-LITERAL]中定义的数据类型。RDF处理器可能认识到其他的数据类型的IRI，但是当其它数据类型的IRI被识别，数据类型和数据类型IRI之间的映射是指必须被明确地指定，并且在所有RDF转换或操作期间必须固定。实际上，这可以通过将IRI链接到数据类型的外部规范来实现，该规范描述了数据类型本身的组成部分以及IRI标识数据类型的事实，从而固定了该IRI的数据类型映射的值。

以`rdf：langString`作为其数据类型的文字是一种例外情况，将对其进行特殊处理。IRI`rdf：langString`被分类为数据类型IRI，并且即使未为其定义L2V映射，也将其解释为引用该数据类型。`rdf：langString`<u>的值空间是带有语言标签的字符串的所有对的集合。</u>下面给出了以其为类型的文字的语义。

RDF文字语法允许任何IRI都可以在类型文字中使用，即使未将其识别为引用数据类型也是如此。尽管RDF应用程序可能会发出警告，但具有此类“未知”数据类型IRI（不在公认的数据类型集合中）的文字不应视为错误。此类文字应像IRI一样对待，并假定表示全集**IR**中的某些事物。无法识别数据类型IRI的RDF处理器将无法检测到某些隐含的东西。例如：

```
ex:a ex:p "20.0000"^^xsd:decimal .
```

蕴含

```
ex:a ex:p "20.0"^^xsd:decimal .
```

对于无法识别数据类型IRI` xsd：decimal`的处理器将不可见。

### 7.1 D-interpretations

令D为识别数据类型的IRI的集合。一个D解释是一个满足以下条件的简单解释：

数据类型文字的语义条件：

```
如果rdf:langString属于D，则对带有词汇格式sss和语言标签ttt的所有语言标签字符串E，有IL(E)=<sss,ttt'>
(ttt'是ttt转换的小写)。
对于D中其他的IRI aaa,I(aaa)是由aaa定义的数据类型，并且对于所有文字"sss"^^aaa，有IL("sss"^^aaa) = L2V(I(aaa))(sss)
```

如果文字类型错误，则L2V(I(aaa))映射将没有值，因此文字无法表示任何内容。在这种情况下，任何包含文字的三元组都必须为false。因此，**任何包含错误类型文字的三元组（进而是任何图）都将是 D不可满足的**。<u>这仅适用于在D中使用识别数据类型IRI的文字类型</u>；具有不可识别的IRI类型的文字不是错误类型，并且不能产生D-unsatisfiable图。

特殊数据类型`rdf：langString`没有错误类型的文字。具有这种类型的任何语法合法的文字将在每个 D-interpretation中表示一个值，其中D包括`rdf：langString`。类型为`xsd：string`唯一的类型错误的文字是那些包含与[XML10]中的Char产生不匹配的Unicode代码点的文字。

### 7.2 数据类型蕴含

当一个图在一些D-interpretation中有真值时，则他是 D-satisfiable或者是satisfiable recognizing D；并且当每个D-interpretation满足S也D-满足 G时，一个图S <u>D蕴含</u> 或 entails recognizing D 图G。

与简单解释的情况不同，图可能没有可满足的 D-interpretations。RDF处理器可以将无法满足的图为发出错误状态的信号，但这不是必需的。

<u>一个D-unsatisfiable图D蕴含任何图。</u>

在所有这些语言中，“ D”都用作表示一组<u>数据类型IRIs</u>的参数，并且不同的D集将产生可满足性和必要性的不同概念。识别出的数据类型越多，含义就越强，所以如果D ⊂ E 并且S E蕴含 G则S一定D蕴含 G。当D是一个空集时，如果S D蕴含 G则S蕴含G。

#### 7.2.1 数据类型蕴含模式

与简单蕴含不同，不可能给出一个语法标准来检测所有D蕴含，由于所识别的数据类型的词法-值映射关系的特定属性，因此可以保留该值。例如如果D包含`xsd:decimal`则

```
ex:a ex:p "25.0"^^xsd:decimal .
```

D蕴含

```
ex:a ex:p "25"^^xsd:decimal .
```

通常，当文字的词法字符串映射到数据类型的“字法对值”映射下的相同值时，任何包含具有识别的数据类型IRI 的三元组的文字D蕴含另一个文字。<u>如果D中的两个不同数据类型将词法字符串映射到一个公共值，那么包含一个用一种数据类型的文字类型的三元组可能会D-蕴含另一个包含用不同数据类型的文字类型的三元组。</u>例如：

`"25"^^xsd:integer`  和`"25.0"^^xsd:decimal`具有相同的值，所以当D也包含`xsd:integer`时， 以上也D-entails`ex:a ex:p "25"^^xsd:integer .`

**错误类型文字是唯一可以使图D不满足的方法，**但是当数据类型与稍后定义的RDFS词汇表结合使用时，可能会导致其他各种不满足的图表。