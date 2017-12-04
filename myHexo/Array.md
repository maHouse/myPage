---
title: 数组基础知识
date: 2017-06-23 20:57:30
tags:
---
# 我们得知道这节的主题是什么： #
**一是数组是什么，长什么样子，我们看到了就能一眼认出来这个是不是数组；**

**二我们怎么创建数组，讲解的是数组的声明方式；**

**三是我们须知数组的一些属性和方法，因为数组不是你想怎么来就怎么来的，你必须按一定的套路走才是可行的；**

**四是我们要知道数组与对象和字符串的关系，因为是近亲所以有更多有意思的事情。**

**好了接下来一一说。**

### 一、数组的定义 ###
**数组是一系列值（元素）的有序集合体。**从这里我们来解读一下。

①数组既然是个集合体，那么就有一个范围，来表示这就是一个数组单元，数组的开始和结束标识符就是"[]",双引号里面的中括号就是一个数组的最基本的构成；

for example1：

	var a = [];
	var b = [,1,2,7,true,'ww',{x:1},1];
②数组里面的元素是什么，可以使任意数据类型的值的杂合，当然有时候杂合的东西的不容易操作。比如例子1里面的b数组就是个各种数据类型都有的杂合体，JSON无疑就是个数组里面加对象；

③有序，数组里面的元素是有顺序的，顺序是从整数0开始排序的（也就是每个数组的元素都有个索引值，这个索引值就是从0开始的），因此数组b中的两个元素数字1，他们的索引是不一样的，明显的是第一个索引值小，第二个大。

现在我们知道了数组的长相，能辨认数组了，那么接下来我们就看看怎么声明一个数组：
### 二、数组的声明 ###
声明数组的方式总的来说有两种：

①仿照对象的声明，通过数组直接量的形式，例子就是例子1；

②就是用个关键字new 来声明：

for example2：

	var a = new Array(1,2,3,true,'test');
综合老看推荐的是使用数组直接量的方式来声明一个数组。

这里我们在稍微的说点，稀松数组就是数组里存在一个空的元素，而多维数组就是几个数组的嵌套了，因为数组的元素不是固定的，所以可以有这效果。

### 三、数组的属性和方法 ###
数组知识的重点，就是这个数组的属性和方法，再有个重点就是数组与对象和字符串的关系。这次我们就先谈谈这个，分增、删、改、查4个方面来说。
#### 1.增 ####
增的意思是数组的元素增多了，这个增多可以是原来数组的值的增加，也可以是形成了一个新的数组，这就意味着数组的length是变大了。好！开始介绍方法。

①、先来个不改变原来数组的:concat(),他的意思是把两个数组的元素给拼接起来,或者就是把元素添加到数组的元素中去：
for example4：

	var d = [1,3];
	var dd = d.concat('1',2,4);//不改变原来的数组[1,3,'1',2,4]
	var ddd = d.concat([1,3,5],[4,7]);//[1,3,1,3,5,4,7]
②、通过索引为数组增加值，变化的是原来的数组：

for example3：
	
	var c = [];
	c[0] = 1;//c = [0]
	c[1] = 2;//c = [0,2]
	c[3] = 'test';//c = [0,2,'test']
③、push(),改变了原来的数组，意思是往原来的数组的末尾加一个或多个元素,这个和pop()组成了类似于栈的先进后出的感觉：

for example4：

	var pushes = ['hello','2',22];
		pushes.push(11,77);//这样既可['hello','2',22,11,77]

④、unshift(),类似于push()，不过unshift()是把元素添加到了原数组元素的第一个位置：

for example4：

	var unshifted = ['hello','2',22];
		unshifted.unshift(11,77);//这样既可[11,77,'hello','2',22]
⑤、splice(),这个比较特殊，因为他能增加也能减少，同样的是他改变的也是原来的数组,splice()方法里有三个参数，第一个为必填的，后两个为可选的，第一个参数是数组的索引值，是个整数，第二个意思是从第一个索引值的位置开始要去掉几个元素,这个值可以为负值，最后一个参数意思是去掉了参数，我们用什么来填补，是我们要填补的元素,返回的值是删除元素组成的数组：

for example5：

	var s = ['hello','2',22,11,77];
		s.splice(0,3);//返回['hello','2',22]
		s.splice(0,3，'world',33);//返回['hello','2',22]
以上就是增，下面说说删。
#### 2.删 ####
因为说过了增，删的部分大部分是对应的，所以我们就把那些不对应的讲解一下：

①、delete，这是个运算符，他删除的是元素的值，至于数组的length是不变的，去掉的是数组的元素值，去不掉的是数组的属性。


for example6：

	var del = [1,2,3];
	delete del[2];//在原数组中就没有了元素3，但是数组的length仍旧是3

②、pop(),对比push()去吧！

③、shift(),对比unshift()去吧！

④、splice()
#### 3.改和查 ####
把改和查放在一起，因为我们会感觉到他们是一体的。

①、slice(),能接收两个参数，第一个参数是必须的，为整数值，第二个参数为可选的，为整数值，都可为负值，不改变原有的数组,截取的值包前不包后：

for example7：

	var sl = [1,2,3];
	sl.slice(0,1);//返回[1]

②、reverse(),把数组的元素值给倒过来排序，改变了原来的数组元素的排序：

for example7：

	var revers = [1,2,3];
	revers.reverse();//返回[3,2,1]
③、sort(),把数组的元素按英文字母的顺序进行排序，也就是ASC11的顺序，这个可以接受一个比较函数，进行更好运算：

for example7：

	var revers = [1,2,3,10];
	revers.sort();//返回[1,10,2,3]
	revers.sort(function(x,y) {
			return x-y;
		});//返回的就是按数字大小比较后的顺序的数组了
对一个字符串数组执行不区分大小写的字符表排序，比较函数首先将参数转化为小写字符串，再开始比较：

for example8：

	var a = ['ant','Bug','cat','Dog'];
	a.sort();//返回['Bug','Dog','ant','cat']
	a.sort(function(x,y) {
			var a = x.toLowerCase();
			var b = y.toLowerCase();
			if (a < b) return -1;
			if (a > b) return 1;
			return 0;
		});//返回['ant','Bug','cat','Dog']
这里我们再简单的说说例子7和例子8中，比较函数的工作原理，我们传进去的是两个参数，所以这个比较函数每次的比较是两个两个进行比较的，每次比较后就进行一次两个元素位置的调换，而如果前(n-1)个元素没有排好序的话（说的就是最后一个元素前的元素如果已经排好序，那就直接进行第n-1个元素与最后一个元素的比较就可以了），正常情况下，我们是要用前面n-1个元素来与第n个元素进行比较的。如果想自己看看操作的话，就用chrome自己打个断点，自己看看他的工作流程。

④、遍历：for循环和forEach(),都是遍历数组内的每个元素，不同的是前者有个break可以跳出，后者想要终止的话，则必须把forEach()放到try()块中,里面传进去的是个函数：

for example8：

	function foreach(a,f,t) {
		try {a.forEach(f,t);}
		catch(e) {
			if (e === foreach.break) return;
			else throw e;
		}
	}
	foreach.break = new Error("StopIteration");
⑤、映射，这里就是用map()，就是数学函数里映射，一对一，里面包括一个函数：

for example9：

	var a = [1,3,4];
	b = a.map(function(x) {
	return x*x;});//返回的就是[1,9,16],返回的是新的数组，不修改原来的数组
⑥、过滤，就是用某个条件作为筛选，用到的是filter(),这个方法会过滤掉undefined和null，他返回的总是稠密数组不会是稀疏数组：

for example10：

	var a = [5,4,3,2,1];
	smallValues = a.filter(function(x) { return x < 3;});
	other = a.filter(function(x) { return x%2 == 0;});//参数可以多个

	a = a.filter(function(x) { return x !== undefined && x !== null;});//过滤掉undefined和null元素

⑦、every()和some()，这里返回的是个boolean值：

for example11：

	var a = [1,2,3,4,5];
	a.every(function(x) {return x < 10;});//返回的就是true
	a.every(function(x) { return x % 2 == 0;});//返回false
而如果用some()来代替every()的话，就全是true了，因为some()意思是存在而every()是全部的意思。

⑧、reduce()和reduceRight()，他们的意思就是通过某个函数把数组的元素整合为单个值，需要传进去两个参数，不过以数值和对象为主，前者是从低索引开始后者从高索引开始：

for example 12：

	var a = [1,2,3,4,5];
	var sum = a.reduce(function(x,y) {
		return x+y;},0);//0是初始值，可以不要，返回15
	var max = a.reduce(function(x,y) {
		return (x > y) ? x :y;});//求最大值
	var product = a.reduce(function(x,y) {
		return x * y;},1);//乘积
下面说说对象的形式吧！

for example 13:

	var objects = [{x:1},{y:2},{z:3}];
	var merged = objects.reduce(union);//返回{x:1,y:2,z:3}，是对象的集合

	var objects = [{x:1,a:1},{y:2,a:2},{z:3,a:3}];
	var leftUnion = objects.reduce(union);//返回{x:1,a:1,y:2,z:3}
	var rightUnion = objects.reduceRight(union);//返回{x:1,y:2,z:3,a:3}
⑨、indexOf()和lastIndexOf()，里面填写的是元素，返回的是这个元素的位置的索引，如果元素重复，返回的是第一个元素的位置的索引值，这两个方法方向相反，如果没有的话就返回-1,有两个参数，第二个参数可选，意思是要从哪个索引开始：

for example 14:
	
	var a = [0,1,2,1,0];
	a.indexOf(1);//1
	a.indexOf(3);//-1，因为没有这个值
	a.lastIndexOf(1);//3

for example 15:

	//在数组中查找所有出现的x,并返回一个匹配数组索引的数组
	function findAll(a,x) {
		var results = [],
			len = a.length,
			pos = 0;
		while(pos < len) {
			pos = a.indexOf(x,pos);
			if (pos === -1) break;
			results.push(pos);
			pos +=1;
		}
		return results;
	}
### 四、数组的亲密关系 ###

数组的亲密关系牵扯到两个：一是字符串；二是对象，无论哪个都可以有数组的一些特性，如:自动更新length、设置length来获取一段数组、从Array.prototype来继承有用的方法、其类属性为Array：

#### 1.与字符串的关系 ####
①、join()和split()这是一对，前者把数组元素以某种方式拼接起来并输出一个字符串，后者是把字符串以某个元素为分界线分为数组，不再举例子；

②、toString()和toLocalString(),与join()类似，把数组元素转化为字符串，后者是以本地化分隔符连接元素生成字符串；

③、说说继承Array.prototype的方法：

for example 16:

	var s = "JavaScript";
	Array.prototype.join.call(s," ");//"J a v a S c r i p t",继承了join()方法
	Array.prototype.filter.call(s,function(x) {
			return x.match(/[^aeiou]/);}).join("");//筛选掉了元音字母，"JvScrpt"
切记，字符串不能改变的，他们是只读的，对于那些push()、sort()、reverse()、splice()这些方法是不能使用的。

#### 2.类数组对象 ####

这个我们还是能经常碰到的，譬如用document.getElementsByName()这个方法时得到的就是个类数组。

for example 17：

	var a = {"0":"a","1":"b","2":"c",length:3};//类数组对象
	Array.prototype.join.call(a,'+');//'a+b+c'
	Array.prototype.slice.call(a,0);//['a','b','c'],真正的数组副本
	Array.prototype.map.call(a,function(x) {
			return x.toUpperCase();});//['A','B','C']
### 五、其他：数组类型###
判断是不是数组我们用两个方式：
#### 1.Array.isArray() ####
	Array.isArray([]);//true
	Array.isArray({});//false
#### 2.instanceOf运算符 ####
	[] instanceOf Array;//true
	({}) instanceOf Array;//false

# 告一段落，下次看函数 #



		