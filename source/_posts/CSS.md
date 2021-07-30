---
title: CSS
date: 2021-02-05 18:21:34
updated: 2021-07-30 21:21:51
tags: 前端
---

CSS，全称为Cascading Style Sheet（层叠样式表），用于控制网页外观，使**结构**和**样式**分离。

## CSS引入方式

（1）外部样式表。

（2）内部样式表。

（3）行内样式表。

还有一种@import方式，他是先加载HTML后加载CSS，而link先加载CSS后加载HTML。如果HTML在CSS之前加载，页面用户体验非常差。外部样式表多用于公有样式，内部样式表多用于私有样式，而行内样式更多用于小修改或者优先级方面。

<!--more-->

## CSS单位

### px

根据屏幕分辨率的不同，1px的大小会发生变化。

### %

（1）width、height、font-size的百分比是相对于**父元素**“相同属性”的值来计算的。

（2）line-height的百分比是相对于父元素的font-size值来计算的。

（3）vertical-align的百分比是相对当前元素的line-height值来计算的。

### em

em是相对于“**当前元素**”的字体大小而言的。其中，1em等于“当前元素”字体大小。

（1）首行缩进使用text-indent:2em实现

（2）使用em作为统一单位

（3）使用em作为字体大小单位

### rem

相对于**根元素**（即html元素）的字体大小。

## CSS选择器

### 元素选择器

```html
元素{
	属性1: 取值1;
	......
	属性n: 取值n;
}
```

### id选择器

```html
#id{
	属性1: 取值1;
	......
	属性n: 取值n;
}
```

id属性具有唯一性

### class选择器

```html
.class{
    属性1: 取值1;
    ......
    属性n: 取值n;
}
```

class属性可以为同一页面的相同元素或不同元素设置相同的class。

为了避免class命名的重复，命名时一般取父元素的class名作为前缀

### 层次选择器

| 选择器 | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| M    N | 后代选择器，选择M元素内部后代N元素（**所有**N元素）          |
| M > N  | 子代选择器，选择M元素内部子代N元素（所有**第1级**N元素，直接子元素） |
| M ~ N  | 兄弟选择器，选择M元素**后**所有的同级N元素                   |
| M + N  | 相邻选择器，选择M元素**相邻**的下一个元素                    |

#### 后代选择器

```html
选择器1 选择器2{
    属性1: 取值1;
    ......
    属性n: 取值n;	
}
```

表示选择器1的**所有**（包括子、孙元素）后代中的选择器2元素，两个选择器间用空格分开。

#### 子代选择器

选取的是元素内部某一个子元素（只限子元素）

#### 兄弟选择器

选中元素后面（**不包括前面**）的某一类兄弟元素。

#### 相邻选择器

选中元素后面（不包括前面）的某一个“相邻”的兄弟元素。

### 群组选择器

```html
选择器1，选择器2{
    属性1: 取值1;
    ......
    属性n: 取值n;		
}
```

表示同时对多个选择器进行相同的操作，选择器之间用逗号分开。

### 属性选择器

```html
[type='radio']{
    属性1: 取值1;
    ......
    属性n: 取值n;	
}
```

**记**：选择器的优先级：`!important > 内联 > ID选择器 > 类选择器 > 标签选择器`；

## 字体样式

定义字体大小**font-size**可以使用关键字[small, medium, large]，但很少使用。

字体**font-family**从左到右自动降级。

**font-weight**: bold;

## 文本样式

定义文本的对齐方式：**text-align**[left, center, right]，也可以用于图片，可用于inline、inline-block元素，但不能用于block元素(可先转换为inline-block)。

text-align与margin: 0 auto的区别：

（1）“text-align:center；”实现的是文字、inline元素以及inline-block元素的水平居中。

（2）“margin:0 auto；”实现的是块元素的水平居中。（要指定块元素的宽度）

（3）“text-align:center；”在父元素中定义，“margin:0 auto；”在当前元素中定义。

**vertical-align**属性用于定义<u>周围</u>文字、inline元素或inline-block元素的基线相对于<u>该元素</u>的基线的垂直对齐方式。如果想要在div中实现图片的垂直居中，我们可以先为div定义display:table-cell，也就是将块元素转化为table-cell元素（表格单元格），然后再使用vertical-align:middle就可以实现了。table-cell元素的vertical-align属性是针对自身而言。vertical-align定义的是内部子元素相对于自身的对齐方式。

定义文本的修饰效果：**text-decoration**[none, underline, line--through, overline]，none表示去除划线效果；line-through表示中划线。

定义文本的大小写转换：**text-transform**[none, uppercase, lowercase, capitalize]

定义行高，表示一行的高度：**line-height**[10px]，指两行基线之间的垂直距离。vertical-align属性中的top、middle、baseline、bottom这四个属性值分别对应的就是：顶线、中线、基线、底线。

![顶线、中线、基线、底线](https://tvax2.sinaimg.cn/large/005SZbikly1goy9atx6ptj30cd03474j.jpg)

**行距**：上一行的底线与下一行的顶线的垂直距离。

**半行距**：行距的一半。

将标签h1的权重赋给网站的logo，指定h1元素的长宽与logo的长宽一样，然后定义h1的背景图片（background-image）为logo。使用LOGO图片作为h1标签的背景图片，然后使用“text-indent:-9999px；”来隐藏h1的文字内容。

## 列表样式

定义列表项符号：**list-style-type**[ol: decimal, lower-roman, upper-roman, lower-alpha, upper-alpha; ul: disc, circle, square]，常用none去掉列表项符号。

定义列表项图片：**list-style-image**[url(图片路径)]

## 表格样式

定义表格标题的位置：**caption-side**[top, bottom]

定义是否去除单元格之间的空隙：**border-collapse**[separate, collapse]

定义表格边框的边距：**border-spacing**[10px]

## 图片样式

定义图片水平对齐方式：**text-align**，在**父**元素定义是否对齐

定义图片垂直对齐方式：**vertical-align**，定义**周围**的行内元素或文本**相对**于该元素的垂直方式，在img元素定义

## 背景样式

定义背景图片：**background-image**[url(图片路径)]

定义背景图片重复：**background-repeat**[repeat(default), repeat-x, repeat-y, no-repeat]

定义背景图片位置：**background-position**[top left, top center, top right, center left, center center, center right, bottom left, bottom center, bottom right]

可缩写为：`background:url("src/img.jpg" no-repeat 50px 50px);`

## 超链接伪类

定义鼠标未经过元素时的样式：a:visited

定义鼠标经过a元素时的样式：a:hover，可以用于多种元素。

定义鼠标样式：cursor[default, pointer, text, url(图片地址)]，其中自定义鼠标样式的图片后缀名为.cur

## 盒子模型

![盒子模型](https://tva3.sinaimg.cn/large/005SZbikly1gnl3egg2gdj30b006k76u.jpg)

**记**： 只有块元素可以设置width和height，如果为行内元素设置width和height，可以使用display属性将行内元素转换为块元素。**所有的元素都可以看成一个盒子！**

“display:none”与“visibility:hidden”的区别：

“display:none”的元素被隐藏之后，不占据原来的位置。

“visibility:hidden”的元素被隐藏之后，依然占据原来的位置。

**内容区**有三个属性：width、height和overflow。使用width和height属性可以指定盒子内容区的高度和宽度。在这里注意一点，width和height这两个属性是针对内容区而言，并不包括padding部分。当内容信息太多而超出内容区所占范围时，可以使用overflow溢出属性来指定处理方法。

**边框**，border属性，其中“border:0”需要占用内存。“border:none”不需要占用内存。

**外边距**，叠加的情况

同级元素的外边距叠加

![同级元素的外边距叠加](https://tvax1.sinaimg.cn/large/005SZbikly1govz75o35fj30hr0art9j.jpg)

父子元素的外边距叠加

![父子元素的外边距叠加](https://tva4.sinaimg.cn/large/005SZbikly1govz7nfywtj30ha06sq3j.jpg)

空元素的外边距叠加

![空元素的外边距叠加](https://tvax3.sinaimg.cn/large/005SZbikly1govz7x62oej30ha08uwfk.jpg)

（1）水平外边距永远不会有叠加，水平外边距指的是margin-left和margin-right。

（2）垂直外边距只会在以上三种情况下会叠加，垂直外边距指的是margin-top和margin-bottom。（3）外边距叠加之后的外边距高度等于发生叠加之前的两个外边距中的最大值。

（4）外边距叠加针对的是block以及inline-block元素，不包括inline元素。因为inline元素的margin-top和margin-bottom设置无效。

**负margin**

（1）当元素的margin-top或者margin-left为负数时，“当前元素”会被拉向指定方向。

（2）当元素的margin-bottom或者margin-right为负数时，“后续元素”会被拉向指定方向。

应用：图片与文字对齐、自适应两列布局、元素垂直居中（父元素“position:relative”，子元素“position:absolute, top:50%, left:50%, margin-top:**-**height/2, margin-left:**-**width/2），tab选项卡

“**display:table-cell**”可以实现以下三种功能。

（1）图片垂直居中于元素。

```css
父元素{
    display: table-cell;
    vertical-align:center;
}
子元素{
    vertical-align:center;
}
```

（2）等高布局。（设置左右两个盒子分别**display:table-cell**，高度相等，并且高度由最大高度决定，自适应）

（3）自动平均划分元素，并且在一行显示。

```css
父元素{
    display:table;
}
子元素{
    display:table-cell;
}
```

**注：**将父元素的font-size设置为0，可以消除inline-block元素的间距。

## 浮动布局

**正常文档流**：默认情况下页面元素的布局情况，一个页面从上到下分为一行一行，其中块元素独占一行，相邻行内元素在每一行中按照从左到右直到该行排满。

**脱离文档流**：元素不按照默认情况布局，可能通过设置**浮动**或**定位**是的元素脱离了文档流。

设置浮动属性后，块元素的宽度就不再延伸。

**float：**当一个元素定义了“float:left”或“float:right”时，不管这个元素之前是inline、inline-block或者其他类型，都会变成block类型，并且后面的元素会紧跟着填上空缺的位置。如果浮动元素的高度大于父元素的高度，或者父元素没有定义高度，此时浮动元素会脱离父元素。但如果父元素也是浮动时，则父元素会自适应的包含子元素。

清除浮动：

（1）**clear**[left, right, both]，清除浮动给周围元素带来的影响。应用于浮动**后面**的元素。

（2）**overflow**:hidden 应用于浮动元素的**父**元素，而不是当前的浮动元素。但是会隐藏超出父元素的内容部分。

（3）**“::after伪元素”+“clear:both”**：`.clearfloat::after{clear:both;}`

## 定位布局

固定定位：**postition**[fixed]，相对于浏览器而言，被固定的元素不会随着滚动条的拖动改变位置。

相对定位：**position**[relative]，指元素的位置相对于它的原始位置计算的，会将元素转换为块元素。

绝对定位：**position**[absolute]，相对于父元素而言，定义在任意位置。

静态定位：**position**[static]

如果想实现子元素相对父元素定位，父元素`position:relative`，子元素`position:absolute`，绝对定位元素是相对于外层第一个设置了“position:relative；”“position:absolute；”或“position:fixed”的祖先元素来进行定位的。

**z-index**属性只有在元素定义“position:relative”“position:absolute或者“position:fixed”时才会被激活。

**记**：尝试用浮动布局，定位布局可能会使元素脱离原来的位置，使得布局不可控。

## CSS特性

### 继承性

子元素会继承父元素的字体及颜色，但padding，margin和border等不具备继承性。

注：某些子元素标签样式的优先级高于继承的样式。如a标签。

### 层叠性

对于同一个元素来说，如果我们重复定义多个相同的属性，并且这些样式具有相同的权重，CSS会以最后定义的属性的值为准。

（1）引用方式冲突。

​			**行内样式>（内部样式=外部样式）**

（2）继承方式冲突。

​			最近的祖先元素的优先级最高

（3）指定样式冲突。

​			当直接指定的“**当前元素**”样式发生冲突时，样式权值高者获胜。

​			常用的有：行内样式> id选择器> class选择器>元素选择器

| 选择器      | 权值 |
| ----------- | ---- |
| 通配符      | 0    |
| 伪元素      | 1    |
| 元素选择器  | 1    |
| class选择器 | 10   |
| 伪类        | 10   |
| 属性选择器  | 10   |
| id选择器    | 100  |
| 行内样式    | 1000 |

（4）继承样式与指定样式冲突。

​		指定样式优先。

（5）! important。

​		如果一个样式使用!important来声明，则这个样式会覆盖CSS中任何的其他样式声明。

## 图形

### 实现三角形

```css
#triangle{
    border-width: 10px;
    border-style: solid;
    border-color: red transparent transparent transparent;
}
```

### 带边框的三角形

<u>子元素的绝对定位是根据父元素的“内容边界（content）”进行定位的。</u>由于盒子width和height都是0，因此content是在盒子的中心（也就是中心点）。

```css
#outer{
	position: relative(absolute);
}
#inner{
    position: absolute;
    top: -(border-width)+1;
    left: -(border-width);
}
```

### 圆形

元素的宽度和高度定义为相同值，然后四个角的圆角半径定义为宽度（或高度）的一半。

```css
border-radius: px/百分比/em;
```

### 半圆

高度height设为宽度width的一半，并且左上角和右上角的圆角半径定义与元素的高度一致，而右下角和左下角的圆角半径定义为0。

### 椭圆

元素的宽度和高度不相等，其中四个角的圆角水平半径定义为宽度的一半，垂直半径定义为高度的一半。

```css
border-radius:x/y;
```

x表示圆角的水平半径，y表示圆角的垂直半径。

## 性能优化

（1）属性缩写。

- 边框缩写：`border: 1px solid red;`
- padding或margin缩写
- 背景缩写：`background:url("src/img.jpg" no-repeat 50px 50px);`
- 字体缩写：`font: "宋体" 10px/1em bold;`
- 颜色值缩写：`color: #abc(aabbcc);`

（2）语法压缩。

- 最后一个属性后的分号
- url的引号
- 属性值为0的单位
- 省略可以继承的属性

（3）图片压缩。

（4）选择器优化。

​	浏览器对选择器规则是从右到左进行解析的，id选择器和class选择器前不要加元素名。

（5）CSS模块化。

（6）压缩工具。

（7）CSS Sprite技术。

（8）性能评估。

## 雪碧图

（1）使用Photoshop或者其他工具将小背景图合并成为一张大背景图，其中每一张小背景图要精确调整。

（2）使用background-image属性引入大背景图，并且结合background-position属性定位取出相应的图标。

优点：减少HTTP请求数，从而提高页面的加载速度。

## Something

### 包含块

一个元素的包含块是由离它最近的“块级祖先元素”的“内容边界”决定的。但当元素被设置为绝对定位时，此时该元素的包含块是由离它最近的“position:relative”或“position:absolute”的祖先元素决定。

**1．根元素**

根元素（HTML元素），是一个页面中最顶端的元素，它没有父元素。根元素存在的包含块，被称为“初始包含块（initial containing block）。

**2．固定定位元素**

如果元素的position属性为fixed，那么它的包含块是当前可视窗口，也就是当前浏览器窗口。

**3．静态定位元素和相对定位元素**

如果元素的position属性为static或relative，那么它的包含块是它最近的块级祖先元素创建的。<u>**祖先元素必须是block、inline-block或者table-cell类型。**</u>

**4．绝对定位元素**

如果元素的position属性为absolute，那么它的包含块是由最近的position属性不为static的祖先元素。这里的祖先元素可以是块元素，也可以是行内元素。

（1）如果祖先元素是块元素，则包含块的范围为祖先元素的padding edge形成。

（2）如果祖先是行内元素，则包含块取决于祖先元素的direction属性。

### 层叠上下文

如果一个元素具备以下任何一个条件（不考虑CSS3），则该元素会创建一个新的层叠上下文。

（1）根元素。

（2）z-index不为auto的定位元素。

在同一层叠上下文中的层叠级别：

（1）背景和边框（父级）：也就是当前层叠上下文的背景和边框。

（2）负z-index:z-index为负值的“内部元素”。

（3）块盒子：普通文档流下的块盒子（block-level box）。

（4）浮动盒子：非定位的浮动元素（除了position:relative的浮动盒子）。

（5）行内盒子：普通文档流下的行内盒子（inline-level box）。

（5）z-index:0:z-index为0的“内部元素”。

（6）正z-index:z-index为正值的“内部元素”。

![层叠级别](https://tva2.sinaimg.cn/large/005SZbikly1goyir7empuj30hl0c7jsn.jpg)

层叠上下文是可以嵌套的，内部元素所创建的层叠上下文均受制于父元素创建的层叠上下文。不同的层叠上下文中，比较的是“父级元素层叠级别”。元素显示顺序以“父级层叠上下文”的层叠级别来决定显示的先后顺序，与自身的层叠级别无关。

### 格式上下文

#### 行级格式上下文

行内盒子，即inline-level box。元素类型（即display属性）为inline、inline-block、inline-table的元素，会生成行内盒子。行内盒子会参与行级格式上下文。

#### 块级格式上下文（Block Formatting Context，BFC）

块盒子，即block-level box。元素类型（即display属性）为block、table、list-item的元素，会生成块盒子。块盒子会参与块级格式上下文。

如果一个元素具备以下任何一个条件，则该元素都会**创建**一个新的BFC。

（1）根元素。

（2）float属性除了none以外的值，也就是“float:left”和“float:right”。

（3）position属性除了static和relative以外的值，也就是“position:absolute”和“position:fixed”。

（4）overflow属性除了visible以外的值，“overflow:auto”“overflow:hidden”和“overflow:scroll”。

（5）元素类型（即display属性）为inline-block、table-caption、table-cell。block、table、list-item等类型的元素的是**参与**BFC，而不是创建BFC。

根元素也会创建BFC，在默认情况下一个页面中所有的元素都处于同一个BFC中。

（1）在一个BFC内部，盒子会在垂直方向上一个接着一个地排列。

（2）在一个BFC内部，相邻的margin-top和margin-bottom会叠加。

（3）在一个BFC内部，每一个元素的左外边界会紧贴着包含盒子的左边，即使存在浮动也是如此。

（4）在一个BFC内部，如果存在内部元素是一个新的BFC，并且存在内部元素是浮动元素。则该BFC的区域不会与float元素的区域重叠。

（5）BFC就是页面上的一个隔离的盒子，该盒子内部的子元素不会影响到外面的元素。

（6）计算一个BFC的高度时，其内部浮动元素的高度也会参与计算。



