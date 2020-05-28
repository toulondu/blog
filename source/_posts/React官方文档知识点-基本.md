---
title: React官方文档知识点-基本
date: 2020-05-28 17:56:23
categories:
- web前端
tags:
- React
- 基础
---

最近准备面试前端工作，很多很久没使用的东西是应该复习一下了。首当其冲就是前端领域的三大框架。在这里就不再比较这三个框架的优劣，我从15年底开始用react，也算是很早期的一批用户了，所以先从官方文档出发复习一下React的知识点，顺便也学习一下新的特性。

官方文档比较多，我全部看了一遍。也将其中的要点摘录来下来，如果有缘读到这篇文章的你恰巧没有那么多时间复习，可以大概地瞟一眼我的摘录笔记。大概会有三篇，基本、进阶和Context+Hook

# 正文
### 1.React不直接修改数据的原因：
- 简化复杂的功能，不直接在数据上修改可以让我们追溯并复用游戏的历史记录。
- 跟踪数据的改变，直接修改数据，很难跟踪到数据的改变，需要可变对象可以与改变之前的版本进行对比，这样整个对象树都需要被遍历一次。
跟踪不可变数据的变化相对来说就容易多了。如果发现对象变成了一个新对象，那么我们就可以说对象发生改变了。
- 确定在react中何时重新渲染，不可变性最主要的优势在于它可以帮助我们在 React 中创建 pure components。我们可以很轻松的确定不可变数据是否发生了改变，从而确定何时对组件进行重新渲染。


### 2.函数组件
如果你想写的组件只包含一个 render 方法，并且不包含 state，那么使用函数组件就会更简单。  
PS：当我们把 Square 修改成函数组件时，我们同时也把 onClick={() => this.props.onClick()} 改成了更短的 onClick={props.onClick}（注意两侧都没有括号）。
```
function Square(props) {
  return (
    <button className="square" onClick={props.onClick}>
      {props.value}
    </button>
  );
}
```

### 3.key
我们需要给每一个列表项一个确定的 key 属性，它可以用来区分不同的列表项和他们的同级兄弟列表项。  
每当一个列表重新渲染时，React 会根据每一项列表元素的 key 来检索上一次渲染时与每个 key 所匹配的列表项。如果 React 发现当前的列表有一个之前不存在的 key，那么就会创建出一个新的组件。如果 React 发现和之前对比少了一个 key，那么就会销毁之前对应的组件。如果一个组件的 key 发生了变化，这个组件会被销毁，然后使用新的 state 重新创建一份。

key 是 React 中一个特殊的保留属性（还有一个是 ref，拥有更高级的特性）。当 React 元素被创建出来的时候，React 会提取出 key 属性，然后把 key 直接存储在返回的元素上。虽然 key 看起来好像是 props 中的一个，但是你不能通过 this.props.key 来获取 key。React 会通过 key 来自动判断哪些组件需要更新。组件是不能访问到它的 key 的。

我们强烈推荐，每次只要你构建动态列表的时候，都要指定一个合适的 key。如果你没有找到一个合适的 key，那么你就需要考虑重新整理你的数据结构了，这样才能有合适的 key。

另外，设置key的元素是map() 方法中的元素，假如你的每个li都是包含在一个ListItem组件中返回的，你就应该在ListItem上加key，而不是li

### 4.防注入攻击
React DOM 在渲染所有输入内容之前，默认会进行转义。它可以确保在你的应用中，永远不会注入那些并非自己明确编写的内容。所有的内容在渲染之前都被转换成了字符串。这样可以有效地防止 XSS（cross-site-scripting, 跨站脚本）攻击。

### 5.将一个元素渲染为dom
```
<div id="root"></div>
```
成为‘根’dom节点，仅使用react创建的应用通常只有一个根dom节点。  
想要将一个 React 元素渲染到根 DOM 节点中，只需把它们一起传入 ReactDOM.render()
```
const element = <h1>Hello, world</h1>;
ReactDOM.render(element, document.getElementById('root'));
```

### 6.按需更新
React DOM 会将元素和它的子元素与它们之前的状态进行比较，并只会进行必要的更新来使 DOM 达到预期的状态。  
尽管每一秒我们都会新建一个描述整个 UI 树的元素，React DOM 只会更新实际改变了的内容 
根据我们的经验，考虑 UI 在任意给定时刻的状态，而不是随时间变化的过程，能够消灭一整类的 bug。

### 7.组件名称必须以大写字母开头。

React 会将以小写字母开头的组件视为原生 DOM 标签。例如，```<div />```代表 HTML 的 div 标签，而 ```<Welcome />``` 则代表一个组件，并且需在作用域内使用 Welcome。

### 8.生命周期
当 Clock 组件第一次被渲染到 DOM 中的时候，就为其设置一个计时器。这在 React 中被称为“挂载（mount）”。

同时，当 DOM 中 Clock 组件被删除的时候，应该清除计时器。这在 React 中被称为“卸载（unmount）”。

于是我们有了componentDidMount() 和 componentWillUnmount()

当你需要在class中添加不参与数据流的额外字段，而它又需要初始化时，就在componentDidMount()来做。
比如在componentDidMount()中初始化一个计时器，在componentWillUnmount()中销毁它。

关于state:
- 不要直接修改
- state的更新可能是异步的，出于性能考虑，React 可能会把多个 setState() 调用合并成一个调用。
因为 this.props 和 this.state 可能会异步更新，所以你不要依赖他们的值来更新下一个状态。
```
// Wrong
this.setState({
  counter: this.state.counter + this.props.increment,
});
```
要解决这个问题，可以让 setState() 接收一个函数而不是一个对象。这个函数用上一个 state 作为第一个参数，将此次更新被应用时的 props 做为第二个参数：
```
// Correct
this.setState((state, props) => ({
  counter: state.counter + props.increment
}));
```

### 9.事件
在 React 中另一个不同点是你不能通过返回 false 的方式阻止默认行为。你必须显式的使用 preventDefault 。
```
function ActionLink() {
  function handleClick(e) {
    e.preventDefault();
    console.log('The link was clicked.');
  }

  return (
    <a href="#" onClick={handleClick}>
      Click me
    </a>
  );
}
```
在这里，e 是一个合成事件。React 根据 W3C 规范来定义这些合成事件，所以你不需要担心跨浏览器的兼容性问题。如果想了解更多，请查看 [SyntheticEvent](https://zh-hans.reactjs.org/docs/events.html) 参考指南。

### 10.条件渲染
之所以可以用 条件 && 组件 的方式来渲染组件。是因为在 JavaScript 中，true && expression 总是会返回 expression, 而 false && expression 总是会返回 false。

因此，如果条件是 true，&& 右侧的元素就会被渲染，如果是 false，React 会忽略并跳过它。

其次，在组件的 render 方法中返回 null 并不会影响组件的生命周期。即使返回null，componentDidUpdate 依然会被调用。

### 11.表单
React中，可变状态通常保存在组件的state属性中，且只能通过setState来更新。但HTML里类似input,select,textarea等元素本身自己维护着state。

一般情况下，我们还是倾向于让react的state成为组件的唯一数据源，即由react来渲染input，并控制用户输入过程中发生的操作，这样改造了的表单输入元素叫做“受控”组件。
```
...
<form ...>
    <input type="text" value={this.state.value} onChange={this.handleChange} />
</form>
...
```
如上,显示的值将始终为 this.state.value，这使得 React 的 state 成为唯一数据源。由于 handlechange 在每次按键时都会执行并更新 React 的 state，因此显示的值将随着用户输入而更新。

对于受控组件来说，输入的值始终由 React 的 state 驱动。你也可以将 value 传递给其他 UI 元素，或者通过其他事件处理函数重置，但这意味着你需要编写更多的代码。

[这里](https://zh-hans.reactjs.org/docs/forms.html)了解诸如 textarea,select等受控组件在react中的用法。

当需要处理多个 input 元素时，我们可以给每个元素添加 name 属性，并让处理函数根据 event.target.name 的值选择要执行的操作。

其次，类似```<input type="file" /> ```这种value为只读的元素，它们叫做非受控组件。
有时使用受控组件会很麻烦，因为你需要为数据变化的每种方式都编写事件处理函数，并通过一个 React 组件传递所有的输入 state。当你将之前的代码库转换为 React 或将 React 应用程序与非 React 库集成时，这可能会令人厌烦。在这些情况下，你可能希望使用非受控组件, 这是实现输入表单的另一种方式。

当然，直接找一些成熟的解决方案也是非常好的解决方式，比如[Formilk](https://jaredpalmer.com/formik) 就是不错的选择。但它们大多也是建立在受控组件和管理state的基础上的，所以了解一下原理并没有错。

### 12.状态提升
其实关于这个，只要用过React的人就应该非常熟悉，没什么值得再进一步探讨的。这里就提炼一下要点：  
- 在 React 应用中，任何可变数据应当只有一个相对应的唯一“数据源”，如果当前组件的state在其它组件中被需要，我们就应该考虑将这个变量提升到共同的父组件中。利用好自上而下的数据流。
- 如果某些数据可以由 props 或 state 推导得出，那么它就不应该存在于 state 中。
- 利用好[React开发者工具](https://github.com/facebook/react/tree/master/packages/react-devtools)来定位UI中的bug

提升方式与双向绑定的区别与优缺点： react的这种数据单向绑定方式实际是大势所趋，使得我们在追踪数据变化和调试程序的时候逻辑更加清晰，而双向绑定虽然一定程度减少了代码量，但一让查错变得困难，二也会在大量双向绑定存在的时候产生性能问题。

### 13.不需要组件继承
组件可以接受任意 props，包括基本数据类型，React 元素以及函数。

如果你想要在组件间复用非 UI 的功能，我们建议将其提取为一个单独的 JavaScript 模块，如函数、对象或者类。组件可以直接引入（import）而无需通过 extend 继承它们。

### 14.容器组件
有一个特殊的prop：children。使得别的组件可以通过 JSX 嵌套，将任意组件作为子组件传递给它们。
```
<SomeComp color="blue">
      <h1 className="Dialog-title">
        Welcome
      </h1>
      <p className="Dialog-message">
        Thank you for visiting our spacecraft!
      </p>
</SomeComp>
```
出现在标签开始结束标记之间的这些内容，就会在SomeComp内部以props.children的形式存在。
