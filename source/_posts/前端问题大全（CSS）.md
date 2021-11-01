---
title: 前端问题大全（CSS） # 文章标题  
date: 2020/12/16  # 文章发表时间
tags:
- CSS
categories: CSS # 分类
thumbnail: https://pic.superbed.cn/item/5c7e45bb3a213b041746125e
---

本文记录了一直以来CSS走过的各种坑，以及各种前端CSS面试问题(因为是不定时补充，想起什么就写什么了，所以顺序没有特定排列，可能有点乱，可以关键词快速搜索)，持续更新中~~~

css加载不会阻塞DOM树的 解析

css加载会阻塞DOM树的 渲染

css加载会阻塞后面js语句的执行

### 盒模型

盒模型有两种，标准模型（box-sizing:content-box）和IE模型（box-sizing:border-box）。假设页面有一个div，其样式如下：

<pre class="line-numbers language-glsl">
    div {
        width: 50px;
        height: 50px;
        background: #ff00ff;
        border: 1px solid #ffff00;
        padding: 20px 10px;
        margin: 20px;
    }
</pre>

标准模型下，盒子的宽度为50px+(1*2)px+(10*2)px=72px，高度为50px+(1*2)px+(20*2)px=92px，也就是默认把border和padding算在了盒子最后的宽高内。

IE模型下，盒子的宽度为50px，高度为50px，也就是在某一方向的(padding+border)*2小于盒子的width时，最后的宽高依然等于盒子原始的width和height，如果某一方向的(padding+border)*2大于盒子的width时，此时宽高计算方式为(padding+border)*2。

除了模型本身外需要关注的一点就是margin。如果盒子是上下结构，则之间的间距为拥有较大margin的盒子，如果是左右结构，则间距为两个盒子的margin值相加。

### 水平垂直居中

方法1：div绝对定位水平垂直居中【margin:auto实现绝对定位元素的居中】

<pre class="line-numbers language-glsl">
    div {
        width: 200px;
        height: 200px;

        position:absolute;
        left:0;
        top:0;
        right:0;
        bottom:0;
        margin:auto;
    }
</pre>

方法2：div绝对定位水平垂直居中【margin 负间距】

<pre class="line-numbers language-glsl">
    div {
        width: 200px;
        height: 200px;

        position:absolute;
        left:50%;
        top:50%;
        margin-left: -100px;
        margin-top: -100px;
    }
</pre>

方法3：div绝对定位水平垂直居中【Transforms 变形】

<pre class="line-numbers language-glsl">
    div {
        width: 200px;
        height: 200px;

        position:absolute;
        left:50%;
        top:50%;
        transform: translate(-50%,-50%);
    }
</pre>

方法4：css不定宽高水平垂直居中

<pre class="line-numbers language-glsl">
    .box {
        diplay:flex;
        justify-content: content;
        align-items: center;
    }
    .box>div{
        width: 200px;
        height: 200px;
    }
</pre>

方法5：将父盒子设置为table-cell元素，可以使用text-align:center和vertical-align:middle实现水平、垂直居中。比较完美的解决方案是利用三层结构模拟父子结构

<pre class="line-numbers language-glsl">
    .A
        .B
            .C

    /*
    table-cell实现居中
    将父盒子设置为table-cell元素，设置
    text-align:center,vertical-align: middle;
    子盒子设置为inline-block元素
    */

    .A{
        diplay: table;
    }
    .A .B{
        diplay:table-cell;
        text-align: center;
        vertical-align: middle;
    }
    .A .C{
        display:inline-block;
    }
</pre>

方法6：未知div宽高在div中上下左右居中，把父元素设为display:flex，里面的div设为margin:auto

### position

static :是默认值

relative :相对定位 相对于自身原有位置进行偏移，仍处于标准文档流中

absolute :绝对定位 相对于最近的已定位的祖先元素, 有已定位(指position不是static的元素)祖先元素, 以最近的祖先元素为参考标准。如果无已定位祖先元素, 以body元素为偏移参照基准, 完全脱离了标准文档流。

fixed :固定定位的元素会相对于视窗来定位,这意味着即便页面滚动，它还是会停留在相同的位置。一个固定定位元素不会保留它原本在页面应有的空隙。

inherit :规定应该从父元素继承 position 属性的值。

absolute当一直向上查找都没有非static的父元素的时候，是相对于初始包含块来定位的，而初始包含块并不是以html或body进行定位的。

relative是相对于他之前的起始位置来定位的，我们可以设置它的水平或垂直偏移量，让这个元素相对于它在文档流中的位置的起始点进行移动。有一点要注意， 在使用相对定位时，就算元素被偏移了，但是他仍然占据着它没偏移前的空间。

### 块级，行级，内联元素

块级：div,ul,li,p,button,dl,dt,dd,h1-h5,table,td,th,tr,form,fields

行级：a,img,span,input,iframe,strong

内联：a,span,strong,br (不支持宽高，可转换成display：block 或者 display：inline-block)

### 移动端问题

1.iOS页面滑动卡顿：`-webkit-overflow-scrolling: touch;`

2.iOS下margin属性某些情况下无效：(1)html,body的高度是百分比的时候，margin-bottom在safari里失效(2)可以用padding代替margin

3.iOS绑定事件不执行：添加样式cursor: pointer,点击后可能有背景闪一下的问题:-webkit-tap-highlight-color:transparent;

### IE问题

IE8以下不支持getElementByClassName; IE用 currentStyle，其他浏览器用getComputedStyle;

IE(8及以下)有自己私有的滤镜：filter:alpha(opacity=0~100); border-radius IE8没用，解决方法设背景图片;

### 清除浮动

清浮动，最好给浮动元素的父级加 .clear{zoom:1;}.clear:after{content:””;display:block;clear:both;}

### 两列布局，一边固定一边自适应

1.父元素display:flex，自适应边flex：1

2.固定边用float/absolute，自适应边用margin 

例：左边固定，右边自适应(float/position方法)

![03](https://pic.superbed.cn/item/5c7e45ec3a213b0417461552)

但是如果是右边固定，左边自适应,HTML结构中的right需要写在前面，而且position方法，还需要加上right：0

![04](https://pic.superbed.cn/item/5c7e45ec3a213b0417461555)

3.

### 伪类，伪元素

伪类：用于向某些选择器添加特殊的效果。:hover,:active,:focus,:first-child,:link,:vistited

伪元素:用于将特殊的效果添加到某些选择器。:before,:after,:first-letter,:first-line

CSS3新增的伪类:例:

    p:first-of-type 选择属于其父元素的首个 <p> 元素的每个 <p> 元素。

    p:last-of-type  选择属于其父元素的最后 <p> 元素的每个 <p> 元素。

    p:only-of-type  选择属于其父元素唯一的 <p> 元素的每个 <p> 元素。

    p:only-child    选择属于其父元素的唯一子元素的每个 <pre> 元素。

    p:nth-child(2)  选择属于其父元素的第二个子元素的每个 <pre> 元素。

    :enabled、:disabled 控制表单控件的禁用状态。

    :checked，单选框或复选框被选中。

CSS3新增的伪元素: ::selection

### 超出省略号,换行等

<pre class="line-numbers language-glsl">
    <!-- 超出省略号 -->
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;

    word-break: break-all;  //自动换行

    text-transform: uppercase/lowercase  //强制大/小写

    border-collapse：collapse //table里相邻td的border合并为一条
</pre>

### border

border:none;与border:0;的区别:

1.性能方面：

【border:0;】把border设为“0”像素虽然在页面上看不见，但按border默认值理解，浏览器依然对border-width/border-color进行了渲染，即已经占用了内存值。

【border:none;】把border设为“none”即没有，浏览器解析“none”时将不作出渲染动作，即不会消耗内存值。

2.兼容性方面：

【border:none;】当border为“none”时对IE6/7无效，边框依然存在。

【border:0;】所有浏览器都一致把边框隐藏。

对于border:0;与border:none;个人更向于使用,border:none;，因为border:none;毕竟在性能消耗没有争议，而且兼容性可用背景属性解决不足以成为障碍（只需要在同一选择符上添加背景属性即可）。

### display

display:none与visibility:hidden的区别 :(可以类比于Vue中的v-if和v-show)

display:none ,不为被隐藏的对象保留其物理空间 

visibility：hidden ,为被隐藏的对象保留其物理空间

也就是display当他的值变成block 的时候，它所在的结构才会被加载进来。而visibility就会在加载页面的同时就已经把它加载进来了，因为他的值为hidden的时候，它所占的空间还在。

display:inline的元素，垂直方向上的padding/margin无效，水平方向的有。

### jquery语法转换为原生语法

append()的参数可以是节点或者字符串，而appendChild()的参数只能是节点，如果想传入字符串，可以用insertAdjacentHTML("afterEnd",string)代替：
beforeBegin: 插入到标签开始前，afterBegin:插入到标签开始标记之后，beforeEnd:插入到标签结束标记前，afterEnd:插入到标签结束标记后

### 响应式布局 媒体查询 

`<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0,user-scalable=no">`

首先这句代码在移动端开发是基本会出现的，它意思就是：整个网站的宽度让它默认等于屏幕宽度（width=device-width），原始缩放比例（initial-scale=1）为1.0，即网页初始大小占屏幕面积的100%。基本所有常用的浏览器都能设置这句话，IE9也是可以的。

### 零碎

readonly 属性规定输入字段为只读。只读字段是不能修改的。不过，用户仍然可以使用 tab 键切换到该字段，还可以选中或拷贝其文本。

input，img，iframe等元素都不能包含其他元素，所以不能通过伪元素插入内容

使用dataURL技术，图片数据以base64字符串格式嵌入到了页面中，与HTML成为一体。Base64编码的数据体积通常是原数据的体积4/3，也就是dataURL形式的图片会比二进制格式的图片体积大1/3。dataURL形式的图片不会被浏览器缓存，这意味着每次访问这样页面时都被下载一次。这是一个使用效率方面的问题——尤其当这个图片被整个网站大量使用的时候。

H5新增的img属性，srcset（根据不同像素比的设备，使用不同的src地址）,sizes,picture属性

    <img
        srcset="http://placehold.it/2000 2000w,
                http://placehold.it/1500 1500w,
                http://placehold.it/1000 1000w,
                http://placehold.it/500 500w
        "

        sizes="(max-width: 500px) 500px,
            (max-width: 1000px) 1000px,
            (max-width: 1500px) 1500px,
            2000px
        "

        src="http://placehold.it/500/abc"
    />

    //picture 元素内部的 source 与 img 的关系像是相片与相框的关系

    <picture>
        
        <source media="(max-width: 500px)" srcset="http://placehold.it/500” />
        <source media="(max-width: 1000px)" srcset="http://placehold.it/1000” />
        <source media="(max-width: 1500px)" srcset="http://placehold.it/1500” /> 
        <img src="http://placehold.it/500/abc” />

    </picture>
