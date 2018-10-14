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
| 选择器 | 描述 |
|---|---|
| [attribute] | 用于选取带有指定属性的元素 |
| [attribute=value]  | 用于选取带有指定属性和值的元素 |
| [attribute~=value] | 用于选取属性值中包含指定词汇的元素 |
| [attribute|=value] | 用于选取带有以指定值开头的属性值的元素,该值必须是整个单词 |
| [attribute^=value] | 匹配属性值以指定值开头的每个元素 |
| [attribute$=value] | 匹配属性值以指定值结尾的每个元素 |
| [attribute*=value] | 匹配属性值中包含指定值的每个元素 |
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
### 2.css文本
css文本属性可定义文本的外观。
通过文本属性，你可以改变文本的颜色、字符间距，对齐文本，装饰文本，对文本进行缩进等。
#### 缩进文本
text-indent属性可以方便地实现文本缩进
所有段落的首行缩进5em:
```css
  p {
    text-indent: 5em;
  }
```
注意：可以为所有块级元素应用text-indent，但无法将该属性应用于行内元素，图像之类的替换元素也无法应用
  text-indent属性。不过，如果一个块级元素的首行中有一个图像，它会随改行的其余文本移动
* 使用负值：可以实现很多有趣的效果，比如"悬挂缩进"，即第一行悬挂在元素中余下部分的左边：
```css
  p {
    text-indent: -5em;
  }
```
不过在为text-indent设置负值时要当心，如果对一个段落设置了负值，那么首行的某些文本可能会超出浏览器窗口的左边界。
为了避免出现这种显示问题，建议针对负缩进再设置一个外边距或内边距：
```css
  p {
    text-indent: -5em;
    padding-left: 5em;
  }
```
* 使用百分比值：相对于缩进元素的父元素的宽度
在下例中，缩进值是父元素的20%，即100个像素：
```css
  div {width: 500px;}
  p {text-indent: 20%;}
  <div>
    <p>this is a paragragh</p>
  </div>
```
* 继承
```css
  div#outer {width: 500px;}
  div#inner {text-indent: 10%;}
  p {width: 200;}
  
  <div id="outer">
  <div id="inner">some text
    <p>this is a paragragh</p>
  </div>
  </div>
```
标记中的段落会缩进50像素，这是因为段落继承id为inner的div元素的缩进值
#### 水平对齐
text-align: 会影响一个元素中的文本行互相之间的对齐方式。
可能值有left、right、center、justify、inherit
你可能会认为text-align: center与<CENTER>元素的作用一样，但实际上二者大不相同
<CENTER>不仅影响文本，还会把整个元素居中。text-align不会控制元素的对齐，而只影响内部内容。
#### 字间距
word-spacing可以改变字之间的标准间隔。word-spacing属性接受一个正长度值或负长度值。
```css
  p.spread {word-spacing: 30px;}
  p.tight {word-spacing: -0.5em;}
  
  <p class="spread">
    This is a paragraph
  </p>
  <p class="tight">
    this is a paragraph
  </p>
```
#### 字母间距
letter-spacing: 字母间隔修改的是字符或字母之间的间隔
```css
  h1 {letter-spacing: -0.5em}
  h4 {letter-spacing: 20px}
  
  <h1>this is a head 1 </h1>
  <h4>this is a head 4 </h4>
```
#### 字符转换
text-transform: 处理文本的大小写。有4个值：
* none: 对文本不做任何改动
* uppercase: 转换为大写
* lowercase: 转换为小写
* capitalize: 只对每个单词的首字母大写
#### 文本装饰
text-decoration有5个值：
* none: 关闭原本应用到一个元素上的所有装饰
* underline: 对元素加下划线
* overline: 对元素加上划线
* line-through: 在文本中间画一个贯穿线
* blink: 让文本闪烁
```css
  <!--所有超链接即有下划线，又有上划线-->
  a:link a:visited {text-decoration: underline overline;}
```
#### 处理空白符
white-space: 对文档中的空格、换行和tab字符的处理
* normal
```css
  p {
    white-space: normal;<!--丢掉多余的空白符，换行字符(回车)会转换为空格，一行中多个空格会转为一个空格-->
  }
  <p>This   paragraph has  
    many spaces    in it</p>
```
* nowrap:防止元素中的文本换行，除非使用了一个br元素
* pre-wrap和pre-line
如果元素white-space设置为pre-wrap，那么该元素中的文本会保留空白符序列，但文本行会正常地换行。pre-line:会合并空白符序列，但保留换行符

### css字体
css字体属性定义文本的字体系列、大小、加粗、风格和变形
#### css字体系列
在css中，有两种不同类型的字体系列名称：
* 通用字体系列 - 拥有相似外观的字体系统组合(如：Serif 或 Monospace)
* 特定字体系列 - 具体的字体系列(Times 或 Courier)
font-family: 定义文本的字体系列
```css
  body {
    font-family: sans-serif Times, 'New York',Georgia;
  }
```
注意，只有当字体名中有一个或多个空格(如：New York)，或字体名包括#或$之类的符号，才需要加引号
#### 字体风格
font-style: 规定斜体文本。该属性有三个值：
* normal - 文本正常显示
* italic - 文本斜体显示
* oblique - 文本倾斜显示
italic是一种简单的字体风格，对每个字母的结构有些小改动，来反映变化的外观。oblique是正常竖直文本的一个倾斜版本
#### 字体变形
font-variant: 设定小型大写字母，即采用不同大小的大写字母
```css
  p {font-variant: small-caps;}
```
#### 字体加粗
font-weight: 设置文本的粗细
使用bold关键字设置文本为粗体
关键字100~900为字体指定9级加粗度。数字400等价于normal,而700等价于bold
#### 字体大小
font-szie: 设置文本的大小，值可以是绝对或相对值
绝对值：将文本设置为指定的大小；不允许用户在所有浏览器中改变文本大小；绝对大小在确定了输出的物理尺寸时很有用
相对大小：相对周围的元素来设置大小；允许用户在浏览器改变文本大小
* 使用em来设置字体大小
1em等于当前的字体尺寸。如果一个元素的font-size为16像素，那么对于该元素，1em就等于16像素。浏览器默认的文本大小是16像素。因此1em
的默认尺寸是16像素。公式：em=pixels/16。(注：16等于父元素的默认字体大小，假设父元素的font-size为20px,那公式：em=pixels/20)
### css链接
#### 设置链接样式
链接特殊性在于能够根据它们所处的状态来设置它们的样式。链接的四种状态：
* a:link - 普通的、未被访问的链接
* a:visited - 用户已访问的链接
* a:hover - 鼠标指针位于链接的上方
* a:active - 链接被点击的时刻
```css
  a:link {color: #ff0000;}
  a:visited {color: #00ff00;}
  a:hover {color: #ff00ff;}
  a:active {color:#0000ff;}
```
当为链接的不同状态设置样式时，请按照以下次序规则：
* a:hover 必须位于a:link 和 a:visited之后
* a:active 必须位于a:hover之后
### css列表
css列表属性允许你放置、改变列表项标志，或者将图像作为列表项标志。
#### 列表类型
如，在一个无序列表中，列表项的标记是出现在各列表项旁边的圆点。在有序列表中，标志可能是字母、数字或另外某种计数体系中的一个符号
要修改用于列表项的标志类型，可以使用list-style-type:
```css
  ul {list-style-type: square}
```
列表项图像list-style-image:
```css
  ul li {list-style-image: url('/images/logo.jpg')}
```
列表标志位置list-style-position
可以将以上3个列表样式属性合并为一个方便的属性：list-style
```css
  li {list-style: url('logo.jpg') square inside}
```
### CSS表格
#### 表格边框
```css
  table,th,td {
    border: 1px solid blue;
  }
```
折叠边框border-collapse: 是否表格边框折叠为单一边框
```css
  table {
    border-collapse: collapse;
  }
  table,th,td {
    border: 1px solid black;
  }
```
| 属性 | 描述 |
|----|----|
| border-collapse | 设置是否把表格边框合并为单一的边框 |
| border-spacing | 设置分割单元格边框的距离 |
| caption-side | 设置表格标题的位置 |
| empty-cells | 设置是否显示表格中的空单元格 |
| tablye-layout | 设置显示单元、行和列的算法 |
### css轮廓
轮廓是绘制于元素周围的一条线，位于边框边缘的外围，可起到突出元素的作用。css outline属性规定元素轮廓的样式、颜色和宽度

| 属性 | 描述 |
|----|----|
| outline | 在一个声明中设置所有的轮廓属性 |
| outline-color | 设置轮廓的颜色 |
| outline-style | 设置轮廓的样式 |
| outline-width | 设置轮廓的宽度 |
## css框模型
### css框模型概述
css框模型(Box Model)规定了元素框处理元素内容、内边距、边距和外边距的方式
#### 图解
![](http://www.w3school.com.cn/i/ct_boxmodel.gif)

提示：背景应用于由内容和内边距、边框组成的区域。
在css中，width和height指的是内容区域的宽度和高度。增加内边距、边框和外边距不会影响内容区域的尺寸但是会增加元素框的总尺寸
#### css内边距
css padding属性定义元素边框与元素内容之间的空白区域
* padding: 接受长度值或百分比值，但不允许使用负值。
```css
  <!--h1元素各边都有10像素的内边距-->
  h1 {padding: 10px;}
  <!--可以按照上、右、下、左的顺序分别设置各边的内边距-->
  h1 {padding: 10px 0.25em 2ex 20%}
```
* 单边内边距属性
```css
  h1 {
    padding-top: 10px;
    padding-right: 0.25em;
    padding-bottom: 2ex;
    padding-left: 20%;
  }
```
### css边框
css border属性允许你规定元素边框的样式、宽度和颜色
#### 边框样式
border-style属性定义了10个不同的非inherit样式，包括none
|值|描述|
|--|---|
|none|定义无边框|
|hidden|与"none"相同。不过应用于表时除外，对于表，hidden用于解决边框冲突|
|dotted|定义点状边框|
|dashed|定义虚线|
|solid|定义实线|
|double|定义双线|
|groove|定义3D凹槽边框。其效果取决于border-color的值|
|ridge|定义3D垄状边框。其效果取决于border-color的值|
|inset|定义3D inset边框。其效果取决于border-color的值|
|outset|定义3D outset边框。其效果取决于border-color的值|
|inherit|规定应该从父元素继承边框样式|

```css
  p.aside {border-style: solid dotted dashed double;}
```
上面，类名为aside定义了四种边框样式：实线上边框、点线右边框、虚线下边框和双线左边框
#### 定义单边样式
border-top-style,border-right-style,border-bottom-style,border-left-style
```css
  p {border-style: solid solid solid none;}
  <!--等价于-->
  p {border-style: solid;border-left-style: none;}
```
注意：如果使用第二种方法，必须把单边属性放在简写属性之后。
#### 边框的宽度
为边框指定宽度有两种方法：可以指定长度值，如2px或0.1em；或使用3个关键字之一，如：thin、medium、thick
注释：css没有定义3个关键字的具体宽度。
```css
  p {border-style: solid; border-width: 15px 5px 15px 5px;}
  或简写为(值复制)
  p {border-style: solid; border-width: 15px 5px;}
```
### css margin属性
margin属性可以接受任何长度单位，可以是像素、英寸、毫米或em
```css
  h1 {margin: 0.25in}
  h2 {margin: 10px 0px 15px 5px}
  h3 {margin: 10%}
```
单边外边距属性：margin-top,margin-right,margin-bottom,margin-left
### css 外边距合并
外边距合并指的是，当两个垂直外边距相遇时，它们将形成一个外边距。合并后的外边距的高度等于两个发生合并的外边距
的高度中的较大者
当一个元素出现在另一个元素上面时，第一个元素的下外边距与第二个元素上外边距会发生合并。如图：
![](http://www.w3school.com.cn/i/ct_css_margin_collapsing_example_1.gif)
当一个元素包含在另一个元素中时(假设没有内边距或边框把外边距分隔开)，它们上/下外边距会发生合并，如图：
![](http://www.w3school.com.cn/i/ct_css_margin_collapsing_example_2.gif)
## css 定位
css定位属性允许你对元素进行定位
定位的思想：允许你定义元素框相对于其正常位置应该出现的位置，或者相对于父元素、另一个元素甚至浏览器窗口本身的位置。
#### 1.一切皆为框
div、h1或p元素常常被称为块级元素。这意味着这些元素显示为一块内容。与之相反，span和strong等元素称为"行内元素"
可以使用display属性改变生成的框的类型
|值|描述|
|--|--|
|none|此元素不会被显示|
|block|此元素将显示为块级元素，此元素前后会带有换行符|
|inline|默认。此元素会被显示为内联元素，元素前后没有换行符|
|inline-block|行内块元素(css2.1 新增)|
|list-item|此元素会作为列表显示|
|run-in|此元素会根据上下文作为块级元素或内联元素显示|
|table|此元素会作为块级表格来显示(类似<table>),表格前后带有换行符|
|inline-table|此元素会作为内联表格来显示(类似<table>),表格前后没有换行符|
|table-row-group|此元素会作为一个或多个行的分组来显示(类似<tbody>)|
|table-header-group|此元素会作为一个或多个行的分组来显示(类似<thead>)|
|table-footer-group|此元素会作为一个或多个行的分组来显示(类似<tfoot>)|
|table-row|此元素会作为一个表格行显示(类似<tr>)|
|table-column-group|此元素会作为一个或多个列的分组来显示(类似<colgroup>)|
|table-column|此元素会作为一个单元格列显示(类似<col>)|
|table-cell|此元素会作为一个表格单元格显示(类似<td>和<th>)|
|table-caption|此元素会作为一个表格标题显示(类似<caption>)|
|inherit|规定应该从父元素继承display属性的值|
  
#### 2.css定位机制
css有三种基本的定位机制：普通流、浮动和绝对定位。
所有框默认都在普通流中定位，也就是说，普通流中的元素的位置由元素在(X)HTML中的位置决定。
块级框从上到下一个接一个地排列，框之间的垂直距离是由框的垂直外边距计算出来
行内框在一行中水平布置。可以使用水平内边距、边框和外边距调整它们的间距。但是，垂直内边距、边框和外边距不影响行内框的高度。由一行形成的水平框
称为行框，行框的高度总是足以容纳它包含的所有行内框。不过，设置行高可以增加这个框的高度
#### 3.position属性
* static: 元素框正常生成。块级元素生成一个矩形框，作为文档流的一部分，行内元素则会创建一个或多个行框，置于其父元素中。
* relative: 元素框偏移某个距离
* absolute: 元素框从文档流完全删除，并相对于其包含块定位。包含块可能是文档中的另一个元素或者时初始包含块。
* fixed: 元素框的表现类似absolute，不过其包含块是视窗本身。




















# 感谢
[w3school](http://www.w3school.com.cn/css/index.asp)
