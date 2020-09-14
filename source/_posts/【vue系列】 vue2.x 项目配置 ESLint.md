---
title: 【vue系列】 vue2.x 项目配置 ESLint
date: 2020-01-08
tags: 
---

协同开发过程中，不同的编码习惯写出的代码，差异很大。日常维护代码或者修复bug的时候时候，看各种风格的代码，影响效率不说，还有可能改出低级问题。目前我们的项目还没有代码规范，我决定要做点改变，于是考虑使用 `ESlint` 来做代码规范检查。

考察了[ESlint的官方网站](http://eslint.cn/)和一些技术贴，决定先给我们其中的一个 `Vue2.0` 项目配置 `ESLint`。在配置 `ESLint` 过程中走了一些弯路，把配置过程记录下来，方便日后查看。


<!--more-->


协同开发过程中，不同的编码习惯写出的代码，差异很大。日常维护代码或者修复bug的时候时候，看各种风格的代码，影响效率不说，还有可能改出低级问题。目前我们的项目还没有代码规范，我决定要做点改变，于是考虑使用 `ESlint` 来做代码规范检查。

考察了[ESlint的官方网站](http://eslint.cn/)和一些技术贴，决定先给我们其中的一个 `Vue2.0` 项目配置 `ESLint`。在配置 `ESLint` 过程中走了一些弯路，把配置过程记录下来，方便日后查看。

预测获得的收益：

- 项目编码风格统一，师出一人；
- 开发过程中修复语法错误，减少潜在 `bug`；
- 规定规范编码，减少代码冗余，提高代码质量；

预期风险：

担心整个项目添加了 `ESLint`，其他同事冒火，因为是多页面应用，在项目根目录下做了配置，新需求可以使用新配置，老项目可以自行选择要不要改一轮eslint的报告提示，这次给我们的 `Vue2.x` 项目配置了 `ESLint` 也给团队做点贡献。

## 配置ESLint

咳咳，配置流程来了。

#### 1. 安装 eslint
既然要用  `ESLint` ，就要安装它，通过命令： `cnpm I eslint -D` 安装。

#### 2. 初始配置 eslint
`eslint` 安装好之后，运行  `eslint —init`命令做些简单配置，我们可以选择我们需要的检测的环境、文件类型等等。这步完成，在项目根目录下会生成一个 `eslintrc.js` 文件。 

#### 3. 安装 eslint-loader
我们的项目需要webpack来编译，需要对应的loader，安装 `eslint-loader`，运行命令 `cnpm I eslint-loader -D`。

> 如果有 `command not found: eslint` 的报错提示，可以运行 `./node_modules/.bin/eslint --init` 来进行eslint配置化安装

#### 4. 配置 webpack
打开 `webpack` 的 `config` 配置文件配置：

```javascript
obj.module = {
    rules: [
        // ...
        { 
            test: /\.(js|vue)$/,
            loader: 'eslint-loader',
            enforce: 'pre',
            include: [ path.resolve(commonDir, '../index.js'), resolve('./pages'), resolve('./components')], // 
            options: {
                formatter: require('eslint-friendly-formatter')
            }
        }
    ]
};
```

#### 5. 安装 eslint-friendly-formatter
> `eslint-friendly-formatter` 这个模块是为了方便本地规范检查代码。记得运行 `cnpm i eslint-friendly-formatter -D` 安装上。

#### 6. 安装 babel-eslint
操作到这里关于 `eslint` 的基本配置就完成了，把项目跑起来，咿竟然有 `parset` 的报错，说最新的eslint解析有问题。查询资料是说需要 `babel-eslint`，按依赖安装的方式安装上。

#### 7. 安装 eslint-plugin-vue
`Vue.js` 的官方 `ESLint` 插件： `eslint-plugin-vue` ，这个插件允许 我们使用 `ESLint` 检测 `.vue` 文件的 `<template>` 和 `<script>`，有助于实时检测代码。 把这个插件安装上，最终的 `.eslintrc.js`: 

```javascript
module.exports = {
    "root": true,
    "env": {
        "browser": true,
        "es6": true
    },
    "extends": [
        "eslint:recommended",
        "plugin:vue/essential"
    ],
    "globals": {
        "Atomics": "readonly",
        "SharedArrayBuffer": "readonly"
    },
    "parserOptions": {
        "parser": 'babel-eslint',
        "ecmaVersion": 2018,
        "sourceType": "module"
    },
    "plugins": [
        "vue"
    ],
    "rules": {
    }
};
```
配置参数：
- root 为true根配置文件，否则会按照目录树向上搜索
- env 执行环境
- extends 对基础配置进行扩展，可以配置共享配置包  `eslint-config-airbnb`等。配置包需要安装。
- globals 指定全局变量，无视 no-undef 规则
- parserOptions 语法解析器选项
- plugins 插件 ，插件命名规范是 `eslint-plugin-` 为前缀
- rules 校验规则

#### 8. 配置小结
- 需要安装的依赖有
```
cnpm i eslint eslint-loader eslint-friendly-formatter babel-eslint  eslint-plugin-vue  -D
```
- webpack 配置
- .eslintrc.js  配置

配置完事，项目跑起来一堆的报错，来来来，💃开始自我折磨吧。经历磨练才能写出更优雅更规范的代码。看着页面不多，花了几个小时来处理 `ESLint` 的警告和报错。😓


## 配置问题
#### Use the latest vue-eslint-parser

在[eslint-plugin-vue官网](https://eslint.vuejs.org/user-guide/#faq)看到关于这个问题的解释和解决方案:

`eslint-plugin-vue` 里的大多数规则都需要 `vue-eslint-parser` 来解析 `template` 的AST， 然而 `babel-eslint` 和 `vue-eslint-parser` 在解析上有冲突，解决办法是把 `"parser": "babel-eslint",` 移入到 `parserOptions` 内。

![](https://user-gold-cdn.xitu.io/2020/1/9/16f883f2c8ba9c24?w=1000&h=593&f=jpeg&s=163247)

```javascript
- "parser": "babel-eslint",
+ "parser": "vue-eslint-parser",
  "parserOptions": {
+     "parser": "babel-eslint",
      "sourceType": "module"
  }
```

## 疑问
这次配置ESLint，留下了一些疑惑，得空扒源码吧。

- `eslint` 是如何检查代码的？
- `eslint-loader` 都处理了什么？
- 为什么需要 `eslint-friendly-formatter`？
- 为什么用 `babel-eslint`，它是怎样工作的？
- 为什么需要 `eslint-plugin-vue`？

#### 1. `ESLint` 是如何检查代码的
出于对这个问题的疑惑，去查了一些资料，了解了 [JS Linter 进化史](https://zhuanlan.zhihu.com/p/34656263)。`ESLint`  的核心诉求是将代码解析与代码检查逻辑分开，支持自定义规则，每个规则是一个插件，超强的开发能力让ESLint变的更好用。

引用JS Linter 进化史的一段话：
>  ESLint 核心部分专注于配置生成、规则管理、上下文维护、遍历 AST、报告产出等主流程。目前 ESLint 支持自定义的解析器、规则插件、预编译插件、结果报告，它更像是一个平台，对核心的流程设定约束，并开放各方面的能力，从而适应复杂多变的实际场景。

#### 2. 为什么需要 `eslint-loader`
`eslint-loader` 的作用是让 `webpack` 可以处理eslint。

#### 3. 为什么需要 `eslint-friendly-formatter`
看 `eslint-friendly-formatter`介绍的时候，该插件的作者说了下面这段话：

![](https://user-gold-cdn.xitu.io/2020/1/10/16f8e09c6180cfb3?w=1514&h=258&f=jpeg&s=80891)

大概是讲，他使用eslint的时候，发现eslint的报告提示在命令行中不友好，他不能在命令行中直接点击跳转到报错的代码处，没有记录报错的行列，对应的报错提示相关文档也不方便直接查看。于是他写了这个插件来处理以上问题。

现在这个插件被 eslint 官方推荐了。

#### 4. 为什么用 `babel-eslint`，它是怎样工作的
通常情况下 `eslint` 使用默认的解析器 `babel`。但是 `eslint` 本身不能检测实验阶段的语法，`ESNext` 的很多实验阶段的语法浏览器厂商支持了，开发者也在用。刚好 `eslint` 允许使用自定义解析器，所以就有了 `babel-eslint` 插件来处理这个事。

`babel-eslint` 导出了 `eslint` 可以使用的AST。

> AST 是 Abstract Syntax Tree（抽象语法树）

#### 5. 为什么需要 `eslint-plugin-vue`
`Vue` 单文件组件不是普通的 `Javascript`，不能使用默认的解析器。所以不得不有了 `eslint-plugin-vue` 用于对 `template` 和 `<script>` 进行解析生成特定的增强的 `AST`。

可以对比着 [vue风格指南](https://cn.vuejs.org/v2/style-guide/)看。目前我们使用的 `优先级A：必要的` 的规范，即 `plugin:vue/essential` 。`plugin:vue/essential` 的具体规则可以访问[eslint-plugin-vue](https://eslint.vuejs.org/rules/)官网。

## 参考

[JS Linter 进化史](https://zhuanlan.zhihu.com/p/34656263)

## 最后

很多学习 Vue 的小伙伴知识碎片化严重，我想整理出系统化的一套关于Vue的学习系列博客。在自我成长的道路上，也希望能够帮助更多人进步。

不看后悔，Vue进阶系列

- [【vue-进阶】vue-router源码分析](https://juejin.im/post/6844904012630720526)
- [【vue-进阶】深入理解Vuex](https://juejin.im/post/6844904033602256910)
- [【vue-进阶】你不知道的 vue-devtools](https://juejin.im/post/6844904036152377351)
- [【vue-进阶】vue 和 echats 结合的春天，vue插件vechart](https://juejin.im/post/6844904007605960711)
- [【vue-进阶】优雅地用Vue生成动态表单](https://juejin.im/editor/posts/5ddb82b1f265da7e243d0a72)
- [【vue-进阶】 Vue2.0 项目配置 ESLint](https://juejin.im/post/6844904041063907341)
- [【vue进阶】vue2.0 项目配置 Mocha 单元测试](https://juejin.im/post/6844904051839090702)

都看到这里了，赶紧关注 `MiaoMiao`、点赞、评论留言啦～







