### DOM操作
多数情况下 React的虚拟DOM足以用来创建你想要的用户体验 不需要直接操作底层真实的DOM 通过将组件组合到一起 可以把复杂的交互聚合为呈现给用户的连贯整体

然而在某些情况下  为了实现某些需求不得不去操作底层DOM 最常见的情景包括：需要与一个没有使用React的第三方类库进行整合 或者执行一个React没有原生支持的操作

为了使这些操作更容易 React提供了一个可用于处理受其自身控制的DOM节点方法 这些方法仅在组件生命周期的特定阶段才能被访问到

#### 访问受控的DOM节点
想要访问受React控制的DOM节点 首先必须能访问到负责控制这些DOM的组件

这可以通过为子组件添加一个**ref属性**来实现
```JSX
var DoodleArea = React. createClass ({
     render : function (){
          return <canvas ref = "mainCanvas" />
     }
});
```
这样你就可以通过this.refs.mainCanvas访问到<canvas>组件

但是你要保证赋给每个子组件的ref值是在所有组件中是唯一的 如果另一个子组件赋值也是mainCanvas 那么操作就会失败

一旦访问到了刚刚讨论的子组件 就可以通过它的**getDOMNode()方法**访问到底层的DOM节点

**请不要试图在render方法中这样做 因为在render方法完成并且更新之前底层的DOM节点可能不是最新的(甚至还未创建)**

同样 直到组件被挂载你才能去调用getDOMNode()方法——此时componentDidMount事件处理器才会被触发
```JSX
var DoodleArea = React.createClass ({
     render : function (){
          //render方法调用时 组件未挂载 会引起异常！
          this.getDOMNode();
          return <canvas ref = “mainCanvas” />
     },
     componentDidMount : function (){
          var canvasNode = this.refs.mainCanvas.getDOMNode()
          //这里是有效的！现在可以访问H5 Canvas节点 并且可以随意调用painting方法
     }
});
```
注意 componentDidMount内部并不是getDOMNode方法的唯一执行环境 事件处理器也可以在组件挂载后触发 所以**你也可以在事件处理器中调用getDOMNode** 同componentDidMount方法
```JSX
 var RichText = React.createClass ({
     render : function (){
               return <div ref = "editableDiv" contentEditable = "true" onKeyDown =  {this.handleKeyDown} />;
     },
     handleKeyDown : function (){
          var editor = this.refs.editableDiv.getDOMNode();
          var html = editor.innerHTML;
          //现在我们可以存储用户已经输入的HTML内容！
     }
});
```
尽管React本身没有提供访问组件原声HTML内容的方法 但是keyDown处理器可以访问到div对应的底层DOM节点(可以访问原生的HTML) 在此处可以保存用户已经输入内容的一份拷贝 计算并展示出文字个数 等等

请记住 **尽管refs和getDOMNode很强大** 但请在没有其他方式能够实现需求功能时再选择使用它们 **使用它们会成为React在性能优化上的障碍** 并且会增加应用的复杂性

#### 整合非React类库
有很多好的JavaScript类库并没有使用React构建

一些类库不需要访问DOM(比如日期和时间操作库) 但是如果需要使用它们 **保持它们的状态和React状态之间的同步**是成功整合的关键

栗子：

假设你需要使用一个autocomplete类库 包括下列实例代码：
```JSX
autocomplete ({
     target : document.getElementById("cities"),
     data : [
          "San Francisco",
          "St.Louis",
          "Amsterdam",
          "Los Angeles"
     ],
     events :[
          select : function (city){
               alert ("You have selected the city of" + city);
          }
     ]
});
```
这个autocomplete函数需要一个目标DOM节点、一个能用作数据展现的字符串清单、以及一些监听事件

创建React组件：
```JSX
var AutocompleteCities ＝ React.createClass ({
     render : function (){
          return <div id = "cities" ref = "autocompleteTarget" />
     },
     getDefaultProps : function (){
          return {
               data : [
                    "San Francisco",
                    "St.Louis",
                    "Amsterdam",
                    "Los Angeles"
               ],
          };
     },
     handleSelect : function (city){
          alert ("You have selected the city of" + city);
     }
});
```
添加componentDidMount处理器－－>将该类库封装到React中(它可以通过autocompleteTarget所指向子组件的底层DOM节点来连接这两个端口)
```JSX
var AutocompleteCities ＝ React.createClass ({
     render : function (){
          return <div id = "cities" ref = "autocompleteTarget" />
     },
     getDefaultProps : function (){
          return {
               data : [
                    "San Francisco",
                    "St.Louis",
                    "Amsterdam",
                    "Los Angeles"
               ],
          };
     },
     handleSelect : function (city){
          alert ("You have selected the city of" + city);
     }
     componentDidMount : function (){
          autocomplete({
               target : this.refs.autocompleteTarget.getDOMNode(),
               data : this.props.data,
               events : {
                    select : this.handleSelect
               }
          });
     }
});
```
componentDidMount方法只会为每个DOM节点调用一次 因此不用担心在同一节点上两次调用autocomplete方法(在这个示例中)是否会有副作用

如果担心componentDidMount方法内导致DOM节点无法移除 有可能导致内存泄漏或者其他的问题 请确保指定一个componentWillUnmount监听器 用于在组件的DOM节点移除时清理它自身

#### 侵入式插件
一个出色的插件它仅仅会修改自己的子元素 但不幸的是 事实往往并非如此

隐藏额外操作＋添加额外的清理工作－－－>否则可能会遇到DOM被意外修改的错误

假如我们使用了一个非常糟糕的插件 它修改了父元素 我们无能为力 并且它和React不兼容 这时最好的办法就是找另一个插件或者修改源码

面对这种侵入式的插件 保护好React的最佳方式就是吧DOM的操控权掌握在自己手里

**栗子：**

虚构了一个JQuery插件－－它触发了自定义事件、修改了它依附的元素

React会认为下面的组件渲染了一个单独的div 没有子元素 也没有props
```JSX
var SuperSelect = React.createClass ({
     render : function (){
          return ;
     }
});
```
我们在componentDidMount方法中做一些繁琐而丑陋的初始化工作
```JSX
var SuperSelect = React.createClass ({
     render : function (){
          return ;
     },
     componentDidMount : function (){
          var el = this.el = document.createElement('div');
          this.getDOMNode().appendChild(el);
          $(el).superSelect(this.props);
          $(el).on('superSelect' , this.handleSuperSelectChange);
     } ,
     handleSuperSelectChange : function (){
          ......
     }
});
```
此时在组件渲染好的div内又插入了一个div 我们可以自己控制内层div 这同样意味着我们有责任去完成清理工作
```JSX
componentWillUnmount : function (){
     //从DOM中移除节点
     this.getDOMNode().removeChild(this.el);
     //移除superSelect上的所有监视器
     $(this.el).off();
}
```
如果设置了全局事件监听器、定时器、初始AJAX请求 这些都需要被清理掉

处理更新
* 模拟卸载而后重新挂载
* 使用插件的更新操作API(高效、清晰)

卸载/重新挂载方案的代码：
```JSX
componentDidUpdate : function (){
     this.componentWillUnmount();
     this.componentDidMount();
}
```
假想情形：
```JSX
componentWillReceiveProps : function (nextProps){
     $(this.el).superSelect('update' , nextProps);
}
```
封装其他类库和插件的难度与它们中存在的变量个数有关

* 对于简单的插件 最好是将其重写为React组件的形式来封装它
* 对于复杂的插件 可能没有办法重写

#### 总结
当仅使用虚拟DOM无法满足需求时 可以考虑ref属性 它允许你访问指定的元素 并且在componentDidMount执行后 可以使用getDOMNode方法修改它们底层DOM节点

这允许你使用React没有原生支持的功能 或者与那些没有被设计成可以与React整合的第三方类库进行整合