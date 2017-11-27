---
title: 客户端存储
date: 2017-09-05 21:55:34
tags:
---
web应用允许使用浏览器提供的API实现将数据存储到用户的电脑上。这种客户端存储相当于赋予了web记忆功能。就是把用户上次浏览的东西存储到了本地。这个依然有同源策略的影响，不过同一站点的则可以通信。web应用可以选择存储数据的有效期：可以是临时储存到当前窗口关闭时，也可以永久存储到硬盘上，数年不消失。

客户端存储的形式有：
#### web存储 ####
它包括localStorage对象和sessionStorage对象，这两个对象是持久化关联数组，是名/值对的映射表，名和值都是字符串。web存储易于应用、支持大容量数据存储同时兼容当前所有浏览器，不兼容早期浏览器。
#### cookie ####
它是一种早期的客户端存储机制，起初是针对服务器脚本设计使用的。尽管在客户端提供了繁琐的JS操作cookie，但是难用，只适合存储少量的文本数据。不仅如此，任何cookie形式的数据，不论服务器端是否需要，每一次HTTP请求都会把这些数据传输到服务器端。目前使用的原因是所有就浏览器都支持。但随着web storage的使用，cookie会回到最初的形态，作为一种被服务器端脚本使用的客户端存储机制。
#### 离线的web应用 ####
HTML5定义了它，用以缓存web页面以及相关资源（脚本、CSS文件、图像等）。它实现的是将web应用整体存储到客户端，而不仅仅是存储数据。它能够让web“安装”到客户端，在没有网络时仍可以使用。
#### 文件系统API ####
因为现在主流浏览器都支持一个文件对象，用来将文件通过XHR上传到服务器。与相关的规范定义了一组API，用于操作一个私有的本地文件系统。在该文件系统中可以进行对文件的读写操作。
#### 1、localStorage和sessionStorage ####
Window对象上定义了两个属性：localStorage和sessionStorage。这两个属性都代表同一个Storage对象---一个持久化关联数组，数组使用字符串来索引，储存的值也都是字符串形式的。Storage对象在使用上和一般的JS对象没有区别：设置对象的属性为字符串值，随后浏览器会将该值存储起来。两个属性的区别在于存储的有效期和作用域不同：数据可以存储多长时间以及谁拥有数据的访问权。

	var name = localStorage.username;//查询一个存储的值
	name = localStorage["username"];//等价于数组表示法
	if(!name) {
		name = prompt("what is your name");//询问用户一个问题
		localStorage.username = name;//存储用户的答案
	}
	for(var name in localStorage) {//迭代所有存储的名字
		var value = localStorage[name];//查询每个名字对应的值
	}
#### 存储有效期和作用域 ####
localStorage存储的数据是永久的，除非刻意删除，或者设置浏览器设置删除，否则永不过期。

sessionStorage的作用域限定在文档源（document origin）级别，文档源是通过协议、主机名、端口三者确定的。

同源的文档源共享localStorage数据，非同源的不可以。当然不同的浏览器之间也不可以。

sessionStorage存储的数据则有所不同，一旦标签页本关闭了则会把存储的数据删除。它的作用域是在文档源中，而且还在窗口中。不过，如果一个标签页中含有两个iframe框架，包含的元素是同源的，则之间共享。
#### 存储API ####
两者通过设置属性来存储字符串值。
调用setItem()方法，将对应的名字和值传递进去，可以实现数据存储。用setItem()，将名字传进去，可以获得对应的值。调用removeItem()，将名字传进去，可以删除对应的数据。用clear()可以删除所有存储的数据。用length属性以及key()方法，传入一个从0开始的长度，以及枚举所有存储数据的名字。

	localStorage.setItem("x",1);//以x的名字存储一个数值
	localStorage.getItem("x");//获取值
	for(var i = 0;i < localStorage.length;i++){length表示所有名字/值对的总数
		var name = localStorage.key(i);//获取第i对的名字
		var value = localStorage.getItem(name);//获取该对的值
	}
	localStorage.removeItem("x");//删除x项
	localStorage.clear();//全部删除


	//识别使用的是哪类存储机制
	var memory = window.localStorage || (window.UserDataStorage && new UserDataStorage()) || new cookieStorage();
	
	//然后在对应的机制中查询数据
	var username = memory.getItem("username");
它们支持addEventListener()和attachEvent()方法。有5个重要的属性：

	key 		被设置或移除的名字或键名。
	newValue 	保存该项的新值。
	oldValue	改变或删除该项前，保存该项原先的值。
	storageArea	好比针对于Window的这两个属性
	url			触发该存储变化脚本所在文档的URL
### 二、应用程序存储和离线web应用###
