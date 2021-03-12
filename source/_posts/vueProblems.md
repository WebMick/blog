---
title: vue项目问题及解决方案
date: 2020-12-31 16:14:12
tags:
---

最近要求使用vue进行前后端分离项目，不断摸索踩坑之后，总结出如下几点vue项目开发中常见的问题及解决办法。

<!-- more -->
- 列表进入详情页的传参问题
- 本地开发环境请求服务器接口跨域的问题
- UI库的按需加载
- 如何优雅的只在当前页面中覆盖ui库中组件的样式
- 定时器问题
- rem文件的导入问题
- Vue-Awesome-Swiper基本能解决你所有的轮播需求
- 打包后生成很大的.map文件的问题
- fastClick 的300ms延迟解决方案
- 查看打包后各文件的体积，帮你快速定位大文件
- 路由懒加载（也叫延迟加载）
- hiper打开速度测试

##### 列表进入详情页的传参问题
例如商品列表页面前往商品详情页面，需要传一个商品id;
```html
<router-link :to="{path: 'detail', query: {id: 1}}">前往detail页面</router-link>
```
c页面的路径为```http://localhost:8080/#/detail?id=1```，可以看到传了一个参数```id=1```，并且就算刷新页面```id```也还会存在。此时在c页面可以通过```id```来获取对应的详情数据，获取```id```的方式```是this.$route.query.id```
vue传参方式有：```query```、```params```+动态路由传参。
说下两者的区别：
1.query通过path切换路由，params通过name切换路由
```html
// query通过path切换路由
<router-link :to="{path: 'Detail', query: { id: 1 }}">前往Detail页面</router-link>
// params通过name切换路由
<router-link :to="{name: 'Detail', params: { id: 1 }}">前往Detail页面</router-link>
```
2.query通过this.$route.query来接收参数，params通过this.$route.params来接收参数。
```javascript
// query通过this.$route.query接收参数
created () {
    const id = this.$route.query.id;
}

// params通过this.$route.params来接收参数
created () {
    const id = this.$route.params.id;
}
```
3.query传参的url展现方式：/detail?id=1&user=123&identity=1&更多参数
params＋动态路由的url方式：/detail/123
4.params动态路由传参，一定要在路由中定义参数，然后在路由跳转的时候必须要加上参数，否则就是空白页面：
```javascript
{      
    path: '/detail/:id',      
    name: 'Detail',      
    component: Detail    
}
```
注意，params传参时，如果没有在路由中定义参数，也是可以传过去的，同时也能接收到，但是一旦刷新页面，这个参数就不存在了。这对于需要依赖参数进行某些操作的行为是行不通的，因为你总不可能要求用户不能刷新页面吧。例如：
```javascript
// 定义的路由中，只定义一个id参数
{
    path: 'detail/:id',
    name: 'Detail',
    components: Detail
}

// template中的路由传参，
// 传了一个id参数和一个token参数
// id是在路由中已经定义的参数，而token没有定义
<router-link :to="{name: 'Detail', params: { id: 1, token: '123456' }}">前往Detail页面</router-link>

// 在详情页接收
created () {
    // 以下都可以正常获取到
    // 但是页面刷新后，id依然可以获取，而token此时就不存在了
    const id = this.$route.params.id;
    const token = this.$route.params.token;
}
```

##### 本地开发环境请求服务器接口跨域的问题
本地开发项目请求服务器接口的时候，因为客户端的同源策略，导致了跨域的问题。
```javascript
proxyTable: {
      // 用‘/api’开头，代理所有请求到目标服务器
      '/api': {
        target: 'http://jsonplaceholder.typicode.com', // 接口域名
        changeOrigin: true, // 是否启用跨域
        pathRewrite: { //
          '^/api': ''
        }
      }
}
// 注意：配置好后一定要关闭原来的server，重新npm run dev启动项目。不然无效。
```

##### UI库的按需加载:
为什么要使用按需加载的方式而不是一次性全部引入，原因就不多说了。这里以```vant```的按需加载为例，演示vue中ui库怎样进行按需加载：
- 安装： cnpm i vant -S
- 安装babel-plugin-import插件使其按需加载：  ```cnpm i babel-plugin-import -D```
- 在 .babelrc文件中中添加插件配置 ：
```javascript
libraryDirectory { 
    
    "plugins": [ 
        // 这里是原来的代码部分
        // …………

        // 这里是要我们配置的代码
        ["import", 
            { 
                "libraryName": "vant", 
                "libraryDirectory": "es", 
                "style": true 
            }
        ] 
    ] 
}
```
- 在main.js中按需加载你需要的插件：
```javascript
// 按需引入vant组件
import {   
    DatetimePicker,   
    Button,   
    List 
} from 'vant';
```
- 使用组件：
```javascript
// 使用vant组件
Vue.use(DatetimePicker)  
    .use(Button)  
    .use(List);
```
- 最后在在页面中使用：
```html
<van-button type="primary">按钮</van-button>
```
ps：出来vant库外，像antiUi、elementUi等，很多ui库都支持按需加载，可以去看文档，上面都会有提到。基本都是通过安装babel-plugin-import插件来支持按需加载的，使用方式与vant的如出一辙，可以去用一下。

##### 如何优雅的只在当前页面中覆盖ui库中组件的样式
首先我们vue文件的样式都是写在<style lang="less" scoped></style>标签中的，加scoped是为了使得样式只在当前页面有效.

通过深度选择器解决,可以这样做：

```css
.van-tabs /deep/ .van-ellipsis { color: blue};
```

这里的深度选择器/deep/是因为我用的less语言，如果你没有使用less/sass等，可以用>>>符号。


##### 定时器问题:
我在a页面写一个定时，让他每秒钟打印一个1，然后跳转到b页面，此时可以看到，定时器依然在执行。这样是非常消耗性能的。

###### 解决方法1：
首先我在data函数里面进行定义定时器名称：
```javascript
data() {            
    return {                              
        timer: null  // 定时器名称          
    }        
}
```
然后这样使用定时器：
```javascript
this.timer = (() => {
    // 某些操作
}, 1000)
```
最后在beforeDestroy()生命周期内清除定时器：
```javascript
beforeDestroy() {
    clearInterval(this.timer);        
    this.timer = null;
}
```
###### 解决方案2：
该方法是通过$once这个事件侦听器器在定义完定时器之后的位置来清除定时器。以下是完整代码：
```javascript
const timer = setInterval(() =>{                    
    // 某些定时器操作                
}, 500);            
// 通过$once来监听定时器，在beforeDestroy钩子可以被清除。
this.$once('hook:beforeDestroy', () => {            
    clearInterval(timer);                                    
})
```
类似于其他需要在当前页面使用，离开需要销毁的组件（例如一些第三方库的picker组件等等），都可以使用此方式来解决离开后以后在背后运行的问题。

##### rem文件的导入问题：
我们在做手机端时，适配是必须要处理的一个问题。例如，我们处理适配的方案就是通过写一个rem.js，原理很简单，就是根据网页尺寸计算html的font-size大小，基本上小伙伴们都知道，这里直接附上代码，不多做介绍。
```javascript
;(function(c,d){var e=document.documentElement||document.body,a="orientationchange" in window?"orientationchange":"resize",b=function(){var f=e.clientWidth;e.style.fontSize=(f>=750)?"100px":100*(f/750)+"px"};b();c.addEventListener(a,b,false)})(window);复制代码
```
这里说下怎么引入的问题，很简单。在```main.js```中，直接```import './config/rem'```导入即可。import的路径根据你的文件路径去填写。

##### Vue-Awesome-Swiper基本能解决你所有的轮播需求
vue-awesome-swiper这个轮播组件，真的非常强大，基本可以满足我们的轮播需求。swiper相信很多人都用过，很好用，也很方便我们二次开发，定制我们需要的轮播效果。vue-awesome-swiper组件实质上基于swiper的，或者说就是能在vue中跑的swiper。下面说下怎么使用：
- 安装 ```cnpm install vue-awesome-swiper --save```
- 在组件中使用的方法，全局使用意义不大：
```javascript
// 引入组件
import 'swiper/dist/css/swiper.css' 
import { swiper, swiperSlide } from 'vue-awesome-swiper'

// 在components中注册组件
components: {
    swiper,
    swiperSlide
}

// template中使用轮播
// ref是当前轮播
// callback是回调
// 更多参数用法，请参考文档
<swiper :options="swiperOption" ref="mySwiper" @someSwiperEvent="callback">            
    <!-- slides -->            
    <swiper-slide><div class="item">1</div></swiper-slide>            
    <swiper-slide><div class="item">2</div></swiper-slide>            
    <swiper-slide><div class="item">3</div></swiper-slide>            
          
    <!-- Optional controls -->            
    <div class="swiper-pagination"  slot="pagination"></div>            
    <div class="swiper-button-prev" slot="button-prev"></div>            
    <div class="swiper-button-next" slot="button-next"></div>            
    <div class="swiper-scrollbar"   slot="scrollbar"></div>
</swiper>
```
```javascript
// 参数要写在data中
data() {            
    return {     
        // swiper轮播的参数           
        swiperOption: { 
            // 滚动条                   
            scrollbar: {                        
                el: '.swiper-scrollbar',                    
            }, 
            // 上一张，下一张                   
            navigation: {                        
                nextEl: '.swiper-button-next',                        
                prevEl: '.swiper-button-prev',                    
            },
            // 其他参数…………   
        }            
    }                    
},
```
swiper需要配置哪些功能需求，自己根据文档进行增加或者删减。

##### 打包后生成很大的.map文件的问题
项目打包后，代码都是经过压缩加密的，如果运行时报错，输出的错误信息无法准确得知是哪里的代码报错。 而生成的.map后缀的文件，就可以像未加密的代码一样，准确的输出是哪一行哪一列有错可以通过设置来不生成该类文件。但是我们在生成环境是不需要.map文件的，所以可以在打包时不生成这些文件：

在config/index.js文件中，设置```productionSourceMap: false```,就可以不生成.map文件

##### fastClick的300ms延迟解决方案
开发移动端项目，点击事件会有300ms延迟的问题。至于为什么会有这个问题，请自行百度即可。这里只说下常见的解决思路，不管vue项目还是jq项目，都可以使用fastClick解决。
安装 ```fastClick```:
```javascript
cnpm install fastclick -S
```
在main.js中引入```fastClick```和初始化:
```javascript
import FastClick from 'fastclick'; // 引入插件
FastClick.attach(document.body); // 使用 fastclick
```

##### 查看打包后各文件的体积，帮你快速定位大文件
如果你是vue-cli初始化的项目，会默认安装webpack-bundle-analyzer插件，该插件可以帮助我们查看项目的体积结构对比和项目中用到的所有依赖。也可以直观看到各个模块体积在整个项目中的占比。
```javascript
npm run build --report // 直接运行，然后在浏览器打开http://127.0.0.1:8888/即可查看复制代码
```
记得运行的时候先把之前**npm run dev**开启的本地关掉

##### 路由懒加载（也叫延迟加载）
路由懒加载可以帮我们在进入首屏时不用加载过度的资源，从而减少首屏加载速度。
路由文件中，
非懒加载写法：
```javascript
import Index from '@/page/index/index';
export default new Router({  
    routes: [    
        { 
            path: '/', 
            name: 'Index',     
            component: Index 
        }
    ]
})
```
路由懒加载写法：
```javascript
export default new Router({
  routes: [    
        { 
            path: '/', 
            name: 'Index', 
            component: resolve => require(['@/view/index/index'], resolve) 
        }
   ]
})
```

##### Hiper：一款令人愉悦的性能分析工具
hiper工具的测试结果，我们可以看到DNS查询耗时、TCP连接耗时、第一个Byte到达浏览器的用时、页面下载耗时、DOM Ready之后又继续下载资源的耗时、白屏时间、DOM Ready 耗时、页面加载总耗时。
在我们的编辑器终端中全局安装：
```javascript
cnpm install hiper -g
```
**使用：**终端输入命令：hiper 测试的网址
```javscript
# 当我们省略协议头时，默认会在url前添加`https://`

 # 最简单的用法
 hiper baidu.com

 # 如何url中含有任何参数，请使用双引号括起来
 hiper "baidu.com?a=1&b=2"

 #  加载指定页面100次
 hiper -n 100 "baidu.com?a=1&b=2"

 #  禁用缓存加载指定页面100次
 hiper -n 100 "baidu.com?a=1&b=2" --no-cache

 #  禁JavaScript加载指定页面100次
 hiper -n 100 "baidu.com?a=1&b=2" --no-javascript
 
 #  使用GUI形式加载指定页面100次
 hiper -n 100 "baidu.com?a=1&b=2" -H false

 #  使用指定useragent加载网页100次
hiper -n 100 "baidu.com?a=1&b=2" -u "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.181 Safari/537.36"复制代码

 ```







