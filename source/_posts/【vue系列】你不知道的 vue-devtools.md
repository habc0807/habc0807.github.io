---
title: 【vue系列】你不知道的 vue-devtools
date: 2020-01-01
tags: 
---

vue-devtools 使用了很长一段时间了，使用过程中发现 vue-devtools 下的 vuex 面板功能很厉害，整个页面的状态可以任意切换到之前的一个时间节点，当时只知道时光穿梭这个概念，没有仔细研究，最近决定入坑研究下。

记的之前有个Vue项目有线上bug，因为是线上项目通常会禁用 `vue-devtools`，不能直接使用 `vue-devtools` 来排查问题，经过探索找到解决办法。相信很有人都有 “如何使用 `vue-devtools` 查看线上项目” 的疑惑。

日常编程，有些开发者没有翻墙，或者懒得翻墙。不能直接到 `chrome` 商店搜索 `vue-devtools` 下载安装，那么 “不翻墙怎么安装 `vue-devtools`” 呢？

今天就来说说，这些你可能不知道的`vue-devtools`。

<!--more-->


vue-devtools 使用了很长一段时间了，使用过程中发现 vue-devtools 下的 vuex 面板功能很厉害，整个页面的状态可以任意切换到之前的一个时间节点，当时只知道时光穿梭这个概念，没有仔细研究，最近决定入坑研究下。

记的之前有个Vue项目有线上bug，因为是线上项目通常会禁用 `vue-devtools`，不能直接使用 `vue-devtools` 来排查问题，经过探索找到解决办法。相信很有人都有 “如何使用 `vue-devtools` 查看线上项目” 的疑惑。

日常编程，有些开发者没有翻墙，或者懒得翻墙。不能直接到 `chrome` 商店搜索 `vue-devtools` 下载安装，那么 “不翻墙怎么安装 `vue-devtools`” 呢？

今天就来说说，这些你可能不知道的`vue-devtools`。


## 1. vue-devtools 的时间旅行 - time travel 

> 建议使用过 `Vuex` 之后，再阅读本模块

时间旅行是一种很有趣想法。爱因斯坦的广义相对论说，时间旅行实际上是可能的。不过，到目前为止，时间旅行的机器还没有问世。


在前端中，`Vuex` 借鉴 `Flux` 单向数据流思想，采用集中式统一管理保存状态。但是这些状态不会随时间改变而变化。为了使状态，可能被捕获、重播或重现，通过时间来回旅行。尤大开发了 `vue-devtools` 这样的工具，可以帮助我们的 Vue 应用可以实现这种时间旅行！

#### 保证时光旅行的前提

`Vuex` 通过以下两个原则实现了这一点：
- 单一的状态树 `store`
- 只能通过 `mutation` 更改状态

#### `vue-devtools` 的视图面板

`vue-devtools` 的最新版 5.0beta 版为调试工具又带来新功能，增加了`Routing` 、`Performance`、`Settings` 面板，大大提高了我们的开发调试效率，不得不说是个神器。

![v5.0beta](https://user-gold-cdn.xitu.io/2020/1/2/16f6400affb5431c?w=870&h=507&f=png&s=87407)

- `Components` 组件面板: 检查组件、按组件过滤、显示原始的组件名、可编辑组件的data；
- `Vuex` 状态面板: mutations记录、过滤mutations、state\getters\mutations\actions\props 检查器可折叠
- `Events` 事件面板: 事件历史、按组件搜索事件；
- `Routing` 路由面板: 查看路由历史记录、路径变更详情；
- `Performance` 性能面板: 每秒帧数帧率、组件渲染的性能图表、组件各生命周期函数的耗时统计和分析；
- `Settings` 设置面板: 调整工具界面的主题、正规化组件名、调整工具的界面布局。

这次要探讨的时光旅行并是什么新特性，`time travel` 在 `Vuex` 状态面板，接下来我们来看看 `Vuex` 检查器

#### time travel 时间旅行（划重点）

`Vuex` 面板上有个神奇的功能：`time travel`。可以进行时光穿梭是因为，所有的状态变更只能通过 `mutation` 状态，并且每次状态的改变都会生产一个全新的 `state` 对象。把每次变更的 `state` 对象事件都记录下来，展现出一个 `mutation` 列表，当你想展现什么时间段的状态，只需要切换到那个时间段的 `state` 对象。

> 每次产生新state的时候，需要对原state对象进行深拷贝，保证每次状态变更原是state不变，操作新state。

`time travel` 的内部实现，流程如下。

> 主要分` vue-devtools-master` 源码中的 `time travel` 内部实现（v4.1.5）。

（1）触发`time travel`

`vuex` 面板左侧的 `mutation` 列表，每一行，鼠标放到该行的时候，会显示三个小图标，它们的功能是：`commit` 这次的 `mutation` 、重置 `mutation` 、对这次的状态时光旅行。

点击时钟小图标，触发 `vue-devtools-master 2/src/devtools/views/vuex/VuexHistory.vue` 下的 `timeTravelTo` 函数，该操作是一个 `action` 行为。

（2）触发 `action`

该 `action` 在 `vue-devtools-master 2/src/devtools/views/vuex/VuexHistory.vue` 中：

```javascript
export function timeTravelTo ({ state, commit }, entry) {
  travelTo(state, commit, state.history.indexOf(entry))
}

function travelTo (state, commit, index) {
  const { history, base, inspectedIndex } = state
  const targetSnapshot = index > -1 ? history[index].snapshot : base

  // 更改当前 mutation 列表的选中行
  bridge.send('vuex:travel-to-state', stringify(parse(targetSnapshot).state))
  if (index !== inspectedIndex) {
    commit('INSPECT', index)
  }
  
  // 触发commit 操作
  commit('TIME_TRAVEL', index)
}
```

`timeTravelTo` 调用的时候，执行 `travelTo` 方法，该方法主要做了两件事：一是告诉 `vue-devtools`，更改了vuex面板的时光旅行，选中的样式和状态你更改下；二是触发 `commit` 操作，去触发 `mutation`。


（3）触发 `mutation`

这部分的处理在 `vue-devtools-master 2/src/devtools/views/vuex/module.js` 中：

```javascript
const mutaions = {
'INSPECT' (state, index) {
    state.inspectedIndex = index
  },
'TIME_TRAVEL' (state, index) {
    state.activeIndex = index
 }
}

const getters = {
  inspectedState ({ base, inspectedIndex }, getters) {
    const entry = getters.filteredHistory[inspectedIndex]
    const res = {}

    if (entry) {
      res.mutation = {
        type: entry.mutation.type,
        payload: entry.mutation.payload ? parse(entry.mutation.payload) : undefined
      }
    }

    const snapshot = parse(entry ? entry.snapshot : base)
    if (snapshot) {
      res.state = snapshot.state
      res.getters = snapshot.getters
    }

    return res
  },

  filteredHistory ({ history, filterRegex }) {
    return history.filter(entry => filterRegex.test(entry.mutation.type))
  }
}
```

`INSPECT` 和 `TIME_TRAVEL` 会直接更改 `inspectedIndex` 和 `activeIndex`，当它们变更的时候会触发getter的 `inspectedState` 重新计算。确定进行这次的时光旅行，更改状态。

一次完整的时光旅行就完成了，怎么样？这些东西过一遍理解下原理，没想象中那么难吧。

## 2. 如何使用 `Vue Devtools` 调试Vue线上项目

线上的vue项目 `vue-devtools` 被禁用了，那么线上项目的bug怎么使用 `vue-devtools` 调试呢。

#### 获取Vue实例

通常情况下 vue 的源码都被打包到 `vendor.xxx.js`  文件中，也可能不是，可以去chrome开发者工具 `Sources` 面板中去找，这里以掘金为例。


![](https://user-gold-cdn.xitu.io/2020/1/3/16f6a46b67a960d1?w=1000&h=554&f=jpeg&s=134637)

- 在chrome开发者工具 `Sources` 面板，搜索 `prototype.$nextTick`；
- 在前一行代码上打断点，刷新页面；
- `mn.prototype.$nextTick` 的 `mn`就是vue，它 会根据不同的打包压缩有不同的命名；
- 在控制台中保存 `vue` 实例: `vv = mn;`；
- 如果保存失败，点击下一步执行几次，直到vue实例化完成，并保存好vue实例。

#### 启用 `vue-devtools` 工具

- 在控制台中执行代码：`vv.config.devtools = true;`如果报错，八成是vue实例还没完成。请继续对断点执行。直到可以运行成功。
- 然后在控制台中执行代码：`__VUE_DEVTOOLS_GLOBAL_HOOK__.emit('init', vv);`，对 `vue-devtools` 进行初始化。

![](https://user-gold-cdn.xitu.io/2020/1/3/16f6a4b689a8ff7e?w=2050&h=606&f=jpeg&s=183351)

- 关闭控制台，再打开，看到控制台中多了 Vue 面板
- 然后就可以尽情排查问题了。


![](https://user-gold-cdn.xitu.io/2020/1/3/16f6a4e194ba49e4?w=1000&h=366&f=jpeg&s=77892)


## 3. 不翻墙，手动安装 vue-devtools

通常情况下如果我们想对 `vue` 项目，使用 `vue-devtools` 工具调试，可以到 `chrome` 商店，直接搜索 `vue-devtools` ，下载安装就好了，只是直接从chrome商店下载插件需要翻墙。如果不方便翻墙可以用下面的步骤手动安装。

![ended.jpg](https://user-gold-cdn.xitu.io/2020/1/2/16f64061d559c9b7?w=1240&h=573&f=jpeg&s=33489)


- 找到[vue-devtools](https://github.com/vuejs/vue-devtools.git)的github项目，克隆到本地；
- 进入到项目根目录，安装依赖 `npm install` 或着 `cpm install`；
- 找到 `vue-devtools/shells/chrome/manifest.json` 文件，修改 `persistent` 为 `true`；
- 编译构建 `npm run build`；
- 打开chrome浏览器，输入 `chrome://extensions` 进入扩展程序页面，点击 `加载已解压的扩展程序…` 选择 `vue-devtools>shells>chrome` 文件夹 （记得打开开发者模式
- `Vue.js devtools` 就安装好了，点击扩展应用卡片上的 `详细信息`，勾选下面的三个选项。

> 最后，记得在项目中开启 devtools哦！

```javascript
new Vue({
  router,
  store,
  devtools: true, // 看这里 👀
  render: h => h(App)
}).$mount('#app');
```

![详细信息](https://user-gold-cdn.xitu.io/2020/1/2/16f640d2d776e20b?w=1632&h=1308&f=jpeg&s=163247)


## 参考 

- [Time travel in Redux](https://cuyu.github.io/javascript/2017/04/25/Time-travel-in-Redux)
- [[译] 深度介绍 Vue DevTools 5.0 新特性](https://juejin.im/post/6844903689099051021)
- [怎样用Vue Devtools调上线的页面](https://juejin.im/post/6844903671956766728)