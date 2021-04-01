---
title: SPARQL 1.1 笔记(三)
date: 2021-03-21 15:10:43
tags: 语义Web
---

## 17 表达式与测试值

SPARQL`FILTERs`根据给定的约束来限制图模式匹配的解决方案。特别是， `FILTERs`消除一些解决方案，这些解决方案在被替换为表达式时会导致的有效布尔值`false`或产生错误。有效布尔值在第17.2.2节中定义和在XML查询语言中定义的错误。<u>这些错误在`FILTER`评估之外没有影响</u>。

RDF文字可能有一个数据类型IRI：

```turtle
@prefix a:          <http://www.w3.org/2000/10/annotation-ns#> .
@prefix dc:         <http://purl.org/dc/elements/1.1/> .

_:a   a:annotates   <http://www.w3.org/TR/rdf-sparql-query/> .
_:a   dc:date       "2004-12-31T19:00:00-05:00" .

_:b   a:annotates   <http://www.w3.org/TR/rdf-sparql-query/> .
_:b   dc:date       "2004-12-31T19:01:00-05:00"^^<http://www.w3.org/2001/XMLSchema#dateTime> .
```

第一个`dc:date`三元组的宾语没有类型信息。第二个具有数据类型`xsd:dateTime`。

<!--more-->

SPARQL表达式是根据语法构造的，并提供对函数（由IRI命名）和运算符函数（由SPARQL语法中的关键字和符号调用）的访问。SPARQL运算符可用于比较类型文字的值：

```SPARQL
PREFIX a:      <http://www.w3.org/2000/10/annotation-ns#>
PREFIX dc:     <http://purl.org/dc/elements/1.1/>
PREFIX xsd:    <http://www.w3.org/2001/XMLSchema#>

SELECT ?annot
WHERE { ?annot  a:annotates  <http://www.w3.org/TR/rdf-sparql-query/> .
        ?annot  dc:date      ?date .
        FILTER ( ?date > "2005-01-01T00:00:00Z"^^xsd:dateTime ) }
```

SPARQL运算符在第17.3节中列出，并与它们的语法结果关联。

此外，SPARQL还提供了调用任意功能的能力，包括第17.5节中列出的XPath强制转换函数的子集。这些函数在SPARQL查询中通过名称（IRI）来调用。例如：

```
... FILTER ( xsd:dateTime(?date) < xsd:dateTime("2005-01-01T00:00:00Z") ) ...
```

本节中的印刷约定：XPath运算符标有前缀`op:`。XPath运算符没有命名空间。 `op:`是标签约定。

### 17.1 操作数数据类型

SPARQL函数和运算符对RDF术语和SPARQL变量进行操作。这些函数和运算符的子集来自XQuery 1.0和XPath 2.0函数和运算符，具有XML Schema类型的值参数和返回类型。RDF`typed literals`作为参数传递给这些函数和运算符被映射到XML模式类型`lexical form`的字符串值的和原子数据类型对应的数据类型IRI。返回的类型值以相同的方式映射回RDF `typed literals`。

SPARQL还有其他运算符，它们对RDF术语的特定子集进行运算。当引用类型时，以下术语表示一个带有相应数据类型IRI的`typed literal`：

- `xsd:integer`
- `xsd:decimal`
- `xsd:float`
- `xsd:double`
- `xsd:string`
- `xsd:boolean`
- `xsd:dateTime`

以下术语标识了SPARQL值测试中使用的其他类型：

- 数字表示`typed literals`带有数据类型`xsd:integer`，`xsd:decimal`，`xsd:float`，和`xsd:double`。
- 简单文字表示`plain literal`不带有 `language tag`。
- RDF术语表示的类型`IRI`，`literal`和`blank node`。
- 变量表示SPARQL变量。

以下类型是从数字类型派生的，它们是采用数字参数的函数和运算符的有效参数：

- `xsd:nonPositiveInteger`
- `xsd:negativeInteger`
- `xsd:long`
- `xsd:int`
- `xsd:short`
- `xsd:byte`
- `xsd:nonNegativeInteger`
- `xsd:unsignedLong`
- `xsd:unsignedInt`
- `xsd:unsignedShort`
- `xsd:unsignedByte`
- `xsd:positiveInteger`

SPARQL语言扩展可以将其他类型视为从XML模式数据类型派生。

### 17.2 过滤器评估

SPARQL提供了XQuery Operator Mapping定义的功能和运算符的子集。XQuery 1.0的2.2.3节“表达式处理”描述了XPath函数的调用。以下规则适应了XQuery和SPARQL在数据和执行模型上的差异：

- 与XPath / XQuery不同，SPARQL函数不处理节点序列。在解释XPath函数的语义时，假定每个参数都是单个节点的序列。
- 用错误类型的参数调用的函数将产生类型错误。有效的布尔值参数（在下面的运算符映射表中标记为“ xsd：boolean（EBV）”）使用第17.2.2节中的EBV规则被强制为`xsd:boolean`。
- 除了BOUND，COALESCE， NOT EXISTS和EXISTS之外，所有函数和运算符均按RDF术语进行操作，并且如果未绑定任何参数，将产生类型错误。
- 除逻辑或（`||`）或逻辑与（`&&`）之外的任何其他遇到错误的表达式都将产生错误。
- 一个逻辑或，如果遇到一个分支是TRUE和另一个是FALSE或错误，将返回TRUE。
- 如果另一个分支为TRUE，则逻辑“与”仅在一个分支上遇到错误将返回错误，如果另一个分支为FALSE，则将返回FALSE。
- 在两个分支上都遇到错误的“逻辑或”或“逻辑与”都将产生*任何*一个错误。

| A    | B    | A \|\| B | A && B |
| ---- | ---- | -------- | ------ |
| T    | T    | T        | T      |
| T    | F    | T        | F      |
| F    | T    | T        | F      |
| F    | F    | F        | F      |
| T    | E    | T        | E      |
| E    | T    | T        | E      |
| F    | E    | E        | F      |
| E    | F    | E        | F      |
| E    | E    | E        | E      |

#### 17.2.1 调用方式

SPARQL定义了用于在参数列表上调用函数的语法。除非另有说明，否则将按以下方式调用它们：

- 计算参数表达式，产生参数值。没有定义参数评估的顺序。
- 根据需要提升数字参数以适合该函数或运算符的预期类型。
- 函数或运算符在参数值上调用。

如果这些步骤中的任何一个失败，则调用将产生一个错误。错误的影响在“过滤器评估”中定义。

也有“函数形式”，它们对每种形式指定的函数具有不同的评估规则。

#### 17.2.2 有效布尔值（EBV）

有效布尔值被用于计算参数的逻辑功能的逻辑与，逻辑或，和FN：not，以及表达式`FILTER`评估的结果。

XQuery有效布尔值规则依赖于XPath的fn：boolean的定义。以下规则反映了`fn:boolean`适用于SPARQL查询中存在的参数类型的规则：

- 如果词法形式对该数据类型`xsd:boolean`**无效**（例如，“ abc” ^^ xsd：integer），则任何类型为`xsd:boolean`或数字的文字的EBV均为false。
- 如果参数是一个带有数据类型`xsd:boolean`的类型文字，并且它有一个有效的词汇形式，EBV是参数的值。
- 如果参数是普通文字或数据类型为`xsd:string`的带类型文字，则如果值的长度为零，则EBV为false；否则，EBV为true。
- 如果参数是数字类型或具有从数字类型派生的数据类型的类型文字，并且具有有效的词法形式，则如果操作数值为NaN或数值等于零，则EBV为false；否则，为EBV为true。
- 所有其他参数，包括未绑定的参数，都会产生类型错误。

一个EBV`true`表示为类型文字，其数据类型为`xsd:boolean`而词汇值为“ true”；EBV为`false`时，将其表示为类型文字，其数据类型为，`xsd:boolean`而词汇值为“ false”。

### 17.3 操作符映射

SPARQL语法标识用于构造约束的一组运算符（例如&&，*，isIRI）。下表将这些语法产品中的每一个与适当的操作数以及由XQuery 1.0和XPath 2.0函数和运算符或17.4节中指定的SPARQL运算符定义的运算符函数相关联。在为给定的一组参数选择运算符定义时，将使用具有最特定参数的定义。例如，在评估`xsd:integer = xsd:signedInt`时，将应用`=`定义两个`numeric`参数，而不是具有两个RDF术语的定义。该表的排列方式是使最可行的候选者最具体。<u>在没有适当操作数的情况下调用运算符会导致类型错误。</u>

SPARQL遵循XPath的方案，用于数字类型提升和数字类型运算符的参数的子类型替换。XPath的操作映射规则是数字操作数（`xsd:integer`，`xsd:decimal`，`xsd:float`，`xsd:double`，和从一个派生类型数字型）适用于SPARQL运营商。一些运算符与嵌套函数表达式关联，例如`fn:not(op:numeric-equal(A, B))`。注意，按照XPath的定义，如果`fn:not`和`op:numeric-equal`的参数是错误将产生错误。

`fn:compare`的排序是通过XPath和`http://www.w3.org/2005/xpath-functions/collation/codepoint`定义。该排序规则允许基于代码点值进行字符串比较。可以使用RDF术语等效性来测试代码点字符串等效性。

**SPARQL一元运算符**

| Operator               | Type(A)           | Function                  | Result type |
| :--------------------- | ----------------- | ------------------------- | ----------- |
| XQuery Unary Operators |                   |                           |             |
| ! A                    | xsd:boolean (EBV) | fn:not(A)                 | xsd:boolean |
| + A                    | numeric           | op:numeric-unary-plus(A)  | numeric     |
| - A                    | numeric           | op:numeric-unary-minus(A) | numeric     |

**SPARQL二元运算符**

| Operator                                                     | Type(A)                                                      | Type(B)                                                      | Function                                                     | Result type                                               |
| :----------------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | --------------------------------------------------------- |
| Logical Connectives                                          |                                                              |                                                              |                                                              |                                                           |
| [A \|\| B](https://www.w3.org/TR/sparql11-query/#rConditionalOrExpression) | xsd:boolean [(EBV)](https://www.w3.org/TR/sparql11-query/#ebv-arg) | xsd:boolean [(EBV)](https://www.w3.org/TR/sparql11-query/#ebv-arg) | [logical-or](https://www.w3.org/TR/sparql11-query/#func-logical-or)(A, B) | xsd:boolean                                               |
| [A && B](https://www.w3.org/TR/sparql11-query/#rConditionalAndExpression) | xsd:boolean [(EBV)](https://www.w3.org/TR/sparql11-query/#ebv-arg) | xsd:boolean [(EBV)](https://www.w3.org/TR/sparql11-query/#ebv-arg) | [logical-and](https://www.w3.org/TR/sparql11-query/#func-logical-and)(A, B) | xsd:boolean                                               |
| **XPath Tests**                                              |                                                              |                                                              |                                                              |                                                           |
| [A = B](https://www.w3.org/TR/sparql11-query/#rRelationalExpression) | numeric                                                      | numeric                                                      | [op:numeric-equal](http://www.w3.org/TR/xpath-functions/#func-numeric-equal)(A, B) | xsd:boolean                                               |
| [A = B](https://www.w3.org/TR/sparql11-query/#rRelationalExpression) | simple literal                                               | simple literal                                               | [op:numeric-equal](http://www.w3.org/TR/xpath-functions/#func-numeric-equal)([fn:compare](http://www.w3.org/TR/xpath-functions/#func-compare)(A, B), 0) | xsd:boolean                                               |
| [A = B](https://www.w3.org/TR/sparql11-query/#rRelationalExpression) | xsd:string                                                   | xsd:string                                                   | [op:numeric-equal](http://www.w3.org/TR/xpath-functions/#func-numeric-equal)([fn:compare](http://www.w3.org/TR/xpath-functions/#func-compare)([STR](https://www.w3.org/TR/sparql11-query/#func-str)(A), [STR](https://www.w3.org/TR/sparql11-query/#func-str)(B)), 0) | xsd:boolean                                               |
| [A = B](https://www.w3.org/TR/sparql11-query/#rRelationalExpression) | xsd:boolean                                                  | xsd:boolean                                                  | [op:boolean-equal](http://www.w3.org/TR/xpath-functions/#func-boolean-equal)(A, B) | xsd:boolean                                               |
| [A = B](https://www.w3.org/TR/sparql11-query/#rRelationalExpression) | xsd:dateTime                                                 | xsd:dateTime                                                 | [op:dateTime-equal](http://www.w3.org/TR/xpath-functions/#func-dateTime-equal)(A, B) | xsd:boolean                                               |
| [A != B](https://www.w3.org/TR/sparql11-query/#rRelationalExpression) | numeric                                                      | numeric                                                      | [fn:not](http://www.w3.org/TR/xpath-functions/#func-not)([op:numeric-equal](http://www.w3.org/TR/xpath-functions/#func-numeric-equal)(A, B)) | xsd:boolean                                               |
| [A != B](https://www.w3.org/TR/sparql11-query/#rRelationalExpression) | simple literal                                               | simple literal                                               | [fn:not](http://www.w3.org/TR/xpath-functions/#func-not)([op:numeric-equal](http://www.w3.org/TR/xpath-functions/#func-numeric-equal)([fn:compare](http://www.w3.org/TR/xpath-functions/#func-compare)(A, B), 0)) | xsd:boolean                                               |
| [A != B](https://www.w3.org/TR/sparql11-query/#rRelationalExpression) | xsd:string                                                   | xsd:string                                                   | [fn:not](http://www.w3.org/TR/xpath-functions/#func-not)([op:numeric-equal](http://www.w3.org/TR/xpath-functions/#func-numeric-equal)([fn:compare](http://www.w3.org/TR/xpath-functions/#func-compare)([STR](https://www.w3.org/TR/sparql11-query/#func-str)(A), [STR](https://www.w3.org/TR/sparql11-query/#func-str)(B)), 0)) | xsd:boolean                                               |
| [A != B](https://www.w3.org/TR/sparql11-query/#rRelationalExpression) | xsd:boolean                                                  | xsd:boolean                                                  | [fn:not](http://www.w3.org/TR/xpath-functions/#func-not)([op:boolean-equal](http://www.w3.org/TR/xpath-functions/#func-boolean-equal)(A, B)) | xsd:boolean                                               |
| [A != B](https://www.w3.org/TR/sparql11-query/#rRelationalExpression) | xsd:dateTime                                                 | xsd:dateTime                                                 | [fn:not](http://www.w3.org/TR/xpath-functions/#func-not)([op:dateTime-equal](http://www.w3.org/TR/xpath-functions/#func-dateTime-equal)(A, B)) | xsd:boolean                                               |
| [A < B](https://www.w3.org/TR/sparql11-query/#rRelationalExpression) | numeric                                                      | numeric                                                      | [op:numeric-less-than](http://www.w3.org/TR/xpath-functions/#func-numeric-less-than)(A, B) | xsd:boolean                                               |
| [A < B](https://www.w3.org/TR/sparql11-query/#rRelationalExpression) | simple literal                                               | simple literal                                               | [op:numeric-equal](http://www.w3.org/TR/xpath-functions/#func-numeric-equal)([fn:compare](http://www.w3.org/TR/xpath-functions/#func-compare)(A, B), -1) | xsd:boolean                                               |
| [A < B](https://www.w3.org/TR/sparql11-query/#rRelationalExpression) | xsd:string                                                   | xsd:string                                                   | [op:numeric-equal](http://www.w3.org/TR/xpath-functions/#func-numeric-equal)([fn:compare](http://www.w3.org/TR/xpath-functions/#func-compare)([STR](https://www.w3.org/TR/sparql11-query/#func-str)(A), [STR](https://www.w3.org/TR/sparql11-query/#func-str)(B)), -1) | xsd:boolean                                               |
| [A < B](https://www.w3.org/TR/sparql11-query/#rRelationalExpression) | xsd:boolean                                                  | xsd:boolean                                                  | [op:boolean-less-than](http://www.w3.org/TR/xpath-functions/#func-boolean-less-than)(A, B) | xsd:boolean                                               |
| [A < B](https://www.w3.org/TR/sparql11-query/#rRelationalExpression) | xsd:dateTime                                                 | xsd:dateTime                                                 | [op:dateTime-less-than](http://www.w3.org/TR/xpath-functions/#func-dateTime-less-than)(A, B) | xsd:boolean                                               |
| [A > B](https://www.w3.org/TR/sparql11-query/#rRelationalExpression) | numeric                                                      | numeric                                                      | [op:numeric-greater-than](http://www.w3.org/TR/xpath-functions/#func-numeric-greater-than)(A, B) | xsd:boolean                                               |
| [A > B](https://www.w3.org/TR/sparql11-query/#rRelationalExpression) | simple literal                                               | simple literal                                               | [op:numeric-equal](http://www.w3.org/TR/xpath-functions/#func-numeric-equal)([fn:compare](http://www.w3.org/TR/xpath-functions/#func-compare)(A, B), 1) | xsd:boolean                                               |
| [A > B](https://www.w3.org/TR/sparql11-query/#rRelationalExpression) | xsd:string                                                   | xsd:string                                                   | [op:numeric-equal](http://www.w3.org/TR/xpath-functions/#func-numeric-equal)([fn:compare](http://www.w3.org/TR/xpath-functions/#func-compare)([STR](https://www.w3.org/TR/sparql11-query/#func-str)(A), [STR](https://www.w3.org/TR/sparql11-query/#func-str)(B)), 1) | xsd:boolean                                               |
| [A > B](https://www.w3.org/TR/sparql11-query/#rRelationalExpression) | xsd:boolean                                                  | xsd:boolean                                                  | [op:boolean-greater-than](http://www.w3.org/TR/xpath-functions/#func-boolean-greater-than)(A, B) | xsd:boolean                                               |
| [A > B](https://www.w3.org/TR/sparql11-query/#rRelationalExpression) | xsd:dateTime                                                 | xsd:dateTime                                                 | [op:dateTime-greater-than](http://www.w3.org/TR/xpath-functions/#func-dateTime-greater-than)(A, B) | xsd:boolean                                               |
| [A <= B](https://www.w3.org/TR/sparql11-query/#rRelationalExpression) | numeric                                                      | numeric                                                      | [logical-or](https://www.w3.org/TR/sparql11-query/#func-logical-or)([op:numeric-less-than](http://www.w3.org/TR/xpath-functions/#func-numeric-less-than)(A, B), [op:numeric-equal](http://www.w3.org/TR/xpath-functions/#func-numeric-equal)(A, B)) | xsd:boolean                                               |
| [A <= B](https://www.w3.org/TR/sparql11-query/#rRelationalExpression) | simple literal                                               | simple literal                                               | [fn:not](http://www.w3.org/TR/xpath-functions/#func-not)([op:numeric-equal](http://www.w3.org/TR/xpath-functions/#func-numeric-equal)([fn:compare](http://www.w3.org/TR/xpath-functions/#func-compare)(A, B), 1)) | xsd:boolean                                               |
| [A <= B](https://www.w3.org/TR/sparql11-query/#rRelationalExpression) | xsd:string                                                   | xsd:string                                                   | [fn:not](http://www.w3.org/TR/xpath-functions/#func-not)([op:numeric-equal](http://www.w3.org/TR/xpath-functions/#func-numeric-equal)([fn:compare](http://www.w3.org/TR/xpath-functions/#func-compare)([STR](https://www.w3.org/TR/sparql11-query/#func-str)(A), [STR](https://www.w3.org/TR/sparql11-query/#func-str)(B)), 1)) | xsd:boolean                                               |
| [A <= B](https://www.w3.org/TR/sparql11-query/#rRelationalExpression) | xsd:boolean                                                  | xsd:boolean                                                  | [fn:not](http://www.w3.org/TR/xpath-functions/#func-not)([op:boolean-greater-than](http://www.w3.org/TR/xpath-functions/#func-boolean-greater-than)(A, B)) | xsd:boolean                                               |
| [A <= B](https://www.w3.org/TR/sparql11-query/#rRelationalExpression) | xsd:dateTime                                                 | xsd:dateTime                                                 | [fn:not](http://www.w3.org/TR/xpath-functions/#func-not)([op:dateTime-greater-than](http://www.w3.org/TR/xpath-functions/#func-dateTime-greater-than)(A, B)) | xsd:boolean                                               |
| [A >= B](https://www.w3.org/TR/sparql11-query/#rRelationalExpression) | numeric                                                      | numeric                                                      | [logical-or](https://www.w3.org/TR/sparql11-query/#func-logical-or)([op:numeric-greater-than](http://www.w3.org/TR/xpath-functions/#func-numeric-greater-than)(A, B), [op:numeric-equal](http://www.w3.org/TR/xpath-functions/#func-numeric-equal)(A, B)) | xsd:boolean                                               |
| [A >= B](https://www.w3.org/TR/sparql11-query/#rRelationalExpression) | simple literal                                               | simple literal                                               | [fn:not](http://www.w3.org/TR/xpath-functions/#func-not)([op:numeric-equal](http://www.w3.org/TR/xpath-functions/#func-numeric-equal)([fn:compare](http://www.w3.org/TR/xpath-functions/#func-compare)(A, B), -1)) | xsd:boolean                                               |
| [A >= B](https://www.w3.org/TR/sparql11-query/#rRelationalExpression) | xsd:string                                                   | xsd:string                                                   | [fn:not](http://www.w3.org/TR/xpath-functions/#func-not)([op:numeric-equal](http://www.w3.org/TR/xpath-functions/#func-numeric-equal)([fn:compare](http://www.w3.org/TR/xpath-functions/#func-compare)([STR](https://www.w3.org/TR/sparql11-query/#func-str)(A), [STR](https://www.w3.org/TR/sparql11-query/#func-str)(B)), -1)) | xsd:boolean                                               |
| [A >= B](https://www.w3.org/TR/sparql11-query/#rRelationalExpression) | xsd:boolean                                                  | xsd:boolean                                                  | [fn:not](http://www.w3.org/TR/xpath-functions/#func-not)([op:boolean-less-than](http://www.w3.org/TR/xpath-functions/#func-boolean-less-than)(A, B)) | xsd:boolean                                               |
| [A >= B](https://www.w3.org/TR/sparql11-query/#rRelationalExpression) | xsd:dateTime                                                 | xsd:dateTime                                                 | [fn:not](http://www.w3.org/TR/xpath-functions/#func-not)([op:dateTime-less-than](http://www.w3.org/TR/xpath-functions/#func-dateTime-less-than)(A, B)) | xsd:boolean                                               |
| XPath Arithmetic                                             |                                                              |                                                              |                                                              |                                                           |
| [A * B](https://www.w3.org/TR/sparql11-query/#rMultiplicativeExpression) | numeric                                                      | numeric                                                      | [op:numeric-multiply](http://www.w3.org/TR/xpath-functions/#func-numeric-multiply)(A, B) | numeric                                                   |
| [A / B](https://www.w3.org/TR/sparql11-query/#rMultiplicativeExpression) | numeric                                                      | numeric                                                      | [op:numeric-divide](http://www.w3.org/TR/xpath-functions/#func-numeric-divide)(A, B) | numeric; but xsd:decimal if both operands are xsd:integer |
| [A + B](https://www.w3.org/TR/sparql11-query/#rAdditiveExpression) | numeric                                                      | numeric                                                      | [op:numeric-add](http://www.w3.org/TR/xpath-functions/#func-numeric-add)(A, B) | numeric                                                   |
| [A - B](https://www.w3.org/TR/sparql11-query/#rAdditiveExpression) | numeric                                                      | numeric                                                      | [op:numeric-subtract](http://www.w3.org/TR/xpath-functions/#func-numeric-subtract)(A, B) | numeric                                                   |
| SPARQL Tests                                                 |                                                              |                                                              |                                                              |                                                           |
| [A = B](https://www.w3.org/TR/sparql11-query/#rRelationalExpression) | RDF term                                                     | RDF term                                                     | [RDFterm-equal](https://www.w3.org/TR/sparql11-query/#func-RDFterm-equal)(A, B) | xsd:boolean                                               |
| [A != B](https://www.w3.org/TR/sparql11-query/#rRelationalExpression) | RDF term                                                     | RDF term                                                     | [fn:not](http://www.w3.org/TR/xpath-functions/#func-not)([RDFterm-equal](https://www.w3.org/TR/sparql11-query/#func-RDFterm-equal)(A, B)) | xsd:boolean                                               |

通过评估该参数的有效布尔值，可以将标记为“（EBV）”的xsd：boolean函数参数强制转换为xsd：boolean。

#### 17.3.1 操作符可扩展性

SPARQL语言扩展可以在运算符和运算符功能之间提供附加的关联；这等于在上面的表中添加一些行。除了上面定义的语义中的类型错误之外，没有其他运算符可以产生替换任何结果的结果。该规则的结果是，在将`FILTER`作为未扩展的应用实现后，SPARQL`FILTER`将产生*至少*相同的中间绑定。

特别是在`ORDER BY`子句中使用时，期望'<'运算符的其他映射来控制操作数的相对顺序。

### 17.4 函数定义

本节定义了SPARQL查询语言引入的运算符和函数。这些示例显示了由适当的语法构造调用的运算符的行为。

#### 17.4.1 函数形式

##### 17.4.1.1 Bound

```
xsd:boolean  BOUND (variable var)
```

如果`var`绑定到值，则返回`true`。否则返回false。值为NaN或INF的变量被认为是绑定的。

**数据**

```turtle
@prefix foaf:        <http://xmlns.com/foaf/0.1/> .
@prefix dc:          <http://purl.org/dc/elements/1.1/> .
@prefix xsd:          <http://www.w3.org/2001/XMLSchema#> .

_:a  foaf:givenName  "Alice".

_:b  foaf:givenName  "Bob" .
_:b  dc:date         "2005-04-04T04:04:04Z"^^xsd:dateTime .
```

```SPARQL
PREFIX foaf: <http://xmlns.com/foaf/0.1/>
PREFIX dc:   <http://purl.org/dc/elements/1.1/>
PREFIX xsd:   <http://www.w3.org/2001/XMLSchema#>
SELECT ?givenName
 WHERE { ?x foaf:givenName  ?givenName .
         OPTIONAL { ?x dc:date ?date } .
         FILTER ( bound(?date) ) }
```

| givenName |
| --------- |
| "Bob"     |

可以通过指定引入变量的可选图形模式来测试图形模式是否*未*表达，并测试该变量是否未绑定。在逻辑编程中，这被称为*“否定或失败”*。

此查询将人员`name`匹配，但*没有*表达`date`：

```
PREFIX foaf: <http://xmlns.com/foaf/0.1/>
PREFIX dc:   <http://purl.org/dc/elements/1.1/>
SELECT ?name
 WHERE { ?x foaf:givenName  ?name .
         OPTIONAL { ?x dc:date ?date } .
         FILTER (!bound(?date)) }
```

| name    |
| ------- |
| "Alice" |

因为鲍勃（Bob）`dc:date`是众所周知的，所以`"Bob"`不是该查询的解决方案。

##### 17.4.1.2 IF

```
rdfTerm  IF (expression1, expression2, expression3)
```

`IF`函数形式计算的第一个参数，将其解释为有效布尔值，然后如果EBV为真，返回的值`expression2`，否则返回的值`expression3`。在`expression2`和`expression3`中只有一个进行评估。<u>如果评估第一个参数有错误，则对`IF`表达式的评估会引发错误。</u>

示例：假设?x = 2，?z = 0并且?y在某些查询解决方案中未绑定：

| `IF(?x = 2, "yes", "no")`    | returns "yes"                                         |
| ---------------------------- | ----------------------------------------------------- |
| `IF(bound(?y), "yes", "no")` | returns "no"                                          |
| `IF(?x=2, "yes", 1/?z)`      | returns "yes", the expression `1/?z` is not evaluated |
| `IF(?x=1, "yes", 1/?z)`      | raises an error                                       |
| `IF("2" > 1, "yes", "no")`   | raises an error                                       |

##### 17.4.1.3 COALESCE

```
rdfTerm  COALESCE(expression, ....)
```

`COALESCE`函数形式返回所述<u>第一个表达式的RDF术语值没有错误</u>的计算结果。在SPARQL中，评估未绑定变量会引发错误。

如果没有一个参数求值为RDF术语，则会引发错误。如果没有正确计算没有表达式的表达式，则会引发错误。

示例：假设?x = 2并且?y在某些查询解决方案中未绑定：

| `COALESCE(?x, 1/0)` | returns 2, the value of `x`               |
| ------------------- | ----------------------------------------- |
| `COALESCE(1/0, ?x)` | returns 2                                 |
| `COALESCE(5, ?x)`   | returns 5                                 |
| `COALESCE(?y, 3)`   | returns 3                                 |
| `COALESCE(?y)`      | raises an error because `y` is not bound. |

##### 17.4.1.4 NOT EXISTS和EXISTS

`EXISTS`是一个采用图形模式的过滤运算符。 `EXISTS`返回`true`/`false` 取决于**模式**是否与给定当前组图模式中绑定的数据集匹配，查询评估中此时的数据集和活动图匹配。不会发生变量的其他绑定。`NOT EXISTS`形式转化成`fn:not(EXISTS{...})`。

```
 xsd:boolean  NOT EXISTS { pattern }
```

如果模式匹配返回false，否则返回true。

`NOT EXISTS { pattern }` 等效于 `fn:not(EXISTS { pattern })`.

```
 xsd:boolean  EXISTS { pattern }
```

如果模式匹配返回true，否则返回false。

`pattern`中的变量与当前解决方案映射中绑定的值绑定。模式`pattern`中未绑定到当前解决方案映射中的变量将参与模式匹配。

为方便起见，我们引入了一个Exists函数 ，该函数对SPARQL代数表达式求值并返回true或false，这取决于该模式是否存在任何解决方案（给定解决方案映射正在由过滤器操作进行测试的情况下）。

##### 17.4.1.5 logical-or

```
xsd:boolean  xsd:boolean left || xsd:boolean right
```

返回一个`left`和`right`的逻辑`OR`。请注意，逻辑或运算符对其参数的有效布尔值进行运算。

注意：有关操作符`||`对错误的处理，请参阅第17.2节。

##### 17.4.1.5 logical-and

```
 xsd:boolean  xsd:boolean left && xsd:boolean right
```

返回一个`left`和`right`的逻辑`AND`。请注意，逻辑与运算符对其参数的有效布尔值进行运算。

注意：有关操作符`&&`对错误的处理，请参阅第17.2节。

##### 17.4.1.7 RDFterm-equal

```
 xsd:boolean  RDF term term1 = RDF term term2
```

如果`term1`和`term2`与资源描述框架（RDF）中定义的RDF术语相同，则返回TRUE；如果参数都是文字的，但不是相同的RDF术语*，则产生类型错误; 否则返回FALSE。`term1`并且`term2`是相同的，如果以下任一为真：

- term1和term2是等效的IRI。
- term1和term2是等效的文字。
- term1和term2是相同的空白节点。

```turtle
@prefix foaf:       <http://xmlns.com/foaf/0.1/> .

_:a  foaf:name       "Alice".
_:a  foaf:mbox       <mailto:alice@work.example> .

_:b  foaf:name       "Ms A.".
_:b  foaf:mbox       <mailto:alice@work.example> .
```

此查询查找具有多个`foaf:name`三元组的人员：

```SPARQL
PREFIX foaf: <http://xmlns.com/foaf/0.1/>
SELECT ?name1 ?name2
WHERE { ?x foaf:name  ?name1 ;
        foaf:mbox  ?mbox1 .
        ?y foaf:name  ?name2 ;
        foaf:mbox  ?mbox2 .
        FILTER (?mbox1 = ?mbox2 && ?name1 != ?name2)
      }
```

| name1   | name2   |
| ------- | ------- |
| "Alice" | "Ms A." |
| "Ms A." | "Alice" |

在此查询中，文件在特定的日期和时间进行了注释（New Year's Day 2005, measures in timezone +00:00），RDF术语不同，但具有相同的值：

```turtle
@prefix a:          <http://www.w3.org/2000/10/annotation-ns#> .
@prefix dc:         <http://purl.org/dc/elements/1.1/> .

_:b   a:annotates   <http://www.w3.org/TR/rdf-sparql-query/> .
_:b   dc:date       "2004-12-31T19:00:00-05:00"^^<http://www.w3.org/2001/XMLSchema#dateTime> .
```

```SPARQL
PREFIX a:      <http://www.w3.org/2000/10/annotation-ns#>
PREFIX dc:     <http://purl.org/dc/elements/1.1/>
PREFIX xsd:    <http://www.w3.org/2001/XMLSchema#>

SELECT ?annotates
WHERE { ?annot  a:annotates  ?annotates .
        ?annot  dc:date      ?date .
        FILTER ( ?date = xsd:dateTime("2005-01-01T00:00:00Z") ) 
      }
```

| annotates                                 |
| ----------------------------------------- |
| \<http://www.w3.org/TR/rdf-sparql-query/> |

在两个类型的文字测试中调用RDFterm-equal，以获取等效值。扩展的实现可能支持其他数据类型。处理查询的实现在不支持的数据类型（以及不相同的词法形式和数据类型IRI）上进行等效性测试的处理返回错误，指示无法确定值是否相等。例如，当测试`"iiii"^^my:romanNumeral = "iv"^^my:romanNumeral`或  `"iiii"^^my:romanNumeral != "iv"^^my:romanNumeral`。时，未扩展的实现将产生错误.

##### 17.4.1.8 sameTerm

```
xsd:boolean  sameTerm (RDF term term1, RDF term term2)
```

如果`term1`和`term2`与资源描述框架（RDF）中定义的RDF术语相同，则返回TRUE ；否则返回FALSE。

```turtle
@prefix foaf:       <http://xmlns.com/foaf/0.1/> .

_:a  foaf:name       "Alice".
_:a  foaf:mbox       <mailto:alice@work.example> .

_:b  foaf:name       "Ms A.".
_:b  foaf:mbox       <mailto:alice@work.example> .
```

此查询查找具有多个`foaf:name`三元组的人员：

```SPARQL
PREFIX foaf: <http://xmlns.com/foaf/0.1/>
SELECT ?name1 ?name2
WHERE { ?x foaf:name  ?name1 ;
        foaf:mbox  ?mbox1 .
         ?y foaf:name  ?name2 ;
         foaf:mbox  ?mbox2 .
         FILTER (sameTerm(?mbox1, ?mbox2) && !sameTerm(?name1, ?name2))
      } 
```

| name1   | name2   |
| ------- | ------- |
| "Alice" | "Ms A." |
| "Ms A." | "Alice" |

与RDFterm-equal不同，<u>sameTerm可用于测试不支持的数据类型的非等价类型文字</u>：

```turtle
@prefix :          <http://example.org/WMterms#> .
@prefix t:         <http://example.org/types#> .

_:c1  :label        "Container 1" .
_:c1  :weight       "100"^^t:kilos .
_:c1  :displacement  "100"^^t:liters .

_:c2  :label        "Container 2" .
_:c2  :weight       "100"^^t:kilos .
_:c2  :displacement  "85"^^t:liters .

_:c3  :label        "Container 3" .
_:c3  :weight       "85"^^t:kilos .
_:c3  :displacement  "85"^^t:liters .
```

```SPARQL
PREFIX  :      <http://example.org/WMterms#>
PREFIX  t:     <http://example.org/types#>

SELECT ?aLabel ?bLabel
WHERE { ?a  :label        ?aLabel .
        ?a  :weight       ?aWeight .
        ?a  :displacement ?aDisp .

        ?b  :label        ?bLabel .
        ?b  :weight       ?bWeight .
        ?b  :displacement ?bDisp .

        FILTER ( sameTerm(?aWeight, ?bWeight) && !sameTerm(?aDisp, ?bDisp)) }
```

| aLabel        | bLabel        |
| ------------- | ------------- |
| "Container 1" | "Container 2" |
| "Container 2" | "Container 1" |

同样重量的盒子的检验也可以使用'='运算符（RDFterm-equal）进行，因为检验`"100"^^t:kilos = "85"^^t:kilos`会导致错误，从而消除了潜在的解决方案。

##### 17.4.1.9 IN

```
boolean  rdfTerm IN (expression, ...)
```

该`IN`操作符的测试在右边的表达式列表的值在左侧的RDF项是否被找到。该测试是使用“ =”运算符完成的，该运算符将测试由运算符映射确定的相同值。

右侧的零项列表是合法的。

如果在术语列表的其他位置未找到要测试的RDF术语，则比较中的错误会导致`IN`表达式引发错误。

该`IN`运算符等效于SPARQL表达式：

```
（lhs = expression1）|| （lhs = expression2）|| ...
```

**示例**

| `2 IN (1, 2, 3)`                          | true            |
| ----------------------------------------- | --------------- |
| `2 IN ()`                                 | false           |
| `2 IN (<http://example/iri>, "str", 2.0)` | true            |
| `2 IN (1/0, 2)`                           | true            |
| `2 IN (2, 1/0)`                           | true            |
| `2 IN (3, 1/0)`                           | raises an error |

##### 17.4.1.10 NOT IN

```
boolean  rdfTerm NOT IN (expression, ...)
```

该`IN`操作符的测试在右边的表达式列表的值在左侧的RDF项是否找不到。该测试是使用“ !=”运算符完成的，该运算符将测试由运算符映射确定的相同值。

右侧的零项列表是合法的。

如果在术语列表的其他位置未找到要测试的RDF术语，则比较中的错误会导致`IN`表达式引发错误。

该`IN`运算符等效于SPARQL表达式：

```
(lhs != expression1) && (lhs != expression2) && ...
```

`NOT IN (...)`等效于 `!(IN (...))`.

**示例**

| `2 NOT IN (1, 2, 3)`                          | false           |
| --------------------------------------------- | --------------- |
| `2 NOT IN ()`                                 | true            |
| `2 NOT IN (<http://example/iri>, "str", 2.0)` | false           |
| `2 NOT IN (1/0, 2)`                           | false           |
| `2 NOT IN (2, 1/0)`                           | false           |
| `2 NOT IN (3, 1/0)`                           | raises an error |

#### 17.4.2 RDF Term上的函数

##### 17.4.2.1 isIRI

```
xsd:boolean  isIRI (RDF term term)
xsd:boolean  isURI (RDF term term)
```

如果term是一个IRI，则返回true，否则返回false。isURI是isIRI运算符的备用写法。

```turtle
@prefix foaf:       <http://xmlns.com/foaf/0.1/> .

_:a  foaf:name       "Alice".
_:a  foaf:mbox       <mailto:alice@work.example> .

_:b  foaf:name       "Bob" .
_:b  foaf:mbox       "bob@work.example" .
```

这个查询匹配人名和邮箱是IRI的人

```SPARQL
PREFIX foaf: <http://xmlns.com/foaf/0.1/>
SELECT ?name ?mbox
 WHERE { ?x foaf:name  ?name ;
            foaf:mbox  ?mbox .
         FILTER isIRI(?mbox) }
```

| name    | mbox                         |
| ------- | ---------------------------- |
| "Alice" | \<mailto:alice@work.example> |

##### 17.4.2.2 isBlank

```
 xsd:boolean  isBlank (RDF term term)
```

如果term是空节点，则返回true，否则返回false。

```
@prefix a:          <http://www.w3.org/2000/10/annotation-ns#> .
@prefix dc:         <http://purl.org/dc/elements/1.1/> .
@prefix foaf:       <http://xmlns.com/foaf/0.1/> .

_:a   a:annotates   <http://www.w3.org/TR/rdf-sparql-query/> .
_:a   dc:creator    "Alice B. Toeclips" .

_:b   a:annotates   <http://www.w3.org/TR/rdf-sparql-query/> .
_:b   dc:creator    _:c .
_:c   foaf:given    "Bob".
_:c   foaf:family   "Smith".
```

这个查询匹配带有`dc:creator`的人，使用来自FOAF词汇表中的谓语表示的名字。

```
PREFIX a:      <http://www.w3.org/2000/10/annotation-ns#>
PREFIX dc:     <http://purl.org/dc/elements/1.1/>
PREFIX foaf:   <http://xmlns.com/foaf/0.1/>

SELECT ?given ?family
WHERE { ?annot  a:annotates  <http://www.w3.org/TR/rdf-sparql-query/> .
  		?annot  dc:creator   ?c .
  		OPTIONAL { ?c  foaf:given   ?given ; 
  					   foaf:family  ?family } .
  		FILTER isBlank(?c)
}
```

| given | family  |
| :---: | :-----: |
| "Bob" | "Smith" |

在此示例中，有两个`dc:creator`谓词对象，但是只有一个（`_:c`）是空白节点。

##### 17.4.2.3 isLiteral

```
 xsd:boolean  isLiteral (RDF term term)
```

如果term是文字，则返回true，否则返回false

```
@prefix foaf:       <http://xmlns.com/foaf/0.1/> .
              
_:a  foaf:name       "Alice".
_:a  foaf:mbox       <mailto:alice@work.example> .

_:b  foaf:name       "Bob" .
_:b  foaf:mbox       "bob@work.example" .
```

此查询与17.4.2.1中的查询类似，不同之处在于，该查询使用`name`和`mbox`**文字**进行匹配。这可用于查找错误的数据（`foaf:mbox`应将IRI作为其宾语）。

```
PREFIX foaf: <http://xmlns.com/foaf/0.1/>
SELECT ?name ?mbox
WHERE { ?x foaf:name  ?name ;
           foaf:mbox  ?mbox .
        FILTER isLiteral(?mbox) }
```

**结果**

| name  |        mbox        |
| :---: | :----------------: |
| "Bob" | "bob@work.example" |

##### 17.4.2.4 isNumeric

```
 xsd:boolean  isNumeric (RDF term term)
```

如果`term`为数字值，返回`true`，否则返回`false`。 `term` 如果它具有适当的数据类型（请参阅“操作数数据类型”一节）并且具有有效的词法形式，则为数字，从而使其成为采用数字参数的函数和运算符的有效参数。

**示例**

| `isNumeric(12)`                           | true  |
| ----------------------------------------- | ----- |
| `isNumeric("12")`                         | false |
| `isNumeric("12"^^xsd:nonNegativeInteger)` | true  |
| `isNumeric("1200"^^xsd:byte)`             | false |
| `isNumeric(<http://example/>)`            | false |

##### 17.4.2.5 str

```
 simple literal  STR (literal ltrl)
 simple literal  STR (IRI rsrc)
```

返回`ltrl`（一个文字）的词法形式; 返回`rsrc`（IRI）的**代码点**表示形式。这对于检查IRI的部分（例如主机名）很有用。

```
@prefix foaf:       <http://xmlns.com/foaf/0.1/> .

_:a  foaf:name       "Alice".
_:a  foaf:mbox       <mailto:alice@work.example> .

_:b  foaf:name       "Bob" .
_:b  foaf:mbox       <mailto:bob@home.example> .
```

这个查询选出在foaf文档中用他们的`work.example`的人

```
PREFIX foaf: <http://xmlns.com/foaf/0.1/>
SELECT ?name ?mbox
 WHERE { ?x foaf:name  ?name ;
            foaf:mbox  ?mbox .
         FILTER regex(str(?mbox), "@work\\.example$") }
```

**查询结果**

|  name   |            mbox             |
| :-----: | :-------------------------: |
| "Alice" | <mailto:alice@work.example> |

##### 17.4.2.6 lang

```
 simple literal  LANG (literal ltrl)
```

返回`ltrl`的语言标签（如果有的话）。如果`ltrl`没有语言标签，则返回`""`。请注意，<u>RDF数据模型不包括带有空语言标签的文字。</u>

```
@prefix foaf:       <http://xmlns.com/foaf/0.1/> .

_:a  foaf:name       "Robert"@en.
_:a  foaf:name       "Roberto"@es.
_:a  foaf:mbox       <mailto:bob@work.example> .
```

这个查询找到西班牙语的名字和邮箱

```
PREFIX foaf: <http://xmlns.com/foaf/0.1/>
SELECT ?name ?mbox
 WHERE { ?x foaf:name  ?name ;
            foaf:mbox  ?mbox .
         FILTER ( lang(?name) = "es" ) }
```

**查询结果**

|     name     |           mbox            |
| :----------: | :-----------------------: |
| "Roberto"@es | <mailto:bob@work.example> |

##### 17.4.2.7 datatype

```
 iri  DATATYPE (literal literal)
```

返回的数据类型`literal`的IRI。

- 如果文字是带类型的文字，则返回数据类型IRI。
- 如果文字是简单文字，则返回 `xsd:string`
- 如果文字是带有语言标签的文字，则返回 `rdf:langString`

```
@prefix foaf:       <http://xmlns.com/foaf/0.1/> .
@prefix eg:         <http://biometrics.example/ns#> .
@prefix xsd:        <http://www.w3.org/2001/XMLSchema#> .

_:a  foaf:name       "Alice".
_:a  eg:shoeSize     "9.5"^^xsd:float .

_:b  foaf:name       "Bob".
_:b  eg:shoeSize     "42"^^xsd:integer .
```

这个查询找到所有人鞋是整型的名字和鞋码

```
PREFIX foaf: <http://xmlns.com/foaf/0.1/>
PREFIX xsd:  <http://www.w3.org/2001/XMLSchema#>
PREFIX eg:   <http://biometrics.example/ns#>
SELECT ?name ?shoeSize
 WHERE { ?x foaf:name  ?name ; eg:shoeSize  ?shoeSize .
         FILTER ( datatype(?shoeSize) = xsd:integer ) }
```

**结果**

| name  | shoeSize |
| :---: | :------: |
| "Bob" |    42    |

在SPARQL 1.0中，未定义函数 `DATATYPE`是具有语言标签的文字。因此，当`DATATYPE` 使用带有语言标签的文字调用未扩展的实现时，将引发错误。 运算符可扩展性允许实现返回结果而不是引发错误。SPARQL 1.1定义了将`DATATYPE`应用到带有语言标记为的文字的结果 `rdf:langString`。

##### 17.4.2.8 IRI

```
 iri  IRI(simple literal)
 iri  IRI(xsd:string)
 iri  IRI(iri)
 iri  URI(simple literal)
 iri  URI(xsd:string)
 iri  URI(iri)
```

该`IRI`函数通过解析字符串参数来构造IRI。根据查询的基本IRI解析IRI，并且必须得出绝对IRI。

该`URI`函数是`IRI`的同义词。

如果函数传递了IRI，则它返回的IRI不变。

传递除简单文字，xsd: string或IRI之外的任何RDF术语都是错误的。

一个实现*可以*规范IRI。

**示例**

| `IRI("http://example/")` | \<http://example/> |
| ------------------------ | ------------------ |
| `IRI(<http://example/>)` | \<http://example/> |

##### 17.4.2.9 BNODE

```
blank node  BNODE()
blank node  BNODE(simple literal)
blank node  BNODE(xsd:string)
```

该`BNODE`函数构造一个空白节点，该节点不同于要查询的数据集中的所有空白节点，并且不同于通过调用此构造函数为其他查询解决方案创建的所有空白节点。如果没有使用参数形式，则每个调用都将构造一个不同的空白节点。如果使用带有简单文字的形式，则每个调用将为不同的简单文字生成不同的空白节点，并为一个解决方案映射的表达式中的相同简单文字的调用生成相同的空白节点。

##### 17.4.2.10 STRDT

```
literal  STRDT(simple literal lexicalForm, IRI datatypeIRI)
```

该`STRDT`函数使用参数指定的词法形式和类型构造文本。

| `STRDT("123", xsd:integer)`                    | "123"^^<http://www.w3.org/2001/XMLSchema#integer> |
| ---------------------------------------------- | ------------------------------------------------- |
| `STRDT("iiii", <http://example/romanNumeral>)` | "iiii"^^<http://example/romanNumeral>             |

##### 17.4.2.11 STRLANG

```
literal  STRLANG(simple literal lexicalForm, simple literal langTag)
```

该`STRLANG`函数使用参数指定的词法形式和<u>语言</u>标记构造文本。

| `STRLANG("chat", "en")` | “猫” |
| ----------------------- | ---- |
|                         |      |

##### 17.4.2.12 UUID

```
iri  UUID()
```

从UUID URN方案返回一个新的IRI 。每次调用都会`UUID()` 返回不同的UUID。它不能是“ nil” UUID（全零）。UUID的变体和版本取决于实现。

| `UUID()` | `<urn:uuid:b9302fb5-642e-4d3b-af19-29a8f6d894c9>` |
| -------- | ------------------------------------------------- |
|          |                                                   |

##### 17.4.2.13 STRUUID

```
simple literal  STRUUID()
```

返回一个字符串，该字符串是UUID的特定于方案的部分。也就是说，作为简单文字，是生成UUID，转换为简单文字并删除initial`urn:uuid:`的结果。

| `STRUUID()` | `"73cd4307-8a99-4691-a608-b5bda64fb6c1"` |
| ----------- | ---------------------------------------- |
|             |                                          |

#### 17.4.3 字符串函数

##### 17.4.3.1 SPARQL函数中的字符串

###### 17.4.3.1.1字符串参数

某些函数（例如 REGEX， STRLEN，CONTAINS）将一个`string literal`作为参数，并接受简单文字，带语言标签的纯文字或数据类型为xsd: string的文字。然后，它们以文字的词汇形式起作用。

为此，在函数描述中使用了该术语`string literal`。使用任何其他RDF术语将导致对该函数的调用引发错误。

###### 17.4.3.1.2参数兼容性规则

函数 STRSTARTS， STRENDS， CONTAINS， STRBEFORE和 STRAFTER具有两个参数。这些参数必须兼容，否则调用这些函数之一将引发错误。

两个参数的兼容性定义为：

- 参数是简单文字或xsd：string的类型文字
- 参数是带有<u>相同</u>语言标签的普通文字
- 第一个参数是带有语言标签的普通文字，第二个参数是简单文字或类型为xsd：string的文字

|     Argument1     |    Argument2    | Compatible? |
| :---------------: | :-------------: | :---------: |
|       "abc"       |       "b"       |     yes     |
|       "abc"       | "b"^^xsd:string |     yes     |
| "abc"^^xsd:string |       "b"       |     yes     |
| "abc"^^xsd:string | "b"^^xsd:string |     yes     |
|     "abc"@en      |       "b"       |     yes     |
|     "abc"@en      | "b"^^xsd:string |     yes     |
|     "abc"@en      |     "b"@en      |     yes     |
|     "abc"@fr      |     "b"@ja      |     no      |
|       "abc"       |     "b"@ja      |     no      |
|       "abc"       |     "b"@en      |     no      |
| "abc"^^xsd:string |     "b"@en      |     no      |

###### 17.4.3.1.3 字符串文字返回类型

返回字符串文字的函数使用与<u>第一个参数相同类型</u>的字符串文字（简单文字，具有相同语言标记的纯文字，xsd：string）执行此操作。这包括SUBSTR， STRBEFORE和STRAFTER。

函数CONCAT根据其所有参数的详细信息返回字符串文字。

##### 17.4.3.2 STRLEN

```
xsd:integer  STRLEN(string literal str)
```

该`strlen`函数对应于XPath的 fn：string-length 函数，并返回`xsd:integer`等于文字形式的字符长度。

| `strlen("chat")`             | 4    |
| ---------------------------- | ---- |
| `strlen("chat"@en)`          | 4    |
| `strlen("chat"^^xsd:string)` | 4    |

##### 17.4.3.3 SUBSTR

```
string literal  SUBSTR(string literal source, xsd:integer startingLoc)
string literal  SUBSTR(string literal source, xsd:integer startingLoc, xsd:integer length)
```

该`substr`函数与XPath fn：substring函数相对应， 并返回与`source`输入参数相同类型的文字（简单文字，带语言标签的文字， `xsd:string`类型文字），但具有从源词汇形式的子字符串形成的词汇形式。

参数`startingLoc`和`length`可以是xsd：integer的派生类型。

字符串中第一个字符的索引为**1**。

| `substr("foobar", 4)`                | "bar"             |
| ------------------------------------ | ----------------- |
| `substr("foobar"@en, 4)`             | "bar"@en          |
| `substr("foobar"^^xsd:string, 4)`    | "bar"^^xsd:string |
| `substr("foobar", 4, 1)`             | "b"               |
| `substr("foobar"@en, 4, 1)`          | "b"@en            |
| `substr("foobar"^^xsd:string, 4, 1)` | "b"^^xsd:string   |

##### 17.4.3.4 UCASE

```
string literal  UCASE(string literal str)
```

该`UCASE`函数对应于XPath fn：upper-case函数。它返回一个字符串文字，其词汇形式是参数的词汇形式的大写形式。

| `ucase("foo")`             | "FOO"             |
| -------------------------- | ----------------- |
| `ucase("foo"@en)`          | "FOO"@en          |
| `ucase("foo"^^xsd:string)` | "FOO"^^xsd:string |

##### 17.4.3.5 LCASE

```
string literal  LCASE(string literal str)
```

该`LCASE`函数对应于XPath fn：lower-case函数。它返回一个字符串文字，其词汇形式是参数的词汇形式的小写形式。

| `lcase("BAR")`             | "bar"             |
| -------------------------- | ----------------- |
| `lcase("BAR"@en)`          | "bar"@en          |
| `lcase("BAR"^^xsd:string)` | "bar"^^xsd:string |

##### 17.4.3.6 STRSTARTS

```
xsd:boolean  STRSTARTS(string literal arg1, string literal arg2)
```

该`STRSTARTS`函数对应于XPath fn：starts-with函数。参数必须与参数兼容， 否则会引发错误。

对于这样的输入对，如果词汇`arg1` 的形式以 `arg2`的形式开始，则该函数返回true，否则返回false。

| `strStarts("foobar", "foo")`                         | true |
| ---------------------------------------------------- | ---- |
| `strStarts("foobar"@en, "foo"@en)`                   | true |
| `strStarts("foobar"^^xsd:string, "foo"^^xsd:string)` | true |
| `strStarts("foobar"^^xsd:string, "foo")`             | true |
| `strStarts("foobar", "foo"^^xsd:string)`             | true |
| `strStarts("foobar"@en, "foo")`                      | true |
| `strStarts("foobar"@en, "foo"^^xsd:string)`          | true |

##### 17.4.3.7 STRENDS

```
xsd:boolean  STRENDS(string literal arg1, string literal arg2)
```

该`STRENDS`函数对应于XPath fn：ends-with函数。参数必须与参数兼容， 否则会引发错误。

对于此类输入对，如果词法形式`arg1` 的结尾是`arg2`，则该函数返回true ，否则返回false。

| `strEnds("foobar", "bar")`                         | true |
| -------------------------------------------------- | ---- |
| `strEnds("foobar"@en, "bar"@en)`                   | true |
| `strEnds("foobar"^^xsd:string, "bar"^^xsd:string)` | true |
| `strEnds("foobar"^^xsd:string, "bar")`             | true |
| `strEnds("foobar", "bar"^^xsd:string)`             | true |
| `strEnds("foobar"@en, "bar")`                      | true |
| `strEnds("foobar"@en, "bar"^^xsd:string)`          | true |

##### 17.4.3.8 CONTAINS

```
xsd:boolean  CONTAINS(string literal arg1, string literal arg2)
```

该`CONTAINS`函数对应于XPath fn：contains。参数必须与参数兼容， 否则会引发错误。

| `contains("foobar", "bar")`                         | true |
| --------------------------------------------------- | ---- |
| `contains("foobar"@en, "foo"@en)`                   | true |
| `contains("foobar"^^xsd:string, "bar"^^xsd:string)` | true |
| `contains("foobar"^^xsd:string, "foo")`             | true |
| `contains("foobar", "bar"^^xsd:string)`             | true |
| `contains("foobar"@en, "foo")`                      | true |
| `contains("foobar"@en, "bar"^^xsd:string)`          | true |

##### 17.4.3.9 STRBEFORE

```
literal  STRBEFORE(string literal arg1, string literal arg2)
```

该`STRBEFORE`函数对应于XPath fn：substring-before函数。参数必须与参数兼容， 否则会引发错误。

对于兼容的参数，如果第二个参数的词法部分作为第一个参数的词法部分的子字符串出现，则该函数将返回与第一个参数 `arg1`相同的文字（简单文字，纯文字相同语言标记，xsd：string）。结果的词法形式是`arg1` 的词法形式的子串，该子串`arg1` 先于`arg2`的词法形式出现。如果`arg2`的词法形式为 空字符串，则将其视为匹配项，结果的词法形式为空字符串。

如果没有这种情况，则返回一个空的简单文字。

| strbefore("abc","b")            | "a"            |
| ------------------------------- | -------------- |
| strbefore("abc"@en,"bc")        | "a"@en         |
| strbefore("abc"@en,"b"@cy)      | error          |
| strbefore("abc"^^xsd:string,"") | ""^^xsd:string |
| strbefore("abc","xyz")          | ""             |
| strbefore("abc"@en, "z"@en)     | ""             |
| strbefore("abc"@en, "z")        | ""             |
| strbefore("abc"@en, ""@en)      | ""@en          |
| strbefore("abc"@en, "")         | ""@en          |

##### 17.4.3.10 STRAFTER

```
literal  STRAFTER(string literal arg1, string literal arg2)
```

该`STRAFTER`函数对应于XPath fn：substring-after函数。参数必须与参数兼容， 否则会引发错误。

对于兼容的参数，如果第二个参数的词法部分作为第一个参数的词法部分的子字符串出现，则该函数将返回与第一个参数 `arg1`相同的文字（简单文字，纯文字相同语言标记，xsd：string）。结果的词法形式是`arg1` 的词法形式的子串，该子串`arg1` 跟随的词法形式`arg2`的第一次出现。如果`arg2` 的词法形式为空字符串，则将其视为匹配项，结果的词法形式为的词法形式`arg1`。

如果没有这种情况，则返回一个空的简单文字。

| strafter("abc","b")            | "c"               |
| ------------------------------ | ----------------- |
| strafter("abc"@en,"ab")        | "c"@en            |
| strafter("abc"@en,"b"@cy)      | error             |
| strafter("abc"^^xsd:string,"") | "abc"^^xsd:string |
| strafter("abc","xyz")          | ""                |
| strafter("abc"@en, "z"@en)     | ""                |
| strafter("abc"@en, "z")        | ""                |
| strafter("abc"@en, ""@en)      | "abc"@en          |
| strafter("abc"@en, "")         | "abc"@en          |

##### 17.4.3.11 ENCODE_FOR_URI

```
simple literal  ENCODE_FOR_URI(string literal ltrl)
```

该`ENCODE_FOR_URI`函数对应于XPath fn：encode-for-uri函数。根据fn：encode-for-uri函数转换保留字符后，它将返回具有从其输入的词法形式获得的简单文本 。

| `encode_for_uri("Los Angeles")`             | `"Los%20Angeles"` |
| ------------------------------------------- | ----------------- |
| `encode_for_uri("Los Angeles"@en)`          | `"Los%20Angeles"` |
| `encode_for_uri("Los Angeles"^^xsd:string)` | `"Los%20Angeles"` |

##### 17.4.3.12 CONCAT

```
string literal  CONCAT(string literal ltrl1 ... string literal ltrln)
```

该`CONCAT`函数对应于XPath fn：concat函数。该函数接受字符串文字作为参数。

返回的文字的词法形式是通过串联其输入的词法形式而获得的。如果所有输入文字都是类型文字`xsd:string`，则返回的文字也是`xsd:string`，如果所有输入文字是具有相同语言标签的纯文字，则返回的文字是具有相同语言标签的纯文字，在所有其他情况下，返回的文字是一个简单的文字。

| `concat("foo", "bar")`                         | "foobar"             |
| ---------------------------------------------- | -------------------- |
| `concat("foo"@en, "bar"@en)`                   | "foobar"@en          |
| `concat("foo"^^xsd:string, "bar"^^xsd:string)` | "foobar"^^xsd:string |
| `concat("foo", "bar"^^xsd:string)`             | "foobar"             |
| `concat("foo"@en, "bar")`                      | "foobar"             |
| `concat("foo"@en, "bar"^^xsd:string)`          | "foobar"             |

##### 17.4.3.13 langMatches

```
 xsd:boolean  langMatches (simple literal language-tag, simple literal language-range)
```

如果第一个参数`language-tag`在每个基本过滤模式中都和第二个参数`language-range`匹配，则返回ture。`language-range`是一个基本语言， "*" 的`language-range`匹配任何非空`language-tag`字符串。

```
@prefix dc:       <http://purl.org/dc/elements/1.1/> .

_:a  dc:title         "That Seventies Show"@en .
_:a  dc:title         "Cette Série des Années Soixante-dix"@fr .
_:a  dc:title         "Cette Série des Années Septante"@fr-BE .
_:b  dc:title         "Il Buono, il Bruto, il Cattivo" .
```

该查询使用 `langMatches`和 `lang` 查找该节目的法语标题，该节目的英语名称为“ That Seventies Show”：

```
PREFIX dc: <http://purl.org/dc/elements/1.1/>
SELECT ?title
 WHERE { ?x dc:title  "That Seventies Show"@en ;
            dc:title  ?title .
         FILTER langMatches( lang(?title), "FR" ) }
```

**结果**

|                  title                   |
| :--------------------------------------: |
| "Cette Série des Années Soixante-dix"@fr |
| "Cette Série des Années Septante"@fr-BE  |

`langMatches( lang( ?v ), "*" )`将不匹配没有语言标记的文字，因为`lang( ?v )`它将返回一个空字符串，因此

```
PREFIX dc: <http://purl.org/dc/elements/1.1/>
SELECT ?title
 WHERE { ?x dc:title  ?title .
         FILTER langMatches( lang(?title), "*" ) }
```

将返回所有带有语言标签的标题：

|                  title                   |
| :--------------------------------------: |
|         "That Seventies Show"@en         |
| "Cette Série des Années Soixante-dix"@fr |
| "Cette Série des Années Septante"@fr-BE  |

##### 17.4.3.14 REGEX

```
 xsd:boolean  REGEX (string literal text, simple literal pattern)
 xsd:boolean  REGEX (string literal text, simple literal pattern, simple literal flags)
```

调用XPath fn：matches函数以与`text`正则表达式进行匹配`pattern`。

```
@prefix foaf:       <http://xmlns.com/foaf/0.1/> .

_:a  foaf:name       "Alice".
_:b  foaf:name       "Bob" .
```

```
PREFIX foaf: <http://xmlns.com/foaf/0.1/>
SELECT ?name
 WHERE { ?x foaf:name  ?name
         FILTER regex(?name, "^ali", "i") }
```

**结果**

| name    |
| ------- |
| "Alice" |

##### 17.4.3.15 REPLACE

```
 string literal  REPLACE (string literal arg, simple literal pattern, simple literal replacement )
 string literal  REPLACE (string literal arg, simple literal pattern, simple literal replacement,  simple literal flags)
```

该`REPLACE`函数对应于XPath fn：replace函数。它用替换字符串替换每个不重复出现的正则表达式pattern。

| replace("abcd", "b", "Z")      | "aZcd" |
| ------------------------------ | ------ |
| replace("abab", "B", "Z","i")  | "aZaZ" |
| replace("abab", "B.", "Z","i") | "aZb"  |

#### 17.4.4 数字函数

##### 17.4.4.1 abs

```
 numeric  ABS (numeric term)
```

返回的绝对值`arg`。如果`arg`不是数字值，则会引发错误。

对于XDM数据类型的术语，此函数与fn：numeric-abs相同 。

| `abs(1)`    | `1`   |
| ----------- | ----- |
| `abs(-1.5)` | `1.5` |

##### 17.4.4.2 round

```
 numeric  ROUND (numeric term)
```

返回最接近参数的没有小数部分的数字。如果有两个这样的数字，则返回最接近正无穷大的数字。如果`arg`不是数字值，则会引发错误。

| `round(2.4999)` | `2.0`  |
| --------------- | ------ |
| `round(2.5)`    | `3.0`  |
| `round(-2.5)`   | `-2.0` |

##### 17.4.4.3 ceil

```
 numeric  CEIL (numeric term)
```

返回最小的数字（最接近负无穷大），且不包含不小于`arg`的值的小数部分。如果`arg`不是数字值，则会引发错误。

| `ceil(10.5)`  | `11.0`  |
| ------------- | ------- |
| `ceil(-10.5)` | `-10.0` |

##### 17.4.4.4 floor

```
 numeric  FLOOR (numeric term)
```

返回最大（最接近正无穷大）的数字，且不包含不大于`arg`的值的小数部分。如果`arg`不是数字值，则会引发错误。

| `floor(10.5)`  | `10.0`  |
| -------------- | ------- |
| `floor(-10.5)` | `-11.0` |

##### 17.4.4.5 RAND

```
 xsd:double  RAND ( )
```

返回介于0（含）和1.0e0（不含）之间的伪随机数。每次调用此函数时，都会产生不同的数字。数字的产生概率应大致相等。

| `rand()` | `"0.31221030831984886"^^xsd:double` |
| -------- | ----------------------------------- |
|          |                                     |

#### 17.4.5 日期和时间函数

##### 17.4.5.1 now

```
 xsd:dateTime  NOW ()
```

返回当前查询执行的XSD dateTime值。在任何一个查询执行中，对此函数的所有调用都必须返回相同的值。未指定返回的确切时刻。

| `now()` | `"2011-01-10T14:45:13.815-05:00"^^xsd:dateTime` |
| ------- | ----------------------------------------------- |
|         |                                                 |

##### 17.4.5.2 year

```
 xsd:integer  YEAR (xsd:dateTime arg)
```

返回`arg`整数的年份部分。

| `year("2011-01-10T14:45:13.815-05:00"^^xsd:dateTime)` | `2011` |
| ----------------------------------------------------- | ------ |
|                                                       |        |

##### 17.4.5.3 month

```
 xsd:integer  MONTH (xsd:dateTime arg)
```

以整数形式返回`arg`的月份部分。

| `month("2011-01-10T14:45:13.815-05:00"^^xsd:dateTime)` | `1`  |
| ------------------------------------------------------ | ---- |
|                                                        |      |

##### 17.4.5.4 day

```
 xsd:integer  DAY (xsd:dateTime arg)
```

以整数形式返回`arg`的日期部分。

| `day("2011-01-10T14:45:13.815-05:00"^^xsd:dateTime)` | `10` |
| ---------------------------------------------------- | ---- |
|                                                      |      |

##### 17.4.5.5 hours

```
 xsd:integer  HOURS (xsd:dateTime arg)
```

以整数形式返回`arg`的小时部分。该值以XSD dateTime的词汇形式给出。

| `hours("2011-01-10T14:45:13.815-05:00"^^xsd:dateTime)` | `14` |
| ------------------------------------------------------ | ---- |
|                                                        |      |

##### 17.4.5.6 minutes

```
 xsd:integer  MINUTES (xsd:dateTime arg)
```

返回的词汇形式`arg`的分钟部分。该值以XSD dateTime的词汇形式给出。

| `minutes("2011-01-10T14:45:13.815-05:00"^^xsd:dateTime)` | `45` |
| -------------------------------------------------------- | ---- |
|                                                          |      |

##### 17.4.5.7 seconds

```
 xsd:decimal  SECONDS (xsd:dateTime arg)
```

返回的词汇形式`arg`的秒部分。

| `seconds("2011-01-10T14:45:13.815-05:00"^^xsd:dateTime)` | `13.815` |
| -------------------------------------------------------- | -------- |
|                                                          |          |

##### 17.4.5.8 timezone

```
 xsd：dayTimeDuration   TIMEZONE（xsd：dateTime  arg）
```

以xsd: dayTimeDuration的形式返回`arg`时区部分。如果没有时区，则会引发错误。

| `timezone("2011-01-10T14:45:13.815-05:00"^^xsd:dateTime)` | `"-PT5H"^^xsd:dayTimeDuration` |
| --------------------------------------------------------- | ------------------------------ |
| `timezone("2011-01-10T14:45:13.815Z"^^xsd:dateTime)`      | `"PT0S"^^xsd:dayTimeDuration`  |
| `timezone("2011-01-10T14:45:13.815"^^xsd:dateTime)`       | error                          |

##### 17.4.5.9 tz

```
 simple literal  TZ (xsd:dateTime arg)
```

以简单文字的形式返回`arg`的时区部分。如果没有时区，则返回空字符串。

| `tz("2011-01-10T14:45:13.815-05:00"^^xsd:dateTime)` | `"-05:00"` |
| --------------------------------------------------- | ---------- |
| `tz("2011-01-10T14:45:13.815Z"^^xsd:dateTime)`      | `"Z"`      |
| `tz("2011-01-10T14:45:13.815"^^xsd:dateTime)`       | `""`       |

#### 17.4.6 哈希函数

##### 17.4.6.1 MD5

```
 simple literal  MD5 (simple literal arg)
 simple literal  MD5 (xsd:string arg)
```

以十六进制数字字符串形式返回MD5校验和，该校验和是基于的简单文字或词汇形式`xsd:string`的UTF-8表示形式计算得出的 。十六进制数字*应*小写。

| `MD5("abc")`             | `"900150983cd24fb0d6963f7d28e17f72"` |
| ------------------------ | ------------------------------------ |
| `MD5("abc"^^xsd:string)` | `"900150983cd24fb0d6963f7d28e17f72"` |

##### 17.4.6.2 SHA1

```
 simple literal  SHA1 (simple literal arg)
 simple literal  SHA1 (xsd:string arg)
```

以十六进制数字字符串形式返回SHA1校验和，以简单文字或词汇形式`xsd:string`的UTF-8表示形式计算得出 。十六进制数字*应*小写。

| `SHA1("abc")`             | `"a9993e364706816aba3e25717850c26c9cd0d89d"` |
| ------------------------- | -------------------------------------------- |
| `SHA1("abc"^^xsd:string)` | `"a9993e364706816aba3e25717850c26c9cd0d89d"` |

##### 17.4.6.3 SHA256

```
 simple literal  SHA256 (simple literal arg)
 simple literal  SHA256 (xsd:string arg)
```

以十六进制数字字符串形式返回SHA256校验和，以简单文字或词汇形式`xsd:string`的UTF-8表示形式计算得出。十六进制数字*应*小写。

##### 17.4.6.4 SHA384

```
 simple literal  SHA384 (simple literal arg)
 simple literal  SHA384 (xsd:string arg)
```

以十六进制数字字符串形式返回SHA384校验和，该校验和是基于的简单文字或词汇形式`xsd:string`的UTF-8表示形式计算得出的 。十六进制数字*应*小写。

##### 17.4.6.5 SHA512

```
 simple literal  SHA512 (simple literal arg)
 simple literal  SHA512 (xsd:string arg)
```

以十六进制数字字符串的形式返回SHA512校验和，以十基于的简单文字或词汇形式`xsd:string`的UTF-8表示形式计算得出 `xsd:string`。十六进制数字*应*小写。

### 17.5 XPath构造函数

SPARQL导入在第17.1节“从原始类型到原始类型的转换”中定义的XQuery 1.0和XPath 2.0函数和运算符[ FUNCOP ]中定义的XPath构造函数的子集。SPARQL构造函数包括SPARQL操作数数据类型的所有XPath构造函数，以及RDF数据模型施加的其他数据类型。通过在源类型的操作数上为目标类型调用构造函数来执行SPARQL中的强制转换。

XPath仅定义从一种XML Schema数据类型到另一种的数据类型转换。其余的定义如下：

- 将IRI转换为`xsd:string`会产生一个类型文字，该文字具有包含IRI的代码点的词法值，并且数据类型为`xsd:string`。
- 将简单文字转换为任何XML Schema数据类型的定义为，将一个`xsd:string`与字符串值转换为目标文字类型的文字值的乘积。

下表总结了始终允许（Y），绝对不允许（N）且取决于词法值（M）的转换操作。例如，从`xsd:string`（第一行）到`xsd:float`（第二列）的转换操作取决于词法值（M）。

bool = xsd:boolean
dbl = xsd:double
flt = xsd:float
dec = xsd:decimal
int = xsd:integer
dT = xsd:dateTime
str = xsd:string
IRI = IRI
ltrl = `simple literal`

| From \ To | str  | flt  | dbl  | dec  | int  | dT   | bool |
| --------- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| str       | Y    | M    | M    | M    | M    | M    | M    |
| flt       | Y    | Y    | Y    | M    | M    | N    | Y    |
| dbl       | Y    | Y    | Y    | M    | M    | N    | Y    |
| dec       | Y    | Y    | Y    | Y    | Y    | N    | Y    |
| int       | Y    | Y    | Y    | Y    | Y    | N    | Y    |
| dT        | Y    | N    | N    | N    | N    | Y    | N    |
| bool      | Y    | Y    | Y    | Y    | Y    | N    | Y    |
| IRI       | Y    | N    | N    | N    | N    | N    | N    |
| ltrl      | Y    | M    | M    | M    | M    | M    | M    |

### 17.6 可扩展的值测试

应该注意的是，指定在某些情况下返回错误的任何函数或运算符都是有效的扩展点。也就是说，在这些错误情况下，实现可能会返回非错误值，并且仍然符合此建议。

一个PrimaryExpression语法规则可以是由一个名为IRI的扩展函数的调用。扩展函数将一些RDF术语作为参数，并返回RDF术语。这些功能的语义由识别功能的IRI识别。

使用扩展功能的SPARQL查询可能具有有限的互操作性。

例如，考虑一个名为的函数`func:even`：

```
 xsd:boolean   func:even (numeric value)
```

该函数将在FILTER中这样调用：

```
PREFIX foaf: <http://xmlns.com/foaf/0.1/>
PREFIX func: <http://example.org/functions#>
SELECT ?name ?id
WHERE { ?x foaf:name  ?name ;
           func:empId   ?id .
        FILTER (func:even(?id)) }
```

对于第二个示例，请考虑一个`aGeo:distance`计算两点之间距离的函数，该函数在这里用于查找格勒诺布尔附近的地点：

```
 xsd:double   aGeo:distance (numeric x1, numeric y1, numeric x2, numeric y2)
PREFIX aGeo: <http://example.org/geo#>

SELECT ?neighbor
WHERE { ?a aGeo:placeName "Grenoble" .
        ?a aGeo:locationX ?axLoc .
        ?a aGeo:locationY ?ayLoc .

        ?b aGeo:placeName ?neighbor .
        ?b aGeo:locationX ?bxLoc .
        ?b aGeo:locationY ?byLoc .

        FILTER ( aGeo:distance(?axLoc, ?ayLoc, ?bxLoc, ?byLoc) < 10 ) .
      }
```

扩展功能可能用于测试核心SPARQL规范不支持的某些应用程序数据类型，它可能是数据类型格式之间的转换，例如从另一种日期格式转换为XSD dateTime RDF术语。