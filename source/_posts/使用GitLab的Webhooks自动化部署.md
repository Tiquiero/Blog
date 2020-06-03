---
title: 使用GitLab的Webhooks自动化部署 # 文章标题  
date: 2018/11/28 19:41:05  # 文章发表时间
tags:
- Webhooks
categories: Webhooks # 分类
thumbnail: https://pic.superbed.cn/item/5c7e463c3a213b0417461d31 # 略缩图
---

之前用Gitbook做了一个公司内部的文档中心，然后部署在了一台用作部门服务器的Mac上。因为公司用的svn，Mac上使用svn不方便所以就用的公司内部的私有GitLab仓库。之前一直都是在本地开发代码提交到GitLab上，再去服务机上更新代码，觉得每次这样实在太麻烦了，就去了解了下自动化部署，就发现了GitLab的Webhooks。

Webhooks是个在特定情况下触发的一种api，也叫钩子，GitLab中解释: Webhooks钩子用于在项目发生相关事件时通知外部服务器。Github也有类似的，叫Git Hooks。Git Hooks就是那些在Git执行特定事件（如commit、push、receive等）后触发运行的脚本。GitLab的webhooks跟git hook类似。也是当项目发生提交代码、提交tag等动作会自动去调用url，这个url可以是更新代码，或者其他操作。

### 设置Webhooks

登陆GitLab后，在项目Setting页面，找到Web Hooks：

![02](https://pic.superbed.cn/item/5c7e463c3a213b0417461d2f)

URL就是需要部署到的服务器的网址，之后每次有相关事件发生的时候， GitLab 都会主动往这个地址发送一个 POST 请求，当然你也可以选择任何事件都发个 POST 通知你。下面会详说。然后选择触发该钩子的事件，GitLab支持五种事件。添加Web Hooks后，在下面的列表后面，还有个TEST HOOK ,可以测试这个钩子是否可以。

### 自动化部署代码

首先要写一个自动拉取GitLab上代码的shell脚本 `deploy.sh`

<pre class="line-numbers language-glsl">
    PROJECT_DIR='/data/projects/onlineDoc'    # 等号左右不能有空格,这是你项目在服务器上的存储路径

    echo 'start'
    cd $PROJECT_DIR

    echo 'pull code'
    git reset --hard origin/master && git clean -f
    git pull && git checkout master

    echo 'finished'
</pre>

接下来就要写一个监听程序deploy.js去接受请求了，因为跑的node，所以可以找一个node的中间件，[node-gitlab-webhook](https://www.npmjs.com/package/node-gitlab-webhook)(Github也有类似的[github-webhook-handler](https://www.npmjs.com/package/github-webhook-handler))。

首先起一个服务，createHandler里的path就是上面设置Web Hooks里的URL里的服务器ip后的path，监听端口可以自己设，这个端口也就是上面URL里的端口，比如我这里的是7777，URL最后就是 http:your server ip:7777/path

<pre class="line-numbers language-glsl">
    var http = require('http')
    var createHandler = require('node-gitlab-webhook')
    var handler = createHandler({ path: '/incoming', secret: '123456'}) // 一定要有密码参数,如果GitLab那边没有设置随便写就行
    
    http.createServer(function (req, res) {
        handler(req, res, function (err) {
            res.statusCode = 404
            res.end('no such location')
        })
    }).listen(7777)
</pre>

然后注册push事件：

<pre class="line-numbers language-glsl">
    handler.on('push', function (event) {
        console.log(
            'Received a push event for %s to %s',
            event.payload.repository.name,
            event.payload.ref
        )
        execFunc('sh ./deploy.sh')
    })  

    function execFunc(content) {
        var exec = require('child_process').exec
        exec(content, function(error, stdout, stderr) {
            if (error) {
            console.error('exec error:' + error)
            return
            }
            console.log('stdout:' + stdout)
            console.log('stderr:' + stderr)
        })
    }
</pre>

完整代码可以看 [这里](https://github.com/Tiquiero/Webhooks)。

接下来就可以把服务跑起来了，为了防止node的服务自己挂掉导致不能访问文档中心，我又用了进程管理服务[PM2](http://pm2.keymetrics.io/docs/usage/quick-start/)

<pre class="line-numbers language-glsl">
    npm install pm2@latest -g
    npm start deploy.js
</pre>

这样就算这段node代码某处出错挂了，它也会自动重新启动一个进程，保证服务仍可用。

到此GitLab的Webhooks自动化部署就完成了，以后只需要在本地提交代码，服务器就会自动更新代码并部署。