---
title: jQuery（中）
date: 2017-09-02 00:26:19
tags:
---
### 四、使用jQuery处理事件 ###
#### 1、事件处理程序的简单注册 ####
给单击事件注册一个事件处理程序，只要用click()方法：

	$("p").click(function() { $(this).css("backgroundColor","gray");});
比较与addEventListener()和attachEvent()方法简单多了。
![](image/jqueryclass.jpg)
**注意：**

①、focus和blur不支持事件冒泡，同样mouseenter和mouseleave也不支持。

②、resize和unload事件类型只在Window对象中触发，应该在$(window)上调用resize()和unload()方法。

③、scroll()方法也常用于Window对象上，也可以用于其他有滚动条的元素上。load()方法最好是直接将初始化函数传递给$()。

④、我们常用hover()给mouseenter(f)吓mouseleave(f)进行事件注册处理程序。再有类似的是toggle()，可以将事件处理程序函数绑定到单击事件，可以指定两个或多个处理程序函数。单击事件发生时，jQuery每次会调用一个处理函数。例如调用toggle(f,g,h)，第一单击调用f函数，第二次是g，第三次是h，再调用则f函数是第四次。该方法可以用来显示隐藏
#### 2、jQuery事件对象 ####
jQuery通过定义自己的Event对象来隐藏浏览器之间的差异。jQuery会将以下所有字段从原生Event对象中复制到jQuery Event对象上：
![](image/jqueryclass2.jpg)![](image/jqueryclass3.jpg)
**注释：**

handler指当前正被调用的事件处理程序函数的引用。
#### 4、事件处理程序的高级注册 ####
之前我们用的都是简单的单一的调用事件处理程序，更复杂的我们用bind()来作为命名的事件类型绑定处理程序，该程序会绑定到jQuery对象中的每一个元素上。

最简单的方法是bind()需要一个事件类型字符串作为第一个参数，第二参数是个事件处理程序。例如：调用
	
	$("p").click(f);
等价于：

	$("p").bind('click',f);
bind()可以有第三个参数，这两个参数之间可以传入任何值。jQuery在调用程序之前会将指定的值设置为Event对象的data属性。

bind还能为多个事件类型注册同一个程序函数：例如：

	$("a").hover(f);
等价：

	$("a").bind("mouseenter mouseleave",f);
bind允许为注册的事件处理程序指定命名空间。这个为后续的卸载做好准备，尤其在模块化中尤其有用。事件命名空间类似于CSS的类选择器。要绑定事件处理器到命名空间，添加(.)和命名空间名到事件类型字符串中即可：

	$('a').bind('mouseover.myMod',f);//在这个myMod空间中，把mouseover处理程序绑定到所有a元素

还可以绑定到多个命名空间：

	$('a').bind('mouseover.myMod.yourMod',f);	
bind还有最后一个特性是，第一个参数可以是对象，该对象把事件名映射到处理程序函数，再次使用hover()来展示，调用
	
	$('a').hover(f,g);
等价：

	$('a').bind({mouseenter:f,mouseleave；g});
传入的对象属性名可以用空格分隔，也可以包括空间名。如果第一个对象参数之后还绑定了第二个参数，其值会用作每一个事件绑定的数据参数。

与bind()类似的还有一个one()不同的是这个使用过事件处理程序后就会自动销毁，如它的名字，永远只会触发一次。
binde和one没有事件捕获的处理程序。
#### 5、注销事件处理程序 ####
用bind注册的事件处理程序可以用unbind()来注销，unbind仅仅注销bind注册的事件处理程序和相关jQuery方法注册的事件处理程序。通过addEventListener()或attachEvent()注册的不会被销毁。不带参数时，unbind会注销jQuery对象中所有元素的所有事件处理程序：

	$("*").unbind();
	$('a').unbind("mouseover mouseout");
	$('a').unbind("mouseover.myMod mouseout.myMod");
	$('a').unbind(".myMod");//取消这个命名空间中的所有处理程序
	$('a').unbind("click.ns1.ns2");//取消绑定同时在ns1和ns2命名空间下的单击程序
	$('a').unbind("click",myClickHandler);//注销选中元素的某个事件的处理程序
#### 6、触发事件 ####
当用户使用鼠标、键盘或触发其他事件时，注册的事件处理程序会自动调用，如果能手动调用就更好了。不带参数的调用：

	$("#my_form").submit();//就和用户单击提交按钮一样
等价：

	$("#my_form").trigger("submit");//trigger类似于bind，不过不能指定多个字符串作为事件类型
	$("button").trigger("click.ns1");//触发某个命名空间下的单击程序
	$("button").trigger("click!");//触发没有命名空间的单击处理程序	
和trigger类似的有个triggerHandler()，不过后者没有冒泡行为。
#### 7、自定义事件####
使用自定义事件，你会发现，用jQuery.event.trigger()函数来代替trigger()方法，来全局触发处理器会更有用：

	//用户单击logoff按钮，广播一个自定义事件，给任何需要保存状态的感兴趣的观察者
	//然后导航到logoff页面

	$("#logoff").click(function() {
		$.event.trigger("logoff");//广播一个事件
		window.location = "logoff.php";//导航到新页面
	});
#### 8、实时事件 ####
bind()方法绑定的多是指定文档元素，静态的，但是jQuery的Web应用常用来动态创建新元素。这时候我们用delegate()和undelegate()方法来代替bind()和unbind()，通常我们在$(document)上调用delegate(),并传入一个jQuery选择器字符串、一个jQuery事件类型字符串以及一个jQuery事件处理程序函数。

当指定类型的事件冒泡到内部处理程序时，会调用指定的处理程序函数，因此为了同时处理新老的a元素上的mouseover事件，可注册：

	$(document).delegate("a","mouseover",linkHandler);
用bind()处理静态部分，用delegate()处理动态部分：

	$("a").bind("mouseover",linkHandler);
	$(".dynamic").delegate("a","mouseover",linkHandler);
实时事件依赖于冒泡事件。当事件冒泡到document对象时，它可能已经传递给了很多静态事件处理程序。如果这些程序中有一个调用了Event对象的cancelBubble()方法，实时事件处理程序将永远不会调用。

jQuery定义了一个live()方法，也可以用来注册实时事件，live也有两参数和三参数的调用形式，应用更普遍。

	$("a").live("mouseover",linkHandler);
	$("a",$(".dynamic")).live("mouseover",linkHandler);
jQuery对象通过context和selector属性来使得这些值可用，通常，仅带一个参数调用$()时，context是当前文档。下面意思是一样的。

	x.live(type,handler);
	$(c.context).delegate(x.selector,type,handler);
注销实时事件，使用die()或undelegate(),可以带一个或两个参数调用die()：
	
	$('a').die('mouseover');//移除a元素上mouseover事件的所有实时处理程序
	$('a').die('mouseover',linkHandler);//只移除一个指定的实时处理程序
undelegate()也不带任何参数调用。它会注销从选中元素委托的所有实时事件处理程序。
### 五、动画效果 ###
先说个禁用动画的操做吧`$(".stopmoving").click(function() {jQuery.fx.off = true;});`这样的话，当网页在页面中加入带有stopmoving类元素时，用户通过单击此元素就能禁用动画。
#### 1、简单动画 ####
这里有三类共9种的动画方法用来隐藏和显示元素。

**fadeIn()、fadeOut()、fadeTo()**

前两者通过改变CSS的opacity属性来显示或隐藏元素，隐藏时保留了元素的占位。两者都可接受可选的时长和回调参数。最后的fadeTo()需要传入一个opacity作为目标值，fadeTo()会将当前opacity值变化到目标值，fadeTo()方法的第一参数是时长，第二参数是opacity目标值，第三参数是可选的回调函数。

**show()、hide()、toggle()**

hide会把元素从布局中移除，就像把CSS的display属性设置为none一样。不带参数表示立即，带时长则会有个时间度上的变化。hide()表示把元素的opacity减少到0，高度也变为0，show()则反向操作。toggle()传入true和不带参数的show()一样，false作为参数则和hide()一样，必须传入时长作为参数。

**slideDown()、slideUp()、slideToggle()**

slideUp()会隐藏jQuery对象中的元素，高度变为0，然后设置CSS的display属性为none，slideDown()是反向操作，slideToggle()则是一种综合。三种方法都接受可选的时长和回调函数。
#### 2、自定义动画 ####
animate()的第一个参数指定动画内容，剩余参数指定如何定制动画。第一个参数是必须的：它必须是一个对象，该对象的属性指定要变化的CSS属性和它们的目标值。animate()会将每个元素的这些CSS属性从初始值变化到指定的目标值。
	
	$('img').animate({height:0});
第二个参数是可选的，可以传入一个对象给animate()方法。我们接下来要明白一些细节东西。
#### 动画属性对象 ####
animate()方法的第一个参数必须是对象，对象的属性名必须是CSS属性名，这些属性的值必须是动画的目标值。动画只支持数值属性：对于颜色、字体或display等枚举属性是无效的。

	$("p").animate({
		"margin-left":"+=.5in",//增加段落缩进
		opacity:"-=.1"//同时减少不透明度
	});

	$("img").animate({"width":"+=100"},500,"linear");
### 六、jQuery中的Ajax ###
#### jQuery的Ajax状态码 ####
jQuery的所有Ajax工具，包括load()方法，会调用回调函数提供请求成功或失败的异步消息。这些回调函数的第二个参数是一个字符串，可以去下列值：
	
	success 表示请求成功完成
	notmodified 表示请求已正常完成，但返回的响应内容是HTTP 304 “Not Modified”，表示请求的URL内容和上次请求的相同，没有变化。
	error  如果请求没有成功完成，原因是某些HTTP错误。
	timeout 如果Ajax请求没有在选定的时间内完成，会调用错误回调
#### jQuery的Ajax数据类型 ####
可以给jQuery.get()或jQuery.post()传递下面6中类型作为参数，使用dataType选项也可以传递这些类型给jQuery.ajax()方法。
	
	text 	将服务器的响应作为纯文本返回，不作任何处理
	html	响应式纯文本，同text一样
	xml 	请求的URL被认为指向XML格式的数据
	script 	请求的URL被认为指向JavaScript文件，被当做脚本执行
	json 	请求的URL被认为指向JSON格式的数据文件，会使用jQuery.parseJSON()来解析返回的内容，
			得到JSON对象后传入回调函数。jQuery.getJSON()使用该类型
	jsonp	请求的URL被认为指向服务器脚本，可以支持JSON格式的数据作为参数传递给客户端指定的函数
**注意：**

如果调用jQuery.get()、jQuery.post()或jQuery.ajax()函数时没有指定以上类型的任何一个。jQuery会检查HTTP响应中的Content-Type头。含有相应的则会被相应的解析出来，以上都不符合的话，数据被当做纯文本处理。
#### 1、load()方法 ####
load()方法是所有jQuery工具中最简单的：向它传递一个URL，它会异步加载URL的内容，然后内容将插入每一个选中元素中，替换已经存在的任何内容。

	setInterval(function() {$('#status').load("status_report.html");},60000);

之前的章节用load()方法，是用来注册load事件的处理程序，如果给该方法传递的第一参数是函数不是字符串，则load则成为了事件处理程序而不是Ajax的方法。

除了必要的第一参数外，load()还可以接受两个可选参数。第一个可选参数表示数据，可以追加到URL后面，或与请求一起发送。如果传入的是字符串，则会追加到URL后面（放到？或者&后面）。如果传入的是对象则会被转化为一个用&分割的名/值对后与请求一起发送。通常情况下，load()方法发送HTTP  GET请求，但如果传入数据对象，则它会发送POST请求：

	//加载特定区号的天气预报
	$('#temp').load("us_weather_report.html","zipcode=02134");
	
	//使用对象作为数据，并指定华氏温度
	$('#temp').load("us_weather_report.html",{zipcode:02134,units:'F'});
load()方法的另一个可选参数是回调函数，当Ajax请求成功或未成功，以及URL加载完毕并插入选中元素时，会调用该回调函数。如果没有指定任何数据，回调函数可以作为第二参数传入。
#### 2、Ajax工具函数 ####
jQuery的其他Ajax高级工具不是方法，而是函数，可以用jQuery或$直接调用，而不是在jQuery对象上调用。
#### jQuery.getScript() ####
第一个参数是JS的文件的URL。会异步加载文件，加载后全局作用域执行代码。能同时适用于同源和跨源脚本：
	
	//从其他服务器动态加载脚本
	jQuery.getScript("http://example.com/js/widget.js");
可以传入回掉函数作为第二参数，意味着jQuery加载和执行完毕后调用一次该回调函数：

	//加载一个类库，并在加载完成时立刻使用
	jQuery.getScript("js/jquery.my_plugin.js",function() {
		$('div').my_plugin();//使用加载的类库
	});
jQuery.getScript()通常用XHR对象获取要执行的脚本内容。对于跨域请求，jQuery会使用script元素来加载脚本。同源情况下，回调函数的第一个参数是脚本的文本内容，第二个是success状态码，第三个参数是获取脚本内容的XHR对象。同源时，jQuery.getScript()返回的也是XHR对象。

传递给这个方法的回调函数，仅仅在请求成功完成时才会被调用。如果需要在发生错误时以及成功时都得到通知，则需要调用底层的jQuery.ajax()，其他三个工具函数也是如此。
#### jQuery.getJSON() ####
与jQuery.getScript类似，它会获取文本，然后处理一下，再调用指定的回调函数。jQuery.getJSON()获取文本后，不会将它当作脚本执行，而是解析为JSON。这个方法只有传入了回调函数才有用。成功加载URL，以及内容成功解析为JSON，解析的结果作为第一参数传入回调函数。与前者工具一样，回调函数的第二个和第三个参数是success状态码和XHR对象：

	jQuery.getJSON("data.json",function(data) {
		//data  参数是对象{x:1,y:2}
	});
与上个工具不同，这个方法接受一个可选的数据对象参数，如果是数据，则必须作为是第二个参数，回调函数为第三个参数。类似于load()。

如何识别是个JSONP请求，数据字符串在末尾或&前含有“=?”字符串则是。
#### jQuery.get()和jQuery.post() ####
他们获取指定URL的内容，如果有数据的话，还可以指定数据，最后则将结果传递给制定的回调函数。前者用来发送GET请求，后者用来发送HTTP的POST请求，其他是一样的。与jQuery.getJSON()一样，这两个方法也接受相同的三个参数：必须的URL，可选的数据字符串或对象，以及一个说是可选但实际中总会用的回调函数。第一个参数返回数据，第二是success字符串，第三个XHR对象。

除此之外，还能接受第四个参数，该参数指定请求的数据类型。load()中庸html类型。jQuery.getScript()使用script类型，jQuery.getJSON()用json类型。后面这两者灵活些，是可选的数据类型。
#### 3、jQuery.ajax()函数 ####
jQuery的所有Ajax工具最后都会调用jQuery.ajax()。它仅接受一个参数：一个选项对象，该对象的属性指定Ajax请求如何执行的很多细节，例如，jQuery.getScript(ur,callback)与一下jQuery.ajax()的调用等价：
	
	jQuery.ajax({
		type:"GET",
		url:url,
		data:null,
		dataType:"script",
		success:callback
	});
success后面还可以有个属性，error。
### 七、工具函数 ###
**jQuery.browser**

browser不是一个函数而是一个对象，可用于客户端嗅探。可以用来解决浏览器相关的bug。

	if($.browser.mozilla && parseInt($.browser.version) < 4) {
	
		//在此解决一个假设的Firefox bug
	}
**jQuery.contains()**

接受两个文档作为参数，如果第一个元素包含第二个元素，则返回true，否则为false

**jQuery.each()**

和each()不同，each()方法只能遍历jQuery对象，而jQuery.each()工具函数可以遍历数组元素或对象属性。第一个参数是要遍历的数组或对象，第二参数是要在每个数组元素或对象属性上调用的函数。

**jQuery.extend()、jQuery.globalEval()、jQuery.gerp()、jQuery.inArray()、jQuery.isArray()、jQuery.isEmptyObject()、jQuery.isFunction()、jQuery.trim()**
### 八、jQuery的插件扩展 ###
知道jQuery.fn是所有jQuery对象的原型对象。如果给该对象添加一个函数，该函数会成为一个jQuery方法。例子如下：

	jQuery.fn.printIn = function() {
		
		//将所有参数合并为空格分隔的字符串
		var msg = Array.prototype.join.call(arguments," ");
		//遍历jQuery对象中的每个元素
		this.each(function() {

			//将参数字符串作为纯文本添加到每一个元素后面，并添加一个<br/>
			jQuery(this).append(document.createTextNode(msg).append("<br/>"));
		});

		//返回这个为佳修改的jQuery对象
		return this;
	}
jQuery插件约定：

不要依赖$标识符

如果插件代码不返回自己的值，请确保返回jQuery对象以便链式调用

如果扩展方式拥有两个以上参数或配置选项，请允许用户能使用对象的方式传递选项。

不要污染jQuery方法的命名空间，等。