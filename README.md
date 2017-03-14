# Eagle

> A configurable scraping platform, fitted to every data and source.

## Overview

## Target

* Configurable with code again.
* Use cloud and web to every one.
* Distributed and less source used.
* Easy for operators and powerful for coder.
* Fetch 99% data on the Internet.

## Architecture

## Todo

* Crack Android and IOS for app's data like WeChat.
* Use access token for api using.

## Scrapy System

> Every thing on Internet is source and can be fetched.
> We want every thing we need.
> Crawl and Scape

> WYSIWYG

### 功能模块

#### V0.1

**基础模块**
1. 管理与调度 C
2. 下载器 D
3. 解析与抽取 AP
4. UI界面 U

**基础设施**
1. 消息队列 Q
2. 存储组件 S
3. 缓存单元 M

#### V1.0

**基础模块**中3、4具体由插件层提供

**基础设施** Backend由插件层适配

**扩展服务**
1. 代理服务 PR
2. 定时任务服务 CS

### 组织结构图

```

    +------------------------------+
    |             Clerk            |```````````④ 业务层
    +----+--------------------+----+
         |    Configuration   |````````````````③ 配置层
    +----+--------------------+----+
    |          Extensions          |```````````② 插件层
    +------+----------------+------+
           | Infrastructure |``````````````````① 基础层
           +----------------+

```

**1. Infrastructure layer**

基础设施层，提供最基本的运行时环境与API支持，给插件层提供所有的控制能力。

**2. Extension layer**

插件层，功能模块在这里实现。比如，代理中间件，存储模块，DSL模块，设备Headers模块，事件机制等。对于依赖外部设备的情景，外部设备的控制逻辑，也是由插件层提供。

这里并不提供服务级别逻辑，比如代理中间件，代理服务端应该是独立的服务，插件层这里完成的是对代理服务端的连接。那么如果我们需要请求注入呢，首先我们应该在服务端提供这个接口，然后插件提供给配置层。

一些需要在请求参数粒度的控制，可能需要代码逻辑的支持，比如增加参数时间戳此类。在除了尽可能多的提供原生（）函数级别的控制外，还应该开发接口交由代码或者是DSL控制。一个HTTP请求的结构是可枚举的，但是内容不是，所以这里可以提供代码级别的控制。

再如，需要对配置层的爬虫进行分组管理，并且设置访问权限，资源限制等也均由插件层来提供。

**3. Configuration layer**

配置层，配置业务通用逻辑，该层可供普通用户新增，提供私有与公有访问权限。
比如，微博Web爬虫，Web客户端爬虫，都在这里完成。通过配置项来控制与维护请求的过程以及结果态，同时提供自定义的接口或方法等给业务层使用。例如，当抓取到新内容时，回调业务层注册的回调逻辑。

**4. Clerk layer**

业务层，完成所有的业务逻辑。比如，抓取微博新内容的存储，字段提取，通知等。业务层可以一般情况下是使用配置层的数据与控制逻辑，但也可以直接使用插件层的插件。比如使用账号管理插件，来使用账号用作抓取；代理配置选择；调用与控制外部设备等的功能。


### 使用场景示例

业务-UI操作生成配置项

```yaml
group: ugc
tag:
    - ugc
    - web
    - api
init:
    pusher:
        extensions.Pusher("scheme://[username:password@]host:port[/channel]")
    mongo: # storage.Mongo | extensions.Mongo("mongo://...")
        indexs: []
        db: ugc
        coll: weibo
        fields:
            pubtime:
            content: 
            author: 
            comments_count: 0
            requotes_count: 0
            comments: []
            requotes: []
    email: extensions.Email("...")
    operators:
        - email: operator_one@myhexin.com
          phone: 13888888888
        - email: operator_two@myhexin.com
          phone:
functions:
    - 
weibo:
    on_newmsg: # on_newmsg(func (data){}), data: {count: 1, data: []}
        - pusher.push({Count: data.count})
        - map((d)->email.send(d.email), data), operators)
        - function: map(d)
          params: data.data
          do: mongo.save(d)
          return:
    on_functions:        
    on_xxxxxx_error: #
    on_unknow_error: #
```

sth. ==> `.so`

配置-UI操作生成配置项

这个结合另一个代理抓取的项目，可以在编辑页面中来抓包帮助分析接口与逻辑。

类似于Phone Browser Over Remote，就此还可以直接是Phone Over Remote

```yaml
weibo:
    scheme: http
    host: m.weibo.cn
    version:
    prefix:
    client: extension.client.random() # client not just only set User-Agent
    requests:
        login: # request结构
        unlogin:
        check_new:
        get_new:
        follow:
        unfollow:
        create:
        delete:
        requote:
        comment:
        del_comment:
        del_requote:
        search:
    logic: # 这里用流程图实现，或代码
```

```yaml
---request结构
url: /unread
params:
    t: now(1489472015964)

```

插件-逻辑代码

```python
class Test(object):
    pass
```

基础-逻辑代码

```python

```

### 逻辑组件



### 具体实现


### 其他

1. 静态资源下载[类似缓存系统]
    > http://cdn-fetch.iwencai.com/{{appid}}/{{uid}}?token={{token}}
    > 1. `appid`为使用者id，可以与资源id关联，设置访问权限
    > 2. `uid`为资源id，可以直接为uri的base64编码(推荐)，或者uuid指向uri
    > 3. `token`为认证token，fetch时必须参数，其他操作可根据配置确定是否需要