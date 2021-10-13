---
title: 前端问题大全（HTML） # 文章标题  
date: 2020/11/06  # 文章发表时间
tags:
- HTML
categories: HTML # 分类
thumbnail: https://pic.superbed.cn/item/5c7e45bb3a213b041746125e
---

本文记录了一直以来JS走过的各种坑，以及各种前端HTML面试问题(因为是不定时补充，想起什么就写什么了，所以顺序没有特定排列，可能有点乱，可以关键词快速搜索)，持续更新中~~~

### HTML语义化

根据内容的结构化（内容语义化），选择合适的标签（代码语义化）便于开发者阅读和写出更优雅的代码的同时让浏览器的爬虫和机器很好地解析。尽可能少的使用无语义的标签,在语义不明显时，既可以使用div或者p时，尽量用p, 因为p在默认情况下有上下间距，对兼容特殊终端有利；

渐进增强 progressive enhancement：针对低版本浏览器进行构建页面，保证最基本的功能，然后再针对高级浏览器进行效果、交互等改进和追加功能达到更好的用户体验。
优雅降级 graceful degradation：一开始就构建完整的功能，然后再针对低版本浏览器进行兼容。降级（功能衰减）意味着往回看；而渐进增强则意味着朝前看，同时保证其根基处于安全地带。

### Cookie、session、localStorage以及sessionStorage

cookie：cookie的内容主要包括：名字、值、过期时间、路径和域。路径与域一起构成cookie的作用范围。若不设置时间，则表示这个cookie的生命期为浏览器会话期间，关闭浏览器窗口，cookie就会消失。这种生命期为浏览器会话期的cookie被称为会话cookie。 会话cookie一般不存储在硬盘而是保存在内存里，若设置了过期时间，浏览器就会把cookie保存到硬盘上，关闭后再打开浏览器这些cookie仍然有效直到超过设定的过期时间。Cookie是把用户的数据写给用户的浏览器。

session：在WEB开发中，服务器可以为每个用户浏览器创建一个会话对象（session对象），注意：一个浏览器独占一个session对象(默认情况下)。Session对象由服务器创建。Session技术把用户的数据写到用户独占的session中。服务器创建session出来后，会把session的id号，以cookie的形式回写给客户端，这样，只要客户端的浏览器不关，再去访问服务器时，都会带着session的id号去，服务器发现客户端浏览器带session id过来了，就会使用内存中与之对应的session为之服务。

cookie和session的区别： 

1.cookie数据存放在客户的浏览器上，session数据放在服务器上 

2.cookie不是很安全，别人可以分析存放在本地的cookie并进行cookie欺骗，考虑*到安全应当使用session 

3.session会在一定时间内保存在服务器上，当访问增多，会比较占用你服务器的性能，考虑到减轻服务器性能方面，应当使用cookie，或者使用专门的session服务器

4.单个cookie保存的数*据不能超过4K，很多浏览器都限制一个站点最多保存20个cookie。而对于 session ，其默认大小一般是1024k

5.session中保存的是对象，cookie中保存的是字符串 

6.session不区分路径，同一个用户在访问一个网站期间，所有的session在任何一个地方都可以访问到，而cookie中如果设置了路径参数，那么同一个网站中不同路径下的cookie互相是访问不到的。

Web Storage（localStorage，sessionStorage）的概念和cookie相似，区别是它是为了更大容量存储设计的。sessionStorage、localStorage和cookie的区别 

共同点：都是保存在浏览器端、且同源的 

区别： 

1、cookie数据始终在同源的http请求中携带（即使不需要），即cookie在浏览器和服务器间来回传递，而sessionStorage和localStorage不会自动把数据发送给服务器，仅在本地保存。cookie数据还有路径（path）的概念，可以限制cookie只属于某个路径下 

2、存储大小限制也不同，cookie数据不能超过4K，同时因为每次http请求都会携带cookie、所以cookie只适合保存很小的数据，如会话标识。sessionStorage和localStorage虽然也有存储大小的限制，但比cookie大得多，可以达到5M或更大 

3、数据有效期不同，sessionStorage：仅在当前浏览器窗口关闭之前有效；localStorage：始终有效，窗口或浏览器关闭也一直保存，因此用作持久数据；cookie：只在设置的cookie过期时间之前有效，即使窗口关闭或浏览器关闭 

4、作用域不同，sessionStorage在不同的浏览器窗口中不能共享，即使是同一个页面；localstorage在所有同源窗口中都是共享的；cookie也是在所有同源窗口中都是共享的 

### 浏览器事件循环

HTML标准：

    为了协调事件（event），用户交互（user interaction），脚本（script），渲染（rendering），网络（networking）等，用户代理（user agent）必须使用事件循环（event loops）。
    有两类事件循环：一种针对浏览上下文（browsing context），还有一种针对worker（web worker）。

事件循环的主要机制就是任务队列机制。一个事件循环里有很多个任务队列（task queues）来自不同任务源，每一个任务队列里的任务是严格按照先进先出的顺序执行的，但是不同任务队列的任务的执行顺序是不确定的。按我的理解就是，浏览器会自己调度不同任务队列。

JavaScript中为了不阻塞UI的渲染，很多JavaScript任务都是异步的，它们包括键盘、鼠标I/O输入输出事件、窗口大小的resize事件、定时器（setTimeout、setInterval）事件、Ajax请求网络I/O回调等。当这些异步任务发生的时候，它们将会被放入浏览器的事件任务队列中去。在浏览器内部中存在一个消息循环池，也叫Event Loop（事件循环），JavaScript引擎在运行时后单线程的处理这些事件任务。例如用户在网页中点击了button事件，则它们会被放入在这个事件循环池中，需要等到JavaScript运行时执行线程空闲时候才会按照队列先进先出的原则被一一执行。

虽然JavaScript是单线程执行的，但是浏览器并不是单线程执行的，它们有JavaScript的执行线程、UI节点的渲染线程，图片等资源的加载线程，以及Ajax请求线程等。在Chrome设计中，为了防止因一个Tab window的奔溃而影响整个浏览器，它的每一个Tab被设计为一个进程；在Chrome设计中存在很多的进程，并利用进程间通讯来完成它们之间的同步，因此这也是Chrome快速的法宝之一。对于Ajax的请求也需要特殊线程来执行，当需要发送一个Ajax请求的时候，浏览器会开辟一个新的线程来执行HTTP的请求，它并不会阻塞JavaScript线程的执行，HTTP请求状态变更事件会被作为回调放入到浏览器的事件队列中等待被执行。
