### 表单

#### 多个表单元素与change处理器
在使用约束的表单组件时 没人愿意重复地为每个组件编写change处理器

还好有几种方式可以在React中重用一个事件处理器

**第一个栗子：**

* 通过.bind传递其他参数
```JSX
var MyForm = React.createClass ({
     getInitialState : function (){
          return {
               given_name : "",
               family_name : ""
          };
     },
     handleChange : function (name,event){
          var newState = { };
          newState[name] = event.target.value;
          this.setState(newState);
     },
     submitHandler : function (event){
          event.preventDefault ();
          var words = [
               "Hi",this.state.given_name, this.state.family_name
          ];
          alert (words.join(""));
     },
     render : function (){
          return (
               <form onSubmit = {this.submitHandler}>
                    <label htmlFor = "given_name">Given Name : </label>
                    <br />
                    <input type = "text" name = "given_name" value = {this.state.given_name} onChange = {this.handleChange.bind(this,"given_name")} />
                    <br />
                    <label htmlFor = "family_name">Family Name : </label>
                    <br />
                    <input type = "text" name = "family_name" value = {this.state.family_name} onChange = {this.handleChange.bind(this,"family_name")} />
                    <br />
                    <button type = "submit">Speak</button>
               </form>
          );
     }
});
```
**第二个栗子：**

* 使用DOMNode的name属性来判断需要更新哪个组件的状态
```JSX
var MyForm = React.createClass ({
     getInitialState : function (){
          return {
               given_name : "",
               family_name : ""
          };
     },
     handleChange : function (event){
          var newState = {};
          newState[event.target.name] = event.target.value;
          this. setState(newState);
     },
     submitHandler : function (event){
          event.preventDefault ();
          var words = [
               "Hi" , this.state.given_name , this.state.family_name
          ];
          alert (words.join[""]);
     },
     render : function (){
          return (
               <form onSubmit = {this.submitHandler}>
                    <label htmlFor = "given_name">Given Name : </label>
                    <br />
                    <input type = "text" name = "given_name" value = {this.state.given_name} onChange = {this.handleChange.bind(this,"given_name")} />
                    <br />
                    <label htmlFor = "family_name">Family Name : </label>
                    <br />
                    <input type = "text" name = "family_name" value = {this.state.family_name} onChange = {this.handleChange.bind(this,"family_name")} />
                    <br />
                    <button type = "submit">Speak</button>
               </form>
          );
     }
});
```
**React还在addon中提供了一个mixin－－> React.addons.LinkedStateMixin**

React.addons.LinkedStateMixin为组件提供了一个linkState方法 linkState返回一个对象 包含value和requestChange两个属性

* **value**－－> 根据提供的name属性从state中获取对应的值

* **requestChange**－－> 是一个函数 使用新的值更新同名的state

this.linkState('given_name');


栗子
```JSX
//返回
value : this.state.given_name,
requestChange : function (newValue){
     this.setState ({
          given_name : newValue
     });
}
```
需要把这个对象传递给一个React特有的非DOM属性valueLink valueLink使用对象提供的value更新表单域的值 并提供一个onChange处理器 当表单域更新时使用新的值调用requestChange
```JSX
var MyForm = React.createClass ({
     mixins : [React.addons.LinkedStateMixin],
     getInitialState : function (){
          return {
               given_name : "",
               family_name : ""
          };
     },
     submitHandler : function (){
          event.preventDefault ();
          var words = [
               "Hi" , this.state.given_name , this.state.family_name
          ];
          alert (words.join(""));
     },
     render : function (){
          return (
               <form onSubmit = {this.submitHandler}>
                    <label htmlFor = "given_name">Given Name : </label>
                    <br />
                    <input type = "text" name = "given_name" valueLink = {this.LinkState(‘given_name’)} />
                    <br />
                    <label htmlFor = "family_name">Family Name : </label>
                    <br />
                    <input type = "text" name = "family_name" valueLink = {this.LinkState('family_name')} />
                    <br />
                    <button type = "submit">Speak</button>
               </form>
          );
     }
});
```
这种方法便于控制表单域 把其值保存在父组件的state中 而且 其数据流仍然与其他约束的表单保持一致

但是使用这种方式往数据流中添加定制功能时复杂度会增加－－>只在特定情形下使用
传统约束表单组件提供了同样的功能而且更加灵活

#### 自定义表单组件
编写自定义组件的时候 接口应当与其他表单组件保持一致 这可以帮助用户理解代码 明白如何使用自定义组件 且无需深入到组件的实现细节里

栗子

创建一个自定义单选框组件 其接口与React的select组件保持一致
```JSX
var Radio = React.createClass ({
     propTypes : {
          onChange : React.PropTypes.func
     },
     getInitialState : function (){
          return {
               value : this.props.defaultValue
          };
     },
     handleChange : function (event){
          if (this.props.onChange){
               this.props.onChange(event);
          }
          this.setState ({
               value : event.target.value
          });
     },
     render : function (){
          var children = {}
          var value = this.props.value || this.state.value

          React.Children.forEach(this.props.children , function(child , i){
               var label = (
                    <label>
                         <input type = "radio" name = {this.props.name} value = {child.props.vlaue} checked = {child.props.value ==vlaue} onChange = {this.handleChange} />
                         <br />
                    </label>
               );
               children[‘label’ + i ] = label;
          }.bind(this));
          return this.transferPropsTo(<span>{children}</span>);
     }
});
```
我们创建了一个同时支持约束和非约束接口的约束组件

首先要确保的是传递给onChange的属性值的类型必须是函数 然后把defaultValue保存到state中

组件每次渲染时都会基于传递给组件的选项(作为子元素传进来)来创建标签和单选框 同时我们还需要确保每次渲染时插入的子元素都拥有相同的key 这样React会把```<input />```保留在DOM中 当用户使用键盘时 React可以维护当前的focus状态

接下来设置状态value、name和checked 绑定onChange处理器 然后渲染新的子元素
```JSX
//非约束的
var MyForm = React.createClass ({
     submitHandler : function (event){
          event.preventDefault ();
          alert (this.refs.radio.state.value);
     },
     render : function (){
          return(
               <form onSubmit = {this.submitHandler}>
                    <Radio ref = "radio" name = "my_radio" defaultValue = "B">
                         <option value = "A">First Option</option>
                         <option value = "B">Second Option</option>
                         <option value = "C">Third Option</option>
                    </Radio>
                    <button type = "submit">Speak</button>
               </form>
          );
     }
});
```
当使用非约束组件时 我们不得不改变下接口 因为**调用this.refs.radio的.getDOMNode方法时 得到的是DOMNode 而不是处于激活状态的```<input /> ```**在React中我们无法更改getDOMNode()的实现来重写这种行为

因为值已经被保存到了组件的state中 所以我们无须再从DOMNode中获取 直接从state中获取即可
```JSX
//约束的
var MyForm = React.create ({
     getInitialState : function (){
          return {
               my_radio : "B"
          };
     },
     handleChange : function (event){
          this.setState ({
               my_radio : event.target.value
          });
     },
     submitHandler : function (event){
          event.preventDefault ();
          alert (this.state.my_radio);
     },
     render : function (){
          return (
               <form onSubmit = {this.submitHandler}>
                    <Radio name = "my_radio" value = {this.state.my_radio} onChange = {this.handleChange}>
                         <option value = "A">First Option</option>
                         <option value = "B">Second Option</option>
                         <option value = "C">Third Option</option>
                    </Radio>
                    <button type = "submit">Speak</button>
               </form>
          );
     }
});
```
作为约束组件来使用时 操作起来与一个多选框没什么区别 传递给onChange的事件直接来自于被选中的```<input />``` 因此 你可以通过它来读取当前的值

#### Focus
因为React的表单并不总是在浏览器加载时被渲染的 所以表单的输入域的自动聚焦操作起来有点不一样 React实现了autoFocus属性 因此在组件第一次挂载时 如果没有其他的表单聚焦时 React就会把焦点放到这个组件对应的表单域中 
```JSX
//jsx
<input type = "text" name = "given_name" autoFocus = "true" />
```
还有一种方法是调用DOMNode的focus()方法 手动设置表单域聚焦

#### 可用性
使用React编写出来的组件常常缺乏可用性

例如 有的表单缺乏对键盘操作的支持 要提交表单就只能通过超链接的onClick事件 而无法通过在键盘上敲击回车键来实现 而这明明是HTML表单默认的提交方式

要编写具有高可用性的好组件其实也不难 只是编写组件时需要花时间进行更多思考 好用的组件源于对各种细节的雕琢

#### 总结
React把状态管理从DOM中提取到组件中 以此来帮助我们管理表单的状态 这允许我们更严格地控制表单元素的操作 创建更复杂的组件用于项目

表单是用户在应用中会碰到的最复的杂交互之一 在创建和组织表单组件时 要时刻考虑的重要一点就是表单的可用性
