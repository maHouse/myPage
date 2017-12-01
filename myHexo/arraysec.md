---
title: 脚本话文档（二）
date: 2017-08-09 23:02:10
tags:
---
### 六、创建、插入和删除节点 ###
我们已经看到用HTML和纯文本字符串如何来查询和修改文档内容，已经看到我们能够遍历Document来检查组成Document的每个Element和Text节点。每个节点级别的修改文档也是可能的。Document类型定义了创建Element和Text对象的方法，Node类型定义了在节点树中插入、删除和替换的方法，例子中节点的创建和插入：

for example 6：

	//从指定的URL，异步加载和执行文档
	
	function loadasync(url) {
		var head = document.getElementsByTagName("head")[0];//查找文档的<head>标签
		var s = document.createElement("script");//创建一个<script>元素
		s.src = url;//设置它的src属性值
		head.appendChild(s);//将<script>插入到head中
	}
创作多个节点时的捷径：使用DocumentFragment
#### 1、创建节点 ####
创建新的Element节点可以使用Document对象的createElement()方法，对于HTML文档来说改名字不区分大小写，对于XML文档则区分。

Text节点用类似的方法创建：

	var newnode = document.createTextNode("text node content");
Document也定义了一些其他的工厂方法，像不经常使用createComment()。可以使用createDocumentFragment()方法。在有一种就是复制已经存在的节点。每个节点有一个cloneNode()方法来返回该节点的全新副本。给方法传递参数 true是深复制，复制其所有的后代节点，用false的话仅仅执行一个浅复制。
#### 2、插入节点 ####
插入节点有两种Node的方法：appendChild()或者insertBefore()。

appendChild()需要在插入的Element节点上调用，它插入指定节点使其成为那个节点的最后一个子节点。

insertBefore()就像appendChild()，除了接受两个参数。第一个参数就是待插入的节点，第二个是已经存在的节点，新节点在插入在该节点的前面。这个方法在新节点的父节点上调用，方法的第二个参数必须是该父节点的子节点。

for example 7：

	//将child节点插入到parent中，使其成为第n个子节点

	function insertAt(parent,child,n) {
		if(n < 0 || n > parent.childNodes.length) throw new Error("invalid index");
		else if(n == parent.childNodes.length) parent.appendChild(child);
		else parent.insertBefore(child,parent.childNodes[n]);
	}
for example 8：

	//展示了一个函数，基于表格指定列中单元格的值来进行排序，他没有创建新的节点，只是用appendChild()来改变已存在的顺序

	//表格的行排序
	//根据指定表格每行第n个单元格的值，对第一个<tbody>中的行进行排序
	//如果存在comparator函数则使用它，否则按照字母表顺序比较

	function sortrows(table,n,comparator) {
		var tbody = table.tBodies[0];//第一个tbody，可能是隐式创建的
		var rows = tbody.getElementsByTagName("tr");//tbody中的所有行
		rows = Array.prototype.slice.call(rows,0);//真实数组中的快照
		
		//基于第n个td元素的值对行排序
		rows.sort(function(row1,row2) {
			var cell1 = row1.getElementsByTagName("td")[n];//获得第n个单元格
			var cell2 = row2.getElementsByTagName("td")[n];//同上
			var val1 = cell1.textContent || cell1.innerText;//获得文本内容
			var val2 = cell2.textContent || cell2.innerText;//同上
			if(comparator) return comparator(val1,val2);//进行比较
			if(val1 < val2) return -1;
			else if(val1 > val2) return 1;
		 	else return 0;
		});
		
		//在tbody中按他们的顺序把行添加到最后
		//这将自动把他们从当前位置移走，故没有必要事先删除他们
		//如果tbody还包含了出tr的任何其他元素，这些节点将会悬浮到顶部位置

		for(var i = 0;i < rows.length;i++) tbody.appendChild(rows[i]);
	}

	//查找表格的th元素（假设只有一行），让他们可以单击
	//以便单击列标题，按该列对行排序	
	function makeSortable(table) {
		var headers = table.getElementsByTagName("th");
		for(var i = 0;i < headers.length;i++) {
			(function(n) {//嵌套函数来创建本地作用域
				headers[i].onclick = function() { sortrows(table,n);};
			}(i));//将i的值赋给局部变量n
		}
	}
#### 3、删除和替换节点 ####
removeChild()方法是从文档树种删除一个节点。注意的是这个方法不是在待删除的节点上调用，而是在其父节点上调用。在父节点上调用该方法，并将需要删除的子节点作为参数传递给它，在文档中删除n节点，可以这么写：

	n.parentNode.removeChild(n);
replaceChild()方法删除一个子节点并用一个新的节点取而代之。在父节点上调用该方法，第一个参数是新节点，第二个参数是需要代替的节点。例如，用一个文本字符串来替换节点n，代码如下：

	n.parentNode.replaceChild(document.createTextNode("[REDACTED]"),n);
replaceChild()的另一种用法：

	//用一个新的<b>元素替换n节点，并使n成为该元素的子节点
	function embolden(n) {
		//假如参数为字符串而不是节点，将其当作元素处理的id
		if(typeof n == "string") n = document.getElementById(n);
		var parent = n.parentNode;//获得n的父节点
		var b = document.createElement("b");//创建一个<b>元素
		parent.replaceChild(b,n);//用该<b>元素替换节点n
		b.appendChild(n);//使n成为<b>元素的子节点
	}
#### 4、使用DocumentFragment ####
DocumentFragment是一个特殊的Node，他作为一个临时的容器，如下这样创建一个DocumentFragment：

	var frag = document.createDocumentFragment();
像Document节点一样，DocumentFragment是独立的，不是任何其他文档的一部分。它的parentNode总是null，但是类似于Element，它可以有任意多的子节点，可以用appendChild()、insertBefore()等方法来操作他们。

DocumentFragment上午特殊之处是它使得一组节点被当作一个节点看待，如果给appendChild()、insertBefore()或replaceChild()传递一个DocumentFragment，会是所有的节点都被使用。以下函数是使用DocumentFragment来倒序排列一个节点的子节点：

	//倒序排列节点n的子节点
	function reverse(n) {
		//创建一个DocumentFragment作为临时容器
		var f = document.createDocumentFragment();
		//从后至前循环子节点，将每个子节点移动到文档片段中
		//n的最后一个节点变成f的第一个节点，反之亦然
		//注意，给f添加一个节点，该节点自动会从n中删除
		while(n.lastChild) f.appendChild(n.appendChild);
		
		//最后，把f的所有子节点一次性全部移回n中
		n.appendChild(f);
	}
for example 9:

	//使用innerHTMLshixianInsertAdjacentHTML()
	//本模块为不支持它的浏览器定义了Element.insertAdjacentHTML
	//还定义了一些可移植的HTML插入函数，他们的名字比insertAdjacent更符合逻辑
	//Insert.before()、Insert.after()、Insert.atStart()和Insert.atEnd()

	var Insert = (function() {
		//如果元素有原生的insertAdjacentHTML，
		//在4个函数名更明了的HTML插入函数中使用它
		if (document.createElement("div").insertAdjacentHTML) {
			return {
				before:function (e,h) {e.insertAdjacentHTML("beforebegin",h);},
				after:function (e,h) {e.insertAdjacentHTML("afterend",h);},
				atStart:function (e,h) {e.insertAdjacentHTML("afterbegin",h);},
				atEnd:function (e,h) {e.insertAdjacentHTML("beforeend",h);}
			};
		}


		//否则，无原生的insertAdjacentHTML
		//实现同样的4个函数，并使用他们来定义insertAdjacentHTML

		//首先，定义了一个工具函数，传入HTML字符串，返回一个DocumentFragment，
		//它包含了解析后的HTML的表示

		function fragment(html) {
			var elt = document.createElement("div");//创建空元素
			var frag = document.createDocumentFragment();//创建空文档片段
			elt.innerHTML = html;//设置元素内容
			while(elt.firstChild)//移动所有的节点
				frag.appendChild(elt.firstChild);//从elt到frag
			return frag;//返回frag
		}
		var Insert = {
			before:function(elt,html) {
				elt.parentNode.insertBefore(fragment(html),elt);
			},
			after:function(elt,html) {
				elt.parentNode.insertBefore(fragment(html),elt.nextSibling);
			},
			atStart:function(elt,html) {
				elt.insertBefore(fragment(html),elt.firstChild);
			},
			atEnd:function(elt,html) {elt.appendChild(fragment(html));}
		};
			
		//基于以上函数实现insertAdjacentHTML
		Element.prototype.insertAdjacentHTML = function (pos,html) {
			switch(pos.toLowerCase()) {
				case "beforebegin": return Insert.before(this,html);
				case "afterend": return Insert.after(this,html);
				case "afterbegin": return Insert.atStart(this,html);
				case "beforeend": return Insert.atEnd(this,html);
			}
		};
		return Insert;//最后返回4个插入函数
	}()) ;