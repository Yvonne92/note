###数据流

在React中数据流是单向的——从父节点传递到子节点 如果顶层某个组件的某个prop改变了React会递归的向下遍历整个组件树 重新渲染所有使用这个属性的组件

React组件内部具有自己的状态 这些状态只能在组件内修改 它接受props和state作为参数 返回一个虚拟的DOM表现

* **props是什么？**
* **state是什么？**
* **什么时候用props什么时候用state？**

####props

props是properties的缩写 它可以将任意类型的数据传递给组件

可以在挂载组件的时候设置它的props：
```JSX
var surveys=[{title:'Superheroes'}];
<ListSurveys surveys={surveys}/>
```
可以通过this.props访问props 但不能通过这种方式修改它
一个组件不可以自己修改自己的props
在JSX中可以把props设置为字符串：
```JSX
<a href='/surveys/add'>Add Survey</a>
```
//也可以使用{}语法来设置 注入Javascript传递任意类型的变量
```JSX
<a href={'/surveys/'+survey.id}>{survey.title}</a>
```
还可以使用JSX的展开语法把props设置成一个对象：
```JSX
var ListSurveys=React.createClass({
     render:function(){
          var props={
               one:'foo',
               two:'bar'
          };
     return <SurveyTable{…props}/>
     }
});
```
props还可以用来添加事件处理器
```JSX
var SaveButton=React.createClass({
     render:function(){ 
          return  (
               <a className='button save' onClick={this.handleClick}>Save </a>
          );
     },
     handleClick:function(){
          ...
     }
});
```
#### PropTypes(非强制性)

通过在组件中定义一个配置对象 React提供了一种炎症props的方式：
```JSX
var SurveyTableRow=React.createClass({
     propTypes:{
          survey:React.PropTypes.shapes({
               id:React.PropTypes.number.isRequired
          }).isRequired,
          onClick:React.PropTypes.func
     },
     ...
});
```
组件初始化时 如果传递的属性和propTypes不匹配，则会打印一个console.warn日志
如果是可选的配置则可以去掉.isRequired

#### getDefaultProps

可以为组件添加getDefaultProps函数来设置属性的默认值－>只针对非必要属性
```JSX
var SurveyTable=React.createClass({ 
     getDefaultProps:function(){
          return {
               surveys:[]
          };
     }
     ...
});
```
getDefaultProps并不是在组件实例化时被调用而是在React.createClass调用时就被调用了 返回值会被缓存起来－>不能在getDefaultProps中使用任何特定的实例数据

#### state
* 每个组件都可以拥有自己的state state与props的区别在于前者只存在于组件的内部
* state可以通过setState来修改也可以使用getInitialState方法提供一组默认值
* setState被调用－>render就被调用－>render返回值变化－>虚拟DOM更新－>真实DOM更新
* 万万不能修改this.state 要通过this.setState方法修改

#### 总结
* 使用props在整个组件树中传递数据和配置
* 避免在组件内部修改this.props或调用this.setProps 把props当作只读的
* 使用props来做事件处理器 与子组件通信
* 使用state存储简单的视图状态 比如下拉框是否可见这样的状态
* 使用this.setState来设置状态 而不要用this.state直接修改状态