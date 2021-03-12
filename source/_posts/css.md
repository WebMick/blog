---
title: CSS 常用小知识
date: 2020-12-28 14:12:49
tags: css
categories: css
---
CSS 常用小知识....
<!-- more -->

##### css 一行文本超出...

```css
overflow: hidden;
text-overflow:ellipsis;
white-space: nowrap;
```

##### 多行文本超出显示...

```css
display: -webkit-box;
-webkit-box-orient: vertical;
-webkit-line-clamp: 3;
overflow: hidden;
```

##### IOS 手机容器滚动条滑动不流畅

```css
overflow: auto;
-webkit-overflow-scrolling: touch;
```

##### 修改滚动条样式

隐藏 ```div``` 元素的滚动条

```css
div::-webkit-scrollbar {
    display: none;
}
```

- ```div::-webkit-scrollbar``` 滚动条整体部分
- ```div::-webkit-scrollbar-thumb``` 滚动条里面的小方块，能向上向下移动（或往左往右移动，取决于是垂直滚动条还是水平滚动条
- ```div::-webkit-scrollbar-track``` 滚动条的轨道（里面装有 Thumb
- ```div::-webkit-scrollbar-button``` 滚动条的轨道的两端按钮，允许通过点击微调小方块的位置
- ```div::-webkit-scrollbar-track-piece``` 内层轨道，滚动条中间部分（除去
- ```div::-webkit-scrollbar-corne```r 边角，即两个滚动条的交汇处
- ```div::-webkit-resizer``` 两个滚动条的交汇处上用于通过拖动调整元素大小的小控件注意此方案有兼容性问题，一般需要隐藏滚动条时我都是用一个色块通过定位盖上去，或者将子级元素调大，父级元素使用 ```overflow-hidden``` 截掉滚动条部分。暴力且直接。

##### 使用 css 写出一个三角形角标

元素宽高设置为 0，通过 ```border``` 属性来设置，让其它三个方向的 ```border``` 颜色为透明或者和背景色保持一致，剩余一条 ```border``` 的颜色设置为需要的颜色。

```css
div {
    width: 0;
    height: 0;
    border: 5px solid #transparent;
    border-top-color: red;
}
```

##### 解决 ios audio 无法自动播放、循环播放的问题

```ios``` 手机在使用 ```audio``` 或者 ```video``` 播放的时候，个别机型无法实现自动播放，可使用下面的代码 ```hack```.

```javascript
// 解决ios audio无法自动播放、循环播放的问题
var music = document.getElementById('video');
var state = 0;

document.addEventListener('touchstart', function(){
    if(state==0){
        music.play();
        state=1;
    }
}, false);

document.addEventListener("WeixinJSBridgeReady", function () {
    music.play();
}, false);

//循环播放
music.onended = function () {
    music.load();
    music.play();
}
```

##### 水平垂直居中

定位：
```css
div {
    width: 100px;
    height: 100px;
    position: absolute;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
    margin: auto;
}
```

flex:
```css
.parent {
    display: flex;
    justify-content: center;
    align-items: center;
}
```

##### calc

这是一个 ```css``` 属性，我一般称之为 ```css``` 表达式。可以计算 ```css``` 的值。最有趣的是他可以计算不同单位的差值。很好用的一个功能，
缺点是不容易阅读。接盘侠没办法一眼看出 ```20px``` 是啥。

```css
div {
    width: calc(25% - 20px);
}
```

##### Proxy 和 Object.defineProperty 区别

Proxy 的意思是代理，我一般教他拦截器，可以拦截对象上的一个操作。用法如下，通过 ```new``` 的方式创建对象，第一个参数
是被拦截的对象，第二个参数是对象操作的描述。实例化后返回一个新的对象，当我们对这个新的对象进行操作时就会调用我们描述中对应的方法。

```javascript
new Proxy(target, {
    get(target, property) {

    },
    set(target, property) {

    },
    deleteProperty(target, property) {

    }
})
```

Proxy  区别于 ```Object.definedProperty```。Object.defineProperty 只能监听到属性的读写，而 ```Proxy``` 除读写外还可以监听属性的删除，方法的调用等。

通常情况下我们想要监视数组的变化，基本要依靠重写数组方法的方式实现，这也是 Vue 的实现方式，而 ```Proxy``` 可以直接监视数组的变化。

```javascript
const list = [1, 2, 3];
const listproxy = new Proxy(list, {
    set(target, property, value) {
        target[property] = value;
        return true; // 标识设置成功
    }
});

list.push(4);
```

Proxy 是以非入侵的方式监管了对象的读写，而 ```defineProperty``` 需要按特定的方式定义对象的属性。

##### 解析 get 参数

通过 ```replace``` 方法获取 ```url``` 中的参数键值对，可以快速解析 ```get``` 参数。

```javascript
const q = {};
location.search.replace(/([^?&=]+)=([^&]+)/g,(_,k,v)=>q[k]=v);
console.log(q);
```

##### 解析连接 url
可以通过创建 `a` 标签，给 `a` 标签赋值 `href` 属性的方式，获取到协议，```pathname```，```origin``` 等 ```location``` 对象上的属性。
```javascript
// 创建a标签
const aEle = document.createElement('a');
// 给a标签赋值href路径
aEle.href = '/test.html';
// 访问aEle中的属性
aEle.protocol; // 获取协议
aEle.pathname; // 获取path
aEle.origin;
aEle.host;
aEle.search;
```