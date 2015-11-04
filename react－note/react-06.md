### Mixin

#### 为什么使用Mixin?
React回避子类组件 但是我们知道 到处重复地编写同样的代码是不好的 

所以为了将同样的功能添加到多个组件当中 **你需要将这些通用的功能包装成一个mixin 然后导入到你的模块中**  可以说 相比继承 React更喜欢这种组合的方式

#### 写一个简单的Mixin
现在假设我们在写一个app 我们知道在某些情况下我们需要在好几个组件当中设置默认的name属性。

现在，我们不再是像以前一样在每个组件中写多个同样的getDefaultProps方法，我们可以像下面一样定义一个mixin：
```JSX
var DefaultNameMixin = {
     getDefaultProps : function () {
          return {name : "Skippy"} ;
     }
} ;
```
他没什么特殊的 就是一个简单的对象

#### 加入到React组件中
为了使用mixin 我们只需要简单得在组件中加入mixins属性 然后把刚才我们写的mixin包裹成一个数组 将它作为该属性的值即可：
```JSX
var ComponentOne = React.createClass ( {
     mixins : [DefaultNameMixin],
     render : function () {
          return 
               <h2>Hello  {this.props.name}<h2> ;
     }
}) ;
React.render (<ComponentOne  /> , document.body ) ;
```
#### 重复使用
我们可以在任何其他组件中包含我们的mixin：
```JSX
var ComponentTwo = React.createClass ({
     mixins : [DefaultNameMixin],
     render : function (){
          return (
               <div>
                    <h4> {this.props.name}</h4>     
                    <p>Favorite food : {this.props.food}</p> 
               </div>
          ) ;
     }
}) ;
```
#### 生命周期方法会被重复调用！
如果你的mixin当中包含生命周期方法 不要焦急 你仍然可以在你的组件中使用这些方法，而且它们都会被调用：
```JSX
var DefaultNameMixin = {
     getDefaultProps : function (){
          return {name : "Skippy"};
     }
};
var OuterComponent = React.createClass ({
     render : function (){
          return (
               <div>
                    <ComponentOne />
                    <ComponentTwo />
               </div>
          );
     }
});
var ComponentOne = React.createClass ({
     mixins : [DefaultNameMixin],
     render : function (){
          return <h2>Hello  {this.props.name}<h2> ;
     }
});
var ComponentTwo = React.createClass ({
     mixins : [DefaultNameMixin],
     getDefaultProps : function (){
          return {food : "Pancakes"};
     }
     render : function (){
          return (
               <div>
                    <h4> {this.props.name}</h4>     
                    <p>Favorite food : {this.props.food}</p> 
               </div>
          ) ;
     }
}) ;
React.renderComponent (<OuterComponent /> , document.body);
```
两个getDefaultProps方法都将被调用 所以我们可以得到默认为Skippy的name属性和默认为Pancakes的food属性 任何一个生命周期方法或属性都会被顺利地重复调用，但是下面的情况除外：
* **render**：包含多个render方法是不行的。React 会抛出异常
* **displayName**：你多次的对它进行设置是没有问题的，但是，最终的结果只以最后一次设置为准。

需要指出的是，mixin是可以包含在其他的mixin中的：
```JSX
var UselessMixin = {
     componentDidMount : function (){
          console.log("asdas");
     }
};
var LolMixin = {
     mixins : [UselessMixin]
};
var PantsOpinion = React.createClass ({
     mixins : [LolMixin],
     render : function (){
          return (
               <p>I dislike pants</p>
          );
     }
});
React. renderComponent(<PansOpinion /> , document.body);
```
程序会在控制台打印出**asdas**

#### 包含多个Mixins
我们的mixins要包裹在数组当中 提醒了我们可以在组件中包含多个mixins：
```JSX
var DefaultNameMixin = {
     getDefaultProps : function (){
          return {name : "Lizie"};
     }
};
var DefaultFoodMixin = {
     getDefaultProps : function (){
          return {food : "Pancakes"};
     }
};
var ComponentTwo = React.createClass ({
     mixins : [DefaultNameMixin , DefaultFoodMixin],
     render : function (){
          return (
               <div>
                    <h4>{this.props.name}</h4>
                    <p>Favorite food : {this.props.food}</p>
               </div>
          );
     }
});
```
#### 注意事项

**设置相同的Prop和State**

**设置相同的方法**－－－>在不同的mixin中定义相同的方法或者mixin和组件包含了相同的方法会抛出异常
```JSX
var LogOnMountMixin = {
     componentDidMount : function (){
          console.log("mixin mount method");
          this.logBlah()
     },
     // add a logBlah method here...
     logBlah : function (){
          console.log("blah");
     }
};
var MoreLogOnMountMixin = {
     componentDidMount : function (){
          console.log("another mixing mount method");
     },
     //...and again here
     logBlah : function (){
          console.log("something other than blah");
     }
};
```
#### 多个生命周期方法的调用顺序
如果我们的组件和mixin中都包含了相同的生命周期方法会怎样呢？
* 我们的mixin方法首先会被调用 然后再是组件中的方法被调用

当我们的组件中包含多个mixin 而这些mixin中又包含相同生命周期方法时 调用顺序又如何？
* 它们会根据mixins中的顺序从左到右的进行调用

实例代码：
```JSX
var LogOnMountMixin = {
     componentDidMount : function (){
          console.log("mixin mount method");
     }
};
var MoreLogOnMountMixin = {
     componentDIdMount : function (){
          console.og("another mixin mount method");
     }
};
var ComponentOne = React.createClass ({
     mixins : [MoreLogOnMountMixin , LogOnMountMixin],
     componentDidMount : function (){
          console.log("component one mount method");
     },
     ......
});
var ComponentTwo = React.createClass ({
     mixins : [LogOnMountMixin , MoreLogOnMountMixin],
     componentDidMount : function (){
          console.log("component two mount method");
     },
     ......
});
```
控制台将输出：
```JSX
another mixin mount method
mixin mount method
component one mount method

mixin mount method
another mixing mount method
component two mount method
```

原文出处：

http://simblestudios.com/blog/development/react-mixins-by-example.html
