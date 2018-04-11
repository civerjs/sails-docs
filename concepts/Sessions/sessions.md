# 了解Sails中的session

就我们的目的而言，**session**与少数几个组件同义，它们允许您在请求间存储有关user agent的信息。

> **user agent** 代表设备（例如计算机上的浏览器选项卡，智能手机应用程序）上所使用的软件（例如，浏览器或本机应用程序）。它与cookie或访问令牌一对一关联。

session可能非常有用，因为请求/响应周期是**stateless**。请求/响应周期被认为是无状态的，因为客户端和服务器本身都不会在关于特定请求的不同请求之间存储任何信息。因此，当对user agent（例如`res.send()`）做出响应时，请求/响应的生命周期结束。

请注意，我们将在浏览器user agent的上下文中讨论session。 尽管您可以在Sails中使用session进行任何您喜欢的操作，但纯粹用于存储用户代理身份验证的状态通常是最佳做法。认证是一个过程，允许用户代理证明他们有一定的身份。 例如，为了访问某些受保护的功能，我可能需要证明我的浏览器选项卡实际上与数据库中的特定用户记录相对应。 如果我为您提供了一个唯一的名称和密码，您可以查找名称并将其与存储的密码进行比较。 密码。如果有匹配，会通过验证。但是，如何在请求之间存储“认证”？ 这就是session存在的意义。


### session由什么构成
在Sails中实现session有三个主要组件:
1. **session store**保留信息
2. 管理session的中间件
3. 一个与请求一起发送并存储会话ID的cookie（默认情况下为`sails.sid`）

**session store**可以存储在内存中（例如默认的Sails session存储区）或数据库中（例如Sails为此目的已经内置了对使用Redis的支持）。 Sails构建在Connect中间件的基础上来管理会话; 其中包括使用**cookie**在user agent上存储会话标识（`sid`）。


### *request*,*response*,*session*的周期
当一个`request`发送给Sails时，请求头由session中间件解析。

##### 场景1：请求标头没有*cookie属性*

如果头部不包含cookie属性，则在会话中创建一个“sid”，并将默认sesssion字典添加到“req”（例如`req.session`）。此时，您可以更改sesssion属性（通常在控制器/action中）。例如，让我们看看下面的*登录*操作。

```javascript
module.exports = {

  login: function(req, res) {

    // Authentication code here

    // If successfully authenticated

    req.session.userId = foundUser.id;   // returned from a database

    return res.json(foundUser);

  }
}
```

在这里，我们为`req.session`添加了一个`userId`属性。

> **注意：**该属性将不会存储在*session store*中，并且在发送响应之前也不可用于其他请求。

一旦发送了响应，任何新的请求都将有权访问`req.session.userId`。 由于我们在请求头中没有cookie*属性*，因此我们将为其建立一个cookie。

##### 场景2：请求头部具有一个cookie*属性*和一个`Sails.sid`

当user agent发出下一个请求时，会检查存储在cookie上的`Sails.sid`的真实性，并且如果它与session stroe中现有的`sid`匹配，session stroe的内容将作为属性加入`req`字典（例如`req.session`）。 我们可以在`req.session`（例如`req.session.userId`）上访问属性或者在其上设置属性（例如`req.session.userId == someValue`）。 session stroe中的值可能会更改，但通常`Sails.sid`和`sid`不会更改。


### 什么时候`Sails.sid`改变了？
在开发过程中，Sails会话存储(session store)*在内存*中。 因此，当您关闭Sails服务器时，当前session store会去西天（消失..）。 当Sails重新启动时，尽管user agent请求在Cookie中包含“Sails.sid”，但sid不再存在于session store中。 因此，一个新的`sid`将被生成并在cookie中被替换。如果用户代理cookie过期或被删除，`Sails.sid`也会改变。


>通过修改`projectName/config/session.js`中的`cookie.maxAge`属性，可以将Sails Cookie的生命周期从其默认设置（例如永不过期）更改为新设置。


### 使用* Redis *作为会话存储

Redis是一个键/值数据库，可用作与Sails实例分开的session store。 session的这种配置有两个好处。 首先是session store将在Sails重启之间保持可用。第二是如果负载均衡后面有多个Sails实例，则所有实例都可以指向一个统一的session store。


#### 开发中启用Redis会话存储(session store)

要在开发中启用Redis作为session store，首先确保您的计算机上运行了本地Redis实例（`redis-server`）。 然后，用`sails lift --redis`升级你的应用程序。

这只是`sails lift --session.adapter=@sailshq/connect-redis --sockets.adapter = @sailshq /socket.io-redis`的快捷方式。 这些软件包默认包含在新Sails应用程序的依赖项中，但是如果您使用的是升级版本，则需要"npm install @sailshq / connect-redis"和"npm install @sailshq/socket.io-redis"。

> 注意：此内置配置使用您的本地Redis实例。 有关高级会话配置选项，请参阅 [Reference > Configuration > sails.config.session](https://sailsjs.com/documentation/reference/configuration/sails-config-session).

#### session和cookie是如何创建的

这个cookie的值是通过首先用一个可配置的*secret*来加密sid来创建的，它只是一个很长的字符串。

> You can change the session `secret` property in `projectName/config/session.js`.

The Sails `sid` (e.g. `Sails.sid`) then becomes a combination of the plain `sid` followed by a hash of the `sid` plus the `secret`.  To take this out of the world of abstraction, let's use an example.  Sails creates a `sid` of `234lj232hg234jluy32UUYUHH` and a `session secret` of `9238cca11a83d473e10981c49c4f`. These values are simply two strings that Sails combine and hash to create a `signature` of `AuSosBAbL9t3Ev44EofZtIpiMuV7fB2oi`.  So the `Sails.sid` becomes `234lj232hg234jluy32UUYUHH.AuSosBAbL9t3Ev44EofZtIpiMuV7fB2oi` and is stored in the user agent cookie by sending a `set-cookie` property in the response header.

**What does this prevent?** It prevents a user from guessing the `sid` as well as prevents a evil doer from spoofing a user into making an authetication request with a `sid` that the evil doer knows.  This could allow the evil doer to use the `sid` to do bad things while the user is authenticated via the session.

### 禁用sessions

Even if your Sails app is designed to be accessed by non-browser clients, such as toasters, you are strongly encouraged to use sessions for authentication.  While it can sometimes be complex to understand, the built-in session mechanism in Sails (session store + HTTP-only cookies) is a tried and true solution that is generally [less brittle, easier to use, and lower-risk than rolling something yourself](http://cryto.net/~joepie91/blog/2016/06/13/stop-using-jwt-for-sessions/).

That said, sessions may not always be an option (for example, if you must [integrate with a different authentication scheme](https://github.com/sails101/jwt-login) like JWT).  In these cases, you can disable sessions on an app-wide or per-request basis.

##### 禁用整个应用的sessions

To entirely turn off session support for your app, add the following to your `.sailsrc` file:

```javascript
"hooks": {
  "session": false
}
```

This disables the core Sails session hook.  You can also accomplish this by setting the `sails_hooks__session` environment variable to `false`.

##### 禁用某些请求的sessions

To turn off session support on a per-route (or per-request) basis, use the [`sails.config.session.isSessionDisabled` setting](https://sailsjs.com/documentation/reference/configuration/sails-config-session#?properties).  By default, Sails enables session support for all requests except those that [look like](https://sailsjs.com/documentation/reference/application/advanced-usage/sails-looks-like-asset-rx) they're pointed at static assets like images, stylesheets, etc.

<docmeta name="displayName" value="Sessions">
