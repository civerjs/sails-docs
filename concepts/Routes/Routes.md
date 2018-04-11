# 路由Routes

### 概述

任何Web应用程序最基本的功能是能够解释发送的URL请求，然后响应。 为了做到这一点，您的程序必须能够区分多个URL。

像大多数Web框架一样，Sails提供了一个路由器：一个将URL映射到actions和views的机制。**路由**是一些规则，告诉Sails在传入请求时该做什么。Sails中有两种主要类型的路由：**自定义**（或“explicit”）和**自动**（或“implicit”）。



### 自定义路由

Sails允许你以任何喜欢的方式设计应用程序的URL--没有框架限制。

每个Sails项目都带有[`config/routes.js`](https://sailsjs.com/documentation/reference/configuration/sails-config-routes)。这是一个简单的[Node.js模块](http://nodejs.org/api/modules.html)，导出自定义对象或“显式”**路由**。 例如，下面这个`routes.js`文件定义了六条路由; 其中一些指向控制器的行为，而另一些则直接指向视图。


```javascript
// config/routes.js
module.exports.routes = {
  'get /signup': { view: 'conversion/signup' },
  'post /signup': 'AuthController.processSignup',
  'get /login': { view: 'portal/login' },
  'post /login': 'AuthController.processLogin',
  '/logout': 'AuthController.logout',
  'get /me': 'UserController.profile'
}
```

每个**路由**由一个**地址**（在左边，例如“get /me”）和一个**目标**（在右边，例如UserController.profile）组成。**地址**是URL路径和（可选）特定的[HTTP方法](http://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol#Request_methods)。 **目标**可以通过多种不同方式定义（[参见主题的扩展概念部分](https://sailsjs.com/documentation/concepts/routes/custom-routes#?route-target)），上面两种不同的语法是最常见的。 当Sails收到一个传入请求时，它会检查所有自定义路由的**地址**是否匹配。 如果找到匹配的路由，则请求将传递给它的**目标**。

例如，我们这样理解`'get /me': 'UserController.profile'`:

> "Hey Sails, when you receive a GET request to `http://mydomain.com/me`, run the `profile` action of `UserController`, would'ya?"

What if I want to change the view layout within the route itself?  No problem we could:

```javascript
'get /privacy': {
    view: 'users/privacy',
    locals: {
      layout: 'users'
    }
  },
```

#### 注意
+ 一个请求匹配一个路由**地址**并不意味着它直接传递给该路由的**目标**。 例如，HTTP请求通常首先通过一些[中间件]。 如果路由指向一个控制器[action]，请求将需要通过任何配置的[policies]。 最后，还有一些特殊的[路由选项]，可以让某些路由被“跳过” 要求。
+ 路由器还可以编程方式将**路由绑定**到有效的路由目标，包括中间件（即`function（req，res，next）{}`）。 但如果可能，您应该始终使用传统的[路由目标语法](https://sailsjs.com/documentation/concepts/routes/custom-routes#?route-target)，它简化了开发过程，简化了培训，并使您的应用程序更易于维护。



### 自动路由

除了自定义路由之外，Sails会自动为您绑定多条输入。 如果URL与自定义路由不匹配，可能会匹配一个自动路由并响应。 Sails中的自动路线的主要类型是:

* [Blueprint routes](https://sailsjs.com/documentation/reference/blueprint-api?q=blueprint-routes), which provide your [controllers](https://sailsjs.com/documentation/concepts/controllers) and [models](https://sailsjs.com/documentation/concepts//models-and-orm/models) with a full REST API.
* [Assets](https://sailsjs.com/documentation/concepts/assets), such as images, Javascript and stylesheet files.


##### 未处理的请求

如果没有自定义路由或自动路由匹配请求URL，则Sails将发回默认的404响应。 这个响应可以通过添加一个`api/ responses/ notFound.js`文件来定制。 有关更多信息，请参见[自定义响应](https://sailsjs.com/documentation/concepts/extending-sails/custom-responses)。


##### request方法中未处理的错误

如果在处理request期间抛出未处理的错误（例如，在某些[action代码](https://sailsjs.com/documentation/concepts/actions-and-controllers)中），Sails将发送默认的500响应 。这个响应可以通过添加一个`api/responses/serverError.js`文件来定制。参见[custom responses](https://sailsjs.com/documentation/concepts/extending-sails/custom-responses) 更多信息。


### 支持的协议

Sails路由器是“协议不可知的”; 它知道如何处理[HTTP请求](http://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol)以及通过[WebSockets](http://en.wikipedia.org/wiki/Websockets)发送的消息。 它侦听简单格式（称为JWR（JSON-WebSocket请求/响应））发送给Socket.io来实现此目的。 该规范在[客户端套接字SDK](https://sailsjs.com/documentation/reference/web-sockets/socket-client)中实现并可用于开箱即用。


#### 注意
+ 高级用户可以选择完全绕开路由器，并直接发送底层的，完全可定制的WebSocket消息到底层的Socket.io服务器。 您可以直接在应用程序的[`onConnect`](https://sailsjs.com/documentation/reference/configuration/sails-config-sockets#?commonlyused-options)中绑定套接字事件(located in [`config/sockets.js`](https://sailsjs.com/documentation/anatomy/config/sockets.js).) 但请记住，在大多数情况下，您最好利用请求解释器进行套接字通信 - 通过HTTP和WebSockets维护路由的一致有助于保持可维护性。




<docmeta name="displayName" value="Routes">
<docmeta name="nextUpLink" value="/documentation/concepts/actions-and-controllers">
<docmeta name="nextUpName" value="Actions">
