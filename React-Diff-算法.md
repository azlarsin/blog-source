title: React - Diff 算法
author: Christopher Chedeau
date: '2017-04-10 19:01:23'
tags: [react, diff algorithm, diff 算法, virtual dom]

---

一直想找时间好好学习下，顺便又强行翻一篇。

<!-- desc -->
> from [https://calendar.perfplanet.com/2013/diff/](https://calendar.perfplanet.com/2013/diff/) 2013.12.28

[React](http://facebook.github.io/react/) 是由 facebook 开发来创建用户界面的 Javascript 库。性能，一直是从头到尾开发其时考虑的重点。在这篇文章中，我会展现 React 的 diff 算法与渲染工作，之后你可以来优化你自己的 apps。

## Diff Algotithm
在我们了解实现细节之前，让我们先总体了解下 React 是怎样工作的，这点很重要。（PS：Before we go into the implementation details it is important to get an overview of how React works.）

```jsx
var MyComponent = React.createClass(
{ 
	render: function() { 
		if (this.props.first) { 
			return (
				<div className="first">
					<span>A Span</span>
				</div>
			); 
		} else { 
			return (
				<div className="second">
					<p>A Paragraph</p>
				</div>
			); 
		} 
	} 
});
```

在任何时候，都是由你自己定义你的 UI 的样子。render 的结果并不是一个实际的 DOM node，它们只是轻巧的 JavaScript objects。我们称其为虚拟节点（*Virtual Dom*）。

*React* 将使用 *Virtual Dom* 去尝试找到从前一次渲染到下次渲染之间的最小变化。例如，当我们挂载了 `<MyComponent first={true} />` 后，使用 `<MyComponent first={false} />` 替换它，然后卸载(unmount)它，他们产生的 DOM 指令：

> None to first
> 
> - Create node: `<div className="first"><span>A Span</span></div>`
> 
> First to second
> 
> - Replace attribute: `className="first"` by `className="second"`
> - Replace node: `<span>A Span</span>` by `<p>A Paragraph</p>`
> 
> Second to none
> 
> - Remove node: `<div className="second"><p>A Paragraph</p></div>`


#### 逐层次的(PS：Level by Level)
找到两个任意树结构的最小变化次数，是一个 O( n^3 ) 的问题。就如你能够猜测出的，这在我们的使用案例中并不易于掌控。*React* 使用简单而又强大的启发式方法找到了一个逼近 O(n) 的算法。(PS：React uses simple and yet powerful heuristics to find a very good approximation in O(n).)

*React* 只尝试逐层次的使 树 一致，这样彻底的减少了复杂度，并且由于在 web 应用中，很少会将一个 component 移动到树的其他层级，所以（性能）损失并不大。（PS：This drastically reduces the complexity and isn’t a big loss as it is very rare in web applications to have a component being moved to a different level in the tree.）一般他们只会在 children 中横向移动。

![](//blog.azlar.cc/images/translation/react_diff/react_tree_move.png)


#### List
假定我们现在有一个组件，通过一次循环渲染了 5 个该组件，现在插入一个该组件到这个 list 的中间。在现有的信息下，如何去实现两个 list 之间组件的映射（关系）会变得非常困难。


默认地，*React* 将前后两个 list 的第一个组件相关联起来，etc（PS：`By default, React associates the first component of the previous list with the first component of the next list, etc`，这个 etc 不知道该咋翻译）。你可以提供一个属性 `key`，来帮助 *React* 计算出映射关系。事实上，在 children 中找到一个唯一的 key 通常是比较简单的。

![](//blog.azlar.cc/images/translation/react_diff/list_with_key.png)

#### 组件（Components）
一个 React app 一般是由很多用户自定义的组件组成的，最终转变为一个主要由 div 们组成的树。这个附加的信息会被 diff 算法考虑到，React 只会匹配同 class 下的组件们。

举例来说，如果一个 `<Header>` 被 `<ExampleBlock>` 替换，React 会删除 `header` 后创建一个 `example block`。我们不需要将事件浪费在试图匹配两个不太可能相似的组件上。

![](//blog.azlar.cc/images/translation/react_diff/diff_component.png)

## 事件委托（Event Delegation）
绑定事件监听器（event listener）到 DOM 节点上是慢到发指且极耗内存的。作为替代的，React 实现了一项流行的技术：”事件委托“。React 走的更远，并且重新实现了一个遵循 W3C 的事件系统。这代表 IE8 中事件控制器的 bug 成为了历史，所有的事件名在跨浏览器上保持一致。

让我来解释一下它是怎么实现的。一个单独的事件监听器是绑定在 document 的根部的。当一个 event 被触发，浏览器告诉我们目标 DOM 节点。为了能分层级的在 DOM 中传播事件，React 不在 虚拟 DOM 中做迭代（PS：iterate）。

反之我们利用了每个 React 组件都有一个对层级编码的唯一 id 的事实。我们可以使用简单的字符串操作去获取所有父级的 id。我们发现，将所有的事件监听器存储到一张哈希表，比直接将它们绑定到虚拟 DOM 上效率更高。

这里有个展示事件被绑定到虚拟 DOM 的过程中发生了什么的例子：

```javascript
// dispatchEvent('click', 'a.b.c', event) 
clickCaptureListeners['a'](event); 
clickCaptureListeners['a.b'](event); 
clickCaptureListeners['a.b.c'](event); 
clickBubbleListeners['a.b.c'](event); 
clickBubbleListeners['a.b'](event); 
clickBubbleListeners['a'](event);
```

## 渲染
#### 批量处理
无论何时当你在一个组件调用 `setState` 的时候，React 会标记它为被污染的。在事件循环的结尾，React 会着眼于所有被污染的组件并重新渲染它们。

这个批处理意味着在事件循环中，DOM 只会恰好更新一次（PS：`there is exactly one time when the DOM is being updated`）。这是创建一个高效 app 的关键属性，并且其很难通过常用的 Javascript 实现（PS：`This property is key to building a performant app and yet is extremely difficult to obtain using commonly written JavaScript`）。但默认在 React 中，你直接就可以使用。

![](//blog.azlar.cc/images/translation/react_diff/setState_update.png)

#### 子树的渲染（Sub-tree Rendering）
当 `setState` 被调用时，组件会为其 children 重建虚拟 DOM。如果你在一个跟元素调用 `setState`，整个 React app 将会重新渲染。所有的组件，就算它们并没有改变，它们的 `render` 也将被调用。这点听起来似乎有点恐怖与低效，但实际上，由于我们并没有触碰到真实的 DOM ，所以仍然可以正常工作。

首先，我们在讨论用户的显示界面。由于屏幕空间是有限的，你一般同时在显示成百上千的同类元素（PS：`you’re usually displaying on the orders of hundreds to thousands of elements at a time`）。JavaScript 已经具备足够的速度使整个界面变得可控。

另一个重要的点在于，当在写 React 代码时，你通常不会在每次有变化的时候到 root 节点调用 `setState`。你会在那个收到变化事件的组件或其上的几个组件内调用（`setstate`）。你会极少的直接去往顶部（root）。这意味着用户界面只会有局部的变化。

![](//blog.azlar.cc/images/translation/react_diff/dirty_with_rerendered.png)

#### 有选择的子树渲染（Selective Sub-tree Rendering）
最后，你有可能会阻止某些子树重渲染。如果你在组件内执行了如下方法：

```javascript
boolean shouldComponentUpdate(object nextProps, object nextState)
```

基于组件的前一个与后一个 props/state，你可以告诉 React 这个组件没有改变且不需被重渲染。如果执行的恰到好处，这会为你带来显著的性能提升。

为了能使用它，你必须能比对 JavaScript 对象（objects）。现在因为这个比对应该是深比对还是浅比对，已经产生了很多的问题；若应是深比对，我们是否需要使用 不可边的数据结（mmutable data structures）果或者深层拷贝。

而且你应该记住这个方法会一直被调用，所以你要确保比对耗费的时间少于你探索的时间更少于组件渲染的时间，虽然重渲染并不是严格的需要。（PS：`so you want to make sure that it takes less time to compute than heuristic than the time it would have taken to render the component, even if re-rendering was not strictly needed. `）

![](//blog.azlar.cc/images/translation/react_diff/shouleComponentUpdate.png)


## 结论
让 React 变快的并不是什么新技术：我们很早就已知道访问 DOM 代价很昂贵，你得将读写操作批量处理，事件委派器速度更快一点...

人们仍然谈论它们，是由于在实践中，它们很难在寻常的 JavaScript 代码中实现。React 得以突出的缘故在于所有的优化会默认执行。这使得搬起石头砸自己的脚和让你的 app 变慢变得很难。

React 的性能耗费模型也很容易理解：每个 `setState` 会重渲染整个子树。如果你仍想挤出性能，尽可能少的调用 `setState` 并且使用 `shouleComponentUpdate` 阻止重渲染一个大的子树。