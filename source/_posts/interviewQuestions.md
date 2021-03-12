---
title: 高频知识，轻松应对编程题
date: 2020-12-31 15:14:08
tags: interview
categories: javascript
---
作为前端开发，JS是重中之重，这些都是高频面试题，希望对你能有所帮助。

关于源码都紧遵规范，都可跑通MDN示例，其余的大多会涉及一些关于JS的应用题和本人面试过程

<!-- more -->

##### 数组扁平化
数组扁平化是指将一个多维数组变为一个一维数组
```javascript
const arr = [1, [2, [3, [4, 5]]], 6];
// => [1, 2, 3, 4, 5, 6]
```
###### 方法一：使用flat()
```javascript
const res1 = arr.flat(Infinity);
```
###### 方法二：利用正则
```javascript
const res2 = JSON.stringify(arr).replace(/\[|\]/g, '').split(',');
// 但数据类型都会变为字符串
// 改良版
const res3 = JSON.parse('[' + JSON.stringify(arr).replace(/\[|\]/g, '') + ']');
```
###### 方法三：使用reduce
```javascript
const flatten = arr => {
  return arr.reduce((pre, cur) => {
    return pre.concat(Array.isArray(cur) ? flatten(cur) : cur);
  }, [])
}
const res4 = flatten(arr);
```
###### 方法四：函数递归
```javascript
const res5 = [];
const fn = arr => {
  for (let i = 0; i < arr.length; i++) {
    if (Array.isArray(arr[i])) {
      fn(arr[i]);
    } else {
      res5.push(arr[i]);
    }
  }
}
fn(arr);
```

##### 数组去重
```javascript
const arr = [1, 1, '1', 17, true, true, false, false, 'true', 'a', {}, {}];
// => [1, '1', 17, true, false, 'true', 'a', {}, {}]
```
###### 方法一：利用Set
```javascript
const res1 = Array.from(new Set(arr));
```
###### 方法二：两层for循环+splice
```javascript
const unique1 = arr => {
  let len = arr.length;
  for (let i = 0; i < len; i++) {
    for (let j = i + 1; j < len; j++) {
      if (arr[i] === arr[j]) {
        arr.splice(j, 1);
        // 每删除一个树，j--保证j的值经过自加后不变。同时，len--，减少循环次数提升性能
        len--;
        j--;
      }
    }
  }
  return arr;
}
```
###### 方法三：利用indexOf
```javascript
const unique2 = arr => {
  const res = [];
  for (let i = 0; i < arr.length; i++) {
    if (res.indexOf(arr[i]) === -1) res.push(arr[i]);
  }
  return res;
}
```
当然也可以用include、filter，思路大同小异。
###### 方法四：利用include
```javascript
const unique3 = arr => {
  const res = [];
  for (let i = 0; i < arr.length; i++) {
    if (!res.includes(arr[i])) res.push(arr[i]);
  }
  return res;
}
```
###### 方法五：利用filter
```javascript
const unique4 = arr => {
  return arr.filter((item, index) => {
    return arr.indexOf(item) === index;
  });
}
```
###### 方法六：利用Map
```javascript
const unique5 = arr => {
  const map = new Map();
  const res = [];
  for (let i = 0; i < arr.length; i++) {
    if (!map.has(arr[i])) {
      map.set(arr[i], true)
      res.push(arr[i]);
    }
  }
  return res;
}
```

##### 类数组转化为数组
类数组是具有```length```属性，但不具有数组原型上的方法。常见的类数组有```arguments```、```DOM```操作方法返回的结果。

###### 方法一：Array.from
```javascript
Array.from(document.querySelectorAll('div'))
```
###### 方法二：Array.prototype.slice.call()
```javascript
Array.prototype.slice.call(document.querySelectorAll('div'))
```
###### 方法三：扩展运算符
```javascript
[...document.querySelectorAll('div')]
```
###### 方法四：利用concat
```javascript
Array.prototype.concat.apply([], document.querySelectorAll('div'));
```

##### 打印出当前网页使用了多少种HTML元素
```javascript
const fn = () => {
  return [...new Set([...document.querySelectorAll('*')].map(el => el.tagName))].length;
}
// 值得注意的是：DOM操作返回的是类数组，需要转换为数组之后才可以调用数组的方法
```

##### debounce（防抖）
触发高频时间后n秒内函数只会执行一次,如果n秒内高频时间再次触发,则重新计算时间。
```javascript
const debounce = (fn, time) => {
  let timeout = null;
  return function() {
    clearTimeout(timeout)
    timeout = setTimeout(() => {
      fn.apply(this, arguments);
    }, time);
  }
};
```
防抖常应用于用户进行搜索输入节约请求资源，```window```触发```resize```事件时进行防抖只触发一次。

##### throttle（节流）
高频时间触发,但n秒内只会执行一次,所以节流会稀释函数的执行频率。
```javascript
const throttle = (fn, time) => {
  let flag = true;
  return function() {
    if (!flag) return;
    flag = false;
    setTimeout(() => {
      fn.apply(this, arguments);
      flag = true;
    }, time);
  }
}
// 节流常应用于鼠标不断点击触发、监听滚动事件。
```

##### 滚动加载
原理就是监听页面滚动事件，分析clientHeight、scrollTop、scrollHeight三者的属性关系。
```javascript
window.addEventListener('scroll', function() {
  const clientHeight = document.documentElement.clientHeight;
  const scrollTop = document.documentElement.scrollTop;
  const scrollHeight = document.documentElement.scrollHeight;
  if (clientHeight + scrollTop >= scrollHeight) {
    // 检测到滚动至页面底部，进行后续操作
    // ...
  }
}, false);
```

##### 图片懒加载
可以给img标签统一自定义属性data-src='default.png'，当检测到图片出现在窗口之后再补充src属性，此时才会进行图片资源加载。
```javascript
function lazyload() {
  const imgs = document.getElementsByTagName('img');
  const len = imgs.length;
  // 视口的高度
  const viewHeight = document.documentElement.clientHeight;
  // 滚动条高度
  const scrollHeight = document.documentElement.scrollTop || document.body.scrollTop;
  for (let i = 0; i < len; i++) {
    const offsetHeight = imgs[i].offsetTop;
    if (offsetHeight < viewHeight + scrollHeight) {
      const src = imgs[i].dataset.src;
      imgs[i].src = src;
    }
  }
}
window.addEventListener('scroll', lazyload);
// 可以使用节流优化一下
// window.addEventListener('scroll', setTimeout(()->{
//     lazyload
// },20));
```
