---
title: assets-inline-plugin~第一个NPM包/Webpack插件 # 文章标题  
date: 2019/11/01 14:42:05  # 文章发表时间
tags:
- Webpack
categories: Webpack # 分类
thumbnail: https://pic.superbed.cn/item/5c7e46fd3a213b0417462bec # 略缩图
---

没有发布过npm包的同学，可能会对NPM对开发有一种蜜汁敬畏，觉得这是一个很高大上的东西。其实在了解了之后发现，npm包就是一个我们平时经常写的一个export出来的模块而已，只不过跟其它业务代码耦合性低，具有较高的独立性。然后在 [Webpack初体验](http://tiquiero.tech/2018/05/24/Webpack%E5%88%9D%E4%BD%93%E9%AA%8C/) 的时候，在使用html-webpack-plugin插件时刚好有个想法，能不能把引用的相关 assets 文件(如 css, js)以inline的方式引入进文件里。所以就决定尝试一下如何编写一个webpack插件，并且发布在NPM上，也就有了这个 [assets-inline-plugin](https://www.npmjs.com/package/assets-inline-plugin)。

### 如何写一个webpack插件

首先我们要先去了解两个对象，[compiler](https://www.webpackjs.com/api/compiler-hooks/)和[compilation](https://www.webpackjs.com/api/compilation-hooks/)：

compiler对象包涵了Webpack环境所有的的配置信息，这个对象在Webpack启动时候被构建，并配置上所有的设置选项包括 options，loaders，plugins。当启用一个插件到Webpack环境的时候，这个插件就会接受一个指向compiler的参数。运用这个参数来获取到Webpack环境。更多详细可看 [源代码](https://github.com/webpack/webpack/blob/master/lib/Compiler.js)。

compilation代表了一个单一构建版本的物料。在webpack中间件运行时，每当一个文件发生改变时就会产生一个新的compilation从而产生一个新的变异后的物料集合。compilation列出了很多关于当前模块资源的信息，编译后的资源信息，改动过的文件，以及监听过的依赖。compilation也提供了插件需要自定义功能的回调点。更多详细可看 [源代码](https://github.com/webpack/webpack/blob/master/lib/Compilation.js)。

#### 插件基本结构

官方教程基本结构：

<pre class="line-numbers language-glsl">
  function HelloWorldPlugin(options) {
    // Setup the plugin instance with options...
  }

  HelloWorldPlugin.prototype.apply = function(compiler) {
    compiler.plugin('done', function() {
      console.log('Hello World!'); 
    });
  };

  module.exports = HelloWorldPlugin;
</pre>

options是你在webpack配置文件里给插件的配置项参数。Plugins是可以用自身原型方法apply来实例化的对象，apply这个函数是提供给webpack运行时调用的，webpack会在这里注入compiler对象。compiler有很多生命周期钩子函数，除了done还有emit,run,afterEmit等等，我们可以在具体某个生命周期里去做相应的操作。

assets属性是compilation里最重要的部分，因为如果你想借助webpack帮你生成文件，你需要像官方教程里那样在assets上写上对应的文件信息。

或者你可以不用函数的方式，用定义一个类的方式也行，比如我这个插件就是这样的结构：

<pre class="line-numbers language-glsl">
  class HelloWorldPlugin{
    constructor(options){

    }
    apply(compiler){
      compiler.plugin('compilation',compilation => {

      });
    }
  }

  module.exports = HelloWorldPlugin;
</pre>

### assets-inline-plugin

html-webpack-plugin是会把打包的资源js文件以script标签的形式自动引入模板页里，现在我们要换成inline的形式去引入的话，其实最核心的操作就是在html-webpack-plugin插件调用模板去插入资源文件之前更改资源。

#### 参数判断

因为这个插件比较简单，所以只设定有两个参数inline和remove。inline是需要包含的资源形式配置项，是需要inline js还是css资源文件，取值可以是boolean or regexp or object，默认是为true，格式可以如下：

    inline: true
    inline: /\.(css|js)$/
    inline: { css: /\.css$/, js: /\.js$/ }
    inline: { css: true, js: true }


remove表示是否需要清除inline进来了的文件，默认为true。

首先我们需要判断配置的参数，代码如下：

<pre class="line-numbers language-glsl">
  class HtmlInlinePlugin {
    constructor(options) {
      isObject(options) || (options = {});

      if (!isObject(options.inline)) {
        if (options.hasOwnProperty('inline')) {
          let inline = isRegExp(options.inline) ? options.inline : !!options.inline;
          options.inline = { js: inline, css: inline };
        }
        else options.inline = { js: true, css: true };
      }
      else {
        // 取反操作“!”会得到与目标对象代表的布尔型值相反的布尔值，而再做一次取反“!!”就得到了与其相同的布尔值。
        options.inline.js = isRegExp(options.inline.js) ? options.inline.js : !!options.inline.js;
        options.inline.css = isRegExp(options.inline.css) ? options.inline.css : !!options.inline.css;
      }

      this.outDir = '';
      this.inlineAsserts = [];
      if(options.remove){
        this.options = Object.assign({}, options);
      }else{
        this.options = Object.assign({ remove: true }, options);
      }
  }
  apply(compiler) {
    ···
  }

  function isObject(obj) {
    return Object.prototype.toString.call(obj) === '[object Object]';
  }

  function isRegExp(obj) {
    return Object.prototype.toString.call(obj) === '[object RegExp]';
  }
</pre>

#### 修改打包资源

对文件的更改操作在apply这个方法里去执行，主要是通过compilation这个对象来操作。在调用模板之前更改资源，我们借助了html-webpack-plugin的插件事件 [html-webpack-alter-asset-plugin](https://www.npmjs.com/package/html-webpack-alter-asset-plugin)。

html-webpack-plugin允许其他插件来改变html文件内容。具体的事件如下：

    Async（异步事件）:
        * html-webpack-plugin-before-html-generation
        * html-webpack-plugin-before-html-processing
        * html-webpack-plugin-alter-asset-tags
        * html-webpack-plugin-after-html-processing
        * html-webpack-plugin-after-emit

    Sync（同步事件）:
        * html-webpack-plugin-alter-chunks

这个插件事件包含的参数htmlPluginData有相应的插件数据，数据里的head是一个数组，我们通过遍历这个数组来在文件头部插入css资源：

<pre class="line-numbers language-glsl">
  pluginData.head = pluginData.head.map(tag => {
      let assetUrl = tag.attributes.href;
      if (this.testAssertName(assetUrl, this.options.inline.css)) {
        tag = { tagName: 'style', closeTag: true ,attributes: {type: 'text/css'}};
        this.updateTag(tag, assetUrl, compilation);
      }

      return tag;
  });
</pre>

同理，我们循环插件数据里的body数组，来在文件主体插入js资源：

<pre class="line-numbers language-glsl">
    pluginData.body = pluginData.body.map(tag => {
      let assetUrl = tag.attributes.src;
      if (this.testAssertName(assetUrl, this.options.inline.js)) {
        tag = { tagName: 'script', closeTag: true ,attributes: {type: 'text/javascript'}};
        this.updateTag(tag, assetUrl, compilation);
      }

      return tag;
    });
</pre>

插入这些资源文件后，我们需要把这些更新进compilation里。assetUrl分别是获取到css的href和js的src路径：

<pre class="line-numbers language-glsl">
    updateTag(tag, assetUrl, compilation) {
        let publicUrlPrefix = compilation.outputOptions.publicPath || '';

        //path.posix.relative(from, to) 方法返回从 from 到 to 的相对路径（基于当前工作目录）
        //在 POSIX 上：path.relative('/data/orandea/test/aaa', '/data/orandea/impl/bbb');
        // 返回: '../../impl/bbb'
        let assetName = path.posix.relative(publicUrlPrefix, assetUrl);
        let asset = compilation.assets[assetName];

        let source = asset.source();
        if (typeof source !== 'string') {
          source = source.toString();
        }

        // remove sourcemap comments
        tag.innerHTML = sourceMappingURL.removeFrom(source);

        // mark inlined asserts which will be deleted lately
        this.inlineAsserts.push(assetName);

        return tag;
      }
</pre>

最后，我们把内联进来的资源原文件给删除，

<pre class="line-numbers language-glsl">
  removeInlineAsserts() {
      if (!this.outDir) return;

      this.inlineAsserts.forEach(file => {
        let filePath = path.join(this.outDir, file);

          //如果路径存在
        if (fs.existsSync(filePath)) {
            //删除文件
          fs.unlinkSync(filePath);
        }

        rmdirSync(this.outDir, file);
      })
  }
</pre>

该函数是路径下一层层循环判断文件是否存在：

<pre class="line-numbers language-glsl">
  function rmdirSync(outDir, file) {
    //path.sep平台特定的路径片段分隔符，Windows 上是 \
    file = path.join(file).split(path.sep);
    file.pop();

    if (!file.length) return;

    file = file.join(path.sep);
    let dirPath = path.join(outDir, file);

    if (fs.existsSync(dirPath)) {
      //同步读取目录，返回一个所包含的文件和子目录的数组
      let files = fs.readdirSync(dirPath);

      if (!files.length) {
        fs.rmdirSync(dirPath);
      }
    }
    rmdirSync(outDir, file);
  }
</pre>

### 创建和发布NPM包

首先去 [NPM](https://www.npmjs.com) 注册一个账户。如果是第一次发布包，执行`npm adduser`，然后输入前面注册好的NPM账号，密码和邮箱，将提示创建成功。如果不是第一次发布包，执行`npm login`进行登录，同样输入NPM账号，密码和邮箱。(npm adduser成功的时候默认你已经登陆了，所以不需要再进行npm login了)

接着进入项目文件夹下，然后输入`npm publish`进行发布。

当终端显示如下面的信息时，就代表版本号为1.0.0的包发布成功啦！前往NPM官网就可以查到你的包了。

`+ sugars_demo@1.0.0`

但要注意的是，每次更新时，必须修改版本号后才能更新，比如将1.0.0修改为1.0.1后就能进行更新发布了。
这里的包版本号有一套规则，采用的是semver（语义化版本），通俗点意思就是版本号：大改.中改.小改。

### Tips

有时，我们会看到一些npm包有很漂亮的版本号图标：

![02](https://pic.superbed.cn/item/5c7e466b3a213b04174620d1)

这些图标，其实可以在https://shields.io/ 上去获取图标地址，也可以自定义制作，拉到网站最下面：

![03](https://pic.superbed.cn/item/5c7e466b3a213b04174620d3)

输入相关信息后就会生成图标了：

![04](https://pic.superbed.cn/item/5c7e466b3a213b04174620d5)

然后我们还可以生成有关这个npm包的信息图标，就像这样的：

![05](https://pic.superbed.cn/item/5c7e466b3a213b04174620de)

这个图标我们也可以在上自定义制作,只需要输入你的npm包名，然后选择喜欢的样式就行啦：

![06](https://pic.superbed.cn/item/5c7e466b3a213b04174620e0)

插件完整的代码可以看 [这里](https://github.com/Tiquiero/assets-inline-plugin)。