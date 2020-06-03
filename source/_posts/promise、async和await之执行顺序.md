---
title: promise、async和await之执行顺序 # 文章标题  
date: 2019/5/22 18:39:05  # 文章发表时间
tags:
- JS
categories: JS # 分类
thumbnail: https://pic.superbed.cn/item/5c7f706d3a213b0417574779
---

先来看一段代码：

<pre class="line-numbers language-glsl">
    async function async1(){
        console.log('async1 start')
        await async2()
        console.log('async1 end')
    }
    async function async2(){
        console.log('async2')
    }
    console.log('script start')
    setTimeout(function(){
        console.log('setTimeout') 
    },0)  
    async1();
    new Promise(function(resolve){
        console.log('promise1')
        resolve();
    }).then(function(){
        console.log('promise2')
    })
    console.log('script end')
</pre>

要正确的理解这段代码的话，得好好理解async，await，Promise这些的概念。

# async

MDN上是这样描述async的：

> async 函数中可能会有 await 表达式，这会使 async 函数暂停执行，等待表达式中的 Promise 解析完成后继续执行 async 函数并返回解决结果。

阮一峰老师的解释通俗一点：

> async 函数返回一个 Promise 对象，当函数执行的时候，一旦遇到 await 就会先返回，等到触发的异步操作完成，再接着执行函数体内后面的语句。

所以MDN描述的暂停执行，实际上是让出了线程（跳出async函数体）然后继续执行后面的脚本的。

# Promise

Promise是异步编程的一种解决方案，它有三种状态，分别是pending-进行中、resolved-已完成、rejected-已失败。当Promise的状态由pending转变为resolved或rejected时，会执行相应的方法，并且状态一旦改变，就无法再次改变状态，这也是它名字promise-承诺的由来。在ES6中，Promise终于成为了原生对象，可以直接使用。

<pre class="line-numbers language-glsl">
    let promise = new Promise ( (resolve, reject) => {
        if ( success ) {
            resolve(a) // pending ——> resolved 参数将传递给对应的回调方法
        } else {
            reject(err) // pending ——> rejectd
        }
    })
</pre>

Promise.prototype.then()，then()是Promise原型链上的方法，它包含两个参数方法，分别是已成功resolved的回调和已失败rejected的回调：

<pre class="line-numbers language-glsl">
    promise.then(
        () => { console.log('this is success callback') },
        () => { console.log('this is fail callback') }
    )
</pre>

Promise.prototype.catch()，.catch()的作用是捕获Promise的错误，与then()的rejected回调作用几乎一致。但是由于Promise的抛错具有冒泡性质，能够不断传递，这样就能够在下一个catch()中统一处理这些错误。同时catch()也能够捕获then()中抛出的错误，所以建议不要使用then()的rejected回调，而是统一使用catch()来处理错误。

<pre class="line-numbers language-glsl">
    promise.then(
        () => { console.log('this is success callback') }
    ).catch(
        (err) => { console.log(err) }
    )
</pre>

then() 和 catch() 都会返回一个新的Promise对象，可以链式调用：

<pre class="line-numbers language-glsl">
    promise.then(() => { 
        console.log('this is success callback') 
    })
    .catch((err) => { 
        console.log(err) 
    })
    .then(...)
    .catch(...)
</pre>

# await

>语法：[return_value] = await expression;

>表达式（express）：一个 Promise 对象或者任何要等待的值。

>返回值（return_value）：返回 Promise 对象的处理结果。如果等待的不是 Promise 对象，则返回该值本身。

所以，当await操作符后面的表达式是一个Promise的时候，它的返回值，实际上就是Promise的回调函数resolve的参数。我们都知道Promise是一个立即执行函数，但是他的成功（或失败：reject）的回调函数resolve却是一个异步执行的回调。当执行到resolve()时，这个任务会被放入到回调队列中，等待调用栈有空闲时事件循环再来取走它。

# setTimeout、setInterval

setTimeout、setInterval都是宏任务，是在最后执行的。setTimeout是在多少时间间隔后把事件扔进队列里，不是马上执行。比如setTimeout(fn,0)的含义是，指定某个任务在主线程最早可得的空闲时间执行，意思就是不用再等多少秒了，只要主线程执行栈内的同步任务全部执行完成，栈为空就马上执行。setInterval是间隔多少时间就把事件扔进队列一次。

宏任务一般是：包括整体代码script，setTimeout，setInterval。

微任务：Promise，process.nextTick。

# 题目分析

铺垫完这些概念，我们回过头看上面那道题目。一开始只是两个函数的定义，不执行。接着是第一个console.log，执行直接输出`script start`。然后遇到了setTimeout，不管，先扔在后面。

然后是async1()，执行async1 这个函数，首先会打印出`async1 start`。然后执行到 await async2()，发现 async2 也是个 async 定义的函数，所以直接执行了“console.log('async2')”，输出`async2`。同时async2返回了一个Promise，划重点：此时返回的Promise会被放入到回调队列中等待，await会让出线程（js是单线程），接下来就会跳出 async1函数 继续往下执行。

然后执行到 new Promise，promise是立即执行的，所以先打印出来`promise1`，然后执行到 resolve 的时候，resolve这个任务就被放到回调队列中等待，然后跳出Promise继续往下执行，输出`script end`。

接下来同步的事件都循环执行完了，调用栈现在已经空出来了，那么事件循环就会去回调队列里面取任务继续放到调用栈里面了。这时候取到的第一个任务，就是前面 async1里 执行到 await async2()时async2放进去的Promise，执行Promise时又触发resolve，划重点：这个resolve又会被放入任务队列继续等待，然后再次跳出 async1函数 继续下一个任务。

接下来取到的下一个任务，就是前面 new Promise 放进去的 resolve回调，也就是那个.then()里的操作，输出`promise2`。

调用栈再次空出来了，事件循环就取到了下一个任务，终于轮到的那个async2放进去的Promise的resolve回调！！执行它什么也不会打印的，因为 async2 并没有return东西。此时 await 定义的这个 Promise 已经执行完并且返回了结果，所以可以继续往下执行 async1函数 后面的代码了，输出`async1 end`。最后别忘了！还有一个我们扔在后面不管的setTimeout，输出`setTimeout`。

所以正确答案就是：

    script start
    async1 start
    async2
    promise1
    script end
    promise2
    async1 end
    setTimeout
