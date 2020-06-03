---
title: JS中的深拷贝和浅拷贝 # 文章标题  
date: 2019/3/15 20:09:05  # 文章发表时间
tags:
- JS
categories: JS # 分类
thumbnail: https://pic.superbed.cn/item/5c7e47bf3a213b0417463d1a # 略缩图
---

# 基本类型和引用类型

ECMAScript变量可能包含两种不同数据类型的值：基本类型值和引用类型值。基本类型值指的是那些保存在栈内存中的简单数据段，即这种值完全保存在内存中的一个位置。而引用类型值是指那些保存堆内存中的对象，意思是变量中保存的实际上只是一个指针，这个指针指向内存中的另一个位置，该位置保存对象。

目前基本类型有：Boolean、Null、Undefined、Number、String、Symbol(ES6)，引用类型有：Object、Array、Function。

# 深拷贝与浅拷贝

我们先看两个简单的案例：

<pre class="line-numbers language-glsl">
    //案例1
    var num1 = 1, num2 = num1;
    console.log(num1) //1
    console.log(num2) //1

    num2 = 2; //修改num2
    console.log(num1) //1
    console.log(num2) //2

    //案例2
    var obj1 = {x: 1, y: 2}, obj2 = obj1;
    console.log(obj1) //{x: 1, y: 2}
    console.log(obj2) //{x: 1, y: 2}

    obj2.x = 2; //修改obj2.x
    console.log(obj1) //{x: 2, y: 2}
    console.log(obj2) //{x: 2, y: 2}
</pre>

案例1中的值就为基本类型，案例2中的值就为引用类型。案例2中的赋值，会改变之前对象的就是典型的浅拷贝，而不会改变之前对象的就是深拷贝，并且深拷贝与浅拷贝的概念只存在于引用类型。

### Array自有方法

slice()方法：

<pre class="line-numbers language-glsl">
    //一维数组：
    var arr1 = [1, 2], arr2 = arr1.slice();
    console.log(arr1); //[1, 2]
    console.log(arr2); //[1, 2]

    arr2[0] = 3; //修改arr2
    console.log(arr1); //[1, 2]
    console.log(arr2); //[3, 2]

    //二维数组：   
    var arr1 = [1, 2, [3, 4]], arr2 = arr1.slice();
    console.log(arr1); //[1, 2, [3, 4]]
    console.log(arr2); //[1, 2, [3, 4]]

    arr2[2][1] = 5; 
    console.log(arr1); //[1, 2, [3, 5]] 
    console.log(arr2); //[1, 2, [3, 5]]
</pre>

可以看到，一维数组arr2的修改没有影响到arr1，而二维数组影响到了，所以slice()方法只能实现一维数组的深拷贝。

具备同等特性的还有：concat、Array.from() 。

### Object自有方法

1.Object.assign()：

<pre class="line-numbers language-glsl">
    //一维对象
    var obj1 = {x: 1, y: 2}, obj2 = Object.assign({}, obj1);
    console.log(obj1) //{x: 1, y: 2}
    console.log(obj2) //{x: 1, y: 2}

    obj2.x = 2; //修改obj2.x
    console.log(obj1) //{x: 1, y: 2}
    console.log(obj2) //{x: 2, y: 2}

    //二维对象
    var obj1 = {
        x: 1, 
        y: {
            m: 1
        }
    };
    var obj2 = Object.assign({}, obj1);
    console.log(obj1) //{x: 1, y: {m: 1}}
    console.log(obj2) //{x: 1, y: {m: 1}}

    obj2.y.m = 2; //修改obj2.y.m
    console.log(obj1) //{x: 1, y: {m: 2}}
    console.log(obj2) //{x: 2, y: {m: 2}}
</pre>

可以看出Object.assign()也只能实现一维对象的深拷贝。

2.JSON.parse(JSON.stringify(obj))

<pre class="line-numbers language-glsl">
    var obj1 = {
        x: 1, 
        y: {
            m: 1
        }
    };
    var obj2 = JSON.parse(JSON.stringify(obj1));
    console.log(obj1) //{x: 1, y: {m: 1}}
    console.log(obj2) //{x: 1, y: {m: 1}}

    obj2.y.m = 2; //修改obj2.y.m
    console.log(obj1) //{x: 1, y: {m: 1}}
    console.log(obj2) //{x: 2, y: {m: 2}}
</pre>

看起来JSON.parse(JSON.stringify(obj))可以完美的实现深拷贝，但是查询MDN文档后会发现有这样的一段描述：

> undefined、任意的函数以及 symbol 值，在序列化过程中会被忽略（出现在非数组对象的属性值中时）或者被转换成 null（出现在数组中时）。

什么意思呢，我们把刚刚上面那个例子改一下：

<pre class="line-numbers language-glsl">
    var obj1 = {
        x: 1,
        y: undefined,
        z: function add(z1, z2) {
            return z1 + z2
        },
        a: Symbol("foo")
    };
    var obj2 = JSON.parse(JSON.stringify(obj1));
    console.log(obj1) //{x: 1, y: undefined, z: ƒ, a: Symbol(foo)}
    console.log(JSON.stringify(obj1)); //{"x":1}
    console.log(obj2) //{x: 1}
</pre>

可以看到，y,z,a都被忽略了，验证了MDN文档那句话的意思：JSON.parse(JSON.stringify(obj))的使用也是有局限性的，不能深拷贝含有undefined、function、symbol值的对象。不过JSON.parse(JSON.stringify(obj))简单粗暴，已经满足90%的使用场景了。

### 递归解决

我们发现JS 提供的自有方法并不能彻底解决Array、Object的深拷贝问题，我们可以自己使用递归写一个方法实现深拷贝：

<pre class="line-numbers language-glsl">
    function deepCopy(obj){
        let result = {},
            keys = Object.keys(obj),
            key = null,
            temp = null;

        keys.map(function(item,index){
            key = keys[index];
            temp = obj[key];

            if(temp && typeof temp === 'object'){
                // 如果字段的值也是一个对象则递归操作
                result[key] = deepCopy(temp);
            }else{
                result[key] = temp;
            }
        });

        return result;
    }

    var obj1 = {
        x: {
            m: 1
        },
        y: undefined,
        z: function add(z1, z2) {
            return z1 + z2
        },
        a: Symbol("foo")
    };

    var obj2 = deepCopy(obj1);
    obj2.x.m = 2;

    console.log(obj1); //{x: {m: 1}, y: undefined, z: ƒ, a: Symbol(foo)}
    console.log(obj2); //{x: {m: 2}, y: undefined, z: ƒ, a: Symbol(foo)}
</pre>

这个递归函数可以完美解决上面那些问题，我们也可以用第三方库：jquery的$.extend和lodash的_.cloneDeep来解决深拷贝。上面虽然是用Object来验证的，但对于Array也同样适用，因为Array也是特殊的Object。