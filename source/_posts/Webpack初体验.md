---
title: Webpack初体验 # 文章标题  
date: 2018/12/14 21:02:05  # 文章发表时间
tags:
- Webpack
categories: Webpack # 分类
thumbnail: https://pic.superbed.cn/item/5c7e47133a213b0417462e67 # 略缩图
---

webpack 是一个现代 JavaScript 应用程序的静态模块打包器(module bundler)。当 webpack 处理应用程序时，它会递归地构建一个依赖关系图(dependency graph)，其中包含应用程序需要的每个模块，然后将所有这些模块打包成一个或多个 bundle。

这篇文章只是最简单的来一遍如何入门使用webpack，当然实际项目使用中会复杂的多，包括不同的需求和复杂的配置。这里就先不多说这些，更多详细的可以看[webpack中文文档](https://www.webpackjs.com/concepts/)。

### 安装 webpack,创建项目

先全局安装webpack和webpack-cli，这样可以在全局使用webpack命令：`npm install webpack -g`  `npm install webpack-cli -g`

新建项目文件夹，在该文件夹路径下执行`npm init`，会生成package.json文件。

接着新建几个项目文件，新建app和src文件夹，app下新建一个index.html，如图：

![02](https://pic.superbed.cn/item/5c7e47133a213b0417462e65)

在src下新建一个index.js文件，写一句测试代码`document.write("It index.");`

接着我们执行`webpack --mode development`，会发现项目里自动创建了一个dist文件夹，里面有个main.js文件，我们打开index.html，页面显示“It index”，这就说明我们的webpack打包成功了。

这就是最简单的，webpack4支持的0配置打包。没有任何配置项，默认情况下webpack会将src文件夹下的入口文件index.js进行打包。当你执行打包命令后，webpack会自动查找项目中src目录下的index.js文件，然后进行打包，生成一个dist目录并存在一个打包好的main.js文件。当然，0配置下文件名字都要是定义好的，不能变。

webpack --mode development/production，这是webpack4的打包命令了，不再是之前 webpack 文件a 文件b的方式了。我们可以在package.json里新增两个命令来简写一下：

<pre class="line-numbers language-glsl">
    "scripts": {
        "dev": "webpack --mode development",
        "build": "webpack --mode production"
    }
</pre>

### webpack配置项

实际使用中我们是不可能0配置的，所以简单了解一下webpack的常用配置项和插件。

在项目根目录下新建一个webpack.config.js文件，这是webpack的配置文件:

<pre class="line-numbers language-glsl">
    const path = require('path');

    module.exports = {
        entry: './src/test.js',
        output: {
            path: path.resolve(__dirname,'dist'),
            filename : 'bundle.js' 
        },
    }
</pre>

entry可以指定打包的入口文件，output是打包后的输出配置项，path是打包后文件的输出路径，filename打包后的文件名。

引入的path是node.js的一个模块，__dirname获得当前文件所在目录的完整目录名，path.resolve拼接返回相对于当前的工作目录的绝对路径。path.resolve和path.join的区别：

<pre class="line-numbers language-glsl">
    //对于以/开始的路径片段，path.join只是简单的将该路径片段进行拼接，而path.resolve将以/开始的路径片段作为根目录，在此之前的路径将会被丢弃
    path.join('/a', '/b') // 'a/b'
    path.resolve('/a', '/b') // '/b'

    //path.resolve总是返回一个以相对于当前的工作目录（working directory）的绝对路径。
    path.join('./a', './b') // 'a/b'
    path.resolve('./a', './b') // '/Users/username/Projects/webpack-demo/a/b'
</pre>

还有更多详细配置项可以看 [相关文档介绍](https://www.webpackjs.com/configuration/)。

### webpack插件

webpack的强大离不开非常丰富的插件，我们可以使用各种各样的插件来定制我们想要的功能，例如：生产环境中我们可能会有缓存的问题，我们可以在打包后的文件名中加入随机的hash值来避免这个问题：`filename : 'bundle.[hash].js'`，这样每次生成的打包文件就会带有一串hash值了。可是这样问题就来了，我们怎么在index.html引入打包后的这个js呢，如果每次都要去更改引入文件名岂不是太麻烦了。这时候我们就可以借助插件来自动帮我们引入每次打包后的文件。

[html-webpack-plugin](https://www.npmjs.com/package/html-webpack-plugin)，这应该是webpack中最常见的插件之一了。安装`npm install html-webpack-plugin --save-dev`，看看他的最基本用法，在webpack配置文件中加入以下配置项：

<pre class="line-numbers language-glsl">
    module.exports = {
        ···
        plugins: [
            new HtmlWebpackPlugin()
        ]
    }
</pre>

打包后，我们会发现在dist文件夹下生成了两个文件，index.html和bundle.[hash].js，而且index.html里自动引入了打包生成的bundle.[hash].js。所有的这些都是 html-webpack-plugin 的功劳。它会自动帮你生成一个 html 文件，并且引用相关的 assets 文件(如 css, js)。当然，html-webpack-plugin还支持很多配置项，例如生成的文件名，或者可以套用模板文件来生成：

<pre class="line-numbers language-glsl">
    module.exports = {
        ···
        plugins: [
           new HtmlWebpackPlugin({
                filename: './index.html',
                template:'./app/index.html'
            })
        ]
    }
</pre>

html-webpack-plugin更多配置项可以看 [官方文档](https://www.npmjs.com/package/html-webpack-plugin)。

还有各种各样的插件，例如每次打包dist都会生成文件，多次打包之后我们想要dist只保留最新的文件，就可以用clean-webpack-plugin插件，每次打包前删除dist里的文件：

<pre class="line-numbers language-glsl">
    module.exports = {
        ···
        plugins: [
            ···
            new CleanWebpackPlugin(['dist'],{
                root: __dirname,
            })
        ]
    }
</pre>

### webpack，sources

实现项目中，我们可能还需要引入其他资源文件，例如css文件。webpack可以把以指定入口的一系列相互依赖的模块打包成一个文件，这里的模块指的不只是js，也可以是css。在webpack中，就可以通过css-loader，可以实现在js文件中通过import/require的方式，来引入css。

安装`npm install css-loader --save-dev,npm install style-loader --save-dev`。

接着创建一个css文件，并在入口文件里引入该css文件，然后在webpack配置文件中增加以下配置：

<pre class="line-numbers language-glsl">
    module.exports = {
        ···
        module: {
            rules:[
                {
                    test:/\.css$/,
                    loader: 'style-loader!css-loader'
                }
            ]
        },
    }
</pre>

打开打包之后的js文件，就会发现已经把css里的内容一起打包进去了。如果你还想把这些引入的相关assets 文件(如 css, js)通过inline的方式直接引入到index.html里，你可以试试我自己写的这个webpack插件 [assets-inline-plugin]()。有关这个插件的使用和详细过程，可以看我的这篇文章 [自己的第一个NPM包--assets-inline-plugin](http://tiquiero.tech/2018/06/01/assets-inline-plugin~%E7%AC%AC%E4%B8%80%E4%B8%AAWebpack%E6%8F%92%E4%BB%B6/)。

好了，这些就是webpack的基本使用，你可以自己去发掘配置各种选项，打造一个完美的打包工具~