title: 前端 bug or 解决方案集合
author: azlar
date: '2017-10-14 10:16:24'
tags: [前端, bug, 坑, 解决方案, javascript, node, css]

---

流水账式记录前端遇到的各种问题，包括不限于框架、node、浏览器问题等。
<!-- desc -->

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


