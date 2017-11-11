title: React 小记
author: azlar
date: '2017-03-10 14:05:36'
tags: [react, react router]

---

记录一些 React 有关的小问题。
<!-- desc -->

## props can't update child Component
`React` 中每一个 *Loop* 都需要一个唯一的 key，用以区分每一条记录，如：

```jsx

{
	<ul>
		this.state.list.map((li, index) => {
			<ListChildComponent key={ "key-" + index } { ...li }>
			</ListChildComponent>
		})
	</ul
}
```

### 问题
有些时候，移动了 `ListChildComponent` 的顺序，如用户手动拖拽 `ListChildComponent` 的操作，会重置 `state.list` 的顺序，而上图中的写法，会导致渲染的属性并没有改变。

### 解决
一直以为是由于子组建没有渲染的缘故，后来发现是由于 key 的设置有问题，key 需要确保唯一性，而上例中的 key 在移动顺序后，会与之前的 key 相冲突，如

```jsx
<ListChildComponent key="key-0" />
<ListChildComponent key="key-1" />
<ListChildComponent key="key-2" />
<ListChildComponent key="key-3" />
<ListChildComponent key="key-4" />
<ListChildComponent key="key-5" />
```

中，当 `key-5` 移动到 `key-0` 与 `key-1` 之间时候，其新 key 为 `key-1`，与原有的 key 冲突。

故只需要给其一个唯一的 key 即可，入 { "key-" + id }。


## 子组建触发 constructor 方法
子组建有时候需要重新刷新 state，比如一个公用的状态组件，会跟随其传入的元素不同，而显示不同的数据，如：

```jsx
<ParentComponent >
    <ChildComponent data={ data } />
</ParentComponent>
```

### 问题
必须使用 `componentWillReceiveProps(props)` 来作为判断依据而重新刷新 state：

```jsx
class ChildComponent extends React.Component {
    constructor(props) {
        super(props);
            
        this.state = {
            id: data.id,
            element: data.element
        }
    }

    componentWillReceiveProps(props) {
        if(this.state.id != props.id) {
            this.setState({
                id: data.id,
                element: data.element
            })
        }
    }

    //...render
}
```

### 解决
为子组建设置唯一 key，这样 React 会将其当做两个不同的组件而重新渲染：

```jsx
//parent-render
<ParentComponent >
    <ChildComponent data={ data } key={ "child-" + data.id }  />
</ParentComponent>

//child
class ChildComponent extends React.Component {
    constructor(props) {
        super(props);
            
        this.state = {
            id: data.id,
            element: data.element
        }
    }
	/*
    componentWillReceiveProps(props) {
        if(this.state.id != props.id) {
            this.setState({
                id: data.id,
                element: data.element
            })
        }
    }
    */

    //...render
}
```

### 另
最好不要给子组建定义 state，能直接使用 props，坚决不定义 state。