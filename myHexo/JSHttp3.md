---
title: 脚本化HTTP大结局
date: 2017-08-29 21:10:51
tags:
---
#### 中止请求和超时 ####
可以用XHR对象的abort()方法来取消正在执行的HTTP请求，调用abort()方法在这个对象上触发abort事件。可通过XHR对象的“onabort"属性是否存在判断浏览器是否支持abort事件。

调用abort()主要原因是完成取消或超时请求取消的时间太长或响应变得无关时间，XHR2还定义了timeout属性来指定请求自动中止后的毫秒数，也定义了timeout事件用于当超时发生时触发（不是abort事件）（写本章时XHR对象不支持timeout和ontimeout属性）。可以用setTimeout()和abort()方法实现自己的超时。

	//实现超时
	//发起HTTP GET请求获取指定URL的内容，如果响应成功到达，传入responseText给回调函数
	//如果响应在timeout毫秒内没有到达，中止这个请求，浏览器可能在abort()后触发”readystatechange“
	//如果是部分请求结果到达，甚至可能设置status属性
	//所以需要设置一个标记，当部分且超时的响应到达时不会调用回调函数
	//如果使用load事件就没有这个风险

	function timeGetText(url,timeout,callback) {
		var request = new XMLHttpRequest();//创建新请求
		var timedout = false;              //是否超时

		//启动计时器，在timeout毫秒后将中止请求
		var timer = setTimout(function() {//如果触发，启动一个计时器
								timeout = true;//设置标记
								request.abort();//然后中止请求
							}
							,timeout);//中止请求之前的时长
		request.open("GET",url);//获取指定的URL
		request.onreadystatechange = function() {//定义事件处理程序
			if(request.readyState !== 4) return;//忽略未完成的请求
			if(timeout) return;//忽略中止请求
			clearTimeout(timer);//取消等待超时
			if(request.status === 200)//如果请求成功
				callback(request.responseText);//把responseText传给回调函数
		};
		request.send(null);//立即发送请求
	}
#### 跨域HTTP请求 ####
作为同源策略的一部分，XHR对象通常仅可以发起和文档具有相同服务器的HTTP请求，这个限制关闭了安全漏洞，但也阻止了大量适合使用的跨域请求。可以在`<form>`和`<iframe>`元素中使用跨域URL，而浏览器显示最终的跨域文档。同源策略，浏览器不允许原始脚本查找跨域文档的内容。使用XHR，文档内容都是通过responseText属性暴露，所以同源策略不允许XHR进行跨域请求。（注意的是script并未真正的受限制于同源策略，它可以加载并执行任何来源的脚本）

我们可以用HTTP中选择发送合适的CROS（Cross-Origin Resourse Sharing，跨域资源共享）允许跨域访问网站。而IE8通过XDomainRequest对象支持它。如果浏览器支持XHR的CROS且实现跨域请求的网站决定使用CROS允许跨域请求，那么同源策略将不会放宽而跨域请求就会正常工作。

怎么实现呢？首先如果给XHR的open()方法传入用户名和密码，那么它们绝不会通过跨域请求发送（用分布式密码破解攻击成为可能）。跨域请求也不包含任何证书：cookie和HTTP身份验证令牌（token）通常不会作为请求的内容部分发送且任何作为跨域响应来接收的cookie都会扔掉。如果跨域真的要这么做话，则必须在用send()之前设置XHR的withCredentitals属性为true，这样做不常见，但测试withCredentitals的存在性是测试浏览器是否支持CROS的一种方法。

下面的例子使用XMLHttpRequest实现HTTP HEAD请求以下载文档中a元素链接资源的类型、大小和时间等信息。这个HEAD请求按需发起，且由此产生的链接信息会出现在工具提示中。这个例子假设跨域链接不可用，但通过支持CROS的浏览器尝试下载他。

	//使用HEAD和CROS请求链接详细信息
	//这个常见的JS模块查询有href属性但没有title属性的所有a元素
	//并给它们注册onmouseover事件处理程序
	//这个事件处理程序使用XHR HEAD请求取得链接资源的详细信息
	//然后把这些详细信息设置为链接的title属性
	//这样它们将会在工具提示中显示

	whenReady(function() {
		var supportCROS = (new XMLHttpRequest()).withCredentitals !== undefined;
		var links = document.getElementsByTagName('a');
		for(var i = 0;i < links.length;i ++) {
			var links = links[i];
			if(!links.href) continue;
			if(link.title) continue;
			if(link.host !== location.host || link.protocol !== location.protocol) {
				link.title = "站外链接";
				if(!supportCROS) continue;
			}
			if(link.addEventListener)
				link.addEventListener("mouseover",mouseoverHandler,false);
			else
				link.attachEvent("onmouseover",mouseoverHandler);
		}
		function mouseoverHandler(e) {
			var link = e.target || e.srcElement;
			var url = link.href;
			var req = new XMLHttpRequest();
			req.open("HEAD",url);
			req.onreadystatechange = function() {
				if(req.readyState !== 4) return;
				if(req.status === 200) {
					var type = req.getResponseHeader("Content-Type");
					var size = req.getResponseHeader("Content-Length");
					var date = req.getResponseHeader("Last-Modified");
					link.title = "类型：" + type + "\n" + "大小：" + size + "\n" + "时间：" + date;
				}
				else{
					if(!link.title)
						link.title = "Couldn't fetch details:\n" + req.status + " " + req.statusText;
				}
			};
			req.send(null);
			if(link.removeEventListener)
				link.removeEventListener("mouseover",mouseoverHandler,false);
			else
				link.detachEvent("onmouseover",mouseoverHandler);
		}
	});
### 二、借助script发送HTTP请求：JSONP ###
我们可以通过只设置script元素的src属性，然后浏览器会发送一个HTTP请求来下载src属性所指向的URL，不受同源策略的影响，包含JSON编码数据的响应体会自动立即执行，不过对于不可信的服务器，这么做是不可取的。

假设已经有了个服务，处理GET请求并返回JSON编码的数据，同源策略下的文档在代码中使用XHR和JSON.parse()。假如启用了CROS，新的浏览器下，跨域的文档也可以用XHR享受到该服务。就浏览器下，我们就用script来实现啦！这就是P的意义所在。

当通过script元素调用数据时，响应内容必须用JS函数名和圆括号包括起来，不是一段JSON数据：

	[1,2,{"buckle":"my shoe"}]
会发送一个包裹后的JSON响应：
	
	handleResponse([1,2,{"buckle":"my shoe"}])
包裹后的响应会成为script元素的内容，它先判断JSON编码后的数据，然后把它传递给handleResponse()函数，我们可以假设，文档会拿这些数据做些有意思的事情。

为了可行起见，我们必须以某种方式告诉服务器，它正在从一个script元素调用，必须返回一个JSONP响应，而不是普通的JSON响应。可以通过在URL中添加一个查询参数来实现：例如：“?json”或“&json”。

实际操作中，支持JSONP的服务不会强制指定客户端必须实现的回调函数名称，比如handleResponse，相反，它们使用查询参数的值，允许客户端指定一个函数名，然后使用函数名去填充响应。下面的例子中使用一个名为jsonp的查询参数来指定回调函数的名称，许多支持JSONP的服务器都能分辨出这个参数名，另一个常见的参数名是callback，为了让使用到的服务支持类似的特殊需求，就要做些修改。

例子中，定义了一个getJSONP()函数，它发送JSONP请求，我们注意它是如何创建一个新的script元素，设置URL，并插入到文档中的。正是该插入操作触发HTTP请求，例子中为每个请求都创建了一个全新的内部回调函数，并作为getJSONP()函数的一个属性存了起来，最后就是回掉函数做了必要的工作：删除脚本元素，并删除自身。

	//使用script元素发送JSONP请求
	//根据指定的URL发送一个JSONP请求
	//然后把解析得到的响应数据传递给回调函数
	//在URL中添加一个名为jsonp的查询参数，用于指定该请求的回调函数的名称


	function getJSONP(url,callback) {
		
		//为本次请求创建一个唯一的回调函数
		var cbnum = "cb" + getJSONP.counter++;//每次自增计数器
		var cbname = "getJSONP." + cbnum;//作为JSONP函数的属性


		//将回调函数名称以表单编码的形式添加到URL的查询部分中
		//使用jsonp作为参数名，一些支持JSONP的服务
		//可能使用其他的参数名，比如callback
		if(url.indexOf("?") === -1)//URL没有查询部分
			url += "?json=" + cbname;//作为查询部分添加参数
		else
			url += "&jsonp=" + cbname;//作为新的参数添加它

		//创建script元素用于发送请求
		var script = document.createElement("script");

		//定义将被脚本执行的回调函数
		getJSONP[cbnum] = function(response) {
			try{
				callback(reponse);//处理响应数据
			}
			finally{//即使回调函数或响应抛出错误
				delete getJSONP[cbnum];//删除该函数
				script.parentNode.removeChild(script);//移除script元素
			}
		};

		//立即触发HTTP请求
		script.src = url;
		document.body.appendChild(script);
	}
	getJSONP.counter = 0;//创建唯一回调函数名称的计数器
### 三、基于服务器端推送事件的Comet技术 ###
在服务器端推送事件的标准中定义了一个EventSource对象，简化了Comet应用程序的编写可以传递一个URL给EventSource()构造函数，然后在返回的实例上监听消息事件。

	var ticker = new EventSource("stockprices.php");
	ticker.onmessage = function(e) {
		var type = e.type;
		var data = e.data;
		//现在处理事件类型和事件的字符串数据
	}
与message事件关联的事件对象有一个data属性，这个属性保存服务器作为该事件的负载发送的任何字符串。如同其他类型的事件一样，该对象还有一个type属性，默认值是message，事件源可以修改这个值。onmessage事件处理程序接收一个给定的服务器事件源发出的所有事件，如果有必要，也可以根据type属性派发一个事件。

客户端（创建一个EventSource对象时会）建立一个到服务器的链接，服务器保持这个链接处于打开状态，一个服务器端推送事件的协议就是这么简单。当发送一个事件，服务器端在链接中写入几行文本，抛给客户端的事件可能看起来是：

	event:add  设置时间对象的类型
	data:GOOG  设置data属性
	data:999   追加新的一行和更多的数据，一个空行会触发消息事件
该协议还有一些额外的细节，允许事件携带给定ID，然后再次脸上的客户端告诉服务器它收到的最后一个事件的ID，这样服务器就可以重新发送客户端错过的事件。

Comet架构的一个常见应用是聊天应用，聊天客户端可以通过XMLHttpRequest向聊天室发送新的消息，也可以通过EventSource对象订阅聊天信息，下面的例子使用了EventSource写的聊天客户端。

	//使用EventSource写的简易聊天客户端
	<script>
	window.onload = function() {
		var nick = prompt("Enter your nickname");//获取用户昵称
		var input = document.getElementById("input");//找出input表单元素
		input.focus();//设置键盘焦点

		//通过EventSource注册新消息的通知
		var chat = new EventSource("/chat");
		chat.onmessage = function(event) {//当捕获一条消息时
			var msg = event.data;//从事件对象中取得文本数据
			var node = document.createTextNode(msg);//把它放入一个文本节点
			var div = document.createElement("div");//创建一个div
			div.appendChild(node);//将文本插入到div中
			document.body.insertBefore(div,input);//将div插入input之前
			input.scrollIntoView();//保证input元素可见
		}

	
		//使用XMLHttpRequest把用户的消息发送给服务器
		input.onchange = function() {//当用户输入完成
			var msg = nick + ":" + input.value;//组合用户名和用户输入的信息
			var xhr = new XMLHttpRequest();//创建新的XHR
			xhr.open("POST","/chat");//发送到/chat
			xhr.setRequestHeader("Content-Type","text/  plain;charset=UTF-8");//指明为普通的UTF-8文本
			xhr.send(msg);//发送消息
			input.value = "";//准备下次输入
		}
	};
	</script>
	<!--聊天的UI只是一个单行文本域-->
	<!--新的聊天信息会插入input域之前-->
	<input id="input" style="width:100%"/>

XMLHttpRequest实现下载过程中会触发readystatechange事件的浏览器，所以可以用XHR来模拟EventSource，不再写。