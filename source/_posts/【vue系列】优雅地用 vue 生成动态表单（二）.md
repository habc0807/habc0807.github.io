---
title: 【vue系列】优雅地用 vue 生成动态表单（二）
date: 2020-07-06
tags: 
---

工作台模块，新需求要做请假审批，开需求评审的时候，了解到请假有：年假 、病假、调休、事假 、婚嫁、丧假期、产假、陪产假等8种类型。小小第一个想法就是不想写多个表单页面，也不想在一个表单页面中写很多的判断类型逻辑。于是就试探性询问能否由后端配合做动态表单，毕竟之前做一次。但是后端和产品都一致认为这次表单的变更不大，没必要再开发一个表单的配置系统。不开心呐，需求评审完成之后小小开始调研实现方案和技术选型。

<!--more-->

## 需求背景

工作台模块，新需求要做请假审批，开需求评审的时候，了解到请假有：年假 、病假、调休、事假 、婚嫁、丧假期、产假、陪产假等8种类型。小小第一个想法就是不想写多个表单页面，也不想在一个表单页面中写很多的判断类型逻辑。于是就试探性询问能否由后端配合做动态表单，毕竟之前做一次。但是后端和产品都一致认为这次表单的变更不大，没必要再开发一个表单的配置系统。不开心呐，需求评审完成之后小小开始调研实现方案和技术选型。

## 寻找技术方案和技术选型

思考着能否由前端全权动态表单操控处理？这样就可以减轻后端的开发时长和压力，当然这次需求就比较重前端了。那就来个动态表单，根据不同的请假类型，自动生成表单，虽然在首次开发需要更多的思考和设计，但是后期方便维护啊。小小也是对需求、对业务、对代码有要求的人。哼～

那么问题来了，如果采用动态表单的话，小程序并没有动态组件可以方便处理，于是小小考虑选用小程序内嵌h5的方式，h5采用vue框架。技术方案和技术选型就确定好了。

![](https://user-gold-cdn.xitu.io/2020/6/19/172cbbb59533bfcd?w=1694&h=1414&f=jpeg&s=195653)

针对8种请假类型：年假 、病假、调休、事假 、婚嫁、丧假期、产假、陪产假的表单，提炼出需要用到的组件，以及需要生成动态表单的配置表。

## 动态组件那些事

![](https://user-gold-cdn.xitu.io/2020/6/19/172cbbded186770c?w=1596&h=516&f=jpeg&s=151734)

这份组件图，主要给我们的内部开发人员看，一目了然地知道怎么去做代码的组织和构建。下面总结了动态表单由哪些组件组成，各个组件之间的关系和依赖，以及组件依赖的三方和框架。

总结需要开发的组件有：
- 日期选择器（依赖 mint-ui 的 datetime-picker）
- 带上、下午的日期选择器（vue-halfday-datepicker插件，针对底层使用的 mint-ui 的picker二次封装的npm）
- 请假时长计算器（ leave-days-calculator插件，依赖开始和结束时间）
- 结束时间自动计算器（依赖开始和请假时长，下期开发～）
- 多行输入框组件
- 上传图片组件（针对内部上传组件二次封装）

其中 `请假时长计算器` 在笔者的[造轮子 | 如何开发一个请假时长计算器？](https://juejin.im/post/6844904182915268622)这篇文章中查看实现细节和详情，`带上、下午的日期选择器` 可以查看笔者[vue-halfday-datepicker插件的封装过程](https://juejin.im/post/6844904191270322183)这篇文章。

创建组件的细节请参考[【vue系列】优雅地用 vue 生成动态表单（一）](https://juejin.im/post/6844904004309237773)，这里就不再赘述了。

## 动态表单配置表

动态组件开发完成，动态表单的配置表就能出来了。
```
// 请假类型的map表，同后端一一对应
export const typeObj = {
    1: '年假',
    2: '病假',
    3: '调休',
    4: '事假',
    5: '婚假',
    6: '丧假',
    7: '产假',
    8: '陪产假',
    9: '补假',
}

/**
 * 所有请假表单的动态配置config
 */
export const setFormConfig = {
    // 年假
    1: [
        {
            "name": "start_time",
            "type": "halfdaydate",
            "title": "开始时间",
            "prompt_msg": "请选择",
            "val": null,
        }, 
        {
            "name": "end_time",
            "type": "halfdaydate",
            "title": "结束时间",
            "prompt_msg": "请选择",
            "val": null,
        }, 
        {
            "name": "leave_length",
            "type": "leavelength",
            "title": "请假时长",
            "prompt_msg": "请选择",
            "val": null,
        }, 
        {
            "name": "notes",
            "type": "multiple",
            "title": "请假事由（必填）：",
            "prompt_msg": "请输入请假事由",
            "val": null,
        }
    ],

    // 病假
    2: [
        {
            "name": "start_time",
            "type": "halfdaydate",
            "title": "开始时间",
            "prompt_msg": "请选择",
            "val": null,
        }, 
        {
            "name": "end_time",
            "type": "halfdaydate",
            "title": "结束时间",
            "prompt_msg": "请选择",
            "val": null,
        }, 
        {
            "name": "leave_length",
            "type": "leavelength",
            "title": "请假时长",
            "prompt_msg": "请选择",
            "val": null,
        }, 
        {
            "name": "notes",
            "type": "multiple",
            "title": "请假事由（必填）：",
            "prompt_msg": "请输入请假事由",
            "val": null,
        }, 
        {
            "name": "imgs",
            "type": "images",
            "title": "上传图片：",
            "prompt_msg": "请上传图片",
            "val": null,
        }
    ],

    // 调休
    3: [
        {
            "name": "start_time",
            "type": "halfdaydate",
            "title": "开始时间",
            "prompt_msg": "请选择",
            "val": null,
        }, 
        {
            "name": "end_time",
            "type": "halfdaydate",
            "title": "结束时间",
            "prompt_msg": "请选择",
            "val": null,
        }, 
        {
            "name": "leave_length",
            "type": "leavelength",
            "title": "请假时长",
            "prompt_msg": "请选择",
            "val": null,
        }, 
        {
            "name": "overtime_compensation",
            "type": "date",
            "title": "调休替代日期",
            "prompt_msg": "请选择",
            "val": null,
        }, 
        {
            "name": "notes",
            "type": "multiple",
            "title": "请假事由（必填）：",
            "prompt_msg": "请输入请假事由",
            "val": null,
        }, 
        {
            "name": "imgs",
            "type": "images",
            "title": "上传图片：",
            "prompt_msg": "请上传图片",
            "val": null,
        }
    ],

    // 事假
    4: [
        {
            "name": "start_time",
            "type": "halfdaydate",
            "title": "开始时间",
            "prompt_msg": "请选择",
            "val": null,
        }, 
        {
            "name": "end_time",
            "type": "halfdaydate",
            "title": "结束时间",
            "prompt_msg": "请选择",
            "val": null,
        }, 
        {
            "name": "leave_length",
            "type": "leavelength",
            "title": "请假时长",
            "prompt_msg": "请选择",
            "val": null,
        }, 
        {
            "name": "notes",
            "type": "multiple",
            "title": "请假事由（必填）：",
            "prompt_msg": "请输入请假事由",
            "val": null,
        }
    ],

    // 婚假
    5: […],
    // 丧假
    6: […]
    // 产假
    7: […],
    // 陪产假
    8: […]
]
```
## 动态组件统一管理

动态组件统一导入导出，方便管理，后期增、删、改这一个文件就够了。

```javascript
// 多行文本输入框组件
export { default as Multiple } from './multiple.vue' 

// 上传图片组件
export { default as Images } from './images.vue' 

// 日期选择器组件(整天)
export { default as Date } from './date_picker.vue' 

// 半天日期选择器组件（上/下午）
export { default as Halfdaydate } from './halfdaydate_picker/index.vue' 

// 请假时长计算器
export { default as leavelength } from './leaveLength/index.vue'
```
在动态表单页面，统一动态引入，通过 `component` 动态组件的方式渲染，还不了解vue动态组件的戳这里 [动态组件 & 异步组件](https://cn.vuejs.org/v2/guide/components-dynamic-async.html#%E5%9C%A8%E5%8A%A8%E6%80%81%E7%BB%84%E4%BB%B6%E4%B8%8A%E4%BD%BF%E7%94%A8-keep-alive)。

```javascript
<template>
    <div class="g-context">
        <section class="form-wrap">
	    // …
            <component 
                v-for="(item, number) in freedomConfig" 
                :key="item.name"
                :is="item.type" 
                :item="item" 
                :preFormData="preFormData"
                :number="number"
                @changeComponent="changeComponentHandle"
            ></component>
        </section>
         // ….
    </div>
</template>

<script>
// …
import * as itemElements from '../../components/itemElement'

export default {
    // …
    props: ['type'], // type 请假类型
    components: itemElements,
}
```
通过请假类型和动态表单的配置表，结合动态组件，将准确的显示各个请假类型需要的表单，表单数据汇总和校验的方式跟[【vue系列】优雅地用 vue 生成动态表单（一）](https://juejin.im/post/6844904004309237773)几乎一样，可以去这篇查看细节处理。

其中关于 `请假时长计算器` 的处理，需要根据，开始时间和结束时间动态计算，这里再具体说说。
动态组件生成的属性 `:preFormData="preFormData”`，`preFormData` 是整个表单的数据汇总，透出给各个组件，在 `请假时长计算器`组件中 `watch` 该属性。

```javascript
watch: {
    preFormData: {
        handler(v, o) {
            this.throttleHandle(() => {
                const { start_time, end_time} = v

                // 公共方法计算出请假时长
                let leaveDays = leaveDaysCalculator(start_time, end_time)
                this.value = leaveDays

                this.$emit('changeComponent', {
                    number: this.number,
                    value: leaveDays
                })
            })
        },
        deep: true,
        immediate: true
    }
},
```
需要注意下 `watch` 的用法，这使用了 `deep、immediate` 属性，其中 `deep: true`代表深度监听，`immediate: true` 代表该回调将会在侦听开始之后被立即调用， 还不清楚用法的戳 [watch的用法](https://cn.vuejs.org/v2/api/#watch)。

## 小结

小小的处理方法不一定是最优的，或者可以采用双向数据绑定的方案。总体上来说，小小想做的事情想把如何创建动态表单的思路，以及设计方案表述出来做个记录，大致思路是封装一层动态化渲染机制，基于某个JSON 来做动态组件的渲染。如果你有好思路或者疑惑，欢迎在评论区留言与我讨论。

目前业界已经有很完善的解决方案，比如阿里表单统一解决方案 [Formily](https://formilyjs.org/#/bdCRC5/dzUZU8il)（react实现），大家可以自行学习了解，看了别人方案，发现自己还有更多需要学习的地方，加油～
