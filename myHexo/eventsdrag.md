---
title: 事件处理程序（三）
date: 2017-08-16 23:29:42
tags:
---
### 拖放事件 ###
已经说过拖动事件，但不是真正的拖放，拖放是在拖放源和拖放目标之间传输数据的用户界面。这个是进行人机交互的，蛮麻烦的。本节演示如何创建自定义拖放源和自定义拖放目标，前者是传输数据不是文本内容，后者以某种方式拖放数据而不仅仅是显示它。

整个的进行是建立在JS  API的两个事件集，一个是在拖放源上触发，一个在拖放目标上触发，所有的事件对象都类似鼠标事件对象，还拥有dataTransfer属性，这个属性引用DataTransfer对象，这个对象定义DnD API的方法和属性。

拖放源事件相当简单，任何有HTML draggable属性的文档元素都是拖放源。当用户使用鼠标放在拖放源上时，并没有选择元素内容，而是在这个元素上触发了dragstart事件。这个事件的处理程序就调用dataTransfer.setData()指定当前可用的拖放源数据（和数据类型）。HTML5中，可用dataTransfer.items.add()代替。也可以设置dataTransfer.effectAllowed来指定支持移动、复制和链接传输中的几种，同时可以调用dataTransfer.setDragImage()或dataTransfer.addElement()指定图片或文档元素用作拖动时的视觉表现。

拖动过程中，浏览器在拖放源上触发拖动事件。如果想更新拖动图片或修改提供的数据，可以监听这些事件，但一般不需要注册拖动事件处理程序。

当防止数据发生时会触发dragend事件，如果拖放源支持移动操作，就会检查dataTransfer.dropEffect去看看是否实际执行了移动操作。如果执行了，数据就会被传输到其他地方，你应该从拖放原种删除他们。

实现简单的自定义拖放源只需要dragstart事件。如下例子，在span元素中用“hh：mm”格式显示当前时间，并每分钟更新一次时间，假设这是示例要做的一切，用户能选择时钟中显示的文本，然后拖动这个时间。例子中JS代码通过设置时钟元素的draggable属性为true和定义ondragstart事件处理程序来使得时钟成为自定义拖放源。事件处理程序使用dataTransfer.setData()指定一个完整的时间戳字符串（包括日期、秒和时区信息）作为待拖动的数据。它还调用dataTransfer.setDragIcon()指定待拖动的图片（一个时钟图标）。

for example 2：

	//自定义一个拖放源
	<script src="whenReady.js"></script>
	<script>
	whenReady(function() {
		var clock = document.getElementById("clock");//时钟元素
		var icon = new Image();//用于拖动的元素
		icon.src = "clock-icon.png";//图片URL

		//每分钟显示一次时间
		function displayTime() {
			var now = new Date();//获取当前时间
			var hrs = now.getHours(),mins = now.getMinutes();
			if(mins < 10) mins = "0" + mins;
			clock.innerHTML = hrs + ":" + mins;//显示当前时间
			setTimeout(displayTime,60000);//一分钟后将再次执行
		}
		displayTime();

		//使时钟能够拖动
		//我们也能通过HTML属性实现这个目的：<span draggable="true">
		clock.draggable = true;

		//设置拖动事件处理程序
		clock.ondragstart = function(event) {
			var event = event || window.event;//用于IE兼容性

			//dataTransfer属性是拖放API的关键
			var dt = event.dataTransfer;

			//告诉浏览器正在拖动的是什么
			//把Date()构造函数用做一个返回时间戳字符串的函数
			dt.setData("Text",Date() + "\n");

			//在支持的浏览器中，告诉他拖动图标来表现时间戳
			//没有这行代码，浏览器也可以使用时钟文本图像作为拖动的值
			if(dt.setDragImage) dt.setDragImage(icon,0,0);
		};
	});
	</script>
	<style>
	#clock {font:bold 24pt sans;background:#ddf;padding:10px;border:solid black 2px;border-radius:10px;}
	</style>
	<h1>从时钟中拖出时间戳</h1>
	<span id="clock"></span>
	<textarea cols=60 rows=20></textearea>
拖放目标比这个更棘手。任何元素都可以是拖放目标，这不需要像拖放源一样要设置HTML属性，只需要简单的定义合适的事件监听程序即可。在HTML5中可以在拖放目标上定义dropzone属性来取代定义后面介绍的一部分事件处理程序。有4个事件在拖放目标上触发。当拖放元素进入文档元素时，浏览器在这个拖放元素上触发dragenter事件。拖放目标应该使用dataTransfer.types属性确定拖放对象的可用数据是否是他能理解的格式。如果检查成功拖放目标必须要让用户和浏览器都知道他对放置感兴趣。

如果元素不取消浏览器发给他的dragenter事件，浏览器将不会作为这次拖放的拖放目标，并不会向它在发送任何事件。如果取消了，浏览器将发送dragover事件表示拥护继续在目标上拖动对象。再此吃惊的是，拖放目标必须监听且取消所有这些事情来表明它继续对放置感兴趣。如果拖放目标只想指定它只允许移动、复制或链接操作，它应该使用dragover事件处理程序来设置dataTransfer.dropEffect。

如果用户移动拖放对象离开通过取消事件表明有兴趣的拖放目标，则在拖放目标上触发dragleave事件。这个事件的处理程序应该是恢复元素的边框或背景颜色或取消任何其他为响应dragenter事件而执行的可视化反馈。遗憾的是，dragenter和dragleave事件会冒泡，如果拖放目标内部还有嵌套元素，想知道dragleave事件表示拖放对象从拖放目标离开到目标外的事件还是目标内的事件很困难。

最后，用户把拖放对象放置到拖放目标上，在拖放目标上会触发drop事件。这个事件的处理程序应该使用dataTransfer.getData()获取传输的数据并做一些适当的处理。另外，如果用户在拖放目标放置一或多个文件，dataTransfer.files属性将是一个leishuzudeFile对象。在HTML5 API中，drop事件处理程序将能遍历dataTransfer.items[]的元素去检查文件和非文件数据。

下一个例子演示如何使ul元素成为拖放目标，同时如何使他们中的li元素成为拖放源。

for example 3：

	//作为拖放目标和拖放源的列表
	//DnD  API相当复杂，浏览器不完全兼容
	//这个例子基本正确，但每个浏览器会有点不同，都有点bug
	//不解决浏览器兼容问题
	whenReady(function() {
		var lists = document.getElementByTagName("ul");
		var regexp = /\bdnd\b/;
		for(var i = 0;i < lists.length;i++) 
			if(regexp.test(lists[i].className)) dnd(lists[i]);

		function dnd(list) {
			var original_class = list.className;
			var entered = 0;
		list.ondragenter = function(e) {
			e = e || window.event;
			var from = e.relatedTarget;
			entered++;
			if((from && !ischild(from,list)) || entered ==1 ){
				var dt = e.dataTransfer;
				var types = dt.types;
				if(!types || (types.contains && types.contains("text/pain")) || (types.indexOf && types.indexOf("text/pain") !=-1)) {
					list.className = original_class + "droppable";
					return false;
				}
				return;
			}
			return false;
		};
		list.ondragover = function(e) { return false; }
		list.ondragleave = function(e) {
			e = e || window.event;
			var to = e.relatedTarget;
			entered--;
			if((to && !ischild(to,list)) || entered <= 0) {
				list.className = original_class;
				entered = 0;
			}
			return false:
		};
		list.ondrop = function(e) {
			e = e || window.event;
			var dt = e.dataTransfer;
			var text = dt.getData("Text");
			if(text) {
				var item = document.createElement("li");
				item.draggable = true;
				item.appendChild(document.createTextNode(text));
				list.appendChild(item);
				list.ClassName = original_class;
				entered = 0;
				return false;
			}
		};
		var items = list.getElementsByTagName("li");
		for(var i = 0;i < items.length;i++) 
			items[i].draggable = true;
			list.ondragstart = function(e) {
				var e = e || window.event;
				var target = e.target || e.srcElement;
				if(target.tagName !== "LI") return false;
				var dt = e.dataTransfer;
				dt.setData("Text",target.innerText || target>textContent);
				dt.effectAllowed = "copyMove";
			};
			list.ondragend = function(e) {
				e = e || window.event;
				var target = e.target || e.srcElement;
				if(e.dataTransfer.dropEffect === "move") 
					target.parentNode.removeChild(target);
				function isChild(a,b) {
					for(;a;a = a.parentNode) 
						if(a ===b) return true;
					return false;
					}
				}
	});