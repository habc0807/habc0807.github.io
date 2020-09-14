---
title: 【vue系列】当 element-ui 按需引入遇到 vue-router 路由懒加载
date: 2020-02-29
tags: 
---

当前的一个项目用到了 `element-ui` 的部分组件，因为是只是使用部分组件，我们只引入需要的组件，以达到减小项目体积的目的。按照 `element-ui` 按需引入的方法首先，安装了 `babel-plugin-component`

<!--more-->


## 问题背景

当前的一个项目用到了 `element-ui` 的部分组件，因为是只是使用部分组件，我们只引入需要的组件，以达到减小项目体积的目的。按照 `element-ui` 按需引入的方法首先，安装了 `babel-plugin-component`

```
npm install babel-plugin-component -D
```

然后在根目录下添加 `.babelrc `文件：

```javascript
{
    "presets": [["es2015", { "modules": false }]],
    "plugins": [
      [
        "component",
        {
          "libraryName": "element-ui",
          "styleLibraryName": "theme-chalk"
        }
      ]
    ]
}
```

接下来，我们只希望引入部分组件，比如 Table 和 TableColumn，那么需要在 `index.js` 入口文件中写入以下内容：

```javascript
import router from './router'
import { Table, TableColumn } from 'element-ui'

Vue.use(Table)
Vue.use(TableColumn)

new Vue({
    el: '#myApp',
    router,
    template: `<router-view></router-view>`,
})
```

随着项目业务的不断增多，打包的Javascript文件变的非常大，影响页面加载。于是想针对不同路由对应的组件分割成不同的代码块，然后当路由被访问的时候再加载对应组件。

结合 Vue 的异步组件和 Webpack的代码分割功能，实现路由组件的懒加载。使用 `Vue-Router` 官网推荐的 `动态import` 语法定义代码分块点（split point）

```javascript
// …

const App = () => import('./pages/app')
const Subtable = () => import('./pages/subtable')
const routes = [
    {
        path: '/',
        name: 'app',
        meta: {
            title: '下属报表'
        },
        component: App
    },
    {
        path: '/subtable',
        name: 'app',
        meta: {
            title: '下属统计表'
        },
        component: Subtable
    }
]
// …
```

配置好进行打包构建，报错了，官网有段提示：

> 添加 [syntax-dynamic-import](https://babeljs.io/docs/plugins/syntax-dynamic-import/) 插件，才能使 Babel 可以正确地解析语法。安装了插件，发现依旧报错，仔细观察插件需要在.**babelrc** 中配置：

```javascript
{
 "plugins": ["@babel/plugin-syntax-dynamic-import"]
}
```

这里跟 `element-ui` 的 `.babelrc` 的配置有冲突。只能想其他办法解决了。

## 路由组件按需加载

#### Vue的异步加载组件（不推荐）

通过 `resolve => require([‘../path’], resolve)` 的方式可以实现按需加载，并且一个组件会打包成一个js文件。

```javascript
const App = resolve => require(['./pages/app'], resolve)
const Subtable = resolve => require(['./pages/subtable'], resolve)
```

#### webpack2中的require.ensure() （不推荐）

通过 `require.ensure()` 实现按需加载，接收两个参数，第二个参数可以指定chunkName。

```javascript
const App = r => require.ensure([], () => r(require('./pages/app')), 'app')
const Subtable = r => require.ensure([], () => r(require('./pages/subtable')), 'subtable')
```

### es提案的动态import（推荐）

虽然推荐该方法，但是在当前情况下使用报错，目前可以采取方案一或者方案二。问题搞定，就酱吧。😄

