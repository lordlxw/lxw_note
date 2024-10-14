# Html&Css

## 一、Html



> Html:
>
> 块级元素：有些元素总是在浏览器窗口显示时另起一行，常用的块级元素有<h1>、<p>、<ul>、<li>等



> 	<div>  标签允许你将一组元素集中到一个块级元素内    
>
> 
>
> <span>元素是<div>的内联版本，可以在其上的class或id特性上应用css样式。
>
> 
>
> <meta> 元素位于<head>元素中 他对用户不可见，功能却很强大，能把页面相关信息告诉搜索引擎，最常用的是name特性和content特性，name即属性，content即给这属性设定的值
>
> ​    常见的meta 属性有：
>
> * description：包含一段有关页面的描述信息
> * robots：用于指定搜索引擎是否可以将此页面加入他们搜索结果中
> * keywords：用于包含一组以逗号分隔的关键词列表，这个现在不常用的
> * http-equiv特性：可以设置author、pragma、expires

> *lorem100* 将生成 100 字数的假文



## 二、CSS样式

> 写在html里的样式 
>
> html中的head里添加<link>告诉浏览器在何处寻找用于定义样式的css文件
>
> 三种添加css的方法：
>
> 1、最常见的link（外部）
>
> <head>
>  <link href="css/styles.css" type="text/css" rel="stylesheet"/>
> </head>
> 其中href指定css文件位置，type指明所链接的文档类型，值应该是text/css，rel特性表明html页面与被链接文件的关系。当链接到一个css文件时，该特性应为stylesheet
>
> 2、内部样式表
>
> 将css放在html里的style里面
>
> 3、内联样式
>
> 只影响一个元素，也是在style里面添加

### 1、CSS选择器

> css常用选择器：
>
> 通用选择器、类型选择器、类选择器、ID选择器、子元素选择器、后代选择器、相邻兄弟选择器、普通兄弟选择器
>
> 通用选择器：*{}  ——应用页面的所有元素
>
> 类型选择器：h1,h2,h3{} ——应用<h1>\<h2><h3>元素

> 简而言之：
>
> .类
>
> #ID
>
> 例：<p class="box1" id="para1">Lorem</p>
>
> .box1{样式}
>
> #para1{样式}
>
> 也可以：
>
> .listdemo li 说明在listdemo类里设置li的属性  

-----------------------------------------------------------------

| 示例  | 说明                           |
| ----- | ------------------------------ |
| div,p | 选择所有<div>元素和<p>元素     |
| div>p | 选择所有<p>元素的父级<div>元素 |
| div p | 选择<div>元素内的所有<p>元素   |

> *：选择所有

> 特性选择器：
>
> []
>
> p[class="dog"]
>
> p[class~="dog"] ）——class的特性值其中有一项是dog
>
> p[attr^"d"]——某个特性值以d开头
>
> p[attr*"do"]——某个特性值中含有“do”
>
> p[attr$"g"]——某个特性以字母g结尾

### 2、CSS定位

> static-静态定位
>
> relative-相对定位
>
> absolute-绝对定位
>
> fixed-固定定位
