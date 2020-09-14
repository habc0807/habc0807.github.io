---
title: 【vue系列】封装公共弹窗组件的正确方式
date: 2020-03-30
tags: 
---

最近一个项目向Vue框架搭建的新项目迁移，但是项目中没有使用vue ui库，也还没有封装公用的弹窗组件。于是我就实现了一个简单的弹窗组件。

<!--more-->


最近一个项目向Vue框架搭建的新项目迁移，但是项目中没有使用vue ui库，也还没有封装公用的弹窗组件。于是我就实现了一个简单的弹窗组件。在开发的之前考虑到以下几点：

> 1. 组件标题，按钮文案，按钮个数、弹窗内容均可定制化;
>
> 2. 弹窗垂直水平居中 考虑实际在微信环境头部不可用，ios微信环境中底部返回按钮的空间占用;
>
> 3. 遮罩层和弹窗内容分离，点击遮罩层关闭弹窗;
> 
> 4. 多个弹窗同时出现时弹窗的z-index要不之前的要高;
>
> 5. 点击遮罩层关闭弹窗和处理弹窗底部的页面内容不可滚动.


其中包含了要实现的主要功能，以及要处理的问题。

## 实现的步骤

1. 完成页面结构和样式以及过渡动画
2. 定制弹窗标题、按钮和主题内容
3. 组件开关
4. z-index处理
5. 点击遮罩层关闭弹窗
6. 处理弹窗底部的页面内容不可滚动

#### 1. 完成页面结构和样式

先创建一个弹窗组件vue文件，实现基本的结构与样式。

```
<template>
    <div class="dialog">
        <div class="dialog-mark"></div>
        <transition name="dialog">
            <div class="dialog-sprite">
                <!-- 标题 -->
                <section v-if="title" class="header">临时标题</section>
    
                <!-- 弹窗的主题内容 -->
                <section class="dialog-body">
                    临时内容
                </section>
    
                <!-- 按钮 -->
                <section class="dialog-footer">
                    <div class="btn btn-confirm">确定</div>
                </section>
            </div>
        </transition>
    </div>
</template>

<script>
    export default {
        data(){
            return {}
        }
    }
</srcipt>


<style lang="less" scoped>
    // 弹窗动画
    .dialog-enter-active,
    .dialog-leave-active {
        transition: opacity .5s;
    }
    
    .dialog-enter,
    .dialog-leave-to {
        opacity: 0;
    }
    
    // 最外层 设置position定位 
    // 遮罩 设置背景层，z-index值要足够大确保能覆盖，高度 宽度设置满 做到全屏遮罩
    .dialog {
        position: fixed;
        top: 0;
        right: 0;
        width: 100%;
        height: 100%;
        // 内容层 z-index要比遮罩大，否则会被遮盖
        .dialog-mark {
            position: absolute;
            top: 0;
            height: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, .6);
        }
        .dialog-sprite {
            // 移动端使用felx布局
            position: absolute;
            top: 10%;
            left: 15%;
            right: 15%;
            bottom: 25%;
            display: flex;
            flex-direction: column;
            max-height: 75%;
            min-height: 180px;
            overflow: hidden;
            z-index: 23456765435;
            background: #fff;
            border-radius: 8px;
            .header {
                padding: 15px;
                text-align: center;
                font-size: 18px;
                font-weight: 700;
                color: #333;
            }
            .dialog-body {
                flex: 1;
                overflow-x: hidden;
                overflow-y: scroll;
                padding: 0 15px 20px 15px;
            }
            .dialog-footer {
                position: relative;
                display: flex;
                width: 100%;
                // flex-shrink: 1;
                &::after {
                    content: '';
                    position: absolute;
                    top: 0;
                    left: 0;
                    width: 100%;
                    height: 1px;
                    background: #ddd;
                    transform: scaleY(.5);
                }
                .btn {
                    flex: 1;
                    text-align: center;
                    padding: 15px;
                    font-size: 17px;
                    &:nth-child(2) {
                        position: relative;
                        &::after {
                            content: '';
                            position: absolute;
                            left: 0;
                            top: 0;
                            width: 1px;
                            height: 100%;
                            background: #ddd;
                            transform: scaleX(.5);
                        }
                    }
                }
                .btn-confirm {
                    color: #43ac43;
                }
            }
        }
    }
</style>
```

#### 2. 定制弹窗标题、按钮和主题内容

省略样式代码，我们将标题设置为可定制化传入，且为必传属性。

按钮默认显示一个确认按钮，可以定制化确认按钮的文案，以及可以显示取消按钮，并且可定制化取消按钮的文案，以及它们的点击事件的处理。

主题内容建议使用`slot`插槽处理。不清楚的可以到vue官网学习[slot](https://cn.vuejs.org/v2/guide/components.html#%E9%80%9A%E8%BF%87%E6%8F%92%E6%A7%BD%E5%88%86%E5%8F%91%E5%86%85%E5%AE%B9)。

```
<template>
    <div class="dialog">
        <div class="dialog-mark"></div>
        <transition name="dialog">
            <div class="dialog-sprite">
                <!-- 标题 -->
                <section v-if="title" class="header">{{ title }}</section>
    
                <!-- 弹窗的主题内容 -->
                <section class="dialog-body">
                    <slot></slot>
                </section>
    
                <!-- 按钮 -->
                <section class="dialog-footer">
                    <div v-if="showCancel" class="btn btn-refuse" @click="cancel">{{cancelText}}</div>
                    <div class="btn btn-confirm" @click="confirm">{{confirmText}}</div>
                </section>
            </div>
        </transition>
    </div>
</template>

<script>
    export default {
        props: {
            title: String,
            showCancel: {
                typs: Boolean,
                default: false,
                required: false,
            },
            cancelText: {
                type: String,
                default: '取消',
                required: false,
            },
            confirmText: {
                type: String,
                default: '确定',
                required: false,
            },
        },
        data() {
            return {
                name: 'dialog',
            }
        },
        
        ...
        
        methods: {
            /** 取消按钮操作 */
            cancel() {
                this.$emit('cancel', false);
            },
    
            /** 确认按钮操作 */
            confirm() {
                this.$emit('confirm', false)
            },
        }
    }
</script>
```

#### 3. 组件开关

弹窗组件的开关由外部控制，但是没有直接使用show来直接控制。而是对show进行监听，赋值给组件内部变量showSelf。

这样处理也会方便组件内部控制弹窗的隐藏。下文中的点击遮罩层关闭弹窗就是基于这点来处理的。

```
// 只展示了开关相关代码
<template>
    <div v-if="showSelf" class="dialog" :style="{'z-index': zIndex}">
    </div>
</template>

<script>
    export default {
        props: {
            //弹窗组件是否显示 默认不显示 必传属性
            show: {
                type: Boolean,
                default: false,
                required: true,
            },
        },
        data() {
            return {
                showSelf: false,
            }
        },
        watch: {
            show(val) {
                if (!val) {
                    this.closeMyself()
                } else {
                    this.showSelf = val
                }
            }
        },
        created() {
            this.showSelf = this.show;
        },
    }
</script>
```

#### 4. z-index处理

首先我们要保证弹窗组件的层级z-inde足够高，其次要确保弹窗内容的层级比弹窗遮罩层的层级高。

后弹出的弹窗比早弹出的弹窗层级高。（没有完全确保实现）

```
<template>
    <div v-if="showSelf" class="dialog" :style="{'z-index': zIndex}">
        <div class="dialog-mark" :style="{'z-index': zIndex + 1}"></div>
        <transition name="dialog">
            <div class="dialog-sprite" :style="{'z-index': zIndex + 2}">
            
               ...
               
            </div>
        </transition>
    </div>
</template>

<script>
    export default {
        data() {
            return {
                zIndex: this.getZIndex(),
            }
        },
        methods: {
            /**  每次获取之后 zindex 自动增加 */
            getZIndex() {
                let zIndexInit = 20190315;
                return zIndexInit++
            },
        }
    }
</script>
```

#### 5. 点击遮罩层关闭弹窗和处理弹窗底部的页面内容不可滚动

这里我们需要注意的地方是，当组件挂载完成之后，通过给body设置overflow为hidden，来防止滑动弹窗时，弹窗下的页面滚动。

当点击遮罩层层时，我们在组件内部就可以将弹窗组件隐藏。v-if隐藏时也是该组件的销毁。

```
<template>
    <div v-if="showSelf" class="dialog" :style="{'z-index': zIndex}">
        <div class="dialog-mark" @click.self="closeMyself" :style="{'z-index': zIndex + 1}"></div>
    </div>
</template>

<script>
    export default {
        data() {
            return {
                zIndex: this.getZIndex(),
            }
        },
        mounted() {
            this.forbidScroll()
        },
        methods: {
            /** 禁止页面滚动 */
            forbidScroll() {
                this.bodyOverflow = document.body.style.overflow
                document.body.style.overflow = 'hidden'
            },
            
           /** 点击遮罩关闭弹窗 */
            closeMyself(event) {
                this.showSelf = false;
                this.sloveBodyOverflow()
            },
            
            /** 恢复页面的滚动 */
            sloveBodyOverflow() {
                document.body.style.overflow = this.bodyOverflow;
            },
        }
    }
</script>

```

## 组件最后实现的效果

![1.jpeg](https://user-gold-cdn.xitu.io/2020/3/23/171081e798afd360?w=313&h=476&f=jpeg&s=12636)


![2.jpeg](https://user-gold-cdn.xitu.io/2020/3/23/171081e79945fc4d?w=317&h=486&f=jpeg&s=12271)

最终的完整组件代码如下：

```
<template>
    <div v-if="showSelf" class="dialog" :style="{'z-index': zIndex}">
        <div class="dialog-mark" @click.self="closeMyself" :style="{'z-index': zIndex + 1}"></div>
        <transition name="dialog">
            <div class="dialog-sprite" :style="{'z-index': zIndex + 2}">
                <!-- 标题 -->
                <section v-if="title" class="header">{{ title }}</section>
    
                <!-- 弹窗的主题内容 -->
                <section class="dialog-body">
                    <slot></slot>
                </section>
    
                <!-- 按钮 -->
                <section class="dialog-footer">
                    <div v-if="showCancel" class="btn btn-refuse" @click="cancel">{{cancelText}}</div>
                    <div class="btn btn-confirm" @click="confirm">{{confirmText}}</div>
                </section>
            </div>
        </transition>
    </div>
</template>

<script>
    export default {
        props: {
            //弹窗组件是否显示 默认不显示 必传属性
            show: {
                type: Boolean,
                default: false,
                required: true,
            },
            title: {
                type: String,
                required: true,
            },
            showCancel: {
                typs: Boolean,
                default: false,
                required: false,
            },
            cancelText: {
                type: String,
                default: '取消',
                required: false,
            },
            confirmText: {
                type: String,
                default: '确定',
                required: false,
            },
        },
        data() {
            return {
                name: 'dialog',
                showSelf: false,
                zIndex: this.getZIndex(),
                bodyOverflow: ''
            }
        },
        watch: {
            show(val) {
                if (!val) {
                    this.closeMyself()
                } else {
                    this.showSelf = val
                }
            }
        },
        created() {
            this.showSelf = this.show;
        },
        mounted() {
            this.forbidScroll()
        },
        methods: {
            /** 禁止页面滚动 */
            forbidScroll() {
                this.bodyOverflow = document.body.style.overflow
                document.body.style.overflow = 'hidden'
            },
    
            /**  每次获取之后 zindex 自动增加 */
            getZIndex() {
                let zIndexInit = 20190315;
                return zIndexInit++
            },
    
            /** 取消按钮操作 */
            cancel() {
                this.$emit('cancel', false);
            },
    
            /** 确认按钮操作 */
            confirm() {
                this.$emit('confirm', false)
            },
    
            /** 点击遮罩关闭弹窗 */
            closeMyself(event) {
                this.showSelf = false;
                this.sloveBodyOverflow()
            },
    
            /** 恢复页面的滚动 */
            sloveBodyOverflow() {
                document.body.style.overflow = this.bodyOverflow;
            },
        }
    }
</script>

<style lang="less" scoped>
    // 弹窗动画
    .dialog-enter-active,
    .dialog-leave-active {
        transition: opacity .5s;
    }
    
    .dialog-enter,
    .dialog-leave-to {
        opacity: 0;
    }
    
    // 最外层 设置position定位 
    // 遮罩 设置背景层，z-index值要足够大确保能覆盖，高度 宽度设置满 做到全屏遮罩
    .dialog {
        position: fixed;
        top: 0;
        right: 0;
        width: 100%;
        height: 100%;
        // 内容层 z-index要比遮罩大，否则会被遮盖
        .dialog-mark {
            position: absolute;
            top: 0;
            height: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, .6);
        }
    }
    
    .dialog-sprite {
        // 移动端使用felx布局
        position: absolute;
        top: 10%;
        left: 15%;
        right: 15%;
        bottom: 25%;
        display: flex;
        flex-direction: column;
        max-height: 75%;
        min-height: 180px;
        overflow: hidden;
        z-index: 23456765435;
        background: #fff;
        border-radius: 8px;
        .header {
            padding: 15px;
            text-align: center;
            font-size: 18px;
            font-weight: 700;
            color: #333;
        }
        .dialog-body {
            flex: 1;
            overflow-x: hidden;
            overflow-y: scroll;
            padding: 0 15px 20px 15px;
        }
        .dialog-footer {
            position: relative;
            display: flex;
            width: 100%;
            // flex-shrink: 1;
            &::after {
                content: '';
                position: absolute;
                top: 0;
                left: 0;
                width: 100%;
                height: 1px;
                background: #ddd;
                transform: scaleY(.5);
            }
            .btn {
                flex: 1;
                text-align: center;
                padding: 15px;
                font-size: 17px;
                &:nth-child(2) {
                    position: relative;
                    &::after {
                        content: '';
                        position: absolute;
                        left: 0;
                        top: 0;
                        width: 1px;
                        height: 100%;
                        background: #ddd;
                        transform: scaleX(.5);
                    }
                }
            }
            .btn-confirm {
                color: #43ac43;
            }
        }
    }
</style>

```

## 使用方式

1. 在父组件中将弹窗组件引入
```
import TheDialog from './component/TheDialog'
```
2. 组件中components注册
```
components: {
    TheDialog
}
```
3. 在template中使用
```
<the-dialog :show="showDialog" @confirm="confirm2" @cancel="cancel" :showCancel="true" :title="'新标题'" :confirmText="`知道了`" :cancelText="`关闭`">
        <p>主题内容</p>
        <p>主题内容</p>
        <p>主题内容</p>
        <p>主题内容</p>
        <p>主题内容</p>
        <p>主题内容</p>
        <p>主题内容</p>
        <p>主题内容</p>
        <p>主题内容</p>
        <p>主题内容</p>
        <p>主题内容</p>
</the-dialog>
  
<the-dialog :show="showDialog2" @confirm="confirm2" :title="'弹窗组件标题'" :confirmText="`知道了`">
        <p>主题内容</p>
        <p>主题内容</p>
        <p>主题内容</p>
        <p>主题内容</p>
        <p>主题内容</p>
        <p>主题内容</p>
        <p>主题内容</p>
        <p>主题内容</p>
        <p>主题内容</p>
        <p>主题内容</p>
        <p>主题内容</p>
</the-dialog>

<script>
    export default {
        data() {
            return {
                // 控制两个弹窗组件的初始显示与隐藏
                showDialog: true, 
                showDialog2: true,
            }
        },
        methods: {
            cancel(show) {
                this.showDialog = show
            },
            confirm(show) {
                this.showDialog = show
            },
            cancel2(show) {
                this.showDialog2 = show
            },
            confirm2(show) {
                this.showDialog2 = show;
            },
        }
    }
</script>
```

此文简单记录了一个简单弹窗组件的实现步骤。主要使用了vue的slot插槽接受父组件传来的弹窗内容；通过props接收从父组件传过来的弹窗定制化设置以及控制弹窗的显示与隐藏；子组件通过$emit监听事件传送到父组件去进行逻辑处理。

## 其它
不看后悔的[Vue系列](https://juejin.im/post/6844904081731878920)，在这里：[https://juejin.im/post/6844904081731878920](https://juejin.im/post/6844904081731878920)

很多学习 Vue 的小伙伴知识碎片化严重，我整理出系统化的一套关于Vue的学习系列博客。在自我成长的道路上，也希望能够帮助更多人进步。戳 [链接](https://juejin.im/post/6844904081731878920)