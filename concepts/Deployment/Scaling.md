# 弹性

如果您对应用程序有大流量的预期（或者您已经拥有流量），您需要建立一个可扩展的架构。随着越来越多的请求点击您的应用程序，您可以添加新的服务器。


### 性能

在产品发布中，Sails像Connect，Express或Socket.io应用程序一样执行（[示例](http://serdardogruyol.com/?p=111)）。 如果你有自己的标准，你想分享一下，请写一篇博客文章或者文章和推文[@sailsjs](http://twitter.com/sailsjs)。 除了基准测试之外，请记住，大多数性能和可伸缩性量化标准都是特定于应用程序的。 应用程序的实际性能与实现业务逻辑和模型调用的方式有很大关系，而不是关于您所使用的底层框架。


### 架构示例

```
                             ....
                    /  Sails.js server  \      /  Database (e.g. Mongo, Postgres, etc)
Load Balancer  <-->    Sails.js server    <-->    Socket.io message queue (Redis)
                    \  Sails.js server  /      \  Session store (Redis, Mongo, etc.)
                             ....
```


### 准备APP进行群集部署

Node.js（以及Sails.js）应用程序弹性扩展。 这是一种强大而有效的方法，但它涉及一些规划。 在规模上，您希望能够将您的应用程序复制到多个Sails.js服务器上，并将它们放在负载均衡后面。

扩展App的一大挑战是这些集群无法共享内存，因为它们物理上在不同的机器上。 最重要的是，不能保证用户会在请求之间（无论是HTTP还是socket）“抓住”同一台服务器，因为负载均衡会将每个请求route到具有最多可用资源的Sails服务器。 可扩展服务器端app最重要的是它应该是**stateless**。 这意味着能够将相同的代码部署到不同的服务器上，多个服务器处理多个传入请求，并仍然有效。 幸运的是，Sails应用程序已经为这种部署做好了准备，几乎可以立即使用。 但在将您的应用程序部署到多个服务器之前，您需要做一些事情:


+ 确保您的应用程序中可能使用的依赖关系，依赖于共享内存。
+ 确保你的数据库模型（例如MySQL，Postgres，Mongo）是可扩展的（例如分片/集群）

+ **如果你的app使用sessions:**
  + 配置您的应用程序以使用共享session存储，例如Redis（取消`config/session.js`中的`adapter`注释选项），然后安装“@sailshq / connect-redis” session适配器作为您的应用程序的依赖项（例如 `npm install @ sailshq /connect-redis --save`）。 有关配置用于产品的session存储的更多信息，请参阅[sails.config.session](https://sailsjs.com/documentation/reference/configuration/sails-config-session#?production-config)文档。
  

+ **如果你的app使用sockets:**
  + 配置您的应用程序使用Redis，作为共享消息队列来传递socket.io消息。 默认情况下，Socket.io（因此Sails.js）应用程序支持Redis，为了启用远程redis pubsub服务器，请在`config/env/production.js`中取消相关注释。
  + 安装“@ sailshq /socket.io-redis”适配器作为您的应用程序的依赖项(例如`npm install @ sailshq /socket.io-redis`)
  + **如果你的集群在一台服务器上 (使用 [pm2 cluster mode](http://pm2.keymetrics.io/docs/usage/cluster-mode/))**
  + 为了避免由于Grunt任务引起的文件冲突问题，请始终在`production`环境中启动您的应用程序，考虑[完全关闭Grunt](https://sailsjs.com/documentation/concepts/assets/disabling-grunt)。 请参阅[这里](https://github.com/balderdashy/sails/issues/3577#issuecomment-184786535)以获取有关单服务器集群中的Grunt问题的更多详细信息
  + 注意数据库中的[`config /bootstrap.js`代码](https://sailsjs.com/documentation/reference/configuration/sails-config-bootstrap)数据，以避免引导程序运行多次时发生冲突 （群集中每个节点一次）

### 将Node/Sails应用程序部署到PaaS

Deploying your app to a PaaS like Heroku or Modulus is dead simple. Depending on your situation, there may still be a few devils in the details, but Node support with hosting providers has gotten _really good_ over the last couple of years.  Take a look at [Hosting](https://sailsjs.com/documentation/concepts/deployment/Hosting) for more platform-specific information.

### 部署您自己的群集

+ Deploy multiple instances (aka servers running a copy of your app) behind a [load balancer](https://en.wikipedia.org/wiki/Load_balancing_(computing)) (e.g. nginx)
  + Configure your load balancer to terminate SSL requests
  + But remember that you won't need to use the SSL configuration in Sails-- the traffic will already be decrypted by the time it reaches Sails.
  + Lift your app on each instance using a daemon like `forever` or `pm2` (see https://sailsjs.com/documentation/concepts/deployment for more about daemonology)


### 优化

在Node/Sails应用程序中优化节点就像在任何其他服务器端应用程序中优化节点一样;例如， 识别和手动优化慢查询，减少查询次数等。  特别是对于Node应用程序，如果您发现有一个流量很大的节点正在吃掉CPU，请寻找同时阻塞的模型、服务或机器，这些可能会在循环或递归中一遍又一遍地被调用。

但要记住:

> 不成熟的优化是万恶之源。  -[Donald Knuth](http://c2.com/cgi/wiki?PrematureOptimization)

无论您使用的是什么工具，将精力和时间花在编写高质量、排版、可读性强的代码非常重要。 这样，如果你在优化应用程序中代径时，这样做更容易。



### 注意

> + 您不一定要在session中使用Redis--您可以使用任何于Connect或Express兼容的会话存储。 有关更多信息，请参阅[sails.config.session](sailsjs.com/documentation/reference/configuration/sails-config-session)。

> + 一些Redis托管提供商（例如Redis To Go）会存在空闲连接超时。 大多数情况下，您希望关闭timeout以避免应用程序出现意外。 但如何关闭timoeout的取决于提供商（您可能需要联系他们的支持团队）。

<docmeta name="displayName" value="Scaling">
