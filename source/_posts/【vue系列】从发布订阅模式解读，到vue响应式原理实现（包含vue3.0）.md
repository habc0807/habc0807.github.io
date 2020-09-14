---
title: 【vue系列】从发布订阅模式解读，到vue响应式原理实现（包含vue3.0）
date: 2020-07-27
tags: 
---

事情是这样的，技术群里有小伙伴想让笔者讲讲发布订阅、观察者模式、以及`vue3` 改用了`proxy`之后，发布订阅有什么改变。虽然最近挺忙的，但是既然笔者应允了，就不能食言。

<!--more-->


## 前言
事情是这样的，技术群里有小伙伴想让笔者讲讲发布订阅、观察者模式、以及`vue3` 改用了`proxy`之后，发布订阅有什么改变。虽然最近挺忙的，但是既然笔者应允了，就不能食言。

![](https://user-gold-cdn.xitu.io/2020/7/26/1738ac8acb155771?w=678&h=281&f=jpeg&s=24995)

于是就有了这篇，笔者会从发布订阅模式的基础实现，到 `Vue` 中 `EventBus` 中的发布订阅模式实现，再到`vue2.x响应式原理`中发布订阅模式使用，并手动实现一个vue响应式。最后简单聊聊`vue3.0中响应式原理`中发布订阅模式跟vue2.x的异同点。

## 聊聊发布订阅模式
观察者模式又叫做`发布-订阅模式`，是我们最常用的设计模式之一（@小伙伴：我理解的发布订阅模式和观察者模式是一种。）。它定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖对于它的对象或者函数都将得到通知和更新。观察者模式提供了一个订阅模型，其中对象订阅事件并在发生时得到通知，这种模式是事件驱动的编程基石，它有利于良好的面向对象的设计。在 Javascript 开发中，我们一般用实践模型来替代传统的发布 - 订阅模式。

设计模式里每种模式都有很多实现方式。现实中的发布订阅模式比如有：`订牛奶`、`发短信`、`自定义事件`、`所有UI界面的事件监听`。

### 手工实现一个简易版发布订阅

- 1、首先要指定好谁充当发布者
- 2、发布者有个缓存队列，用于存放回调函数以便通知订阅者
- 3、发布消息的时候，发布者便利缓存队列，触发里面的回调函数 

```javascript
{
    const map = {}

    const listen = (key, fn) => {
        if(!map[key]) {
            map[key] = []
        }
        map[key].push(fn)
    }

    const trigger = (key, data) => {
        map[key].forEach(item => item(data));
    }

    // 测试用例
    listen('event1', () => { console.log('this is listen 1')})
    listen('event2', () => { console.log('this is listen 2')})
    
    trigger('event1') // this is listen 1
    trigger('event2') // this is listen 2
}
```

打印的结果

```
this is listen 1
this is listen 2
```

### 手工实现一个全局的发布订阅
发布订阅模式可以用一个全局的 Event对象来实现，订阅者不需要了解信息来自哪个发布者，发布者也不知道消息回推送给哪些订阅，Event 作为一个类似“中介者” 的角色，把订阅者和发布者联系起来。

```
{
    /**
     * 发布-订阅的通用实现
     */
    var Event = (function() {
        const map = {}

        // 缓存队列
        const listen = (key, fn) => {
            if(!map[key]) {
                map[key] = []
            }

            map[key].push(fn)
        }

        // 发布消息
        const trigger = (...rest) => {
            const key = rest[0]
            const args = rest.slice(1)
            const fns = map[key]

            if(!Array.isArray(fns) || fns.length === 0) {
                return false
            }

            fns.forEach(item => item(...args))
        }

        // 取消订阅事件
        const remove = (key, fn) => {
            const fns = map[key]

            if(!fns) return false
            if (!fn) fns && (fns.length = 0)

            fns.forEach((item, idx) => {
                if(fn === item) {
                    fns.splice(idx, 1)
                }
            })
            map[key] = fns
        }

        return {
            listen,
            trigger,
            remove
        }
    })();

    // 测试用例
    Event.listen(
        'event1',
        (...args) => { console.log('this is listen 1: ', args) }
    )

    const event1Cb = () => { console.log('this is listen 1.1') }
    Event.listen(
        'event1',
        event1Cb
    )

    Event.listen(
        'event2',
        () => { console.log('this is listen 2') }
    )

    Event.trigger('event1', '这是登录的用户信息', '这是用户的权限')
    Event.trigger('event2')

    Event.remove('event1', event1Cb)
    Event.trigger('event1')
}
```
写了几遍发布订阅，还是很简单的，这里还需要解析吗？大家都可以看懂的吧。😁

## vue中的 eventBus
在vue中的数据通信，非常常见的有父子组件通信，兄弟组件通信。父子组件通信很简单，父组件会通过 `props` 向下传递给子组件，子组件通过 $emit事件告诉给父组件。跨多级组件间通信可以用 `provide` 和 `inject`，但是它存在数据不好实时更新的问题。像一个负责页面多处弹窗展现，每次只显示一个弹窗的场景，用`Eventbus` - Vue事件总线更适合，当然对于不同视图公用数据、更新数据的，业务更复杂的场景，建议使用 `Vuex`来处理组件之间的数据通信。这次我们先来说说 `EventBus`通信的使用和实现原理。

`EventBus` 事件总线，不仅仅是 Vue 中独有的，像安卓、后端等都存在这个概念以及它的大量使用。毕竟这是一种通用的发布订阅设计模式和解决方案。在 Vue 中，使用 `EventBus` 来作为所有组件共用的事件中心，可以向该中心注册发送事件或接收事件。

### 如何使用EventBus
通过实例化 Vue，创建一个不具备 DOM 的事件总线。Vue 实例同时在其事件接口中提供了`$emit`、`$on`,  `$off` 和 `$once`方法。

> 注意 Vue 的事件系统不同于浏览器的 `EventTarget` API。尽管它们工作起来是相似的，但是 `$emit`、`$on`, 和 `$off` 并不是 `dispatchEvent`、`addEventListener` 和 `removeEventListener` 的别名。


```
const vm = new Vue();

vm.$on('test', function (msg) {
  console.log(msg)
})
vm.$emit('test', 'hi')
// => "hi
```

监听当前实例上的自定义事件。事件可以由 `vm.$emit` 触发，回调函数会接收所有传入事件触发函数的额外参数。你可能有疑问，`EventBus` 到底是如何实现的，它的原理是什么？

### 纯手工实现一个简单的 EventBus

一个简单的 EventBus，需要满足实现：发事件、监听事件、销毁事件和一次性监听事件等。

- 通过 `$emit(eventName, eventHandler)` 触发一个事件
- 通过 `$on(eventName, eventHandler)` 侦听一个事件
- 通过 `$once(eventName, eventHandler)` 一次性侦听一个事件
- 通过 `$off(eventName, eventHandler)` 停止侦听一个事件

简单实现如下：

```javascript
{
    class EventBus {
        constructor() {
            this.listeners = {}
        }
        /**
         * 缓存事件监听
         * @param {String} type 事件类型
         * @param {Function} cb 回调函数
         */
        on(type, cb) {
            if (!this.listeners[type]) {
                this.listeners[type] = []
            }
            this.listeners[type].push(cb)
        }

        /**
         * 
         * @param {String} type 事件类型
         * @param  {...params} args 参数列表，传回给callback
         */
        emit(type, ...args) {
            if (this.listeners[type] && this.listeners[type].length > 0) {
                const types = this.listeners[type]
                types.forEach(cb => cb(...args));
            }
        }

        /**
         * 移除事件监听
         * 传两个参 移除该事件类型的 回调函数
         * 传一个类型 移除该类型下的所有回调函数列表
         * @param {*} type 
         * @param {*} cb 
         */
        off(type, cb) {
            if (this.listeners[type]) {
                const curIndex = this.listeners[type].findIndex(it => it === cb)
                if (curIndex >= 0) {
                    this.listeners[type].splice(curIndex, 1)
                }
                // 只传type时，移除该事件的所有监听者
                if (this.listeners[type].length === 0) {
                    delete this.listeners[type]
                }
            }
        }
    }

    // 实例化事件总线
    const eb = new EventBus()

    // 注册一个下班事件监听
    eb.on('下班', (params) => {
        console.log('下班啦，撤了！')
    })
    // 发布`下班`事件
    eb.emit('下班')

    // 注册一个回家事件监听
    eb.on('回家', (eat, sleep) => {
        console.log(`下班回家${eat}、${sleep}。`)
    })
    // 发布`回家`事件
    eb.emit('回家', '吃饭', '睡觉')

    // 移除事件监听 测试
    const toBeOffFn = () => {
        console.log('这是一个可以被移除的事件。')
    }
    eb.on('offFn', toBeOffFn)
    eb.emit('offFn')
    eb.off('offFn', toBeOffFn)
    eb.emit('offFn')
    eb.off('offFn')

    console.log(eb)
}
```
现在回头再看 `EventBus`,会不会觉得很简单。它的实现原理基于发布订阅模式。上面我们完成一个事件总线。这种 `EventBus` 优点是降低耦合性，方便理解，缺点也很明显，使用的越多，就越难维护。

## vue2.x 响应式原理

上面的 `EventBus` 是比较传统的先发布后订阅的发布订阅模式，发布订阅模式必须要先订阅再发布吗？比如这种需求：对一个对象上的所有属性进行代理，某个属性被使用了，就将使用这个属性的函数，存储到它的依赖队列。当该属性被修改了，就通知更新改依赖队列。

这个场景，其实就是 `vue的响应式原理`，通过 `Object.defineProperty` 设置 `setter` 与 `getter` 函数，用来实现 `响应式`以及 `依赖收集`，在 `getter`的时候去收集依赖的函数，`setter`的时候将该属性收到到的依赖都更新一遍。订阅者对象需要提供一个 `update` 的方法，来供发布者在需要的是进行回调。接下来实现一个简单的纯手工vue。


```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>响应式vue</title>
</head>
<body>
    <div id="app"></div>
    <script>
    (function() {
        class Dep{
            constructor(){
                this.subs = []
            }

            addSub(sub) {
                if(sub && (this.subs.indexOf(sub) === -1)) {
                    this.subs.push(sub)
                }
            }

            notify() {
                this.subs.length > 0 && this.subs.forEach(sub => {
                    sub.update()
                })
            }
        }
        Dep.depTarget = null;

        // 我依赖别人，别人变了的话，调用我的update
        class Watcher{
            constructor(value, getter) {
                this.getter = getter
                this.value = this.get()
                this.val = value
            }

            get (){
                Dep.depTarget = this
                this.getter()
                Dep.depTarget = null 
                return this.val
            }

            update() {
                this.value = this.get()
            }
        }

        const typeTo = (val) => Object.prototype.toString.call(val)

        function defineReactive(obj, key, val) {
            let dep = new Dep()
            Object.defineProperty(obj, key, {
                enumerable: true,
                configurable: true,
                get(){
                    dep.addSub(Dep.depTarget)
                    return val;
                },
                set(newValue){
                    if(newValue === val) return;
                    val = newValue;
                    dep.notify()
                }
            })
        }

        function walk(obj) {
            Object.keys(obj).forEach(key => {
                if(typeTo(obj[key]) === '[object Object]'){
                    walk(obj[key])
                }
                defineReactive(obj, key, obj[key])
            })
        }

        function observe(obj){
            if(typeTo(obj) !== '[object Object]') {
                return null
            }
            walk(obj)
        }

        class Vue{
            constructor(options) {
                this.$options = options;
                this._data = options.data();
                this.render = options.render;
                this.$el = typeof options.el === 'string' 
                    ? document.querySelector(options.el) 
                    : options.el;
                observe(this._data)
                new Watcher(this._data, ()=> {
                    this.$mount()
                })
            }

            createElement(tagName, data, children){
                let element = document.createElement(tagName)
                if(Object.prototype.toString.call(children) === '[object Array]'){
                    children.forEach(child => {
                        element.appendChild(child)
                    });
                } else {
                    element.textContent = children
                }
                return element
            }

            $mount(){
                const elements = this.render(this.createElement)
                this.$el.innerHTML = ''
                this.$el.appendChild(elements)
            }
        }

        window.app = new Vue({
            el: '#app',
            data(){
                return {
                    info: {
                        message: '个人信息'
                    },
                    age: 3
                }
            },
            render(createElement) {
                return createElement(
                    'div',
                    {
                        attr: {
                            title: this._data.info.message
                        }
                    },
                    [
                        createElement('span', {}, `黑宝快${this._data.age}岁了`)
                    ]
                )
            }
        });

        setTimeout(() => {
            window.app._data.info.message = '更改文案';
            window.app._data.age = 6;
            // console.log('window.app._data.info.message: ', window.app._data.info.message)
        }, 1000)
    })();
    </script>
</body>
</html>
```

其中需要注意的是，每个对象的属性都有一个 `Dep`，用来存放依赖的该属性的函数，存储在闭包 `subs` 队列中。当该属性被触发 `set`时，则 `update` 所有的依赖函数。`vue`的响应式原理也是基于发布订阅模式实现的。看到这里有木有恍然大悟。

## vue3.0 响应式原理
vue2.0的响应式存在一个严重缺陷：无法监听 属性的添加和删除、数组索引和长度的变更。最新推出的Vue3.0将采用的ES6的新API - `Proxy`，对目标对象的操作之前提供了拦截。手动实现一个 `vue3.0响应式`吧。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>手工vue</title>
</head>
<body>
    <div id="app"></div>
    <script>
        (function () {
            class Dep {
                constructor() {
                    this.subs = []
                }

                addSub(sub) {
                    if (sub && (this.subs.indexOf(sub) === -1)) {
                        this.subs.push(sub)
                    }
                }

                notify() {
                    this.subs.length > 0 && this.subs.forEach(sub => {
                        sub.update()
                    })
                }
            }
            Dep.depTarget = null;

            class Watcher {
                constructor(value, getter) {
                    this.getter = getter
                    this.value = this.get()
                    this.val = value
                }

                get() {
                    Dep.depTarget = this
                    this.getter()
                    Dep.depTarget = null
                    return this.val
                }

                update() {
                    this.value = this.get()
                }
            }

            const typeTo = (val) => Object.prototype.toString.call(val)

            function observe(obj) {
                let dep = new Dep()

                if (typeTo(obj) !== '[object Object]') {
                    return null
                }

                return new Proxy(obj, {
                    get(target, key, receiver) {
                        dep.addSub(Dep.depTarget)
                        return target[key];
                    },
                    set(target, key, value, receiver) {
                        let newValue = Reflect.set(target, key, value, receiver)
                        dep.notify()
                        return newValue;
                    }
                })
            }

            class Vue {
                constructor(options) {
                    this.$options = options;
                    this._data = options.data();
                    this.render = options.render;
                    this.$el = typeof options.el === 'string' ?
                        document.querySelector(options.el) :
                        options.el;
                    this.$data = observe(this._data)
                    new Watcher(this._data, () => {
                        this.$mount()
                    })
                }

                createElement(tagName, data, children) {
                    let element = document.createElement(tagName)
                    if (Object.prototype.toString.call(children) === '[object Array]') {
                        children.forEach(child => {
                            element.appendChild(child)
                        });
                    } else {
                        element.textContent = children
                    }
                    return element
                }

                $mount() {
                    const elements = this.render(this.createElement)
                    this.$el.innerHTML = ''
                    this.$el.appendChild(elements)
                }
            }

            window.app = new Vue({
                el: '#app',
                data() {
                    return {
                        info: {
                            message: '个人信息'
                        },
                        age: 3
                    }
                },
                render(createElement) {
                    return createElement(
                        'div', {
                            attr: {
                                title: this.$data.info.message
                            }
                        },
                        [
                            createElement('span', {}, `黑宝(我家猫)快${this.$data.age}岁了`)
                        ]
                    )
                }
            });

            setTimeout(() => {
                window.app.$data.info.message = 4;
                window.app.$data.age = 4;
            }, 1000)
        })();
    </script>
</body>
</html>
```
可以看到，基本除了是将 `Object.defineProperty` 改用 `Proxy`来实现，还是保持原来的发布订阅模式。这里只是用来演示，实际上`Vue3.0` 的响应式实现更复杂一些，大家可以自己去看源码。

## 最后
到此我们今天的主要内容就讲完了。从发布订阅模式到 Vue 的 `EventBus`，再到 `Vue2.x` 的响应式实现中使用的发布订阅模式，以及 `Vue3.x的响应式实现` 中使用的发布订阅模式。希望今天的介绍，可以让你对发布订阅模式和vue的响应式原理有个初入的理解。

如果这篇文章对你理解发布订阅模式起到了帮助，就请三连关注、点赞、评论吧，你的支持是笔者写文的最大动力。

## 参考
- 《JavaScript设计模式与开发实践》
- [从一道面试题简单谈谈发布订阅和观察者模式](https://juejin.im/post/6844904018964119566)
- [掘金小册 - 《剖析 Vue.js 内部运行机制》](https://juejin.im/book/6844733705089449991/section/6844733705211084808)
