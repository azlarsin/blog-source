title: 前端 bug or 解决方案集合
author: azlar
date: '2017-10-14 10:16:24'
tags: [前端, bug, 坑, 解决方案, javascript, node, css]

---

流水账式记录前端遇到的各种问题，包括不限于框架、node、浏览器问题等。
<!-- desc -->
## js

### firefox can't drag issue
这个问题是一个同事在用我写的一个 [小库](https://github.com/azlarsin/draggable-tree) 时发现的，在火狐内不能正常拖拽。（拽不起来）

#### 解决
火狐的拖拽机制需要用户设置 setData 才会生效。

另： 设置　`text/html`　可以防止火狐拖拽结束后打开新窗口。

另2：若需在 IE 下使用，只能设置 `setData('text', something)`。

```js
dom.addEventListener("dragstart", function (e){
	e.dataTransfer.setData('text/html', null);
});
```

#### 参考
> [https://stackoverflow.com/questions/19055264/why-doesnt-html5-drag-and-drop-work-in-firefox](https://stackoverflow.com/questions/19055264/why-doesnt-html5-drag-and-drop-work-in-firefox)
> 
> [https://stackoverflow.com/questions/12803235/drag-and-drop-not-working-in-ie-javascript-html5](https://stackoverflow.com/questions/12803235/drag-and-drop-not-working-in-ie-javascript-html5)


### 编写 npm 包时本地调试
#### npm link
```bash
cd ~/projects/node-redis    # go into the package directory
npm link                    # creates global link
cd ~/projects/node-bloggy   # go into some other package directory.
npm link redis              # link-install the package
```

or

```bash
cd ~/projects/node-bloggy  # go into the dir of your main project
npm link ../node-redis     # link the dir of your dependency

# The second line is the equivalent of doing:
# (cd ../node-redis; npm link)
# npm link node-redis

```

#### npm unlink
```bash
npm unlink redis
npm install
```

or

```bash
sudo npm rm --global foo
npm ls --global foo
```

#### 参考
> [https://docs.npmjs.com/cli/link](https://docs.npmjs.com/cli/link)
> 
> [https://stackoverflow.com/questions/19094630/how-do-i-uninstall-a-package-installed-using-npm-link](https://stackoverflow.com/questions/19094630/how-do-i-uninstall-a-package-installed-using-npm-link)


### webpack 打包 vendor.js 的 hash 值每次都会变化
```js
{
	output: {
		path: path.resolve(__dirname, "build", "assets"),
		// filename: 'main.[chunkhash:6].js'
		filename: "[name].[chunkhash:6].js"
	},
 
	plugins: [
		new webpack.optimize.CommonsChunkPlugin({
		    name: [ "vendor", "manifest" ],
		    path: path.resolve(__dirname, "build", "assets"),
		    // filename: "vendor.[chunkhash:6].js",
		    // minChunks: Infinity,
		}),   
	]
}
```

#### 参考
> [https://github.com/webpack/webpack/issues/1315](https://github.com/webpack/webpack/issues/1315)
> 
> CommonsChunkPlugin marks [first chunk](https://github.com/webpack/webpack/blob/master/lib/optimize/CommonsChunkPlugin.js#L47) as [Entry-chunk](https://webpack.github.io/docs/code-splitting.html#entry-chunk), which 'contains the runtime plus a bunch of modules'.
> 
> You can solve it by creating separate 'entry chunk':
> 
> plugins: [
>		
>	new webpack.optimize.CommonsChunkPlugin({name: 'vendor'}),
> 
> 	new webpack.optimize.CommonsChunkPlugin({name: 'meta', chunks: ['vendor']})
> 
> ]
> 
> It marks meta.js as entry chunk instead vendors.js.

> As result you'll get three files:

> - main.[hash from app content].js
> 
> - vendor.[hash from vendors content].js
> 
> - meta.[hash].js – webpack runtime, which hash depends on an app and vendors file names


### css child's height expands parent with transition

```html
<style>
.child {
	height: 0;
	transition: height .3s;	
}

.child:hover {
	height: auto;
}

</style>

<div class="wrapper">
	<div class="child">
		contents..
	</div>
</div>
```

以上代码，当子元素的高度设置为 `auto` 的时候，父级 wrapper 不会产生渐变效果；如需父级有被撑开的效果，需要为 child 的渐变状态设置一个明确的高度（如：100px）。


### firefox padding-top 
画正方形的时候，我们常用的方式是以一个伪元素的 padding-top 撑开元素的 height，以使其满足各种场景（resize 交互或响应式），
而 firefox 的 padding-top 计算的标准是基于元素的高度的。


### hot update 不更新页面
#### 前提
先参考 [https://gaearon.github.io/react-hot-loader/getstarted/](https://gaearon.github.io/react-hot-loader/getstarted/) 配置完毕。

能在页面 console 接收到如下信息：

```text
[HMR] Checking for updates on the server...
[HMR] Updated modules:
[HMR]  - 348
[HMR]  - 120
[HMR] Consider using the NamedModulesPlugin for module names.
[HMR] App is up to date.
```

`<head>` 标签内页会加载相应 `<script>` 但是页面始终不更新。。

#### 解决
##### 方案
```javascript
if (module.hot) {
  module.hot.accept('./containers/Root', () => render(Root));
}
```

to

```javascript
if (module.hot) {
  module.hot.accept();
}
```

##### 参考
> [https://github.com/gaearon/react-hot-loader/issues/581](https://github.com/gaearon/react-hot-loader/issues/581)
> 
> ```
> @egorovli I spent hours to trying to get my rhl 3 update to correctly apply edits, then your solution with module.hot.accept() worked. The recommended way, with module.hot.accept(path, callback) failed silently - with "Component does not know how to update itself". I think there must be something about the relative paths or anything in my typescript setup that fails it.

> Is there any documentation/links on what this does, exactly - since we don't get the chance to explicitly call the render function again? Apart from the very annoying store error, are there any other disadvantages to using this simple syntax?

> UPDATE: I had an old version of the webpack-hot-middleware, this was most likely the cause of my problems.
> ```


### npm install ${react-native plugin}, then packages are removed
#### 场景
其实是 npm 的一个 bug，安装一个包的时候会卸载所有包，并且 package.json 的信息也被维护了，

#### 解决
run `npm install` again.

> 参考：[https://github.com/npm/npm/issues/17379](https://github.com/npm/npm/issues/17379)



### svg use mouseEvent doesn't fire when xlinkHref-element is animating
#### 场景
1. 将 `use` 标签放在 svg 最底层，以便实现 `zIndex` 效果
2. `xlinkHref` 关联的元素具有动画效果，如 0% ~ 1% 渐变、hover 放大
3. 由于 `use` 覆盖了 元素，所以 *mouseLeave* 事件放到 `use` 上


##### bug
当关联元素进行 0% ~ 1% 动画时，`use` 标签的 *mouseLeave* 不会被触发。

##### some code
```html
<svg width="400" height="400" class="turntable" viewBox="-200 -200 400 400">
<defs>
<filter id="shadow">
	<feDropShadow stdDeviation="1">
</feDropShadow>
</filter>
</defs>
<circle cx="0" cy="0" r="200" fill="#fff"></circle>
<path id="pie-0" d="M0 0 L200 0 A200 200 0 1 1 -200 2.4492935982947064e-14" fill="rgba(171, 63, 184, 0.2160970845642396)" style="transform: scale(1);">
</path>
<path id="pie-1" d="M0 0 L-200 2.4492935982947064e-14 A200 200 0 1 1 200 -4.898587196589413e-14" fill="rgba(247, 90, 105, 0.6436378563060889)" style="transform: scale(1);">
</path>
<use xlink:href="#pie-0">
</use>
</svg>
```

##### img
![](//blog.azlar.cc/images/web-bugs/pie_animation_1.gif)
![](//blog.azlar.cc/images/web-bugs/pie_animation_2.gif)

#### 解决
暂无较好解决方案。

### str.match()
用正则时，会有些不同。

> 
> 返回值
> 存放匹配结果的数组。该数组的内容依赖于 regexp 是否具有全局标志 g。

> 说明
> match() 方法将检索字符串 stringObject，以找到一个或多个与 regexp 匹配的文本。这个方法> 的行为在很大程度上有赖于 regexp 是否具有标志 g。
> 
> 如果 regexp 没有标志 g，那么 match() 方法就只能在 stringObject 中执行一次匹配。如果没有找到任何匹配的文本， match() 将返回 null。否则，它将返回一个数组，其中存放了与它找到的匹配文本有关的信息。该数组的第 0 个元素存放的是匹配文本，而其余的元素存放的是与正则表达式的子表达式匹配的文本。除了这些常规的数组元素之外，返回的数组还含有两个对象属性。index 属性声明的是匹配文本的起始字符在 stringObject 中的位置，input 属性声明的是对 stringObject 的引用。

> 如果 regexp 具有标志 g，则 match() 方法将执行全局检索，找到 stringObject 中的所有匹配子字符串。若没有找到任何匹配的子串，则返回 null。如果找到了一个或多个匹配子串，则返回一个数组。不过全局匹配返回的数组的内容与前者大不相同，它的数组元素中存放的是 stringObject 中所有的匹配子串，而且也没有 index 属性或 input 属性。

> 注意：**在全局检索模式下，match() 即不提供与子表达式匹配的文本的信息，也不声明每个匹配子串的位置。如果您需要这些全局检索的信息，可以使用 RegExp.exec()。**


## React Native
### controled component
在 ios 端，input 其实是不可控的具体可见：

[https://github.com/facebook/react-native/issues/5370](https://github.com/facebook/react-native/issues/5370)

### animation with nativeDriver wrong callback
callback 其实会生效，但是值其实会回到 0。所以如果 animation 的退出值（如父级淡出）不为 0 时，此 animation 会回到值为 0。

（例：子级 slide-down，父级 fadeout，子级退出后触发 callback，会回到 slide 起始的位置）

[https://github.com/facebook/react-native/issues/11328](https://github.com/facebook/react-native/issues/11328)


## nodeJS

