### 动画
现在我们能够编写一组复杂的组件了 接下来美化一下它们 

React的TransitionGroup插件配合CSS3可以让我们在项目中整合动画效果的工作变得易如反掌

React选择了一种偏声明式的方法来实现动画

**CSS渐变组(CSS TransitionGroup)**会在合适的渲染及重渲染时间点有策略的添加和移动元素的class 以此来简化将CSS动画应用于渐变的过程 这意味着唯一需要你完成的任务就是给这些class写明合适的样式

间隔渲染以牺牲性能为代价提供了更多的扩展性和可控性 这种方法需要更多次的渲染 但同时也允许你为CSS之外的内容(比如滚动条的位置及canvas绘图)添加动画

#### CSS渐变组
大栗子

问卷编辑器中渲染问题列表：
```JSX
<ReactCSSTransitionGroup transitionName ='question'>
     {questions}
</ReactCSSTransitionGroup>
```
ReactCSSTransitionGroup是一款插件 它在文件最顶部通过```var ReactCSSTransitionGroup = React.addons.ReactCSSTransitionGroup;```语句被引入
它会自动在合适的时候处理组件的重渲染 同时根据当时的渐变状态调整渐变组的class以便实现组件样式的改变

####给渐变class添加样式
按照惯例 为元素添加```transitionName = 'question'```意味着给它添加了4个class：```question-enter```、```question-enter-active```、```question-leave```及```question-leave-active``` 当子组件进入或退出ReactCSSTransitionGroup时 CSSTransitionGroup插件会自动添加或移除这些class

栗子
```css
.survey-editor .question-enter {
     transform : scale(1.2);
     transform : transform 0.2s cubic-bezier(.97 , .85 , .5 , 1.21)
}
.survey-editor .question-enter-active {
     transform : scale(1);
}
.survey-editor .question-leave {
     transform : translateY(0);
     opacity : 0;
     transition : opacity 1.2s , transform 1s cubic-buzier (.52 , -0.25 , .52 , .95);
}
.survey-editor .question-leave-active {
     opacity : 0;
     transform : translateY(-100%);
}
```
注意：survey-editor选择器并不是ReactCSSTransitionGroup需要的 它们只是简单的用来确保这些样式只会在编辑器里生效

#### 渐变生命周期
question-enter与question-enter-active的区别在于：

* question-enter这个class是组件被添加到渐变组后立即添加上的
* question-enter-active是在下一轮渲染时添加的

默认情况下渐变组同时启用了进入和退出的动画 你可以通过给组件添加```transitionEnter = {false}```或```transitionLeave = {false}```属性来禁用其中一个或全部禁用 除了可以控制选择哪些动画效果外 我们还能根据一个可配置的值在特定的情况下禁用动画

栗子：
```JSX
<ReactCSSTransitionGroup tansitionName = 'question'
     transitionEnter = {this.props.enableAnimations}
     transitionLeave = {this.props.enableAnimations}>
     {questions}
</ReactCSSTransitionGroup>
```
#### 使用渐变组的隐患
使用渐变组时主要有两个重要的隐患需要注意：

* 渐变组会延迟子组件的移除直到动画完成 这意味着如果你把一个列表的组件包裹进一个ReactCSSTransitionGroup中 却没有为transitionName属性指定的class明确任何CSS 这些组件将永远无法被移除－－甚至当你尝试不再渲染它们时也不可以

* 渐变组的每一个子组件都必须设置一个独一无二的key属性 渐变组使用这个属性来判断组件究竟是进入还是退出 因此如果没有设置key属性动画可能无法执行 同时组件也会变的无法移除

注意：即使渐变组只有一个子元素 它也需要设置一个key属性

#### 间隔渲染
使用CSS3动画能够获得巨大的性能提升并拥有简洁的代码 但它们并不总是解决问题的正确工具 有些时候你必须要为比较老的、不支持CSS3的浏览器做兼容 还有些时候你想为CSS属性之外的东西添加动画 比如滚动条位置或者canvas绘画 在这些情况下 间隔渲染能够满足我们的要求 但是相比于CSS动画来说 它会带来一定的性能损耗

**间隔渲染最基本的思想就是周期性的触发组件的状态更新 以明确当前处于整个动画时间中的什么阶段 通过组件的render方法中加入这个状态值 组件能够在每次状态更新触发的重新渲染中正确表示当前的动画阶段**

因为这种方法涉及多次重渲染 所以通常最好和requestAnimationFrame一起使用以避免不必要的渲染 不过在requestAnimationFrame不被支持或不可用的情况下降级到不那么智能的setTimeout就是唯一的选择了

* 使用requestAnimationFrame实现间隔渲染

假设你希望使用间隔渲染将一个div从屏幕的一边移向另一边 可以通过给它添加```position : absolute```并随着时间变化不停更新left或top属性来实现 根据消耗时间内的变化总量 用requestAnimationFrame来实现这个动画应该可以得出一个流畅的画面

栗子：
```JSX
var Positioner = React.createClass ({
     getInitialState : function (){
          return {position : 0};
     },
     resolveAnimationFrame : function (){
          var timestamp = new Date ();
          var timeRemaining = Math.max (0 , this.props.animationCompleteTimestamp - timestamp);
          if (timeRemaining>0){
               this.setState ({position : timeRemaining});
          }
     },
     componentWillUpdate : function (){
          if (this.props.animationCompleteTimestamp){
               requestAnimationFrame (this.resolveAnimationFrame);
          }
     },
     render : function (){
          var divStyle = {left : this.state.position};
          return <div style = {divStyle}>This will animate!</div>
     }
});
```
注解：

1. 组件的props中设置了一个名为animationCompleteTimestamp的值 它和requestAnimationFrame的回调中返回的时间戳一起被用来计算剩余多少位移 计算的结果存在this.state.position中 而render方法会用它来确定div的位置

2. 由于requestAnimationFrame被componentWillUpdate方法调用 所以只要组件的props有任何变动(比如改变了animationCompleteTimestamp)它就会被触发 它又包含了在resolveAnimationFrame中的this.setState调用 这意味着一旦animationCompleteTimestamp被设置 组件就会自动调用后续的requestAnimationFrame方法 直到当前时间超过了animationCompleteTimestamp为止

3. 注意 这套逻辑只在基于时间戳的情况下成立 对animationCompleteTimestamp所做的改变会触发逻辑 而this.state.position的值完全依赖于当前时间与animationCompleteTimestamp的差 正因如此 render方法可以自由的在各种动画中使用this.state.position 包括设置滚动条位置 在canvas上绘画 以及任何中间状态

#### 使用setTimeout实现间隔渲染
尽管requestAnimationFrame总体上能够以最小的性能损耗实现最流畅的画面 但它在较老的浏览器上是无法使用的 而且它被调用的次数可能比你想象的更频繁(也更加无法预测) 在这些情况下你可以使用setTimeout
```JSX
var Positioner = React.createClass ({
     getInitialState : function (){return {position : 0};},
     resolveSetTimeout : function (){
          var timestamp = new Date ();
          var timeRemaining = Math.max(0,this.props.animationCompleteTimestamp - timestamp);
         if (timeRemaining>0){
               this.setState ({position : timeRemaining});
          }
     },
     componentWillUpdate : function (){
          if (this.props.animationCompleteTimestamp){
               setTimeout (this.resolveSetTimeout , this.props.timeoutMs);
          }
     },
     render : function (){
          var divStyle = {left : this.state.position};
          return <div style = {divStyle}>This will animate!</div>
     }
});
```
由于setTimeout接受一个显示的时间间隔 而requestAnimationFrame是自己来决定这个时间间隔的 因此这个组件需要额外依赖一个变量this.props.timeoutMs 以此来明确要使用的间隔

开源库ReactTweenState基于这种动画方式提供了一套方便的抽象接口

#### 总结
使用这些动画技术 你现在可以：

1. 在状态改变过程中 使用CSS3和渐变组高效的应用渐变动画
2. 使用requestAnimationFrame为CSS之外的东西添加动画 如滚动条位置或Canvas绘画
3. 当requestAnimationFrame不被支持时降级到setTimeout方法