---
title: 【vue系列】vue2.x 项目配置 Mocha 单元测试
date: 2020-01-20
tags: 
---

我们团队中的项目众多，基本上都没有配置单元测试，其中一个 vue2.x 项目是我们的主站项目，很早之前我就想给这个项目加上单测，只是一直忙于业务就没顾上。正巧马上过年了，最近需求不多，忙完需求，就开始搞搞单测。

从有配置单测的想法我就一直在思考，我们这个项目是否真的需要加单测，我们要测试什么，那么多的测试运行器又该怎么选择，以及怎么在团队中推行单测。基于这些思考，开始实施了。

<!--more-->

我们团队中的项目众多，基本上都没有配置单元测试，其中一个 vue2.x 项目是我们的主站项目，很早之前我就想给这个项目加上单测，只是一直忙于业务就没顾上。正巧马上过年了，最近需求不多，忙完需求，就开始搞搞单测。

从有配置单测的想法我就一直在思考，我们这个项目是否真的需要加单测，我们要测试什么，那么多的测试运行器又该怎么选择，以及怎么在团队中推行单测。基于这些思考，开始实施了。

## 我们是否真的需要单元测试

目前整个团队的所有的项目都没单测，而其它团队的项目很多都有对应的测试工具，总觉得我们技术栈少了点什么。这虽然是我要加单测的充分条件，却是我想配置单测的初衷。

#### 项目背景

其次我们这个项目维持了四五年，进行了至少4次项目架构重构的升级，这个项目中有主要有 v7 版、v8 版，总计 近200 个页面，目前大部分的业务都在 v8 版。当然我们还有 v9 版，由于项目业务不断增多，不得不对仓库进行拆分，v9版 是基于 vue-cli 搭建的的一个新仓库，这里就不在赘述了。前两版的技术栈选型分别是：

- v7版  `juicer + jQuery`
- v8版 `vue2.0 + webpack`

就现状来看，给v8版配置单测比较合适，一方面主要业务在v8上，另一方面从使用的 `vue2.0 + webpack` 技术栈来看，更适合写单元测试。

#### 预期期望

我期望给 v8 版的公共组件和工具方法都加上单元测试，这对团队代码的可靠性验证来说肯定是有益的。后期迭代，也可以通过单测防止意外情况的发生。嗯，这就是要加单测的必要条件吧。

除此之外，组件的单元测试还可以为我们带来更多的好处：

- 提供描述组件行为的文档
- 节省手动测试的时间
- 减少研发新特性时产生的 bug
- 改进设计
- 促进重构

自动化测试可以帮助团队中的开发者可以维护复杂的基础代码。单元测试是测试体系中最有用的，即能帮助开发者思考如何设计一个组件，又可以帮助开发者重构一个现有组件。

一开始，当大家对一个项目的添加单测的愿景还不够清晰的时候，单元测试可能会拖慢开发进度，但是一旦大家的愿景建立起来并且有真实的用户对这个应用产生兴趣，那么单元测试就是绝对有必要的了，它们会确保基础代码的可维护性和可扩展性。


## 如何选择测试运行器

单元测试应该具备：
- 可以快速运行
- 易于理解
- 只测试一个独立单元的工作

[Vue Test Utils](https://vue-test-utils.vuejs.org/zh/)  是 Vue 组件单元测试的官方库，官方推荐两个测试运行期：

- `Jest` 是功能最全的测试运行器。它所需的配置是最少的，默认安装了 `JSDOM`，内置断言且命令行的用户体验非常好。不过你需要一个能够将单文件组件导入到测试中的预处理器。我们已经创建了 `vue-jest` 预处理器来处理最常见的单文件组件特性，但仍不是 `vue-loader` 100% 的功能。
- `mocha-webpack` 是一个 `webpack + Mocha` 的包裹器，同时包含了更顺畅的接口和侦听模式。这些设置的好处在于我们能够通过 webpack + `vue-loader` 得到完整的单文件组件支持，但这本身是需要 **很多配置** 的。

我个人更偏向于 `mocha-webpack`，毕竟测试单文件组件，上手又快、功能又全。

## 配置 mocha 单元测试

**浏览器环境**
Vue Test Utils 依赖浏览器环境，用 `JSDOM` 在 `Node` 虚拟浏览器环境运行测试。

`Vue` 的单文件组件在它们运行于 `Node` 或浏览器之前是需要预编译的。我们直接使用 `webpack`。针对 `mocha` 需要基于 `webpack` + `vue-loader` 进行设置。

选择了 `mocha-webpack` 测试单文件组件，是通过 webpack 编译所有的测试文件然后在测试运行器中运行。这样做的好处是可以完全支持所有 webpack 和 `vue-loader` 的功能。`mocha-webpack` 能够提供了非常流畅的体验。

####  1. 设置 mocha-webpack
v8项目基于已经安装并配置好了 `webpack`、`vue-loader` 和 `Babel`。首先要做的是安装测试依赖：

```javascript
npm install --save-dev @vue/test-utils mocha mocha-webpack
```

在 `package.json`中定义一个测试脚本。

```javascript
// package.json
{
  "scripts": {
    "test": "mocha-webpack --webpack-config webpack.config.js --require test/setup.js test/**/*.spec.js"
  }
}
```

#### 2. 暴露 NPM 依赖
将依赖外置以提升测试的启动速度，通过 `webpack-node-externals` 外置所有的 `NPM` 依赖：
```javascript
// webpack.config.js
const nodeExternals = require('webpack-node-externals')

module.exports = {
  // ...
  externals: [nodeExternals()]
}

```

####  3. 源码表
源码表在 `mocha-webpack` 中需要通过内联的方式获取。推荐配置为：

```javascript
module.exports = {
  // ...
  devtool: 'inline-cheap-module-source-map'
}
```

#### 4. 设置浏览器环境
`Vue Test Utils` 需要在浏览器环境中运行。我们可以在 `Node` 中使用 `jsdom-global` 进行模拟：

```javascript
npm install --save-dev jsdom jsdom-global
```

然后在 `test/setup.js` 中写入：
```javascript
npm install --save-dev jsdom jsdom-global
```
这行代码会在 Node 中添加一个浏览器环境，这样 Vue Test Utils 就可以正确运行了。

#### 5. 选用 `expect` 断言库
推荐 Jasmine/Jest 风格的 `expect` 断言
```javascript
npm install --save-dev expect
```
然后在 `test/setup.js` 中编写：
```javascript
require('jsdom-global')()

global.expect = require('expect')
```
#### 6. 为测试优化 Babel
`mocha` 是比较常用的 `node` 测试框架，但是只支持 `commonjs` 模块，要让 `mocha` 支持 `ES6` 模块，需要 `babel`的帮助。`babel-loader` 将会自动使用相同的.babelrc配置文件。

## 写单测的注意事项 
- describe 测试套件
- 每个 it 块只做一个断言，it块称为"测试用例"
- 让测试描述更简短清晰
- 只提供测试需要的最小化数据
- 把重复的逻辑重构到了一个工厂函数中

## 小结：
目前已经写了一些单测，预期是给公共组件和工具类函数都加单测。业务需求示情况添加单测。

Vue 组件单元测试的一般思路：渲染这个组件，然后断言这些标签是否匹配组件的状态。

通常，在实现完成后添加测试时，开发们倾向于接受程序生成的输出，而不必通过手动计算仔细检查结果。
将要求转化为可伪造的测试。

## 疑问
好像已经形成了习惯，每次都会留下一些疑问，后面慢慢解疑吧。
- 如何在组件中测试 Vue-router
- 如何在组件中测试 Vuex
- 如何提升测试的启动速度？
- chai断言库与 Jest的 expect 有什么区别
- Sinon 是什么
- mixin、watch、computed如何添加单测

## 参考
- [mocha官网]( https://mochajs.org/)
- [mocha-webpack]( http://zinserjan.github.io/mocha-webpack/)
- [jest/expect](https://jestjs.io/docs/zh-Hans/expect)
- [Vue Test Utils 官网](https://vue-test-utils.vuejs.org/zh/)
- [使用 Mocha 的示例工程](https://github.com/vuejs/vue-test-utils-mocha-webpack-example)