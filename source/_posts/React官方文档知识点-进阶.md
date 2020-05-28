---
title: React官方文档知识点-进阶
date: 2020-05-28 17:56:33
categories:
- web前端
tags:
- React
- 基础
---

承接上一篇文章。

# 正文
### 1.无障碍辅助功能
无障碍辅助功能是让所有人都能够获得服务的一种设计。
关于React中对此功能的全面要点：[点击这里](https://zh-hans.reactjs.org/docs/accessibility.html)

### 2.代码分割
大多数 React 应用都会使用 Webpack，Rollup 或 Browserify 这类的构建工具来打包文件。 打包是一个将文件引入并合并到一个单独文件的过程，最终形成一个 “bundle”。 接着在页面上引入该 bundle，整个应用即可一次性加载。

而现在的前端应用大多都会整合非常多第三方库，为了避免打包出的代码包过大导致加载时间边长。我们应该尽早思考对代码进行代码分割。代码分割是上诉打包工具支持的一项技术，能够创建多个包并在运行时动态加载。  
它可以让你“懒加载”用户所需要的内容，避免加载永远不需要的代码，并在初始加载时减少所需加载的代码量，从而显著提高应用性能。

比如你的网站有20个网页，而某个用户可能只使用其中2个网页，另外18个网页的内容，用户是不需要加载的。

使用方法：
- import
```
//使用前
import { add } from './math';
console.log(add(16, 26));

//使用后
import("./math").then(math => {
  console.log(math.add(16, 26));
});
```
如果你自己配置 Webpack，你可能要阅读下 Webpack 关于[代码分割](https://webpack.docschina.org/guides/code-splitting/)的指南。你的 Webpack 配置应该[类似于此](https://gist.github.com/gaearon/ca6e803f5c604d37468b0091d9959269)。

- React.lazy
React.lazy 函数能让你像渲染常规组件一样处理动态引入（的组件）。
```
//使用前
import OtherComponent from './OtherComponent';

//使用后
const OtherComponent = React.lazy(() => import('./OtherComponent'));
```
此代码将会在组件首次渲染时，自动导入包含 OtherComponent 组件的包。
React.lazy 接受一个函数，这个函数需要动态调用 import()。它必须返回一个 Promise，该 Promise 需要 resolve 一个 default export 的 React 组件。  
然后你应在 **Suspense** 组件中渲染 lazy 组件，如此使得我们可以使用在等待加载 lazy 组件时做优雅降级（如 loading 指示器等）。
```
import React, { Suspense } from 'react';
const OtherComponent = React.lazy(() => import('./OtherComponent'));

function MyComponent() {
  return (
    <div>
      <Suspense fallback={<div>Loading...</div>}>
        <OtherComponent />
      </Suspense>
    </div>
  );
}
```
fallback 属性接受任何在组件加载过程中你想展示的 React 元素。你可以将 Suspense 组件置于懒加载组件之上的任何位置。你甚至可以用一个 Suspense 组件包裹多个懒加载组件。

PS：React.lazy 目前只支持默认导出（default exports）

- 异常捕获边界(Error boundaries)
如果模块加载失败（如网络问题），它会触发一个错误。你可以通过[异常捕获边界（Error boundaries）](https://zh-hans.reactjs.org/docs/error-boundaries.html)技术来处理这些情况，以显示良好的用户体验并管理恢复事宜。

- 基于路由的代码分割
另一个很简单的代码分割方式就是基于路由。
使用React.lazy 和 [React Router](https://react-router.docschina.org/) 这类的第三方库，来配置基于路由的代码分割。

### 3.Context
Context 提供了一个无需为每层组件手动添加 props，就能在组件树间进行数据传递的方法。
使用方式
```
// Context 可以让我们无须明确地传遍每一个组件，就能将值深入传递进组件树。
// 为当前的 theme 创建一个 context（“light”为默认值）。
const ThemeContext = React.createContext('light');
class App extends React.Component {
  render() {
    // 使用一个 Provider 来将当前的 theme 传递给以下的组件树。
    // 无论多深，任何组件都能读取这个值。
    // 在这个例子中，我们将 “dark” 作为当前的值传递下去。
    return (
      <ThemeContext.Provider value="dark">
        <Toolbar />
      </ThemeContext.Provider>
    );
  }
}

// 中间的组件再也不必指明往下传递 theme 了。
function Toolbar() {
  return (
    <div>
      <ThemedButton />
    </div>
  );
}

class ThemedButton extends React.Component {
  // 指定 contextType 读取当前的 theme context。
  // React 会往上找到最近的 theme Provider，然后使用它的值。
  // 在这个例子中，当前的 theme 值为 “dark”。
  static contextType = ThemeContext;
  render() {
    return <Button theme={this.context} />;
  }
}
```
请谨慎使用context，因为这会使得组件的复用性变差。

如果你只是想避免层层传递一些属性，组件组合（component composition）有时候是一个比 context 更好的解决方案。

比如在一个Page组件内，它层层向下传递user和avatarSize属性，从而较深的Avatar组件可以读取到这些属性。如果最后只有Avatar需要这个变量，层层传递就显得比较蠢，此时除了使用context，还可以的一种方法是直接在page中构造好avatar组件，然后传递下去。

这种对组件的控制反转减少了在你的应用中要传递的 props 数量，这在很多场景下会使得你的代码更加干净，使你对根组件有更多的把控。但是，这并不适用于每一个场景：这种将逻辑提升到组件树的更高层次来处理，会使得这些高层组件变得更复杂，并且会强行将低层组件适应这样的形式，这可能不会是你想要的。

[context的使用方法](https://zh-hans.reactjs.org/docs/context.html)

### 4.错误边界
部分 UI 的 JavaScript 错误不应该导致整个应用崩溃，为了解决这个问题，React 16 引入了一个新的概念 —— 错误边界。

错误边界是一种 React 组件，这种组件可以捕获并打印发生在其子组件树任何位置的 JavaScript 错误，并且，它会渲染出备用 UI。错误边界在渲染期间、生命周期方法和整个组件树的构造函数中捕获错误。

错误边界无法捕获以下场景中产生的错误：
- 事件处理,因为事件处理器不会再渲染期间触发，so使用try/catch来捕获。
- 异步代码（例如 setTimeout 或 requestAnimationFrame 回调函数）
- 服务端渲染
- 它自身抛出来的错误（并非它的子组件）

如果一个 class 组件中定义了 static **getDerivedStateFromError()** 或 **componentDidCatch()** 这两个生命周期方法中的任意一个（或两个）时，那么它就变成一个错误边界。
当抛出错误后，请使用 static getDerivedStateFromError() 渲染备用 UI ，使用 componentDidCatch() 打印错误信息。

```
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    // 更新 state 使下一次渲染能够显示降级后的 UI
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    // 你同样可以将错误日志上报给服务器
    logErrorToMyService(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      // 你可以自定义降级后的 UI 并渲染
      return <h1>Something went wrong.</h1>;
    }

    return this.props.children; 
  }
}
```
注意错误边界仅可以捕获其子组件的错误，它无法捕获其自身的错误。如果一个错误边界无法渲染错误信息，则错误会冒泡至最近的上层错误边界，这也类似于 JavaScript 中 catch {} 的工作机制。

错误边界的粒度由你来决定，可以将其包装在最顶层的路由组件并为用户展示一个 “Something went wrong” 的错误信息，就像服务端框架经常处理崩溃一样。你也可以将单独的部件包装在错误边界以保护应用其他部分不崩溃。

PS:自 React 16 起，任何未被错误边界捕获的错误将会导致整个 React 组件树被卸载。

### 5.Refs 和 DOM
Refs 提供了一种方式，允许我们访问 DOM 节点或在 render 方法中创建的 React 元素。  
在某些情况下，你需要在典型数据流之外强制修改子组件。被修改的子组件可能是一个 React 组件的实例，也可能是一个 DOM 元素。  

下面是几个适合使用 refs 的情况：
- 管理焦点，文本选择或媒体播放。
- 触发强制动画。
- 集成第三方 DOM 库。

避免使用 refs 来做任何可以通过声明式实现来完成的事情。举个例子，避免在 Dialog 组件里暴露 open() 和 close() 方法，最好传递 isOpen 属性。

两种使用refs的方式：
1.React.createRef()
```
class CustomTextInput extends React.Component {
  constructor(props) {
    super(props);
    // 创建一个 ref 来存储 textInput 的 DOM 元素
    this.textInput = React.createRef();
    this.focusTextInput = this.focusTextInput.bind(this);
  }

  focusTextInput() {
    // 直接使用原生 API 使 text 输入框获得焦点
    // 注意：我们通过 "current" 来访问 DOM 节点
    this.textInput.current.focus();
  }

  render() {
    // 告诉 React 我们想把 <input> ref 关联到
    // 构造器里创建的 `textInput` 上
    return (
      <div>
        <input
          type="text"
          ref={this.textInput} />
        <input
          type="button"
          value="Focus the text input"
          onClick={this.focusTextInput}
        />
      </div>
    );
  }
}
```
React 会在组件挂载时给 current 属性传入 DOM 元素，并在组件卸载时传入 null 值。ref 会在 componentDidMount 或 componentDidUpdate 生命周期钩子触发前更新。

- 当 ref 属性用于 HTML 元素时，构造函数中使用 React.createRef() 创建的 ref 接收底层 DOM 元素作为其 current 属性。
- 当 ref 属性用于自定义 class 组件时，ref 对象接收组件的挂载实例作为其 current 属性。

2.回调ref
```
class CustomTextInput extends React.Component {
  constructor(props) {
    super(props);

    this.textInput = null;

    this.setTextInputRef = element => {
      this.textInput = element;
    };

    this.focusTextInput = () => {
      // 使用原生 DOM API 使 text 输入框获得焦点
      if (this.textInput) this.textInput.focus();
    };
  }

  componentDidMount() {
    // 组件挂载后，让文本框自动获得焦点
    this.focusTextInput();
  }

  render() {
    // 使用 `ref` 的回调函数将 text 输入框 DOM 节点的引用存储到 React
    // 实例上（比如 this.textInput）
    return (
      <div>
        <input
          type="text"
          ref={this.setTextInputRef}
        />
        <input
          type="button"
          value="Focus the text input"
          onClick={this.focusTextInput}
        />
      </div>
    );
  }
}
```
如果 ref 回调函数是以内联函数的方式定义的，在更新过程中它会被执行两次，第一次传入参数 null，然后第二次会传入参数 DOM 元素。这是因为在每次渲染时会创建一个新的函数实例，所以 React 清空旧的 ref 并且设置新的。通过将 ref 的回调函数定义成 class 的绑定函数的方式可以避免上述问题，但是大多数情况下它是无关紧要的。

注：var func_name = function(){}这种形式定义的函数为内联函数。

在极少数情况下，你可能希望在父组件中引用子节点的 DOM 节点。通常不建议这样做，因为它会打破组件的封装，但它偶尔可用于触发焦点或测量子 DOM 节点的大小或位置。  
这种情况下我们推荐使用 [ref 转发](https://zh-hans.reactjs.org/docs/forwarding-refs.html)。Ref 转发使组件可以像暴露自己的 ref 一样暴露子组件的 ref。关于怎样对父组件暴露子组件的 DOM 节点，在 ref 转发文档中有一个详细的例子。

### 6.Fragments
React 中的一个常见模式是一个组件返回多个元素。Fragments 允许你将子列表分组，而无需向 DOM 添加额外节点。
```
render() {
  return (
    <React.Fragment>
      <ChildA />
      <ChildB />
      <ChildC />
    </React.Fragment>
  );
}
```
还有一种段语法,看起来有点诡异，即空标签：
```
class Columns extends React.Component {
  render() {
    return (
      <>
        <td>Hello</td>
        <td>World</td>
      </>
    );
  }
}
```
使用显式 <React.Fragment> 语法声明的片段可能具有 key。一个使用场景是将一个集合映射到一个 Fragments 数组。
```
function Glossary(props) {
  return (
    <dl>
      {props.items.map(item => (
        // 没有`key`，React 会发出一个关键警告
        <React.Fragment key={item.id}>
          <dt>{item.term}</dt>
          <dd>{item.description}</dd>
        </React.Fragment>
      ))}
    </dl>
  );
}
```
key 是目前唯一可以传递给 Fragment 的属性。

### 7.高阶组件
高阶组件（HOC）是 React 中用于复用组件逻辑的一种高级技巧。HOC 自身不是 React API 的一部分，它是一种基于 React 的组合特性而形成的设计模式。

可以对应高阶函数来理解，高阶函数为函数作为参数传递，函数作为返回值。
而高阶组件则是参数为组件，返回值为新组件的函数。
```
const EnhancedComponent = higherOrderComponent(WrappedComponent);
```
HOC 在 React 的第三方库中很常见，例如 Redux 的 connect 和 Relay 的 createFragmentContainer。

一大作用就是将组件间相同的业务逻辑进行抽象，比如2个组件都依赖于某个外部数据进行渲染，那么获取这个外部数据和监听其改变的方式就可以放到高阶组件中进行：
```
// 此函数接收一个组件...
function withSubscription(WrappedComponent, selectData) {
  // ...并返回另一个组件...
  return class extends React.Component {
    constructor(props) {
      super(props);
      this.handleChange = this.handleChange.bind(this);
      this.state = {
        data: selectData(DataSource, props)
      };
    }

    componentDidMount() {
      // ...负责订阅相关的操作...
      DataSource.addChangeListener(this.handleChange);
    }

    componentWillUnmount() {
      DataSource.removeChangeListener(this.handleChange);
    }

    handleChange() {
      this.setState({
        data: selectData(DataSource, this.props)
      });
    }

    render() {
      // ... 并使用新数据渲染被包装的组件!
      // 请注意，我们可能还会传递其他属性
      return <WrappedComponent data={this.state.data} {...this.props} />;
    }
  };
}
```
HOC 不会修改传入的组件，也不会使用继承来复制其行为。相反，HOC 通过将组件包装在容器组件中来组成新组件。HOC 是纯函数，没有副作用。HOC 不需要关心数据的使用方式或原因，而被包装组件也不需要关心数据是怎么来的。

不要试图在 HOC 中修改组件原型（或以其他方式改变它）。因为这种修改会导致组件行为永远发生变化。

HOC 不应该修改传入组件，而应该使用组合的方式，通过将组件包装在容器组件中实现功能：
```
function logProps(WrappedComponent) {
  return class extends React.Component {
    componentDidUpdate(prevProps) {
      console.log('Current props: ', this.props);
      console.log('Previous props: ', prevProps);
    }
    render() {
      // 将 input 组件包装在容器中，而不对其进行修改。Good!
      return <WrappedComponent {...this.props} />;
    }
  }
}
```

您可能已经注意到 HOC 与容器组件模式之间有相似之处。容器组件担任分离将高层和低层关注的责任，由容器管理订阅和状态，并将 prop 传递给处理渲染 UI。HOC 使用容器作为其实现的一部分，你可以将 HOC 视为参数化容器组件。

来看看redux的connect
```
// React Redux 的 `connect` 函数
const ConnectedComment = connect(commentSelector, commentActions)(CommentList);
```
发生了什么？！如果你把它分开，就会更容易看出发生了什么。
```
// connect 是一个函数，它的返回值为另外一个函数。
const enhance = connect(commentListSelector, commentListActions);
// 返回值为 HOC，它会返回已经连接 Redux store 的组件
const ConnectedComment = enhance(CommentList);
```
换句话说，connect 是一个返回高阶组件的高阶函数！

这种形式可能看起来令人困惑或不必要，但它有一个有用的属性, 充当装饰器，多个装饰器可以一起使用。
```
// 而不是这样...
const EnhancedComponent = withRouter(connect(commentSelector)(WrappedComponent))

// ... 你可以编写组合工具函数
// compose(f, g, h) 等同于 (...args) => f(g(h(...args)))
const enhance = compose(
  // 这些都是单参数的 HOC
  withRouter,
  connect(commentSelector)
)
const EnhancedComponent = enhance(WrappedComponent)
```
许多第三方库都提供了 compose 工具函数，包括 lodash （比如 lodash.flowRight）， Redux 和 Ramda。

注意点：
- 不要在 render 方法中使用 HOC
- 务必复制静态方法
- Refs 不会被传递

### 8.与其它视图库集成
得益于 ReactDOM.render() 的灵活性 React 可以被嵌入到其他的应用中。

虽然 React 通常被用来在启动的时候加载一个单独的根 React 组件到 DOM 上，ReactDOM.render() 同样可以在 UI 的独立部分上多次调用，这些部分可以小到一个按钮，也可以大到一个应用。
如：
```
$('#container').html('<button id="btn">Say Hello</button>');
$('#btn').click(function() {
  alert('Hello!');
});
```
改造成：
```
function Button(props) {
  return <button onClick={props.onClick}>Say Hello</button>;
}

function HelloButton() {
  function handleClick() {
    alert('Hello!');
  }
  return <Button onClick={handleClick} />;
}

ReactDOM.render(
  <HelloButton />,
  document.getElementById('container')
);
```

### 9.深入JSX
实际上，JSX 仅仅只是 React.createElement(component, props, ...children) 函数的语法糖。
值得说的点：
- Props 默认值为 “True”，如果你没给 prop 赋值，它的默认值是 true。以下两个 JSX 表达式是等价的：
```
<MyTextBox autocomplete />

<MyTextBox autocomplete={true} />
```
- 属性展开：如果你已经有了一个 props 对象，你可以使用展开运算符 ... 来在 JSX 中传递整个 props 对象。还可以选择只保留当前组件需要接收的 props，并使用展开运算符将其他 props 传递下去。
```
const Button = props => {
  const { kind, ...other } = props;
  const className = kind === "primary" ? "PrimaryButton" : "SecondaryButton";
  return <button className={className} {...other} />;
};
```
- 函数作为子元素。 你可以将任何东西作为子元素传递给自定义组件，只要确保在该组件渲染之前能够被转换成 React 理解的对象。这种用法并不常见，但可以用于扩展 JSX。

- false, null, undefined, and true 是合法的子元素。但它们并不会被渲染。值得注意的是有一些 “falsy” 值，如数字 0，仍然会被 React 渲染。 所以不要用 aa.length && Component 这种形式。改成aa.length>0即可。

### 10.性能优化
UI 更新需要昂贵的 DOM 操作，而 React 内部使用几种巧妙的技术以便最小化 DOM 操作次数。

**首先，使用生产版本。**
React 默认包含了许多有用的警告信息。这些警告信息在开发过程中非常有帮助。然而这使得 React 变得更大且更慢，所以你需要确保部署时使用了生产版本。可通过chrome的[React开发者工具来检查](https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi)。图标为蓝色表示生产版本，图片红色表示开发模式。
[这里](https://zh-hans.reactjs.org/docs/optimizing-performance.html)有webpack,Brunch,Browserify,Rollup来进行生产构建的方法。

**使用chrome performance标签分析组件**
在开发模式下，你可以通过支持的浏览器可视化地了解组件是如何 挂载、更新以及卸载的。  
在 Chrome 中进行如下操作：
1. 临时禁用所有的 Chrome 扩展，尤其是 React 开发者工具。他们会严重干扰度量结果！
2. 确保你是在 React 的开发模式下运行应用。
3. 打开 Chrome 开发者工具的 Performance 标签并按下 Record。
4. 对你想分析的行为进行复现。尽量在 20 秒内完成以避免 Chrome 卡住。
5. 停止记录。
6. 在 User Timing 标签下会显示 React 归类好的事件。
这能帮助你查看是否有不相关的组件被错误地更新，以及 UI 更新的深度和频率。

**使用开发者工具中的分析器对组件进行分析**
在开发模式下，React 开发者工具会出现分析器标签。 你可以在[《介绍 React 分析器》](https://zh-hans.reactjs.org/blog/2018/09/10/introducing-the-react-profiler.html)这篇博客中了解概述。 你也可以在 [YouTube](https://www.youtube.com/watch?v=nySib7ipZdk) 上观看分析器的视频指导。

**虚拟化长列表**
如果你的应用渲染了长列表（上百甚至上千的数据），我们推荐使用“虚拟滚动”技术。这项技术会在有限的时间内仅渲染有限的内容，并奇迹般地降低重新渲染组件消耗的时间，以及创建 DOM 节点的数量。

[react-window](https://react-window.now.sh/) 和 [react-virtualized](https://bvaughn.github.io/react-virtualized/) 是热门的虚拟滚动库。 它们提供了多种可复用的组件，用于展示列表、网格和表格数据。 如果你想要一些针对你的应用做定制优化，你也可以创建你自己的虚拟滚动组件，就像 Twitter 所做的。

**避免调停**
React 构建并维护了一套内部的 UI 渲染描述。它包含了来自你的组件返回的 React 元素。即虚拟DOM，使得 React 避免创建 DOM 节点以及没有必要的节点访问，因为 DOM 操作相对于 JavaScript 对象操作更慢。

当一个组件的 props 或 state 变更，React 会将最新返回的元素与之前渲染的元素进行对比，以此决定是否有必要更新真实的 DOM。当它们不相同时，React 会更新该 DOM。  
即使 React 只更新改变了的 DOM 节点，重新渲染仍然花费了一些时间。在大部分情况下它并不是问题，不过如果它已经慢到让人注意了，你可以通过覆盖生命周期方法 shouldComponentUpdate 来进行提速。该方法会在重新渲染前被触发。其默认实现总是返回 true，让 React 执行更新。  
如果你知道在什么情况下你的组件不需要更新，你可以在 shouldComponentUpdate 中返回 false 来跳过整个渲染过程。其包括该组件的 render 调用以及之后的操作。  
在大部分情况下，你可以继承 React.PureComponent 以代替手写 shouldComponentUpdate()。它用当前与之前 props 和 state 的浅比较覆写了 shouldComponentUpdate() 的实现。

**不可变数据的力量**
这点之前已经说过，比如对于数组，用concat避免直接在原来的数组上push。 对于对象，用Object.assign避免直接修改对象中的值。

当处理深层嵌套对象时，以 immutable （不可变）的方式更新它们令人费解。如遇到此类问题，请参阅 [Immer](https://github.com/mweststrate/immer) 或 [immutability-helper](https://github.com/kolodny/immutability-helper)。这些库会帮助你编写高可读性的代码，且不会失去 immutability （不可变性）带来的好处。

### 11.Portals
Portal 提供了一种将子节点渲染到存在于父组件以外的 DOM 节点的优秀的方案。

通常来讲，当你从组件的 render 方法返回一个元素时，该元素将被挂载到 DOM 节点中离其最近的父节点。然而，有时候将子元素插入到 DOM 节点中的不同位置也是有好处的。 
一个 portal 的典型用例是当父组件有 overflow: hidden 或 z-index 样式时，但你需要子组件能够在视觉上“跳出”其容器。例如，对话框、悬浮卡以及提示框。
```
render() {
  // React 并*没有*创建一个新的 div。它只是把子元素渲染到 `domNode` 中。
  // `domNode` 是一个可以在任何位置的有效 DOM 节点。
  return ReactDOM.createPortal(
    this.props.children,
    domNode
  );
}
```
PS:当在使用 portal 时, 记住[管理键盘焦点](https://zh-hans.reactjs.org/docs/accessibility.html#programmatically-managing-focus)就变得尤为重要。

对于模态对话框，通过遵循 [WAI-ARIA 模态开发实践](https://www.w3.org/TR/wai-aria-practices-1.1/#dialog_modal)，来确保每个人都能够运用它。

[这里](https://codepen.io/gaearon/pen/yzMaBd)有一个关于用portal创建模态对话框的示例。

一个从 portal 内部触发的事件会一直冒泡至包含 React 树的祖先，即便这些元素并不是 DOM 树 中的祖先。假设存在如下
```
<html>
  <body>
    <div id="app-root"></div>
    <div id="modal-root"></div>
  </body>
</html>
```
在 #app-root 里的 Parent 组件能够捕获到未被捕获的从兄弟节点 #modal-root 冒泡上来的事件。
[codepen实例](https://codepen.io/gaearon/pen/jGBWpE)  
在父组件里捕获一个来自 portal 冒泡上来的事件，使之能够在开发时具有不完全依赖于 portal 的更为灵活的抽象。例如，如果你在渲染一个 <Modal /> 组件，无论其是否采用 portal 实现，父组件都能够捕获其事件。

### 12.Profiler API
Profiler 测量渲染一个 React 应用多久渲染一次以及渲染一次的“代价”。 它的目的是识别出应用中渲染较慢的部分，或是可以使用类似 [memoization 优化](https://zh-hans.reactjs.org/docs/hooks-faq.html#how-to-memoize-calculations)的部分，并从相关优化中获益。
[这里](https://zh-hans.reactjs.org/docs/profiler.html)了解详情。

### 13.React Diffing的原理 协调算法
在某一时间节点调用 React 的 render() 方法，会创建一棵由 React 元素组成的树。在下一次 state 或 props 更新时，相同的 render() 方法会返回一棵不同的树。React 需要基于这两棵树之间的差别来判断如何有效率的更新 UI 以保证当前 UI 与最新的树保持同步。 
这个算法问题有一些通用的解决方案，即生成将一棵树转换成另一棵树的最小操作数。 然而，即使在最前沿的算法中，该算法的复杂程度为 O(n 3 )，其中 n 是树中元素的数量。  
这无疑有些难以接受，React基于真实使用环境做出2个假设，在其基础上提出了一套O(n)的算法：
1. 两个不同类型的元素会产生出不同的树；
2. 开发者可以通过 key prop 来暗示哪些子元素在不同的渲染下能保持稳定；

**Diffing算法**
当对比两颗树时，React 首先比较两棵树的根节点。不同类型的根节点元素会有不同的形态。

1. 比对不同类型的元素
当根节点为不同类型的元素时，React 会拆卸原有的树并且建立起新的树。
当拆卸一棵树时，对应的 DOM 节点也会被销毁。在根节点以下的组件也会被卸载，它们的状态会被销毁。
2. 比对同一类型的元素
当比对两个相同类型的 React 元素时，React 会保留 DOM 节点，仅比对及更新有改变的属性。比如className变了就只更新className属性，当style属性更新是，也仅更新有所改变的属性，比如color等。
处理完当前节点之后，React 继续对子节点进行递归。
3. 比对同类型的组件元素
当一个组件更新时，组件实例保持不变，这样 state 在跨越不同的渲染时保持一致。React 将更新该组件实例的 props 以跟最新的元素保持一致，并且调用该实例的 componentWillReceiveProps() 和 componentWillUpdate() 方法。
下一步，调用 render() 方法，diff 算法将在之前的结果以及新的结果中进行递归。
4. 对子节点进行递归
在默认条件下，当递归 DOM 节点的子元素时，React 会同时遍历两个子元素的列表；当产生差异时，生成一个 mutation。
在子元素列表末尾新增元素时，更变开销比较小。React 会针对每个子元素 mutate 而不是保持相同的 ```<li>Duke</li>``` 和 ```<li>Villanova</li>``` 子树完成。这种情况下的低效可能会带来性能问题。
所以引进了key属性。

**总之**
React 可以在每个 action 之后对整个应用进行重新渲染，得到的最终结果也会是一样的。在此情境下，重新渲染表示在所有组件内调用 render 方法，这不代表 React 会卸载或装载它们。React 只会基于以上提到的规则来决定如何进行差异的合并。  
由于 React 依赖探索的算法，因此当以下假设没有得到满足，性能会有所损耗。
1. 该算法不会尝试匹配不同组件类型的子树。如果你发现你在两种不同类型的组件中切换，但输出非常相似的内容，建议把它们改成同一类型。在实践中，我们没有遇到这类问题。
2. Key 应该具有稳定，可预测，以及列表内唯一的特质。不稳定的 key（比如通过 Math.random() 生成的）会导致许多组件实例和 DOM 节点被不必要地重新创建，这可能导致性能下降和子组件中的状态丢失。


### 14.render props
术语 “render prop” 是指一种在 React 组件之间使用一个值为函数的 prop 共享代码的简单技术

具有 render prop 的组件接受一个函数，该函数返回一个 React 元素并调用它而不是实现自己的渲染逻辑。
```
<DataProvider render={data => (
  <h1>Hello {data.target}</h1>
)}/>
```
使用 render prop 的库有 React Router、Downshift 以及 Formik。  
更具体地说，render prop 是一个用于告知组件需要渲染什么内容的函数 prop。
```
class Cat extends React.Component {
  render() {
    const mouse = this.props.mouse;
    return (
      <img src="/cat.jpg" style={{ position: 'absolute', left: mouse.x, top: mouse.y }} />
    );
  }
}

class Mouse extends React.Component {
  constructor(props) {
    super(props);
    this.handleMouseMove = this.handleMouseMove.bind(this);
    this.state = { x: 0, y: 0 };
  }

  handleMouseMove(event) {
    this.setState({
      x: event.clientX,
      y: event.clientY
    });
  }

  render() {
    return (
      <div style={{ height: '100vh' }} onMouseMove={this.handleMouseMove}>

        {/*
          Instead of providing a static representation of what <Mouse> renders,
          use the `render` prop to dynamically determine what to render.
        */}
        {this.props.render(this.state)}
      </div>
    );
  }
}

class MouseTracker extends React.Component {
  render() {
    return (
      <div>
        <h1>移动鼠标!</h1>
        <Mouse render={mouse => (
          <Cat mouse={mouse} />
        )}/>
      </div>
    );
  }
}
```
这项技术使我们共享行为非常容易。要获得这个行为，只要渲染一个带有 render prop 的 <Mouse> 组件就能够告诉它当前鼠标坐标 (x, y) 要渲染什么。

关于 render prop 一个有趣的事情是你可以使用带有 render prop 的常规组件来实现大多数高阶组件 (HOC)。 例如，如果你更喜欢使用 withMouse HOC而不是 <Mouse> 组件，你可以使用带有 render prop 的常规 <Mouse> 轻松创建一个
```
// 如果你出于某种原因真的想要 HOC，那么你可以轻松实现
// 使用具有 render prop 的普通组件创建一个！
function withMouse(Component) {
  return class extends React.Component {
    render() {
      return (
        <Mouse render={mouse => (
          <Component {...this.props} mouse={mouse} />
        )}/>
      );
    }
  }
}
```

重要的是要记住，render prop 是因为模式才被称为 render prop ，你不一定要用名为 render 的 prop 来使用这种模式。事实上， **任何被用于告知组件需要渲染什么内容的函数 prop 在技术上都可以被称为 “render prop”.**


### 15.静态类型检查
像 Flow 和 TypeScript 等这些静态类型检查器，可以在运行前识别某些类型的问题。他们还可以通过增加自动补全等功能来改善开发者的工作流程。出于这个原因，我们建议在大型代码库中使用 Flow 或 TypeScript 来代替 PropTypes。

Flow介绍在[这里](https://zh-hans.reactjs.org/docs/static-type-checking.html)

**typescript**
TypeScript 是一种由微软开发的编程语言。它是 JavaScript 的一个类型超集，包含独立的编译器。作为一种类型语言，TypeScript 可以在构建时发现 bug 和错误，这样程序运行时就可以避免此类错误。您可以通过[此文档](https://github.com/Microsoft/TypeScript-React-Starter#typescript-react-starter) 了解更多有关在 React 中使用 TypeScript 的知识。

完成以下步骤，便可开始使用 TypeScript：
- 将 TypeScript 添加到你的项目依赖中。
- 配置 TypeScript 编译选项
- 使用正确的文件扩展名
- 为你使用的库添加定义

如果你使用create-react-app,那么使用```npx create-react-app my-app --template typescript```即可创建一个使用TypeScript的新项目。

关于怎么在现有项目中引入TS，见[这里](https://zh-hans.reactjs.org/docs/static-type-checking.html)



**Kotlin**
Kotlin 是由 JetBrains 开发的一门静态类型语言。其目标平台包括 JVM、Android、LLVM 和 JavaScript。

JetBrains 专门为 React 社区开发和维护了几个工具：React bindings 以及 Create React Kotlin App。后者可以通过 Kotlin 快速编写 React 应用程序，并且不需要构建配置。

### 16.严格模式
StrictMode 是一个用来突出显示应用程序中潜在问题的工具。与 Fragment 一样，StrictMode 不会渲染任何可见的 UI。它为其后代元素触发额外的检查和警告。
```
import React from 'react';

function ExampleApplication() {
  return (
    <div>
      <Header />
      <React.StrictMode>
        <div>
          <ComponentOne />
          <ComponentTwo />
        </div>
      </React.StrictMode>
      <Footer />
    </div>
  );
}
```
StrictMode 目前有助于：
- 识别不安全的生命周期
- 关于使用过时字符串 ref API 的警告
- 关于使用废弃的 findDOMNode 方法的警告
- 检测意外的副作用
- 检测过时的 context API
详情见[这里](https://zh-hans.reactjs.org/docs/strict-mode.html)

### 17.PropTypes 类型检查
第15点中说过，Flow和TypeScript可以对整个应用程序进行类型检查，但如果你没有使用这些扩展。你可以使用内置的PropTypes来进行类型检查。
```
import PropTypes from 'prop-types';

class Greeting extends React.Component {
  render() {
    return (
      <h1>Hello, {this.props.name}</h1>
    );
  }
}

Greeting.propTypes = {
  name: PropTypes.string
};
```
当传入的 prop 值类型不正确时，JavaScript 控制台将会显示警告。出于性能方面的考虑，propTypes 仅在开发模式下进行检查。

类型很多，如下，还可以自定义：
```
import PropTypes from 'prop-types';

MyComponent.propTypes = {
  // 你可以将属性声明为 JS 原生类型，默认情况下
  // 这些属性都是可选的。
  optionalArray: PropTypes.array,
  optionalBool: PropTypes.bool,
  optionalFunc: PropTypes.func,
  optionalNumber: PropTypes.number,
  optionalObject: PropTypes.object,
  optionalString: PropTypes.string,
  optionalSymbol: PropTypes.symbol,

  // 任何可被渲染的元素（包括数字、字符串、元素或数组）
  // (或 Fragment) 也包含这些类型。
  optionalNode: PropTypes.node,

  // 一个 React 元素。
  optionalElement: PropTypes.element,

  // 一个 React 元素类型（即，MyComponent）。
  optionalElementType: PropTypes.elementType,

  // 你也可以声明 prop 为类的实例，这里使用
  // JS 的 instanceof 操作符。
  optionalMessage: PropTypes.instanceOf(Message),

  // 你可以让你的 prop 只能是特定的值，指定它为
  // 枚举类型。
  optionalEnum: PropTypes.oneOf(['News', 'Photos']),

  // 一个对象可以是几种类型中的任意一个类型
  optionalUnion: PropTypes.oneOfType([
    PropTypes.string,
    PropTypes.number,
    PropTypes.instanceOf(Message)
  ]),

  // 可以指定一个数组由某一类型的元素组成
  optionalArrayOf: PropTypes.arrayOf(PropTypes.number),

  // 可以指定一个对象由某一类型的值组成
  optionalObjectOf: PropTypes.objectOf(PropTypes.number),

  // 可以指定一个对象由特定的类型值组成
  optionalObjectWithShape: PropTypes.shape({
    color: PropTypes.string,
    fontSize: PropTypes.number
  }),
  
  // An object with warnings on extra properties
  optionalObjectWithStrictShape: PropTypes.exact({
    name: PropTypes.string,
    quantity: PropTypes.number
  }),   

  // 你可以在任何 PropTypes 属性后面加上 `isRequired` ，确保
  // 这个 prop 没有被提供时，会打印警告信息。
  requiredFunc: PropTypes.func.isRequired,

  // 任意类型的数据
  requiredAny: PropTypes.any.isRequired,

  // 你可以指定一个自定义验证器。它在验证失败时应返回一个 Error 对象。
  // 请不要使用 `console.warn` 或抛出异常，因为这在 `onOfType` 中不会起作用。
  customProp: function(props, propName, componentName) {
    if (!/matchme/.test(props[propName])) {
      return new Error(
        'Invalid prop `' + propName + '` supplied to' +
        ' `' + componentName + '`. Validation failed.'
      );
    }
  },

  // 你也可以提供一个自定义的 `arrayOf` 或 `objectOf` 验证器。
  // 它应该在验证失败时返回一个 Error 对象。
  // 验证器将验证数组或对象中的每个值。验证器的前两个参数
  // 第一个是数组或对象本身
  // 第二个是他们当前的键。
  customArrayProp: PropTypes.arrayOf(function(propValue, key, componentName, location, propFullName) {
    if (!/matchme/.test(propValue[key])) {
      return new Error(
        'Invalid prop `' + propFullName + '` supplied to' +
        ' `' + componentName + '`. Validation failed.'
      );
    }
  })
};
```
**限制单个元素**
你可以通过 PropTypes.element 来确保传递给组件的 children 中只包含一个元素。
```
// children必须只有一个元素，否则控制台会打印警告。
MyComponent.propTypes = {
  children: PropTypes.element.isRequired
};
```
**默认Prop值**
您可以通过配置特定的 defaultProps 属性来定义 props 的默认值：
```
class Greeting extends React.Component {
  render() {
    return (
      <h1>Hello, {this.props.name}</h1>
    );
  }
}

// 指定 props 的默认值：
Greeting.defaultProps = {
  name: 'Stranger'
};

// 不传递name将渲染出 "Hello, Stranger"：
```

### 18.非受控组件
在大多数情况下，我们推荐使用 受控组件 来处理表单数据。在一个受控组件中，表单数据是由 React 组件来管理的。另一种替代方案是使用非受控组件，这时表单数据将交由 DOM 节点来处理。即使用类似ref等功能来从DOM节点获取表单数据。

**默认值**
在 React 渲染生命周期时，表单元素上的 value 将会覆盖 DOM 节点中的值，在非受控组件中，你经常希望 React 能赋予组件一个初始值，但是不去控制后续的更新。 在这种情况下, 你可以指定一个 defaultValue 属性，而不是 value。

**文件输入**
```<input type="file" />``` 始终是一个非受控组件，因为它的值只能由用户设置，而不能通过代码控制。  
使用 File API 与文件进行交互。
```
class FileInput extends React.Component {
  constructor(props) {
    super(props);
    this.handleSubmit = this.handleSubmit.bind(this);
    this.fileInput = React.createRef();
  }
  handleSubmit(event) {
    event.preventDefault();
    alert(
      `Selected file - ${this.fileInput.current.files[0].name}`
    );
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          Upload file:
          <input type="file" ref={this.fileInput} />
        </label>
        <br />
        <button type="submit">Submit</button>
      </form>
    );
  }
}

ReactDOM.render(
  <FileInput />,
  document.getElementById('root')
);
```

### 19. Web Components
React 和 Web Components 为了解决不同的问题而生。Web Components 为可复用组件提供了强大的封装，而 React 则提供了声明式的解决方案，使 DOM 与数据保持同步。两者旨在互补。作为开发人员，可以自由选择在 Web Components 中使用 React，或者在 React 中使用 Web Components，或者两者共存。

大多数开发者在使用 React 时，不使用 Web Components，但可能你会需要使用，尤其是在使用 Web Components 编写的第三方 UI 组件时。

```
class HelloMessage extends React.Component {
  render() {
    return <div>Hello <x-search>{this.props.name}</x-search>!</div>;
  }
}
```
Web Components 通常暴露的是命令式 API。例如，Web Components 的组件 video 可能会公开 play() 和 pause() 方法。要访问 Web Components 的命令式 API，你需要使用 ref 直接与 DOM 节点进行交互。如果你使用的是第三方 Web Components，那么最好的解决方案是编写 React 组件包装该 Web Components。

Web Components 触发的事件可能无法通过 React 渲染树正确的传递。 你需要在 React 组件中手动添加事件处理器来处理这些事件。

常见的误区是要注意在 Web Components 中应该使用 class 而非 className。

**在 Web Components 中使用 React**
```
class XSearch extends HTMLElement {
  connectedCallback() {
    const mountPoint = document.createElement('span');
    this.attachShadow({ mode: 'open' }).appendChild(mountPoint);

    const name = this.getAttribute('name');
    const url = 'https://www.google.com/search?q=' + encodeURIComponent(name);
    ReactDOM.render(<a href={url}>{name}</a>, mountPoint);
  }
}
customElements.define('x-search', XSearch);
```
注意：
如果使用 Babel 来转换 class，此代码将不会起作用。请查阅该 [issue](https://github.com/w3c/webcomponents/issues/587) 了解相关讨论。 在加载 Web Components 前请引入 [custom-elements-es5-adapter](https://github.com/webcomponents/polyfills/tree/master/packages/webcomponentsjs#custom-elements-es5-adapterjs) 来解决该 issue。