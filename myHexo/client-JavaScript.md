---
title: Web浏览器中的JavaScript
date: 2017-07-18 21:01:24
tags:
---
前一部分介绍了JavaScript语言核心，这一部分转向Web浏览器中JavaScript的实现。web浏览器是如何呈现页面的呢？一些呈现静态信息的页面叫做文档(加入JavaScript会有动态的效果但是信息本身是静态的，不过页面可以动态的载入信息)。下面会说到JavaScript代码是如何在HTML文档中嵌入并执行的，还会介绍兼容性、可访问性和安全性等问题。

### 一、客户端JavaScript ###
Window对象是所有客户端JavaScript特性和API的主要接入点。Window对象定义了一些属性，指代Location对象的location属性，Location对象指定当前显示在窗口里载入新的URL。
	
	//设置location属性，从而跳转到新的web页面
	window.location = "http://www.oreilly.com/";

Window对象还定义了一些方法，比如alert()方法，还有setTimeout()，可以注册一个函数，在给定的一段时间后触发一个回调：

	//等待两秒，然后说 hello
	setTimeout(function() {alert("hello JavaScript!");},2000);
客户端JavaScript中，Window对象也是全局对象，Window对象处于作用域的顶端，它的属性和方法实际上是全局变量和全局函数。Window对象有一个引用自身属性叫做window。

Window对象有一个最重要的属性是document，它引用Document对象，后者表示显示在窗口中的文档。Document对象有一些重要方法，比如getElementById(),可以基于元素id属性的值返回单一的文档元素：

	//查找 id = "timestamp"的元素
	var timestamp = document.getElementById("timestamp");
getElementById()返回的Element对象有其他重要的属性和方法，比如允许脚本获取它的内容，设置属性值等：

	//如果元素为空，往里面插入当前的日期和时间
	if(timestamp.firstChild == null)
		timestamp.appendChild(document.createTextNode(new Date().toString()));
每个Element对象都有style和className属性，允许脚本指定文档元素的CSS样式，或者修改应用元素上的CSS类名。设置这些CSS相关属性会改变文档元素的呈现：

	//显示修改目标元素的呈现
	timestamp.style.backgroundColor = "yellow";

	//或者只改变类，让样式表指定具体内容
	timestamp.className = "highlight";
事件处理程序可以让JavaScript代码修改窗口、文档和组成文档的元素的行为。事件处理程序的属性名是以单词“on”开始的：

	//当用户单机timestamp元素时，更新它的内容
	timestamp.onclick = function () {this .innerHTML = new Date().toString();}

	//显示内容的简单客户端JavaScript：
	<!DOCTYPE html>
	<html>
	<head>
	<style>
	.reveal*{display:none;}
	.reveal *.handle{display:block;}<!...除了这个handle的元素...>
	</style>
	
	<script>
	window.onload = function() {
		var elements = document.getElementsByName("reveal");
		for (var i = 0;i < elements.length;i++) {
			var elt = elements[i];
			var title = elt.getElementsByName("handle")[0];
			addRevealHandler(title,elt);
			function addRevealHandler(title,elt) {
				title.onclick = function () {
					if(elt.className == "reveal")
						elt.calssName = "ravealed";
					else if (elt.className == "revealed")
						elt.className = "reveal";
				}
			}
		}
	}
	</script>
    </head>
	<body>
	<div class="reveal">
	<h1 class="handle"> Click Here to Reveal Hidden Text</h1>
	<p>This paragraph is hidden.It appears when you click on the title.</p>
	</div>
	</body>
	</html>
#### 1、web文档里的JavaScript ####
用户的体验不应该依赖于JavaScript，但它却可以增强用户体验，通过如下的方式来实现：

》创建动画和其他视觉效果，巧妙的引导和帮助用户进行页面导航。

》对表格的列进行分组，让用户更容易找到所需要的。

》隐藏某些内容，当用户“深入”到内容里时，在逐渐展示详细信息。

Web浏览器是简单操作系统的概念，这样就可以把Web应用定义为用JavaScript访问更多浏览器提供的高级服务(比如网络、图像和数据存储)的页面。高级服务里最有名的是XMLHttpRequest对象，后者可以对HTTP请求编程来启用网络。Web应用使用这个服务从服务器获取新信息，而不用重新载入页面，达到局部刷新的效果。
### 二、在HTML里嵌入JavaScript ###
嵌入到HTML文档里共有4种方式：

《 内联，放置在`<script></script>`之间。

《 放置在由`<script>`标签的src属性指定的外部文件中。

《放置在HTML事件处理程序中，该事件处理程序由onclick或者onmouseover这样的HTML属性值指定。

《放在一个URL里，这个URL使用特殊的“javascript”协议。

嵌入到HTML文档里方法总共是这几种，但是我们常用的是第二种，达到了表现形式与行为的分离的目的。

	//这是个简单的JavaScript数字时钟程序
	<!DOCTYPE html>
	<html>
	<head>
	<title>
	Digital Clock
	</title>
	<script>
	//定义一个函数用以显示当前的时间
	function displayTime() {
		var elt = document.getElementById("clock");
		var now = new Date();
		elt.innerHTML = now.toLocaleTimeString();
		setTimeout(displayTime,1000);
	}
	window.onload = displayTime;
	</script>
	<style>
	#clock {
		font:bold 24pt sans;
		background:#ddf;
		padding:10px;
		border:solid black 2px;
		border-radius:10px;
	}
	</style>
	</head>
	<body>
	<h1>Digital Clock</h1>
	<span id="clock"></span>
	</body>
	</html>
其他的引入方式这里不再做介绍，下面我们仅仅说一下外部文件中的脚本：

	<script src="../../scripts/util.js"></script>//相对路径
### 三、JavaScript程序的执行 ###
JavaScript程序的执行有2个阶段：

第一阶段，载入文档内容，并执行`<script>`元素里的代码(包括内联的和外部脚本)。脚本通常会按它们在文档中出现的顺序执行。所有脚本里的JavaScript代码都是从上到下，按照它在条件、循环以及其他控制语句中的出现顺序执行。这个阶段是JavaScript代码自己的执行阶段，不参与与HTML文档的互动工作。

当文档载入完成，并且所有的脚本执行完成后，JavaScript就进入它的第二阶段。这个阶段是异步的，而且由事件驱动。在事件驱动阶段，Web浏览器调用事件处理程序函数(第一阶段里执行的脚本指定的HTML事件处理程序)，来响应异步发生的事件。这个阶段就是JavaScript代码与HTML文档的互动，不过这里面由事件驱动罢了！

事件驱动阶段发生的第一个事件是load事件，这是说明文档加载完成。因为JavaScript代码的执行是单线程的，所以这两个阶段的执行在同一个时间内只能执行一个操作。同样只有当JavaScript代码执行完成之后，文档才能继续下去。

JavaScript代码有个快速生成内容的方式就是通过document.write()来实现：

	//载入时生成文档内容
	<h1>Table of Factorials</h1>
	<script>
	function factorial(n) {
		if(n <= 1) return n;
		else return n*fatorial(n-1);
	}
	document.write('<table>');
	document.write('<tr><th>n</th><th>n!</th></tr>');
	for(var i = 1;i <= 10;i++) {
		document.write('<tr><td>' + i + '</td><td>' + fatorial(i) + '</td></tr>');
	}
	document.write('</table>');
	document.write('Generated at' + new Date());//输出时间戳
	</script>
当脚本把文本传递给document.write()时，这个文本被添加到文档输入流中，HTML会创建一个文本节点插入到这个文本节点之后，我们不推荐这么使用。脚本的执行只在默认的情况下是同步和阻塞的。要有所改变我们可以当文本加载完成之后执行代码，也可以在文档加载之前尽快执行代码。前者就是defer的使用意义，后者就是async的使用意义，但是这两者的共同使用则是采用async属性。这两者就是JavaScript代码的同步、异步和延迟。不同的是延迟脚本的执行同样是按顺序走的，而异步脚本则可能是无序执行的。

动态创建脚本并把它插入到文档中来实现脚本的异步载入和执行，通过loadasync()函数来实现这个工作。

	//异步载入并执行脚本
	function loadasync() {
		var head = document.getElementsByName('head')[0];//找到<head>元素
		var s = document.createElement('script');//创建一个<script>元素
		s.src = url;//设置其src属性
		head.appendChild(s);//将script元素插入到head标签中
	}
注意：这个loadasync()函数会动态的载入脚本成为正在执行的JavaScript程序的一部分，不是通过web页面内联包含也不是来自页面的静态引用。
### 1、事件的驱动JavaScript ###
事件都有名字比如click、change、load、mouseover、keypress或者readystatechange，指示发生的事件的通用类型。事件还有目标，就是对象，意思是哪个对象发生了什么事情，所以当我们谈到事件的时候，必须同时包含事件类型（名字）和目标（对象）：比如一个单击事件发生在HTMLButtonElement对象上，或者一个readystatechange事件发生在XMLHttpRequest对象上。

如果想要程序响应一个事件，写一个函数，叫做“事件处理程序”、“事件监听器”或者“回调”。然后注册这个函数，这样当事件发生时就会调用它。可以如下写代码：

	window.onload = function () {...};
	document.getElementById("button1").onclick = function () {..};
	function handleResponse() {...}
	request.onreadystatechange = handleRequest;
对于大部分事件来说，会把一个对象传递给事件处理程序作为参数，那个对象的属性提供了事件的详细信息。比如传递给单击事件的对象，会有一个属性说明鼠标的哪个按钮被单击(IE里面事件信息被储存在全局event对象里)。

有些事件的目标是文档元素，它们会经常往上传递给文档树，这个过程叫做“冒泡”，例如用户在`<button>`元素上单击鼠标，单击事件就会在按钮上触发。如果注册在按钮上的函数没有处理（并且冒泡停止）该事件，事件会冒泡到按钮嵌套的容器元素，这样任何注册在容器元素上的单击事件都会调用。

大部分可以成为事件目标的对象都有一个叫做addEventListener()的方法，允许注册多个监听器：

	window.addEventListaner("load",function () {...},false);
	request.addEventListener("readystatechange",function () {...},false);
这个函数的第一个参数是事件的名称，不好意思的是在IE中，IE8之前的版本中只能运用一个相似的办法，用attachEvent():

	window.attachEvent("onload",function () {...});
setTimeout()函数叫做“回调逻辑”不是“事件处理程序”，但它和setInterval()都是异步的。

	//当文档载入完成时调用一个函数
	//注册函数f，当文档载入完成时执行这个函数f
	//如果文档已经载入完成，尽快已异步方式执行它
	function onLoad(f) {
		if(onLoad.loaded)//如果文档已经载入完成
			window.setTimeout(f,0);//将f放入异步队列，并尽快执行它
		else if(window.addEventListener)//注册事件的标准方法
			window.addEventListener("loade",f,false);
		else if(window.attachEvent)//IE8以及更早的IE版本浏览器注册事件的方法
			window.attachEvent("onload",f);
	}
	//给onLoad设置一个标志，用来指示文档是否完成
	onLoad.loaded = false;
	//注册一个函数，当文档载入完成时设置这个标志
	onLoad(function () {onLoad.loaded = true;});
### 四、兼容性和互用性 ###
客户端JavaScript兼容性和交互性的问题可以归纳为3类：
#### 演化 ####
Web平台在进化当中，什么标准都在进化
#### 未实现 ####
没有实现，是浏览器开发商没有实现某个特性，比如`<canvas>`元素，IE8就不支持，而其他浏览器就已经实现了
#### bug ####
每个浏览器都有bug

#### 解决兼容性问题的办法 ####
1、

处理兼容性问题的方法之一是使用类库，比如jQuery，因为它们定义了新的客户端API并兼容所有浏览器。jQuery中事件处理程序的注册是使用bind()方法来完成的，如果用jQuery来做开发，就不用考虑addEventListener()和attachEvent()之间的兼容性。

2、功能测试

就是用某种方法测试你要使用的功能在某个浏览上是不是可用，不可用的话就不用，或者改用其它通用的方法。

	if(element.addEventListener) {//在使用这个W3C方法之前首先检测它是否可用
		element.addEventListener("keydown",handler,false);
		element.addEventListener("keypress",handler,false);
	}
	else if(element.attachEvent) {//在使用该IE方法之前首先检测它
		element.attachEvent("keydown",handler);
		element.attachEvent("keypress",handler);
	}
	else {//否则，选择普遍支持的技术
		element.onkeydown = element.onkeypress = handler;
	}

3、怪异模式和标准模式

标准模式是符合W3Cschool的，怪异模式不属于

4、浏览器测试

5、IE的条件注释

很多的不兼容性都是针对IE浏览器的，我们可以用一种不怎么规范的方式来处理不兼容性问题。
例如：

	<!--[if IE 6]>
	This content is actually inside an HTML comment.
	It will only be displayed in IE6.
	<![endif]-->
	
	<!--[if ite IE7]>
	This content will only be displayed by IE 5,6 and 7 and earlier.
	Ite stands for "less than or equal". You can also use "It","gt",and "gte".
	<![endif]--]>

	<!--[if !IE]><-->
	This is normal HTML content,but IE will not display it
	because of the comment above and the comment below.
	<!--><![endif]-->

	This is normal content,displayed by all browsers.
由于这个类库只有IE需要，因此有理由在页面里使用条件注释引用它，这样其他浏览器就不会载入它：

	<!--[if IE]<script src="excanvas.js"></script><![endif]-->
### 五、安全性 ###
Web浏览器中包含JavaScript解释器，一旦载入Web页面，就可以让任意的JavaScript代码在计算机里执行，不过这里存在着安全隐患，浏览器厂商在下面两个方面进行着权衡和博弈：

定义强大的客户端API，启用强大的Web应用；

阻止恶意代码读取或修改数据、盗取隐私、诈骗或浪费时间。

下面会涉及到JavaScript的安全限制和安全问题，这个是要意识到的。
#### 1、JavaScript不能做什么 ####
客户端JavaScript没有权限来写入或删除客户计算机上的任意文件或列出任意目录，就意味着JavaScript程序不能删除数据或植入病毒。类似的，客户端JavaScript没有任何通用的网络能力。客户端JavaScript程序可以对HTTP协议编程，并且HTML5也有一个附属标准WebSockets，定义了一个类套接字的API，用于和指定的服务器通信。但是这些API都不允许对于范围更广的网络进行直接访问。通用的Internet客户端和服务器不能同时使用客户端JavaScript来写。
#### 2、同源策略 ####
同源策略是对JavaScript代码能够操作哪些Web内容的一条完整的安全限制。当Web页面使用多个`<iframe>`元素或者打开其他浏览器窗口的时候，这一策略通常会发挥作用。具体来说，脚本只能读取和所属文档来源相同的窗口和文档的属性。

文档的来源包含协议、主机，以及载入文档的URL端口。从不同Web服务器载入的文档具有不同的来源。通过同一主机的不同端口载入的文档具有不同的来源。使用http:协议载入的文档和使用https:协议载入的文档具有不同的来源，即使它们来自于同一个服务器。

脚本本身的来源和同源策略并不相关，相关的是脚本所嵌入的文档的来源，这个很重要。假设一个来自于主机A的脚本被包含到宿主B的一个Web页面中。这个脚本的来源是主机B，并且可以完整的访问包含它的文档的内容。如果脚本打开一个新窗口并载入来自于主机B的另一个文档，脚本对这个文档的内容也具有完全访问的权限。但是，如果脚本打开第三个窗口并载入一个来自于主机C的文档，同源策略就会阻止访问这个文档。

限制恶意脚本侵入文档，同源策略做的很好了，但是某些时候同源策略就显得过于严格了。下面就是3中不严格的同源策略。

#### 第一个就是主域名相同而子域名不同的情况。 ####
例如，来自于home.example.com的文档里的脚本想要合法的读取从developer.example.com载入的文档的属性，或者来自于orders.example.com的脚本可能要读取catalog.example.com上的文档的属性。为了支持这种类型的多域名站点，可以使用Document对象的domain属性。

默认的情况下，属性domain存放的是载入文档的服务器的主机名。可以设置这一属性，不过使用的字符串必须是具有有效的域前缀或它本身。因此，一个domain属性的初始值是字符串“home.example.com”就可以把它设置为“example.com”，但是不能设置为“home.example”或者“example.com”。另外，domain值中必须有一个点，不能把它设置为“com”或其他顶级域名。

这样文档就有了同源性，可以互相读取属性，切记这是主域名相同的情况下。
#### 第二个跨域资源共享 ####
请求头和新的Access-Control-Allow-Origin响应头来扩展HTTP。它允许服务器用头信息显示地列出源，或者使用通配符来匹配所有的源并允许由任何地址请求文件，这样XMLHttpRequest就不会被同源策略所限制了。
#### 第三种叫做跨文档消息 ####
允许来自一个文档的脚本可以传递文本消息到另一个文档里的脚本，而不管脚本的来源是否不同。调用Window对象上的postMessage()方法，可以异步传递信息事件到窗口的文档里。
### 3、跨站脚本 ###
跨站脚本(Cross-site scripting)或者叫做XSS，这个术语表示一类安全问题，也就是攻击者向目标Web站点注入HTML标签或者脚本。防止XSS攻击是服务器Web开发者的一项基本工作。然而，客户端JavaScript开发者必须意识到或者预防跨站脚本。

来看个例子，它是如何使用JavaScript通过用户的名字来向用户问好：
	
	<script>
	var name = decodeURICompnent(window.location.search.substring(1)) || "";
	document.write("Hello" + name);
	</script>
这两行脚本使用window.location.search来获得获得它们自己的URL中以“？”
开始的部分。它使用document.write()来向文档添加动态生成的内容。

来防止XSS攻击的方式是，在使用任何不可信的数据来动态的创建文档内容之前，从中移除HTML标签。可以通过添加如下一行代码来移除`<script>`标签两边的尖括号。
	
	name = name.replace(/</g,"&lt;").replace(/>/g,"&gt;");
### 拒绝服务攻击 ###
就是不让开客户端使用者使用客户端浏览器
### 客户端框架 ###
jQuery框架之外还有Prototype类库(专门针对DOM和Ajax实现的一套实用工具)、Dojo(是一个大型的框架包含众多的UI组件集合、包管理系统、数据抽象层等)、YUI(Yahoo开发的，和Dojo一样很庞大，包括语言工具、DOM工具、UI组件，目前YUI2和YUI3是两个不同的版本)等。