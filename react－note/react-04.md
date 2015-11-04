### 事件处理

React通过将事件处理器绑定到组件上来处理事件

#### 绑定事件处理器

React处理的事件本质上和原生的JavaScript事件一样：

MouseEvents事件用于点击事件处理器 change事件用于表单元素变化 等等

所有事件在命名上与JavaScript规范一致 并在相同情景被触发

例如在Save按钮上绑定onClick事件处理器
```JSX
<button className="btn btn-save" onClick={this.handleSaveClicked}>
     save
</button>
```
React对处理各种事件类型提供了了友好的支持 其中绝大部分事件不需要额外的处理就能工作 但是触控事件需要通过以下代码手动启用：
```JSX
React.initializeTouchEvents(true);
```
#### 事件和状态

让组件随着用户的输入而改变 例如在调查问卷里对菜单的拖拽

#### 根据状态进行渲染

例如展开问题清单 需要充分利用React组件的内部状态 

组件状态的默认是null 但可以用getInitialState方法将其初始化为合理的值
```JSX
getInitialState:function(){
     return {
          dropZoneEntered:false,//表示当前用户没有拖拽任何内容到放置区域
          title:'',
          introduction:'',
          questions:[]
     };
}
```
现在可以在render方法里读取this.state向用户展示表单里的数据了
```JSX
render:function(){
     var questions=this.state.questions;
     ...
     return (
          <div className='survey-editor'>
               <div className='survey-canvas col-md-9'>
                    <SurveyForm
                         title={this.state.title}
                         introduction={this.state.introduction}
                         onChange={this.handleFormChange}
                    />
               </div>
               ... 
          </div>
     );
}
```
#### 更新状态

更新组件的内部状态会触发组件重绘

在拖拽的**事件处理器方法**中更新状态－>再次运行render－>读取this.state的新数据

更新组件两种方案：
* 组件的setState方法－－－>仅仅把传入的对象合并到已有的state对象
* 组件的replaceState方法－－－>一个新的state对象完整的替换掉原有的state

栗子
```JSX
getInitialState:function(){
     return {
          dropZoneEntered:false,
          title:'Fantastic Survey',
          introduction:'This survey is fantastic!',
          questions:[]
     };
}
this.setState({title:"Fantastic Survey 2.0"})
{
     dropZoneEntered:false,
     title:'Fantastic Survey 2.0',
     introduction:'This survey is fantastic!',
     questions:[]
};
this.replaceState({title:"Fantastic Survey 2.0"})
{
     title:'Fantastic Survey 2.0'
};
```
**事件处理器方法**
```JSX
handleFormChange:function(formData){
     this.setState(formData);
},
...
```
#### 事件对象

用户输入更多信息

栗子
```JSX
var AnswerEssayQuestion=React.createClass({
     handleComplete:function(event){
          this.callMethodOnProps('onCompleted',event.target.value);
     },
     render:function(){
          return (
               <div className="form-group" >
                    <label className="survey-item-label">
                         {this.props.label}
                    </label>
                    <div className="survey-item-content">
                         <textarea className="form-control" rows="3" onBlur={this.handleComplete} />
                    </div>
               </div>
          );
     }
});
```
* handleComplete方法会接受一个事件对象 并通过存取event.target.value值为textarea赋值
* 在事件处理器中使用event.target.value是获取表单中input值的常规方法 尤其在onChange事件处理器中
* callMethodOnProps是由一个叫做PropsMethodMixin的mixin提供的

SyntheticEvent

* React把原生事件封装在SyntheticEvent实例里 而不是将原生的浏览器事件对象传给事件处理器
* SyntheticEvent在表现和功能上都与浏览器的原声事件一致 并且可以消除某些跨浏览器差异
* 对于那些需要浏览器原声事件的场景 可以通过SyntheticEvent的nativeEvent属性访问

#### 总结

从用户输入到更新界面 处理步骤：

1. 在React组件上绑定事件处理器

2. 在事件处理器中更新内部状态 组件状态的更新会触发重绘

3. 实现组件的render函数用来渲染this.state的数据