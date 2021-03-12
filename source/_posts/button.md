---
title: 纯 CSS 制作赛博朋克 2077 “故障风”按钮
date: 2020-12-30 15:18:00
tags: css
categories: css
---

赛博朋克 2077 这个游戏，在它的网页上，有一个 Available Now 的按钮，当游标移到它之上的时候，会有一个好像故障的毛刺效果
以下就是模仿并且实现这个效果的过程。

{% raw %}
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<style>
    .box{
        display: flex;
        justify-content: center;
        align-items: center;
        margin:0 auto;
        width: 500px;
        height:300px;
        background-color: #F8F005;
    }
    .button,.button::after{
        position: relative;
        width: 380px;
        height: 86px;
        font-size: 36px;
        background: linear-gradient(45deg, transparent 5%, #ff013c 5%);
        border: none;
        outline: none;
        box-shadow: 6px 0px 0px #00E6F6;
        color: #fff;
        letter-spacing: 2px;
        line-height: 88px;
    }
    .button::after{
        --slice-0: inset(50% 50% 50% 50%);
        --slice-1: inset(80% -6px 0 0);
        --slice-2: inset(50% -6px 30% 0);
        --slice-3: inset(10% -6px 85% 0);
        --slice-4: inset(40% -6px 43% 50%);
        --slice-5: inset(80% -6px 5% 0);
        position: absolute;
        display: block;
        content: 'AVAILABLE NOW';
        top: 0;
        left: 0;
        bottom: 0;
        background: linear-gradient(45deg, transparent 3%, #00e6f6 3%, #00e6f6 5%, #ff013c 5%);
        text-shadow: -3px -3px 0 #f8f005, 3px 3px 0 #00e6f6;
        clip-path: inset(80% -6px 0 0);
    }
    .button:hover::after{
        animation: glitch 1s steps(2, end);
    }
    @keyframes glitch {
        0% {
            clip-path: var(--slice-1);
            transform: translate(-20px, -10px);
        }
        
        10% {
            clip-path: var(--slice-3);
            transform: translate(10px, 10px);
        }
        
        20% {
            clip-path: var(--slice-1);
            transform: translate(-10px, 10px);
        }
        
        30% {
            clip-path: var(--slice-3);
            transform: translate(0px, 5px);
        }
        
        40% {
            clip-path: var(--slice-2);
            transform: translate(-5px, 0px);
        }
        
        50% {
            clip-path: var(--slice-3);
            transform: translate(5px, 0px);
        }
        
        60% {
            clip-path: var(--slice-4);
            transform: translate(5px, 10px);
        }
        
        70% {
            clip-path: var(--slice-2);
            transform: translate(-10px, 10px);
        }
        
        80% {
            clip-path: var(--slice-5);
            transform: translate(20px, -10px);
        }
        
        90% {
            clip-path: var(--slice-1);
            transform: translate(-10px, 0px);
        }
        
        100% {
            clip-path: var(--slice-1);
            transform: translate(0);
        }
    }
</style>
<div class="box">
    <button class="button">AVAILABLE NOW</button>
</div>
{% endraw %}
<!-- more -->

##### 编写 HTML 代码
```html
<div class="box">
    <button class="button">AVAILABLE NOW</button>
</div>
```

##### CSS 部分
```css
.box{
    display: flex;
    justify-content: center;
    align-items: center;
    margin:0 auto;
    width: 500px;
    height:300px;
    background-color: #F8F005;
}
.button,.button::after{
    position: relative;
    width: 380px;
    height: 86px;
    font-size: 36px;
    background: linear-gradient(45deg, transparent 5%, #ff013c 5%);
    border: none;
    outline: none;
    box-shadow: 6px 0px 0px #00E6F6;
    color: #fff;
    letter-spacing: 2px;
    line-height: 88px;
}
.button::after{
    --slice-0: inset(50% 50% 50% 50%);
    --slice-1: inset(80% -6px 0 0);
    --slice-2: inset(50% -6px 30% 0);
    --slice-3: inset(10% -6px 85% 0);
    --slice-4: inset(40% -6px 43% 50%);
    --slice-5: inset(80% -6px 5% 0);
    position: absolute;
    display: block;
    content: 'AVAILABLE NOW';
    top: 0;
    left: 0;
    bottom: 0;
    background: linear-gradient(45deg, transparent 3%, #00e6f6 3%, #00e6f6 5%, #ff013c 5%);
    text-shadow: -3px -3px 0 #f8f005, 3px 3px 0 #00e6f6;
    clip-path: inset(80% -6px 0 0);
}
.button:hover::after{
    animation: glitch 1s steps(2, end);
}
@keyframes glitch {
    0% {
        clip-path: var(--slice-1);
        transform: translate(-20px, -10px);
    }
    
    10% {
        clip-path: var(--slice-3);
        transform: translate(10px, 10px);
    }
    
    20% {
        clip-path: var(--slice-1);
        transform: translate(-10px, 10px);
    }
    
    30% {
        clip-path: var(--slice-3);
        transform: translate(0px, 5px);
    }
    
    40% {
        clip-path: var(--slice-2);
        transform: translate(-5px, 0px);
    }
    
    50% {
        clip-path: var(--slice-3);
        transform: translate(5px, 0px);
    }
    
    60% {
        clip-path: var(--slice-4);
        transform: translate(5px, 10px);
    }
    
    70% {
        clip-path: var(--slice-2);
        transform: translate(-10px, 10px);
    }
    
    80% {
        clip-path: var(--slice-5);
        transform: translate(20px, -10px);
    }
    
    90% {
        clip-path: var(--slice-1);
        transform: translate(-10px, 0px);
    }
    
    100% {
        clip-path: var(--slice-1);
        transform: translate(0);
    }
}
```