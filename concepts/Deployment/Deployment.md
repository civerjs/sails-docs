# 部署

### 部署之前

在启动任何Web应用程序之前，您应该问自己几个问题:

+ 你期望的流量是多少？
+ 您是否有合同要求满足任何时间正常运行，例如 服务级别协议（SLA）？
+ 什么类型user Agent将“点击”你的基础服务？
  + 桌面浏览器
  + 移动web浏览器 (以及什么形式的条件？ 平板电脑还是手机？ 或两者？)
  + 来自智能电视或游戏机的嵌入式浏览器
  + Android/iOS/Windows Phone apps
  + PhoneGap/Electron apps
  + 开发者 (cURL, Postman, AJAX requests, WebSocket front-end apps)
  + 其他设备 (tvs, watches, toasters..?)
+ 他们会需要什么? (e.g. HTML? JSON? XML?)
+ 你会利用Socket.io的实时特性吗?
  + e.g. 聊天室, 实时分析表, app内消息通知
+ 你如何跟踪崩溃和错误？
  + 您是否将`sails.log（）`与[Papertrail](https://papertrailapp.com/)等托管服务结合使用？ <!--Or are you using a custom logger from NPM like [Winston](https://github.com/winstonjs/winston)?  Or even easier, sticking with built-in logging from `sails.log()` in combination with a hosted service like [Papertrail](https://papertrailapp.com/)?-->
+ Have you tried lifting locally with the `NODE_ENV` environment variable set to "production"?
  + 测试这个的快速方法是运行 `NODE_ENV=production node app` (or, as a shortcut: `sails lift --prod`).


### 配置您的应用程序用于生产环境

您可以提供仅适用于生产的配置[几种不同的方式](https://sailsjs.com/documentation/reference/configuration).  大多数应用程序使用混合的环境变量和`config / env / production.js`。但不管你如何去做，本节和[Scaling section](https://sailsjs.com/documentation/concepts/deployment/scaling) 的文档涵盖了在开始生产之前应该检查或更改的配置设置。



### 部署在单个服务器上

Node.js非常快。 对于许多应用程序来说，一台服务器足以应付预期的流量 - 至少在一开始就是如此。

> 本节重点介绍_single-server Sails deployment_。 这种部署本身在规模上受到限制。 有关在负载均衡后部署Sails/Node应用程序的信息，请参阅[扩展]（https://sailsjs.com/documentation/concepts/deployment/scaling）。

许多团队决定将他们的应用程序部署在负载均衡或代理之后（例如在Heroku或Modulus之类的PaaS中，或在nginx服务器之后）。  这通常是正确的方法，因为它可以帮助您的应用面向未来，以防扩展性需求发生变化时您需要添加更多服务器。如果您使用的是负载均衡或代理，则可以忽略下面列表中的一些内容:

+ 不要担心配置Sails以支持SSL证书。 负载均衡/代理服务器或您的PaaS提供商总是会解决SSL问题。
+ 您不需要为将应用设置在端口80上运行而担心（如果不在nginx这样的代理之后）。 大多数PaaS提供商会自动为您提供端口。 如果您使用代理服务器，请参阅这个文档（无论您是否需要为Sails应用程序配置端口）。 

> 如果您的应用使用socket并且您使用的是nginx，请确保将其配置为将websocket消息中继到您的服务器。 你可以在[nginx关于这个主题的文档]中找到关于代理WebSockets的指导(http://nginx.org/en/docs/http/websocket.html)。


##### 将`NODE_ENV`环境变量设置为`'production'`

将应用程序的环境配置配置为“production”会让Sails打起精神; 即您的应用程序正在生产环境中运行. 这是最重要的一步. 如果您在部署Sails应用程序之前只有时间修改一个设置，那么就是这个设置！

当您的应用在生产环境中运行时:
  + Sails中的中间件和其他依赖转换为使用更高效的代码。
  + 所有[models' migration settings](https://sailsjs.com/documentation/concepts/models-and-orm/model-settings)都被强制迁移：'safe'。 这是一种故障安全防护措施，可防止在部署期间无意中损坏您的生产数据。
  + 您的asset托管以生产模式运行（如果相关）。 开箱即用，这意味着您的Sails应用会将所有样式表，客户端脚本和预编译的JST模板编译为缩小的`.css`和`.js`文件，以减少页面加载时间并减少带宽消耗。
  + 来自`res.serverError（）`的错误消息和堆栈跟踪仍将被记录，但不会在响应中发送（这是为了防止潜在的攻击者访问到任何敏感信息，例如加密密码或您的Sails应用程序位于服务器的文件系统中的路径 ）


>**注意:**
>如果你以其他方式将``sails.config.environment``(https://sailsjs.com/documentation/reference/configuration/sails-config#?sailsconfigenvironment)设置为`'production'`，那就太酷了。 请注意，Sails会自动为您设置“NODE_ENV”环境变量为“production”（或以其他方式记录警告 - 并在控制台显示！）。  这个环境变量如此重要的原因在于它是Node.js中的通用约定，无论您使用的是哪种框架。  Sails中的内置中间件和依赖项_expect_ NODE_ENV可以在生产环境中设置，否则它们将使用效率较低的代码路径，这些代码路径仅用于开发用途。

##### 设置一个`sails.config.sockets.onlyAllowOrigins`值

如果你的应用程序启用了socket（也就是说，你已经安装了`sails-hook-sockets`模块），那么为了安全起见，你需要将`sails.config.sockets.onlyAllowOrigins`设置来源数组，并允许它通过websockets连接到你的应用程序。  你可能会在你的应用程序的`config / env / production.js`文件中设置它. 参考 [socket configuration documentation](https://sailsjs.com/documentation/reference/configuration/sails-config-sockets) 了解更多关于 `onlyAllowOrigins`.


##### 将您的应用配置为在端口80上运行

无论是通过使用`sails_port`环境变量，设置`--port`命令行选项还是更改生产配置文件，都将以下内容添加到Sails配置的顶级:

```javascript
port: 80
```

> 如上所述，如果您的应用将运行在负载均衡或代理之后，请忽略此步骤。



##### 为您的模型设置生产数据库

如果所有应用程序的模型都使用默认数据存储，那么设置生产数据库就如在[config/env/production.js](https://sailsjs.com)中配置`sails.config.datastores.default`一样简单(https://sailsjs.com/documentation/concepts/configuration#?environmentspecific-files-config-env)查看正确设置。

如果您的应用程序使用多个数据库，配置过程也是类似操作。 对于应用程序使用的每个数据库，请将一个项目添加到[config / env / production.js]中的`sails.config.datastores`字典[config/env/production.js](https://sailsjs.com/documentation/concepts/configuration#?environmentspecific-files-config-env).

请记住，如果您使用版本控制(e.g. git), 那么如果将它们包含在应用程序的配置文件中，则会将任何敏感凭据（如数据库密码）签入提交。  此问题的常见解决方案是提供某些敏感的配置设置作为环境变量。  See [Configuration](https://sailsjs.com/documentation/concepts/configuration) for more information.

如果您使用的是像MySQL这样的关系数据库，那么还有一个额外的步骤。 请记住Sails在生产环境中如何将所有模型设置为`migrate：safe`？  这意味着在发布应用程序时不会执行自动迁移...这意味着默认情况下你的表不会存在。  首次设置Sails应用程序的关系数据库时，处理此问题的常见方法如下:
  + 在生产数据库服务器上创建数据库 (e.g. `frenchfryparty`)
  + 在本地配置您的应用以使用此生产数据库, but _don't set the environment to `'production'`, and leave your models' configuration set to `migrate: 'alter'`_.  Now run `sails lift` **once**-- and when the local server finishes lifting, kill it.
    + **小心！**只有在生产数据库中没有数据时才应该这样做。

如果这会让您感到紧张，或者无法远程连接到生产数据库，则可以跳过上述步骤。 相反，只需转储您的本地模式并将其导入生产数据库即可。


##### 启用CSRF保护

Protecting against CSRF is an important security measure for Sails apps.  If you haven't already been developing with CSRF protection enabled (see [`sails.config.security.csrf`](https://sailsjs.com/documentation/reference/configuration/sails-config-security#?sailsconfigsecuritycsrf)), be sure to [enable CSRF protection](https://sailsjs.com/documentation/concepts/security/csrf#?enabling-csrf-protection) before going to production.



##### 启用 SSL

If your API or website does anything that requires authentication, you should use SSL in production.  To configure your Sails app to use an SSL certificate, use [`sails.config.ssl`](https://sailsjs.com/documentation/reference/configuration/sails-config).

> As mentioned above, ignore this step if your app will be running behind a load balancer or proxy (e.g. on a PaaS like Heroku).



##### 激活您的应用

部署的最后一步实际上是启动服务器。 例如:

```bash
NODE_ENV=production node app.js
```

Or if you're more comfortable with command-line options you can use `--prod`:

```bash
node app.js --prod
# (Sails will set `NODE_ENV` automatically)
```

As you can see, instead of `sails lift` you should start your Sails app with `node app.js` in production.  This way, your app does not rely on having access to the `sails` command-line tool; it just runs the `app.js` file bundled in your Sails app (which does exactly the same thing).


##### ...保持服务

Unless you are not deploying to a PaaS like Heroku, you will want to use a tool like [`pm2`](http://pm2.keymetrics.io/) or [`forever`](https://github.com/foreverjs/forever) to make sure your app server will start back up if it crashes.  Regardless of the daemon you choose, you'll want to make sure that it starts the server as described above.



### Next steps
+ [Security](https://sailsjs.com/documentation/concepts/security)
+ [Hosting options](https://sailsjs.com/documentation/concepts/deployment/hosting)
+ [Scaling your Sails/Node.js app](https://sailsjs.com/documentation/concepts/deployment/scaling)
+ [Complete API Reference](https://sailsjs.com/documentation/reference)


<docmeta name="displayName" value="Deployment">
