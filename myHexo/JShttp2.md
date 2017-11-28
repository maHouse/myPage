---
title: 脚本话HTTP（二）
date: 2017-08-23 23:26:56
tags:
---
	//使用表单编码数据发起一个HTTP POST请求
	function postData(url,data,callback) {
		var request = new XMLHttpRequest();
		request.open("POST",url);
		request.onreadystatechange = function() {
			if(request.readyState === 4 && callback) 
				callback(request);
		};
		request.setRequestHeader("Content-Type","application/x-www-form-urlencode");//设置Content-Type
		request.send(encodeFormData(data));//发送表单编码的数据
	}
表单数据同样可以通过GET请求来提交，表单提交的目的是只读查询，用GET请求比POST请求更合适。GET请求没有主体，所以需要发送给服务器的表单编码数据“负载”要作为URL（后跟一个问号）的查询部分。下面的例子使用GET请求。

	function getData(url,data,callback) {
		var request = new XMLHttpRequest();
		request.open("GET",url + "?" + encodeFormData(data));
		request.onreadystatechange = function() {
			if(request.readyState === 4 && callback) 
				callback(request);
		};
		request.send(null);
	}
HTML表单在提交的时候会对表单数据进行URL编码，但当使用XMLHttpRequest能给我们编码自己想要的任何数据。随着服务器上的适当支持，我们的pizza查询数据将编码成一个更清晰的URL，
	
	http://restaurantfinder.example.com/02134/1km/pizza
#### JSON编码的请求 ####
在POST请求主体中使用表单编码是常见的惯例，任何情况下它都不是HTTP协议级别的必需品。近年来，作为Web交换格式的JSON得到了普及。下面展示了如何使用JSON.stringfy()编码请求主体，不同仅在后两行。

	//使用JSON编码主体来发起HTTP POST请求
	function postJSON(url,data,callback) {
		var request = new XMLHttpRequest();
		request.open("POST",url);
		request.onreadystatechange = function() {
			if(request.readyState === 4 && callback)
				callback(request);//调用回调函数
		};
		request.setRequestHeader("Content-Type","application/json");
		request.send(JSON.stringfy(data));
	}
#### XML编码的请求 ####
XML有时也用于数据传输的编码。JS对象的用表单编码或JSON编码版本表达的pizza查询，也能用XML文档来表示他。

	<query>
		<find zipcode="02134" radius="1km">
		pizza
		</find>
	</query>
在目前展示的示例中。XHR的send()方法的参数是一个字符串或null，实际上可以在这里传入XML Document对象，下面的例子展示了如何创建一个简单的XML Document对象并使用它作为HTTP请求的主体。

	//使用XML文档作为其主体的HTTP POST请求
	
	function postQuery(url,what,where,radius,callback) {
		var request = new XMLHttpRequest();
		request.open("POST",url);
		request.onreadystatechange = function() {
			if(request.readyState === 4 && callback)
				callback(request);
		};
			
		//Create an XML document with root element <query>
		
		var doc = document.impementation.createDocument("",query,null);
		var query = doc.documentElement;//<query>元素
		var find = doc.createElement("find");//创建find元素
		query.appendChild(find);//并把它添加到query中去
		find.setAttribute("zipcode",where);//设置find属性
		find.setAttribute("radius",radius);
		find.appendChild(doc.createTextNode(what));//设置find内容

		//现在向服务器发送XML编码的数据
		//注意将自动设置Content-Type头，对于纯文本没必要设置头
		request.send(doc);
	}
#### 上传文件 ####
HTML表单的特性之一是当用户通过`<input type="file">`元素选择文件时，表单将在它产生的POST请求主体中发送文件内容。HTML表单始终能上传文件，后来可以用send()方法传入File对象来实现上传文件。

	//使用HTTP POST请求上传文件
	//查找有data-uploadto属性的全部<input type="file">元素
	//并注册onchange事件处理程序，这样任何选择的文件都会自动通过
	//POST方法发送到指定的“uploadto”URL
	//服务器的响应是忽略的
	
	whenReady(function() {//当文档准备就绪时运行
		var elts = document.getElementsByTagName("input");//所有的input元素
		for(var i = 0;i < elts.length;i++) {遍历他们
			var input = elts[i];
			if(input.type !== "file") continue;//跳过所有非文件上传元素
			var url = input.getAttribute("data-uploadto");//获取上传URL
			if(!url) continue;//跳过任何没有URL的元素
			input.addEventListener("change",function() {//当用户选择文件时
				var file = this.files[0];//假设单个文件选择
				if(!file) return;//如果没有文件，不做任何事情
				var xhr = new XMLHttpRequest();//创建新请求
				xhr.open("POST",url);
				xhr.send(file);//把文件作为主体发送
			},false);
		}
	});
	
#### multipart/form-data请求 ####

当HTML表单同时包含文件上传元素和其他元素时，浏览器不能使用普通的表单编码而必须使用称为“multipart/form-data”的特殊Content-Type来用POST方法来提交表单。这种编码包括使用长“边界”字符串把请求主体分离为多个部分。对于文本数据。手动创建“multipart/form-data”请求主体是可能的，但复杂。

	//使用POST方法发送multipart/form-data请求主体
	
	function postFormData(url,data,callback) {
		if(typeof FormData === "undefined")
			throw new Error("FormData is not implemented");
		var request = new XMLHttpRequest();
		request.open("POST",url);
		request.onreadystatechange = function() {
			if(request.readyState === 4 && callback)
				callback(request);
		};
		var formdata = new FormData();
		for(var name in data) {
			if(!data.hasOwnProperty(name)) continue;
			var value = data[name];
			if(typeof value === "function") continue;

			//每个属性变成请求的一部分
			//这里允许File对象
			formdata.append(name,value);//作为一部分添加名/值对
		}

		//在multipart/form-data请求主体中发送名/值对
		//每个都是请求的一部分，注意，当传入FormData对象时
		//send()会自动设置Content-Type头
		request.send(formdata);
	}
#### 4、进度事件 ####
XMLHttpRquest对象在请求的不同阶段触发不同类型的事件，可以不需要检查readyState属性。

当调用send()，触发单个loadstart事件。当正在加载服务器的响应时，XHR对象会发生progress事件，通常每隔50毫秒左右，所以可以使用这些事件给用户反馈请求的进度。如果请求迅速，可能从不会触发progress事件。当事件完成，会触发load事件。

一个完成的请求不一定是成功的请求，load事件的处理程序应该检查XHR对象的status状态码来确定收到的是“200 OK”而不是“404 Not Found"的HTTP响应。

HTTP请求无法完成有3种情况，请求超时触发timeout事件，请求中止，触发abort事件，太多重定向这样的网络错误会阻止请求完成，这些情况发生时触发error事件。

对于任何具体请求，浏览器将只会触发load、abort、timeout、error事件中的一个。规范指出一旦这些事件中的一个发生后，浏览器会触发loadend事件。

可以通过XHR对象的addEventListener()方法为这些事件中的每个都注册处理程序：

	if("onprogress" in (new XMLHttpRequest())) {
		//支持progress事件
	}

除了type和timestamp这样常用的Event对象属性外，还有3个有用的属性。loaded属性是目前传输的字节数值，total属性是“Content-Length”头传输的整体长度（单位是字节）。如果不知道内容长度则为0.如果知道内容长度则lengthComputable属性为true；否则为false，可知，total和loaded属性对progress事件处理程序相当有用：

	request.onprogress = function(e) {
		if(e.lengthComputable) 
			progress.innerHTML = Math.round(100 * e.loaded / e.total) + "%Complete";
	}
#### 上传进度事件 ####
除了为监听HTTP响应的加载定义的这些有用的事件外，还有用于监听HTTP请求上传的事件。XHR对象将有upload属性。upload属性值是一个对象，它定义了addEventListener()方法和整个progress事件集合，比如onprogress和onload。（没有定义onreadystatechange属性）

对于XHR对象x，设置x.onprogresss以监控响应的下载进度，并且设置x.upload.onprogress以监控请求的上传进度。

下面的例子展示了如何使用upload progress事件把上传进度反馈给用户，也展示了如何拖放API中获取File对象和如何使用FormData API在单个XHR请求中上传多个文件。

	//监控HTTP上传进度
	//查找所有含有“fileDropTarget”类的元素
	//并注册DnD事件处理程序使它们能响应文件的拖放
	//当文件放下时，上传它们到data-uploadto属性指定的URL

	whenready(function() {
		var elts = document.getElementsByClassName("fileDropTarget");
		for(var i = 0;i < elts.length;i ++) {
			var target = elts[i];
			var url = target.getAttribute("data-uploadto");
			if(!url) continue;
			createFileUploadDropTarget(target,url);
		}
		function createFileUploadDropTarget(target,url) {
			var uploading = false;
			console.log(target,url);
			target.ondragenter = function(e) {
				console,log("dragenter");
				if(uploading) return;
				var types = e.dataTransfer.types;
				if(types && ((types.contains && types.contains("Files")) || (types.indexOf && types.indexOf("Files") !== -1))) {
					target.classList.add("wantdrop");
					return false;
				}
			};
			target.ondragover = function(e) {
				if(!uploading) return false;
			};
			target.ondragleave = function(e) {
				if(!uploading) target.calssList.remove("wantdrop");
			};
			target.ondrop = function(e) {
				if(uplaoding) return false;
				var files = e.dataTransfer.files;
				if(files && files.length) {
					uploading = true;
					var message = "Uploading files:<ul>";
					for(var i = 0;i < files.length;i ++)
						message += "<li>" + files[i].name + "</li>";
					message += "</ul>";
				target.innerHTML = message;
				target.classList.remove("wantdrop");
				target.classList.add("uploading");
				var xhr = new XMLHttpRequest();
				xhr.open("POST",url);
				var body = new FormData();
				for(var i = 0;i < files.length;i ++)
					body.append(i,files[i]);
				xhr.upload,onprogress = function(e) {
					if(e.lengthComputable) {
						target.innerHTML = message + Math.round(e.loaded / e.total * 100) + "% Complete";
					}
				};
				xhr.upload.onload = function(e) {
					uploading = false;
					target.classList.remove("uploading");
					target.innerHTML = "Drop files to upload";
				};
				xhr.send(body);
				return false;
			}
		}
	});