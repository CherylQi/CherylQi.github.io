---
title: SPARQL 1.1 笔记(四)
date: 2021-03-21 16:28:44
tags: 语义Web
---

## 18 SPARQL 的定义

给定查询字符串和RDF数据集，本节定义了用于评估图形模式和解决方案修饰符的正确行为。这并不意味着SPARQL实现必须使用此处定义的过程。

执行SPARQL查询的结果由一系列步骤定义，从SPARQL查询作为字符串开始，将该字符串转换为抽象语法形式，然后将抽象语法转换为包含SPARQL代数运算符的SPARQL抽象查询。然后，在RDF数据集上评估此抽象查询。

### 18.1 初始定义

#### 18.1.1 RDF Terms

SPARQL是根据IRIs 定义的。IRIs是RDF URI引用的子集，它忽略了空格的使用。

```
Definition: RDF Term
I是所有IRIs的集合.
RDF-L是所有RDF Literals的集合
RDF-B是RDF图中所有空节点的集合
RDF Terms的集合, RDF-T, 是I ∪ RDF-L ∪ RDF-B.
```

**RDF Term**此定义从RDF数据模型收集了几个基本概念 ，但已更新为引用IRI而不是RDF URI引用。

<!--more-->

#### 18.1.2 简单文字

```
简单的文字是所有没有语言标签或数据类型IRI的RDF文字集合
```

#### 18.1.3 RDF数据集

```
一个RDF数据集是这样的集合：
{ G, (<u1>, G1), (<u2>, G2), . . . (<un>, Gn) }				//一个默认图和多个命名图
其中，G和Gi是图，每一个<ui>是一个IRI。每一个<ui>是不同的。
G是默认图。(<ui>, Gi) 是命名图。
```

```
活动图：是数据集中用于与基本图模式匹配的图
```

```
RDF数据集合并
有DS1 = { G1, (<u11>, G11), (<u12>, G12), . . . (<u1n>, G1n) },
DS2 = { G2, (<u21>, G21), (<u22>, G22), . . . (<u2m>, G2m) }
然后定义关于DS1和DS2的RDF数据集的合并为：
DS={ G, (<u1>, G1), (<u2>, G2), . . . (<uk>, Gk) }
其中：
N1是{ <u1j> j = 1 to n }，N2是{ <u2j> j = 1 to m }

G是G1和G2的合并
(<ui>, Gi) 是<ui>属于N1中但不属于N2
(<ui>, Gi) 是<ui>属于N2中但不属于N1
(<ui>, Gi) 是<ui>与N1中的<uj>相同，并且与N2中的<uk>相同。并且Gi是G1j和G2k的合并。
```

#### 18.1.4 查询变量

```
一个查询变量是一个集合V中的成员，V来自RDF-T，是无限且不相交的。
```

#### 18.1.5 三元组模式

```
一个三元组模式是这个集合的成员：(RDF-T ∪ V) x (I ∪ V) x (RDF-T ∪ V)
```

三元组模式的定义包含文字主语，由于RDF图可能不包含文字主语，因此任何以文字为主语的SPARQL三元组模式都无法在任何RDF图上匹配。

#### 18.1.6 基本图模式

```
一个基本图模式是一个三元组模式集合
```

空图模式是一个空集的三元组模式

#### 18.1.7 属性路径模式

```
一个属性路径是一个三元组序列，ti属于序列ST，n = length(ST)-1，i从0到n，ti的宾语是与ti+1的主语相同的term。
t0的主语是路径的起点
tn的宾语是路径的终点
如果每个ti都是G的一个三元组，则一个属性路径是图G中的一条路径
```

一条属性路径不会跨越数据集中的多个图

```
属性路径表达式
一个属性路径表达式是用上述格式中的属性路径的表达式
```

```
属性路径模式
PP是所有路径表达式的集合。一个属性路径模式是该集合的成员：
(RDF-T ∪ V) x PP x (RDF-T ∪ V)
```

一条属性路径是一个三元组模式的概括，包括了属性位置的属性路径表达式。

#### 18.1.8 解决方案映射

一个解决方案是一个从变量集到一个RDF Terms集的映射。

```
解决方案映射
一个解决方案映射，μ, 是一个局部函数μ : V -> RDF-T.
μ的定义域, dom(μ), 是在定义μ后V的子集。
```

```
解决方案序列
一个解决方案序列是一个解决方案列表，可能是无序的。
```

使用由μ给定的变量，将expr（μ）写入表达式expr的值。评估可能会导致错误。

#### 18.1.9 解决方案序列修饰符

```
- Order修饰符：按顺序排列解决方案
- Projection修饰符：选择某些变量
- Distinct修饰符：确保序列中的解决方案是唯一的
- Reduce修饰语：允许消除一些不唯一的解决方案
- Offset修改器：控制解决方案从整体解决方案序列中的位置开始
- Limit修饰符：限制解决方案数量
```

#### 18.1.10 SPARQL 查询

```
一个SPARQL抽象查询是一个元组（E,DS,QF）,其中：
E 是一个SPARQL代数表达式
DS 是一个RDF Dataset数据集
QF 是一个查询形式
```

```
查询级别
一个查询级别是一个图模式，一组组和聚合以及一组解决方案修饰符。
```

<u>查询是一棵“查询级别”的树，其中每个子查询在树中形成一个查询级别。</u>

### 18.2 转化为SPARQL代数

本部分定义了将SPARQL查询字符串中的图形模式和解决方案修饰符转换为SPARQL代数表达式的过程。所描述的过程转换了嵌套查询的一级，该嵌套由子查询使用嵌套的`SELECT`语法形成， 并递归应用于子查询。每个级别都包括图形模式匹配和过滤，然后应用解决方案修饰符。

解析SPARQL查询字符串，并应用第4节中给出的IRI和三元组模式的缩写 。此时，抽象语法树由以下内容组成：

| Patterns                 | Modifiers          | Query Forms | Other   |
| ------------------------ | ------------------ | ----------- | ------- |
| RDF terms                | DISTINCT           | SELECT      | VALUES  |
| Property path expression | REDUCED            | CONSTRUCT   | SERVICE |
| Property path patterns   | Projection         | DESCRIBE    |         |
| Groups                   | ORDER BY           | ASK         |         |
| OPTIONAL                 | LIMIT              |             |         |
| UNION                    | OFFSET             |             |         |
| GRAPH                    | Select expressions |             |         |
| BIND                     |                    |             |         |
| GROUP BY                 |                    |             |         |
| HAVING                   |                    |             |         |
| MINUS                    |                    |             |         |
| FILTER                   |                    |             |         |

转换此类抽象语法树的结果是一个SPARQL查询，该查询在SPARQL代数中使用以下符号：

| Graph Pattern | Solution Modifiers | Property Path      |
| ------------- | ------------------ | ------------------ |
| BGP           | ToList             | PredicatePath      |
| Join          | OrderBy            | InversePath        |
| LeftJoin      | Project            | SequencePath       |
| Filter        | Distinct           | AlernativePath     |
| Union         | Reduced            | ZeroOrMorePath     |
| Graph         | Slice              | OneOrMorePath      |
| Extend        | ToMultiSet         | ZeroOrOnePath      |
| Minus         |                    | NegatedPropertySet |
| Group         |                    |                    |
| Aggregation   |                    |                    |
| AggregateJoin |                    |                    |

*切片*是OFFSET和LIMIT的组合。

*ToList*用于发生从图形模式匹配结果到序列的转换的情况。

*ToMultiSet*用于发生从解决方案序列到多集的转换的情况。

#### 18.2.1 变量作用域

在执行查询的SPARQL代数时，如果有一种方法可以使变量位于解决方案映射的定义域中，则可以将其定义为*作用域内*的变量。下面的定义提供了一种从查询的抽象语法确定这一点的方法。

注意，<u>带有投影的子查询可以隐藏变量</u>。在`FILTER`或`MINUS`中的变量不会导致变量超出这些形式的范围。

令**P**，**P1**，**P2**为图形模式，让**E**，**E1**，... **En**为表达式。在以下情况下，变量`v`在作用域内：

| Syntax Form                 | In-scope variables                         |
| --------------------------- | ------------------------------------------ |
| Basic Graph Pattern (BGP)   | `v` occurs in the BGP                      |
| Path                        | `v` occurs in the path                     |
| Group `{ P1 P2 ... }`       | `v` is in-scope 在一个多个图中 P1, P2, ... |
| `GRAPH term { P }`          | `v` is `term` or `v` is in-scope in P      |
| `{ P1 } UNION { P2 }`       | `v` is in-scope in P1 or in-scope in P2    |
| `OPTIONAL {P}`              | `v` is in-scope in P                       |
| `SERVICE term {P}`          | `v` is `term` or `v` is in-scope in P      |
| `BIND (expr AS v)`          | `v` is in-scope                            |
| `SELECT .. v .. { P }`      | `v` is in-scope                            |
| `SELECT ... (expr AS v)`    | `v` is in-scope                            |
| `GROUP BY (expr AS v)`      | `v` is in-scope                            |
| `SELECT * { P }`            | `v` is in-scope in `P`                     |
| `VALUES v { values }`       | `v` is in-scope                            |
| `VALUES varlist { values }` | `v` is in-scope if `v` is in `varlist`     |

变量`v`一定不能在`(expr AS v)` 形式的作用域内。`(expr AS v)`的作用域适用于 `SELECT`表达式。

`BIND (expr AS v)`要求变量`v`不在使用它的组图模式中的先前元素范围内。

在`SELECT`中，该变量`v`不在该`SELECT`子句的图形模式作用域内，也不得在该子句前面的另一个select表达式中使用。

#### 18.2.2 转换图形模式

本节描述了将SPARQL图形模式转换为SPARQL**代数表达式**的过程。此过程将应用于形成`WHERE`查询子句的组图模式（定界符`{...}`之间的单元），并递归应用于组图模式内的每个语法元素。转换的结果是SPARQL代数表达式。

概括而言，这些步骤的应用如下：

- 扩展 IRI，文字和三元组模式的语法形式。
- 转换属性路径表达式
- 将一些属性路径模式转换为三元组
- 将`FILTER`s收集到组中
- 翻译基本图形模式
- 翻译组中其余的图形模式
- 添加过滤器
- 简化代数表达

translate(graph pattern)此处描述的算法可转换图模式

##### 18.2.2.1 扩展语法形式

扩展第4节中给出的IRI和三元组模式的缩写 。

##### 18.2.2.2 收集`FILTER`元素

`FILTER`表达式适用于它们出现的**整个**组图模式。在转换每个组元素后，将用于执行过滤的代数运算符添加到该组。我们在这里将过滤器收集在一起，并将其从组中删除，然后将其应用于整个转换后的组图模式。

在此步骤中，我们还将转换含有`FILTER` 表达式`EXISTS`和`NOT EXISTS`的图形模式。

```
Let FS := empty set

For each form FILTER(expr) in the group graph pattern:
    In expr, replace NOT EXISTS{P} with fn:not(exists(translate(P))) 
    In expr, replace EXISTS{P} with exists(translate(P))
    FS := FS ∪ {expr}
    End
```

##### 18.2.2.3 转换属性路径表达式

下表提供了从SPARQL语法到SPARQL代数中的术语的属性路径表达式的转换。这将递归地应用于属性路径表达式的所有元素。

此步骤之后的下一步将某些形式转换为三元组模式，然后通过邻接（无中间组模式定界符`{`和`}`） 或其他语法形式将其转换为基本图形模式。总体而言，仅IRI的SPARQL语法属性路径变为三元组模式，并将它们聚合为基本图形模式。

笔记：

- 否定的属性集中形式IRI和^ IRI的顺序不相关。

我们引入以下符号：

- link
- inv
- alt
- seq
- ZeroOrMorePath
- OneOrMorePath
- ZeroOrOnePath
- NPS (for NegatedPropertySet)

| Syntax Form (path)                       | Algebra (path)                                               |
| ---------------------------------------- | ------------------------------------------------------------ |
| `iri`                                    | `link(iri)`                                                  |
| `^path`                                  | `inv(path)`                                                  |
| `!(:iri1|...|:irin)`                     | `NPS({:iri1 ... :irin})`                                     |
| `!(^:iri1|...|^:irin)`                   | `inv(NPS({:iri1 ... :irin}))`                                |
| `!(:iri1|...|:irii|^:irii+1|...|^:irim)` | `alt(NPS({:iri1 ...:irii}),  inv(NPS({:irii+1, ..., :irim})) )` |
| `path1 / path2`                          | `seq(path1, path2)`                                          |
| `path1 | path2`                          | `alt(path1, path2)`                                          |
| `path*`                                  | `ZeroOrMorePath(path)`                                       |
| `path+`                                  | `OneOrMorePath(path)`                                        |
| `path?`                                  | `ZeroOrOnePath(path)`                                        |

##### 18.2.2.4 转换属性路径模式

上一步转换了属性路径表达式。此步骤将属性路径模式转换为三元组模式，或将其作为主语端点，属性路径表达式和宾语端点转换为通用代数运算，以进行路径评估。

笔记：

- X and Y are RDF terms or variables.
- ?V is a fresh variable.
- P and Q are path expressions.
- These are only applied to property path patterns, not within property path expressions.
- Translations earlier in the table are applied in preference to the last translation.
- The final translation simply wraps any remaining property path expression to use a common form `Path(...)`.

| Algebra (path)  | Translation       |
| --------------- | ----------------- |
| `X link(iri) Y` | `X iri Y`         |
| `X inv(iri) Y`  | `Y iri X`         |
| `X seq(P, Q) Y` | `X P ?V . ?V Q P` |
| `X P Y`         | `Path(X, P, Y)`   |

整个路径转换过程的示例（`?_V`是一个新变量）：

```
?s :p/:q ?o

?s :p ?_V .
?_V :q ?o

?s :p* ?o

Path(?s, ZeroOrMorePath(link(:p)), ?o)

:list rdf:rest*/rdf:first ?member

Path(:list, ZeroOrMorePath(link(rdf:rest)), ?_V) .
?_V rdf:first ?member
```

##### 18.2.2.5 转换基本图形模式

在转换属性路径后，将所有相邻的三元组模式收集在一起以形成基本图形模式`BGP(triples)`。

##### 18.2.2.6 转换图形模式

接下来，我们转换每一个剩余的图模式，递归的应用转换过程

如果形式为是GroupOrUnionGraphPattern

```
Let A := undefined
          
For each element G in the GroupOrUnionGraphPattern
    If A is undefined
        A := Translate(G)
    Else
        A := Union(A, Translate(G))
    End

The result is A
```

如果形式为GraphGraphPattern

```
If the form is GRAPH IRI GroupGraphPattern
    The result is Graph(IRI, Translate(GroupGraphPattern))
```

```
If the form is GRAPH Var GroupGraphPattern
    The result is Graph(Var, Translate(GroupGraphPattern))
```

如果形式为GroupGraphPattern

```
Let FS := the empty set
Let G := the empty pattern, a basic graph pattern which is the empty set.

For each element E in the GroupGraphPattern

    If E is of the form OPTIONAL{P} 
        Let A := Translate(P)
        If A is of the form Filter(F, A2)
            G := LeftJoin(G, A2, F)
        Else 
            G := LeftJoin(G, A, true)
            End
        End

    If E is of the form MINUS{P}
        G := Minus(G, Translate(P))
        End

    If E is of the form BIND(expr AS var)
        G := Extend(G, var, expr)
        End

    If E is any other form 
        Let A := Translate(E)
        G := Join(G, A)
        End

   End
   
The result is G.
            
```

如果形式为InlineData

```
The result is a multiset of solution mappings 'data'.
```

通过从变量列表（或单个变量）中相应位置的变量形成一个解决方案映射来形成*数据*，如果`BindingValue`是单词`UNDEF`，则省略绑定。

如果形式是SubSelect

##### 18.2.2.7 组过滤器

在转换组之后，将添加过滤器表达式，以便将它们应用于该组的其余所有部分：

```
If FS is not empty
    Let G := output of preceding step
    Let X := Conjunction of expressions in FS
    G := Filter(X, G)
    End
```

##### 18.2.2.8 简化步骤

一个图形模式的一些组变为`join(Z, A)`，其中Z是空的基本图形模式（这是空集）。可以用A代替它们。空图模式Z是连接的标识：

```
Replace join(Z, A) by A
Replace join(A, Z) by A
```

#### 18.2.3 映射图模式的示例

重写示例的第二种形式是第一种形式，其中空组联接已通过简化步骤删除。

示例：由单个三元模式组成的基本图形模式的组：

```
{ ?s ?p ?o }

Join(Z, BGP(?s ?p ?o) )
BGP(?s ?p ?o)
```

示例：由两个三元组模式组成的基本图形模式的组：

```
{ ?s :p1 ?v1 ; :p2 ?v2 }

BGP( ?s :p1 ?v1 . ?s :p2 ?v2 )
```

示例：由两个基本图形模式的并集组成的组：

```
{ { ?s :p1 ?v1 } UNION {?s :p2 ?v2 } }

Union(Join(Z, BGP(?s :p1 ?v1)),
      Join(Z, BGP(?s :p2 ?v2)) )
      
Union( BGP(?s :p1 ?v1) , BGP(?s :p2 ?v2) )
```

示例：由联合和基本图形模式的联合组成的组：

```
{ { ?s :p1 ?v1 } UNION {?s :p2 ?v2 } UNION {?s :p3 ?v3 } }

Union(
    Union( Join(Z, BGP(?s :p1 ?v1)),
           Join(Z, BGP(?s :p2 ?v2))) ,
    	   Join(Z, BGP(?s :p3 ?v3)) )
    
Union(
    Union( BGP(?s :p1 ?v1) ,
           BGP(?s :p2 ?v2),
    	   BGP(?s :p3 ?v3))
```

示例：由基本图形模式和可选图形模式组成的组：

```
{ ?s :p1 ?v1 OPTIONAL {?s :p2 ?v2 } }

LeftJoin(
    Join(Z, BGP(?s :p1 ?v1)),
    Join(Z, BGP(?s :p2 ?v2)),true)
    
LeftJoin(BGP(?s :p1 ?v1), BGP(?s :p2 ?v2), true)
```

示例：由基本图形模式和两个可选图形模式组成的组：

```
{ ?s :p1 ?v1 OPTIONAL {?s :p2 ?v2 } OPTIONAL { ?s :p3 ?v3 } }

LeftJoin(
    LeftJoin(
        BGP(?s :p1 ?v1),
        BGP(?s :p2 ?v2),
        true) ,
    BGP(?s :p3 ?v3),
    true)
```

示例：由基本图形模式和带有过滤器的可选图形模式组成的组：

```
{ ?s :p1 ?v1 OPTIONAL {?s :p2 ?v2 FILTER(?v1<3) } }

LeftJoin(
     Join(Z, BGP(?s :p1 ?v1)),
     Join(Z, BGP(?s :p2 ?v2)),
     (?v1<3) )
     
LeftJoin(
    BGP(?s :p1 ?v1) ,
    BGP(?s :p2 ?v2) ,
   (?v1<3) )
```

示例：由联合图模式和可选图模式组成的组：

```
{ {?s :p1 ?v1} UNION {?s :p2 ?v2} OPTIONAL {?s :p3 ?v3} }

LeftJoin(
  Union(BGP(?s :p1 ?v1),
        BGP(?s :p2 ?v2)) ,
  BGP(?s :p3 ?v3) ,
  true )
```

示例：由基本图形模式，过滤器和可选图形模式组成的组：

```
{ ?s :p1 ?v1 FILTER (?v1 < 3 ) OPTIONAL {?s :p2 ?v2} }

Filter( ?v1 < 3 ,
  LeftJoin( BGP(?s :p1 ?v1), BGP(?s :p2 ?v2), true) ,
  )
```

示例：涉及BIND的模式：

```
{ ?s :p ?v . BIND (2*?v AS ?v2) ?s :p1 ?v2 }

Join(
   Extend( BGP(?s :p ?v), ?v2, 2*?v) ,
   BGP(?s :p1 ?v2) )
```

示例：涉及BIND的模式：

```
{ ?s :p ?v . {} BIND (2*?v AS ?v2) }

Join(
   BGP(?s :p ?v), ?v2, 2*?v) ,
   Extend({}, ?v2, 2*?v)
)
```

示例：涉及MINUS的模式：

```
{ ?s :p ?v . MINUS {?s :p1 ?v2 } }

Minus(
   BGP(?s :p ?v)
   BGP(?s :p1 ?v2))
```

示例：涉及子查询的模式：

```
{ ?s :p ?o . {SELECT DISTINCT ?o {?o ?p ?z} } }

Join(
   BGP(?s :p ?o) ,
   ToMultiSet(
     Distinct(Project(BGP(?o ?p ?z), {?o})) )
   )
```

#### 18.2.4 转换组，集合，HAVING，最终VALUES子句和SELECT表达式

在此步骤中，我们按以下顺序在查询级别处理子句：

- Grouping
- Aggregates
- HAVING
- VALUES
- Select expressions

##### 18.2.4.1 分组和聚合

步骤：GROUP BY

如果使用了`GROUP BY`关键字，或者由于在投影中使用了聚合，则存在隐式分组，则分组由Group函数执行。它将解决方案集分为一个或多个解决方案的组，且总体基数相同。在隐式分组的情况下，使用固定常数（1）将所有解决方案分组为一个组。

步骤：Aggregates

聚合步骤被应用为查询级别上的转换，用Aggregation（）代数表达式替换了查询级别中的聚合表达式。

下面给出了使用任何聚合的查询级别的转换：

```
Let A := the empty sequence
Let Q := the query level being evaluated
Let P := the algebra translation of the GroupGraphPattern of the query level
Let E := [], a list of pairs of the form (variable, expression)

If Q contains GROUP BY exprlist
   Let G := Group(exprlist, P)
Else If Q contains an aggregate in SELECT, HAVING, ORDER BY
   Let G := Group((1), P)
Else
   skip the rest of the aggregate step
   End

Global i := 1   # Initially 1 for each query processed

For each (X AS Var) in SELECT, each HAVING(X), and each ORDER BY X in Q
  For each unaggregated variable V in X
      Replace V with Sample(V)
      End
  For each aggregate R(args ; scalarvals) now in X
      # note scalarvals may be omitted, then it's equivalent to the empty set
      Ai := Aggregation(args, R, scalarvals, G)
      Replace R(...) with aggi in Q
      i := i + 1
      End
  End

For each variable V appearing outside of an aggregate
   Ai := Aggregation(V, Sample, {}, G)
   E := E append (V, aggi)
   i := i + 1
   End

A := Ai, ..., Ai-1
P := AggregateJoin(A)
```

注意：agg i是一个临时变量。然后在18.2.4.4中将E用于选择表达式的处理。

**示例**

```
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
SELECT (SUM(?val) AS ?sum) (COUNT(?a) AS ?count)
WHERE {
  ?a rdf:value ?val .
} GROUP BY ?a
```

SUM表达式变为agg 1，而COUNT表达式变为agg 2。

```
Let G := Group((?a), BGP(?a rdf:value ?val))
A1 = Aggregation((?val), Sum, {}, G)
A2 = Aggregation((?a), Count, {}, G)
A := (A1, A2)
Let P := AggregateJoin(A)
```

##### 18.2.4.2 HAVING

使用与FILTER（）相同的规则评估HAVING表达式。请注意，由于在评估HAVING子句的逻辑位置，因此SELECT子句投影的表达式对HAVING子句不可见。

```
Let Q := the query level being evaluated
Let P := the algebra translation of the query level so far

For each HAVING(E) in Q
    P := Filter(E, P)
    End
```

##### 18.2.4.3 VALUES

如果查询具有结尾的VALUES子句：

```
Let P := the algebra translation of the query level so far
P := Join(P, ToMultiSet(data))
  where data is a solution sequence formed from the VALUES clause
```

数据的转换与内联数据的转换相同。

##### 18.2.4.4 SELECT表达式

Step: Select expressions

我们有两种形式的抽象语法要考虑：

```
SELECT selItem ... { pattern }
SELECT * { pattern }

Let X := algebra from earlier steps
Let VS := list of all variables visible in the pattern,
           so restricted by sub-SELECT projected variables and GROUP BY variables.
           Not visible: only in filter, exists/not exists, masked by a subselect, 
                        non-projected GROUP variables, only in the right hand side of MINUS

Let PV := {}, a set of variable names
Note, E is a list of pairs of the form (variable, expression), defined in section 18.2.4
  
If "SELECT *"
    PV := VS

If  "SELECT selItem ...:"  
    For each selItem:
        If selItem is a variable
            PV := PV ∪ { variable }
        End
        If selItem is (expr AS variable)
            variable must not appear in VS nor in PV; if it does then generate a syntax error and stop
            PV := PV ∪ { variable }
            E := E append (variable, expr) 
        End
    End

For each pair (var, expr) in E
    X := Extend(X, var, expr)
    End
  
Result is X  
The set PV is used later for projection.
            
```

当将变量用作SELECT的WHERE子句内部或已在此SELECT表达式中用作AS的变量时，将变量用作AS的命名目标（例如... AS？x）会出现语法错误。

#### 18.2.5 转换解决方案修饰符

解决方案修饰符适用于模式匹配后对SPARQL查询的处理。解决方案修饰符按以下顺序应用于查询：

- Order by
- Projection
- Distinct
- Reduced
- Offset
- Limit

步骤：ToList

ToList将一个多集转换为具有相同元素和基数的序列。该序列没有隐含的顺序；重复项不必相邻。

```
Let M := ToList(Pattern)
```

##### 18.2.5.1 ORDER BY

如果查询字符串具有ORDER BY子句

```
M := OrderBy(M, list of order comparators)
```

##### 18.2.5.2 Projection

`PV`在SELECT表达式的处理中计算了一组投影变量。

```
M := Project(M, PV)
```

其中vars是SELECT子句中提到的变量集，或者 如果使用SELECT *，则是查询范围内的所有命名变量。

##### 18.2.5.3 DISTINCT

如果查询包含DISTINCT

```
M := Distinct(M)
```

##### 18.2.5.4 REDUCED

如果查询包含REDUCED

```
M := Reduced(M)
```

##### 18.2.5.5 OFFSET and LIMIT

如果查询包含"OFFSET start" or "LIMIT length"

```
M := Slice(M, start, length)

start defaults to 0

length defaults to (size(M)-start).
```

##### 18.2.5.6 Final Algebra Expression

总体抽象查询为M。

### 18.3基本图形模式

当匹配图形模式时，可能的解决方案形成一个*多集* [multiset ]，也称为*bag*。多集是元素的无序集合，其中每个元素可能出现多次。它由一组元素和基数函数描述，该基数函数给出了多集中该集合中每个元素的出现次数。

解决方案映射写作μ。

dom(μ0)是一个空集，映射为μ0。

由确切的空映射μ0组成的多集Ω0，基数为1。This is the join identity.

映射RDF term到`t : { (x, t) }`的解决方案为μ(x)。

由 μ(?x->t)组成的多集Ω(x)，{ { (x, t) } } 基数为1。

```
兼容映射
对属于dom(μ1)和dom(μ2)的每个v，两个解决方案映射μ1和μ2是兼容的，有μ1(v) = μ2(v).
```

这里, μ1(v) = μ2(v)意味着μ1(v)和μ2(v)是相同的RDF term.

如果μ1和μ2是兼容的，则μ1∪μ2也是一个映射。μ1 ∪ μ2写作merge(μ1, μ2) 

 在一个映射的多集Ω中，解决方案μ的基数为card\[Ω](μ)。

#### 18.3.1 SPARQL 基本图模式匹配

对于该部分查询，基本图形模式与活动图形相匹配。基本图形模式可以通过用术语替换变量和空白节点来实例化，给出实例的两个概念。使用从空白节点到RDF术语的RDF实例映射，σ替换空白节点；通过从查询变量到RDF术语的解决方案映射，变量将被替换。

```
模式实例映射
一个模式实例映射，P，是一个RDF实例映射，σ，和解决方案映射μ的组合。P(x) = μ(σ(x))
```

对于BGP 'x'，P（x）表示替换x中的空白节点b的结果，其中定义σ（b），x中的所有变量v定义为μ（v）。

任何模式实例映射都定义了唯一的解决方案映射，并通过将其分别限制为查询变量和空白节点，而获得的唯一的RDF实例映射。

```
基本图模式匹配
BGP是一个基本图模式并且G是一个RDF图.

当存在模式实例映射P，μ是G的BGP解决方案，使得P(BGP)是G的一个子图，μ是P对BGP中查询变量的限制

card[Ω](μ) = card[Ω](不同的RDF实例映射数量, σ, 使得P = μ(σ)是一个模式实例映射并且P(BGP)是G的一个子图).
```

如果一个基本图形模式是空集，则该解决方案是Ω 0。

#### 18.3.2 空白节点的处理

此定义允许解决方案映射将基本图形模式BGP中的变量绑定到G中的空白节点。由于SPARQL处理结果格式文档中的空白节点标识符（SPARQL查询结果XML格式， SPARQL 1.1查询结果JSON格式和 SPARQL 1.1查询结果CSV和TSV格式）仅限于文档范围，因此不能理解为标识数据集活动图中的节点。如果DS是查询的数据集，则模式解决方案因此应理解为不是来自DS本身的活动图，而是来自称为*作用域图*的RDF*图，*它与DS的活动图等效，但是与DS或BGP不共享空白节点。同一范围图用于单个查询的所有解决方案。范围图纯粹是一种理论构造；实际上，仅通过空白节点标识符的文档作用域约定即可获得效果。

由于RDF空白节点可以为许多模式提供无限多个冗余解决方案，因此可以有无限多个模式解决方案（通过用不同的空白节点替换空白节点来获得）。因此，有必要以某种方式界定基本图形模式的解决方案。SPARQL使用子图匹配标准来确定基本图模式的解决方案。对于从基本图形模式到活动图形的子集的每个不同的模式实例映射，都有一个解决方案。

为简化计算而不是消除冗余进行了优化。即使数据集的活动图是lean，它也允许查询结果包含冗余 ，并且它允许逻辑等效的数据集产生不同的查询结果。

### 18.4 属性路径模式

本节定义属性路径模式的评估 。属性路径模式是主语端点（RDF术语或变量），属性路径表达和宾语端点。属性路径表达式的转换某些形式到其他SPARQL表达式，将长度为1的属性路径转换为三元组模式，而这又被组合成基本图模式。这样就留下了属性路径运算符ZeroOrOnePath，ZeroOrMorePath，OneOrMorePath和NegatedPropertySets，以及这些运算符中包含的路径表达式。

所有其余的属性路径表达式都以端点X和Y的形式`Path(X, path, Y)`存在于代数中 。例如：语法`(:p/:q)*`是ZeroOrMorePath表达式，其中涉及成为代数表达式`ZeroOrMorePath(seq(link(:p), link(:q)))`的序列属性路径。

```
eval(Path(X, PP, Y))
用于评估属性路径模式。这产生了解决方案μ映射的多集，每个解决方案映射都具有所使用变量的绑定（X和Y可以是变量）。一些操作符仅生成一组解决方案映射。

对于在x1, x2, ..., xn中的变量
Var(x1, x2, ..., xn) = { xi | i in 1...n and xi is a variable }
```

| `x:term` | when `x` is an RDF term       |
| -------- | ----------------------------- |
| `x:var`  | when `x` is a variable        |
| `x:path` | when `x` is a path expression |

通过在整个查询评估中匹配活动图来执行所有评估。为了清楚起见，我们在每个定义中都明确省略了活动图形。

```
谓语属性路径评估
使用一些IRI iri，Path(X, link(iri), Y)作为谓语逆属性路径
eval(Path(X, link(iri), Y)) = evaluation of basic graph pattern {X iri Y}
```

如果X和Y都是变量，等同于：

```
eval(Path(X:var, link(iri), Y:var)) = 
    { (X, xn) (Y, yn) | xn and yn are RDF terms and triple (xn iri yn) is in the active graph }
```

如果X是变量，Y是一个RDF term：

```
eval(Path(X:var, link(iri), Y:term)) = 
    { (X, xn) | xn is an RDF term and triple (xn iri Y) is in the active graph }
```

如果X是一个RDF term，Y是一个变量：

```
eval(Path(X:term, link(iri), Y:var)) =
    { (Y, yn) | yn is an RDF term and triple (X iri yn) is in the active graph }
```

如果X和Y都是RDF terms:

```
eval(Path(X:term, link(iri), Y:term)) = 
    { μ0 } if triple (X iri Y) is in the active graph
    = { { } }
    = Ω0

eval(Path(X:term, link(iri), Y:term)) = 
    { } if triple (X iri Y) is not in the active graph
```

非正式地，评估谓词属性路径与在查询评估中此时执行子查询相同 。 `SELECT * { X P Y }`

```
评估逆属性路径
P是一条属性路径，则：
eval(Path(X, inv(P), Y)) = eval(Path(Y, P, X))
```

```
评估序列属性路径
P和Q是属性路径表达式。V是一个新变量：
A = Join( eval(Path(X, P, V)), eval(Path(V, Q, Y)) )
eval(Path(X, seq(P,Q), Y)) = Project(A, Var(X,Y))
```

等同于：SELECT * { X P _:a . _:a Q Y }

使用空白节点的`_:a`行为类似于变量（在简单情况下），但不会出现在`SELECT *`的结果中。

```
评估可选属性路径
P和Q是属性路径表达式。
eval(Path(X, alt(P,Q), Y)) = Union(eval(Path(X, P, Y)), eval(Path(X, Q, Y)))
```

等同于：SELECT * { { X P Y } UNION { X Q Y } }

```
图的节点集
图G的节点集，nodes(G)：
nodes(G) = { n | n是一个RDF Term，用于三元组G的主语和宾语}
```

```
评估零或一路径
eval(Path(X:term, ZeroOrOnePath(P), Y:var)) = { (Y, yn) | yn = X or {(Y, yn)} in eval(Path(X,P,Y)) }
eval(Path(X:var, ZeroOrOnePath(P), Y:term)) = { (X, xn) | xn = Y or {(X, xn)} in eval(Path(X,P,Y)) }
eval(Path(X:term, ZeroOrOnePath(P), Y:term)) = 
    { {} } if X = Y or eval(Path(X,P,Y)) is not empty
    { } othewise
eval(Path(X:var, ZeroOrOnePath(P), Y:var)) = 
    { (X, xn) (Y, yn) | either (yn in nodes(G) and xn = yn) or {(X,xn), (Y,yn)} in eval(Path(X,P,Y)) }
```

我们定义了一个辅助函数ALP，该函数用于ZeroOrMorePath和OneOrMorePath的定义中。请注意，此处给出的算法用于指定功能。一个实现可以自由地通过任何为整个查询产生相同结果的方法来实现评估。ZeroOrMorePath和OneOrMorePath表单根据路径连接的不同节点返回匹配项。

匹配算法基于所有路径，并检测何时在该路径上访问了图形节点（主语或宾语）。

非正式地，此算法尝试通过在每个步骤中应用`路径`来扩展结果的多集 ，并指出它已针对该特定路径访问了哪些节点。如果针对所考虑的路径已访问了某个节点，则该节点不适合进行其他步骤。

```
ALP函数
Let eval(x:term, path) be the evaluation of 'path', starting at RDF term x, 
                       and returning a multiset of RDF terms reached 
                       by repeated matches of path.

ALP(x:term, path) = 
    Let V = empty multiset
    ALP(x:term, path, V)
    return is V

# V is the set of nodes visited

ALP(x:term, path, V:set of RDF terms) =
    if ( x in V ) return 
    add x to V
    X = eval(x,path) 
    For n:term in X
        ALP(n, path, V)
        End
```

```
评估零或多路径
eval(Path(X:term, ZeroOrMorePath(path), vy:var)) =
    { { (vy, n) } | n in ALP(X, path) }

eval(Path(vx:var, ZeroOrMorePath(path), vy:var)) =
    { { (vx, t), (vy, n) } |  t in nodes(G), (vy, n) in eval(Path(t, ZeroOrMorePath(path), vy)) }

eval(Path(vx:var, ZeroOrMorePath(path), y:term)) = 
    eval(Path(y:term, ZeroOrMorePath(inv(path)), vx:var))

eval(Path(x:term, ZeroOrMorePath(path), y:term)) = 
    { { } } if { (vy:var,y) } in eval(Path(x, ZeroOrMorePath(path) vy)
    { } otherwise
```

```
评估一或多路径
eval(Path(X, OneOrMorePath(path), Y))

# For OneOrMorePath, we take one step of the path then start
# recording nodes for results.

eval(Path(x:term, OneOrMorePath(path), vy:var)) =
    Let X = eval(x, path)
    Let V = the empty multiset
    For n in X
        ALP(n, path, V)
        End
    result is V

eval(Path(vx:var, OneOrMorePath(path), vy:var)) =
   { { (vx, t), (vy, n) } |  t in nodes(G), (vy, n) in eval(Path(t, OneOrMorePath(path), vy)) }

eval(Path(vx:var, OneOrMorePath(path), y:term)) =
   eval(Path(y:term, OneOrMorePath(inv(path)), vx))

eval(Path(x:term, OneOrMorePath(path), y:term)) =
    { { } } if { (vy:var, y) } in eval(Path(x, OneOrMorePath(path), vy))
    { } otherwise
	
```

```
评估否定属性集
Write μ' as the extension of a solution mapping:
μ'(μ,x) = μ(x)   if x is a variable
μ'(μ,t) = t   if t is a RDF term
	
Let x and y be variables or RDF terms, and S a set of IRIs:

eval(Path(x, NPS(S), y)) = { μ | ∃ triple(μ'(μ,x), p, μ'(μ,y)) in G, such that the IRI of p ∉ S }
```

### 18.5 SPARQL代数式

对于SPARQL抽象查询中的每个剩余符号，我们定义一个用于求值的运算符。相同名称的SPARQL代数运算符用于评估SPARQL抽象查询节点，如“评估语义”一节中所述。上面已经描述了基本图形模式和属性路径模式的评估。

```
Filter
Let Ω be a multiset of solution mappings and expr be an expression. We define:

Filter(expr, Ω, D(G)) = { μ | μ in Ω and expr(μ) is an expression that has an effective boolean value of true }

card[Filter(expr, Ω, D(G))](μ) = card[Ω](μ)

Note that evaluating an exists(pattern) expression uses the dataset and active graph, D(G). See the evaluation of filter.
```

```
Join
Let Ω1 and Ω2 be multisets of solution mappings. We define:

Join(Ω1, Ω2) = { merge(μ1, μ2) | μ1 in Ω1and μ2 in Ω2, and μ1 and μ2 are compatible }

card[Join(Ω1, Ω2)](μ) =
    for each merge(μ1, μ2), μ1 in Ω1and μ2 in Ω2 such that μ = merge(μ1, μ2),
        sum over (μ1, μ2), card[Ω1](μ1)*card[Ω2](μ2)
```

Join中的解决方案映射μ可能出现在不同的解决方案映射中，多个集合中的μ1和μ2被连接。 μ的基数是所有可能性的基数之和。

```
Diff
Let Ω1 and Ω2 be multisets of solution mappings and expr be an expression. We define:

Diff(Ω1, Ω2, expr) = { μ | μ in Ω1 such that ∀ μ′ in Ω2, either μ and μ′ are not compatible or μ and μ' are compatible and expr(merge(μ, μ')) has an effective boolean value of false }

card[Diff(Ω1, Ω2, expr)](μ) = card[Ω1](μ)
```

Diff在内部用于LeftJoin的定义。

```
LeftJoin

Let Ω1 and Ω2 be multisets of solution mappings and expr be an expression. We define:

LeftJoin(Ω1, Ω2, expr) = Filter(expr, Join(Ω1, Ω2)) ∪ Diff(Ω1, Ω2, expr)

card[LeftJoin(Ω1, Ω2, expr)](μ) = card[Filter(expr, Join(Ω1, Ω2))](μ) + card[Diff(Ω1, Ω2, expr)](μ)

展开写作：
LeftJoin(Ω1, Ω2, expr) =
    { merge(μ1, μ2) | μ1 in Ω1 and μ2 in Ω2, μ1 and μ2 are compatible and expr(merge(μ1, μ2)) is true }
∪
    { μ1 | μ1 in Ω1, ∀ μ2 in Ω2, μ1 and μ2 are not compatible, or Ω2 is empty }
∪
    { μ1 | μ1 in Ω1, ∃ μ2 in Ω2, μ1 and μ2 are compatible and expr(merge(μ1, μ2)) is false. }

```

由于这些是<u>不同</u>的，因此LeftJoin的基数就是定义的这些各个组成部分的基数。

```
Union

Let Ω1 and Ω2 be multisets of solution mappings. We define:

Union(Ω1, Ω2) = { μ | μ in Ω1 or μ in Ω2 }

card[Union(Ω1, Ω2)](μ) = card[Ω1](μ) + card[Ω2](μ)
```

```
Minus

Let Ω1 and Ω2 be multisets of solution mappings. We define:

Minus(Ω1, Ω2) = { μ | μ in Ω1 . ∀ μ' in Ω2, either μ and μ' are not compatible or dom(μ) and dom(μ') are disjoint }

card[Minus(Ω1, Ω2)](μ) = card[Ω1](μ)
```

添加了对dom(μ)和dom(μ')的附加限制，因为如果Ω2中的解决方案映射没有与Ω1的解决方案映射相同的变量，那么Minus（Ω1，Ω2）将为空，不论Ω2的其余部分如何。空解决方案映射与所有其他解决方案映射兼容，因此对于任何模式P，P MINUS {}都将为空。

```
Extend
Let μ be a solution mapping, Ω a multiset of solution mappings, var a variable and expr be an expression, then we define:

Extend(μ, var, expr) = μ ∪ { (var,value) | var not in dom(μ) and value = expr(μ) }

Extend(μ, var, expr) = μ if var not in dom(μ) and expr(μ) is an error

Extend is undefined when var in dom(μ).

Extend(Ω, var, expr) = { Extend(μ, var, expr) | μ in Ω }
```

 [ x | C ] ，对于一个元素序列，C为x上的条件。

card\[L](x)为x在L中的基数。

```
ToList
Let Ω be a multiset of solution mappings. We define:

ToList(Ω) = a sequence of mappings μ in Ω in any order, with card[Ω](μ) occurrences of μ

card[ToList(Ω)](μ) = card[Ω](μ)
```

```
OrderBy
Let Ψ be a sequence of solution mappings. We define:

OrderBy(Ψ, condition) = [ μ | μ in Ψ and the sequence satisfies the ordering condition]

card[OrderBy(Ψ, condition)](μ) = card[Ψ](μ)
```

```
Project
Let Ψ be a sequence of solution mappings and PV a set of variables.

For mapping μ, write Proj(μ, PV) to be the restriction of μ to variables in PV.

Project(Ψ, PV) = [ Proj(Ψ[μ], PV) | μ in Ψ ]

card[Project(Ψ, PV)](μ) = card[Ψ](μ)

The order of Project(Ψ, PV) must preserve any ordering given by OrderBy.
```

```
Distinct
Let Ψ be a sequence of solution mappings. We define:

Distinct(Ψ) = [ μ | μ in Ψ ]

card[Distinct(Ψ)](μ) = 1

The order of Distinct(Ψ) must preserve any ordering given by OrderBy.
```

```
Reduced
Let Ψ be a sequence of solution mappings. We define:

Reduced(Ψ) = [ μ | μ in Ψ ]

card[Reduced(Ψ)](μ) is between 1 and card[Ψ](μ)

The order of Reduced(Ψ) must preserve any ordering given by OrderBy.
```

Reduced解序列修改器不保证定义的基数。

```
Slice
Let Ψ be a sequence of solution mappings. We define:

Slice(Ψ, start, length)[i] = Ψ[start+i] for i = 0 to (length-1)
```

```
ToMultiSet
Let Ψ be a solution sequence. We define:

ToMultiSet(Ψ) = { μ | μ in Ψ }

card[ToMultiSet(Ψ)](μ) = card[Ψ](μ)
```

ListEval是一个函数，用于根据解决方案评估表达式列表并返回结果值列表。

```
ToMultiset

ToMultiset将序列转换为具有与该序列相同的元素和基数的多集。序列的顺序对所得的多集没有影响，并且重复项被保留。
```

```
Exists

exists(pattern)是一个函数，如果在评估时给出当前解映射和活动图，则如果模式评估为非空解序列，则返回true；否则返回false。
```

#### 18.5.1 聚合代数式

Group是根据解决方案的某些属性将解决方案序列分为多个解决方案的函数。

```
Group

Group 根据解决方案序列评估表达式列表，产生一组从键到解决方案序列的部分函数。

Group(exprlist, Ω) = { ListEval(exprlist, μ) → { μ' | μ' in Ω, ListEval(exprlist, μ) = ListEval(exprlist, μ') } | μ in Ω }
```

```
ListEval

ListEval((expr1, ..., exprn), μ) returns a list (e1, ..., en), where ei = expri(μ) or error.

ListEval 保留由于评估列表元素而导致的错误。
```

请注意，尽管ListEval的结果可能是错误，并且可以使用错误进行分组，但是在投影时会删除包含错误值的解决方案。

ListEval((unbound), μ) = (error)，因为对未绑定表达式的求值是错误的。

Aggregation，一个计算标量值作为聚合表达式输出的函数。在SELECT子句，HAVING评估过程和ORDER BY（需要时）中使用它。聚合使用集合函数计算解决方案组上的聚合值。

```
Aggregation

Let exprlist be a list of expressions or *, func a set function, scalarvals a set of partial functions (possibly empty) passed from the aggregate in the query, and let { key1→Ω1, ..., keym→Ωm } be a multiset of partial functions from keys to solution sequences as produced by the grouping step.

Aggregation 将集合函数func应用于给定的多集，并为每个键和该键的解决方案分区生成一个值。

Aggregation(exprlist, func, scalarvals, { key1→Ω1, ..., keym→Ωm } )
   = { (key, F(Ω)) | key → Ω in { key1→Ω1, ..., keym→Ωm } }

where
  M(Ω) = { ListEval(exprlist, μ) | μ in Ω }
  F(Ω) = func(M(Ω), scalarvals), for non-DISTINCT
  F(Ω) = func(Distinct(M(Ω)), scalarvals), for DISTINCT
特殊情况：将COUNT与表达式*一起使用时，如果存在DISTINCT关键字，则F的值将是组解序列，card [Ω]或card [Distinct(Ω)]的基数。
```

*标*量用于绕开分组机制将值传递给基础set函数。例如，聚合表达式`GROUP_CONCAT(?x ; separator="|")`的标量参数为{ "separator" → "|" }。

所有聚合都可以在其参数列表中将关键字`DISTINCT` 作为第一个标记。如果存在此关键字，则func的第一个参数是Distinct（M）。

**示例**给出一个带有下列值的解决方案多集(Ω) ：

| solution | ?x   | ?y   | ?z   |
| -------- | ---- | ---- | ---- |
| μ1       | 1    | 2    | 3    |
| μ2       | 1    | 3    | 4    |
| μ3       | 2    | 5    | 6    |

查询表达式为：

```
 SELECT (ex:agg(?y, ?z) AS ?agg) 
 WHERE { ?x ?y ?z 
 } GROUP BY ?x.
```

 G = Group((?x), Ω) = { ( (1), { μ1, μ2 } ), ( (2), { μ3 } ) }

所以Aggregation((?y, ?z), ex:agg, {}, G) = { ((1), eg:agg({(2, 3), (3, 4)}, {})), ((2), eg:agg({(5, 6)}, {})) }.

```
AggregateJoin

Let S1, ..., Sn be a list of sets, where each set Si contains key to (aggregated) value maps as produced by Aggregate.

Let K = { key | key in dom(Sj) for some 1 <= j <= n } be the set of keys, then
AggregateJoin(S1, ..., Sn) = { agg1→val1, ..., aggn→valn | key in K and key→vali in Si for each 1 <= i <= n }
```

Flatten是用于将列表的多集合折叠为多集合的功能，因此，例如{((1，2), (3，4)}}变为{1, 2, 3, 4}。

```
Flatten

The Flatten(M) function takes a multiset of lists, M {(L1, L2, ...), ...}, and returns the multiset { x | L in M and x in L }.
```

##### 18.5.1.1 Set Functions

作为SPARQL聚合基础的集合函数都具有一个公共签名：SetFunc(M)或SetFunc(M，scalarvals)，其中M是列表的多集，而scalarvals是一个或多个标量值，这些值通过间接传递给set函数。 SPARQL语法中聚合的(...; key = value)语法。SPARQL Query 1.1中的内置聚合支持的唯一用法是`GROUP_CONCAT`，如中所述`GROUP_CONCAT(?x ; separator=", ")`。

请注意，名称“ Set Function”在某种程度上是历史性的-设置函数的参数实际上是多集。由于与SQL Set函数具有通用性，因此保留了该名称，SQL Set Function也可以在多集合上运行。

本文档中定义的一组功能是计数，和，最小值，最大值，平均值，GroupConcat和样品-对应于所述聚集体`COUNT`，`SUM`，`MIN`，`MAX`，`AVG`，`GROUP_CONCAT`，和`SAMPLE`。在以下各节中可以找到定义。系统可能会选择使用本地扩展来扩展此集合，并使用与函数和强制类型转换相同的符号。注意，除非使用分隔符; 时，这要求解析器在确定使用聚合的查询中是否存在任何错误之前，必须先了解某个IRI是否引用了函数，强制转换或聚合。

##### 18.5.1.2 Count

Count是一个SPARQL内置函数，用于对给定表达式在聚合组中具有绑定值和非错误值的次数进行计数。

```
Count

xsd:integer Count(multiset M)
N = Flatten(M)

remove error elements from N

Count(M) = card[N]
```

##### 18.5.1.3 Sum

Sum是一个SPARQL内置函数，它将返回通过对聚合组中的值求和而获得的数值。类型提升按照op：numeric-add函数的方式进行，可传递地应用（请参见下面的定义），因此SUM(?x)的值在一个聚合组中，其中?x的值为1（整数），2.0e0（浮点） ，而3.0（十进制）将为6.0（浮点数）。

```
Sum

numeric Sum(multiset M)
The Sum set function is used by the SUM aggregate in the syntax.

Sum(M) = Sum(ToList(Flatten(M))).

Sum(S) = op:numeric-add(S1, Sum(S2..n)) when card[S] > 1
Sum(S) = op:numeric-add(S1, 0) when card[S] = 1
Sum(S) = "0"^^xsd:integer when card[S] = 0

In this way, Sum({1, 2, 3}) = op:numeric-add(1, op:numeric-add(2, op:numeric-add(3, 0))).
```

##### 18.5.1.4 Avg

Avg内置函数可计算整个组中某个表达式的平均值。它是根据总和和计数来定义的。

```
Avg

numeric Avg(multiset M)
Avg(M) = "0"^^xsd:integer, where Count(M) = 0

Avg(M) = Sum(M) / Count(M), where Count(M) > 0
```

Avg({1, 2, 3}) = Sum({1, 2, 3})/Count({1, 2, 3}) = 6/3 = 2.

##### 18.5.1.5 Min

Min是一个SPARQL内置函数，分别从组中返回最小值。

它利用SPARQL ORDER BY排序定义，以允许对任意类型的表达式进行排序。

```
Min

term Min(multiset M)
Min(M) = Min(ToList(Flatten(M)))

Min({}) = error.

The flattened multiset of values passed as an argument is converted to a sequence S, this sequence is ordered as per the ORDER BY ASC clause.

Min(S) = S0
```

##### 18.5.1.6 Max

Max是一个SPARQL内置函数，分别从组中返回最大值。

它利用SPARQL ORDER BY排序定义，以允许对任意类型的表达式进行排序。

```
Max

term Max(multiset M)
Max(M) = Max(ToList(Flatten(M)))

Max({}) = error.

The multiset of values passed as an argument is converted to a sequence S, this sequence is ordered as per the ORDER BY DESC clause.

Max(S) = S0
```

##### 18.5.1.7 GroupConcat

GroupConcat是一个内置函数，该函数对具有组的表达式的值执行字符串连接。不指定字符串的顺序。串联中使用的分隔符可以使用标量参数SEPARATOR给出。

```
GroupConcat

literal GroupConcat(multiset M)
If the "separator" scalar argument is absent from GROUP_CONCAT then it is taken to be the "space" character, unicode codepoint U+0020.

The multiset of values, M passed as an argument is converted to a sequence S.

GroupConcat(M, scalarvals) = GroupConcat(Flatten(M), scalarvals("separator"))

GroupConcat(S, sep) = "", where |S| = 0

GroupConcat(S, sep) = CONCAT("", S0), where |S| = 1

GroupConcat(S, sep) = CONCAT(S0, sep, GroupConcat(S1..n-1, sep)), where |S| > 1
```

GroupConcat({"a", "b", "c"}, {"separator" → "."}) = "a.b.c".

##### 18.5.1.8 Sample

Sample是一个集合函数，它从传递给它的多重集中返回一个任意值。

```
Sample

RDFTerm Sample(multiset M)
Sample(M) = v, where v in Flatten(M)

Sample({}) = error
```

给出Sample({"a", "b", "c"}), "a", "b"和 "c"都是有效返回值。请注意，对于给定的输入，不要求Sample()是确定性的，唯一的限制是输出值必须存在于输入多集中。

### 18.6 评估语义

eval(D(G), algebra expression)定义为对具有活动图G的数据集D的代数表达式的评估。活动图最初是默认图。

```
D : a dataset
D(G) : D a dataset with active graph G (the one patterns match against)
D[i] : The graph with IRI i in dataset D
P, P1, P2 : graph patterns
L : a solution sequence
F : an expression
```

```
基本图模式的评估
eval(D(G), BGP) = multiset of solution mappings
```

```
属性路径模式的评估
eval(D(G), Path(X, path, Y)) = multiset of solution mappings
```

```
Filter的评估
eval(D(G), Filter(F, P)) = Filter(F, eval(D(G),P), D(G))
```

"替换"是一个过滤函数，用于支持对已转换为`exits`的EXISTS和NOT EXISTS表单进行评估。

```
Substitute

Let μ be a solution mapping.

substitute(pattern, μ) = the pattern formed by replacing every occurrence of a variable v in pattern by μ(v) for each v in dom(μ)
```

```
Exists评估

Let μ be the current solution mapping for a filter and P a graph pattern:

The value exists(P), given D(G) is true if and only if eval(D(G), substitute(P, μ)) is a non-empty sequence.
```

```
Join评估
eval(D(G), Join(P1, P2)) = Join(eval(D(G), P1), eval(D(G), P2))
```

```
LeftJoin评估
eval(D(G), LeftJoin(P1, P2, F)) = LeftJoin(eval(D(G), P1), eval(D(G), P2), F)
```

```
Union评估
eval(D(G), Union(P1,P2)) = Union(eval(D(G), P1), eval(D(G), P2))
```

```
Graph评估
if IRI is a graph name in D
eval(D(G), Graph(IRI,P)) = eval(D(D[IRI]), P)
if IRI is not a graph name in D
eval(D(G), Graph(IRI,P)) = the empty multiset
eval(D(G), Graph(var,P)) =
     Let R be the empty multiset
     foreach IRI i in D
        R := Union(R, Join( eval(D(D[i]), P) , Ω(?var->i) )
     the result is R
```

图的评估使用SPARQL代数联合运算符。解决方案映射的基数是每个联接操作中该解决方案映射的基数的总和。

```
Definition: Evaluation of Group

eval(D(G), Group(exprlist, P)) = Group(exprlist, eval(D(G), P))

Definition: Evaluation of Aggregation

eval(D(G), Aggregation(exprlist, func, scalarvals, P)) = Aggregation(exprlist, func, scalarvals, eval(D(G), P))

Definition: Evaluation of AggregateJoin

eval(D(G), AggregateJoin(A1, ..., An)) = AggregateJoin(eval(D(G), A1), ..., eval(D(G), An))
```

如果eval(D(G), Ai)是一个错误，则忽略

```
Definition: Evaluation of Extend
eval(D(G), Extend(P, var, expr)) = Extend(eval(D(G), P), var, expr)

Definition: Evaluation of ToList
eval(D(G), ToList(P)) = ToList(eval(D(G), P))

Definition: Evaluation of Distinct
eval(D(G), Distinct(L)) = Distinct(eval(D(G), L))
          
Definition: Evaluation of Reduced
eval(D(G), Reduced(L)) = Reduced(eval(D(G), L))
          
Definition: Evaluation of Project
eval(D(G), Project(L, vars)) = Project(eval(D(G), L), vars)
          
Definition: Evaluation of OrderBy
eval(D(G), OrderBy(L, condition)) = OrderBy(eval(D(G), L), condition)
          
Definition: Evaluation of ToMultiSet
eval(D(G), ToMultiSet(L)) = ToMultiSet(eval(D), M))

Definition: Evaluation of Slice
eval(D(G), Slice(L, start, length)) = Slice(eval(D(G), L), start, length)
```

### 18.7 扩展SPARQL基本图匹配

通过重写基本图形模式的匹配条件，可以将整体SPARQL设计用于查询，该查询采用比简单蕴含更复杂的蕴含形式。由于以单一通用形式陈述这些条件是一个开放的研究问题，该条件适用于所有形式的蕴含，并以最佳方式消除了不必要或不适当的冗余，因此本文档仅给出了任何此类解决方案都应满足的必要条件。这些将需要扩展到每个特定案例的完整定义。

基本图形模式与三元组模式的关系相同，而RDF图形与RDF三元组模式的关系相同，并且许多相同的术语都可以应用于它们。特别地，如果在三元组模式的术语之间存在双映射M ，则两个基本图形模式被认为是*等效*的，三元组模式将空白节点映射到空白节点，并将变量，文字和IRI映射到它们自己，从而使三元组（s，p当且仅当三元组（M（s），M（p），M（o））在第二个模式中时，才在第一个模式中。该定义通过在等效模式之间保留变量名称，将RDF图形等效性扩展到基本图形模式。

一个蕴含规则指定

1. RDF图的一个子集，被称为**well-formed**的规则
2. *well-formed*图的子集与*well-formed*图的子集之间的*蕴含*关系。

一些蕴含机制可以将某些RDF图分类为不一致。例如，RDF图：

```
_：x rdf：type xsd：string.
_：x rdf：type xsd：decimal.
```

当D包含XSD数据类型时，是D不一致的。本规范未涵盖查询对不一致图的影响，但必须由特定的SPARQL扩展名指定。

蕴含规则E必须提供基本图模式评估的条件，以便对于任何基本图模式BGP，任何RDF图G和任何满足条件的评估，唯一确定的解决方案多元集取决于RDF图的等效性。我们使用Eval-E（G，BGP）在E上评估G上的BGP的解决方案表示多集。
蕴含必须进一步满足以下条件：

1. 对任何E不一致的活动图AG，蕴含规则E唯一地指定了与AG等效的作用域图SG。

2. 指定了一组E的*well-formed*的图，以便对于Eval-E（SG，BGP）中的任何基本图模式BGP，作用域图SG和解决方案映射μ，对于E的图μ（BGP）是*well-formed*。 

3. 对于任何基本图形图案BGP和范围界定图SG，如果μ 1，...，μ Ñ在EVAL-E（SG，BGP）和BGP 1，...，BGP Ñ是基本图形的图案的所有等效于BGP但不彼此或与SG共享任何空白节点，则

   ​		SG E-entails (SG union μ1(BGP1) union ... union μn(BGPn))

这些条件不能完全确定可能的答案，因为RDF允许无限数量的冗余。因此，此外，必须满足以下条件。

4. 蕴含规则应提供适当的条件，以防止琐碎的无限解决方案多集适用于该制度。

#### 18.7.1 注意事项

（a）SG通常在图形上与AG等效，但是将其限制为E-等效性允许在查询之前将某些形式的规范化（例如，消除语义冗余）应用于源文档。

（b）条件3中的构造确保解决方案映射引入的任何空白节点的使用方式与SG中空白节点的出现方式内部一致。这确保仅当如此标识的空白节点在SG中确实相同时，空白节点标识符才会出现在答案集中的多个答案中。如果扩展名不允许绑定到空白节点，则可以将此条件简化为以下条件：

​			SG E-entails μ(BGP) for each solution mapping μ.

（c）这些条件不强加SPARQL要求SG不与AG或BGP共享空白节点。特别是，它允许SG实际上是AG。这允许使用以下查询协议，其中空白节点标识符在查询和源文档之间或在多个查询之间保留其含义。但是，当前的SPARQL协议规范不支持此类协议。

（d）由于条件1至3仅是答案的必要条件，因此条件4允许以各种方式限制合法答案的情况。

（e）这些条件均未明确涉及BGP中空白节点上的实例映射。对于某些蕴含机制，单个实例映射的存在无法完全捕获空白节点的存在性解释。这些条件允许此类机制为查询模式中的空白节点提供“完全存在”的读数。

显而易见，对于SG为E的情况，SPARQL满足这些条件，因为SG上的SPARQL条件是它与AG在图上等效，但不与AG或BGP共享空白节点（满足第一个条件） ）。唯一不平凡的条件是（3）。对于每个解映射μi，根据基本图形模式匹配的定义，存在映射σi的RDF实例，使得PI(BgPi)是SG的子图，其中Pi是由μi和σi组成的模式实例映射。由于bgPi和SG没有共同的空白节点，所以σi和μi的范围不包含来自bgpi的空节点；因此，解映射μi和pi的σ实例映射rdf i通勤，因此PI(BgPi)=σi(μi(bgpi)。所以

```
P1(BGP1) union ... union Pn(BGPn)
= σ1(μ1(BGP1)) union ... union σn(μn(BGPn))
= [ σ1 + ... + σn]( μ1(BGP1) union ... union μn(BGPn) )

since the domains of the σi RDF instance mappings are all mutually exclusive. Since they are also exclusive from SG,

SG union [ σ1 + ... + σn]( μ1(BGP1) union ... union μn(BGPn) )
= [ σ1 + ... + σn](SG union μ1(BGP1) union ... union μn(BGPn) )

i.e.

SG union μ1(BGP1) union ... union μn(BGPn)

has an instance which is a subgraph of SG, so is simply entailed by SG by the RDF interpolation lemma [RDF-MT].
```

