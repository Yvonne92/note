### ReactJs小结
* ReactJs是基于组件化的开发，所以最终你的页面应该是由若干个小组件组成的大组件。

* 可以通过属性，将值传递到组件内部，同理也可以通过属性将内部的结果传递到父级组件；要对某些值的变化做DOM操作的，要把这些值放到state中。

* 为组件添加外部css样式时，类名应该写成className而不是class;

* 添加内部样式时，应该是style={{opacity:this.state.opacity}}而不是style=“opacity:{this.state.opacity};"。

* 组件名称首字母必须大写。

* 变量名用{}包裹，且不能加双引号。

* ReactJs 从属关系

* 如果组件 Y 在 render() 方法是创建了组件 X，那么 Y 就拥有 X。上面讲过，组件不能修改自身的 props - 它们总是与它们拥有者设置的保持一致。这是保持用户界面一致性的关键性原则。

### 插件

**React.addons** 是为了构建 React 应用而放置的一些有用工具的地方。此功能应当被视为实验性的，但最终将会被添加进核心代码中或者有用的工具库中.

**TransitionGroup** 和 **CSSTransitionGroup**，用于处理动画和过渡，这些通常实现起来都不简单，例如在一个组件移除之前执行一段动画。

**LinkedStateMixin**，用于简化用户表单输入数据和组件 state 之间的双向数据绑定。

**classSet**，用于更加干净简洁地操作 DOM 中的 class 字符串。

**cloneWithProps**，用于实现 React 组件浅复制，同时改变它们的 props 。

**update**，一个辅助方法，使得在 JavaScript 中处理不可变数据更加容易。

**PureRednerMixin**，在某些场景下的性能检测器。

要使用这些插件，需要用 **react-with-addons.js**（和它的最小化副本）替换常规的 **React.js** 。
当通过npm使用react包的时候，只要简单地用 **require('react/addons')** 替换 **require('react')** 来得到带有所有插件的React。

### 在JSX中使用样式


尽管在大部分场景下我们应该将样式写在独立的CSS文件中，但是有时对于某个特定组件而言，其样式相当简单而且独立，那么也可以将其直接定义在JSX中。在JSX中使用样式和真实的样式也很类似，通过style属性来定义，但和真实DOM不同的是，属性值不能是字符串而必须为对象，例如：
```JSX
<div style={{color:'#ff0000',fontSize:'14px'}}>Hello World.</div>
```
乍一看，这段JSX中的大括号是双的，有点奇怪，但实际上里面的大括号只是标准的JavaScript对象表达式，外面的大括号是JSX的语法。所以，样式你也可以先赋值给一个变量，然后传进去，代码会更易读：
```JSX
var style={
	color:'#ff0000',
	fontSize:'14px'
};

var node =<div style={style}>HelloWorld.</div>;
```
在JSX中可以使用所有的的样式，基本上属性名的转换规范就是将其写成驼峰写法，例如“background-color”变为“backgroundColor”, “font-size”变为“fontSize”，这和标准的JavaScript操作DOM样式的API是一致的。
