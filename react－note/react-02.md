### 生命周期方法（按照组件中的调用顺序看每个方法）

#### 实例化：
初次创建：
* **getDefaultProps**
* **getInitialState**
* **componentWillMount**
* **render**
* **componentDidMount**

后续应用：（getDefaultProps不在列表中）：
* **getInitialState**
* **componentWillMount**
* **render**
* **componentDidMount**

#### 存在期：

随着应用状态改变 以及组件逐渐受到影响 组件调用顺序：
* **componentWillReceiveProps**
* **shouldComponentUpdate**
* **componentWillUpdate**
* **render**
* **componentDidUpdate**

#### 销毁&清理期
当组件被使用完成后 componentWillUnmount方法将会继续被调用 清理实例自身


#### 实例化
##### **getDefaultProps**
只会被调用一次 新建实例若父辈组件无指定的props值 则返回的是实例默认的props值
任何复杂的值在实例中都是共享而不是拷贝或克隆 

##### **getInitialState**
初始化每个实例的state 与getDefaultProps不同的是每次实例被创建时该方法都会被调用 在这个方法里你已经可以访问this.props

##### **componentWillMount**
首次渲染之前被调用 是render方法调用前最后一次修改state的机会

##### **render**
创建虚拟DOM表示组件的输出 render是组件里唯一必须的方法 render方法必须满足下面几点：
* **只能通过this.props和this.state访问数据**
* **可以返回null、false或者任何react组件**
* **只能出现一个顶级组件(不能返回一组数组)**
* **不能改变组件的状态或者修改DOM的输出**
* **创建出的结果是虚拟的DOM React随后会和真实的DOM做对比 并判断是否有必要做出修改**

##### **componentDidMount**
在render方法成功调用并且真实的DOM已经渲染之后 
可以用componentDidMount内部通过this.getDOMNode()方法访问到它
这就是你可以用来访问原始DOM的**_生命周期钩子函数_** 
可以将JQuery或者对渲染出的DOM的操作挂载到这个方法上
在React运行在服务器端时componentDidMount方法不会被调用

#### 存在期
此时组件已经渲染好并且用户可以交互 随着用户改变组件或者整个应用的state便会有新的state流入组件库 并且我们将会获得操作它的机会
##### **componentWillReceiveProps**
在任意时刻组件的props都可以通过父辈组件来更改 出现这种情况时 componentWill－ReceiveProps方法会被调用 你将获得更改props对象及更新state的机会
```JSX
componentWillReceiveProps:function(nextProps){
     if(nextProps.checked !== undefined){
          this.setState({
               checked:nextProps.checked
          });
     }
}
```
##### **shouldComponentUpdate**
* **通过shouldComponentUpdate方法在组件渲染时进行精确优化**

* **若你确定某个组件或者它任何子组件不需要渲染新的props或者state 则改方法返回false**

* **返回false是在告诉React要跳过调用render方法以及位于render前后的钩子函数：componentWillUpdate和componentDidUpdate**

* **该方法是非必需的 草率使用它会导致不可思议的bug**

* **若你使用了不可变的数据结构作为state 同时只在render方法中读取props和state中的数据 那你就可以放心的重写shouldComponentUpdate方法来比较新旧props和state了**

* **另一个_性能调优_的选项是React插件提供的PureRenderMixin方法 如果组件是纯净的 即对于相同的props和state 它总会渲染出一样的DOM那么这个mixin就会自动的调用shouldComponentUpdate方法来比较props和state 若比较一致则返回false**

##### **componentWillUpdate**
和componentWillMount类似 组件会在接收到新的props或者state进行渲染之前调用改方法
不可以在该方法中更新props或者state 应该借助componentWillReceiveProps方法在运行时更新state

##### **componentDidUpdate**
和componentDidMount类似 更新已经渲染好的DOM

#### 销毁&清理期
##### **componentWillUnmount**
在componentDidMount方法中添加的所有任务都需要在该方法中撤销

#### 反模式（把计算后的值赋给state）
在getInitialState方法中尝试通过this.props来创建state的做法是种反模式
当组件的state值与它所基于的props不同步 因而无法了解到render函数内部结构时可以认定为一种反模式

经计算后的值不应该赋给state
```JSX
getDefaultProps:function(){
     return {
          date:new Date()
     };
},
getInitialState:function(){
     return{
          day:this.props.date.getDay();
     }
},
render:function(){ 
     return <div>Day:{this.state.day}</div>;
}
```
在渲染时计算值是正确的
```JSX
getDefaultProps:function(){
     return{
          date:new Date()
     };
},
render:function(){ 
     var day=this.props.date.getDay();
     return <div>Day:{day}</div>;
}
```
如果你要的不是同步而是简单的初始化state 
可以在getInitialState方法中使用props
但要明确意图 比如为prop添加initial前缀
```JSX
getDefaultProps:function(){
     return{
          initialValue:'some-default-value'
     };
},
getInitialState:function(){ 
     return {
          value:this.props.initialValue
     };
},
render:function(){
     return <div>{this.props.value}</div>
}
```