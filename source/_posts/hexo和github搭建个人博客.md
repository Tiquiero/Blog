---
title: Hexo + Github搭建个人博客 # 文章标题  
date: 2018/8/17  # 文章发表时间
tags:
- Blog/Hexo
categories: Blog/Hexo # 分类
thumbnail: https://pic.superbed.cn/item/5c7e48373a213b04174644ef # 略缩图
---

本文记录了如何使用hexo配合github来创建个人博客，以及如何使用hexo官网上的主题装饰自己的博客。

# Hexo

Hexo 是一个快速、简洁且高效的博客框架。Hexo 使用 Markdown（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。安装 Hexo 相当简单，安装前默认你已经安装了Node.js和Git(这里暂不详细讲解如何安装这些)。

安装Hexo：`$ npm install -g hexo-cli`

安装 Hexo 完成后，请执行下列命令，Hexo 将会在指定文件夹中新建所需要的文件。

    $ hexo init <folder>
    $ cd <folder>
    $ npm install

新建完成后，指定文件夹的目录如下：
<pre class="line-numbers language-glsl">
        .
        ├── _config.yml           网站的配置信息
        ├── package.json          应用程序的信息,EJS, Stylus 和 Markdown renderer 已默认安装,可以自己移除
        ├── scaffolds             模版文件夹。新建文章时，Hexo 会根据 scaffold 来建立文件
        ├── source                资源文件夹
        |   ├── _drafts
        |   └── _posts            文章 文件夹。除 _posts 文件夹之外，开头命名为 _ (下划线)的文件 / 文件夹和隐藏的文件将会被忽略。
        └── themes                主题 文件夹
</pre>

关于每个文件里的具体参数介绍，可以查看hexo官网的 [配置介绍](https://hexo.io/zh-cn/docs/configuration.html)。

#### Hexo常用基本指令

<pre class="line-numbers language-glsl">
    $ hexo init [folder]                        新建网站
    $ hexo new [layout] title                   新建一篇文章。如果没有设置 layout 的话，默认使用 _config.yml 中的 default_layout 参数代替
    $ hexo generate (hexo g)                    生成静态文件
        参数：
        ├── --deploy(hexo g -d)	             文件生成后立即部署网站
        ├── --watch(hexo g -w) 	             监视文件变动
    $ hexo publish [layout] filename            发表草稿
    $ hexo server  (hexo s)                     启动服务器。默认情况下，访问网址为： http://localhost:4000/
        参数：
        ├── --port,(hexo server -p 4001)	    重设端口
        ├── --static(hexo server -s)	        只使用静态文件
        ├── --log(hexo server -l)	           启动日记记录，使用覆盖记录格式
    $ hexo deploy                               部署网站
        参数：
        ├── --generate(hexo d -g)               部署之前预先生成静态文件
    $ hexo clean                                清除缓存文件 (db.json) 和已生成的静态文件 (public)。(更换主题后需要clean一下)
</pre>

#### 首次体验Hexo

在命令行中输入命令：`hexo g`

![02](https://pic.superbed.cn/item/5c7e484a3a213b04174646aa)

再输入命令启动服务：`hexo s`，然后在浏览器中打开 http://localhost:4000/， 你将会看到：

![03](https://pic.superbed.cn/item/5c7e484a3a213b04174646ad)

到目前为止，Hexo在本地的配置已经基本结束了。

# Github部分

#### 创建代码库

在该部分开始之前，默认已经有个人的github账号并且配置好了git个人信息。

1.登陆进入github后，点击页面右上角的加号，选择New repository，新建一个代码库。

2.在Repository name下填写`yourname.github.io`，Description (optional)下填写一些简单的描述（不写也没有关系），如图所示：
![04](https://pic.superbed.cn/item/5c7e484a3a213b04174646b0)

**注意：比如我的github名称是Tiquiero ,这里你就填 `Tiquiero.github.io`,如果你的名字是abc，那你就填 `abc.github.io`**

3.正确创建之后，你将会看到如下界面：(这边借用了网上的一张截图)

![05](https://pic.superbed.cn/item/5c7e484a3a213b04174646b2)

接下来开启gh-pages功能，点击界面右侧的Settings，你将会打开这个库的setting页面，向下拖动，直到看见GitHub Pages，如图：

![06](https://pic.superbed.cn/item/5c7e484a3a213b04174646b5)

点击Launch automatic page generator，Github将会自动替你创建出一个gh-pages的页面。 如果你的配置没有问题，那么大约15分钟之后，yourname.github.io这个网址就可以正常访问了~ 如果yourname.github.io已经可以正常访问了，那么Github这边的配置已经全部结束了。

# Hexo连接Github Page

#### 配置Git个人信息

默认你已经配置好本机与自己github的个人信息了。[Git个人配置教程](http://www.runoob.com/w3cnote/git-guide.html)

#### 配置Deployment

在_config.yml文件中，找到Deployment，然后按照如下修改：

![07](https://pic.superbed.cn/item/5c7e488b3a213b0417464ca8)

其实到此，你的hexo已经和你的GitHub page关联在一起了。

#### 写博客、发布文章

执行命令：`hexo new post "first page"`
这时候在 hexo\source\ _posts 目录下，会看到自动新建的 `first-page.md` 文件，这个就是博客文章文件。用MarDown编辑器或者其他编辑器打开就可以编辑文章了。文章编辑好之后，运行生成、部署命令：

    hexo g   // 生成
    hexo d   // 部署

    ###或者

    hexo d -g

#### Tips

这里注意需要提前安装一个扩展：`npm install hexo-deployer-git --save`
如果没有安装这个扩展，将会提醒：`deloyer not found:git`

部署成功后，相应的代码已经上传至你的github代码仓库，访问你的地址：`https://yourname.github.io`（例如我的： https://tiquiero.github.io ),就可以看到生成的文章了。

到此，你就已经成功的使用Hexo和GitHub搭建出一个个人博客了。

# Hexo主题使用

Hexo的官方网站上有很多漂亮的[主题](https://hexo.io/themes/)可供下载，如果不想自己麻烦写的话，可以直接下载喜欢的主题，套在自己的博客上，非常方便快捷。每个主题作者一般都会有个详细的使用配置文档，以我自己使用的为例：

1.下载喜欢的主题代码，并放置于themes目录下。

2.在站点根目录的配置文件_config.yml中启用该主题，如图所示：

![08](https://pic.superbed.cn/item/5c7e488b3a213b0417464caa)

3.在你下载的主题代码文件夹里，找到主题配置文件_config.yml，根据使用文档修改想要的主题配置。（因为每个主题可能配置项不同，这里就不详细讲解，以作者的文档为准）。

# 绑定私有域名

#### 域名配置

如果你想给自己的博客一个专属的私有域名，而不是xxxx.github.io的话，你可以把你的私有域名映射到你的GitHub Pages，然后就能使用你自己的域名（例如我自己的：[wwww.tiquiero.tech](wwww.tiquiero.tech)）访问了。

域名购买可以去很多不错的平台选购，我是去的阿里云。因为.com,.cn这些域名现在都要备案了，所以我选择了.tech域名（其实主要是便宜，1元/年:smirk:）。购买了之后一定要先去实名认证一下，不然域名是无法使用的（我一开始就没有实名认证，所有配置好了以后域名也一直访问不了，阿里云也没给我哪里提示，害我一直以为是配置问题:weary:）。

认证了之后，在阿里云控制台左侧菜单栏选择 域名与网站（万网）--> 云解析DNS ，选择进入域名的解析设置：

![09](https://pic.superbed.cn/item/5c7e488b3a213b0417464cad)

方法1：使用IP设置。你可以直接选择新手引导，会自动帮你添加两条解析：

![10](https://pic.superbed.cn/item/5c7e488b3a213b0417464caf)

记录值就是填写你的GitHub Pages的IP地址，IP可直接ping yourname.github.io来获取：

![11](https://pic.superbed.cn/item/5c7e488b3a213b0417464cb4)

方法2：使用CNAME设置，自己添加两条解析记录：

![12](https://pic.superbed.cn/item/5c7e48cb3a213b0417465265)

记录类型都是CNAME，主机记录分别是www，@，记录值都是yourname.github.io。

#### Tips

记录类型：

    A记录： 将域名指向一个IPv4地址（例如：8.8.8.8）
    CNAME：将域名指向另一个域名（例如www.qcloud.com）
    MX： 将域名指向邮件服务器地址
    TXT： 可任意填写，长度限制255，通常做SPF记录（反垃圾邮件）
    NS： 域名服务器记录，将子域名指定其他DNS服务器解析
    AAAA：将域名指向一个iPv6地址（例如：ff06:0:0:0:0:0:0:c3）
    SRV：记录提供特定服务的服务器（例如_xmpp-server._tcp）

然后域名这边的配置就OK啦~

#### Github配置

1.在你博客的根目录下的source文件夹下建立一个叫CNAME的文件，没有后缀，里面的内容写上你的域名xxxx.com，比如我的是：tiquiero.tech。

2.进入你的GitHub，找到设置 Setting，找到 Custom domain ，添加你的域名后保存即可。

OK，重新hexo d -g发布一下，过10分钟后你就可以通过你的域名来访问博客啦~

