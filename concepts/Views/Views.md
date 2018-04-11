# 视图
### 概述

在Sails中，视图是在服务器上编译成HTML页面的标记模板。 大多数情况下，视图用作对传入HTTP请求的响应，例如，为您的主页提供服务。

或者，可以将视图直接编译为HTML字符串以用于后端编码（请参阅[`sails.renderView()`](https://github.com/balderdashy/sails-docs/blob/master/PAGE_NEEDED.md).)）例如，您可以使用此方法发送HTML电子邮件，或构建XML字符串以用于旧版API。


##### 创建一个视图

默认情况下，Sails使用EJS([Embedded Javascript](http://ejs.co/))作为其视图引擎。EJS的语法是非常传统的 - 如果你已经使用过php，asp，erb，gsp，jsp等，你会很容易上手。

如果您更喜欢使用不同的视图引擎，则有多种选择。 Sails通过[Consolidate](https://github.com/visionmedia/consolidate)支持所有与[Express](http://expressjs.com/en/guide/using-template-engines.html)兼容的视图引擎。

默认情况下，视图在APP的[`views /`](https://sailsjs.com/documentation/anatomy/views)文件夹中定义，但与Sails中的所有默认路径一样，它们是[可配置](https：//sailsjs.com/documentation/reference/configuration/sails-config-views)的。 如果您根本不需要提供动态HTML页面（例如，如果您正在为移动应用程序构建API），则可以从应用程序中删除该目录。


##### 编译一个视图

任何你可以访问`res`对象的地方（例如控制器action，自定义响应或策略），你可以使用[`res.view`](https://sailsjs.com/documentation/reference/response-res/res -view)来编译你的一个视图，然后把生成的HTML发送给用户。

您还可以直接在您的`routes.js`文件中将视图接入路由。 只需指出应用`views/`目录中视图的相对路径即可。例如:

```javascript
{
  'get /': {
    view: 'pages/homepage'
  },
  'get /signup': {
    view: 'pages/signup/basic-info'
  },
  'get /signup/password': {
    view: 'pages/signup/choose-password'
  },
  // and so on.
}
```

##### 单页面应用程序呢？

如果您正在为浏览器构建Web应用程序，则部分（或全部）导航会在客户端进行; 客户端代码预加载一些标记模板，然后在用户的浏览器中呈现标记模板，而不需要直接再次访问服务器。

在这种情况下，您有几个选项可用于引导单页应用程序:

+ 使用单个视图，例如`视图/publicSite.ejs`。 优点:
  + 您可以在Sails中使用视图引擎将数据从服务器直接传递到客户端HTML中。这是一种将用户数据等内容传送到客户端JavaScript的简单方法，无需从客户端发送AJAX/WebSocket请求。
+ 在asset文件夹中使用一个HTML页面，例如`asset/index.html`。优点:
  + 尽管无法以此方式将服务器端数据直接传递到客户端，但这种方法允许您分离应用程序的客户端和服务器端部分。
  + 您的asset文件夹中的任何内容都可以移至静态CDN（如Cloudfront或CloudFlare），使您可以利用该提供商分散的数据中心，让您的内容更贴近用户。



<docmeta name="displayName" value="Views">
<docmeta name="nextUpLink" value="/documentation/concepts/assets">
<docmeta name="nextUpName" value="Assets">
