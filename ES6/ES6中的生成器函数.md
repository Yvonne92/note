#####ES6中的生成器函数
####定义生成器函数
在ES6中定义一个生成器函数，在function后跟上[*]即可
```JavaScript
function* foo1() { };
function *foo2() { };
function * foo3() { };

foo1.toString();//"function* foo1() { }"
foo2.toString();//"function* foo2() { }"
foo3.toString();//"function* foo3() { }"
foo1.constructor;//"function GeneratorFunction() {[native code]}"
```

调用生成器函数会产生一个生成器(generator)。生成器拥有的最重要的方法是next()，用来迭代：
```JavaScript
function* foo() { };
var bar = foo();
bar.next();//Object {value:undefined,done:true}
```
第2句语句看上去是函数调用，但这时候函数代码并没有执行，一直到第3行调用next方法才会执行。next方法返回一个拥有value和done两个字段的对象

生成器函数通常和yield关键字同时使用。函数执行到每个yield时都会中断并返回yield的右值(通过next方法返回对象中的value字段)。下次再调用next，函数会从yield的下一个语句继续执行。等到整个函数执行完，next方法返回的done字段会变成true
栗子：
```JavaScript
function* list(){
	for(var i = 0;i < arguments.length;i++){
		yield arguments[i];
	}
	return "done.";
}
var o = list(1,2,3);
o.next();//Object {value:1,done:false}
o.next();//Object {value:2,done:false}
o.next();//Object {value:3,done:false}
o.next();//Object {value:"done.",done:true}
o.next();//Error:Generator has already finished
```
可以看到，每次调用next方法，都会得到当前yield的值。函数执行完之后，再调用next方法会产生异常。

####斐波那契数列
在其他语言中，生成器函数和yield通常会被用来演示生成斐波那契数列(前两个数字都是1，除此之外任何数字都是前两个数之和的数列)。下面是JavaScript的生成器函数版本：
```JavaScript
function* fab(max){
	var count = 0,last = 0,current = 1;

	while(max > count++){
		yield current;
		var tmp = current;
		current += last;
		last = tmp;
	}
}
var o = fab(10),ret,result = [];

while(!(ret = o.next()).done){
	result.push(ret.value);
}

console.log(result);//[1,1,2,3,5,8,13,21,34,55]
```
代码一目了然，相比常见的递归实现，用生成器函数来产生斐波那契数列既高效又直观。