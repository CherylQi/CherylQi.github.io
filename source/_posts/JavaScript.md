---
title: JavaScript
date: 2021-02-10 18:21:46
updated: 2021-04-03 09:20:40
tags: 前端
---

## JavaScript引入方式

### 外部引入

是最常用的方式，将JS代码单独写入一个文件中，引入时`<script src="test.js">`，可以提高网站的性能和可维护性。

### 内部引入

通常在head中引入，也可以在body中引入，在body中引入时放在底部，可以保证浏览器先解析HTML内容，用户的体验好，同时避免JavaScript操作DOM失效。

### 行内引入

直接写在html语句中，这种方法会产生冗余代码，不利于维护。

<!--more-->

## 数据类型

### 基本数据类型

数字：当字符和字符串拼接时，数字会隐式转换为字符串，因此如果将数字转换为字符串可以将数字与空字符串相加，也可以使用a.toString()方法。

字符串：Number()将“数字型字符串"转换为数字；parseInt()可以提取字符串中前部分的整数部分，如果第一个字符非数字，则直接返回Nan，但可以是"+"或"-";parseFloat()既可以提取整数部分也可以提取小数部分。

布尔值

未定义值：已经用var声明，应该有值，但未赋值；当调用函数时，未提供相应参数，即该参数undefined；对象的属性未赋值；函数无返回值时默认返回undefined。

空值：未分配内存，代表一个空对象指针。

### 引用数据类型

数组和对象

## 函数

### 函数调用

在超链接中调用

```html
<a href="javascript.test()">test</a>
```

## 对象

### 字符串对象

`字符串.length`可以获取字符串长度，空格也作为一个字符。

`字符串.toLowerCase();字符串.toUpperCase()`分别将字符串转换为小写和大写。

`字符串.charAt(i)`可以获取字符串中的某个字符。

`字符串.substring(start, end)`可以截取字符串中某一段，其中范围为`[start, end)`。

`字符串.replace(原字符串/正则表达式, 替换字符串)`替换字符串中的某部分为指定字符串，其中第一种方式只会替换第一个符合条件的字符串，第二种方式可以替换所有符合条件的字符串。

`字符串.split("分隔符")`将字符串按照条件分隔，返回值为一个字符串数组，如果`字符串.spilt("")`，则将字符串分隔为一个个字母。

`字符串.indexOf(目标字符串);字符串.lastIndexOf(目标字符串)`分别返回目标字符串首次出现的下标和最后出现的下标，如果不存在则返回-1。

### 数组对象

`数组.length`可以获取数组长度。

`数组.slice(start, end)`可以截取数组的一部分，范围为[start, end)。

`数组.unshift(new1, new2, ..., newn)`表示在数组开头添加新元素。

`数组.push(new1, new2, ..., newn)`表示在数组尾部添加新元素，常用于不知道原数组长度时。

`数组.shift()`表示删除数组的第一个元素。

`数组.pop()`表示删除数组的最后一个元素。

`数组.sort()`将数组排序，通过比较函数确定排序方式为升序还是降序。

`数组.reverse()`将数组反向排列。

`数组.join("连接符")`将数组中的所有元素连接成一个字符串。

### 日期对象

```javascript
var 日期对象 = new Date()
```

​													**获取时间**

| 方法          | 说明                                                        |
| :------------ | :---------------------------------------------------------- |
| getFullYear() | 获取年份，取值为4位数字                                     |
| getMonth()    | 获取月份，取值为0（一月）到11（十二月）之间的整数，输出时+1 |
| getDate()     | 获取日数，取值为1~31之间的整数                              |
| getDay()      | 获取星期几                                                  |
| getHours()    | 获取小时数，取值为0~23之间的整数                            |
| getMinutes()  | 获取分钟数，取值为0~59之间的整数                            |
| getSeconds()  | 获取秒数，取值为0~59之间的整数                              |

​													**设置时间**

| 方法                                                 | 说明                     |
| :--------------------------------------------------- | :----------------------- |
| setFullYear(year, month(可选), day(可选))            | 可以设置年、月、日       |
| setMonth(month, day(可选))                           | 可以设置月、日           |
| setDate(day)                                         | 可以设置日               |
| setHours(hour, min(可选), sec(可选), millisec(可选)) | 可以设置时、分、秒、毫秒 |
| setMinutes(min, sec(可选), millisec(可选))           | 可以设置分、秒、毫秒     |
| setSeconds(sec, millisec(可选))                      | 可以设置秒、毫秒         |

### 数学对象

不需要new关键字创建。

​																	**Math对象的属性**

| 属性 | 说明   | 对应的数学形式 |
| :--- | :----- | :------------- |
| PI   | 圆周率 | π              |

方式：**度数 \* Math.PI/180**

​																	**Math对象的方法**

| 方法         | 说明                 |
| :----------- | :------------------- |
| max(a,b,…,n) | 返回一组数中的最大值 |
| min(a,b,…,n) | 返回一组数中的最小值 |
| floor(x)     | 向下取整             |
| ceil(x)      | 向上取整             |

​																	**Math对象中的三角函数方法**

| 方法        | 说明                   |
| :---------- | :--------------------- |
| sin(x)      | 正弦                   |
| cos(x)      | 余弦                   |
| tan(x)      | 正切                   |
| asin(x)     | 反正弦                 |
| acos(x)     | 反余弦                 |
| atan(x)     | 反正切                 |
| atan2(y, x) | 反正切（注意y、x顺序） |

**随机生成某个范围内的“任意数”**

**（1）Math.random()*m**

表示生成0~m之间的随机数，例如“Math.random()*10”表示生成0-10之间的随机数。

**（2）Math.random()*m+n**

表示生成n~m+n之间的随机数，例如“Math.random()*10+8”表示生成8-18之间的随机数。

**（3）Math.random()*m-n**

表示生成-n~m-n之间的随机数，例如“Math.random()*10-8”表示生成-8-2之间的随机数。

**（4）Math.random()*m-m**

表示生成-m~0之间的随机数，例如“Math.random()*10-10”表示生成-10-0之间的随机数。

**随机数生成某个范围内的“整数”**

对于Math.random()5来说，由于floor()是向下取整，因此Math.floor(Math.random()*5) 生成的是0~4之间的随机整数。生成0~5之间的随机整数：

```javascript
Math.floor(Math.random()*(5+1))
```

也就是说，如果你想生成0到m之间的随机整数：

```javascript
Math.floor(Math.random()*(m+1))
```

如果你想生成1到m之间的随机整数（包括1和m）：

```javascript
Math.floor(Math.random()*m)+1
```

如果你想生成n到m之间的随机整数（包括n和m）：

```javascript
Math.floor(Math.random()*(m-n+1))+n
```

## DOM

### Document Object Model

![DOM树](https://tvax1.sinaimg.cn/large/005SZbikly1gnxsamfjncj30bx05aglm.jpg)

​																				**DOM树**

每一个元素就是一个节点，而每一个节点就是一个对象。**在操作元素时，其实就是把这个元素看成一个对象，然后使用这个对象的属性和方法来进行相关操作**。

### 节点类型

元素节点、属性节点、文本节点

**只有元素节点才可以拥有子节点，属性节点和文本节点都无法拥有子节点**

### 获取元素

***严格区分大小写、获取前应`window.onload = function ()`**

（1）`getElementById("#id")`

（2）`getElementsByTagName("tag")`

（3）`getElementsByClassName(".class")`

（4）`querySelector("选择器")`、`querySelectorAll("选择器")`

（5）`getElementsByName("name")`

（6）`document.title`、`document.body`

获取多个元素的方法返回值为类数组，只可以用数组的length属性和下标形式。

### 创建元素

  (1) `createElement("tag")`   

  (2) `createTextNode(text)`

为创建的元素属性赋值

```javascript
New_Element.id = "element";
New_Element.className = "element_class";
New_Element.type = "text";
...
```

`getAttribute()`可以**获取**自定义属性；`setAttribute()`可以设置自定义属性；`removeAttribute`可以**删除**某个属性；`hasAttribute()`**判断**是否有某个属性。 

### 插入元素

将新创建的元素加入到原有元素的后面

```javascript
origin.append(new);               //文本节点插入元素节点
parent.append(child);               //新元素加到原有元素后面
```

将新创建的元素加入原有元素的前面

```javascript
origin.append(new);               //文本节点插入元素节点
parent.insertBefore(child, ref);               //新元素加到指定ref元素前面
```

### 删除元素

删除父元素的某个子元素

```javascript
parent.removeChild(child);
```

### 复制元素

```javascript
obj.cloneNode(bool);
```

### 替换元素

```javascript
parent.replace(new_child, origin_child);
```

### CSS属性

**获取**CSS属性`getComputedStyle(obj).attr`，也可以写成`getComputedStyle(obj)["attr"]`，根据优先级，可以获取内部样式和外部样式。

**设置**CSS属性`obj.style.attr=""`，也可以写成`obj.style["attr"]=""`，当设置多个属性时`obj.style.cssText=""`。（设置单个属性时采用驼峰型写法）

### DOM遍历

**获取父元素**`obj.parentNode`

**获取子元素**`obj.childNodes`包含元素节点以及文本节点。`obj.children`只获取元素节点。

**获取兄弟元素**`obj.previousSibling`包含前一个兄弟元素节点以及文本节点；`obj.previousElementSibling`只获取前一个兄弟的元素节点。

## 事件

```javascript
obj."Event" = function(){
    ······
};
```

### 鼠标事件

| 事件        | 说明         |
| :---------- | :----------- |
| onclick     | 鼠标单击事件 |
| onmouseover | 鼠标移入事件 |
| onmouseout  | 鼠标移出事件 |
| onmousedown | 鼠标按下事件 |
| onmouseup   | 鼠标松开事件 |
| onmousemove | 鼠标移动事件 |

### 键盘事件

（1）键盘按下：`onkeydown`

（2）键盘松开：`onkeyup`

### 表单事件

（1）`onfocus`获取焦点时触发的事件、`onblur`失去焦点时触发的事件

（2）`onselect`选中文本框中的内容时触发

（3）`onchange`当单选框、复选框和下拉列表选择了某一项时触发

### 编辑事件

（1）`oncopy`是否允许页面内容被复制

（2）`onselectstart`是否允许页面内容被选中

（3 ）`oncontextmenu`是否允许使用鼠标右键

### 页面事件

`onload`表示文档加载完成后再执行的一个事件

```javascript
window.onload = function(){
    ……
}
```

`onbeforeunload`表示离开页面之前触发的一个事件。

```JavaScript
window.onbeforeunload = function(){
    ……
}
```

### 事件监听器

事件处理器`window.onload`多次调用只会执行最后一次。

解决办法：绑定事件监听器，可以为同一个元素添加多个相同的事件。

```javascript
obj.addEventListener("type", function(){},false);
```

解绑事件

```javascript
obj.removeEventListener("type", function(){},false);
```

### 事件对象

| 属性     | 说明            |
| :------- | :-------------- |
| type     | 事件类型        |
| keyCode  | 键码值          |
| shiftKey | 是否按下shift键 |
| ctrlKey  | 是否按下Ctrl键  |
| altKey   | 是否按下Alt键   |

### this闭包

## Window对象

![Window对象](https://tvax1.sinaimg.cn/large/005SZbikly1gnyg92xebqj30df071dg2.jpg)

| 子对象    | 说明                               |
| :-------- | :--------------------------------- |
| document  | 文档对象，用于操作页面元素         |
| location  | 地址对象，用于操作URL地址          |
| navigator | 浏览器对象，用于获取浏览器版本信息 |
| history   | 历史对象，用于操作浏览历史         |
| screen    | 屏幕对象，用于操作屏幕宽度高度     |

**一个窗口就是一个window对象，这个窗口里面的HTML文档就是一个document对象，document对象是window对象的子对象**。

### 窗口操作

打开窗口`window.open(url, target)`，返回值为新窗口的window对象。

关闭窗口`window.close()`

### 对话框

`alert("提示文字")`

`confirm("提示文字")`，如果点击确定则返回true，否则返回false

`prompt("提示文字")`，返回输入的字符串

### 定时器

`setTimeout(code, time);`参数code可以是一段代码，可以是一个函数，也可以是一个函数名。

`clearTimeout()`可以清除定时器，表示暂停

参数time是时间，单位为毫秒，表示要过多长时间才执行code中的代码。

`setInterval(code, time);`与前者不同的是：setTimeout()只执行一次；而setInterval()可以重复执行无数次，可以理解为每隔time执行一次code。同样可以使用`clearInterval()`清除定时器，以暂停。

### location对象

| 属性   | 说明                       |
| :----- | :------------------------- |
| href   | 当前页面地址               |
| search | 当前页面地址“？”后面的内容 |
| hash   | 当前页面地址“#”后面的内容  |

`window.location.href`获取或设置当前页面的地址

`window.location.search`获取和设置当前页面地址“?”后面的内容

`window.location.hash`获取和设置当前页面地址井号（#）后面的内容,常用于锚点链接

### navigator对象

`window.navigator.userAgent`获取浏览器的类型

## Document对象

document对象是浏览器为每个窗口内的HTML页面创建的一个对象，是HTML文档里所有的内容。

### Document对象属性

| 属性              | 说明                          |
| :---------------- | :---------------------------- |
| document.title    | 获取文档的title               |
| document.body     | 获取文档的body                |
| document.forms    | 获取所有form元素              |
| document.images   | 获取所有img元素               |
| document.links    | 获取所有a元素                 |
| document.cookie   | 文档的cookie                  |
| document.URL      | 获取当前文档的URL，不能设置   |
| document.referrer | 返回使浏览者到达当前文档的URL |

### Document对象方法

| 方法                              | 说明                            |
| :-------------------------------- | :------------------------------ |
| document.getElementById()         | 通过id获取元素                  |
| document.getElementsByTagName()   | 通过标签名获取元素              |
| document.getElementsByClassName() | 通过class获取元素               |
| document.getElementsByName()      | 通过name获取元素                |
| document.querySelector()          | 通过选择器获取元素，只获取第1个 |
| document.querySelectorAll()       | 通过选择器获取元素，获取所有    |
| document.createElement()          | 创建元素节点                    |
| document.createTextNode()         | 创建文本节点                    |
| document.write()                  | 输出内容                        |
| document.writeln()                | 输出内容并换行                  |