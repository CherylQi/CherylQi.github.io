---
title: HTML
date: 2021-02-01 16:25:20
tags: 前端
---

# HTML

HTML，全称Hyper Text  Markup Language（超文本标记语言），是一门描述性语言，用于控制网页的结构。

**语法：**

```html
<标签符>内容</标签符>
```

标签也可以说成元素，是标记内容的类别，让浏览器将代码解释成页面效果，呈现给用户。

## **HTML结构**

（1）文档声明：`<!DOCTYPE html>`

（2）html标签对：`<html></html>`

（3）head标签对：`<head></head>`

（4）body标签对：`<body></body>`

<!--more-->

## head标签

##### title

网页的标题

##### meta(自闭合)

**name**属性：keywords表示网页的关键字；description表示用于告诉搜索引擎的网页摘要；author表示本页作者；copyright: 版权信息。**http-equiv**属性：Content-Type定义网页所使用的编码；refresh定义网页自动刷新跳转。

**记**：`<meta charset="utf-8" />`的作用是为了防止页面出现乱码，且这一句必须放在title标签以及其他meta标签前面。

##### style

用于定义元素的CSS样式。

##### script

用于定义页面的JavaScript的代码。

##### link(自闭合)

用于引入外部样式文件。

静态页面与动态页面的区别在于：**是否与服务器进行交互。**

## 文本标签

使用p标签会导致段落与段落之间有一定的间隙，而使用br(break)标签则不会。

br(自闭合)标签是用来给文字换行的，并且只用于**段落(p标签)**中的换行，而p标签是用来给文字分段的。

strong或者em标签内部的文本被强调为重要文本，并且搜索引擎对这两个标签也赋予一定的权重。

常用标签：

（1）粗体标签：strong、b

（2）斜体标签：i、em(emphasized)、cite

（3）上标标签：sup(superscripted)

（4）下标标签：sub(subscripted)

（5）中划线标签：s

（6）下划线标签：u(underline)

（7）大字号标签：big

（8）小字号标签：small

（9）水平线标签：hr(horizon)(自闭合)

  (10)  文本被删除：del 	eg. ~~删除~~

  (11)  更新文本：ins

**块(block)元素和行内(inline)元素**：

块元素：呈现时占据一整行，并且排斥其他元素与其位于同一行；块元素内部可以容纳其他块元素和行内元素。

行内元素：可以与其他行内元素位于同一行，内部可以容纳其他行内元素，但不可以容纳块元素。

inline-block元素：既具备块元素的特点，也具备行内元素的特点。

## 列表标签

### 有序列表（ol, ordered list）

子标签只能是`<li>`标签，并且可以通过type属性更改列表项符号。

### 无序列表（ul, unordered list）

同有序列表，并且文本只能在`<li>`元素内部添加。

### 定义列表

dl即definition list（定义列表）；dt即definition term（定义名词）；dd即definition description（定义描述）。

```html
<dl>
    <dt>名词</dt>
    <dd>描述</dd>
    ……
</dl>
```



## 表格标签

table表示表格；tr（table row）表示行；td（table data cell）表示单元格。tr数即行数，td数即列数。

```html
<table>
    <caption>表格标题</caption>
    <!--表头-->
    <thead>
        <tr>
            <th>表头单元格1</th>
            <th>表头单元格2</th>
        </tr>
    </thead>
    <!--表身-->
    <tbody>
        <tr>
            <td>表行单元格1</td>
            <td>表行单元格2</td>
        </tr>
        <tr>
            <td>表行单元格3</td>
            <td>表行单元格4</td>
        </tr>
    </tbody>
    <!--表脚-->
    <tfoot>
        <tr>
            <td>标准单元格5</td>
            <td>标准单元格6</td>
        </tr>
    </tfoot>
</table>
```

合并行属性：将表格相邻的N个行合并，纵向。

```html
<td rowspan = "跨越的行数"></td>
```

合并列属性：将表格相邻的N个列合并，横向。

```html
<td colspan = "跨越的列数"></td>
```



## 图片标签

```html
<img src="图片路径" alt="提示文字" title="提示文字" />
```

**记**：如果图片作为HTML的一部分，并且想被搜索引擎识别，使用img标签；如果仅仅作为修饰，则使用背景图片，通过CSS实现的。网页中图片大多使用位图。

**alt属性在搜索引擎优化中也很重要，会被赋予一定的权重。**

##### 位图

又叫做“像素图”，它是由像素点组成的图片。对于位图来说，放大图片后，图片会失真；缩小图片后，图片同样也会失真。

（1）jpg可以很好地处理大面积色调的图片，适合存储颜色丰富的复杂图片，如照片、高清图片等。此外，jpg体积较大，并且不支持透明。

（2）png是一种无损格式，可以无损压缩以保证页面打开速度。此外，png体积较小，并且支持透明，不过不适合存储颜色丰富的图片。

（3）gif图片效果最差，不过它适合制作动画。

##### 矢量图

是以一种数学描述的方式来记录内容的图片格式。

优点：图片无论放大、缩小或旋转等都不会失真。

缺点：难以表现色彩丰富的图片效果（非常差）。



## 超链接标签

```html
<a href="链接地址" target="打开方式">文本或图片</a>
```

打开方式：`_self`表示在原窗口打开、`_blank`表示在新窗口打开、`_parent`表示在父窗口打开、`_top`表示在顶层窗口打开。

锚点链接：链接地址为本页面内某部分的`#id`。

不确定链接地址时使用固定链接"#"。



## 表单标签

```html
<form name="Form" method="POST/GET" action="index.php" target="_blank" enctype="编码方式">
    //各种表单标签
</form>
```

### 单行文本框

```html
<input type="text" value="默认内容" maxlength="最多字符数"/>
```

required表示该文本框为必填项。

### 单选框

```html
<input type="radio" name="组名" value="取值" />
```

checked表示默认选中

### 复选框

```html
<input type="checkbox" name="组名" value="取值" />
```

checked表示默认选中

### 按钮

```html
<input type="button" value="取值" />
```

普通按钮一般情况下都是配合JavaScript来进行各种操作的；提交按钮一般都是用来给服务器提交数据的；重置按钮一般用来清除用户在表单中输入的内容。

### 多行文本框

```html
<textarea rows="行数" cols="列数" value="取值">默认内容</textarea>
```

### 下拉列表

```html
<select>
    <option>选项内容</option>
    ……
    <option>选项内容</option>
</select>
```

multiple设置下拉列表可以选择多项；size设置下拉列表显示几个列表项，取值为整数；selected表示是否选中。

## 无语义标签

div和span两个标签没有语义，配合CSS应用样式。

可以包含任何块元素和行内元素，不会与其他元素位于同一行；span是行内元素，可以与其他行内元素位于同一行。