title: GTD 软件体验
author: azlar
date: '2024-01-16 12:26:16'
tags: [gtd, things3, omnifocus]

---
随着 omnifocus4 发布，又想整活了最近。搞了 things3 和 omnifocus4 来体验，记点流水账。
<!-- desc -->
# Apple Reminders
## 优点
地理位置比如：getting in car 之类的。（things3 可以通过 shortcut 实现）
# things3
## 优点
### 1. 神来之笔，someday 和 anytime 的设计
omnifocus 时，经常会有很多任务需要扔到 `Future` 内，但是心里会有点记挂，点开 `Future` 看时，这些任务是分开的（按你扔进来的时间 +x day）、通过新的时间来分组，比较凌乱，如：两个电影任务，一个是 10 天前扔进来的，一个是今天扔的，在 `Future` 内也是分开的。但是在 `Someday` 内，会聚合到一起，体验稍微好点。

同理，`Anytime` 用来放一些一有空就想做的小 task，非常合适。

## 缺点

### 1. reminder 操作不同步
有一个抢购任务，需设置在 `2:59 PM` 给一个提醒，IOS 上的选择器只支持 5min 的最小 gap（虽然支持手动输入，它也会自动定位到附近的 5min 处）；实际方法：日历上方的搜索框内输入 259pm，选择 `today 259pm` 会自动生成一个 reminder。

体验非常割裂（mac 上直接在选择器处输入）、违和（搜索的时候又自动创建了 reminder）。

附：直接在 iOS 内输入 2:59pm（或任意特定 分钟级 时间） 可创建提醒。

### 2. repeat 
1. 设置 weekdays 需要一天一天勾选
2. quick entry 下，无法设置 repeat。需要二次点开

#### 2.3 体验极差的 append-content
repeat 不支持附加信息。如果在 `Today` 栏编辑了 content 后，附加信息不会追加到 next-repeating-task 内。

附：必须从 `Today` 内，点击 repeat 跳转到 `Upcoming` 内对真正的 repeat-source-task 进行编辑，才可实现 content update。然后还需要回到 `Today` 勾选掉当天的 task。

### 3. tag
1. 选择 tag，没有层级概念，全是平铺的（有的地方有层级，有的地方没有）
2. 新增 tag 时，不支持 `a:b:c:d` 层级创建

### 4. undo/redo
经常断，体验割裂。

### 5. 不支持提前勾选 repeat 任务
如：每月 16 号会重复进行 task-A，无法在 15 号时于（Upcnming 内）提前勾选本月的完成。

体验极差，比如某任务十天一次，提前了一两天顺手结束，必须去 Upcnming 内重新编辑 repeating task。也无法拖拽任务到 Today 栏内。

附：其他单次任务可以提前完成；OF 内所有任务都是 checkbox 可以提前在 future 内勾选。

### 6. 不支持 due
当一个任务延期（开始于 x 天前），会一直停留 `Today` tab 里，点开详情，它的所属（When）会自动更新到 Today。

无法准确的记录任务的 开始时间、延期时长。

### 7. 不支持传图
有时图片信息需要记录到 note 内。omnifocus 云同步附件，体验很不错。



# omnifoucs
## 缺点
### 1. 贵
### 2. 经常拖拽、输入法弹起等引起崩溃
本以为是 testflight 版本问题，没想到正式版依然如此。

### 3. 不支持 checklist
对比 things3，note 内容处不支持 checklist。