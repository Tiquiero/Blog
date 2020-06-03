---
title: URLSearchParams和微信分享的一些问题 # 文章标题  
date: 2018/10/13 18:30:05  # 文章发表时间
tags:
- JS
categories: JS # 分类
thumbnail: https://pic.superbed.cn/item/5c7e47303a213b041746313a # 略缩图
---

### URLSearchParams

我们先来看一段代码:

<pre class="line-numbers language-glsl">
    if (!window.URLSearchParams || (new window.URLSearchParams({})).toString() !== '') window.URLSearchParams = class {
        constructor(obj) {
            if (typeof obj === 'string') {
            this._data = obj;
            } else {
                let ret = [];
                for (let it in obj) {
                    ret.push(encodeURIComponent(it) + '=' + encodeURIComponent(obj[it]));
                }
                this._data = ret.join('&');
            }
        }

        toString() { return this._data };
    }
</pre>

什么时候能用到这个转换参数的工具代码呢？比如我们用axios的post方法，默认情况下，axios将JavaScript对象序列化为JSON，想要传递给后台的参数类型和jquery的ajax一样为Form Data的格式。我们可以分别看看这两个数据格式：

![01](https://pic.superbed.cn/item/5c7e47303a213b0417463135)

有人会说直接更改Content-type不就好了,但是修改了后，虽然从控制台看是叫Form data了，但数据其实还是不正确的：

![02](https://pic.superbed.cn/item/5c7e47303a213b0417463137)

其实Form Data的数据格式是`name=value&name=value`这样的形式，而直接更改Content-type后控制台看的那种数据格式，就和使用JSON.stringify序列化数据一样的`"{"a":"hehe","age":10}"`。所以使用URLSearchParams的原理也就是把数据转换成name=value的格式。

### 微信分享

