### 组件的复合
本质上一个组件就是一个JavaScript函数 它接受属性(Props)和状态(State)作为参数 并输出渲染好的HTML

组件一般被用来呈现和表达应用的某部分数据－－>可以把React组件理解为HTML元素的扩展

#### 扩展HTML
* React＋JSX允许我们使用类似HTML的语法创建自定义元素 比起单纯的HTML 它们还可以控制生命周期中的行为 这些都是从React.createClass开始的
* 相较于继承 React偏爱复合 通过结合小巧的简单的组件和数据对象构造大而复杂的组件－－>React组件是不可扩展的 而是通过组件之间的组合来构建应用

#### 组件复合的栗子
一个渲染选择题的组件要满足以下几个条件：
* 接收一组选项作为输入
* 把选项渲染给用户
* 只允许用户选择一个选项
组件的层级从上往下看是这样的：

**MultipleChoice －> RadioInput －> Input(type="radio")**

－-> 表示“有一个” 这是**组合模式**的特征

#### 组装HTML
开始组装组件

React在React.DOM.input的命名空间预定义了input组件－－>把input封装进一个RadioInput组件

**描述想输出的界面**
```JSX
var AnswerRadioInput=React.createClass({
     render:function(){
          return (
               <div className="radio">
                    <label>
                         <input type="radio"/>
                         Label Text
                    </label>
               </div>
          );     
     }
});
```
#### 添加动态属性
定义父元素必须传给单选框的那些属性

* 这个输入框代表什么值或者选项？(必填)
* 用什么文本来描述它？(必填)
* 这个输入框的name是什么？(必填)
* 也许需要自定义id
* 也许要重载它的默认值

**将自定义的input属性类型添加到类的PropTypes对象中**
```JSX
var AnswerRadioInput=React.createClass({
     propTypes:{
          id:React.PropTypes.string,
          name:React.PropTypes.string.isRequired,
          label:React.PropTypes.string.isRequired,
          value:React.PropTypes.string.isRequired,
          checked:React.PropTypes.bool
     },
     ...
});
```
**将非必需的属性添加到getDefaultProps方法中**(在每个新实例中如果父组件没有提供它们的值这些值就会被调用)
```JSX
var AnswerRadioInput=React.createClass({
     propTypes:{...},
     getDefaultProps:function(){
          return {
               id:null,
               checked:false
          };
     },
     ...
});
```
#### 追踪状态
**组件需要记录随时间变化的数据** 尤其是对于每个实例来说都要求是唯一的id

**定义初始状态**
```JSX
var AnswerRadioInput=React.createClass({
     propTypes:{...},
     getDefaultProps:function(){...},
     getInitialState:function(){
          var id=this.props.id?this.props.id:uniqueId('radio-');
          return {
               checked:!!this.props.checked,
               id:id,
               name:id
          };
     },
     ...
});
```
**更新渲染标记 获取新动态的状态和属性**
```JSX
var AnswerRadioInput=React.createClass({
     propTypes:{...},
     getDefaultProps:function(){...},
     getInitialState:function(){...},
     render:function(){
          return (
               <div className="radio">
                    <label htmlFor={this.props.id}>
                         <input type="radio"
                                   name={this.props.name}
                                   id={this.props.id}
                                   value={this.props.value}
                                   checked={this.state.checked}/>
                         {this.props.label}
                    </label>
               </div>
          );
     } 
});
```
#### 整合到父组件当中
**接下来构建下一层－－AnswerMultipleChoiceQuestion**(渲染一列选项让用户选择)

按照之前的模式 创建组件基本的HTML和默认属性
```JSX
var AnswerMultipleChoiceQuestion=React.createClass({
     propTypes:{
          value:React.PropTypes.string,
          choice:React.PropTypes.array.isRequired,
          onCompleted:React.PropTypes.func.isRequired
     },
     getInitialState:function(){
          return {
               id:uniqueId('multiple-choice-'),
               value:this.props.value
          };
     },
     render:function(){
          return (
               <div className="form-group">
                    <label className="survey-item-label" htmlFor={this.state.id}>
                         {this.props.label}
                    </label>
                    <div className="survey-item-content">
                         <AnswerRadioInput .../>
                         ... 
                         <AnswerRadioInput .../>
                    </div >
               </div>
          );
     }
});
```
**生成一系列单选框组件－－>映射选项列表－－>添加辅助函数－－>每一项转化为组件**
```JSX
var AnswerMultipleChoiceQuestion=React.createClass({
     ...
     renderChoices:function(){
          return this.props.choice.map(function(choice,i){
               return AnswerRadioInput({
                    id:"choice-"+i,
                    name:this.state.id,
                    label:choice, 
                    value:choice, 
                    checked:this.state.value === choice 
               });
          }.bind(this));
     },
     render:function(){
          return (
               <div className="form-group">
                    <label className="survey-item-label" htmlFor={this.state.id}>
                         {this.props.label}
                     </label>
                    <div className="survey-item-content">
                         {this.renderChoices()}
                    </div>
               </div>
          );
     }
});
```
**接下来再渲染一列选项**
```JSX
<AnswerMultipleChoiceQuestion choices={arrayOfChoices} .../>
```
#### 父组件、子组件关系
* 子组件与其父组件通信最简单的方式就是使用属性(props)
* 父组件需要通过属性传入一个回调函数(callback) 子组件在需要时调用

**将单选框变化通知给父组件**

**定义AnswerMultipleChoiceQuestion在其子组件变更后需要做什么－－>添加handleChanged方法－－>将其传给所有AnswerRadioInput组件**
```JSX
var AnswerMultipleChoiceQuestion=React.createClass({
     ... 
     handleChanged:function(value){
          this.setState({value:value});
          this.state.onCompleted(value);
     },
     renderChoices:function(){
          return this.props.choice.map(function(choice,i){
               return AnswerRadioInput({
                    ...
                    onChanged:this.handleChanged
               });
          }.bind(this));
     },
     ...
});
```
**给每个单选框监听用户更改－－>把数值向上传给父组件 (将事件处理器关联到输入框的onChange事件上)**
```JSX
var AnswerRadioInput=React.createClass({
     propTypes:{
          ...
          onChanged:React.PropTypes.func.isRequired
     },
     handleChanged:function(e){
          var checked=e.target.checked;
          this.setState({checked:checked}); 
          if(checked){
               this.props.onChanged(this.props.value);
          }
     },
     render:function(){
          return (
               <div className="radio">
                    <label htmlFor={this.state.id}>
                         <input type="radio"
                              ... 
                              onChange={this.handleChanged}/>
                         {this.props.label}
                    </label>
               </div>
          );
     }
});
```
组件的复合只是React提供的用于定制和特殊化组件的**方式之一**

React的**mixin**提供了另一种途径帮助我们来共享通用的代码
