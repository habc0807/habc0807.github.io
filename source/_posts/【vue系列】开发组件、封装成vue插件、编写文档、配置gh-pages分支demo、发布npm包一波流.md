---
title: 【vue系列】开发组件、封装成vue插件、编写文档、配置gh-pages分支demo、发布npm包一波流
date: 2020-06-15
tags: 
---

你是如何界定要不要造轮子的？于我而言，将需要多处调用的重复代码封装成组件、插件，把繁琐的操作流程开发成提效工具，后续能简化工作量，提高工作效率，就值得造轮子。为业务封组件、封插件，为提效造工具，做有价值的事，才能实现自我价值。

<!--more-->

你是如何界定要不要造轮子的？于我而言，将需要多处调用的重复代码封装成组件、插件，把繁琐的操作流程开发成提效工具，后续能简化工作量，提高工作效率，就值得造轮子。为业务封组件、封插件，为提效造工具，做有价值的事，才能实现自我价值。

我们先来看看，这次造的什么轮子，针对我们请假审批的需求，请假的时候选择日期选择器是带上下午的，这在移动端并不多见，通常是日历的形式，我们的产品要求picker滚动器的形式。找了半天都没找到合适的。

![](https://user-gold-cdn.xitu.io/2020/6/15/172b5cf1436e4626?w=696&h=536&f=png&s=83431)
就是这种日期选择器👆。

喏，就是这样效果，没得办法，只能自己上手造一个。以下就是我造轮子的过程。从先开发带半天的日期选择器组件，满足需求使用；业务里多处使用，封装成vue插件；封装完成发现包太大，进而对插件优化；一个精致的vue插件封装完成后，编写插件的使用文档，方便大家使用；然后上传到github，配置gh-pages，展现在线demo；最后发布到npm，一个三方插件就完成了。✌️✌️✌️

我们先来看看封装好的带上、下的日期选择器，怎么用？👇

# vue-halfday-datepicker 

> 一个简单的基于vue的带上、下午的日期选择器

## Demo

[在线地址：https://habc0807.github.io/vue-halfday-datepicker/index.html](https://habc0807.github.io/vue-halfday-datepicker/index.html)

## 特点

- 移动端
- 带上、下午
- 日期选择器
- vue插件

## 安装

```
npm i vue-halfday-datepicker -D
cnpm i vue-halfday-datepicker -D
yarn add vue-halfday-datepicker -D
```

## 引入和注册 
在单页面应用的入口文件里引入和注册

```javascript
import VueHalfdayDatepicker from 'vue-halfday-datepicker'
Vue.use(VueHalfdayDatepicker)
```

## 使用 

``` vue
<template>
  <div id="app">
    <h2>一个基于vue的日期选择器Demo</h2>

    <div class="pickerSelect" @click="onOpenPicker">{{ pickerValue }}</div>
    <vue-halfday-datepicker :show="isShow" @onConfirm="onConfirmHandle" @onCancel="onCancelHandle"/>
  </div>
</template>

<script>
export default {
  name: 'app',
  data () {
    return {
      isShow: false,
      pickerValue: '测试一下：请选择'
    }
  },
  methods: {
    onOpenPicker(e) {
        this.isShow = true
    },
    onConfirmHandle(v) {
        this.value = v
        this.pickerValue = v
        this.isShow = false
    },
    onCancelHandle(e) {
        this.isShow = false
    }
  },
}
</script>
<style>
</style>
```

如果觉得插件还不错，就给个Star，助力下，你的支持，是我输出的动力～～～
[github: vue-halfday-datepicker](https://github.com/habc0807/vue-halfday-datepicker)

# 如何开发一个带上、下午的日期选择器组件

## 寻找解决方案

最初的时候，我想省事，找个三方直接用，在github上扒拉了半天，又扒了vue的UI组件库们，看了一圈，发现VUX的`datetime` 居然有这个组件。开森，赶紧试试，开始按需引入这个组件，通过VUX官网的[手动配置使用](https://doc.vux.li/zh-CN/install/manual-usage.html)，配置真繁琐，重点配完还不能用。当然我不会轻易放弃的，从头到尾按照配置再来一遍，依旧不好使。

![](https://user-gold-cdn.xitu.io/2020/6/12/172a7826dd444cba?w=678&h=309&f=png&s=147876)
## 二次封装picker

没办法，我暂时放弃了VUX。另辟捷径，日期选择器picker，不也是picker吗，为什么不基于picker的基础上做二次封装呢？

于是我找到了 `Mint UI`，使用它的 `picker` 做二次封装。不得不承认 [Mint UI 的快速上手文档](http://mint-ui.github.io/docs/#/zh-cn2/quickstart)写的很赞，配置起来很顺手。因为只需要使用一个组件，按需引入需要借助 `babel-plugin-component` ，以达到减小项目体积的目的。首先，安装 `npm install babel-plugin-component -D
`，然后将 `.babelrc` 修改为： 

```
{
  "presets": [
    ["es2015", { "modules": false }]
  ],
  "plugins": [["component", [
    {
      "libraryName": "mint-ui",
      "style": true
    }
  ]]]
}
```

## 日期选择器如何开发

当然最重要的是 picker 的 `slots` 四列数组的数据处理，以及 picker 改变的数据更新。先来看看 `slots` :

```javascript
slots: [
    {
        flex: 1,
        values: yearsArr,
        className: 'slot1',
        textAlign: 'center',
        defaultIndex: defalutYear,
    }, 
    {
        flex: 1,
        values: monthsArr,
        className: 'slot2',
        textAlign: 'center',
        defaultIndex: defalutMonth
    },
    {
        flex: 1,
        values: daysArr,
        className: 'slot3',
        textAlign: 'center',
        defaultIndex: defalutDay
    },
    {
        flex: 1,
        values: noonArr,
        className: 'slot4',
        textAlign: 'center',
        defaultIndex: 0,
    }
]
```

我们需要四列数组，分别是年份数组，当前年份对应的月份数组，当前月份对应的天数数组，以及一个上、下午数组。其中月份数组通常都是 1 ～ 12月，上、下午数组也很简单，年份数组我们默认指定前后10年就够用了，需要特殊计算的是天数数组。其实计算规则大致是是否是2月，若是通过是否是闰年计算出天数，若不是 4, 6, 9, 11月份为30天，其它月份为31天。嗯，就酱，来看代码👇。

```javascript
/**
 * 是否是闰年
 * @param {*} year 
 */
function isLeapYear (year) {
  return year % 100 !== 0 && year % 4 === 0 || year % 400 === 0
}

/**
 * 计算每年的每个月的天数
 * @param {*} year 
 * @param {*} month 
 */
function getMaxDay (year, month) {
  year = parseFloat(year)
  month = parseFloat(month)
  if (month === 2) {
    return isLeapYear(year) ? 29 : 28
  }

  return [4, 6, 9, 11].indexOf(month) >= 0 ? 30 : 31
}
```

每次picker更新，重新计算数组里的数据值，及时更新选出的日期。组件就开发完成，里面其实还有很多细节地方需要注意，比如默认显示，更新后的显示等，我只是说了说主流程。

# 如何封装一个vue插件 

组件开发完成了，我萌生了封插件的想法。首先这个组件需要引用`mint-ui`三方，按需引入 `picker`，其次，在项目根目录下添加 `.babelrc` 文件来配置按需加载。这一些列的操作，如果后续有需求或者其他小伙伴也要用它的话，就会觉得比较麻烦。我们这个多页面应用，就要把上述的流程再操作一遍。还不如封装一个 vue 插件，用着多方便。

从业务的价值出发，值得造火箭，如果能将以上流程封装成 vue 插件，后续只要引入使用就行了。配置流程，按需加载什么的都放到 vue 插件里，为业务造轮子，这就是我要做的事情。

## 项目初始化 

查封装vue插件的资料的时候，看到有人推荐 `webpack-simple`，我就试试了试，还蛮好用。通过 `vue init webpack-simple vue-插件名`，创建出已 `vue-插件名`命名的文件夹，项目的目录树如下。

```
.
├── README.md
├── dist
├── index.html
├── package.json
├── src
│   ├── App.vue
│   ├── assets
│   │   └── logo.png
│   ├── lib            
│   │   ├── VueHalfdayDatepicker.vue
│   │   └── index.js
│   ├── main.js
└── webpack.config.js
```
很简单，前端的猿猿们一眼就能看的懂吧，其中src为源码部分，lib是插件部分。

## 修改配置项

### 修改package.json

主要更改了 `name、main、author、repository`，具体如下：

- name 指定插件名称
- main 插件最后的构建文件路径
- author 作者的基本信息
- repository 仓库的基本信息

```json
{
  "name": "vue-halfday-datepicker",
  "description": "A date picker",
  "main": "dist/vue-halfday-datepicker.js",
  "version": "1.0.1",
  "author": "habc0807 <gao0807@foxmail.com>",
  "license": "MIT",
  "private": false,
  "scripts": {
    "dev": "cross-env NODE_ENV=development webpack-dev-server --open --hot",
    "build": "cross-env NODE_ENV=production webpack --progress --hide-modules"
  },
  "repository": {
    "type": "git",
    "url": "git+https://github.com/habc0807/vue-halfday-datepicker.git"
  },
  "browserslist": [
    "> 1%",
    "last 2 versions",
    "not ie <= 8"
  ],
  "devDependencies": {
    "babel-core": "^6.26.0",
    "babel-loader": "^7.1.2",
    "babel-plugin-component": "^1.1.1",
    "babel-preset-env": "^1.6.0",
    "babel-preset-stage-3": "^6.24.1",
    "cross-env": "^5.0.5",
    "css-loader": "^0.28.7",
    "file-loader": "^1.1.4",
    "mint-ui": "^2.2.13",
    "vue": "^2.5.11",
    "vue-loader": "^13.0.5",
    "vue-template-compiler": "^2.4.4",
    "webpack": "^3.6.0",
    "webpack-bundle-analyzer": "^3.8.0",
    "webpack-bundle-tracker": "^1.0.0-alpha.1",
    "webpack-dev-server": "^2.9.1"
  },
  "bugs": {
    "url": "https://github.com/habc0807/vue-halfday-datepicker/issues"
  },
  "homepage": "https://github.com/habc0807/vvue-halfday-datepicker#readme"
}
```

### 修改 webpack.config.js

这里我们预留了页面的入口，和插件的入口，先以组件的形式使用，没有问题后，再将组件打包成插件，再在页面中查看展现效果，或者修复插件中的小bug。

```
entry: './src/main.js', // 页面的入口
  // entry: './src/lib/index.js', // 插件的入口
output: {
    path: path.resolve(__dirname, './dist'),
    publicPath: '/dist/',
    filename: 'build.js', // 页面的生成文件
    // filename: 'vue-halfday-datepicker.js', // 插件的生成文件
    libraryTarget: 'umd',
    umdNamedDefine: true
},
```

> 后续在优化插件的过程中，添加一些其它配置，最终的 `webpack.config.js` 在插件优化部分中展示。

## 封装插件 

`VueHalfdayDatepicker.vue` 插件的具体内容如下，其中需要注意的地方，这个插件的封装使用了 `mint-ui` 的 `mt-picker`。我主要要做的事情就是 picker 的四列数组 `slots`，第一个模块介绍过，这里就不赘述了。


```
<template>
    <div v-show="show" class="half-day-date-picker">
        <div class="bg"></div>
        <div class="select-date-container">
            <section class="picker-wrap">
                <div class="picker-toolbar">
                    <div class="btn cancel" @click="onCancel">取消</div>
                    <div class="btn confirm" @click="onConfirm">确定</div>
                </div>

                <mt-picker 
                    :valueKey="''"
                    :defaultIndex=10
                    :visibleItemCount=9
                    :slots="slots" 
                    @change="onValuesChange"
                ></mt-picker>
            </section>
        </div>
    </div>
</template>

<script>
import { Picker } from 'mint-ui'
import 'mint-ui/lib/style.css'

import {
    yearsArr,
    monthsArr,
    daysArr,
    noonArr,
    defalutYear,
    defalutMonth,
    defalutDay
} from '../tools/makeData.js'

export default {
    name: 'vue-halfday-datepicker',
    props: {
        show: {
            type: Boolean,
            required: true,
            default: () => null,
        }
    },
    components: {
        [Picker.name]: Picker,
    },
    data: function() {
        return {
            pickerValue: '',
            slots: [
                {
                    flex: 1,
                    values: yearsArr,
                    className: 'slot1',
                    textAlign: 'center',
                    defaultIndex: defalutYear,
                }, 
                {
                    flex: 1,
                    values: monthsArr,
                    className: 'slot2',
                    textAlign: 'center',
                    defaultIndex: defalutMonth
                },
                {
                    flex: 1,
                    values: daysArr,
                    className: 'slot3',
                    textAlign: 'center',
                    defaultIndex: defalutDay
                },
                {
                    flex: 1,
                    values: noonArr,
                    className: 'slot4',
                    textAlign: 'center',
                    defaultIndex: 0,
                }
            ],
        }
    },
    methods: {
        onOpenPicker(e) {
            this.$emit('onOpenPicker')
        },
        onConfirm(e) {
            this.$emit('onConfirm', this.pickerValue)
        },
        onCancel(e) {
            this.$emit('onCancel')
        },
        /** picker改变的处理 */
        onValuesChange(picker, values) {
            // 按照时间组合字符串
            const pickerValue = values[0] + '-' + values[1] + '-' + values[2] + ' ' + values[3]
            this.pickerValue = pickerValue
            this.$emit('onValuesChange', pickerValue)
        },
    },
}
</script>
<style scoped>
	// …
</style>
```
将此组件封装成插件，vue插件的格式要通过 `install` 来挂载组件，使用 `Vue.component` 来注册一个插件，在外部 `use` 一个插件， `index.js` 的完整内容如下。

```javascript
import VueHalfdayDatepicker from './VueHalfdayDatepicker.vue'

const datePicker = {
  install: function(Vue) {
    Vue.component(VueHalfdayDatepicker.name, VueHalfdayDatepicker)
  }
}

// 这里的判断很重要
if (typeof window !== 'undefined' && window.Vue) { 
  window.Vue.use(datePicker) 
}
export default datePicker
```

## 测试插件

如何测试一个插件开发是否成功呢，我没有直接去构建插件，而是以直接引用本地插件的方式，在 `main.js` 中引入插件

```javascript
import VueHalfdayDatepicker from '../dist/vue-halfday-datepicker.js'
Vue.use(VueHalfdayDatepicker)
```

在 `App.vue` 中使用组件

```vue
<template>
  <div id="app">
    <h2>一个基于vue的日期选择器Demo</h2>

    <div class="pickerSelect" @click="onOpenPicker">{{ pickerValue }}</div>
    <vue-halfday-datepicker :show="isShow" @onConfirm="onConfirmHandle" @onCancel="onCancelHandle"/>
  </div>
</template>

<script>
export default {
  name: 'app',
  data () {
    return {
      isShow: false,
      pickerValue: '测试一下：请选择'
    }
  },
  methods: {
    onOpenPicker(e) {
        this.isShow = true
    },
    onConfirmHandle(v) {
        this.value = v
        this.pickerValue = v
        this.isShow = false
    },
    onCancelHandle(e) {
        this.isShow = false
    }
  },
}
</script>

<style>
	// …
</style>
```
执行 `npm run dev`，紧接着在页面中查看插件，修复插件bug等，在页面中查看没有问题了。接着修改 `webpack.config.js` 的入口文件，和生成文件名称。

```
  // entry: './src/main.js', // 页面的入口
  entry: './src/lib/index.js', // 插件的入口
  output: {
    path: path.resolve(__dirname, './dist'),
    publicPath: '/dist/',
    // filename: 'build.js', // 页面的生成文件
    filename: 'vue-halfday-datepicker.js', // 插件的生成文件
    libraryTarget: 'umd',
    umdNamedDefine: true
},
```
执行 `npm run build`，在目录 `dist`文件夹下 构建生成插件 `vue-halfday-datepicker.js`。然后再配置回页面的构建，查看插件效果。

## 插件优化

一个vue插件基本上就开发好了，我们想看看构建生成的插件各个模块的大小，有哪些可优化的空间，可以借用 `webpack-bundle-analyzer` 可视化工具查看。 在项目根目录下使用命令行 `cnpm i webpack-bundle-analyzer`，去 `webpack.config.js` 中配置 


```
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;

plugins: [
    new BundleAnalyzerPlugin()
]
```

执行 `npm run dev`，看到打包后的插件的各个模块的大小。我们发现插件把 `vue` 打包进去了，是因为 `mint-ui` 引入了 `vue`。插件大小，Parsed size: 161.33kb。

![](https://user-gold-cdn.xitu.io/2020/6/12/172a78d62d4af8b6?w=1000&h=532&f=jpeg&s=79418)

我们通过配置 `externals`来防止将某些 import 的包(package)打包到 bundle 中，而是在运行时(runtime)再去从外部获取这些扩展依赖。通过设置  `externals` 防止把 `vue` 打包进去，打包后插件包小了许多，Parsed size: 64.46kb。

```
externals: {
    vue: 'Vue'
},
```

![](https://user-gold-cdn.xitu.io/2020/6/12/172a78dd5afdf605?w=1000&h=468&f=jpeg&s=65531)

# 推到github上，在gh-pages上配置demo

部署好的[DEMO地址](https://habc0807.github.io/vue-halfday-datepicker/index.html)，快来看看效果。

猿猿们git操作都很溜，我就不赘述github的基本操作了，当我们把插件库推到github之后，默认是推到master分支上，然后新建分支 gh-pages，将master 分支上的内容merge 到gh-pages分支上，并提交到远程。需要注意的点是 gh-pages 分支是github提供的静态网站部署器，默认会读取项目根目录下的 index.html，我的index.html:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width,initial-scale=1,user-scalable=no,viewport-fit=cover">
    <title>vue-halfday-datepicker</title>
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
  </head>
  <body>
    <div id="app"></div>
    <script src="./dist/build.js"></script>
  </body>
</html>
```

其中 `<script src="./dist/build.js"></script>` 里的 dist/build.js 是页面的构建文件，注意 `.gitignore` 中不要配置 `dist` 文件，不然 builds.js 文件就读取不到了。部署好的[DEMO地址](https://habc0807.github.io/vue-halfday-datepicker/index.html)，快来看看效果。

# 发布插件，发布一个npm包

娃哈哈，到最后一步了，我们来看看怎么发布。熟悉该流程的猿猿们，就可以忽悠该模块了，如果本文，能帮助到你，可以为我助力点赞。【不害臊的要关注，要赞，嘻嘻～】

## npm官网注册

去[npm官网](https://www.npmjs.com/)注册，注册的时候，要记住自己的用户名、密码和邮箱，后面我们还要用。

## npm包发布
回到我们vue插件的仓库根目录下，打开命令行，输入命令 `npm login` ，按照提示输入你的用户名和密码，再执行 `npm publish` 进行发布。发布成功，命令里会显示 npm的包名和版本号。

![](https://user-gold-cdn.xitu.io/2020/6/12/172a790ccf97c486?w=1710&h=86&f=jpeg&s=27897)

> 发包过程中如果遇到包同名，或者版本号没更新的小问题，可自行搜索，很容易解决的。

# 总结

希望能帮助大家，了解编写插件的过程和思路，你会发现在业务中遨游，时而做些业务插件更能体现做业务的价值。快去试试吧，你也可以写出各式各样的小插件，欢迎在评论区跟我沟通交流。



