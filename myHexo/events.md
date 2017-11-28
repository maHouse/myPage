---
title: 事件处理
date: 2017-08-14 23:13:04
tags:
---
事件就是Web浏览器通知应用程序发生了什么事情，这一节我们从几个定义开始》》

事件类型（event type）

是一个说明发生什么类型事件的字符串，就是发生了什么事情，这个事情的名字叫做事件类型，也可以称为事件名字（event name）。

事件目标（event target）是发生的事件或与之相关的对象。当讲事件时，我们必须同时指明类型和目标，Window、Document和Element对象是最常见的事件目标，但某些事件是由其它类型的对象触发。如XMLHttpRequest对象触发的readystatechange事件。

事件处理程序（event handler）或事件监听程序（event listener）是处理或响应事件的函数。它通过指明事件类型和时间目标，在Web浏览器中注册他们的事件处理程序函数。在特定的目标上发生特定类型的事件时，浏览器会调用对应的处理程序，当对象上注册的事件处理程序被调用时，我们会说浏览器触发（fire、trigger）和派发（dispatch）了事件。

事件对象（event object）是与特定事件相关且包含有关事件详细信息的对象。事件对象作为参数传递给事件处理程序函数。它本身具有事件类型的type和指定时间目标的target属性。如鼠标事件的相关对象包含鼠标指针的坐标。

事件传播（event propagation）是浏览器决定哪个对象触发其事件处理程序的过程。对于像Window对象的load事件的单个对象的特定事件，必须是不能传播的。当文档元素上发生某个类型的事件时，它会在文档树上向上传播或冒泡（bubble）。事件处理程序能通过调用方法或设置事件对象属性来阻止事件传播，就能阻止冒泡且将无法在容器元素上触发处理程序。

事件传播的另一种形式是事件捕获（event capturing），在容器上注册特定处理程序有机会在事件传播到真是目标之前拦截（捕获）它。IE8之前的版本不支持这个技术。在鼠标的拖放事件中，这个能力是必须的。

### 一、扼要须知 ###
mouseover和mouseout的鼠标事件是有冒泡行为的，但这是不方便的，因此在IE中出现了这些事件不冒泡的版本mouseenter和mouseleave。

#### 1、触摸屏和移动设备事件 ####
这里需要建立新的事件类型，一开始是由触摸屏事件映射到传统的事件类型，但不总是有效。这里主要说Apple和iPad上Safari产生的手势和触摸事件，还有旋转这些设备时产生的orientationchange事件。

Safari的手势事件用于两个手指的缩放和旋转手势。开始时，生成gesturestart事件，结束生成gestureend事件，之间是跟踪手势过程的gesturechange事件队列。这些事件传递对象有属性scale和rotation。scale属性是两个手指之间当前距离和初始距离的比值。“捏紧”手势的scale值小于0，撑开手势的scale值大于1.rotation属性是指从事件开始手指旋转的角度，他以度为单位，正值表示按顺时针旋转。

手势是高级事件，用于已经翻译的手势。要想实现自定义手势，你可以监听低级触摸事件。触摸时触发touchstart事件，移动时触发touchmove事件，手指离开屏幕时触发touchend事件，触摸事件没有触摸坐标。但触摸事件传递的事件对象有一个changeTouches属性，是一个类数组对象，每个元素描述触摸的位置。

当设备允许用户从竖屏旋转到横屏模式时会在Window对象上出发orientationchange事件，此事件本身无用，但在移动版的Safari中，Window对象的orientation属性能给出当前的方位，其值是0、90、180或-90。
### 二、注册事件处理程序 ###
注册事件处理程序有两种基本方式，一是给事件目标或文档元素设置属性。二是更通用的，将事件处理程序传递给对象或元素的一个额方法。下面细说。

#### 1、设置JavaScript对象属性为事件处理程序 ####
事件处理程序属性的名字由“on”后面跟着时间名字组成，都是小写：onclick、onchange、onload、onmouseover、readystatechange。注册事例：

	window.onload = function() {
		var elt = document.getElementById("shiping_address");
		elt.onsubmit = function() { return validate(this);}
	}
事件处理程序属性的缺点是设计都围绕假设每个时间目标对于每种事件类型将最多只有一个处理程序。想编写能够在任意文档中都能使用的脚本代码，更好的方式是使用一种不修改或覆盖任何已有注册处理程序的技术（如addEventListener）。
#### 2、设置HTML标签属性为事件处理程序 ####
不建议使用不做探讨。
#### 3、addEventListener() ####
IE8之前的浏览器都支持的标准事件模型，任何能成为事件目标的对象（Window对象、Document对象和所有的文档元素）都定义了一个叫做addEventListener()的方法，使用这个方法可以为事件目标注册处理程序。addEventListener()接受3个参数。第一个是要注册处理程序的事件类型，不包括“on”前缀，是个字符串形式，第二个参数是指定类型的事件发生时应该调用的函数。最后一个是BooLean。通常情况下是false，如果是true，函数将注册为捕获事件处理程序，并在事件的不同阶段调用。可以忽略第三个参数，不允许忽略的带上即可。

下面的代码在button上注册了click事件的两个处理程序，技术不同：

	<button id="my button">click me</button>
	<script>
	var b = document.getElementById("mybutton");
	b.onclick = function() { alert("Thanks for clicking me");};
	b.addEventListener("click",function() {alert("Thanks again!");},false);
	</script>
两者不相互影响，addEventListener()可以为一个对象一个事件注册多个程序函数，事件发生时按顺序进行。相同的参数在同一个对象上多次调用这个方法是不行的。相对的就是removeEventListener()方法，同样的道理删除事件，即注销事件。
#### 4、attachEvent() ####
IE9之前的IE不支持addEventListener()和removeEventListener(),IE5以后的定义了attachEvent()和detachEvent(),工作方法类似但是有些不同：

①IE事件不支持事件捕获，所以只有两个参数：事件类型和事件处理程序。

②IE方法的第一个参数使用了带“on”前缀的事件处理程序属性名，不是没有前缀。

③attachEvent()允许相同的事件处理程序函数多次注册。

经常看到的是这样子的：

	var b = document.getElementById("mybutton");
	var handler = function() {alert("Thanks!");};
	if(b.addEventListener)
		b.addEventListener("click",handler,false);
	else if(b.attachEvent)
		b.attachEvent("onclick",handler);
这方法的调用中，注册程序可能是按照任何程序，不同于addEventListener()。
### 三、事件程序的调用 ###
#### 1、事件传播 ####
当事件目标是元素或元素时，比较复杂。当调用在目标元素上的事件处理程序函数后，大部分事件会“冒泡”到DOM树根，最后调用在目标元素的祖父元素上注册的事件处理程序。这会一直到Document对象，最后到Window对象。可以在共同的祖先元素上注册一个处理函数来代替。

不会冒泡的事件有focus、blur和scroll事件，文档元素的load事件会冒泡到Document对象上不会到Window对象上。

事件冒泡是事件传播的第三个阶段，目标对象本身的事件处理程序调用是第二个阶段。第一个阶段是目标程序调用之前，称之为捕获阶段。
#### 2、事件取消 ####
支持AddEventListener()的浏览器中，能通过事件对象的preventDefault()来取消事件的默认操作，在IE9之前可以用事件对象的returnValue = false来达到同样的效果。三种方法：

	function cancelHandler(event) {
		var event = event || window.event;//用于IE
		if(event.preventDefault) event.preventDefault();//标准
		if(event.returnValue) event.returnValue = false;//IE
		return false;//用于处理使用对象属性注册的处理程序
	}
我们也能取消事件传播。在支持addEventListener()的浏览器中，可以用stopPropagation()来阻止事件的继续传播，在事件传播的任何时间（3个阶段）调用。在IE9之前中可以使用cancelBubble属性为true来阻止事件的进一步传播。
### 四、文档加载事件 ###
从load事件开始，大部分的Web应用需要浏览器告诉它们文档加载完毕和为操作准备就绪的时间，load事件指导文档和所有图片加载完毕后才发生。所以基于load事件之前的事件的触发会提升Web应用的启动时间。

当文档加载解析完毕并且所有延迟脚本执行完毕后会触发DOMContentLoaded事件，此时图片和异步脚本可能还在加载，而文档已经就绪了。

document.readyState属性会随着文档加载过程而改变，在IE中，每次状态改变都伴随着Document对象上的readystatechange事件，但是它仅仅在load事件之前立即触发，不清楚监听readystatechange取代load会有多大好处。

