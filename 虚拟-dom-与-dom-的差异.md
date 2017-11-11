title: React - 虚拟 dom 与 dom 的差异
author: Bartosz Krajka 
date: '2017-04-05 19:09:50'
tags: [ react, virtual dom, difference between virtual dom and dom]
from: http://reactkungfu.com/2015/10/the-difference-between-virtual-dom-and-dom/

---

尝试翻译一篇虚拟 dom 的文章。
<!-- desc -->
> from [http://reactkungfu.com/2015/10/the-difference-between-virtual-dom-and-dom/](http://reactkungfu.com/2015/10/the-difference-between-virtual-dom-and-dom/)

*Virtual Dom* 在 [React 首页](https://facebook.github.io/react/)就立即吸引了我们的眼球，并且这个特性似乎非常重要。

但 “Virtual Dom” 到底指的是什么？

## DOM
直白点说，DOM 指的是 *Document Object Model*，并且是一串有组织的文本的抽象概念。对 web 开发者来说，这串文本即是我们常称的 *HTML DOM*。HTML 元素(*Elements*)会成为 DOM 中的 节点(*nodes*)。

所以说，HTML 是一串文本，DOM 是其在内存中的代表。

> 好比进程是一个程序的实例，一个程序可以拥有多个进程，同样的，一个 HTML 可以拥有多个 DOM（比如一个页面加载在多个选项卡上）

HTML DOM 提供了一个 API 来遍历和修改所有的 nodes，比如 `getElementById` 与 `removeChild`。

一般我们用 JavaScript 来操作 DOM，当我们想动态的改变页面内容，可以直接修改 DOM：

```javascript
var item = document.getElementBId("myLi");
item.parentNode.removeChild(item);
```

## Issues
HTML DOM 必须是 HTML 文本所允许的树状结构。这种结构可以让我们非常方便的遍历节点，但是简单不代表快速。

时至今日，动态页面、单页面应用已经渐趋平常，我们需要频繁与大量的操作 DOM tree，而 其也变得越来越大与繁重。由此带来的性能问题与开发难度让人很头痛。

> 顺便一说，我独自创建了一个网页，其源代码大小达到了 5G+，工作量不是很难。

试想下，个 DOM 由几千个 `div` 组成。我们现在已经有很多种方法来操作例如 clicks, submit, type-in 等等等等的 事件(events)，一个典型的 事件处理程序，比如 jQuery 一般具备：

* 通过一个 event 找到任意的 node
* 可以更新刚才找到的节点

但是如此的话会产生两个问题：

1. 很难管理。
2. 效率很低。

此时，React 又像开发者伸出了援手：声明。由开发者直接定义一个组件，React 负责渲染，所有的 HTML DOM API 皆由 React 代劳。组件化会很容易管理。

但性能问题仍为解决，此时才是 *Virtual DOM* 登场的时候。

## Virtual DOM
首先来讲，*Virtual DOM* 并不是有 React 发明的，React 使用了它，并且免费向开发者提供。

*Virtual DOM* 是 HTML DOM 的抽象概念，其与浏览器指定的细节相分离，并且更加轻巧。实际上，由于 DOM 本身就是一个抽象概念，*Virtual DOM* 是一个抽象概念的抽象概念。

![](//blog.azlar.cc/images/translation/virtual_dom/meme.jpg)

或许将 *Virtual DOM* 理解为 React 的本地简化版 HTML DOM 的 copy。React 可以使用 *Virtual DOM* 来计算，以便跳过那些通常来说较为冗长且必须为浏览器指定的“真实”的 DOM 操作。

*Virtual DOM* 与真实的 DOM 并没有太大差别，所以 React 代码中的 JSX 部分看起来基本和 HTML 一样：

```jsx
class CommentBox extends React.Component({
  render() {
    return (
      <div className="commentBox">
        Hello, world! I am a CommentBox.
      </div>
    );
  }
});
```

大多数情况，当你想将 HTML 创建为一个 React 组件时，你只需做以下两步：

1. 将 HTML 代码写在 `render` 方法中的 `return` 里
2. 将 `class` 属性替换为 `className`。（`class` 是 JavaScript 的预保留词）

两个 DOM 相较，还有许多细小的的差异：

* 三个在 "真实" DOM 中：`key`, `ref`, `dangerouslySetInnerHTML `。[See More](https://facebook.github.io/react/docs/special-non-dom-attributes.html)
* Virtual Dom 有[一些约束写法](https://facebook.github.io/react/docs/dom-differences.html)。（PS：元素的属性的不同写法等）

## ReactElement vs ReactComponent
在聊到 Virtual DOM 时，我们需要先了解这两者的差异。

### ReactElement
这是 React 的一个主要形态。[React docs](https://facebook.github.io/react/docs/glossary.html#react-elements) 内有提到：

> A `ReactElement` is a light, stateless, immutable, virtual representation of a DOM Element.
> 
> -
> 一个 `ReactElement` 是一个 轻巧、无状态、不可变的，且是一个 DOM 元素的虚拟表现。


**ReactElements 只存在于 Virtual Dom 中。**基础节点由其创建。它们的不可变性使得它们可以快速且简单的被比较与更新。这也是 React 高性能的原因。

只要你想，基本上所有的 HTML tag，比如 `div`、`table` 等等都可以成为 `ReactElement`。点击查看 [所有元素列表](https://facebook.github.io/react/docs/tags-and-attributes.html)

一旦被定义，`ReactElements`就可以被渲染到 “真实” DOM 中。这也是 React 停止控制这些元素的时候了（`PS：render` 里面就不能再对 `state` 做修改了，因为调用 `setState` 后，会触发 `re-render`，形成死循环）。它们会变成又慢又无聊的 DOM 节点了：

```javascript
var root = React.createElement('div');
ReactDOM.render(root, document.getElementById('example'));
// If you are surprised by the fact that `render` 
// comes from `ReactDOM` package, see the Post Scriptum.
```

JSX 可以编译 HTML tags 为 `ReactElements`。所以下边这行代码也可以工作：

```jsx
var root = <div />;
ReactDOM.render(root, document.getElementById('example'));
```

再强调一遍：**ReactElements 是 React 语言中 Virtual DOM 的基础元素（PS：basic items 不知道该咋翻译。。）。**但由于它们的无状态性，对程序员帮助貌似很小，似乎我们更想要的是在层级分明的 HTML 结构中使用各种变量与常量来工作。

所以让我们再来看 ReactComponent。（PS：And here we come to…）

### ReactComponent
`ReactComponent` 与 `ReactElement` 的区别在于：*ReactComponents* 是有状态。

我们一般使用 `React.createClass` 方法来定义一个 `ReactComponent`：

```jsx
var CommentBox = React.createClass({
  render: function() {
    return (
      <div className="commentBox">
        Hello, world! I am a CommentBox.
      </div>
    );
  }
});
```
PS：在 ES6 中我们一般使用：

```jsx
class CommentBox extends React.Component {
	render() { ... }
}
```

那些由 `render()` 中 `return` 的 看起来像 HTML 的代码块可以拥有状态(state)。而且最酷的地方在于：无论何时 state 被改变，组件(component) 就会被重新渲染(rerender)：

```jsx
var Timer = React.createClass({
  getInitialState: function() {
    return {secondsElapsed: 0};
  },
  tick: function() {
    this.setState({secondsElapsed: this.state.secondsElapsed + 1});
  },
  componentDidMount: function() {
    this.interval = setInterval(this.tick, 1000);
  },
  componentWillUnmount: function() {
    clearInterval(this.interval);
  },
  render: function() {
    return (
      <div>Seconds Elapsed: {this.state.secondsElapsed}</div>
    );
  }
});
```

PS: ES6

```jsx
import React from "react";

class Timer extends React.Component {
	constructor（props）{
		super(props);
		
		this.state = {
			secondsElapsed: 0
		};
	}
	
	tick() {
		this.setState({
			secondsElapsed: this.state. secondsElapsed + 1
		});
	}
	
	componentDidMount() {
		this.interval = setInterval(this.tick, 1000);
	}
	
	componentWillUnmount() {
		clearInterval(this.interval);
	}
	
	render() {
		return (
			<div>
				Seconds Elapsed: {this.state.secondsElapsed}
			</div>
		);
	}
}
```

在设计一个动态 HTML 领域，*ReactComponents* 已被证实为一个极好用的工具。它们没有权限访问 *Virtual DOM*，但它们可以简单的被转换为 *ReactElements*：

```jsx
var element = React.createElement(MyComponent);
// or equivalently, with JSX
var element = <MyComponent />;
```

## 到底哪里发生了改变（PS：What makes the difference）
*ReactComponents* 非常好，由于它们易于管理，我们也愿意创建许多的 *Component*。但是他们并没有我们所期望的功能：访问 *Virtual DOM* 的权限。

当一个 `ReactComponent` 在改变 state，我们希望将对 “真实” DOM 的改变（操作）降到最少、最小。
这也正是 React 所处理的问题。`ReactComponent` 被转化为 `ReactElement`，之后 `ReactElement` 在快速与简单的更新后可以被插入到 *Virtual DOM*。

其中是如何工作的？好吧，这是 `diff algorithm`（diff 算法） 的工作。关键点在于：**这种模式比 “寻常” DOM 快很多。**

**当 React 感知到 diff 时，它将会被转化为最底层的（HTML DOM）代码，并与 DOM 中运行、生效**。生成的代码，是基于每个浏览器优化过的。

## 总结
*Virtual DOM* 真的是一个值得被挂在首页的特性吗？**我想说，的确如此。**实践中，React 的性能非常好，这绝对是 *Virtual DOM* 的功劳。


## 我也说点啥~
我对 react 目前来说比较熟悉，但是原理性的东西可能不会描述；这篇文章的时间比较早，概念也比较基础，所以拿来翻译顺便理解下概念。

- ReactElement 即是 React 中的 html 元素

也推荐下官网的这篇：[**React Components, Elements, and Instances**](https://facebook.github.io/react/blog/2015/12/18/react-components-elements-and-instances.html)
