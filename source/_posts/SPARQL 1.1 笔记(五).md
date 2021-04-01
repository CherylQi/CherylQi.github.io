---
title: SPARQL 1.1 笔记(五)
date: 2021-03-21 16:29:11
tags: 语义Web
---

## 19 SPARQL语法

SPARQL语法涵盖SPARQL查询和 SPARQL更新。

### 19.1 SPARQL请求字符串

一个SPARQL请求字符串是一个SPARQL查询字符串或SPARQL更新字符串并且它是Unicode字符的字符串。

为了与将来的Unicode版本兼容，此字符串中的字符可能包含截至本出版物发行之日尚未分配的Unicode代码点。对于具有排除字符类的产品（例如`[^<>'{}|^`])，字符从range `#x0 - #x10FFFF`中排除。

<!--more-->

### 19.2 码点转义序列

在解析下面的EBNF中定义的语法之前，将对SPARQL查询字符串的代码点转义序列进行处理。SPARQL查询字符串的代码点转义序列为：

| Escape                                                       | Unicode code point                                           |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| '\u' [HEX](https://www.w3.org/TR/sparql11-query/#HEX) [HEX](https://www.w3.org/TR/sparql11-query/#HEX) [HEX](https://www.w3.org/TR/sparql11-query/#HEX) [HEX](https://www.w3.org/TR/sparql11-query/#HEX) | A Unicode code point in the range U+0 to U+FFFF inclusive corresponding to the encoded hexadecimal value. |
| '\U' [HEX](https://www.w3.org/TR/sparql11-query/#HEX) [HEX](https://www.w3.org/TR/sparql11-query/#HEX) [HEX](https://www.w3.org/TR/sparql11-query/#HEX) [HEX](https://www.w3.org/TR/sparql11-query/#HEX) [HEX](https://www.w3.org/TR/sparql11-query/#HEX) [HEX](https://www.w3.org/TR/sparql11-query/#HEX) [HEX](https://www.w3.org/TR/sparql11-query/#HEX) [HEX](https://www.w3.org/TR/sparql11-query/#HEX) | A Unicode code point in the range U+0 to U+10FFFF inclusive corresponding to the encoded hexadecimal value. |

其中HEX是十六进制字符 HEX ::= [0-9] | [A-F] | [a-f]

**示例**

```
<ab\u00E9xy>        # Codepoint 00E9 is Latin small e with acute - é
\u03B1:a            # Codepoint x03B1 is Greek small alpha - α
a\u003Ab            # a:b -- codepoint x3A is colon
```

代码点转义序列可以出现在查询字符串中的任何位置。它们是在基于语法规则进行解析之前进行处理的，因此可以用语法中有意义的代码点代替，例如“ `:`”标记前缀名称。

这些转义序列未包含在下面的语法中。仅给出在该点上合法的字符的转义序列。例如，变量“ `?x\u0020y`”是非法的（`\u0020` 是空格，并且在变量名中是不允许的）。

### 19.3 空格

空格（产生`WS`）用于分隔两个终端，否则将（误）识别为一个终端。下方用大写字母表示的规则名称表示空格在哪里；这些构成了构建SPARQL解析器的终端的可能选择。字符串中的空格很重要。否则，令牌之间的空格将被忽略。

例如：?a<?b&&?c>?d

是令牌序列变量' `?a`'，一个IRI' `<?b&&?c>`'和变量' `?d`'，而不是包含运算符`&&`使用' `<`'（小于）和' `>`'（大于）将两个表达式连接起来的表达式。

### 19.4 评论

SPARQL查询中的注释在IRI或字符串之外采用`#`形式，如果注释标记后面没有行尾，则继续到行尾（用字符`0x0D` 或标记`0x0A`）或文件末尾。注释被视为空格。

### 19.5 IRI引用

在转义处理之后，由`IRIREF` 生产和`PrefixedName`（在前缀扩展之后）生产匹配的文本必须符合RFC 3987“用于IRI参考和IRI的ABNF”中2.2节中IRI参考的通用语法。例如， `IRIREF` `<abc#def>`可能出现在SPARQL查询字符串中，但`IRIREF` `<abc##def>`不能出现在查询字符串中。

使用BASE关键字声明的基本IRI必须是绝对IRI。使用PREFIX关键字声明的前缀不能在同一查询中重新声明。

### 19.6 空白节点和空白节点标签

空白节点不能用于：

- `DELETE WHERE`
- `DELETE DATA`
- `DeleteClause`

在 SPARQL更新请求中。

空白节点标签的作用域为发生它们的 SPARQL请求字符串。请求字符串中相同空白节点标签的不同用法指向相同空白节点。为每个请求生成新的空白节点。在请求之间，标签不能引用空白节点。

相同的空白节点标签不能用于：

- SPARQL查询中的两个基本图形模式
- `WHERE`单个SPARQL更新请求中的两个子句
- `INSERT DATA` 单个SPARQL更新请求中的两个操作

请注意，相同的空白节点标签可以出现在SPARQL更新请求的不同 QuadPattern子句中。

### 19.7 字符串中的转义序列

除了码点转义序列，下面的转义序列的任何`string`产品（例如：` STRING_LITERAL1`` STRING_LITERAL2`` STRING_LITERAL_LONG1`` STRING_LITERAL_LONG2`）

| Escape | Unicode code point                           |
| ------ | -------------------------------------------- |
| '\t'   | U+0009 (tab)                                 |
| '\n'   | U+000A (line feed)                           |
| '\r'   | U+000D (carriage return)                     |
| '\b'   | U+0008 (backspace)                           |
| '\f'   | U+000C (form feed)                           |
| '\"'   | U+0022 (quotation mark, double quote mark)   |
| "\'"   | U+0027 (apostrophe-quote, single quote mark) |
| '\\'   | U+005C (backslash)                           |

**示例**

```
"abc\n"
"xy\rz"
'xy\tz'
```

### 19.8语法

语法中使用的EBNF表示法是在可扩展标记语言（XML）1.1 [ XML11 ]第6节表示法中定义的。

笔记：

1. 关键字以不区分大小写的方式进行匹配，但关键字' `a`'与Turtle和N3一致，用于代替IRI `rdf:type`（全部为 `http://www.w3.org/1999/02/22-rdf-syntax-ns#type`），但不区分大小写。
2. 转义序列区分大小写。
3. 在标记输入并选择语法规则时，将选择最长的匹配项。
4. 当具有大写名称的规则用作终端时，SPARQL语法为LL（1）。
5. 语法有两个入口点：`QueryUnit`用于SPARQL查询和`UpdateUnit`用于SPARQL更新请求。
6. 在带符号的数字中，符号和数字之间不允许有空格。该`AdditiveExpression`语法规则允许这种通过覆盖的表达，接着有符号数的两种情况。这些将根据需要对无符号数字进行加法或减法。
7. 标记允许单词之间有任意数量的空格。为了清楚起见，文法中使用了单个空格版本。 `INSERT DATA, DELETE DATA, DELETE WHERE`
8. 该 `QuadData`和 `QuadPattern` 规则都使用规则`Quads`。`QuadData`在`INSERT DATA` 和中 使用 的规则`DELETE DATA`必须不允许在四边形模式中使用变量。
9. `DELETE WHERE`的 `DeleteClause` for`DELETE`或in 不允许使用空白节点语法 `DELETE DATA`。
10. 第19.6节 中给出了限制使用空白节点标签的规则。
11. `VALUES`块 的变量列表中的变量数量 必须与中的每个关联值列表的数量相同 `DataBlock`。
12. 通过引入变量`AS`一个在`SELECT`子句不能已经在范围内。
13. 一个在所分配的变量`BIND`子句不能是已经在使用中的紧接在前 `TriplesBlock`内 `GroupGraphPattern`。
14. 聚合函数可以是聚合的内置关键字之一，也可以是自定义聚合（在语法上是函数调用）。聚合函数只能在SELECT， HAVING和 ORDER BY子句中使用。
15. 仅自定义聚合函数在函数调用中使用`DISTINCT`关键字。

| `[1] `   | `QueryUnit`                 | ::=  | `Query`                                                      |
| -------- | --------------------------- | ---- | ------------------------------------------------------------ |
| `[2] `   | `Query`                     | ::=  | `Prologue( SelectQuery | ConstructQuery | DescribeQuery | AskQuery )ValuesClause` |
| `[3] `   | `UpdateUnit`                | ::=  | `Update`                                                     |
| `[4] `   | `Prologue`                  | ::=  | `( BaseDecl | PrefixDecl )*`                                 |
| `[5] `   | `BaseDecl`                  | ::=  | `'BASE' IRIREF`                                              |
| `[6] `   | `PrefixDecl`                | ::=  | `'PREFIX' PNAME_NS IRIREF`                                   |
| `[7] `   | `SelectQuery`               | ::=  | `SelectClause DatasetClause* WhereClause SolutionModifier`   |
| `[8] `   | `SubSelect`                 | ::=  | `SelectClause WhereClause SolutionModifier ValuesClause`     |
| `[9] `   | `SelectClause`              | ::=  | `'SELECT' ( 'DISTINCT' | 'REDUCED' )? ( ( Var | ( '(' Expression 'AS' Var ')' ) )+ | '*' )` |
| `[10] `  | `ConstructQuery`            | ::=  | `'CONSTRUCT' ( ConstructTemplate DatasetClause* WhereClause SolutionModifier | DatasetClause* 'WHERE' '{' TriplesTemplate? '}' SolutionModifier )` |
| `[11] `  | `DescribeQuery`             | ::=  | `'DESCRIBE' ( VarOrIri+ | '*' ) DatasetClause* WhereClause? SolutionModifier` |
| `[12] `  | `AskQuery`                  | ::=  | `'ASK' DatasetClause* WhereClause SolutionModifier`          |
| `[13] `  | `DatasetClause`             | ::=  | `'FROM' ( DefaultGraphClause | NamedGraphClause )`           |
| `[14] `  | `DefaultGraphClause`        | ::=  | `SourceSelector`                                             |
| `[15] `  | `NamedGraphClause`          | ::=  | `'NAMED' SourceSelector`                                     |
| `[16] `  | `SourceSelector`            | ::=  | `iri`                                                        |
| `[17] `  | `WhereClause`               | ::=  | `'WHERE'? GroupGraphPattern`                                 |
| `[18] `  | `SolutionModifier`          | ::=  | `GroupClause? HavingClause? OrderClause? LimitOffsetClauses?` |
| `[19] `  | `GroupClause`               | ::=  | `'GROUP' 'BY' GroupCondition+`                               |
| `[20] `  | `GroupCondition`            | ::=  | `BuiltInCall | FunctionCall | '(' Expression ( 'AS' Var )? ')' | Var` |
| `[21] `  | `HavingClause`              | ::=  | `'HAVING' HavingCondition+`                                  |
| `[22] `  | `HavingCondition`           | ::=  | `Constraint`                                                 |
| `[23] `  | `OrderClause`               | ::=  | `'ORDER' 'BY' OrderCondition+`                               |
| `[24] `  | `OrderCondition`            | ::=  | `( ( 'ASC' | 'DESC' ) BrackettedExpression )| ( Constraint | Var )` |
| `[25] `  | `LimitOffsetClauses`        | ::=  | `LimitClause OffsetClause? | OffsetClause LimitClause?`      |
| `[26] `  | `LimitClause`               | ::=  | `'LIMIT' INTEGER`                                            |
| `[27] `  | `OffsetClause`              | ::=  | `'OFFSET' INTEGER`                                           |
| `[28] `  | `ValuesClause`              | ::=  | `( 'VALUES' DataBlock )?`                                    |
| `[29] `  | `Update`                    | ::=  | `Prologue ( Update1 ( ';' Update )? )?`                      |
| `[30] `  | `Update1`                   | ::=  | `Load | Clear | Drop | Add | Move | Copy | Create | InsertData | DeleteData | DeleteWhere | Modify` |
| `[31] `  | `Load`                      | ::=  | `'LOAD' 'SILENT'? iri ( 'INTO' GraphRef )?`                  |
| `[32] `  | `Clear`                     | ::=  | `'CLEAR' 'SILENT'? GraphRefAll`                              |
| `[33] `  | `Drop`                      | ::=  | `'DROP' 'SILENT'? GraphRefAll`                               |
| `[34] `  | `Create`                    | ::=  | `'CREATE' 'SILENT'? GraphRef`                                |
| `[35] `  | `Add`                       | ::=  | `'ADD' 'SILENT'? GraphOrDefault 'TO' GraphOrDefault`         |
| `[36] `  | `Move`                      | ::=  | `'MOVE' 'SILENT'? GraphOrDefault 'TO' GraphOrDefault`        |
| `[37] `  | `Copy`                      | ::=  | `'COPY' 'SILENT'? GraphOrDefault 'TO' GraphOrDefault`        |
| `[38] `  | `InsertData`                | ::=  | `'INSERT DATA' QuadData`                                     |
| `[39] `  | `DeleteData`                | ::=  | `'DELETE DATA' QuadData`                                     |
| `[40] `  | `DeleteWhere`               | ::=  | `'DELETE WHERE' QuadPattern`                                 |
| `[41] `  | `Modify`                    | ::=  | `( 'WITH' iri )? ( DeleteClause InsertClause? | InsertClause ) UsingClause* 'WHERE' GroupGraphPattern` |
| `[42] `  | `DeleteClause`              | ::=  | `'DELETE' QuadPattern`                                       |
| `[43] `  | `InsertClause`              | ::=  | `'INSERT' QuadPattern`                                       |
| `[44] `  | `UsingClause`               | ::=  | `'USING' ( iri | 'NAMED' iri )`                              |
| `[45] `  | `GraphOrDefault`            | ::=  | `'DEFAULT' | 'GRAPH'? iri`                                   |
| `[46] `  | `GraphRef`                  | ::=  | `'GRAPH' iri`                                                |
| `[47] `  | `GraphRefAll`               | ::=  | `GraphRef | 'DEFAULT' | 'NAMED' | 'ALL'`                     |
| `[48] `  | `QuadPattern`               | ::=  | `'{' Quads '}'`                                              |
| `[49] `  | `QuadData`                  | ::=  | `'{' Quads '}'`                                              |
| `[50] `  | `Quads`                     | ::=  | `TriplesTemplate? ( QuadsNotTriples '.'? TriplesTemplate? )*` |
| `[51] `  | `QuadsNotTriples`           | ::=  | `'GRAPH' VarOrIri '{' TriplesTemplate? '}'`                  |
| `[52] `  | `TriplesTemplate`           | ::=  | `TriplesSameSubject ( '.' TriplesTemplate? )?`               |
| `[53] `  | `GroupGraphPattern`         | ::=  | `'{' ( SubSelect | GroupGraphPatternSub ) '}'`               |
| `[54] `  | `GroupGraphPatternSub`      | ::=  | `TriplesBlock? ( GraphPatternNotTriples '.'? TriplesBlock? )*` |
| `[55] `  | `TriplesBlock`              | ::=  | `TriplesSameSubjectPath ( '.' TriplesBlock? )?`              |
| `[56] `  | `GraphPatternNotTriples`    | ::=  | `GroupOrUnionGraphPattern | OptionalGraphPattern | MinusGraphPattern | GraphGraphPattern | ServiceGraphPattern | Filter | Bind | InlineData` |
| `[57] `  | `OptionalGraphPattern`      | ::=  | `'OPTIONAL' GroupGraphPattern`                               |
| `[58] `  | `GraphGraphPattern`         | ::=  | `'GRAPH' VarOrIri GroupGraphPattern`                         |
| `[59] `  | `ServiceGraphPattern`       | ::=  | `'SERVICE' 'SILENT'? VarOrIri GroupGraphPattern`             |
| `[60] `  | `Bind`                      | ::=  | `'BIND' '(' Expression 'AS' Var ')'`                         |
| `[61] `  | `InlineData`                | ::=  | `'VALUES' DataBlock`                                         |
| `[62] `  | `DataBlock`                 | ::=  | `InlineDataOneVar | InlineDataFull`                          |
| `[63] `  | `InlineDataOneVar`          | ::=  | `Var '{' DataBlockValue* '}'`                                |
| `[64] `  | `InlineDataFull`            | ::=  | `( NIL | '(' Var* ')' ) '{' ( '(' DataBlockValue* ')' | NIL )* '}'` |
| `[65] `  | `DataBlockValue`            | ::=  | `iri | RDFLiteral | NumericLiteral | BooleanLiteral | 'UNDEF'` |
| `[66] `  | `MinusGraphPattern`         | ::=  | `'MINUS' GroupGraphPattern`                                  |
| `[67] `  | `GroupOrUnionGraphPattern`  | ::=  | `GroupGraphPattern ( 'UNION' GroupGraphPattern )*`           |
| `[68] `  | `Filter`                    | ::=  | `'FILTER' Constraint`                                        |
| `[69] `  | `Constraint`                | ::=  | `BrackettedExpression | BuiltInCall | FunctionCall`          |
| `[70] `  | `FunctionCall`              | ::=  | `iri ArgList`                                                |
| `[71] `  | `ArgList`                   | ::=  | `NIL | '(' 'DISTINCT'? Expression ( ',' Expression )* ')'`   |
| `[72] `  | `ExpressionList`            | ::=  | `NIL | '(' Expression ( ',' Expression )* ')'`               |
| `[73] `  | `ConstructTemplate`         | ::=  | `'{' ConstructTriples? '}'`                                  |
| `[74] `  | `ConstructTriples`          | ::=  | `TriplesSameSubject ( '.' ConstructTriples? )?`              |
| `[75] `  | `TriplesSameSubject`        | ::=  | `VarOrTerm PropertyListNotEmpty | TriplesNode PropertyList`  |
| `[76] `  | `PropertyList`              | ::=  | `PropertyListNotEmpty?`                                      |
| `[77] `  | `PropertyListNotEmpty`      | ::=  | `Verb ObjectList ( ';' ( Verb ObjectList )? )*`              |
| `[78] `  | `Verb`                      | ::=  | `VarOrIri | 'a'`                                             |
| `[79] `  | `ObjectList`                | ::=  | `Object ( ',' Object )*`                                     |
| `[80] `  | `Object`                    | ::=  | `GraphNode`                                                  |
| `[81] `  | `TriplesSameSubjectPath`    | ::=  | `VarOrTerm PropertyListPathNotEmpty | TriplesNodePath PropertyListPath` |
| `[82] `  | `PropertyListPath`          | ::=  | `PropertyListPathNotEmpty?`                                  |
| `[83] `  | `PropertyListPathNotEmpty`  | ::=  | `( VerbPath | VerbSimple ) ObjectListPath ( ';' ( ( VerbPath | VerbSimple ) ObjectList )? )*` |
| `[84] `  | `VerbPath`                  | ::=  | `Path`                                                       |
| `[85] `  | `VerbSimple`                | ::=  | `Var`                                                        |
| `[86] `  | `ObjectListPath`            | ::=  | `ObjectPath ( ',' ObjectPath )*`                             |
| `[87] `  | `ObjectPath`                | ::=  | `GraphNodePath`                                              |
| `[88] `  | `Path`                      | ::=  | `PathAlternative`                                            |
| `[89] `  | `PathAlternative`           | ::=  | `PathSequence ( '|' PathSequence )*`                         |
| `[90] `  | `PathSequence`              | ::=  | `PathEltOrInverse ( '/' PathEltOrInverse )*`                 |
| `[91] `  | `PathElt`                   | ::=  | `PathPrimary PathMod?`                                       |
| `[92] `  | `PathEltOrInverse`          | ::=  | `PathElt | '^' PathElt`                                      |
| `[93] `  | `PathMod`                   | ::=  | `'?' | '*' | '+'`                                            |
| `[94] `  | `PathPrimary`               | ::=  | `iri | 'a' | '!' PathNegatedPropertySet | '(' Path ')'`      |
| `[95] `  | `PathNegatedPropertySet`    | ::=  | `PathOneInPropertySet | '(' ( PathOneInPropertySet ( '|' PathOneInPropertySet )* )? ')'` |
| `[96] `  | `PathOneInPropertySet`      | ::=  | `iri | 'a' | '^' ( iri | 'a' )`                              |
| `[97] `  | `Integer`                   | ::=  | `INTEGER`                                                    |
| `[98] `  | `TriplesNode`               | ::=  | `Collection | BlankNodePropertyList`                         |
| `[99] `  | `BlankNodePropertyList`     | ::=  | `'[' PropertyListNotEmpty ']'`                               |
| `[100] ` | `TriplesNodePath`           | ::=  | `CollectionPath | BlankNodePropertyListPath`                 |
| `[101] ` | `BlankNodePropertyListPath` | ::=  | `'[' PropertyListPathNotEmpty ']'`                           |
| `[102] ` | `Collection`                | ::=  | `'(' GraphNode+ ')'`                                         |
| `[103] ` | `CollectionPath`            | ::=  | `'(' GraphNodePath+ ')'`                                     |
| `[104] ` | `GraphNode`                 | ::=  | `VarOrTerm | TriplesNode`                                    |
| `[105] ` | `GraphNodePath`             | ::=  | `VarOrTerm | TriplesNodePath`                                |
| `[106] ` | `VarOrTerm`                 | ::=  | `Var | GraphTerm`                                            |
| `[107] ` | `VarOrIri`                  | ::=  | `Var | iri`                                                  |
| `[108] ` | `Var`                       | ::=  | `VAR1 | VAR2`                                                |
| `[109] ` | `GraphTerm`                 | ::=  | `iri | RDFLiteral | NumericLiteral | BooleanLiteral | BlankNode | NIL` |
| `[110] ` | `Expression`                | ::=  | `ConditionalOrExpression`                                    |
| `[111] ` | `ConditionalOrExpression`   | ::=  | `ConditionalAndExpression ( '||' ConditionalAndExpression )*` |
| `[112] ` | `ConditionalAndExpression`  | ::=  | `ValueLogical ( '&&' ValueLogical )*`                        |
| `[113] ` | `ValueLogical`              | ::=  | `RelationalExpression`                                       |
| `[114] ` | `RelationalExpression`      | ::=  | `NumericExpression ( '=' NumericExpression | '!=' NumericExpression | '<' NumericExpression | '>' NumericExpression | '<=' NumericExpression | '>=' NumericExpression | 'IN' ExpressionList | 'NOT' 'IN' ExpressionList )?` |
| `[115] ` | `NumericExpression`         | ::=  | `AdditiveExpression`                                         |
| `[116] ` | `AdditiveExpression`        | ::=  | `MultiplicativeExpression ( '+' MultiplicativeExpression | '-' MultiplicativeExpression | ( NumericLiteralPositive | NumericLiteralNegative ) ( ( '*' UnaryExpression ) | ( '/' UnaryExpression ) )* )*` |
| `[117] ` | `MultiplicativeExpression`  | ::=  | `UnaryExpression ( '*' UnaryExpression | '/' UnaryExpression )*` |
| `[118] ` | `UnaryExpression`           | ::=  | ` '!' PrimaryExpression| '+' PrimaryExpression| '-' PrimaryExpression| PrimaryExpression` |
| `[119] ` | `PrimaryExpression`         | ::=  | `BrackettedExpression | BuiltInCall | iriOrFunction | RDFLiteral | NumericLiteral | BooleanLiteral | Var` |
| `[120] ` | `BrackettedExpression`      | ::=  | `'(' Expression ')'`                                         |
| `[121] ` | `BuiltInCall`               | ::=  | ` Aggregate| 'STR' '(' Expression ')'| 'LANG' '(' Expression ')'| 'LANGMATCHES' '(' Expression ',' Expression ')'| 'DATATYPE' '(' Expression ')'| 'BOUND' '(' Var ')'| 'IRI' '(' Expression ')'| 'URI' '(' Expression ')'| 'BNODE' ( '(' Expression ')' | NIL )| 'RAND' NIL| 'ABS' '(' Expression ')'| 'CEIL' '(' Expression ')'| 'FLOOR' '(' Expression ')'| 'ROUND' '(' Expression ')'| 'CONCAT' ExpressionList| SubstringExpression| 'STRLEN' '(' Expression ')'| StrReplaceExpression| 'UCASE' '(' Expression ')'| 'LCASE' '(' Expression ')'| 'ENCODE_FOR_URI' '(' Expression ')'| 'CONTAINS' '(' Expression ',' Expression ')'| 'STRSTARTS' '(' Expression ',' Expression ')'| 'STRENDS' '(' Expression ',' Expression ')'| 'STRBEFORE' '(' Expression ',' Expression ')'| 'STRAFTER' '(' Expression ',' Expression ')'| 'YEAR' '(' Expression ')'| 'MONTH' '(' Expression ')'| 'DAY' '(' Expression ')'| 'HOURS' '(' Expression ')'| 'MINUTES' '(' Expression ')'| 'SECONDS' '(' Expression ')'| 'TIMEZONE' '(' Expression ')'| 'TZ' '(' Expression ')'| 'NOW' NIL| 'UUID' NIL| 'STRUUID' NIL| 'MD5' '(' Expression ')'| 'SHA1' '(' Expression ')'| 'SHA256' '(' Expression ')'| 'SHA384' '(' Expression ')'| 'SHA512' '(' Expression ')'| 'COALESCE' ExpressionList| 'IF' '(' Expression ',' Expression ',' Expression ')'| 'STRLANG' '(' Expression ',' Expression ')'| 'STRDT' '(' Expression ',' Expression ')'| 'sameTerm' '(' Expression ',' Expression ')'| 'isIRI' '(' Expression ')'| 'isURI' '(' Expression ')'| 'isBLANK' '(' Expression ')'| 'isLITERAL' '(' Expression ')'| 'isNUMERIC' '(' Expression ')'| RegexExpression| ExistsFunc| NotExistsFunc` |
| `[122] ` | `RegexExpression`           | ::=  | `'REGEX' '(' Expression ',' Expression ( ',' Expression )? ')'` |
| `[123] ` | `SubstringExpression`       | ::=  | `'SUBSTR' '(' Expression ',' Expression ( ',' Expression )? ')'` |
| `[124] ` | `StrReplaceExpression`      | ::=  | `'REPLACE' '(' Expression ',' Expression ',' Expression ( ',' Expression )? ')'` |
| `[125] ` | `ExistsFunc`                | ::=  | `'EXISTS' GroupGraphPattern`                                 |
| `[126] ` | `NotExistsFunc`             | ::=  | `'NOT' 'EXISTS' GroupGraphPattern`                           |
| `[127] ` | `Aggregate`                 | ::=  | ` 'COUNT' '(' 'DISTINCT'? ( '*' | Expression ) ')'| 'SUM' '(' 'DISTINCT'? Expression ')'| 'MIN' '(' 'DISTINCT'? Expression ')'| 'MAX' '(' 'DISTINCT'? Expression ')'| 'AVG' '(' 'DISTINCT'? Expression ')'| 'SAMPLE' '(' 'DISTINCT'? Expression ')'| 'GROUP_CONCAT' '(' 'DISTINCT'? Expression ( ';' 'SEPARATOR' '=' String )? ')'` |
| `[128] ` | `iriOrFunction`             | ::=  | `iri ArgList?`                                               |
| `[129] ` | `RDFLiteral`                | ::=  | `String ( LANGTAG | ( '^^' iri ) )?`                         |
| `[130] ` | `NumericLiteral`            | ::=  | `NumericLiteralUnsigned | NumericLiteralPositive | NumericLiteralNegative` |
| `[131] ` | `NumericLiteralUnsigned`    | ::=  | `INTEGER | DECIMAL | DOUBLE`                                 |
| `[132] ` | `NumericLiteralPositive`    | ::=  | `INTEGER_POSITIVE | DECIMAL_POSITIVE | DOUBLE_POSITIVE`      |
| `[133] ` | `NumericLiteralNegative`    | ::=  | `INTEGER_NEGATIVE | DECIMAL_NEGATIVE | DOUBLE_NEGATIVE`      |
| `[134] ` | `BooleanLiteral`            | ::=  | `'true' | 'false'`                                           |
| `[135] ` | `String`                    | ::=  | `STRING_LITERAL1 | STRING_LITERAL2 | STRING_LITERAL_LONG1 | STRING_LITERAL_LONG2` |
| `[136] ` | `iri`                       | ::=  | `IRIREF | PrefixedName`                                      |
| `[137] ` | `PrefixedName`              | ::=  | `PNAME_LN | PNAME_NS`                                        |
| `[138] ` | `BlankNode`                 | ::=  | `BLANK_NODE_LABEL | ANON`                                    |
