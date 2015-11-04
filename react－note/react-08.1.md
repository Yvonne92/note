### 表单
React组件的核心理念是可预知性和可测试性 给定同样的props和state任何React组件都会渲染出一样的结果 表单也不例外

在React中表单组件有两种类型：**约束组件**和**无约束组件**

#### 无约束的组件
你可能不想在很多重要的表单中使用无约束的组件 但是它们会帮助你更好的理解约束组件 

无约束组件的构造与React中大多数据组件相比是**反模式**的

**无约束组件的由来：**在HTML中 表单组件与React组件的行为方式并不一致 给定HTML的```<input />```一个值 这个```<input />```的值仍是可以改变的 因为表单组件的值是不受React组件控制的

在React中 这种行为与设置```<input />```的defaultValue一致

我们可以通过defaultValue属性设置```<input />```的默认值
```JSX
var MyForm = React.createClass ({
     render : function (){
          return <input type = "text" defaultValue = "Hello world !" />;
     }
});
```
无约束组件的value并非由父组件设置 而是让```<input />```自己控制自己的值

一个无约束的组件没有太大的用处 除非可以访问它的值 因此需要给```<input />```添加一个ref属性来访问DOM节点的值

**ref是一个不属于DOM属性的特殊属性用来标记DOM节点 可以通过this上下文访问这个节点 为了便于访问组件中所有的ref都添加在了this.refs上**

栗子：

我们在表单中添加一个```<input />``` 并在表单提交时访问它的值
```JSX
var MyForm = React.createClass ({
     submitHandler : function (event){
          event.preventDefault ();
          //通过ref访问输入框
          var helloTo = this.refs.helloTo.getDOMNode().value;
          alert(helloTo);
     },
     render : function (){
          return (
               <form onSubmit = {this.submitHandler}>
                    <input ref = "helloTo" type = "text" defaultValue = "Hello World !" />
                    <br />
                    <button type = "submit">Speak</button>
               </form>
          );
     }
});
```
无约束组件可以用在基本的无需任何验证或者输入控制的表单中

#### 约束组件
约束组件的模式和React其它类型组件的模式一致 表单组件的状态交由React组件控制 状态值被存储在React组件的state中

如果想要更好的控制表单组件 推荐使用约束组件

在约束组件中 输入框的值是由父组件设置的

栗子

将之前的无约束组件改为约束组件：
```JSX
var MyForm = React.createClass ({
     getInitialState : funtion (){
          return {
               helloTo : "Hello World !"
          };
     },
     handleChange : function (event){
          this.setState ({
               helloTo : event.target.value
          });
     },
     submitHandler : function (event){
          event.preventDefault();
          alert (this.state.helloTo);
     },
     render : function(){
          return (
               <form onSubmit = {this.submitHandler}>
                    <input type = "text" value = {this.state.helloTo} onChange = {this.handleChange} />
                    <br />
                    <button type = "submit">Speak </button>
               </form>
          );
     }
});
```
其中最显著的变化是```<input />```的值存储在父组件的state中 因此数据流有了清晰的定义

* getIntialState设置defaultValue
* <input />的值在渲染时被设置
* <input />的值onChange时 change处理器被调用
* change处理器更新state
* 在重新渲染时更新<input />的值

虽然与无约束组件相比代码量增加不少 但是现在可以控制数据流 在用户输入数据时更新state

栗子

当用户输入时把字符都转成大写：
```JSX
handleChange : function (event){
     this.setState ({
          helloTo : event.target.value.toUpperCase()
     });
}
```
你可能会注意到 在用户输入数据后 小写字符转成大写形式并添加到输入框时并不会发生闪烁

这是因为 React拦截了浏览器原生的change事件－－> setState被调用－－> 组件重新渲染输入框－－> React重新计算差异－－>更新输入框的值

你可以使用同样的方式来限制可输入的字符集 或者限制用户向邮件地址输入框中输入不合法字符

#### 表单事件
访问表单事件是控制表单不同部分的一个非常重要的方面

React支持所有的HTML事件 这些事件遵循驼峰命名的约定 且会被转成合成事件
这些事件是标准化的 提供了跨浏览器的一致接口

所有合成事件都提供了event.target来访问触发事件的DOM节点
```JSX
handleEvent : function (){
     var DOMNode = syntheticEvent.target;
     var newValue = DOMNode.value;
}
```
这是访问约束组件最简单的方式之一

#### Label
Label是表单元素很重要的组件 通过Label可以明确的向用户传达你的要求 提升单选框和复选框的可用性

但Label和for属性有一个冲突的地方 因为如果使用JSX 这个属性会被转换成一个JavaScript对象 且作为第一个参数传递给组件的构造器 但由于for属性属于JavaScript的一个保留字 所以我们无法把它作为一个对象的属性

在React中 与class变成了className类似 for变成了htmlFor
```JSX
//JSX
<label htmlFor = "name">Name : </label>
//javascript
React.DOM.label({htmlFor : "name"} , "Name : ");
//渲染后
<label for = "name">Name : </name>
```

#### 文本框和Select
React对```<textarea />```和```<select />```的接口做了些修改 提升了一致性

```<textarea />```被改的更像```<input />```了 允许我们设置value和defaultValue
```JSX
//非约束的
<textarea defaultValue = "Hello world !" />
//约束的
<textarea value = {this.state.helloTo} onChange = {this.handleChange} />
```
```<select />```现在接受value和defaultValue来设置已选项
```JSX
//非约束的
<select defaultValue = "B">
     <option value = "A">First Option</option>
     <option value = "B">Second Option</option>
     <option value = "C">Third Option</option>
</select>
//约束的
<select value = {this.state.helloTo} onChange = {this.handleChange}>
     <option value = "A">First Option</option>
     <option value = "B">Second Option</option>
     <option value = "C">Third Option</option>
</select>
```
React支持多选select 你需要给value和defaultValue传递一个数组
```JSX
//非约束的
<select multiple = "true" defaultValue = {["A" , "B"]}>
     <option value = "A">First Option</option>
     <option value = "B">Second Option</option>
     <option value = "C">Third Option</option>
</select>
```
在使用可多选的select时 select组件的值在选项被选择时不会更新 只有选项的selected属性会发生变化 你可以使用ref或者syntheticEvent.target来访问选项 检查它们是否被选中

栗子

handleChange循环检查DOM 并过滤出哪些选项被选中了
```JSX
var MyForm = React.createClass ({
     getInitialState : function (){
          return {
               options : ["B"]
          };
     },
     handleChange : function (){
          var checked = [ ];
          var sel = event.target;
          for (var i = 0;i < sel.length;i++){
               var option = sel.option[ i ];
               if (option.selected){
                    checked.push(option.value);
               }
          }
          this.setState ({
               option : checked
          });
     },
     submitHandler : function (event){
          event.preventDefault ();
          alert (this.state.options);
     },
     render : function (){
          <form onSubmit = {this.submitHandler}>
               <select multiple = "true" value = {this.state.options} onChange = {this.handleChange}>
                    <option value = "A">First Option</option>
                    <option value = "B">Second Option</option>
                    <option value = "C">Third Option</option>
               </select>
               <br />
               <button type = "submit">Speak</button>
          </form>
     }
});
```

#### 复选框和单选框
复选框和单选框使用的则是另外一种完全不同的控制方式

在HTML中 类型为checkbox或radio的```<input />```与类型为text的```<input />```的行为完全不一样 

通常 复选框和单选框的值是不变的只有checked状态会变化

要控制复选框或者单选框就要控制它们的checked属性 可以在非约束复选框或单选框中使用defaultChecked
```JSX
//非约束的
var MyForm = React.createClass ({
     submitHandler : function (event){
          event.preventDefault ();
          alert (this.refs.checked.getDOMNode().checked);
     },
     render : function (){
          return (
               <form onSubmit = {this.submitHandler}>
                    <input ref = "checked" type = "checkbox" value = "A" defaultChecked = "true" />
                    <br />
                    <button type = "submit">Speak</button>
               </form>
          );
     }
});
//约束的
var MyForm = React.createClass ({
     getIntialState : function (){
          return {
               checked : true
          };
     },
     handleChange : function (event){
          this.setState ({
               checked : event.target.checked
          });
     },
     submitHandler : function (event){
          event.preventDefault ();
          alert (this.state.checked);
     },
     render : function (){
          return (
               <form onSubmit = {this.submitHandler}>
                    <input type = "checkbox" value = "A" checked = {this.state.checked} onChange = {this.handleChange} />
                    <br />
                    <button type = "submit">Speak</button>
               </form>
          );
     }
});
```
这两个例子中 ```<input />```的值一直都是A 只有checked的状态在变化

#### 表单元素的name属性
在React中 name属性对于表单元素并没有那么重要 因为约束组件已经把值存储到了state中 并且表单的提交事件也会遭到拦截 

在获取表单值的时候 name属性并不是必须的 对于非约束的表单组件来说 也可以使用ref来直接访问表单元素

即便如此 name仍是表单组件中非常重要的一部分

* name属性可以让第三方表单序列化类库在React中正常工作
* 对于仍然使用传统提交方式的表单来说 name属性是必须的
* 在用户的浏览器中 name被用在自动填写常用信息中 比如用户地址
* 对于非约束的单选框组件来讲 name是有必要的 它可以作为这些组件分组的依据 确保在同一时刻 同一表单中拥有同样的name单选框只有一个被选中 如果不使用name属性 这一行为可以使用约束的单选框实现

栗子

把状态存储在MyForm组件中 实现了非约束单选框具备的分组功能(**这里没有name属性!**)
```JSX
var MyForm = React.createClass ({
     getInitialState : function (){
          return {
               radio : "B"
          };
     },
     handleChange : function (){
          this.setState ({
               radio : event.target.value
          });
     },
     submitHandler : function (event){
          event.preventDefault ();
          alert (this.state.radio);
     },
     render : function (){
          return (
               <form onSubmit = {this.submitHandler}>
                    <input type = "radio" value = "A" checked = {this.state.radio == "A"} onChange = {this.handleChange} /> A
                    <br />
                    <input type = "radio" value = "B" checked = {this.state.radio == "B"} onChange = {this.handleChange} /> B
                    <br />
                    <input type = "radio" value = "C" checked = {this.state.radio == "C"} onChange = {this.handleChange} /> C
                    <br />
               </form>
          );
     }
});
```