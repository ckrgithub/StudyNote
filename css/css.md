# css
## 一、简介
### 1.css概述
css 指层叠样式表(Cascading Style Sheets)
样式定义如何显示HTML元素
样式通常存储在样式表中
把样式添加到HTML4.0中，是为了解决内容与表现分离的问题
外部样式表可以极大提高工作效率
外部样式表通常存储在css文件中
多个样式定义可层叠为一

### 2.css语法
css规则由两个主要的部分构成：选择器，以及一条或多条声明
```css
  selector {declaration1;declaration2;declaration3;...declarationN }
```
选择器通常是你需要改变样式的html元素。每条声明由一个属性和一个值组成。属性是你希望设置样式属性。每个属性有一个值。属性和值被冒号分开。
```css
  selector {property: value}
```
下面这行代码作用是将h1元素内的文字颜色定义为红色，同时将字体大小设置为14像素。
```css
  h1 {color:red; font-size:14px;}
  h1是选择器，color和font-size是属性，red和14px是值
```
#### 值的不同写法和单位
```css
  p { color: #ff0000; }
  p { color: #f00; }
  p { color: rgb(255,0,0); }
  p { color: rgb(100%,0%,0%); }
  注意，当使用rgb百分比时，即使当值为0时也要写百分比符号。
  如果值为若干单词，则要给值加引号：
  p {font-family: "sans serif";}
```
### 3.多重声明
如果定义不止一个声明，则需要用分号将每个声明分开。
```css
  p {text-align:center;color:red;}
```
#### 空格和大小写
大多数样式表包含不止一条规则，而大多数规则包含不止一个声明。
```css
  body {
    color: #000;
    background: #fff;
    margin: 0;
    padding: 0;
    font-family: Georgia, Palatino, serif;
  }
```
### 4.选择器的分组
你可以对选择器进行分组，这样，被分组的选择器可以分享相同的声明。用逗号将需要分组的选择器分开。
```css
  h1,h2,h3,h4,h5,h6 {
    color: green;
  }
```
#### 继承及其问题
```css
  body {
    font-family: Verdana, sans-serif;
  }
```
根据上面，站点的body元素将使用Verdana字体。通过css继承，子元素将继承最高元素(在本例中是body)所拥有的属性(这些子元素如：p,td,ol,ul,li,dl,dt和dd)。
所有body的子元素都应显示Verdana字体，子元素的子元素也一样。但Netscape 4不支持继承。IE/Windows直到IE6存在表格内的字体样式会被忽略.
### 5.派生选择器
通过依据元素在其位置的上下文关系来定义样式。
在css1中，通过这种方式来应用规则的选择器被称为上下文选择器。在css2中，它们称为派生选择器。
派生选择器允许你根据文档的上下文关系来确定某个标签的样式。
比如，你希望列表中的strong元素变为斜体字，而不是通常的粗体字，可以这样定义一个派生选择器：
```css
  li strong {
    font-style: italic;
    font-weight: normal;
  }
  <p><strong>我是粗体字，不是斜体字</strong></p>
  <ol>
  <li><strong>我是斜体字。因为strong元素位于li元素内</strong></li>
  </ol>
```
```css
  strong {
    color: red;
  }
  h2 {
    color: red;
  }
  h2 strong {
    color: blue;
  }
  <p><strong>red</strong></p>
  <h2>also red</h2>
  <h2><strong>blue</strong></h2>
```
### 6.id选择器
标有特定id的html元素指定特定的样式。id选择器以“#”来定义。
```css
  #red { color: red; }
  #green { color:green; }
  <p id="red">这个是红色</p>
  <p id="green">这个是绿色</p>
  注意： id属性只能在每个html文档中出现一次
```
#### id选择器和派生选择器
id选择器常常用于建立派生选择器
```css
  #sidebar p{
    font-style: italic;
    text-align: right;
    margin-top: 0.5rem;
  }
```
上面样式只会应用于出现在id是sidebar的元素内的段落.
一个选择器，多种用法
即使被标注为sidebar的元素只能在文档中出现一次，这个id选择器作为派生选择器也可以被使用很多次：
```css
  #sidebar p{
    font-style: italic;
    text-align: right;
    margin-top: 0.5em;
  }
  #sidebar h2{
    font-size: 1em;
    font-weight: normal;
    font-style: italic;
    margin: 0;
    line-height: 1.5;
    text-align: right;
  }
```
### 7.单独的选择器
id选择器即使不被用来创建派生选择器，也可以独立发挥作用：
```css
  #sidebar {
    border: 1px dotted #000;
    padding: 10px;
  }
```
上面id为sidebar的元素将拥有一个像素宽的黑色点状边框，同时其周围有10个像素宽的内边距
### 8.类选择器
在css中，类选择器以一个点号显示：
```css
  .center {text-align: center}
  <h1 class="center">居中</h1>
  <p class="center">也居中</p>
```
上面中，所有拥有center类的html元素均为居中.注意：类名的第一个字符不能使用数字.
和id一样，class也可被用作派生选择器：
```css
  .fancy td {
    color: #f60;
    background: #666;
  }
```
上面中，类名为fancy的更大的元素内部的表格单元都会以灰色背景显示橙色文字。(名为fancy的更大的元素可能是一个表格或者一个div)
元素也可以基于它们的类而被选择：
```csss
td.fancy {
  color: #f80;
  background: #666;
}
```
你可以将类fancy分配给任何一个表格元素任意多的次数.那些以fancy标注的单元格都带有灰色背景的橙色。那些没有被分配为fancy的类的
单元格不会受这条规则影响.注意：class为fancy的段落也不会带有灰色背景的橙色,任何其他被标注为fancy的元素也不会受这条规则影响.
### 9.属性选择器
对带有指定属性的html元素设置样式
注意：只有在规定了！DOCTYPE时，IE7和IE8才支持属性选择器。
```css
  [title]
  {
    color: red;
  }
  <h2 title="hello world">hello world</h2>
```
上面带有title属性的所有元素设置样式
#### 属性和值选择器
下面例子为title="w3school"的所有元素设置样式：
```css
  [title=w3school]
  {
    border: 5px solid blue;
  }
```
多个值
下面例子包含指定值的title属性的所有元素设置样式。适用于由空格分隔的属性值
```css
  [title~=hello] { color:red; }
  <h2 title="hello">此处红色</h2>
  <p title="study hello">此处红色</p>
  <h2 title="world">不生效</h2>
```
下面例子为带有包含指定值lang属性的所有元素设置样式.适用于由连字符分隔的属性值
```css
  [land|=en] {color:red;}
  <p lang="en">此处红色</p>
  <p lang="en-us">此处红色</p>
  <p lang="us">不生效</p>
```
#### 设置表单的样式
属性选择器在为不带有class或者id的表单设置样式时特别有用：
```css
  input[type="text"]
  {
    width: 150px;
    display: block;
    margin-bottom: 10px;
    background-color: yellow;
    font-family; Verdana,Arial;
  }
  
  input[type="button"]
  {
    width: 120px;
    margin-left: 35px;
    display: block;
    font-family: Verdana,Arial;
  }
  <form name="input" action="" method="get">
    <input type="text" name="Name" value="背景色为黄色" size="20"></input>
    <input type="button" value="宽度为120的button"></input>
  </form>
```
#### 选择器参考手册
|选择器|描述|
|[attribute]|用于选取带有指定属性的元素|
|[attribute=value]|用于选取带有指定属性和值的元素|
|[attribute~=value]|用于选取属性值中包含指定词汇的元素|
|[attribute|=value]|用于选取带有以指定值开头的属性值的元素,该值必须是整个单词|
|[attribute^=value]|匹配属性值以指定值开头的每个元素|
|[attribute$=value]|匹配属性值以指定值结尾的每个元素|
|[attribute*=value]|匹配属性值中包含指定值的每个元素|
### 10.如何创建css
如何插入样式表
当读到一个样式表时，浏览器会根据它来格式化html文档。插入样式表的方法有三种:
#### 外部样式表
当样式需要应用于很多页面时，外部样式表将是理想的选择。在使用外部样式表情况下，你可以通过改变一个文件来改变整个站点的外观。
每个页面使用<link>标签链接到样式表。<link>标签在(文档的)头部:
```css
  <head>
    <link rel="stylesheet" type="text/css" href="mystyle.css"/>
  </head>
```
外部样式表可以在任何文本编辑器中进行编辑。文件不能包含任何的html标签。样式表应该以.css扩展名进行保存.
```css
  hr {color: sienna;}
  p {margin-left: 20px;}
  body {background-image: url("images/back40.gif");}
```
#### 内部样式表
当单个文档需要特殊的样式时，就应该使用内部样式表。可以使用<style>标签在文档头部定义内部样式表，
```css
  <head>
    <style type="text/css">
      hr {color: sienna;}
      p {margin-left: 20px}
      body {background-image: url('images/back40.gif');}
    </style>
  </head>
```
#### 内联样式
当样式仅需要在一个元素上应用一次时。要使用内联样式，你需要在相关的标签内使用样式(style)属性.
```css
  <p style="color: sienna; margin-left: 20px">这是个段落</p>
```
#### 多重样式
如果某些属性在不同样式表中被同样的选择器定义，那么属性值将从更具体的样式表中被继承过来
```css
  外部样式表：h3 {
    color: red;
    text-align: left;
    font-size: 8pt;
  }
  内部样式表：h3{
    text-align: right;
    font-size: 20pt;
  }
  假如拥有内部样式表的这个页面同时与外部样式表链接，那么h3得到样式：
  h3{
    color: red;
    text-aligh: right;
    font-size: 20pt;
  }
```
## 二、css 样式
### 1.css背景
css允许应用纯色作为背景，也允许使用背景图像创建相当复杂的效果
* 背景色：使用background-color 属性为元素设置背景色
```css
  p {background-color: gray;
```
可以为所有元素设置背景色，这包括body一直到em和a等行内元素。background-color不能继承，默认值时transparent。也就是说，如果一个元素
没有指定背景色，那么背景是透明的。
* 背景图像：要把图像放入背景，需要使用background-image属性，默认值是none
```css
  body {background-image: url('/images/logo.jpg');}
```
* 背景重复：如果需要在页面上对背景图像进行平铺，可以使用background-repeat 属性
属性repeat导致图像水平垂直方向上都平铺。repeat-x和repeat-y分别导致图像只在水平或垂直方向上平铺，no-repeat则不允许图像在任何方向平铺
```css
  body {
    background-image: url('/images/logo.jpg');
    background-repeat: repeat-y;
  }
```
* 背景定位：利用background-position 属性改变图像在背景中的位置
```css
  body {
    background-image: url('/images/logo.jpg');
    background-repeat: no-repeat;
    background-position: center;
  }
```
为background-position属性提供值有很多方法，首先，可以使用一些关键字：top、bottom、left、right和center。也可以使用长度值，如10px或5cm；也可以使用百分数值，如10%
* 关键字
位置关键字可以按任何顺序出现，只要保证不超过两个关键字：一个对应水平方向，另一个对应垂直方向。如果出现一个关键字，则认为另一个关键字是center.
如：希望每个段落的中部上方出现一个图形
```css
  p {
    background-image: url('/images/logo.jpg');
    background-repeat: no-repeat;
    background-position: top;
  }
```
* 百分数值
```css
  body {
    background-image: url('/images/logo.jpg');
    background-repeat: no-repeat;
    background-position: 50% 50%;
  }
```
这会导致图像适当放置，其中心与其元素的中心对齐，也就是说，百分数值同时应用于元素和图像。
* 长度值：元素内边距区左上角的偏移。偏移点是图像的左上角。
```css
  body {
    background-image: url('/images/logo.jpg');
    background-repeat: no-repeat;
    background-position: 50px 100px;
  }
```
* 背景关联：
如果文档比较长，那么当文档向下滚动时，背景图像也会随之滚动。当文档滚动到超过图像的位置时图像就会消失
可以使用 background-attachment属性防止这种滚动。
```css
  body {
    background-image: url('/images/logo.jpg');
    background-repeat: no-repeat;
    background-attachment: fixed
  }
```










# 感谢
[w3school](http://www.w3school.com.cn/css/index.asp)
