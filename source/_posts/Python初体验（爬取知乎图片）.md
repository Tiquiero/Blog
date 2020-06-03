---
title: Python初体验（爬取知乎图片） # 文章标题  
date: 2019/1/7 20:05:05  # 文章发表时间
tags:
- Python
categories: Python # 分类
thumbnail: https://pic.superbed.cn/item/5c7e48f03a213b04174654db # 略缩图
---

首先，我们来看几个知乎的问题 :sunglasses:：

[当一个颜值很高的程序员是怎样一番体验？](https://www.zhihu.com/question/37787176/answer/81607754)

[有哪张不知名的妹子照片让你特别心动念念不忘珍藏至今，一直想知道她的名字？](https://www.zhihu.com/question/267460120)

是不是看的津津有味，不能自拔:heart_eyes:。（知乎看片指日可待，这样赤裸裸的钓鱼贴，是越来越多了:smiling_imp:）但是我们作为程序员，滚动这一个个回答去找寻漂亮小姐姐的照片是不是有点浪费时间又有失程序员的身份呢。所以接下来，我们得用程序员的方式去看小姐姐。

# 初识Python

### 什么是python

Python是用来编写应用程序的高级编程语言，有非常完善的基础代码库，覆盖了网络、文件、GUI、数据库、文本等大量内容。Python适合开放网络应用，包括网站、后台服务等等；其次是许多日常需要的小工具，包括系统管理员需要的脚本任务等等；

缺点：第一个缺点就是运行速度慢，和C程序相比非常慢，因为Python是解释型语言，你的代码在执行时会一行一行地翻译成CPU能理解的机器码，这个翻译过程非常耗时，所以很慢。而C程序是运行前直接编译成CPU能执行的机器码，所以非常快。

第二个缺点就是代码不能加密。如果要发布你的Python程序，实际上就是发布源代码，这一点跟C语言不同，C语言不用发布源代码，只需要把编译后的机器码（也就是你在Windows上常见的xxx.exe文件）发布出去。要从机器码反推出C代码是不可能的，所以，凡是编译型的语言，都没有这个问题，而解释型的语言，则必须把源码发布出去。

### 安装Python

目前，Python有两个版本，一个是2.x版，一个是3.x版，这两个版本是不兼容的。从Python的官方网站下载Python 3.6对应的64位安装程序或32位安装程序，然后，运行下载的EXE安装包：

![02](https://pic.superbed.cn/item/5c7e48f03a213b04174654d9)

### 命令行模式和Python交互模式

在命令行模式运行.py文件和在Python交互式环境下直接运行Python代码有所不同。Python交互式环境会把每一行Python代码的结果自动打印出来，但是，直接运行Python代码却不会。

Python交互模式的代码是输入一行，执行一行，而命令行模式下直接运行.py文件是一次性执行该文件内的所有代码。所以，Python交互模式主要是为了调试Python代码用的，便于初学者学习，它不是正式运行Python代码的环境。

### Python基础

#### 数据类型：

Python内置的一种数据类型是列表：list。list是一种有序的集合，可以随时添加和删除其中的元素。用len()函数可以获得list元素的个数。
另一种有序列表叫元组：tuple。tuple和list非常类似，但是tuple一旦初始化就不能修改。

#### 条件判断：

<pre class="line-numbers language-glsl">
    s = input('birth: ')
    birth = int(s)
    if birth < 2000:
        print('00前')
    else:
        print('00后')
</pre>

#### 循环：

<pre class="line-numbers language-glsl">
    sum = 0
    for x in [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]:
        sum = sum + x
    print(sum)
</pre>

#### 函数：

Python内置了很多有用的函数，如求绝对值的函数abs()，int()函数可以把其他数据类型转换为整数，range()函数可以生成一个整数序列等等。

自定义函数：定义一个函数要使用def语句，依次写出函数名、括号、括号中的参数和冒号:，然后，在缩进块中编写函数体，函数的返回值用return语句返回。

<pre class="line-numbers language-glsl">
    def my_abs(x):
        if x >= 0:
            return x
        else:
            return -x
</pre>

空函数：
如果想定义一个什么事也不做的空函数，可以用pass语句，缺少了pass，代码运行就会有语法错误：

    def nop():
        pass

#### 常用第三方库

requests：用于访问网络资源，Python也有内置的urllib2模块，但用起来比较麻烦，而且，缺少很多实用的高级功能。

`$ pip install requests`

通过GET访问一个页面，只需要几行代码：

<pre class="line-numbers language-glsl">
    >>> import requests
    >>> r = requests.get('https://www.douban.com/') # 豆瓣首页
    >>> r.status_code
    >>> r.text
</pre>

requests的方便之处还在于，对于特定类型的响应，例如JSON，可以直接获取：

<pre class="line-numbers language-glsl">
    >>> r = requests.get('https://query.yahooapis.com/v1/public/yql?q=select%20*%20from%20weather.
    forecast%20where%20woeid%20%3D%202151330&format=json')
    >>> r.json()
    {'query': {'count': 1, 'created': '2017-11-17T07:14:12Z', ...
</pre>

requests对Cookie做了特殊处理，使得我们不必解析Cookie就可以轻松获取指定的Cookie：

<pre class="line-numbers language-glsl">
    >>> r.cookies['ts']
    'example_cookie_12345'
</pre>

Pillow：Python平台上的图像处理标准库。

最常见的图像缩放操作：

<pre class="line-numbers language-glsl">
    from PIL import Image

    # 打开一个jpg图像文件，注意是当前路径:
    im = Image.open('test.jpg')
    # 获得图像尺寸:
    w, h = im.size
    print('Original image size: %sx%s' % (w, h))
    # 缩放到50%:
    im.thumbnail((w//2, h//2))
    print('Resize image to: %sx%s' % (w//2, h//2))
    # 把缩放后的图像用jpeg格式保存:
    im.save('thumbnail.jpg', 'jpeg')
</pre>

# Python爬虫

#### 什么是爬虫

网络爬虫，是一种自动化程序，去自动帮我们匹配到网络上面的数据，然后下载下来，为我们所用。所以网络爬虫的基本操作是抓取网页。每个网页都是一份 HTML文档，所以爬虫的思路其实就是，模拟浏览器，发送一份请求，获得这份HTML文档，再匹配抽取出我们需要的内容。

#### requests简单爬虫

[Requests: HTTP for Humans](http://docs.python-requests.org/zh_CN/latest/user/quickstart.html#id2)

### Python爬取知乎图片

<pre class="line-numbers language-glsl">
    import requests
    url='https://www.zhihu.com/question/37787176'
    r = requests.get(url)
    r.text
</pre>


很明显，知乎服务器发现你是爬虫了，而且它还很恶意的把你的连接挂了很久,刚刚那段代码会很久才跑出结果。那怎么发现的呢？浏览器在发送请求的时候，通常会带上一个User-Agent，这个字符串里通常会包含操作系统信息和浏览器的一些信息。所以只是这样的话，我们的user-agent就是默认的"python-requests/1.2.0"。

<pre class="line-numbers language-glsl">
    import requests
    header = {'User-Agent':'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6) AppleWebKit/537.36 
    (KHTML, like Gecko) Chrome/59.0.3071.115 Safari/537.36'}
    url='https://www.zhihu.com/question/37787176'
    r = requests.get(url,headers=header)
    r.text
</pre>

返回的一大坨HTML怎么办，我们只需要图片，那就做个最简单的正则匹配就好了。看知乎的图片链接都是用https://开头以.jpg等等后缀结尾，中间任意字符。最后拿到就拿到了这样的式子（_r是原图，而以_l,_xs,_is,结尾的都是头像）：

<pre class="line-numbers language-glsl">
    jpg = re.compile(r'https://[^\s]*?_r\.jpg')
    jpeg = re.compile(r'https://[^\s]*?_r\.jpeg')
    gif = re.compile(r'https://[^\s]*?_r\.gif')
    png = re.compile(r'https://[^\s]*?_r\.png')
</pre>

其实到这里已经是能获取到回答下面的图片的了，但是发现只能获取一小部分。因为那只是最前面几条答案的图片，知乎对于后面的所有回答并没有一次性加载出来，而需要点击底部的查看所有回答才会去加载，所以我们也得模拟浏览器执行这个操作。先看看点击加载后面的回答时发生了什么。

cURL是一个利用URL语法在命令行下工作的文件传输工具，绝大多数的linux 和unix都会内置的一个软件.
但是，注意windows下命令行里调用的curl并不是这里提到的那个curl，具体的讨论可以看这篇[：PowerShell 为什么 alias 了 curl 就引起了如此大的争议](https://www.zhihu.com/question/49839038)

<pre class="line-numbers language-glsl">
    curl
    'https://www.zhihu.com/api/v4/questions/22212644/answers
    ?sort_by=default
    &include=data%5B%2A%5D.is_normal%2Cis_collapsed%2Cannotation_action%2Cannotation_detail%2Ccollapse_reason%2Cis_sticky%2Ccollapsed_by%2Csuggest_edit%2Ccomment_count%2Ccan_comment%2Ccontent%2Ceditable_content%2Cvoteup_count%2Creshipment_settings%2Ccomment_permission%2Cmark_infos%2Ccreated_time%2Cupdated_time%2Creview_info%2Crelationship.is_authorized%2Cis_author%2Cvoting%2Cis_thanked%2Cis_nothelp%2Cupvoted_followees%3Bdata%5B%2A%5D.author.follower_count%2Cbadge%5B%3F%28type%3Dbest_answerer%29%5D.topics
    &limit=20
    &offset=3'
    -H 
    'Cookie: 
        aliyungf_tc=AQAAADvok2VGQgEAiE1NfJc49LzrUk7p;   q_c1=ef155c9cc9124e1f951447b3b6fd875f|1500990779000|1500990779000; _zap=84936da1-3813-4842-a846-56214e791bd0;
        l_n_c=1;
        l_cap_id="NDczNThhZjExZWU1NDIzYWJhMDllNjBkZTJiNDUwZGI=|1500990807|075d561c18784c75946ae101e6f7135b5b271cf3";
        r_cap_id="Y2MwNzdlMjMxMjc0NDViMGE0NWJhOTI3NGQzNjQwNGY=|1500990807|769b04b121d8b52602f6d7f3e86ca62bf5f80d33";    cap_id="ZTFkYmRiMjZiZTgyNDdjY2I3YTk3ZDExNDZjZDUyYmY=|1500990807|2f85fb4342ee06cfe26e5ba006d699c778a7787c";
        n_c=1;
        _xsrf=b967f17b-4577-428e-906d-1c306c67decd'
    -H 'Accept-Encoding: gzip, deflate, br'
    -H 'Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.6,en;q=0.4'
    -H 'authorization: oauth c3cef7c66a1843f8b3a9e6a1e3160e20'
    -H 'accept: application/json, text/plain, */*'
    -H 'Referer: https://www.zhihu.com/question/22212644'
    -H 'User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/59.0.3071.115 Safari/537.36'
    -H 'Connection: keep-alive'
</pre>

curl的第一参数就是要请求的url， -H表示一个Header，所以，这个请求总共有8个Header,其中一个是cookie。对，我们要让服务器给我们设置cookie。我们可以看一下服务器设置了哪些cookie。

_xsrf这样的字段通常是用来防止XSRF(跨站请求伪造)的，而且aliyungf_tc，从名称上来看，应该是阿里云高防塞的。那些充斥着大量连字符、意义不明的cookie，都可以忽略，这些通常是在某些中间步骤被设置进去的。

我们上次用到的requests库，提供了一个requests.Session()对象，可以用来很方便的模拟一次会话，它会根据服务器的返回自动设置cookie。

<pre class="line-numbers language-glsl">
    def init(url):
        ua = {'User-Agent':'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/59.0.3071.115 Safari/537.36'}
        s = requests.Session()
        s.headers.update(ua)
        ret=s.get(url)
        s.headers.update({"authorization":"Bearer Mi4xaHNqWEFRQUFBQUFBSUt6am9hdzFEUmNBQUFCaEFsVk5uV1dDV3dDVUJSRm9jYVVXV2FiREZfWHZHMDM0b081Vzd3|1519720350|295ebdcd1530503e6caf61a5942fc1dba85ca363"})
        return s
</pre>
authorization这个请求头字段要手动加上。[authorization](http://blog.csdn.net/u011181633/article/details/43229387)。

HTTP协议中的 Authorization 请求消息头含有服务器用于验证用户代理身份的凭证，Authorization header的典型数据为"Authorization: Basic jdhaHY0="，其中Basic表示基础认证。

接下来我们就可以写获取答案的函数了：

<pre class="line-numbers language-glsl">
    def fetch_answer(s,qid,limit,offset):
        params={
            'sort_by':'default',
            'include':'data[*].is_normal,is_collapsed,annotation_action,annotation_detail,collapse_reason,is_sticky,collapsed_by,suggest_edit,comment_count,can_comment,content,editable_content,voteup_count,reshipment_settings,comment_permission,mark_infos,created_time,updated_time,review_info,relationship.is_authorized,is_author,voting,is_thanked,is_nothelp,upvoted_followees;data[*].author.follower_count,badge[?(type=best_answerer)].topics',
            'limit':limit,
            'offset':offset
        }
        url ="https://www.zhihu.com/api/v4/questions/"+qid+"/answers"
        return s.get(url,params=params)
</pre>

qid,就是url后面的那一串数字question_id,limit表示一次拉去多少个回答、offset表示从第几个开始。为什么limit不直接设置999999来直接获取全部回答呢？这么做当然可以，但是这样的行为十分反常，而且很可能触发某些反爬虫机制，因为这个特征太明显了。牢记一点，你是一个浏览器。（实际上知乎会把大于20的limit当做20去处理）。

获取所有答案：

<pre class="line-numbers language-glsl">
    def fetch_all_answers(url):
        session = init(url)
        q_id = url.split('/')[-1]
        offset = 0
        limit=20
        answers=[]
        is_end=False
        while not is_end:
            ret=fetch_answer(session,q_id,limit,offset)
            #total = ret.json()['paging']['totals']
            answers+=ret.json()['data']
            is_end= ret.json()['paging']['is_end']
            print("Offset: ",offset)
            print("is_end: ",is_end)
            offset+=limit
        return answers
</pre>

我们可以用知乎返回数据paging里的is_end字段，来判断是否获取完毕。最后，我们拿到了一个answer的数组，从每个answer里的'content'字段里找出图片url，循环下载就好了。

<pre class="line-numbers language-glsl">
    url = "https://www.zhihu.com/question/29814297"
    answers=fetch_all_answers(url)
    folder = '29814297'
    for ans in answers:
        imgs = grep_image_urls(ans['content'])
        for url in imgs:
            download(folder,url)
</pre>

# node版本下的爬虫

主要依赖包：

[cheerio](https://cnodejs.org/topic/5203a71844e76d216a727d2e)，可以让我们很方便的处理爬虫爬取的数据。

[superagent](https://cnodejs.org/topic/5378720ed6e2d16149fa16bd)，一个非常方便的客户端请求代理模块。

### Tips

从上述两种爬取方式可以明显的感觉出，当图片数量大的时候，NodeJs方式明显比python效率快的多，相同时间内下载的图片也更多。这里就又涉及到NodeJs和python两者在不同方面下爬虫性能的讨论了。

[爬虫性能：NodeJs VS Python](http://python.jobbole.com/86257/)

在粗略的了解过后，总而言之：

NodeJS：异步的，效率高，对一些垂直网站爬取很可以，但分布式爬取、消息通讯等支持较弱。

python：简洁，如果你对效率没有极端的要求。对于上规模的整站爬取，有专门的爬虫框架Scrapy。Python非常适合做数据的处理，比如函数参数的打包解包，列表解析，矩阵处理，非常方便。

Node的优势在于HTML解析和异步IO，python用上多线程在下载速度能够比过node。

