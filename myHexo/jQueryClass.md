---
title: jQuery类库（上）
date: 2017-08-30 23:09:07
tags:
---
jQuery聚焦于查询，特性有：

①、有丰富的语法（CSS选择器），用来查询文档元素

②、高效的查询方法，用来找到与CSS选择器匹配的文档元素集

③、一套有用的方法，用来操作选中的元素

④、强大的函数式编程技巧，用来批量操作元素集，而不是每次只操作单个

⑤、简洁的语言用法（链式调用），用来表示一系列操作

首先，我们会介绍如何使用jQuery来实现简单查询并操作结果：

①、如何设置HTML属性、CSS样式和类、HTML表单的值和元素内容、位置高宽、以及数据

②、如何改变文档结构：对元素进行插入、替换、包装盒删除操作

③、如何使用jQuery的跨浏览器事件模型

④、如何使用jQuery来实现动画视觉效果

⑤、jQuery的工具函数

⑥、jQuery的Ajax工具，如何使用脚本来发起HTTP请求

⑦、jQuery选择器的所有用法，以及如何使用jQuery的高级选择方法

⑧、如何使用和编写插件来对jQuery进行扩展

⑨、jQuery UI类库
### 一、jQuery基础 ###
jQuery定义了一个全局函数：jQuery()，还用了一个快捷别名：$，这是jQuery在全局命名空间中定义的唯一两个变量。这是jQuery的核心查询方法。

注意：jQuery()是工厂函数，不是构造函数，它返回一个创建的对象，但没有和new关键字一起使用。jQuery定义了很多方法，可以用来操作它们表示的这组元素。例如，我们可以查找所有拥有details类的p元素，将其高亮显示，并将其中隐藏的p元素快速显示出来：

	$("p .detalis").css("background-color","yellow").show("fast");
链式调用就体现出来了。再例如，当用户单击元素时，会调用事件处理程序，使得改元素缓慢向上收缩，最终消失：

	$(".clicktohide").click(function() { $(this).slideUp("show");});
关于jQuery的术语是我们必须知晓的：

jQuery函数是jQuery或$的值，也可以创建jQuery对象，用来注册DOM就绪时需要调用的处理函数，通常我们用$()来引用。

jQuery对象是jQuery函数返回的对象。

选中元素，就是给jQuery函数一个CSS选择器时，返回的与之匹配的文档元素。

jQuery方法是jQuery对象的方法，重要的东西由此而起。
#### 1、jQuery()函数 ####
jQuery类库中，最重要的方法是jQuery()方法，共有4中调用方式：

①、把CSS选择器作为字符串传递给$()方法。还可以将一个元素或jQuery对象作为第二参数传递给$()方法，这时返回的是该特定元素或元素集的子元素中匹配选择器的部分。第二参数是可选的，定义了元素查询的起始点，经常称为上下文（context）。

②、第二种方式传递一个Element、Document或Window对象给$()方法。$()方法返回的只是简单的Element、Document或Window对象封装的jQuery对象。这样可以使用jQuery()方法而不是DOM方法来操作这些元素。jQuery程序中，我们常见到$(document)或$(this)。jQuery对象可以表示文档中的多个元素，也可以传递一个元素数组给$()方法。这个表示返回的jQuery对象是该元素的元素集。

③、传递HTML文本字符串给$()方法，文本必须是代码片段，不能是纯文本。这个方法可以接受可选的第二参数。这个参数可以是与创建元素关联的Document对象，也可以是object对象。
	
	var img = $("<img/>",
				{	src:url,
					css:{borderWidth:5},
					click:handleClick
				});

④、传入一个函数给$()方法。这个就用
	
	$(function() {//文档加载完毕时，相当于onLoad()函数
		//要执行的代码块
	});
#### 2、查询与查询结果 ####
$()返回值是一个jQuery对象，此对象是类数组，有自己的length属性，不用length属性，可以用size()方法代替，用toArray()方法将jQuery对象转化为真实数组。还有3个属性selector属性是创建jQuery对象时的选择器字符串，context属性是上下文，是传递给$()的第二参数，没有的话，默认是Document对象。最后一个是检查jQuery版本的属性jquery。

$()与querySelectorAll()，区别是前者应用面更广，使用更加方便，后者会有选择器上的限制。

下面就是jQuery对象的方法，each()接收一个唯一参数，就是回调函数，相当于forEach()，回调函数里面的this关键字指代Element对象，这里的元素是原生文档元素，不是jQuery对象。each()方法应用并不常见。

map()方法和Array.prototype.map()方法很接近，使用方法和each()方法相同。

	//根据所有标题元素，映射到它们的id，并转化为真是数组，然后排序
	$(":header").map(function(){ return this.id;}).toArray().sort();
再有就是index()方法了，这个方法接受一个元素作为参数，返回该元素在此jQuery对象中的索引值。没有的话返回-1。传入的是jQuery对象，则对该对象的第一个元素进行索引。传入的是字符串，则作为选择器在jQuery对象中进行匹配，并返回该jQuery对象中匹配该选择器的一组元素中第一个元素的索引值。什么都不传入的话，返回jQuery对象中第一个毗邻元素的索引值。

最后就是is()方法了。接受一个选择器作为参数，选择元素中至少有一个匹配元素时，则返回true，可以在each()回调函数中使用：

	$("div").each(function() {//对于每一个<div>元素
		if($(this).is(":hidden")) return;//跳过隐藏元素
		
		//对可见元素做什么
	});
### 二、jQuery的getter和setter ###
我们现对jQuery的getter和setter方法有个概要了解：

jQuery使用同一个方法既当getter又当setter用，不是一对方法；

用作setter时，这些方法经常接受对象参数，还经常接受函数参数，函数会计算需要设置的值。

下面看看实际操作的几个方面：

#### 1、获取和设置HTML属性 ####
attr()方法，相关方法是removeAttr(),使用如下：

	$("form").attr("action");//获取form元素的action属性
	$("#icon").attr("src","icon.gif");//设置src属性为icon.gif
	$("#banner").attr({
					src:"banner.gif",
					alt:"Advertisement",
					width:720,height:64});//一次性设置4个属性
	$("a").attr("target","_blank");//使所有链接在新窗口打开
	$("a").attr("target",function() {
							if(this.host == location.host)
								return "_self";
							else return "_blank";
	});//使站内连接在本窗口中打开，并且让非站内链接在新窗口中打开
	$("a").attr({target:function() {...}});//传入函数参数
	$("a").removeAttr("target");
#### 2、获取和设置CSS属性 ####
这个针对的就是CSS样式，不是元素的HTML属性，这个不能使用复合样式，比如font和margin，只能分开一个样式一个样式的使用，如font-weight、font-family、margin-top、margin-right，可用连字符连接使用，也可以用驼峰命名法使用，如fontWeight、marginTop。样式设置时，会把数字转化为字符串，必要时要加进去px作为后缀。

	$("h1").css("font-weight");
	$("h1").css("fontWeight");
	$("h1").css("font");//错误的存在
	$("h1").css("font-variant","smallcaps");
	$("div .note").css("border","solid black 2px");
	$("h1").css({backgroundColr:"black",
				color:"white",
				fontVariant:"small-caps",
				padding:"10px 10px 20px 4px",
				border:"solid black 2px"});
	$("h1").css("font-size",function(i,curval) {
				return Math.round(1.25 * parseInt(curval));
	});//让所有<h1>字体大小增加25%，curval是当前元素作为参数，第一个参数是该元素的索引值
#### 3、获取设置CSS类 ####
添加、删除、判断class属性是否存在的方法为addClass()、removeClass()和hasClass()，用toggleClass()来达到没有某些类时，我们就添加进去，有的话就删除，是前两个方法的集合。

	$("h1").addClass("hilite");
	$("h1+p").addClass("hilite first");//h1之后的p增加两个类
	$("section").addClass(function(n) {
					return "scetion" + n});//添加自定义类

	$("p").removeClass("hilite");
	$("div").removeClass();//删除所有div中的类

	$("p").hasClass("hilite");//不如is()函数灵活

	//切换CSS类
	
	$("tr:odd").toggleClass("oddrow");
	$("h1").toggleClass(function(n) {
				return "big bold h1-" + n;
	});
	$("h1").toggleClass("hilite",true);//类似于addClass，为false则类似于removeClass
#### 4、获取和设置HTML表单值 ####
val()方法用来设置和获取HTML表单元素的value属性，还可用于获取和设置复选框、单选按钮以及`<select>`元素的选中状态：

	$("input:radio[name=ship]:checked").val();//获取单选按钮的值
	$("#email").val("Invalid email address");//给文本域设置值
	$("input:checkbox).val(["opt1","opt2"]);//选中带有这些名字或值得复选框
	$("input:text").val(function() {
		return this.defaultValue;
	});//重置所有文本域为默认值
#### 5、设置和获取元素内容 ####
text()和html()，前者针对于纯文本，没有参数的时候获取的是匹配元素的所有子孙文本节点的纯文本内容，后者不带参数则会返回第一个匹配的HTML内容。

传入字符串时候，则意味着把原来的纯文本或HTML文本内容给替换掉。

	var title = $("head title").text();//获取文档标题
	var headline = $("h1").html();
	$("h1").text(function(n,current){
		return "$" + (n+1) + ":" + current;
	});
#### 6、获取和设置元素的位置高宽 ####
offset()可以获取或设置元素的位置。返回一个对象，带有left和top属性，用来表示X和Y坐标，有必要时设置position属性使得元素可以定位：

	var elt = $("#sprite");
	var position = elt.offset();
	position.top += 100;
	elt.offset(position);
	
	$("h1").offset(function(index,curpos) {
		return {left:curpos.left + 25 * index,top:curpos.top
	}});//相对于父元素的偏移
下面说三个获取元素宽和高的getter。

	width()//元素自身的宽度，没有内边距和边框
	innerWidth()//元素自身的宽度加上内边距
	outerWidth()//元素的内边距和边框，加上true作为参数返回值还包括外边距
所以

	height() innerHeight() outerHeight()//一样的存在
至于他们之间的关系，你自己捋一下吧。

注意：getter和setter的width和height行为有不对称的地方。当设置CSS的box-sizing属性为border-box时候，宽和高是含有内边距和边框的。

scrollTop()和scrollLeft(),可获取或设置元素的滚动条位置，方法用在Window对象以及Document对象上，用在Document对象上会获取或设置存放在该Document的Window对象的滚动条位置，这个不可以传递参数。

	function page(n) {
		var w = $(window);
		var pagesize = w.height();
		var current = w.scrollTop();
		w.scrollTop(current + n * pagesize);
	}
### 三、修改文档结构 ###
修改文档结构的方法，插入、替换、复制、包装、删除。
#### 1、插入和替换 ####
			操作    			$(target).method(content)   	$(content).method(target)
	目标元素结尾处插入内容			append()							appendTo()
	目标元素起始处插入内容			prepend() 							prependTo()
	目标元素后面插入内容			after()								insertAfter()
	目标元素前面插入内容			before()							insertBefore()
	将目标元素替换为内容			replaceWidth()						replaceAll()
**注释：**

target是目标元素，content是要插入的内容。

如果把字符串传递给第二列会被认为是HTML字符串，传递给第三列则会被认为是选择器。

第二列的方法可以接受函数参数，第三列不可以。

例子：

	$("#log").append("<br/>"+message);//第二列的方法
	$("<br/>+message").appendTo("#log");//第三列方法
	$("h1").map(function() { return this.firstChild;}).before("$");
#### 2、复制元素 ####
clone()返回一个选中元素，里面加上true参数会复制事件处理程序和与元素关联的其他数据。

	$(document.body).append("<div id='linklist'><h1>List of Links</h1></div>");
	$("a").clone().appendTo("#linklist");
	$("#linklist > a").after("<br/>");
#### 3、包装元素 ####
3个包装函数。wrap()包装每个选中元素，wrapInner()包装每个选中元素的内容，wrapAll()将选中元素作为一组来包装。

	$("h1").wrap(document.createElement("i");//把h1包在i里面
	$("h1").wrapInner("<i/>");//包装h1的内容，意味着i在h1里面
	$("body>p:not(:first)").wrapAll("<div class='rest'><?div>");//将所有其他段落包装在另一个div里
#### 4、删除元素 ####
	empty() 			删除选中元素的所有子节点，但不会修改元素本身
	remove()			从文档中移除选中元素及选中元素的所有内容
	detach()			与remove()相似，不过不会移除事件处理程序和其他数据
	unwrap()			与wrap()和wrapAll()相反的操作，会移除选中元素的的父元素
#### 注释： ####
remove()通常不带参数，带参数的话，会被当作选择器，只有符合选择器的才会被移除，这个时候用filter()方法。它会移除所有的事件处理程序以及可能绑定在被移动元素上的其他数据。unwrap()不接受可选的选择器参数。