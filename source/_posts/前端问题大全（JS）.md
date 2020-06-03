---
title: 前端问题大全（JS） # 文章标题  
date: 2019/12/17  # 文章发表时间
tags:
- JS
categories: JS # 分类
thumbnail: https://pic.superbed.cn/item/5c7e45bb3a213b041746125e
---

本文记录了一直以来JS走过的各种坑，以及各种前端JS面试问题(因为是不定时补充，想起什么就写什么了，所以顺序没有特定排列，可能有点乱，可以关键词快速搜索)，持续更新中~~~

### 基本类型和引用类型

基本类型：Boolean,Number,String,Null,Underfined,Symbol(ES6)

引用类型：Array,Object,Function

### 事件冒泡，默认事件

js事件传导过程：事件捕获阶段->目标阶段->事件冒泡阶段

![02](https://pic.superbed.cn/item/5c7e46203a213b0417461b2d)

原生js阻止事件冒泡，event.cancelBubble = true;

jQuery: stopPropagation()；

return false阻止的是obj.on事件名称= fn 所触发的默认行为；

addEventListener绑定的事件需要通过event下面的preventDefault()；

### 千位分隔

1.正常思维

<pre class="line-numbers language-glsl">
    function format(n){
        var tempArr = [],
            num = n.toString(),
            revArr = num .split('').reverse();  //分割，倒序

        for(let i in revArr){
            tempArr.push(revArr[i]);
            if((i+1)%3 === 0 && i != revArr.length-1) tempArr.push(',');
        }

        let result = tempArr.reverse().join('');
        return result;
    }
</pre>

2.正则

<pre class="line-numbers language-glsl">
    function format (num) {
        var reg=/\d{1,3}(?=(\d{3})+$)/g;
        return (num + '').replace(reg, '$&,');
    }
</pre>

### 获取地址栏参数工具函数

<pre class="line-numbers language-glsl">
    export function queryString() {
        let params = {}
        let search = location.search.substring(1)

        if (!search) return {}

        search.split('&').forEach(str => {
            let piece = str.split('=')

            if (piece[1])
            params[piece[0]] = decodeURIComponent(piece[1]);
        })

        return params
    }
</pre>

### 小数点后位数

1.四舍五入，js自带的toFixed(n)方法。

2.不四舍五入，小数点后直接截断n位：`Math.floor(12.2456789 * 10^n)/ 10^n`，或者转换为字符串直接截取：`Number(num.toString.slice(0, index+n+1))`，index为小数点的下标。

### 移动端弹窗背景禁止滚动

情形一：弹窗内容需要滚动，点击弹窗的按钮位置固定，这个时候只需要弹窗的时候禁止背景内容可滑动，弹窗消失的时候恢复。需如下代码:

<pre class="line-numbers language-glsl">
    body,html{
        height:100%;
    }

    //禁止滑动
    $('body,html').css('overflow-y','hidden');

    //恢复滑动
     $('body,html').css('overflow-y','auto');
</pre>

弹窗内容不需要滚动的话，可以直接禁止掉弹出层的滚动事件:    （但如果弹窗内容需要滚动的话，直接禁止这个方法就不可行了。）

<pre class="line-numbers language-glsl">
    box1.addEventListener('touchmove', function(e){
        e.preventDefault()
    })
</pre>

情形二：点击弹窗的按钮位置不固定，这个时候如果用情形一的方法的话，第一次弹窗背景会丢失滑动位置，回到顶部（第一次之后再滑动弹窗又不会了）。该种情形下就需要用一段js，动态去计算之前的滑动位置了:

<pre class="line-numbers language-glsl">
    function() {
        disabled(document.getElementById('overlay'))
        disabled(document.getElementById('button'));
        function disabled(obj) {
            // remember Y position on touch start
            var _clientY = null; 

            obj.addEventListener('touchstart', function(event) {
                if (event.targetTouches.length === 1) {
                    // detect single touch
                        _clientY = event.targetTouches[0].clientY;
                }
            }, false);

            obj.addEventListener('touchmove', function(event) {
                if (event.targetTouches.length === 1) {
                    // detect single touch
                    disableRubberBand(event);
                }
            }, false);

            function disableRubberBand(event) {
                var clientY = event.targetTouches[0].clientY - _clientY;

                if (obj.scrollTop === 0 && clientY > 0) {
                    // element is at the top of its scroll
                    event.preventDefault();
                }

                if (isOverlayTotallyScrolled() && clientY < 0) {
                    //element is at the top of its scroll
                    event.preventDefault();
                }
            }

            function isOverlayTotallyScrolled() {
                // https://developer.mozilla.org/en-US/docs/Web/API/Element/scrollHeight#Problems_and_solutions
                
                return obj.scrollHeight - obj.scrollTop < = obj.clientHeight;
            }
        }
    }
</pre>

完整demo代码可以看 [这里](https://github.com/Tiquiero/disabledScrollBg)(场景重现需要移动端查看)

### iOS中Date对象的兼容问题

var date =new Date()，这段代码在Firefox、Chrome、Safari浏览器中都可以运行。但是如果我想根据字符串获取日期，比如`var date =new Date("2018-07-11 08:00")`，它Firefox、Chrome和安卓中都能运行，但是放在iOS和Safari就会报错，错误是NaN。所以我们换成这种形式就都能兼容了`var date =new Date("2018/07/11 08:00")`。

HTML5新增的日历控件中，type=“datetime-local”。如果是Chrome，控件的日期显示格式是2018/07/11 08:00 ，如果是Safari，日期的显示格式是：2017-08-11T08:00。当我们给它取值赋值的时候，就必须得用`$("#time').val("2018-07-11T08:30"); `，如果用Chrome那种方式的话就会报错。

### iOS下video问题（preload无效，不全屏播放）

    document.addEventListener('touchstart',function(){
        var _video = document.getElementsByTagName('video');
        for(var i = 0;i<_video.length;i++){
            _video[i].play();
            _video[i].pause();
        }
    },{once:true});

这个问题当时是因为做一个多个视频轮播的时候，因为iOS下preload无效，则视频表面没有图像，后面的视频就看不出来，让人误以为后面没有下一个视频了。解决办法当用户touchstart的时候手动让视频播放再停止。{once:true}是只执行一次。

iOS下给video加上`webkit-playsinline="true"`属性，则视频不会全屏播放。

### 闭包

概念：很通俗的讲，闭包就是能够读取其他函数内部变量的函数。由于在Javascript语言中，只有函数内部的子函数才能读取局部变量，因此可以把闭包简单理解成"定义在一个函数内部的函数"。所以，在本质上，闭包就是将函数内部和函数外部连接起来的一座桥梁。

<pre class="line-numbers language-glsl">
    function f1(){
        var n = 1;
        n++;
        function f2(){
            alert(n);
        }
        return f2();
    }
    var result = f1();
    result();        // 2
</pre>

闭包的好处：

a.不增加额外的全局变量；b.执行过程中所有变量都是在匿名函数内部。

匿名函数：

<pre class="line-numbers language-glsl">
    //都会立即执行 

    (function(){
        console.log("() inside!");
    }());

    (function(){
        console.log("() outside!");
    })();
</pre>

使用闭包的注意点:

a.由于闭包会使得函数中的变量都被保存在内存中，内存消耗很大，所以不能滥用闭包，否则会造成网页的性能问题，在IE中可能导致内存泄露。解决方法是，在退出函数之前，将不使用的局部变量全部删除。

b.闭包会在父函数外部，改变父函数内部变量的值。

### 内存泄漏

哪些操作会造成内存泄露：内存泄露是指任何对象在你不再拥有或者不需要的时候仍然存在

1.setTimeout()第一个参数是字符串而不是函数的时候：第一个参数为字符串的时候，js内部会调用eval()函数来动态执行这一段字符串，而eval()可能会隐式的创建全局变量或者闭包作用域，造成内存泄露的可能

2.给dom对象添加的属性是一个对象的引用

3.dom对象和js对象相互引用

<pre class="line-numbers language-glsl">
    function A(ele){
        this.ele = ele;
        ele.name = this;
    }

    new A(document.getElementById('test'));
</pre>

4.给dom对象添加attachEvent绑定事件：因为IE的js对象和dom对象使用的是不同的垃圾回收方式，IE用的是引用计数的垃圾回收，当a引用b而b又引用a时，内存就不会被回收。就像闭包在IE中也会导致内存泄露的问题一样。

### this指向

this的指向在函数定义的时候是确定不了的，只有函数执行的时候才能确定this到底指向谁，实际上this的最终指向的是那个调用它的对象。

1.全局环境下，this就代表window对象。

2.对象环境指向对象；

3.构造函数环境。构造函数中的this 会指向创建出来的实例对象，使用new 调用构造函数时，会先创建出一个空对象，然后用call函数把构造函数中的this指针修改为指向这个空对象。执行完环境后，空对象也就有了相关的属性，然后将对象返回出去

4.DOM环境，在 DOM 事件中使用 this，this 指向了触发事件的 DOM 元素本身。

匿名函数(function(){})()，this指向window，严格模式下是undefined.

更改this指向的操作：

1.使用局部变量来代替this指针

<pre class="line-numbers language-glsl">
    var name = "a";
    var obj = {
    name : "a",
    say : function(){
        var _this = this;  //使用一个变量指向 this
        setTimeout(function(){
        console.log(_this.name);
        },0);
    }
    }
    obj.say();
</pre>

2.new关键字可以改变this的指向

3.使用call 或 apply 方法。call也是函数调用的一种形式，可以通过 函数名.call（）来调用函数。但是提供了一个修改this指向的方法。`fun.call(thisObj[,arg1[,arg2[,...]]])`。第一个参数就是要更改this指向的对象，为必选参数; 之后的参数要根据调用的函数是否需要传入参数(为可选的)。

<pre class="line-numbers language-glsl">
    var name = 'a';
    function say(){
        console.log(this.name);
    };
    say();         //a;
    var obj = {
        name : 'tom',
        say : function(){
            console.log(this.name);
        }
    }
    say.call(obj);  //tom           将 say 函数中的 this 替换为传入的对象
    obj.say();  //tom
    obj.say.call(null);  //a    将 obj.say 函数的 this 替换为了 null，也就意味着指向了全局环境
</pre>

apply的作用和call一样，不同的是传参的形式。apply需要以数组的形式传递参数：

<pre class="line-numbers language-glsl">
    function say(arg1,arg2){
        console.log(this.name,arg1,arg2);
    };
    var obj = {
        name : 'tom',
        say : function(){
            console.log(this.name);
        }
    }
    say.apply(obj,['one','two']);//tom one two
</pre>

### JS继承

##### 1.使用对象冒充实现继承(可以多继承)

实现原理:让父类的构造函数成为子类的方法,然后调用该子类的方法,通过this关键字给所有的属性和方法赋值

<pre class="line-numbers language-glsl">
    function ClassA(name){
        this.name = name;
        this.getName = function(){
            return this.name;
        }
    }

    function ClassB(name){
        this.ClassA = ClassA;
        this.ClassA(name);
        delete this.ClassA;
    }

    var b = new ClassB('www');
    b.getName();
</pre>

##### 2.采用原型链的方式实现继承(不能多继承)

实现原理:使子类原型对象指向父类的实例以实现继承,即重写类的原型.

<pre class="line-numbers language-glsl">
    function Animal(name){
        this.name = name;
    }

    function Dog(age){
        this.age = age;
    }

    Dog.prototype = new Animal('wangwang');
    Animal.prototype.test = 'test';
    var dog = new Dog(2);

    dog.instanceof Animal  //true
    dog.instanceof Dog    //false
    dog.test = test;
    dog.name = 'wangwang';
</pre>

##### 3.使用call/apply的方式实现继承

实现原理:通过call或者apply可以实现函数里面this的改变，利用这一特点，可以实现继承

每个函数对象会有一些方法可以去修改函数执行时里面的this，比较常见得到就是call和apply，通过call和apply可以重新定义函数的执行环境，即this的指向

<pre class="line-numbers language-glsl">
    function  Person(name,age){
        this.name = name;
        this.age = age;
        this.say = function say(){
            console.log("name:" + name);
        }
    }

    //call
    function stu(name,age){
        Person.call(this,name,age);
    }

    //apply
    function teacher(name,age){
        Person.apply(this,[name,age]);
        // 或者 Person.apply(this,arguments);
    }   

    //call与aplly的异同：  
    //1.第一个参数this都一样,指当前对象  
    //2.第二个参数不一样：call的是一个个的参数列表；apply的是一个数组（arguments也可以）  

    var per = new Person('a',20),
        stu = new stu('b',25),
        tea = new teacher('c',28);

    per.say() //a

    stu.say() //b

    tea.say() //c
</pre>

### 原型链

每当定义一个对象的时候，对象中都会包含一些预定义的属性。其中一个属性就是原型对象prototype。原型链是针对构造函数的，比如我创建了一个函数A，然后通过变量B new了函数A，那么变量B就会继承函数A的属性。如果我访问B中某个没有在B中被定义的属性，那它就会往上（也就是创建出它的函数A）查找，这个查找的过程叫原型链。

### 跨域

​同源策略：限制从一个源加载的文档或脚本如何与来自另一个源的资源进行交互。这是一个用于隔离潜在恶意文件的关键的安全机制。浏览器的同源策略，出于防范跨站脚本的攻击，禁止客户端脚本（如 JS）对不同域的服务进行跨站调用（通常指使用XMLHttpRequest请求）。

何为同源？只有当协议、端口、和域名都相同的页面，则两个页面具有相同的源。只要网站的协议名protocol、 主机host、 端口号port 这三个中的任意一个不同，网站间的数据请求与传输便构成了跨域调用，会受到同源策略的限制。 

##### 1.JSONP方式

JSONP（JSON with Padding）是数据格式JSON的一种“使用模式”，利用`<script>`元素的这个开放策略，网页可以得到从其他来源动态产生的JSON数据，而这种使用模式就是所谓的 JSONP。JSONP的兼容性好，不需要XMLHttpRequest的支持

缺点：

a.只能使用Get请求。b.不能注册success、error等事件监听函数，不能很容易的确定JSONP请求是否失败。c.JSONP是从其他域中加载代码执行，容易受到跨站请求伪造的攻击，其安全性无法确保。

##### 2.跨域资源共享CORS方式

跨域资源共享定义了在必须访问跨域资源的时，浏览器与服务器应该如何沟通。他的原理是使用自定义的 HTTP 头部，让服务器与浏览器进行沟通，主要是通过设置响应头的 Access-Control-Allow-Origin 来达到目的的。CORS支持所有类型的HTTP请求 

服务器一般需要增加如下响应头的一种或几种：

<pre class="line-numbers language-glsl">
    Access-Control-Allow-Origin: *
    Access-Control-Allow-Methods: POST, GET, OPTIONS
    Access-Control-Allow-Headers: X-PINGOTHER, Content-Type
    Access-Control-Max-Age: 86400
</pre>

跨域请求默认不会携带Cookie信息，如果需要携带，请配置下述参数：

<pre class="line-numbers language-glsl">
    "Access-Control-Allow-Credentials": true

    // Ajax设置
    "withCredentials": true
</pre>

##### 3.window.name+iframe方式

a.iframe标签的跨域能力；b.window.name属性值在文档刷新后依旧存在的能力（且最大允许2M左右）。

​window.name通过在iframe（一般动态创建）中加载跨域HTML文件来起作用。然后，HTML文件将传递给请求者的字符串内容赋值给window.name。然后，请求者可以检索window.name值作为响应。每个iframe都有包裹它的window，而这个window是top window的子窗口。contentWindow属性返回iframe元素的Window对象。你可以使用这个Window对象来访问iframe的文档及其内部DOM。

<pre class="line-numbers language-glsl">
    //下述用端口,10000表示：domainA,10001表示：domainB

    //localhost:10000
    var iframe = document.createElement('iframe');
    iframe.style.display = 'none'; // 隐藏

    var state = 0; // 防止页面无限刷新
    iframe.onload = function() {
        if(state === 1) {
            console.log(JSON.parse(iframe.contentWindow.name));
            // 清除创建的iframe
            iframe.contentWindow.document.write('');
            iframe.contentWindow.close();
            document.body.removeChild(iframe);
        } else if(state === 0) {
            state = 1;
            // 加载完成，指向当前域，防止错误(proxy.html为空白页面)
            // Blocked a frame with origin "http://localhost:10000" from accessing a cross-origin frame.
            iframe.contentWindow.location = 'http://localhost:10000/proxy.html';
        }
    };

    iframe.src = 'http://localhost:10001';
    document.body.appendChild(iframe);

    //localhost:10001
    ...
    window.name = JSON.stringify({a: 1, b: 2});
</pre>

##### 4.window.postMessage()方法

​HTML5新特性，可以用来向其他所有的 window 对象发送消息。需要注意的是我们必须要保证所有的脚本执行完才发送 MessageEvent，如果在函数执行的过程中调用了它，就会让后面的函数超时无法执行。

Tips:Safari浏览器默认只支持CORS跨域请求。

### Ajax

##### 原生写法

<pre class="line-numbers language-glsl">
    function ajax(method, url, data, success) {
        var xhr = null;
        try {
            xhr = new XMLHttpRequest();
        } catch (e) {
            //IE
            xhr = new ActiveXObject('Microsoft.XMLHTTP');
        }
        
        if (method == 'get' && data) {
            url += '?' + data;
        }
        
        xhr.open(method,url,true);
        if (method == 'get') {
            xhr.send();
        } else {
            xhr.setRequestHeader('content-type', 'application/x-www-form-urlencoded');
            xhr.send(data);
        }
        
        xhr.onreadystatechange = function() {
            if ( xhr.readyState == 4 ) {
                if ( xhr.status == 200 ) {
                    success && success(xhr.responseText);
                } else {
                    alert('出错了,Err：' + xhr.status);
                }
            }
        }
    }
</pre>

application/x-www-form-urlencoded(默认)： 窗体数据被编码为名称/值对。这是标准的编码格式。 

multipart/form-data： 窗体数据被编码为一条消息，页上的每个控件对应消息中的一个部分，这个一般文件上传时用。 如果没有type=file的控件，用默认的application/x-www-form-urlencoded就可以了。 但是如果有type=file的话，就要用到multipart/form-data了。

text/plain： 窗体数据以纯文本形式进行编码，其中不含任何控件或格式字符。 

##### jQuery的ajax

<pre class="line-numbers language-glsl">
    $.ajax({
        url: xxxxxx,
        type: 'GET/POST',
        dataType:'json/jsonp',
        data:{},   //POST时有
        contentType: "application/json",  //传入参数的格式,POST时有
        success: function(result){

        },
        error: function(xhr,status){

        }
    });
</pre>

dataType取值：

1、"xml": 返回 XML 文档，可用 jQuery 处理。

2、"html": 返回纯文本 HTML 信息；包含的 script 标签会在插入 dom 时执行。

3、"script": 返回纯文本 JavaScript 代码。不会自动缓存结果。除非设置了 "cache" 参数。注意：在远程请求时(不在同一个域下)，所有 POST 请求都将转为 GET 请求。（因为将使用 DOM 的 script标签来加载）

4、"json": 返回 JSON 数据 。

5、"jsonp": JSONP 格式。使用 JSONP 形式调用函数时，如 "myurl?callback=?" jQuery 将自动替换 ? 为正确的函数名，以执行回调函数。

6、"text": 返回纯文本字符串

7、"local":返回本地数据（即第一次初始化时只加载本地代码显示的样式，而不加载任何后台返回的数据）

### 函数节流和去抖

函数节流(throttle)：通俗一点就是指定一段时间内js只执行一次。比较常见的场景就是页面的滚动事件，因为滚动高频触发，可能一秒钟就会触发一次操作，节流就是让他五秒钟内滚动只触发一次操作。他的要点是，声明一个变量当标志位，记录当前代码是否在执行。如果空闲，则可以正常触发方法执行。如果代码正在执行，则取消这次方法执行，直接return。

<pre class="line-numbers language-glsl">
    function throttle(fn, wait) {
        var canRun = true;
        return function(){
            if(!canRun){
                // 判断是否已空闲，如果在执行中，则直接return
                return;
            }

            canRun = false;
            setTimeout(function(){
                fn.apply(this, arguments);
                canRun = true;
            }, wait);
        };
    }
</pre>

函数去抖(debounce)：通俗说当调用动作n秒后，才会执行该动作，若在这n毫秒内又调用此动作则将重新计算执行时间。比较常见的场景就是用户注册的时候手机号或者邮箱验证，要等用户完全输入完毕后，再去验证格式。

<pre class="line-numbers language-glsl">
    function debounce(fn, wait) {
        let timeout = null;
        return function () {
            clearTimeout(timeout);
            timeout = setTimeout(() => {
                fn.apply(this, arguments);
            }, wait);
        };
    }
</pre>

### 快速排序

<pre class="line-numbers language-glsl">
    function quickSort(arr) {
        if (arr.length < 1) {
            return arr;
        }
        var left = [],right = [],
            pIndex = Math.floor(arr.length / 2),
            p = arr.splice(pIndex, 1)[0];
        for (var i = 0; i < arr.length; i++) {
            if (arr[i] < p) {
                left.push(arr[i]);
            } else {
                right.push(arr[i]);
            }
        }
        return quickSort(left).concat(p, quickSort(right));
    }
</pre>

### 冒泡排序

<pre class="line-numbers language-glsl">
    function bubbleSort(arr){
        var len = arr.length;
        for(var i = 0;i < len-1 ;i++){
            for(var j = 0;j < len - 1 - j;j++){
                if(arr[j] > arr[j+1]){
                    var temp = arr[j+1];
                    arr[j+1] = arr[j];
                    arr[j] = temp;
                }
            }
        }
        return arr;
    }
</pre>

### js判断数据类型

typeof：一般只能返回如下几个结果："number"、"string"、"boolean"、"object"、"function" 和 "undefined"。typeof方法只能判断基本数据类型（number, string, undefined，boolean），数组一般情况下可以用instanceof来判断，用typeof的话输出的是object。

instanceof：一般用来检测某个对象是不是另一个对象的实例

constructor: a.constructor === Array

prototype: Object.prototype.toString.call(a) === '[Object String/Number/Array/Function..]'

### 函数中的arguments

函数里的arguments是个类数组，除了有实参所组成的类似数组以外，还有自己的属性，如callee，arguments.callee就是当前正在执行的这个函数的引用，它只在函数执行时才存在。因为在函数开始执行时，才会自动创建第一个变量arguments对象。转换为真正数组的方法：

1.[...arguments]，... 把数组或类数组对象展开成一系列用逗号隔开的值，加个[]就变成数组了

2.Array.from(arguments)，ES6的from()能将类数组对象和可遍历对象转化为数组。还有个Array.of()能将一组值转化为数组。

3.Array.prototype.slice.apply(arguments)

### 数组去重

1.`Array.from(new Set(arr))`或者`[...new Set(arr)]`，利用ES6新的数据结构 Set。它类似于数组，但是成员的值都是唯一的，没有重复的值。但这种方法还无法去掉“{}”空对象。

2.利用for嵌套for，然后splice去重

<pre class="line-numbers language-glsl">
    for (var i = 0; i < arr.length; i++) {
        for (var j = i +1; j < arr.length; j++) {
            if (arr[i] == arr[j]) {    //第一个等同于第二个，splice方法删除第二个
                    arr.splice(j,1);
                    j--;
            }
        }
    }
</pre>

3.利用indexOf去重(但indexOf不能判断是否有NaN的元素，可以用下面的includes方法)

<pre class="line-numbers language-glsl">
    function unique(arr) {
        if(!Array.isArray(arr)){
            console.log('type error!');
            return 
        }

        var array = [];
        for(var i = 0;i < arr.length;i++){
            if(array.indexOf(arr[i]) === -1){
                array.push(arr[i]);
            }
        }

        return array;
    }
</pre>

4.利用sort()先排序，再遍历数组及相邻元素比对

<pre class="line-numbers language-glsl">
    function unique(arr) {
        if(!Array.isArray(arr)){
            console.log('type error!');
            return 
        }

        arr = arr.sort();
        var array = arr[0];
        for(var i = 1;i < arr.length;i++){
            if(arr[i] != arr[i-1]){
                array.push(arr[i]);
            }
        }

        return array;
    }
</pre>

5.利用includes

<pre class="line-numbers language-glsl">
    function unique(arr) {
        if(!Array.isArray(arr)){
            console.log('type error!');
            return 
        }

        var array = [];
        for(var i = 1;i < arr.length;i++){
            if(!array.includes(arr[i])){
                array.push(arr[i]);
            }
        }

        return array;
    }
</pre>

6.利用Map数据结构去重

<pre class="line-numbers language-glsl">
    function arrayNonRepeatfy(arr) {
        let map = new Map();
        let array = new Array();
        for(let i = 0;i < arr.length;i++){
            if(map.has(arr[i])){
                map.set(arr[i],1)
            }else{
                map.set(arr[i],0);
                array.push(arr[i]);
            }
        }

        return array ;
    }
</pre>

### 变量/函数提升

变量：变量提升即将变量声明提升到它所在作用域的最开始的部分。初始化的变量不会提升，声明的才会

<pre class="line-numbers language-glsl">
    console.loh(a)   // undefined
    var a = 1;
    //相当于 var a 是提升了，所以没有报错not defined，而是undefined，但是a=1没有提升
</pre>

函数：js中创建函数有两种方式：函数声明式和函数字面量式。只有函数声明才存在函数提升。函数提升在变量提升上面

<pre class="line-numbers language-glsl">
    console.log(f1); // function f1() {}   
    console.log(f2); // undefined 
    function f1() {}  //函数声明式
    var f2 = function() {}  //函数字面量式
</pre>

ES6中新增了let，let与var不同的是:

1. let使用块级作用域，JavaScript的作用域（scope）只有全局和局部，对于var声明的变量，只有函数才能为它创建新的作用域，而let支持块级作用域，花括号就能为它创建新的作用域；

2. let不支持在同作用域中声明标识符相同的变量。相同作用域，var可以反复声明相同标识符的变量，而let是不允许的;

3. let用TDZ(死区)禁止了声明前访问，但并不是let不可以变量提升。当前作用域顶部到该变量声明位置中间的部分，都是该let变量的死区，在死区中，禁止访问该变量。由此，我们给出结论，let声明的变量存在变量提升， 但是由于死区我们无法在声明前访问这个变量。

### 数组随机乱序

<pre class="line-numbers language-glsl">
    var arr = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

    //因为Math.random产生的数在0-1之间
    //所以0.5两边的概率是相等的
    //大于0.5时为升序，小于0.5时为降序

    arr.sort(() => Math.random() - 0.5);
</pre>

但是其实这种并不能真正地随机打乱数组。看了一下ECMAScript中关于Array.prototype.sort(comparefn)的标准，其中并没有规定具体的实现算法，但是提到一点：`Calling comparefn(a,b) always returns the same value v when given a specific pair of values a and b as its two arguments.`

也就是说，对同一组a、b的值，comparefn(a, b)需要总是返回相同的值。而上面的() => Math.random() - 0.5（即(a, b) => Math.random() - 0.5）显然不满足这个条件。

翻看v8引擎数组部分的源码，注意到它出于对性能的考虑，对短数组使用的是插入排序，对长数组则使用了快速排序，至此，也就能理解为什么() => Math.random() - 0.5并不能真正随机打乱数组排序了。（有一个没明白的地方：源码中说的是对长度小于等于 22 的使用插入排序，大于 22 的使用快排，但实际测试结果显示分界长度是 10。）

所以最好方案可以使用Lodash 库中的 shuffle 算法，它使用的实际上是Fisher–Yates 洗牌算法：

<pre class="line-numbers language-glsl">
    function shuffle(arr) {
        let i = arr.length;
        while (i) {
            let j = Math.floor(Math.random() * i--); 
            [arr[j], arr[i]] = [arr[i], arr[j]];   //i,j分别交换值
        } 
    }
</pre>

### iOS下document.title无效

奇怪的bug，iOS下在页面初始化的使用document.title是可以生效的，但在页面渲染了后再动态使用document.title不生效，安卓下是可以的。所以iOS下想动态修改页面title时，只能用到这段工具函数了:

<pre class="line-numbers language-glsl">
    export function setTitle(title) {
        if (typeof title !== 'string') return;

        document.title = title;

        if (/ip(hone|od|ad)/i.test(navigator.userAgent)) {
            var i = document.createElement('iframe');
            i.style.display = 'none';
            i.onload = function () {
                setTimeout(function () {
                    i.remove();
                }, 0)
            }
            document.body.appendChild(i);
        }
    }
</pre>

代码大概意思就是iOS下给页面添加一个iframe时，页面会重新刷新title。所以在修改了title后，创建一个iframe，然后又马上删除这个iframe。

### iOS对position:fixed不太友好的兼容办法

比如在安卓下，点击页面底部的输入框，软键盘弹出，页面移动上移。而ios上面，点击页面底部输入框，软键盘弹出，可能会输入框看不到了。解决办法可以让键盘弹出时让滚动条在最底部的位置

<pre class="line-numbers language-glsl">
    if(isiOS){
        $('textarea').focus(function(){
            window.setTimeout(scrollBottom(),500);
        });
    }

    function scrollBottom() {
        window.scrollTo(0,$('body').height());
    }
</pre>

### history.go()和history.back()的用法与区别

go(-1):返回上一页，原页面表单中的内容会丢失；back():返回上一页，原页表表单中的内容会保留。history.go(-1):后退+刷新；history.back():仅后退。

Chrome和ff浏览器后退页面，会刷新后退的页面，若有数据请求也会提交数据申请。类似于history.go(-1);而safari（包括桌面版和ipad版）的后退按钮则不会刷新页面，也不会提交数据申请。类似于history.back();

### new操作符

<pre class="line-numbers language-glsl">
    function Func(){};
    var newFunc = new Func();
</pre>

1.创建一个空对象。`var obj=new Object();`

2.设置原型链。把 obj 的__proto__指向构造函数Func的原型对象 prototype，此时便建立了 obj 对象的原型链。`obj->Func.prototype->Object.prototype->null`。

3.让Func中的this指向obj，并执行Func的函数体。`var result =Func.call(obj);`

4.判断Func的返回值类型。如果无返回值 或者 返回一个非对象值，则将 obj 作为新对象返回；否则会将 result 作为新对象返回。

<pre class="line-numbers language-glsl">
    if (typeof(result) == "object"){
        func = result;
    }
    else{
        func = obj;;
    }
</pre>

### 错误码

1xx-信息提示，表示服务器已接收了客户端请求，客户端可继续发送请求。

2xx-成功：

200 OK 表示客户端请求成功

204 No Content 成功，但不返回任何实体的主体部分

206 Partial Content 成功执行了一个范围（Range）请求

3xx-重定向：

301和302状态码都表示重定向，就是说浏览器在拿到服务器返回的这个状态码后会自动跳转到一个新的URL地址，这个地址可以从响应的Location首部中获取（用户看到的效果就是他输入的地址A瞬间变成了另一个地址B）——这是它们的共同点。

他们的不同在于。301表示旧地址A的资源已经被永久地移除了（这个资源不可访问了），搜索引擎在抓取新内容的同时也将旧的网址交换为重定向之后的网址；

302表示旧地址A的资源还在（仍然可以访问），这个重定向只是临时地从旧地址A跳转到地址B，搜索引擎会抓取新的内容而保存旧的网址。 SEO302好于301。

303 See Other 请求的资源存在着另一个URI，客户端应使用GET方法定向获取请求的资源

304 Not Modified 服务器内容没有更新，可以直接读取浏览器缓存

4xx-客户端错误：

400 Bad Request 表示客户端请求有语法错误，不能被服务器所理解

401 Unauthonzed 表示请求未经授权，该状态代码必须与 WWW-Authenticate 报头域一起使用

403 Forbidden 表示服务器收到请求，但是拒绝提供服务，通常会在响应正文中给出不提供服务的原因

404 Not Found 请求的资源不存在，例如，输入了错误的URL

5xx-服务器错误：

500 Internel Server Error 表示服务器发生不可预期的错误，导致无法完成客户端的请求

503 Service Unavailable 表示服务器当前不能够处理客户端的请求，在一段时间之后，服务器可能会恢复正常

### 输入URL到页面展示

浏览器工作原理的实质就是实现http协议的通讯。发出解析域名的请求，浏览器会先查看本地host有没有对应的规则，如果没有，浏览器会发出一个 DNS请求到本地DNS服务器。然后TCP 连接，（TCP 三次握手），再向服务器发送HTTP响应请求，和服务器建立连接，服务器处理请求返回一个HTTP响应。浏览器读取数据并加载渲染到页面上，断开连接（TCP 四次挥手）。

输入地址，浏览器查找域名的 IP 地址，这一步包括 DNS 具体的查找过程，包括：浏览器缓存->系统缓存->路由器缓存...。浏览器向 web 服务器发送一个 HTTP 请求，服务器的永久重定向响应（从 http://example.com 到 http://www.example.com），浏览器跟踪重定向地址，服务器处理请求，服务器返回一个 HTTP 响应，浏览器显示 HTML，浏览器发送请求获取嵌入在 HTML 中的资源（如图片、音频、视频、CSS、JS等等），浏览器发送异步请求。

### Node密码hash加密

<pre class="line-numbers language-glsl">
    var crypto = require('crypto');

    var generateSalt = function() {
        var set = '0123456789abcdefghijklmnopqurstuvwxyzABCDEFGHIJKLMNOPQURSTUVWXYZ';
        var salt = '';
        for (var i = 0; i < 10; i++) {
            var p = Math.floor(Math.random() * set.length);
            salt += set[p];
        }
        return salt;
    }

    var md5 = function(str) {
        return crypto.createHash('md5').update(str).digest('hex');
    }

    exports.saltAndHash = function(pass, callback) {
        var salt = generateSalt();
        callback(salt + md5(pass + salt));
    }
</pre>

### =、==、===

= 赋值运算符; == 等于，两边值类型不同的时候，要先进行类型转换，再比较; === 严格等于,不做类型转换，类型不同的一定不等;基本类型string,number通过值来比较，而对象（Date,Array）及普通对象通过指针指向的内存中的地址来做比较。 

    [1] == [1] //false    当比较的值是引用值时，会比较两个值在内存中是否为同一对象，两个数组是不同的对象。

    [] == false  //true   ==会把两边的类型转换成一样的，这里会转成number。false => 0，[]用array的valueOf/toString获取基本类型 => ''，空字符串转成数字之后是0，Number('')。

    if([])/if({})  //true 空数组[]和空对象{}都是object类型，因此直接用于if判断条件时就会被转化为true。

    if({} == false)  //false    false => 0，空对象{}转化为NaN，由于NaN与任何数都不相等

### 设计模式

单例：保证一个类只有一个实例，并提供一个访问他的全局控制点

观察者：定义了一种一对多的依赖关系，让多个观察者对象同时监听一个主题对象，在他的状态发送变化时会通知所有的观察者

工厂：定义一个用以创建对象的接口，让子类来决定实例化哪个类，针对一个(抽象工厂针对的是多个)

装饰者(Decorator)：动态的给一个对象添加一些额外的职责

原型：用原型实例指定创建对象的种类，并且通过拷贝这些原型创建新的对象

### MongoDB 

limit()：限制查询条数的方法，skip()：查询结果跳过若干条记录。skip()与limit()的前后顺序没有要求，不管怎么放置他们执行的顺序都是先sort()后skip()最后limit()。

Mongodb加查询条件的count（）会比不加查询条件的慢很多，换MongoCursor.Size()的话会快很多。MongoCursor，他是mongo的游标，他其实并没有真正的查询到结果，相当于懒加载，在调用find时，MongoDB shell并不立即查询数据库，而是在等待真正开始获取数据时才发送查询。你可以通过游标来对最终结果进行控制。比如限limit,skip,sort等。

mongo的分页，开始几页会很快，越到后面，分页越慢，这是mongo会把查询结果加载到内存，所以skip分页很影响效率，不要轻易使用Skip来做查询，否则数据量大了就会导致性能急剧下降，这是因为Skip是一条一条的数过来的。

### 零碎

shift() 方法用于把数组的第一个元素从其中删除，并返回第一个元素的值。

unshift() 方法可向数组的开头添加一个或更多元素，并返回新的长度。

pop() 方法用于删除并返回数组的最后一个元素。

js宿主对象：Window，Navigator，Image;

navigator.onLine可判断是否是移动端断网脱机状态

addEventListener，attachEvent的不同：

一、适应的浏览器版本不同

attachEvent方法适用于IE    addEventListener方法适用于chrome和FF

二、针对的事件不同

attachEvent中的事件带on   而addEventListener中的事件不带on

三、参数的个数不同

attachEvent方法两个参数：第一个参数为事件名称，第二个参数为接收事件处理的函数； 

addEventListener 有三个参数：第一个参数表示事件名称（不含 on，如 "click"）；第二个参数表示要接收事件处理的函数；第三个参数是一个布尔值，一般为false（true表示该元素在事件的“捕获阶段”（由外往内传递时）响应事件；false表示该元素在事件的“冒泡阶段”（由内向外传递时）响应事件。）

node_modules.bat:

<pre class="line-numbers language-glsl">
    //~dp是变量扩充 ,%0代表批处理本身
    cd /d %~dp0 

    //cmd /c dir 是执行完dir命令后关闭命令窗口(/k是不关闭)
    //mklink /d 使用相对路径方式创建链接(/j是以绝对路径)
    cmd /c mklink /d node_modules ..\..\trunk\node_modules
    #pause
</pre>

数组求和，可以用效率比较高的reduce()方法，语法：`array.reduce(callback[, initialValue])`;接受两个参数，callback和initialValue：

callback函数接受4个参数：previousValue(上次调用回调返回的值)、currentValue(当前被处理的元素)、index(当前被处理的元素索引)以及数组本身。

initialValue用作第一个调用 callback的第一个参数的值。 initialValue参数若指定，则当作最初使用的previous值，如果缺省，则使用数组的第一个元素作为previous初始值，同时current往后排一位。 在没有初始值的空数组上调用 reduce 将报错。

示例：

<pre class="line-numbers language-glsl">
    list.reduce(function(preVal,curVal,index,arr){
        return preVal + curVal;
    });
</pre>

未授权CORS的图片也能被drawimage到canvas画布上，但这样做就会污染(taints)画布。 只要 canvas 被污染, 就不能再从画布中提取数据, 也就是说不能再调用 toBlob(), toDataURL() 和 getImageData() 等方法, 否则会抛出安全错误(security error)。

js能监听动画的状态，animationstart，animationend，animationiteration（除了首次开始动画外，其它每次开始动画迭代都触发animationiteration事件。）