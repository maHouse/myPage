---
title: Window 对象
date: 2017-07-28 18:03:18
tags:
---
本章介绍Window对象的属性和方法，Window对象是以窗口命名的，本章介绍的知识点如下：
### 一、计时器 ###
setTimeout()和setInterval()可以用来注册在指定的时间之后单次或者重复调用的函数。他们都是全局函数，定义为Window对象的方法，不会对窗口做什么事情。

setTimeout()方法实现一个函数在指定的毫秒数之后执行。它返回的是一个值，这个值可以传给clearTimeout()用于取消这个函数的执行。setInterval()和setTimeout()一样，不过这个函数会在指定的毫秒数的间隔里重读调用：

	setInterval(updateClock,60000);//每60秒调用一次updateClock()
和setTimeout()一样，setInterval()也返回一个值，这个值可以传递给clearInterval(),用于取消后续函数的调用。

for example 1：
		
	//安排函数f()在未来的调用模式
	//在等待了若干毫秒之后调用f()
	//如果设置了interval,并没有设置end参数，则对f()调用将不会停止
	//如果没有设置interval和end，只在若干毫秒后调用f()一次
	//只有指定了f()，才会从start=0的时刻开始
	//注意，调用invoke()不会阻塞，它会立即返回

	function invoke(f,start,interval,end) {
		if (!start) start = 0;//默认设置为0毫秒
		if (arguments.length <= 2)//单次调用模式
			setTimeout(f,start);//若干毫秒后的单次调用模式
		else{                   //多次调用模式
			setTimeout(repeat,start);//若干毫秒后调用repeat()
			function repeat() {//在上一行所示的setTimeout()中调用
				var h = setInterval(f,interval);//循环调用f()
				//在end毫秒后停止调用，前提是end已经定义了
				if (end) setTimeout(function() {clearInterval(h);},end);
			}
		} 
	}
### 二、浏览器定位和导航 ###
Window对象的location属性引用的是Location对象，它表示该窗口中所显示的文档的URL，并定义了方法来使用窗口载入新的文档。

Document对象的location属性也引用到Location对象：

window.location === document.location //总是返回true

Document对象也有一个URL属性，是文档首次载入后保存到该文档的URL的静态字符串。
#### １、解析URL ####
Location对象的其他属性－－protocol，host,hostname,port,pathname和search，分别表示URL的各个部分。search属性返回的是问号之后的URL，这部分通常是某种类型的查询字符。

for example 2：

	//提取URL的搜索字符串中的参数
	//这个函数用来解析来自URL的查询串中的name=value参数对
	//它将name=value对存储在一个对象的属性中，并返回该对象
	//使用方法
	//var args = urlArgs();//从URL中解析参数
	//var q = args.q || "";//如果参数定义了的话就使用参数，否则使用一个默认值
	//var n = args.n ? parseInt(args.n) : 10;

	function urlArgs() {
		var args = {};//定义一个空对象
		var query = location.search.substring(1);//查找到查询串，并去掉?
		var pairs = query.split("&");//根据&符号将查询字符串分开
		for (var i = 0;i < pairs.length;i++) {//对于每一个片段
			var pos = pairs[i].indexOf("=");//查找name=value
			if (pos == -1) continue;//如果没有的话，就跳过
			var name = pairs[i].substring(0,pos);提取name
			var value = pairs[i].substring(pos+1);//提取value
			value = decodeURIComponent(value);//对value进行解码
			args[name] = value;//储存为属性
		}
		return args;//返回解析后的参数
	}
### 三、浏览历史 ###
Window对象的history属性引用的是该窗口的History对象。History对象的length属性表示浏览历史列表中的元素数量，但出于安全的因素，脚本不能访问已保存的URL。History对象的back()和forward()方法与浏览器的后退和前进按钮一样，第三个方法是go()，它接受一个参数，这个参数是正整数表示向前跳几页，是负整数表示向后跳几页。

	history.go(-2);//后退两个历史记录，相当于点击后退按钮两次
### 四、浏览器和屏幕信息 ###
脚本有时候需要获取和它们所在的Web浏览器或浏览器所在的桌面相关的信息。本节将介绍Window对象的navigator和screen属性。他们分别引用Navigator和Screen对象。
### 五、对话框 ###
Window对话框我们只讲到3个，alert()向用户显示一条信息并等待用户关闭对话框、confirm()也是一条信息，要求用户单击确定或者取消按钮，并返回一个布尔值、prompt()同样是一条信息，等待用户输入字符串，并返回那个字符串。这三个方法都是阻塞的，只有关闭后才能进行下一步。

	do{
		var name = prompt("what is your name?");//得到一个字符串输入
		var correct = confirm("You entered '" + name + "'.\n" +//得到一个布尔值
					"Click Okay to proceed or Cancel to re-enter");
	} while(!correct)
	alert("Hello," + name);//输出一个纯文本消息
上述三种方式之外还有种稍微复杂的方法：showModalDialog()，显示一个包含HTML格式的“模态对话框”。可以给它传入参数，以及从对话框里返回值。showModalDialog()在浏览器当前窗口中显示一个模态窗口。第一个参数用以指定对话框中HTML内容的URL。第二个参数是一个任意值（数组和对象均可），这个值在对话框中的脚本可以通过window.dialogArguments属性的值访问。第三个参数是一个非标准的列表，包含以分号隔开的name=value对，如果提供这个参数，可以配置对话框的尺寸或其他属性。用dialogwidth和dialogheight来设置对话框窗口的大小，用resizable=yes来允许用户改变窗口大小。

for example 4：

	<!--
		这个HTML文件并不是独立的，这个文件由showModalDialog()所调用，它希望window.dialogArguments是一个由字符串组成的数组，数组的第一个元素将放置在对话框的顶部，剩下的每个元素是每行的输入框的标识，当单击Okay按钮的时候，返回一个数组，这个数组是由每个输入框的值组成，使用诸如这样的代码来调用：
	var p = showModalDialog("multiprompt,html",
				["Enter 3D point coordinates","x","y","z"],
			"dialogwidth:400;dialogheight:300;resizable:yes");
         --!>
	<form>
	<fieldset id="fields"><!--对话框中的正文部分--!>
	<div style="text-align:center"><!--关闭这个对话框的按钮--!>
	<button onclick="okay()">Okay</button><!--设置返回值和关闭事件--!>
	<button onclick="cancel()">Cancel</button><!--关闭时不带任何返回值--!>
	</div>
	<script>
	//创建对话框的主体部分，并在fieldset中显示出来
	var args = dialogArguments;
	var text = "<legend>" + args[0] + "</legend>";
	for (var i = 1;i < args.length;i++) 
		text += "<label>" + args[i] + ":<input id = 'f" + i +"'></label><br>";
	document.getElementById("fields").innerHTML = text;
	
	//直接关闭这个对话框，不设置返回值
	function cancel() { window.close();}

	//读取输入框的值，然后设置一个返回值，之后关闭
	function okay() {
		window.returnValue = [];//返回一个数组
		for(var i = 1;i < args.length;i++) //设置输入框的元素
			window.returnValue[i-1] = document.getElementById("f" + i).value;
		window.close();//关闭对话框，使showModalDialog()返回
	}
	</script>
	</fieldset>
	</form>
#### 六、作为Window对象属性的文档元素 ####
	var $ = function(id) { return document.getElementById(id);};
	ui.prompt = $("prompt");