title: React 将组件渲染至顶层组件之外
author: azlar
date: '2017-11-24 15:02:00'
tags: [react, component render outside]

---

将 React 组件渲染至主组件之外。
<!-- desc -->

## 问题产生
写一些浮层的时候，总是因为 `z-index` 与 `overflow: hidden;` 的问题导致无法完美的控制组件。

例如顶层组件中有一个层级(*zIndex*)较高的 absolute 元素，这样可能导致我们的其他浮层可能会被隐藏；又或者我们想让一个 absolute 元素不被父级的 `overflow: hidden;` 而隐藏。

旧的解决方案一般是将组件（如：Modal）append 到 body 底部，这样可以保证其的层级位置。

### 尝试方案
#### 0. Modal
```js
/**
 * @file Modal
 * @author azlar
 * @date 24/11/2017
 */
 
import React from 'react';
const style = {
    width: '100vw',
    height: '100vh',
    backgroundColor: 'rgba(0,0,0, .7)',
    display: 'flex',
    justifyContent: 'center',
    alignItems: 'center',
    position: 'absolute',
    left: 0,
    top: 0,
    color: 'white',
    fontSize: '300%',
    zIndex: 100
};

class Modal extends React.Component {
    render() {
        let { style, content, clickEvent } = this.props;
        style = style || {};
        content = content || '';

        return (
            <div
                style={ style }
                onClick={ clickEvent }
            >
                { content }
            </div>
        )
    }
}

export default Modal;
```
#### 1. 移动 dom 到 body
##### code
```js
import React from 'react';
import { findDOMNode } from 'react-dom';

class Modal1 extends React.Component {
    constructor(props) {
        super(props);

        this.modalDom = null;
        this.dom = null;
    }

    componentDidMount() {
        this.dom = findDOMNode(this);
        this.modalDom = this.dom.childNodes[0];

        console.log(this.modalDom);

        document.body.appendChild(this.modalDom);
    }

    componentWillUnmount() {
        this.dom.appendChild(this.modalDom);    // trick
    }

    render() {
        return (
            <div>
                <Modal { ...this.props } />
            </div>
        );
    }
}
```

##### 分析
此方法其实是个小 trick，我们绕过 Virtual DOM 去修改了 dom，这里很容易(卸载时)出错。
且事件冒泡仍会影响到 App 组件（姑且算预期）。

#### 2. 使用 ReactDOM.render()
##### code
```js
import React from 'react';
import { findDOMNode } from 'react-dom';

class Modal2 extends React.Component {
    constructor(props) {
        super(props);

        this.state = {
            loading: true
        };
        this.dom = null;

    }

    componentDidMount() {
        let div = document.createElement('div');
        document.body.appendChild(div);

        this.dom = div;

        this.setState({
            loading: false
        });
    }

    componentWillUnmount() {
        unmountComponentAtNode(this.dom);
        document.body.removeChild(this.dom);
    }

    render() {
        return this.state.loading ?
            <h1>Loading...</h1>
            :
            render(<Modal { ...this.props } />, this.dom);
    }
}
```
##### 分析
这个方法算是一个不错的解决方案，相当声明了一个新的 ReactDOM，调试也相对友好很多，很多库都使用的这种方案（如：anti-design 中的各种浮层）。

不会产生事件冒泡（与原顶层组件），此时的 modal 已经脱离原有的组件结构，但仍可以通过回调完成交互。

#### 3. 使用 ReactDOM.createPortal()
##### code
```js
import React from 'react';
import { findDOMNode } from 'react-dom';

class Modal3 extends React.Component {
    render() {
        return createPortal(<Modal { ...this.props } />, document.body);
    }
}
```
##### 分析
React 16 新增的方法，专门为了解决此而生，事件冒泡仍会继续发生；可以理解为想法一的官方版。


## demo
repo: [https://github.com/azlarsin/react-render-outside-demo](https://github.com/azlarsin/react-render-outside-demo)

online demo: [https://blog.azlar.cc/demos/react-render-outside-demo/](https://blog.azlar.cc/demos/react-render-outside-demo/)

