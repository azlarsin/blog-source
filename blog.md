title: Development Calendar
author: azlar
date: '2016-09-08 21:20:20'
tags: [develop calendar]
ignore: false

---



记录开发 blog 的过程。
<!-- desc -->
## todo (del 线代表已完成...)
### 功能
- <del>根据 `Widget` 宽度与 `Tags`、`Articles` 的字体大小计算其最大显示字符数</del>
- `List` 样式 （page 的 comments 数量、样式）
- `Article` 样式
- <del>**`Pagination`**</del>
- <del> `build.js` 移动文件问题（现在需要 sudo 权限，由于有一个 `.DS_STORE` 文件存在） </del>
- <del>`build.js` => `new post`（命令生成 `yaml` 配置文件）</del>
- 归档
- SEO （sitemap）
- <del> !!! github pages **static html map** </del>
- 初次加载的时候，`config.json` 请求了两次：`Widget` 与 `ListComponent` 同时发起了 `ajax` 请求
- 列表筛选，最后更新时间（由于很多流水账会在不同时间续写）
- time-line 与 archive
- build 脚本新功能，自动部署到仓库
- 表格问题，macdown 的表格，marked 貌似不支持，需要自己分析下
- 逻辑优化。。。以前的代码看着太蛋疼了，就差重构了。。。（17.10.4）
- /demos/ 文件夹，每次在 build 之后可能会丢失
- 将目录、定位锚，改为该条字段的拼音，用数字会导致定位不准，非常不便于其他文章指向引用


### bugs
 - [tootip of tags](#toc_18)
 - 迁移项目目录，重构目录结构
 - `css` => `scss` 与样式重新整理、规范
 - 回到顶部小火箭的 `hash`
 - 列表页简介部分可能含有 `markdown` 语法
 - **文章内的 tags 不能有 `-` 否则转义失败**（由于 tag 之前有空格，路由内将空格替换为 - ，故在解析的时候，转义后不能解析，列表页会过滤失败）


## 开发

### 16.9.9
折腾了一天，把代码高亮从 [`prism.js`](http://prismjs.com/) 换到了 [`highlight.js`](https://highlightjs.org/)。其实使用上感觉并不明显，主要是后者高亮主题多（然并卵，还是自己边抄边调了一套高亮）。

后台编译（伪）程序 `build.js`，添加了一些处理细节，之后要开始写 `tag-map` 的生成。

自己在设计方面还是太差，设计出来的东西简直非人能看。

### 16.9.11
- 左部挂件： `tag-list`
- 挂件中的 `tag-list` 与 `list` 中的 `tag` 的选中状态联动。

### 16.9.12
#### 文章预览
本来使用的方式是从 `head 信息` 后截取 140 字符，再向后寻找到第一个 “。”；使用之后发现效果并不好。

于是使用了在 `md 文件` 中添加一个 `<!-- desc -->` 这种方式来实现（参考 `hexo` 的预览文本方式）。这种方式可定制性更强。

P.S. 再次感慨很多时候不能纯凭机器，有时一个标识就可以解决的问题，用语言却可以派生出很多问题。

#### long tag
在 `tag-list` 中，会截取一部分（:hover 的时候提示显示），在文档列表页，不会截取：
![long tag hover hint](//blog.azlar.cc/images/long tag hover effect.gif)

#### 文件移动
调用 `build.js` 的时候，会同时移动图片了，但是不能使用相对路径（由于 `index.html` 在 `build/` 的外面）。

正在考虑要不要将 index.html 移动到 `build` 内部。

#### 建立关联文章 `map`
通过分析每篇文章的 `tags`，获取拥有相同 `tag` 的文章。本来是在前端分析的，前端比较麻烦，而目前我是充分相信后端数据的，故使用后端直接分析得出 `index` 比较方便简单。

```javascript
//code in build.js
realListMap.map((article, index) => {
    let articleTags = article.tags;
    let articleRelated = [];
    for(let i = 0;i < articleTags.length;i++) {
        articleRelated = articleRelated.concat(tags[articleTags[i]].filter((item) => {
           return articleRelated.indexOf(item) < 0;
        }));
    }
    
    article.related = articleRelated;
});
```

### 16.9.13
#### `tag-list` 排序
按文章多少排序：

```javascript
var sortedKeys = Object.keys(tags).sort(function(a,b){return tags[b].length - tags[a].length});

//sortedKeys.map((tag) => {...});
```

#### auto scroll `tag.selected` into viewpoint
自动将当前页面，`tag-list` 中被选中的 `tag` 滑动显示。一般在选中 tag 后，刷新页面，或者点击 `:site/tag/xxx` 直接进入页面时会用到。

```javascript
//componentWillReceiveProps -> setState() -> callback()
let selectedTag = $(".tag-block.selected");
selectedTag[0].scrollIntoView({ behavior: "smooth" });
```

使用后发现，每次点击都会触发，体验很差，故需要判断其是否在视图 `viewport` 内；这个问题刚好之前研究过（两个 `rect` 是否相交）。之前有收藏一篇 [how-to-tell-if-a-dom-element-is-visible-in-the-current-viewport](http://stackoverflow.com/questions/123999/how-to-tell-if-a-dom-element-is-visible-in-the-current-viewport)，从里面选取了一个方法（有点懒得看之前写的判断两矩形相交）：

```javascript
function isElementInViewport (el) {
    //special bonus for those using jQuery
    if (typeof jQuery === "function" && el instanceof jQuery) {
        el = el[0];
    }

    var rect = el.getBoundingClientRect();

    return (
        rect.top >= 0 &&
        rect.left >= 0 &&
        rect.bottom <= (window.innerHeight || document.documentElement.clientHeight) && /*or $(window).height() */
        rect.right <= (window.innerWidth || document.documentElement.clientWidth) /*or $(window).width() */
    );
}
```
于是在视图内的 `tag` 被选中就不会自动滚动至顶部了，因为不会触发 `scrollIntoView()`：

```javascript
var selectedTag = $(".tag-block.selected");

if(selectedTag.length > 0 && !isElementInViewport(selectedTag)) {
    selectedTag[0].scrollIntoView({ behavior: "smooth" });
}
```

### 16.9.14
#### 部署倒 `github pages`
今天把项目部署到了线上后才发现一个 2b 的事情：`github pages` 并不支持 **`rewrite`**。

想了很多办法，甚至一度跑去开通了 `amazon` 的 `aws`，可是在这个过程中，心里一直念叨着：我写这个不就是为了弄一个纯静态的么，上 `vps` 不是坑自己么。。。

github 说对 jekyll 有一个插件支持，但我没调试那个。

于是想通过 `hack` 办法来实现，发现 gp 所有找不到的页面都会定向到 `404.html`，所以直接将该页面写为类似 `index.html` 的入口文件。问题解决。。

但是这样做应该有明显的问题：

- seo 问题
- 资源引用路径必须都使用绝对路径

故这只是一个临时的办法，若想实现静态化，除了虚拟出所有的 html 文件，貌似没有其他办法。（这个会在 `build.js` 生成文件的时候，将所有的 `html map` 生成）

P.S. `404.html` 真是好 `low` 的办法~

#### 移动端样式
部署后顺手写了点样式，由于之前的一部分是在 `PB` 项目开始之前写的，发现之前的样式写的好 low，又提不起精神改，头痛。。。

将样式整理与 `scss` 重写放进 `todo` 吧 -。-


### 16.9.19
#### 静态文件
终于把静态化的脚本写完了，文件目录变成了这样：
![](//blog.azlar.cc/images/static-html-file-structure.png)

基本和 `hexo`、`jeykll` 一个样子了，并且更搓的地方是，我这个还得走请求去请求 `md` 文件然后解析，还不如直接做全静态得了。

不过考虑以后可能会迁移到服务器，到时候会用数据库，所以现在就暂时保持这个蛋疼的模式吧。

#### 路由追加 `trailing slash`
为了配合静态路由，将所有的路由模式由 `domain/about` 改成了 `domain/about/`。

### 16.9.20
#### new post
终于把这个给写了，会生成一个 `.md` 文件：

```shell
node build.js -new about
```

```
about-1.md (about.md exists.)
----
title: about-1
author: azlar
date: '2016-09-20 17:45:14'
tags: ''

---

<!-- desc -->
```
一些小功能：
	
- `yaml` 信息写入
- 检查文件名、更新文件名：
  ```shell
	`node build.js -new a` => `a.md`
	`node build.js -new a` => `a-1.md`
	`node build.js -new a` => `a-2.md`
	`node build.js -new a-1` => `a-1-1.md`
	`node build.js -new %a]` => `a-3.md`
	```
- 默认加了 `desc` 标识符

### 16.9.22
#### tag-widget、article-widget 字符数自动适配
思路：
 
 - 计算 widget-panel 的宽度与字符能使用的宽度（`widgetPanelWidth - padding-left - padding-right`）
 - 计算最多能允许显示多少字符，截取
 - 通过 `state` 切换，动态控制字的截取
 - 为 `window` 绑定 `resize` 事件

效果：
![tag-widget resize event](//blog.azlar.cc/images/tag-widget resize event.gif)


遇到的坑：

 - `unicode` 字符截取、长度的计算。
 - `window.resize` 监控 resize end 的事件。
	```javascript
	clearTimeout(window.resizedFinished);
	window.resizedFinished = setTimeout(() => {
	var widgetPanel = document.getElementById("widget-panel");
	if(null !== widgetPanel) {
		
	    this.setState({
	        widgetPanelWidth: parseInt(window.getComputedStyle(widgetPanel).width)
	    }, () => {
	        console.log(this.state)
	    });
	}
		
	// alert('Resized finished.');
	}, 250);
	```




### 16.11.7
#### live code 运行
对某些特殊的代码（一般是 `javascript` 的 `canvas`），可以在线运行预览了。

在 `.md` 内代码块第一行添加：

```
#run: canvas	
```

解析时，程序会为该代码块分配一个写好的 *canvas* 运行环境（*iframe*）。

#### live code bugfix
##### 描述
运行代码，是通过一个新组件 `<LiveCode />` 实现的，是 `<Content />` 的一个子组件。

由于需要通过 *iframe* 控制，而不巧的，`<Content />` 的顶级 *wrapper* 使用了 `transform` 属性，故置于其中的元素不能使用 `position: fixed;` 浮起来了。

只能把 `<LiveCode />` 作为一个实现层，编译好代码后，挂载到 `<App />` 下的 `live-code-wrapper` 中。

```html
<!--APPComponent render()-->

<div className="blog" id="blog">
	<!-- ... -->
	<div className="overlay live-code" onClick={ this.__closeLiveCodeLayer }>
	    <div id="live-code-wrapper">
	        <span className="close" onClick={ this.__closeLiveCodeLayer }>x</span>
	    </div>
	</div>
</div>
```

这个时候便会触发一个比较尴尬的问题，`<Content />` 不知道 *live-code-wrapper* (*overlay*) 什么时候被关闭了，会不断的触发其子组件 `<LiveCode />` 的 **`componentWillReceiveProps`** 方法。

造成关闭 `overlay` 后，点击文章中任意链接仍会触发 `<LiveCode />` 中的编译挂载（上一个被关闭的运行结果不断浮出来）。

##### 解决
在 `<Content />` 传递给 `<LiveCode />` 的参数中，追加一个唯一串（每次点击 `run the code` 的时候自动更新），用来区分此时（`<LiveCode />`）是否需要运行代码。

```javascript
if(this.props.uuid !== props.uuid) {

	//generate the code & refresh the iframe 
	
}
```
	

### 16.12.16
#### 增加了分页支持
<del>目前还没支持刷新，有点懒得写呀 23333</del>

后边重新增加了 404.html，可以刷新了。	

### 17.2.7
#### 为代码块增加了语言显示
在代码块右上角增加了代码语言显示。

### 17.10.3 - 17.10.4
自己从自己的中秋礼物，keep going~~

#### react react router 升级
放弃了 router 2.x，拥抱新变化吧，升级的过程还是蛮有意思的。
```json
{
	"react": "^16.0.0",
	"react-dom": "^16.0.0",
	"react-router": "^4.2.0",
	"react-router-dom": "^4.2.2",
	"whatwg-fetch": "^2.0.3"
}
```

router change: 

```js
ReactDom.render(
    <Router history={ history } onUpdate={ logPageView } >
        <Route path="/" component={App} BaseOrigin={BaseOrigin}>

            <Route path='article/:articleName(/:ext)/' component={Content} />

            <Route path='about/' component={ Content } params={{ articleName: 'about' }} />

            <Route path='(tag/:tag/)(p/:p/)' component={ List } />

            <Route path="t" component={ Test } />

            <Route path="*" component={ List } onEnter={ enter }/> />

            <IndexRoute component={ List } />
        </Route>
    </Router>
    ,
    document.getElementById('qwe')
);
```
to:

```jsx
// in AppComponent
<Route path="/" component={ this.props.logPageView } />
<Switch>
    <Route path='/article/:articleName/:ext?/' component={ Content } />
    <Route path='/about/' render={ (props) => <Content { ...props } articleName={'about'} /> } />

    <Route path="/p/:p/" component={ List } />

    <Route path="/tag/:tag/" component={ List } />

    <Route path="/tag/:tag/p/:p?" component={ List } />

    <Route path={ '/(tag/:tag/)?(p/:p/)?' } component={ List } />

    <Route path="*" component={ List } />
</Switch>
```

#### 改写、优化
现在看以前的代码，真是设计的惨不忍睹，很多地方重复性太高、结构太复杂（乱）；用 childContext 改写了一部分，设计了一小部分新逻辑。

另外使用了 fetch，具体的还得测试。

#### remove jquery
最近很长一段时间，精力都在原生 js 上，所以这次下笔写，还蛮快的，也没怎么查文档（AC...），包的体积又小了很多（笑...）

#### 一点点心得
学习的过程是个积累的过程，尤其是思维逻辑上的学习、自我理解，只有用的时候才会感觉到，自己有没有变强。（喂，是在吐槽自己去年的代码太呕吐了么。。。。）

### 17.10.14
#### 优化
改了下 content 页内，作者与日期、标题的样式。

生成了 manifest.js 文件，并将所有资源文件移入 'assets' 文件夹。















## 维护

### 16.9.8
由于顶部有 `navigation-bar` 的存在，导致页面自动定位 `anchor` 的时候，会被遮蔽，故需要

- 方法1， css
```css
.blog .wrapper .content *[id]:before {
    display: block;
    content: "";
    margin-top: -82px;
    height: 82px;
    visibility: hidden;
}
```
	
- 方法二，js listen window.hasChange && locatin.href，then auto scroll the navigation-bar.height。
- 这种方法会有一个问题：**`点击同一个 hash 的时候，浏览器会认为当前位置并不是该 hash 所在的位置，然后重新定位，而此次定位，不会触发刚才写的 auto scoll 方法。`** 解决：为目录中的 `<a>` 添加一个 listener：该 hash 与当前 location.hash 相同的时候，preventDefault();
	


### 16.9.11
发现 切换 `tag` 时样式变化错误的 [bug](http://stackoverflow.com/questions/39443139/chrome-compute-style-error-with-transition)，尝试各种方式修复不能。

### 16.9.12
缩小了列表页，文章标题下 `tag` 的字体大小，由于有时候 `tag` 过多、过长，选中增大以后会比较难看。

在 `Widget` 组件中对 `shouldComponentUpdate` 做了一个单独处理：

```javascript
shouldComponentUpdate(nextProps, nextState) {
    return nextState.nowTag != this.state.nowTag;
}
```
由于 `shouldComponentUpdate ` 默认返回 `true`，故会造成一些不必要的更新（`Widget` 的 `props` 由 `App(Root Component)` 传入，又由 `App` 的某个子组建来更新，故 `Widget` 会被多次触发相同 `tag` 的 更新）。


### 16.9.13
瞎忙一天，就为了解决这个问题：[css-absolute-element-hid-by-overflow-auto](http://stackoverflow.com/questions/39463285/css-absolute-element-hid-by-overflow-auto)。

目前发现的是 `chrome` 解析没有父级约束 `.parent{ position: relative; }` 的元素的时候，在 `.grandpa {overflow: auto}` 存在并且 **触发滚动** 的时候，判断独立的 `absolute` 元素会基于其原有位置。

这个问题留待解决啊。

P.S. 感冒了，脑子不是很清醒。

### 16.9.14
#### [css-absolute-element-hid-by-overflow-auto](http://stackoverflow.com/questions/39463285/css-absolute-element-hid-by-overflow-auto)
感谢 `stackoverflow` 的小哥给出的解决方案：[jsfiddle](https://jsfiddle.net/TheBigDMo1/k2ahruyb/4/)，他的坚持不懈真是让我倍感羞愧。这个问题我还是准备先留一下，看看（不久的）将来有没有更好的解决方案。

####

### 16.9.18
#### 移动文件需要 `sodu` 权限 解决
使用 `fs-extra` 的时候，由于没有权限操作 `.DS_STORE`，所以加了 sodu，后来觉得不方便，将 `build` 文件夹删除后，再删除 `source` 内的 `.DS_STORE` 
就可以不用 `sudo` 来 `copy` 了。

### 16.9.20
#### 统一了 `date` 的格式
如果不是字符串（无引号）：

```
---
date: 2016-09-08 21:20:20
---
```

会导致 `yaml` 读取的时候自动转换成 `Date()`，操作起来还得转成 `UTC`，于是直接将所有的时间写成字符串：

```
---
date: '2016-09-08 21:20:20'
---
```

这样就不用每次读的时候转换了。

### 16.9.21
#### 优化了一下创建新文件的命名问题
美化了一下文章标题里含有 空格的情况，由于 `\s-\s` 我经常会使用：

```
bug
`node build.js -new 'a - b'` => `a---b.md`

fixed
`node build.js -new 'a - b'` => `a-b.md`
`node build.js -new 'a -        b'` => `a-b-1.md`
`node build.js -new 'a            -        b'` => `a-b-2.md`
```

### 16.9.26
#### `markdown` 格式错误
`macdown` 的格式与 `marked` 略有差异。写 `list` 的时候，代码块需要挨着上行内容，不能隔行写，否则转义出来的格式有问题。

#### 手机显示文章日期显示错误
由于之前的显示方法是对 `yaml` 信息进行转义显示，手机转义失败（`new Date(date)` 失败），这次直接当做字符串处理了(因为之前保存的时候已经当做字符串处理而部当做 `date` 信息保存)。

#### 左侧 `Widget` bug
##### 描述
当浏览一篇文章时，点击 about 页，`Widget` 会由于之前隐藏了，在渲染 `Tags` 组件的时候，计算出来的宽度是 0；随着 scrollbar 自动滚动到最上，widget 重新获取到宽度，而这个时候左侧的 *tags* 已经渲染完毕，*tag* 显示全部被缩略。

##### 解决
在 `Tags` 组件内，初次渲染时，宽度计算由 `parseInt(window.getComputedStyle(widgetPanel).width)` 变为 `document.body.offsetWidth * 0.22 * 0.9`（与 `css` 一致）。


### 16.10.17
#### border-radius
给 `panel` 增加了 4px 的圆角，不知道以后会不会抽风改成直角。。

#### `plain` 显示
转了一篇纯文本，增加了这个显示模式。

在 `.md` 内的头信息内，增加：

```
plain: true
```

会在文章处，获得一个段落缩进的效果。



### 16.11.18
#### 重写了文件 build 时的生成规则
先来看几条命令：
之前一直使用 `title` 做判断，即生成同名文章的时候，`title` 亦会被修改。

现在在生成 `file` 的时候，由 `title` 生成了一个临时变量 `fileName`，临时存储在 *yamlInfo*，此变量即是文件名，会在判断重名后、生成文件前被销毁。

file 1:

```shell
node build.js -new 'wewe-ewasd asdas'

# wewe-ewasd asdas created.
# It will open after 2 second...
# opening ./source/files/wewe-ewasd-asdas.md
```

file 2:

```shell
bogon:code_b azlar$ node build.js -new 'wewe-ewasd asdas'

# wewe-ewasd-asdas-1 created.
# It will open after 2 second...
# opening ./source/files/wewe-ewasd-asdas-1.md
```

此时，两篇文章的 title 仍然是 `wewe-ewasd asdas`，而文件名不同（`route` 不同）。

### 16.11.22
#### 添加了 `https` 支持
参照：
> https://zhuanlan.zhihu.com/p/22667528

使用 [cloudflare.com](cloudflare.com) 为博客增加 `https` 支持。

#### <del>为图片资源添加了 `lazyload`</del>
<del>采用了一个原生的库：[https://github.com/verlok/lazyload](https://github.com/verlok/lazyload)</del>

暂时先去掉了，效果并不好，一直加载图片资源的问题（disable_cache 下，如果打开某篇文章后再打开别的文章，此时会卡住，必须等到上一篇文章的资源加载完毕），是由于文章都使用同一个 `Component` 导致的，得想办法处理。


### 16.12.20
#### 重写了左侧 widget 的 scroll 事件
##### 原来
今天发现之前写的：

```javascript
$(".tag-box.block").on( 'mousewheel DOMMouseScroll', function ( e ) {
    var event = e.originalEvent,
        d = event.wheelDelta || -event.detail;
    
    this.scrollTop += ( d < 0 ? 1 : (Math.abs(d) == 0 ? 0 : -1) ) * 15;
    e.preventDefault();
});
```

对 *wheel event* 支持的并不好，体验比较差。

##### 重写
问题的关键在于，滚动此 `div` 的时候，会触发 `body` 的 *scroll* ，导致` body` 滚动。


Firefox 比较好，在 `scroll` 事件里，增加一个 `e.stopPropagation();` 即可防止 `scroll` 扩散到 `body`。


chrome 没啥用，只能手写阻止：

	1. 找到当前的 scroll position
	2. 在顶部（底部），继续向上（下）滚动时，阻止事件，防止触发 body

代码（使用 `onWheel` 绑定到 `widget` 上）
```javascript
//end
if (
    e.currentTarget.scrollTop === (e.currentTarget.scrollHeight - parseInt(window.getComputedStyle(e.currentTarget).height))
    && e.deltaY > 0
) {
    e.stopPropagation();
    e.preventDefault();
}

//top
if (e.currentTarget.scrollTop === 0 && e.deltaY < 0) {
    e.stopPropagation();
    e.preventDefault();
}
```

这样会遇到的问题：滚动完毕后，移动鼠标的焦点其实还是在 `widget` 上，继续滚动，仍然会触发 `onWheel`，导致 `body` 不能滚动。

于是为 onWheel 增加了一个 `if(isMouseOver)` 判断，`isMouseOver` 会在 `onMouseEnter` 与 `onScroll` 时设定为 *true*，`onMouseLeave` 的时候设定为 *false*。

P.S. `onScroll` 修改 `isMouseOver`，是因为 mac 支持无焦点滚动，这时候当前页面没有获取到系统的焦点、`mouseEnter` 并不会触发。


### 17.9.22
改了很多东西。。

#### loading 重做
#### webpack 压缩重写
##### 分段
文件分为 `vendor.js` 与 `main.js`，另外把 `css` 文件也拆出去了。

##### 文件版本号
由于之前没有版本控制（其实主要是面试被问到这个），于是突发奇想，为项目配一个版本号，使用 webpack 插件的形式书写：

```js
// webpack config ...
plugins: {
    function() {
        this.plugin("done", function(statsData) {
            let stats = statsData.toJson();

            fs.writeFile("bundle-info.json", JSON.stringify(stats, null, 2), () => {});

            if (!stats.errors.length) {

                let publicPath = path.resolve(__dirname, "build"),
                    files = ["index.html", "404.html"],
                    css = stats.assetsByChunkName.app.find(assetName => path.extname(assetName) === ".css" ) || "main.css";

                files.map(fileName => {
                    let filePath = path.resolve(publicPath, fileName),
                        html = fs.readFileSync(filePath, "utf8");

                    if(html) {
                        let htmlOutput = html
                            .replace(/<script src="(.*)"><\/script>[\r\n]*<script src="(.*)"><\/script>/g, `<script src="//blog.azlar.cc/${stats.assetsByChunkName.vendor[0]}"></script>\r\n<script src="//blog.azlar.cc/${stats.assetsByChunkName.app[0]}"></script>`)
                            .replace(/link rel="stylesheet" href="(.*)"/g, `link rel="stylesheet" href="//blog.azlar.cc/${css}"`);

                        fs.writeFileSync(filePath, htmlOutput);
                    }
                });
            }


            //delete junk files
            let allFiles = new Set(fs.readdirSync(path.resolve(__dirname, "build"))),
                assetNames = new Set(stats.assets.map(asset => asset.name).concat([stats.assetsByChunkName.vendor]));

            let junkFiles = new Set([...allFiles].filter(x => !assetNames.has(x)));

            // console.log(junkFiles);

            for(let fileName of junkFiles) {
                if([".js", ".css", ".map"].indexOf(path.extname(fileName)) !== -1) {
                    fs.unlink(path.resolve(__dirname, "build", fileName), () => {});
                }
            }
        });
    }
}
```

这样就可以在打包后自动替换相关文件了，在 dev 模式中，可以直接输出到普通文件，这样就可以继续使用 debug 了。

#### webpack 压缩过大
主要原因是使用了 `node-uuid` 这个库，加载了过多依赖。


### 17.10.4
Articles 挂件中 Related Post 的链接有点问题，缺失最后的 '/'。

### 17.10.14
修正了 webpack 每次生成文件时，vendor.[chunkhash].js 的 hash 会改变的问题：

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

参考：[https://github.com/webpack/webpack/issues/1315](https://github.com/webpack/webpack/issues/1315)