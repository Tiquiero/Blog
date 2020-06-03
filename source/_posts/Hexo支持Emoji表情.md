---
title: Hexo支持Emoji表情 # 文章标题  
date: 2018/9/24 20:25:05  # 文章发表时间
tags:
- Blog/Hexo
categories: Blog/Hexo # 分类
thumbnail: https://pic.superbed.cn/item/5c7e47dc3a213b0417463f01 # 略缩图
---

写博客的时候发现有些内容不能很好的通过文字表达出来，这时候就需要表情的支持了。但是突然发现Hexo的Markdown里不能像GitHub可以在 Pull Requests, Issues, 提交消息, Markdown 文件里加入表情符（使用方法： :name_of_emoji:）。

查了一下大多数有关Hexo和emoji提到的是hexo-tag-emojis插件，但是这个插件没有对Hexo 3.0提供支持，而且它使用的是 %% 标签选项，这样写起来太麻烦，如果能像Github那样用 :: 就好了。谷歌之后发现可以通过修改Hexo默认的Markdown渲染器来实现渲染emoji。

### 修改渲染器和安装插件

Hexo默认的markdown渲染插件是hexo-renderer-marked，也就是marked渲染器的Hexo版本，这个渲染器不支持插件扩展。在PR中也提到了要支持emoji，但是却迟迟没有marge进来。然后就是另外一个markdown渲染器hexo-renderer-markdown-it，这个渲染器支持插件配置可以使用markdown-it-emoji插件来支持emoji。所以我们要将原来的markdown渲染器换成这个。

首先要在你的Hexo博客目录下卸载原渲染器，安装新的渲染器：

npm un hexo-renderer-marked --save  
npm i hexo-renderer-markdown-it --save

之后再下载markdown-it-emoji插件： `npm install markdown-it-emoji --save`

接着还需要为博客安装twemoji依赖，我们这里使用的emoji表情其实是twitter的[twemoji](https://github.com/twitter/twemoji)。因为该依赖有几百兆之大，所以直接clone或者download都比较慢，后来我用Yarn安装的发现很快。

### 安装Yarn

方法1：你可以下载一份[.msi文件](https://yarnpkg.com/latest.msi)，运行之后将引导你完成 Yarn 的安装。

方法2：通过 Chocolatey 安装，Chocolatey 是一个针对 Windows 平台的包管理工具，你可以根据[这份说明](https://chocolatey.org/install)安装 Chocolatey 。
安装好Chocolatey，可以通过如下指令安装 Yarn 了：`choco install yarn`。

安装完之后可以测试一下Yarn是否安装成功：`yarn --version`

不论哪种方法你都必须事先安装好 Node.js。

安装完依赖后，编辑Hexo的配置文件_config.yml来配置markdown渲染器：

<pre class="line-numbers language-glsl">
    markdown:
    render:
        html: true
        xhtmlOut: false
        breaks: false
        linkify: true
        typographer: true
        quotes: '“”‘’'
    plugins:
        - markdown-it-footnote
        - markdown-it-sup
        - markdown-it-sub
        - markdown-it-abbr
        - markdown-it-emoji
</pre>

重新启动Hexo服务就生效了，输入![image](https://pic.superbed.cn/item/5c7e47dc3a213b0417463efa)会渲染成:smile: :smirk: :relieved:

更多表情代码，可以看[这里](https://www.webpagefx.com/tools/emoji-cheat-sheet/)（可能需要科学上网才能进去哦）。