##编写高质量JavaScript代码的基本要点
####书写可维护的代码
可维护的代码意味着：
*可读的
*一致的
*可预测的
*看上去就像是同一个人写的
*可记录
####最小全局变量
*在函数内部声明的变量只在这个函数内部，函数外面不可用
*全局变量就是在任何函数外面声明的或是未声明直接简单使用的
每个JavaScript环境有一个全局对象，当你在任意函数外面使用this的时候可以访问到。你创建的每个全部变量都变成这个全局对象的属性。在浏览器中，方便起见，该全局对象有个附加属性叫做window，此window指向该全局对象本身
栗子：
```JavaScript
myglobal = "hello";//不推荐写法
console.log(myglobal);//"hello"
console.log(window.myglobal);//"hello"
console.log(window["myglobal"]);//"hello"
console.log(this.myglobal);//"hello"
```

####全局变量的问题
当程序的两个不同部分定义同名但不同作用的全局变量的时候，命名冲突在所难免
web页面包含不是该页面开发者所写的代码也比较常见，例如：
*第三方的JavaScript库
*广告方的脚本代码
*不同类型的小组件，标志和按钮
如果想和其他脚本成为好邻居应尽少使用全局变量

由于JavaScript的两个特征，不自觉的创建出全局变量是出乎意料的容易。
*你甚至不需要声明就可以使用变量
*JavaScript有隐含的全局概念，意味着不声明任何变量都会成为一个全局对象属性
栗子：
```JavaScript
function sum(x,y){
	result = x + y;
	return result;
}
```
此段代码中的result没有任何声明。代码运作正常，但在调用函数后你最后的结果就多了一个全局命名空间，这可以是一个问题的根源
经验法则是始终使用var声明变量
修改sum()函数栗子：
```JavaScript
function sum(x,y){
	var result = x + y;
	return result;
}
```
另一个创建隐式全局变量的反例就是使用任务链进行部分var声明。下面的片段中a是本地变量但b却是全局变量
栗子：
```JavaScript
function foo(){
	var a = b =0;
	// ......
}
```
此现象发生的原因在于这个从右到左的赋值，首先是赋值表达式`b = 0`，此情况下b是未声明的。这个表达式的返回值是0，然后这个0就分配给了通过var定义的这个局部变量a

如果你已经准备好声明变量，使用链分配是比较好的做法，不会产生任何意料之外的全局变量
栗子：
```JavaScript
function foo(){
	var a,b;
	//... a = b = 0;//两个均局部变量
}
```

####忘记var的副作用
隐式全局变量和明确定义的全局变量间有些小差异，就是通过delete操作符让变量未定义的能力
*通过var创建的全局变量（任何函数之外的程序创建）是不能被删除的
*无var创建的隐式全局变量（无视是否在函数中创建）是能被删除的
这表明，在技术上，`隐式全局变量`并不是真正的全局变量，但它们是`全局对象的属性`。属性是可以通过delete操作符删除的，而变量是不能的
栗子：
```JavaScript
//定义三个全局变量
var global_var = 1;
global_novar = 2;//反面教材
(function(){
	global_fromfunc = 3;//反面教材
}());

//试图删除
delete global_var;//false
delete global_novar;//true
delete global_fromfunc;//true

//测试该删除
typeof global_var;//"number"
typeof global_novar;//"undefined"
typeof global_fromfunc;//"undefined"
```
在ES5严格模式下，未声明的变量工作时会抛出一个错误

####访问全局对象
在浏览器中，全局对象可以通过window属性在代码的任何位置访问。但是在其他环境下，这个方便的属性可能被叫做其他什么东西（甚至在程序中不可用）。如果你需要在没有硬编码的window标志符下访问全局对象，你可以在任何层级的函数作用域中做如下操作：
```JavaScript
var global = (function(){
	return this;
}());
```
这种方法可以随时获得全局对象，因为其在函数中被当作函数调用了（不是通过new构造），this总是指向全局对象。实际上这个并不适用于ES5严格模式，所以在严格模式下时你必须采取不同的形式。例如，你在开发一个JS库，你可以将你的代码包裹在一个即时函数中，然后从全局作用域中，传递一个引用指向this作为你即时函数的参数

####单var形式
在函数顶部使用单var语句是比较有用的一种形式，其好处在于：
*提供了一个单一的地方去寻找功能所需要的所有局部变量
*防止变量在定义之前使用的逻辑错误
*帮助你记住声明的全局变量，因此减少了全局变量
*少代码
单var形式栗子：
```JavaScript
function func(){
	var a = 1,
		b = 2,
		sum = a + b;
		myobject = {},
		i,
		j;
	//function body...
}
```

你也可以在声明的时候做一些实际的工作，例如前面代码中的sum = a + b这个情况，另外一个例子就是当你使用DOM引用时，你可以使用单一的var把DOM引用一起指定为局部变量
栗子：
```JavaScript
function updateElement(){
	var el = document.getElementById("result"),
		style = el.style;
	//使用el和style干点其他事儿
}
```

####预解析：var散布的问题
JavaScript中，你可以在函数的任何位置声明多个var语句，并且它们就好像是在函数顶部声明一样发挥作用，这种行为成为hoisting（悬置／置顶解析／预解析）。当你使用了一个变量，然后不久在函数中又重新声明的话，就可能产生逻辑错误。对于JavaScript，只要你的变量是在同一个作用域中（同一函数），它都被当作是声明的，即使是它在var声明前使用的时候
栗子：
```JavaScript
//反例
myname = "global";//全局变量
function func(){
	alert(myname);//"undefined"
	var myname = "local";
	alert(myname);//"local"
}
func();
```
在这个例子中，在第一个alert的时候myname未声明，此时函数肯定自然而然的看全局变量myname，但是实际上并不是那么工作的。第一个alert会弹出"undefined"是因为myname被当作了函数的局部变量（尽管是之后声明的），所有的变量声明当被悬置到函数顶部。
上面代码片段执行的行为可能就像下面这样：
```JavaScript
myname = "global";//global variable
function func(){
	var myname;//等同于 -> var myname = undefined;
	alert(myname);//"undefined"
	myname = "local";
	alert(myname);//"local"
}
func();
```

####for循环
在for循环中，你可以循环取得数组或是数组类似对象的值，例如arguments和HTMLCollection对象。通常的循环形式如下：
```JavaScript
//次佳的循环
for(var i = 0;i < myarray.length;i++){
	//使用marray[i]做点什么
}
```
这种形式的循环的不足在于每次循环的时候数组的长度都要获取一下。这会降低你的代码，尤其当myarray不是数组，而是一个HTMLCollection对象的时候

HTMLCollection指的是DOM方法返回的对象：
```JavaScript
document.getElementsByName()
document.getElementsByClassName()
document.getElementsByTagName()
```
还有其他的一些HTMLCollection，这些是在DOM标准之前引进并且现在还在使用的：
```JavaScript
document.images：页面上所有的图片元素
document.links：所有a标签元素
document.forms：所有表单
document.forms[0].elements：页面上第一个表单中的所有域
```
由于DOM操作一般都是比较昂贵的，所以当你循环获取值时，缓存数组（或集合）的长度是比较好的形式：
```JavaScript
for(var i = 0,max = myarray.length;i < max;i++){
	//使用myarray做点什么
}
```
伴随着单var形式，可以把变量从循环中提出来：
```JavaScript
function looper(){
	var i = 0,
		max,
		myarray = [];
	//...
	for(i = 0;max = myarray.length;i < max;i++){
		//使用myarray[i]做点什么
	}
}
```

####for-in循环
for-in循环应该用在非数组对象的遍历上，使用for-in进行循环也被称为"枚举"

有个很重要的hasOwnProperty()方法，当遍历对象属性的时候可以过滤掉从原型链上下来的属性。
思考下面一段代码：
```JavaScript
//对象
var man = {
	hands : 2,
	legs : 2,
	heads : 1
};
//在代码的某个地方
//一个方法添加给了所有对象
if(typeof Object.prototype.clone === "undefined"){
	Object.prototype.clone = function(){};
}
```
在这个栗子中，在man定义完成后的某个地方，在对象原型上增加了一个很有用的名叫clone()的方法。此原型链是实时的，这就意味着所有的对象自动可以访问新的方法，为了避免枚举man的时候出现clone方法，你需要应用hasOwnProperty()方法过滤原型属性。如果不做过滤，会导致clone()函数显示出来。
```JavaScript
//1.
//for-in循环
for(var i in man){
	if(man.hasOwnProperty(i)){//过滤
		console.log(i,":",man[i]);
	}
}
/* 控制台显示结果 
hands : 2
legs : 2
heads : 1
*/

//2.
//反面例子
//for-in loop without checking hasOwnProperty()
for(var i in man){
	console.log(i,":",man[i])
}
/* 控制台显示结果 
hands : 2
legs : 2
heads : 1
clone:function()
*/
```
另外一种使用hasOwnProperty()的形式是取消Object.prototype上的方法：
```JavaScript
for(var i in man){
	if(Object.prototype.hasOwnProperty.call(man,i)){//过滤
		console.log(i,":",man[i]);
	}
}
```
其好处在于man对象重新定义hasOwnProperty情况下避免命名冲突。也避免了长属性查找对象的所有方法，可以使用局部变量"缓存"它
```JavaScript
var i,hasOwn = Object.prototype.hasOwnProperty;
for(i in man){
	if(hasOwn.call(man,i)){//过滤
		console.log(i,":",man[i]);
	}
}
```

####（不）扩展内置原型
扩增构造函数的prototype属性是个很强大的增加功能的方法，但有时候它太强大了／

增加内置的构造函数（如`Object()`,`Array()`,或`Function()`）挺诱人的，但是这严重的降低了可维护性，使用你代码的其它开发人员很可能期望使用内置的JavaScript方法来持续不断的工作，而不是你另加方法

另外，属性添加到原型中可能会导致不使用hasOwnProperty属性时在循环中显示出来，这会造成混乱。

但你做好准备，你也可以给原型进行自定义的添加，形式如下：
```JavaScript
if(typeof Object.prototype.myMethod !== "function"){
	Object.prototype.myMethod = function(){
		//实现
	};
}
```

####switch模式
你可以通过类似下面的形式的switch语句增强可读性和健壮性：
```JavaScript
var inspect_me = 0,
	result = '';
switch(inspect_me){
case 0:
	result = "zero";
	break;
case 1:
	result = "one";
	break;
default:
	result = "unknown";
}
```

####避免隐式类型转换
JavaScript的变量在比较的时候会隐式类型转换。这就是为什么一些诸如：false == 0或""== 0返回的结果是true。为避免引起混乱的隐含类型转换，在你比较值和表达式类型的时候始终使用`===`和`!==`操作符
```JavaScript
var zero = 0;
if(zero === false){
	//不执行，因为zero为0，而不是false
}

//反面示例
if(zero == false){
	//执行了。。。
}
```

####避免eval()
如果你现在的代码中使用了eval()，记住该咒语"eval()是魔鬼"。此方法接受任意的字符床，并当作JavaScript代码来处理。当有问题的代码是事先知道的（不是运行时确定的），没有理由使用eval()。如果代码是在运行时动态生成，有一个更好的方式不使用eval而达到同样的目标。例如，用方括号表示法来访问动态属性会更好更简单：
```JavaScript
//反面示例
var property = "name";
alert(eval("obj."+property));

//更好的
var property = "name";
alert(obj[property]);
```

使用eval()也带来了安全隐患，因为被执行的代码（例如从网络来）可能已被篡改，这是个很常见的反面教材，当处理Ajax请求得到的JSON响应的时候。在这些情况下，最好使用JavaScript内置方法来解析JSON响应，以确保安全有效。若浏览器不支持JSON.parse()，你可以使用来自JSON.org的库。

同样重要的是记住，给setInterval(),setTimeout()和Function()构造函数传递字符串，大部分情况下，与使用eval()是类似的，因此要避免。在幕后，JavaScript仍需要评估和执行你给程序传递的字符串：
```JavaScript
//反面示例
setTimeout("myFunc()",1000);
setTimeout("myFunc(1,2,3)",1000);

//更好的
setTimeout(myFunc(),1000)
setTimeout(function(){
	myFunc(1,2,3);
},1000);
```
使用新的Function()构造就类似于eval()，如果你必须使用eval()，你可以考虑使用new Function()代替。有一个小的潜在好处，因为在新的Function()中做代码评估是在局部函数作用域中运行，所以代码中任何被评估的通过var定义的变量都不会自动变为全局变量。另一种方法来阻止自动全局变量是封装eval()调用到一个即时函数中

思考下面这个栗子，这里仅un作为全局变量污染了命名空间：
```JavaScript
console.log(typeof un);//"undefined"
console.log(typeof deux);//"undefined"
console.log(typeof trois);"undefined"

var jsstring = "var un = 1;console.log(un);"
eval(jsstring);//logs "1"

jsstring = "var deux = 2;console.log(deux);";
new Function(jsstring){};//logs "2"

jsstring = "var trois = 3;console.log(trois);";
(function(){
	eval(jsstring);
}());//logs "3"

console.log(typeof un);//number
console.log(typeof deux);//"undefined"
console.log(typeof trois);//"undefined"
```
另一个eval()和Function构造不同的是eval()可以干扰作用域链，而Function()更安分守己些。不论你在哪执行Function()，它只看到全局作用域。所以其能很好的避免本地变量污染。在下面这个栗子里，eval()可以访问和修改它外部作用域中的变量，这是Function做不来的：
```JavaScript
(function(){
	var local = 1;
	eval("local = 3;console.log(local)")//logs "3"
	console.log(local);//logs "3"
}());

(function(){
	var local = 1;
	Function("console.log(typeof local);")();//logs undefined
})
```

####parseInt()下的数值转换
使用parseInt()你可以从字符串中获取数值，该方法接受另一个技术参数。当字符串以"0"开头的时候就有可能出现问题，例如，部分时间进入表单域，在ES3中，以"0"开头的字符串被当作8进制处理了，但这在ES5中改变了。为了避免矛盾和意外结果，总是指定技术参数。
```JavaScript
var month = "06";
	year = "09";
month = parseInt(month,10);
year = parseInt(year,10);
```
此例中，如果你忽略了技术参数如parseInt(year)，返回的值将是0，因为"09"被当作8进制（好比执行parseInt(year,8)），而09在8进制中不是个有效数字
替换方法是将字符串转换成数字，包括：
```JavaScript
+"08"//结果是8
Number("08")//8
```
这些通常快于parseInt()，因为parseInt()方法不是简单的解析与转换。但是如果你想输入例如"08hello"，parseInt()将返回数字，而其他以NaN告终。
