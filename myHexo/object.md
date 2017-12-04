---
title: 对象
date: 2017-05-21 23:21:36
tags:
---

## **一、综合概述** ##
**1.全局对象：**

全局属性：undefined、NaN、infinity

全局函数：isNaN()、parseInt()、eval()

全局对象：Math、JSON

构造函数：Date（）、RegExp()、String（）、Object（）、Array（）

**2.包装对象**

JavaScript里有些原始值如：字符串、数字、布尔值，它们是有自己的属性和方法的，那么它们不是对象，属性和方法又从何而来呢？

那我们就先从说对象说起，对象是一系列属性名和属性值的集合体。对象要引用属性值就用"."加上属性名来引用；对象里的方法是用函数表示的，引用的时候用o.m()来表示。

不同的是，原始值里的字符串、数字和布尔值的属性和方法是不能被定义新的值的，而对象里面是能改变的。

在例子中：

	var s = 'test';
	s.len = 4;
	s.indexOf('t') = 0;
在上面的例子中s是字符串，但是具有自己的属性和方法，它的实现方式是在s引用自己的属性和方法时，就会通过new String(s)创建一个临时对象，这个对象会继承s的特点。这个临时对象在用到的时候会被创建出来，一旦不用就会被销毁。

例如：

	var s = 'test';
	s.len = 4;//这里是设置的新的属性值
	var t = s.len;
	console.log(t);//undefined

在s.len = 4;使用完后，这个属性就被销毁了，不会在往下传递了，所以打印的结果是undefined；不是length哟！代码测试如果用下面的代码：
	
	var s = 'test';
	s.length = 4;
	var t = s.length;
	console.log(t);//这个结果就是4了

可以运用构造函数来显示的创建包装对象，

	var s = 'test',n = 1,b = true;
	var S = new String(s);
	var N = new Number(n);
	var B = new Boolean(b);
如果我们用' == '来判断s与S、n与N、b与B的话，结果是true；如果用' === '恒等式来判断的话，就不是这个结果了。

JavaScript中的值的类型会根据需要字型转变，无论进行什么运算，共同的值类型是每个单独的值都能转换过去的，转换的又隐式转换和显式转换

for example： 

	'5' * '6*;//30,//改变就是从字符串变化到了数字类型
	
	 30 + 'object';//这里就只能转变为字符串了

parseInt() =>意思是把值转换为16进制的，目前就是这么多了！不足的话就再加补充。

## 二、对象的创建 ##

ES5中创建对象有3中方式，对象直接量、new关键字和Object.create()，这3种方式。

for example：

①对象直接量，就是直接表示出对象里面的属性和方法：

	var o = {x:1,y:2,z:function (){alert(1);}

②new关键字，就是创建一个新的对象的意思：

	var o = new Object();
	o.x = 1;
	o.y = 2;
③第三种方式利用到了继承的性质，Object.create(_,_);这里面有两个参数，前面的是必须填写的，意思是某个对象继承了那个对象，当然也可以填写null，后面的第二个参数是个可选的参数：

	var o1 = Object.create({x:1,y:2});//o1继承了{x:1,y:2}的属性

	var o2 = Object.create(null);

在o1的情况中涉及到了继承，一会说。

通过原型继承创建一个新对象

	//inherit()返回一个继承原型对象p的属性的新对象
	function inherit(p){
		if (p == null) throw TypeError();
		if (Object.create) return Object.create(p);//如果存在就是用它
		var t = typeof p;
		if (t !== 'object' && t !== 'function') throw TypeError();
		function f(){};
		f.prototype = p;
		return new f();
	}
	

## 三、属性的查询和设置 ##

我们要查询对象的属性时有两种方式：

1.是通过".":例如，o.name或者o.m(),来寻找属性或者方法；

2.通过[]来寻找或者设置：例如，o.['name']，这里就要知道对象有时候也称为关联数组、散列或者字典；

**注意：**JavaScript里面我们是不能通过第一种方式来改变属性名字的，因为我们不能改变一个标识符，不过我们能改变数据类型里的数据，而后一种方式就是数据类型里的字符串，是可以改变的。

既然谈到了属性那么我们得知道JavaScript的对象里面属性有自有属性和继承属性两种，继承属性则意味着这个属性不是自己独有的，是属于另外一个对象的，自己不过是使用了而已。

这里就先捋清原型、原型链、继承、类、对象之间的关系，其中原型和类再加上可扩展性是对象的3个属性。

我们先把一类东西，比如人给抽象出来，把人所具有的一般特征用属性和方法表示出来，意思就是一类人，对象的类属性用toString()返回的是一个字符串，表示的意思对象的类型信息。

原型与继承相关，这里就有涉及到了原型链，我们在查询对象o的属性时首先是到
对象的自有属性里查询，如果没有的话就到对象的原型里查询，再没有的话就到对象原型的原型里查询，一直到null为止，这么看来就有了一个链条的感觉。

如果是给属性赋值的话，失败的话，可能是属性为可读的但不可写，成功的话，就会有不同的答案了，要么创建一个属性，要么在原始对象中设置属性，例外的是如果o的属性是继承于属性x，而x属性是一个具有setter方法的accessor属性的话，就会有o直接调用setter方法，不会创建创建属性了，这个不会改变原型链。

**再谈谈setter和getter方法。**

对象是由属性名、属性值和一组特性组成，ES5中，属性值可以由一个或两个方法来代替，这两个方法就是getter方法和setter方法，由getter方法和setter方法定义的属性就是存取器属性（accessor property），不同于数据属性，存取器属性不具有可写性，要么是可读性要么是读/写性，也就是说如果只有setter方法的话，会返回的值是undefined。


**注意：定义存取器最简单的方法是采用对象的直接量语法的一种扩展写法**

形式：

	var o = {
		data_prop = value,
		get accessor_prop(){},//返回，这里是以逗号结尾的 
		set accessor_prop(value){}//设置
	}


可用的地方是，我们可以传入一个值，然后返回一个不同的新值，这里就是数据双向绑定的最根本的开始喽！

权威指南上有个例子，在这里展示一下：2D笛卡尔点坐标

	var p = {
		//x,y均为普通的可读写的数据属性
		x：1.0，
		y:1.0,
		//r为可读写的存储器属性，它有setter和getter
		//函数体结束后不要忘了带上逗号
		get r(){
			return Math.sqrt(this.x*this.x + this.y*this.y);},
		set r(newValue){
			var oldValue = Math.sqrt(this.x*this.x + this.y*this.y);
			this.x *= ratio;
			this.y *= ratio;
		},
		//theta是只读存取器属性，只有getter方法
		get theta() { return Math.atan2(this.y,this.x);}
	}

## 四、其他 ##
**1.属性访问错误**

同理既然是错误就是我们不想要的系统报错，两种方式：对象存在属性不在或者就是对象本身就不在

**2.删除属性**

只能删除自有属性，继承属性不能被删除，而delete运算符其实是断掉了属性与宿主的关系，对于属性中的属性则没有操作到，就是不是真正的删掉了属性，这个时候可能因为内存泄漏的原因，我们需要遍历属性中的属性，并删去

大家可以试试这个例子：

	var a = {p:{x:1}};
	b = a.p;
	delete a.p;
	console.log(b.x);//结果依旧是1
	//如果delete a.p.x来替代a.p的话，结果就是undefined了
**3.检测属性**

检测属性就用in运算符、hasOwnProperty()、propertyIsEnumerable(),这3种方式，in运算符返回的是布尔值，中间的大哥判断的是某个对象是否具有**自有属性**，最后的大哥是个加强版，自有属性还得可枚举性为真，这三者的返回值都是布尔值

for example：
	
	var o = { x:1 }
	'x' in o;  //true
	'y' in o;  //false
	'toString' in o;//true,o继承toString属性
	//看看用中间的大哥来检测的结果是什么
	o.hasOwnProperty('x');//true,是自有属性
	o.hasOwnProperty('y');//false,没有这个属性
	o.hasOwnProperty('toString');//false,这个属性是继承得来的，不是自有的
	
	//看看最后大哥的返回值及原因吧
	
	var o = inherit({ y:2 });
	o.x = 1;
	o.propertyIsEnumerable('x');//true,o有一个可枚举的自有属性x
	o.propertyIsEnumerable('y');//false,o的y属性是继承过来的
	
除此之外还可以通过非恒等式( !== )来判断一个属性值是不是undefined

	o.x !== undefined;//true
	o.y !== undefined;//false
in运算符能区分不存在的属性和存在但值为undefined的属性

**4.可枚举性**

枚举属性，我们用到的是for/in，它可以枚举遍历对象中的自有属性和继承属性，但对象继承的内置方法不可枚举，添加的属性除非你转化为不可枚举，其他都是可枚举的。

for example:
	var o = {x:1,y:2,z:3};//3个可枚举的自有属性
	o.propertyIsEnumerable('toString');//false,不可枚举
	for(p in o)
	console.log(p);

过滤的方法介绍：

	for(p in o) {
		if (!o.hasOwnProperty(p)) continue;//hasOwnProperty()是自有属性的检测，不是自有属性，那就是跳过继承的属性了
	}

	for(p in o) {
		if(typeof o[p] === 'function') continue;} //跳过方法，用的是数据类型的判断
用来枚举属性的对象的工具函数

for example：

**①.**

	/*把p中的可枚举的属性复制到o中，并返回o
	 如果o和p中含有同名属性，则覆盖o中的属性
	 这个函数并不处理getter和setter以及复制属性*/

	function extend(o,p) {
		for (prop in p) {   //遍历p中的所有属性
			o[prop] = p[prop]; //将属性添加到o中
		}
		return o;
	}
**②.**

	/*把p中的可枚举的属性复制到o中，并返回o
	 如果o和p中含有同名属性，o中的属性则不会受到影响
	 这个函数并不处理getter和setter以及复制属性*/

	function merge(o,p) {
		for(prop in p) {  //遍历p中的所有属性
			if (o.hasOwnProperty[prop]) continue;//筛选掉p中已经存在的属性
			o[prop] = p[prop];将属性添加至o中
		}
		return o;
	}


**③.**

	/*如果o中的属性和p中的属性没有同名的，则从o中删除这个属性
	返回o*/

	function restrict(o,p) {
		for(prop in o) {//遍历o中的所有属性
			if(!(prop in p)) delete o[prop];//如果在p中不存在，则删除它，这个删除还是遍历性的删除
		}
		return o;
	}

**④.**

	/*如果o中的属性和p中的属性有同名的，则从o中删除这个属性
	返回o*/

	function subtract(o,p) {
		for (prop in p) {//遍历p中的所有属性
			delete o[prop];//把它删掉
		}
		return o;
	}
**⑤.**

	/*返回一个数组，这个数组包含的是o中可枚举的自有属性的名字*/
	
	function keys(o) {
		if(typeof o !== 'object') throw TypeError();//参数必须是对象
		var result = [];//将要返回的数组
		for (var prop in o) {//遍历所有的可枚举的属性
			if(o.hasOwnProperty(prop))//判断是否是自有属性
			result.push(prop);//将属性名添加到数组中
		}
		return result;//返回这个数组
	}

**5.属性的特性**

一个属性包括1个名字和4个特性，数据属性这四个特性是：值(value)、可写性(writable)、可枚举性(enumerable)和可配置性(configurable)。而存取器属性不具有值和可写性的属性，具备的是读取(get)、写入(set)、可枚举性和可配置性。

**6.对象的三个属性**

每一个对象都有与之相关的原型(property)、类(class)和可扩展性(extensible)。

**①原型属性**

原型是用来实现继承属性的，原型也是个对象，在实例创建之初就已经设置好了

for example:

	var p = {x:1};//定义一个原型对象
	var o = Object.create(p);//使用这个原型创建一个对象
	p.isPropertyOf(o);//true，o继承p
	Object.prototype.isPropertyOf(o);//true，p继承Object.prototype

**②类属性**

对象的类属性是一个字符串，用以表示对象的类型信息，我们只能通过间接的方式来查询它，默认的就是toString()方法，并返回字符串值。

for example：
	
	function classof(o) {
		if (o === null) return 'Null';
		if (o === undefined) return 'Undefined';
		return Object.prototype.toString.call(0).slice(8,-1);
	}
classof函数可以传入任何参数，数字、字符串乃至布尔值都可以直接调用toString()方法。内置构造函数创建的对象和宿主对象都具有有意义的类属性，通过对象直接量和Object.create创建的对象的类属性是‘Object’，自定义的构造函数创建的对象也是如此，对于自定义的类来说，没办法通过类属性来区分对象的类。

for example：

	classof(null);
	classof(1);
	classof("");
	classof(true);
	classof([]);
	classof(new Date());
	function f() {};
	classof(new f());//Object,这个是一定了
**③可扩展性**

**7.序列化对象**

对象序列化是指将对象的状态转化为字符串，也可以将字符串还原为对象，ES5提供了内置函数JSON.stringfy()和JSON.parse()来实现。

for example：

	var o = {x：1，y:{z:[false,null,'']}};
	s = JSON.stringfy(o);//o就成为了字符串
	p = JSON.parse(s);//p是o的深拷贝

**8.对象方法**

**①toString()方法**

Array.toString()、Date.toString()等等，
	
	var s = {x:1,y:2}.toString();
**②toLocalString()方法**

**③toJSON**

**④valueOf()方法**

与toString()方法类似，但往往当JavaScript需要将对象转化为某种原始值而非字符串时才会用它，尤其是转换为数字的时候。

这一部分的知识就先到这里了，下一组是数组


