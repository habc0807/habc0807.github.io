---
title: 【vue系列】优雅地用 vue 生成动态表单（一）
date: 2019-11-25
tags: 
---

开需求会了，产品说这次需求的表单比较多，目前有19个，后期的表单可能会有增加、修改。我作为这次的前端开发负责人，看到这样的需求，心里知道要这样搞不得把前端累死，首先表单居多，还要变更，以后维护起来也让人心力憔悴。

于是我提议做动态表单，做一个表单的配置系统，在系统里配置表单类型、表单得字段、以及对表单得管理。后来重新评审了需求，系统部分由后端自行开发，前端要处理的部分是动态生成表单，展现提交的表单，以及对表单的处理情况。


<!--more-->

### 需求背景
开需求会了，产品说这次需求的表单比较多，目前有19个，后期的表单可能会有增加、修改。我作为这次的前端开发负责人，看到这样的需求，心里知道要这样搞不得把前端累死，首先表单居多，还要变更，以后维护起来也让人心力憔悴。

于是我提议做动态表单，做一个表单的配置系统，在系统里配置表单类型、表单得字段、以及对表单得管理。后来重新评审了需求，系统部分由后端自行开发，前端要处理的部分是动态生成表单，展现提交的表单，以及对表单的处理情况。

### 数据接口设计

表单类型的接口就不用说了，这个比较简单。我跟后端约定了一个预备创建工单接口，这个接口是后端告知前端，需要生成一个什么样的表单。

##### 预备创建表单接口（其中字段解释说明）：
- `id` 表单字段的id
- `name` 表单字段的名称（存数据库的字段名）
- `type` 表单字段的类型（select_item下拉列表、string单行文本、multiple多行文本、integer单行数字、images上传图片）
- `title` 表单字段的中文名（动态表单的字段名称）
- `prompt_msg` 表单字段的placeholder文案
- `selectObj` type类型为select_item的时候，selectObj有值，默认为null

```javascript
{
	"code": 0,
	"msg": "success",
	"data": {
		"list": [{
			"id": 10,
			"name": "check_type",
			"type": "select_item",
			"title": "审核类型",
			"prompt_msg": "请填写审核类型",
			"selectObj": [{
				"id": 1,
				"item": "预审核"
			}, {
				"id": 2,
				"item": "患者审核"
			}],
			"val": null,
			"rank": 0
		}, {
			"id": 16,
			"name": "bank_branch_info",
			"type": "string",
			"title": "支行信息",
			"prompt_msg": "请填写支行信息",
			"selectObj": null,
			"val": null,
			"rank": 0
		},  {
			"id": 19,
			"name": "project_content",
			"type": "multiple",
			"title": "项目内容",
			"prompt_msg": "请填写项目内容",
			"selectObj": null,
			"val": null,
			"rank": 0
		}, {
			"id": 22,
			"name": "project_extension_time",
			"type": "integer",
			"title": "项目延长时间",
			"prompt_msg": "请填写项目延长时间",
			"selectObj": null,
			"val": null,
			"rank": 0
		}, {
			"id": 24,
			"name": "images",
			"type": "images",
			"title": "图片",
			"prompt_msg": "请上传图片",
			"selectObj": null,
			"val": null,
			"rank": 0
		}]
	}
}
```

### 通过Vue动态组件渲染表单
现在预备创建表单接口文档有了，该怎么渲染动态表单呢？动态表单的元素类型有5类，按照这个类别创建五个元素组件。

##### 1. 上传图片组件
上传图片组件这里使用了 `Uploader` 组件。

```html
<template>
    <div class="default images">
        <div class="lable">{{ item.title }}</div>
        <div v-if="item.val === null" class="content">
            <Uploader 
                :max-num="8"
                :user-imgs="project_image"
                @change="onUploadProject"
            />
        </div>
        <div v-else class="img-wrap">
            <img v-for="(it, idx) in item.val" :src="it" :key="idx" @click="preview(idx, item.val)">
        </div>
    </div>
</template>
```

##### 2. 多行输入框组件
默认多行输入框为3行

```html
<template>
    <div v-if="item"  class="default multiple">
        <div class="lable">{{ item.title }}</div>
        <template>
            <textarea
                rows="3" 
                :placeholder="item.prompt_msg" 
                v-model="value" 
                :value="it.item">
        </template>
    </div>
</template>
```

##### 3. 下拉选择框组件
使用了element-ui的 `el-select`
```html
<template>
    <div v-if="item" class="default select_item">
        <div class="lable select-lable">{{ item.title }}</div>
        <div class="content">
            <el-select
                v-model="value" 
                placeholder="请选择类型" 
                class="el-select-wrap" 
                size="mini"
                @change="onChangeFirstValue"
            >
                <el-option
                    v-for="it in item.selectObj"
                    :key="it.id"
                    :label="it.item"
                    :value="it.item">
                </el-option>
            </el-select>
        </div>
    </div>
</template>
```

其它两个数字单行输入框组件、文本单输入框组件跟多行输入框组件类似。

组件都创建好了，为了方便统一管理这些自定义组件。将组件们引入再导出，通过export default复合的形式。

```javascript
// 单行文本输入框组件
export { default as String } from './string.vue'  

// 单行数字输入框组件
export { default as Integer } from './integer.vue' 

// 多行文本输入框组件
export { default as Multiple } from './multiple.vue' 

// 下拉列表选择器组件
export { default as Select_item } from './select_item.vue' 

// 上传图片组件
export { default as Images } from './images.vue' 
```

再动态表单页面统一引入，以Vue动态组件的形式进行渲染，`is`属性为动态组件名。
```javascript
<template>
    <div class="g-container">
        <component 
            v-for="(item, number) in freedomConfig" 
            :key="item.name"
            :is="item.type" 
            :item="item" 
            :number="number" 
            @changeComponent="changeComponentHandle"
        ></component>
    </div>
</template>

<script>
    import * as itemElements from '../../components/itemElement'
    
    export default {
        components: itemElements,
    }
</script>
```
上面完成后，动态表单展现出来了。表单是动态生成的，如何进行表单验证，和表单数据的汇总呢？

### 表单数据汇总

再动态渲染组件的时候，传入了 `number` 参数，这个参数用来标识当前组件位于动态表单的第几个，方便后期填入数据后，进行数据保存。

默认value属性值为空，对value进行监听，当value变动的时
候进行emit，告诉父组件数据变更了，请保存。

```
data() {
    return {
        value: ''
    }
},
watch: {
    value(v, o) {
        this.throttleHandle(() => {
            this.$emit('changeComponent', {
                number: this.number,
                value: this.$data.value
            })
        })
    }
},
```
但是数据保存到哪里？怎么保存呢？
让后端给一个表单全部字段的接口，取到数据存到data中，每次数据更新就去查找是否存在这个字段，有的话就赋值保存起来。后面提交的时候，就提交这个对象。


### 表单校验
提交的时候，希望用户能够把表单填完再调用提交接口，需要前端校验是否填完没有的话，就给响应的toast轻提示，阻止表单提交。

```javascript
this.checkFrom(freedomConfig, preWordorderData).then(canSubmit => {
    canSubmit && postSubmitWorkorder(preWordorderData).then(res => {
        if (res.code === 0) {
            showLoading()
            this.$router.push(`/detail/${res.data.id}`)
        }
    })
})
```

`checkFrom`为我们的校验方法，循环遍历预创建表单，从data里查看该字段是否有值，没有的话就给于toast提示。并返回一个`promise.resolve(false)`。如果都校验通过返回 `resolve(true)`。这样就可以使checkFrom成为一个异步函数。

> 其中需要注意的是下拉框选择后的值为大于0的数字、上传图片的属性值是数组。

一个动态表单的创建、校验、数据整合就完成了。很多时候需要写大量代码的场景思路上很简单，反倒是抽象一个组件需要考虑的更多。
