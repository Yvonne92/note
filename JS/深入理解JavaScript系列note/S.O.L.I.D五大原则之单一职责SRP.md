####前言
Bob大叔提出并发扬了S.O.L.I.D五大原则，用来更好的进行面向对象编程，五大原则分别是：

* The Single Responsibility Principle（单一职责SRP）
* The Open/Closed Principle（开闭原则OCP）
* The Liskov Substitution Principle（里氏替换原则LSP）
* THe Interface Segregation Principle（接口分离原则ISP）
* The Dependency Inversion Principle（依赖反转原则DIP）

####单一职责
单一职责的描述如下
```
A class should have only one reason to change
类发生改变的原因应该只有一个
```
一个类（JavaScript下应该是一个对象）应该有一组紧密相关的行为的意思是什么呢？遵守单一职责的好处是可以让我们很容易的来维护这个对象，当一个对象封装了很多职责的话，一旦一个职责需要修改，势必会影响该对象其他职责的代码。通过解耦可以让每个职责更有弹性的变化

不过我们如何知道一个对象的多个行为构造多个职责还是单个职责呢？

* Information holder－该对象设计为存储对象并提供对象信息给其他对象。
* Structurer－该对象设计为维护对象和信息之间的关系
* Service provider－该对象设计为处理工作并提供服务给其他对象
* Controller－该对象设计为控制决策一系列负责的任务处理
* Coordinator－该对象不做任何决策处理工作，只是delegate工作到其他对象上
* Interfacer－该对象设计为在系统的各个部分转化信息（或请求）

一旦你知道了这些概念，那就很容易知道你的代码到底是多职责还是单一职责了

####实例代码
该实例代码演示的是将商品添加到购物车，代码非常糟糕，look：
```JavaScript
function Product(id,description){

	this.getId = function(){
		return id;
	};

	this.getDesciption = function(){
		return description;
	};
}

function Cart(eventAggregator){
	
	var items = [];

	this.addItem = function(item){
		items.push(item);
	};
}

(function(){
	
	var products = [
	new Product(1,"Star Was Lego Ship"),
	new Product(2,"Barbie Doll"),
	new Product(3,"Remote Control Airplane")
	],
	cart = new Cart();

	function addToCart(){
		var productId = $(this).attr('id');
		var product = $.grep(products,function(x){
			return x.getId() == productId;
		})[0];
		cart.addItem(product);

		var newItem = $('<li></li>').html(product.getDescription()).attr('id-cart',product.getId()).appendTo("#cart");
	}

	products.forEach(function(product){
		var newItem = $('<li></li>').html(product.getDescription()).attr('id',product.getId()).dblclick(addToCart).appendTo("#products");	
	});
})();
```
该代码声明了2个function分别用来描述product和cart，而匿名函数的职责是更新屏幕和用户的交互，这还不是一个很复杂的栗子，但匿名函数里却包含了很多不相关的职责，让我们看看有多少职责：

1. 首先，有product集合的声明
2. 其次，有一个将product集合绑定到#product元素的代码，而且还附件了一个添加到购物车的事件处理
3. 有Cart购物车的展示功能
4. 有添加product item到购物车并显示的功能

####重构代码
让我们来分解下，以便代码各自存放到各自的对象里，为此我们参考了martinfowler的事件聚合理论在处理代码以便各对象之间进行通信

首先我们先来实现事件聚合的功能，该功能分为2个部分，1个是Event，用于Handler回调的代码，1个是EventAggregator用来订阅发布Event，look：
```JavaScript
function Event(name){
	var handlers = [];

	this.getName = function(){
		return name;
	};

	this.addHandler = function(handler){
		for(var i = 0;i < handlers.length;i++){
			if(handlers[i] == handler){
				handler.splice(i,1);
				break;
			}
		}
	};

	this.fire = function(eventArgs){
		handlers.forEach(function(h){
			h(eventArgs);
		});
	};
}

function EventAggregator(){
	var events = [];

	function getEvent(eventName){
		return $.grep(events,function(event){
			return event.getName() === eventName;
		})[0];
	};

	this.publish = function(eventName,eventArgs){
		var event = getEvent(eventName);

		if(!event){
			event = new Event(eventName);
			event.push(event;)
		}
		event.fire(eventArgs);
	};

	this.subscribe = function(eventName,handler){
		var event = getEvent(eventName);

		if(!event){
			event = new Event(eventName);
			events.push(event);
		}

		event.addHandler(handler);
	}
}
```
然后我们来声明Product对象 look：
```JavaScript
function Product(id,description){
	
	this.getId = function(){
		return id;
	};

	this.getDescription = function(){
		return description;
	};
}
```
接着来声明Cart对象，该对象的addItem的function里我们要触发发布一个事件itemAdded，然后将item作为参数穿进去。
```JavaScript
function Cart(eventAggregator){
	var item = [];

	this.addItem = function(item){
		items.push(item);
		eventAggregator.publish("itemAdded",item);
	};
}
```
CartController主要是接受cart对象和事件聚合器，通过订阅itemAdded来增加一个li元素节点，通过订阅productSelected事件来添加product
```JavaScript
function CartController(cart,eventAggregator){
	eventAggregator.subscribe("itemAdded",function(eventArgs){
		var newItem = $('<li></li>').html(eventArgs.getDescription())
									.attr('id-cart',eventArgs.getId())
									.appendTo("#cart");
	});

	eventAggregator.subscribe("productSelected",function(eventArgs){
		cart.addItem(eventArgs.product);	
	});
}
```
Repository的目的是为了获取（可以从ajax里获取），然后暴露get数据的方法
```JavaScript
function ProductRepository(){
	var products = [new product(1,"Star Wars Lego Ship"),
		new Product(2,"Barbie Doll"),
		new Product(3,"Remote Control Airplane")];

	this.getProducts = function(){
		return products;
	}
}
```
ProductController里定义了一个onProductSelect方法，主要是发布触发productSelected事件，forEach主要是用于绑定数据到产品列表上 look：
```JavaScript
function ProductController(eventAggregator,productRepository){
	var products = productRepository.getProducts();

	function onProductSelected(){
		var productId = $(this).attr('id');
		var product = $.grep(products,function(x){
			return x.getId() == productId;
		})[0];
		eventAggregator.publish("productSelected",{
			product : product
		});
	}

	products.forEach(function(product){
		var newItem = $('<li></li>').html(product.getDescription())
									.attr('id',product.getId())
									.dblclick(onProductSelected)
									.appendTo("#products");
	})
}
```
最后声明匿名函数（需要确保HTML都加载完了才能执行这段代码，比如放在JQuery的ready方法里）：
```JavaScript
(function(){
	var eventAggregator = new EventAggregator(),
		cart = new Cart(eventAggregator),
		cartController = new CartController(cart,eventAggregator),
		productRepository = new ProductRepository(),
		productController = new ProductController(eventAggregator,productRepository);
})();
```
可以看到匿名函数的代码减少了很多，主要是一个对象的实例化代码，代码里我们介绍了Controller的概念，他接受信息然后传递到action，我们也介绍了Repository的概念，主要是用来处理product的展示，重构的结果就是写了一大堆的对象声明，但是好处是每个对象有了自己明确的职责，该展示数据的展示数据，该处理集合的处理集合，这样耦合度就非常低了



