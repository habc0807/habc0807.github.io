---
title: 【vue系列】vue 和 echats 结合的春天，vue插件vechart
date: 2019-11-13
tags: 
---

最近项目需要用到echarts，但是vue和vechart的使用很奇怪，需要操作dom，代码写起来也不优雅。于是就将传统echats.js和vue结合成vue插件，通过vue插件引入、注册的方式使用不要太爽。目前已将该插件上传到npm上了。需要的快来踩[vechart](https://www.npmjs.com/package/vechart)。

<!--more-->

最近项目需要用到echarts，但是vue和vechart的使用很奇怪，需要操作dom，代码写起来也不优雅。于是就将传统echats.js和vue结合成vue插件，通过vue插件引入、注册的方式使用不要太爽。目前已将该插件上传到npm上了。需要的快来踩[vechart](https://www.npmjs.com/package/vechart)。

### vechart 

`vechart` 插件是将传统 `echart.js` 封装成Vue组件和指令使用。

### 插件的安装

#### NPM

```javascript 
npm i vechart
```

#### CNPM

```javascript
cnpm i vechart
```

### 引入插件

```javascript
import Vue from 'vue';
import Vechart from 'vechart';

Vue.use(Vechart);
```

### 基本用法

#### vechart组件的用法

```javascript
<vechart :options="options1" :styles="echartStyle"></vechart>
```

```javascript
options1: {
    series: {
        type: 'pie',
        data: [{
                name: 'A',
                value: 1212
            },
            {
                name: 'B',
                value: 2323
            },
            {
                name: 'C',
                value: 1919
            }
        ]
    }
},
```

#### vechart指令的用法

```javascript
<div v-echarts="options" :styles="echartStyle" ></div>
```


### API

| 参数 | 说明 | 类型 | 默认值 |
| ------ | ------ | ------ | ------ |
| options | 中等文本 | Object | - |
| styles | 默认样式 | Oject | `{ width: '100px',height: '100px'}` |


### DEOM地址
[demo演示地址](https://habc0807.github.io/vechart/dist/index.html)

### Author

[habc0807](https://github.com/habc0807)

### NPM

[npm](https://www.npmjs.com/package/vechart)