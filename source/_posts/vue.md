---
title: VUE 常用指令
date: 2020-12-22 09:40:27
tags: vue
categories: vue
---
入门级范例，结合Vue文档使用更佳。推荐用于新手学习，不推荐直接用于生产环境。

<!-- more -->

在 Vue，除了核心功能默认内置的指令 ( ```v-model``` 和 ```v-show``` )，Vue 也允许注册  自定义指令。它的作用价值在于当开发人员在某些场景下需要对普通 DOM 元素进行操作。

Vue 自定义指令有全局注册和局部注册两种方式。先来看看注册全局指令的方式，通过 ```Vue.directive( id, [definition] )``` 方式注册全局指令。然后在入口文件中进行 ```Vue.use()``` 调用。

批量注册指令，新建 ```directives/index.js``` 文件:

```javascript
import copy from './copy'
import longpress from './longpress'
// 自定义指令
const directives = {
  copy,
  longpress,
}

export default {
  install(Vue) {
    Object.keys(directives).forEach((key) => {
      Vue.directive(key, directives[key])
    })
  },
}
```

在```main.js```中引用

```javascript
import Vue from 'vue'
import Directives from './JS/directives'
Vue.use(Directives)
```

##### 钩子函数
指令定义函数提供了几个钩子函数（可选）
    - bind: 只调用一次，指令第一次绑定到元素时调用，可以定义一个在绑定时执行一次的初始化动作。
    - inserted：被绑定元素插入父节点时调用（父节点存在即可调用，不必存在于 document 中）。
    - update: 被绑定元素所在的模板更新时调用，而不论绑定值是否变化。通过比较更新前后的绑定值。
    - componentUpdated: 被绑定元素所在模板完成一次更新周期时调用。
    - unbind: 只调用一次， 指令与元素解绑时调用。

##### 常用自定义指令
    - 复制粘贴指令 v-copy
    - 长按指令 v-longpress
    - 输入框防抖指令 v-debounce
    - 权限校验指令 v-premission

##### 复制粘贴指令(v-copy)

需求：实现一键复制文本内容，用于鼠标右键粘贴。

思路：
    - 1.动态创建 ```textarea``` 标签，并设置 ```readOnly``` 属性及移出可视区域
    - 2.将要复制的值赋给 ```textarea``` 标签的 ```value``` 属性，并插入到 ```body```
    - 3.选中值 ```textarea``` 并复制
    - 4.将 ```body``` 中插入的 ```textarea``` 移除
    - 5.在第一次调用时绑定事件，在解绑时移除事件

```javascript
const copy = {
  bind(el, { value }) {
    el.$value = value
    el.handler = () => {
      if (!el.$value) {
        // 值为空的时候，给出提示。可根据项目UI仔细设计
        console.log('无复制内容')
        return
      }
      // 动态创建 textarea 标签
      const textarea = document.createElement('textarea')
      // 将该 textarea 设为 readonly 防止 iOS 下自动唤起键盘，同时将 textarea 移出可视区域
      textarea.readOnly = 'readonly'
      textarea.style.position = 'absolute'
      textarea.style.left = '-9999px'
      // 将要 copy 的值赋给 textarea 标签的 value 属性
      textarea.value = el.$value
      // 将 textarea 插入到 body 中
      document.body.appendChild(textarea)
      // 选中值并复制
      textarea.select()
      const result = document.execCommand('Copy')
      if (result) {
        console.log('复制成功') // 可根据项目UI仔细设计
      }
      document.body.removeChild(textarea)
    }
    // 绑定点击事件，就是所谓的一键 copy 啦
    el.addEventListener('click', el.handler)
  },
  // 当传进来的值更新的时候触发
  componentUpdated(el, { value }) {
    el.$value = value
  },
  // 指令与元素解绑的时候，移除事件绑定
  unbind(el) {
    el.removeEventListener('click', el.handler)
  },
}

export default copy
```
使用：给 Dom 加上 v-copy 及复制的文本即可
```javascript
<template>
  <button v-copy="copyText">复制</button>
</template>

<script>
  export default {
    data() {
      return {
        copyText: 'a copy directives',
      }
    },
  }
</script>
```

##### 长按指令(v-longpress)

需求：实现长按，用户需要按下并按住按钮几秒钟，触发相应的事件

思路：
    - 1.创建一个计时器， 2 秒后执行函数
    - 2.当用户按下按钮时触发 mousedown 事件，启动计时器；用户松开按钮时调用 mouseout 事件。
    - 3.如果 mouseup 事件 2 秒内被触发，就清除计时器，当作一个普通的点击事件
    - 4.如果计时器没有在 2 秒内清除，则判定为一次长按，可以执行关联的函数。
    - 5.在移动端要考虑 touchstart，touchend 事件

```javascript
const longpress = {
  bind: function (el, binding, vNode) {
    if (typeof binding.value !== 'function') {
      throw 'callback must be a function'
    }
    // 定义变量
    let pressTimer = null
    // 创建计时器（ 2秒后执行函数 ）
    let start = (e) => {
      if (e.type === 'click' && e.button !== 0) {
        return
      }
      if (pressTimer === null) {
        pressTimer = setTimeout(() => {
          handler()
        }, 2000)
      }
    }
    // 取消计时器
    let cancel = (e) => {
      if (pressTimer !== null) {
        clearTimeout(pressTimer)
        pressTimer = null
      }
    }
    // 运行函数
    const handler = (e) => {
      binding.value(e)
    }
    // 添加事件监听器
    el.addEventListener('mousedown', start)
    el.addEventListener('touchstart', start)
    // 取消计时器
    el.addEventListener('click', cancel)
    el.addEventListener('mouseout', cancel)
    el.addEventListener('touchend', cancel)
    el.addEventListener('touchcancel', cancel)
  },
  // 当传进来的值更新的时候触发
  componentUpdated(el, { value }) {
    el.$value = value
  },
  // 指令与元素解绑的时候，移除事件绑定
  unbind(el) {
    el.removeEventListener('click', el.handler)
  },
}

export default longpress
```
使用：给 Dom 加上 v-longpress 及回调函数即可
```javascript
<template>
  <button v-longpress="longpress">长按</button>
</template>

<script>
export default {
  methods: {
    longpress () {
      alert('长按指令生效')
    }
  }
}
</script>
```
##### 输入框防抖指令(v-debounce)

背景：在开发中，有些提交保存按钮有时候会在短时间内被点击多次，这样就会多次重复请求后端接口，造成数据的混乱，比如新增表单的提交按钮，多次点击就会新增多条重复的数据。

需求：防止按钮在短时间内被多次点击，使用防抖函数限制规定时间内只能点击一次。

思路：
    - 1.定义一个延迟执行的方法，如果在延迟时间内再调用该方法，则重新计算执行时间。
    - 2.将事件绑定在 click 方法上。

```javascript
const debounce = {
  inserted: function (el, binding) {
    let timer
    el.addEventListener('click', () => {
      if (timer) {
        clearTimeout(timer)
      }
      timer = setTimeout(() => {
        binding.value()
      }, 1000)
    })
  },
}

export default debounce
```
使用：给 Dom 加上 v-debounce 及回调函数即可
```javscript
<template>
  <button v-debounce="debounceClick">防抖</button>
</template>

<script>
export default {
  methods: {
    debounceClick () {
      console.log('只触发一次')
    }
  }
}
</script>
```

##### 权限校验(v-permission)

背景：在一些后台管理系统，我们可能需要根据用户角色进行一些操作权限的判断，很多时候我们都是粗暴地给一个元素添加 ```v-if / v-show``` 来进行显示隐藏，但如果判断条件繁琐且多个地方需要判断，这种方式的代码不仅不优雅而且冗余。针对这种情况，我们可以通过全局自定义指令来处理。

需求：自定义一个权限指令，对需要权限判断的 Dom 进行显示隐藏。

思路：
    - 1.自定义一个权限数组
    - 2.判断用户的权限是否在这个数组内，如果是则显示，否则则移除 Dom

```javascript
function checkArray(key) {
  let arr = ['1', '2', '3', '4']
  let index = arr.indexOf(key);
  if (index > -1) {
    return true // 有权限
  } else {
    return false // 无权限
  }
}

const permission = {
  inserted: function (el, binding) {
    let permission = binding.value // 获取到 v-permission的值
    if (permission) {
      let hasPermission = checkArray(permission)
      if (!hasPermission) {
        // 没有权限 移除Dom元素
        el.parentNode && el.parentNode.removeChild(el)
      }
    }
  },
}

export default permission
```
使用：给 ```v-permission``` 赋值判断即可
```javascript
<div class="btns">
  <!-- 显示 -->
  <button v-permission="'1'">权限按钮1</button>
  <!-- 不显示 -->
  <button v-permission="'10'">权限按钮2</button>
</div>
````