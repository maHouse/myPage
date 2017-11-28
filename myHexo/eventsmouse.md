---
title: 事件处理程序（二）
date: 2017-08-16 20:57:45
tags:
---
### 五、鼠标事件 ###
除了mouseenter和mouseleave之外的所有事件都能冒泡，dbload用户双击鼠标触发。传递给鼠标事件处理程序的事件对象有clientX和clientY属性，它们指定了鼠标指针相对于包含鼠标的坐标，加入窗口的滚动偏移量就可以把鼠标位置转换为文档坐标。

altKey、ctrlKey、metaKey和shiftKey属性指定了当事件发生时是否有各种键盘助键按下。drag()接受两个参数，第一个是要拖动的元素，它可以是要发生mousedown事件的元素或包含元素。无论哪种，它必须是CSS position属性的绝对值定位的文档元素。第二个参数是触发mousedown事件对象。如下是定义了用户在按下shift键时能拖动的img：
	
	<img src="draggable.gif" style="position:absolute;left:100px;top:100px;"onmousedown="if(event.shiftKey) drag(this,event);">
drag()注册了接着mousedown事件发生的mousemove和mouseup事件的事件处理程序。mousemove事件处理程序用于响应文档元素的移动，而mouseuo事件处理程序用于注销自己和mousemove事件处理程序。这两者是处理程序注册为捕获事件处理程序。但IE事件模型没有捕获过程，而自己有个setCapture()方法。

for example 1：
	
	//拖动文档元素
	//Drag.js：拖动绝对定位的HTML元素
	//这个模块定义了一个drag()函数，它用于mousedown事件处理程序的调用
	//随后的mousemove事件将移动指定元素，mouseup事件将终止拖动
	//这些实现能同标准和IE两种事件模型一起工作
	//他需要用到已经介绍的getScrollOffsets()方法
	//参数：elementToDrag：接收mousedown事件的元素或某些包含元素
	//他必须是绝对定位的元素
	//它的style.left和style.top值将随着用户的拖动而改变

	function drag(elementToDrag,event) {
		
		//初始鼠标位置，转换为文档坐标
		var scroll = getScrollOffsets();//来自其他地方的工具函数
		var startX = event.clientX + scroll.x;
		var startY = event.clientY + scroll.y;
		
		//在文档坐标下，待拖动元素的初始位置
		//因为elementToDrag是绝对定位的
		//所以我们假设它的offsetParent就是文档的body元素
		var origX = elementToDrag.offsetLeft;
		var origY = elementToDrag.offsetTop;
		
		//计算mousedown事件和元素左上角之间的距离
		//我们将它另存为鼠标移动的距离
		var deltaX = startX - origX;
		var deltaY = startY - origY;
		
		//计算mousedown事件和元素左上角之间的距离
		//我们将它另存为鼠标移动的距离
		if(document.addEventListener) {//标准事件模型
			
			//在document对象上注册捕获事件处理程序	
			document.addEventListener("mosemove",moveHandler,true);
			document.addEventListener("moseup",upHandler,true);
		}
		else if(document.attachEvent) {//IE5到8的IE事件模型

			//在IE事件模型中
			//捕获事件是通过调用元素上的setCapture()捕获它们
			elementToDrag.setCapture();
			elementToDrag.attachEvent("onmosemove",moveHandler);
			elementToDrag.attachEvent("onmoseup",upHandler);
			elementToDrag.attachEvent("onmoseup",upHandler);
		}

		//我们处理了这个事件，不让其他元素看到它
		if (event.stopPropagation) event.stopPropagation();
		else event.cancleBubble = true;
		
		//现在阻止任何默认操作
		if (event.preventDefault) event.preventDefault();
		else event.returnValue = false;

		//当元素正在被拖动时，这就是捕获mousemove事件的处理程序
		//它用于移动这个元素
		function moveHandler(e) {
			if(!e) e = window.event;//IE事件模型
		
			//移动这个元素到当前鼠标位置
			//通过滚动条的位置和初始单击的偏移量来调整
			var scroll = getScrollOffsets();
			elementTopDrag.style.left = (e.clientX + scroll.x - deltaX) + "px";
			elementTopDrag.style.top = (e.clientY + scroll.y - deltaY) + "px";
	
			//同时不让任何其他元素看到这个事件
			if (e.stopPropagation) e.stopPropagation();//标准
			else e.cancelBubble = true;//IE
		}

		//这是捕获在拖动结束时发生的最终mouseup事件的处理程序
		function upHandler(e) {
			if (!e) e = window.event;//IE事件模型
			
			//注销捕获事件处理程序
			if(document.removeEventListener) {//DOM事件模型
				document.removeEventListener("mouseup",upHandler,true);
				document.removeEventListener("mousemove",moveHandler,true);
			}
			else if(document.detachEvent) {//IE5+事件模型
				elementToDrag.detachEvent("onlosecapture",upHadler);
				elementToDrag.detachEvent("onmouseup",upHadler);
				elementToDrag.detachEvent("onmousemove",upHadler);
				elementToDrag.releaseCapture();
			}
			
			//并不让事件进一步传播
			if(e.stopPropagation) e.stopPropagation();
			else e.cancelBUbble = true;
		}
	}

下面的代码展示了在HTML中怎么使用drag():

	<script src="getScrollOffsets.js"></script><!--drag()需要这个-->
	<script src="Drag.js"></script><!--定义drag()-->
	<div style="position:absolute;left:100px;top:100px;width:200px;background-color:white;border:solid black;">
	<!--要拖动的元素-->
	<div style="background-color:gray;border-bottom:dotted black;padding:3px;font-family:sans-serif;font-weight:bold;"onmousedown="drag(this.parentNode,event);">
	<!--拖动我，标题栏的内容-->
	</div>
	<!--可拖动元素的内容-->
	<p>这是一个测试，仅仅是个测试</p>
	</div>
这里的关键是内部div元素的onmousedown属性，注意这里使用this.parentNode指定整个容器元素将被拖动。