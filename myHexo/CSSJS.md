---
title: CSS脚本化
date: 2017-08-14 20:14:30
tags:
---
### 一、概览CSS ###
HTML文档的视觉显示包含很多变量：字体、颜色、间距等，CSS标准列举了这些变量，我们称为样式属性。CSS定义了这些属性以指定字体、颜色、外边距、边框、背景图片、文本对齐方式、元素尺寸和元素位置。为了定义HTML元素的视觉表现，规定了这些CSS属性的值。

CSS样式称之为层叠样式表，是可以一个属性压着另一个属性的，样式来源于：

Web浏览器的默认样式、文档的样式表和每个独立的HTML元素的style属性。
### 二、重要的CSS属性 ###
重要的CSS属性是指定文档中每个元素的可见性、尺寸和精确定位的属性。其他CSS属性允许指定堆叠次序、透明度、裁剪区域、外边距、内边距、边框和颜色。
#### 1、CSS定位元素 ####
#### position属性4个值： ####

**static**是默认值，属于标准文档流；

**absolute**是绝对定位，相对于距离它最近的父元素定位，这个定位脱离了标准文档流。如果最近父元素没有定位，则相当于文档定位；

**fixed**固定定位是相对于浏览器定位的，总是在那里，不会变化自己的位置，不是文档的一部分。注意IE6 不支持这个方法。

**relative**仍属于标准文档流。

有了定位，就有了他们的值，top、bottom、left、right。
#### 第三个维度：z-index ####
它是垂直说的，既然是垂直的，那么元素一定是层叠在一起了，即是有定位在里面，还是脱离了标准文档流。需要这个值来标出文档元素出现的顺序。

**说说CSS盒子模型**

这里我们必须清楚，盒子模型有两种，一种是标准盒子模型，内容的width、height及内边距和边框及外边距，另一种就是不标准的属于IE的，内容content包括了内边距和边框，有点尴尬了吧！

这里就地说到box-sizing:border-box了，只要执行了这个就会应用IE的盒子模型。当以百分比形式为元素设置总体尺寸，又想以像素指定边框和内边距时，这个盒子模型很有用：

	<div style="box-sizing; width:50%; padding:10px;border:2px;">
#### 2、元素显示和可见性 ####
影响元素的可见性：visibility和display。visibility设置为hidden时，不可见；设置为visible可见。display有三个了，一个是块状元素、一个是内联元素，再有就是内敛块元素，设置为display为none时，则为不可见。前者使用时，无论哪个值空间是必须占有的，后者不同，所以后者更适合用于创建开展和折叠轮廓的效果。两者对定位元素的影响是等价的，不过，首选visibility。
#### 3、颜色、透明度和半透明度 ####
记住设置半透明度时候，一般情况是：`opacity：.75;`(透明度，这个是CSS3标准属性),但是filter是针对于IE浏览器的，这个必须得写出来`filter:alpha(opacity=75);`(这个没有小数点)。
#### 4、部分可见：overflow和clip ####
overflow4个值，visible（默认值，可见的）、hidden（隐藏的）、scroll（超出的部分用滚动条）、auto（滚动条只在超出元素尺寸时显示，而非一直显示）。

clip属性的值指定了元素的裁剪区域。语法是：rect(top right bottom left)

	style="clip:rect(0px 100px 100px 0px);"//不允许使用百分比
也可以
	style="clip:rect(auto 100px 100px auto);"
值之间没有逗号，裁剪区域从上边缘开始顺时针设置，将clip设置为auto来停用裁剪功能.
### 三、脚本化样式 ###
可以用JavaScript代码显示如下的相应的CSS属性：
	e.style.fontSize = "24px";
	e.style.fontWeight = "bold";
	e.style.color = "red";
但是不能用

	e.style.font-size = "24px";//这是个错误的写法，切记
定位的属性都包含单位

	e.style.left = 300;//不合适，因为是数字不是字符串
	e.style.left = "300";//不合适，因为缺少单位
	e.style.left = "300px";//正确
	e.style.left = (left_margin + left_border + left_padding) + "px";
脚本化CSS类，就死className和JavaScript的合用。
