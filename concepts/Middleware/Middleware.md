# 中间件Middleware

从技术上讲，您将在Sails应用程序中编写的大部分代码是中间件(Middleware)，它在传入请求和传出响应之间运行 - 即位于请求/响应堆栈的“中间”。  在MVC框架中，术语“Middleware” 通常更具体地指的是在路由处理代码之后运行_before_或_after_的代码 (例如你的[控制器 actions](https://sailsjs.com/documentation/concepts/Controllers?q=actions)), 可以将相同的代码应用于多个路由或操作。 Sails对中间件设计模式提供了强有力的支持。根据您的需求，您可以选择实施:

* HTTP中间件 (在每个HTTP请求之前应用代码 - 请参阅下面的更多详细信息)
* [策略(Policies)](https://sailsjs.com/documentation/concepts/policies) (在一个或多个控制器actions之前应用代码)
* [钩住具有路由功能的组件](https://sailsjs.com/documentation/concepts/extending-sails/hooks/hook-specification/routes) (在一个或多个路由处理程序之前应用代码)
* [自定义响应](https://sailsjs.com/documentation/concepts/custom-responses) (在一个或多个控制器操作后应用代码)

### HTTP中间件

Sails与Express/Connect中间件完全兼容，它是接受`req`，`res`和`next`作为参数的函数。 每个应用程序都使用可配置的中间件堆栈，用于处理HTTP请求。 每次应用程序收到HTTP请求时，其配置的HTTP中间件堆栈都会按顺序运行。

> 请注意，此HTTP中间件堆栈仅用于“true”HTTP请求 - **虚拟请求**被忽略（例如来自活动Socket.io连接的请求）。

##### 内置HTTP中间件

默认情况下，Sails使用几个不同的中间件函数来处理相关低级HTTP任务。 就像解释cookies，解析HTTP request bodies，提供assets，附加到app的路由。 您可以阅读有关默认中间件堆栈的更多信息[here](https://sailsjs.com/documentation/concepts/middleware/conventional-defaults)。


### 配置HTTP中间件堆栈

由于中间件堆栈具有合理的默认值，因此很多Sails应用程序根本不需要修改此配置。 但是对于需要更多灵活性的情况，Sails使添加、重新排序、覆盖和禁用中间件堆栈中的函数变得非常简单。


##### 添加中间件

希望它在middleware chain中运行，要配置一个新的自定义HTTP middleware function,首先在`middleware`（例如“foobar”）中添加一个middleware function。然后在中间件Array中添加（“foobar”）中键名。

除了“排序”（保留用于配置中间件堆栈的顺序）之外，分配给“sails.config.middleware”的一个键的任何值都应该是带有三个参数的函数：
: `req`, `res`和 `next`.  这个函数几乎完全像一个[策略policy](https://sailsjs.com/documentation/concepts/policies); 唯一的实际区别是它在执行时.

##### 初始化中间件
如果您需要运行一些one-time安装代码用于自定义中间件功能的，则需要在发生之前这样做。推荐的方法是使用self-calling（aka ["immediately-invoked"](https://en.wikipedia.org/wiki/Immediately-invoked_function_expression)）封装函数。 在下面的示例中，请注意，不是直接将该值设置为“req，res，next”函数，而是使用self-calling函数来“封装”一些初始设置代码。 那个self-calling的封包函数返回最终的中间件（req，res，next）函数，所以它被设置在键上，就像它直接传入一样。


##### 示例：使用自定义中间件
以下示例显示如何设置三个不同的自定义HTTP中间件功能:

```js
// config/http.js
module.exports.http = {

  middleware: {

    order: [
      'cookieParser',
      'session',
      'passportInit',            // <==== If you're using "passport", you'll want to have its two
      'passportSession',         // <==== middleware functions run after "session".
      'bodyParser',
      'compress',
      'foobar',                  // <==== We can put other, custom HTTP middleware like this wherever we want.
      'poweredBy',
      'router',
      'www',
      'favicon',
    ],


    // An example of a custom HTTP middleware function:
    foobar: (function (){
      console.log('Initializing `foobar` (HTTP middleware)...');
      return function (req,res,next) {
        console.log('Received HTTP request: '+req.method+' '+req.path);
        return next();
      };
    })(),

    // An example of a couple of 3rd-party HTTP middleware functions:
    // (notice that this time we're using an existing middleware library from npm)
    passportInit    : (function (){
      var passport = require('passport');
      var reqResNextFn = passport.initialize();
      return reqResNextFn;
    })(),

    passportSession : (function (){
      var passport = require('passport');
      var reqResNextFn = passport.session();
      return reqResNextFn;
    })()

  },
}
```

##### 覆盖或禁用内置的HTTP中间件

您还可以使用上述策略来覆盖内置中间件，如body parser（请参阅[自定义body parser](https://sailsjs.com/documentation/reference/configuration/sails-config-http#?customizing-the-body-parser)).

> 尽管不推荐这样做，但您甚至可以完全禁用内置的HTTP中间件功能: 只需从`middleware.order`数组中删除它。这可以提供完全的灵活性，但应谨慎使用。如果您选择禁用一个内置中间件，请确保您了解后果。（禁用内置的HTTP中间件可能会导致应用程序工作方式的巨大变化。）


### Sails中的Express中间件

Sails应用程序的一个非常好的事是，他可以充分利用已有的Express/Connect中间件。但当人们尝试这样做时，就会出现一个问题:

> _“我在哪里`app.use()`这件事？”_。

在大多数情况下答案是，在[`sails.config.http.middleware`]中将Express中间件安装为自定义HTTP中间件(https://sailsjs.com/documentation/reference/configuration/sails-config-http). 这会触发它向您的Sails应用程序发送所有HTTP请求，并允许您配置它与其他HTTP中间件相关的运行顺序。

> 你绝不可以重载或删除`route`HTTP中间件。它内置于Sails，没有它，你的应用的显式路由和蓝图路由将无法工作。


##### 将中间件作为策略来使用

要使Express中间件仅应用于特定操作，您可以将Express中间件作为策略包含在内 - 只要确保它为HTTP和虚拟socket请求运行即可。

实现这个，请编辑 [`config/policies.js`](https://sailsjs.com/documentation/reference/configuration/sails-config-policies). 您可以在实际的封装策略中（通常是一个好主意）请求和设置中间件，或者直接在您的policies.js文件中直接请求它。 以下示例对brevit使用后一种策略:

```js
var auth = require('http-auth');
var basic = auth.basic({
  realm: 'admin area'
}, function (username, password, onwards) {
  return onwards(username === 'Tina' && password === 'Bullock');
});

//...
module.exports.policies = {
  '*': [true],

  // Prevent end users from doing CRUD operations on products reserved for admins
  // (uses HTTP basic auth)
  'product/*': [auth.connect(basic)],

  // Everyone can view product pages
  'product/show': [true]
}
```



<!--

  FUTURE:

### Sails中的高级Express中间件

根据您的需求，您可以通过几种不同的方式实现此目的。


一般来说，以下最佳做法适用:

如果你想要一个中间件功能

+ 如果您希望只有当您的app的显式路线或蓝图路线匹配时才运行中间件，则应将其作为策略加入。
+ 这将为所有传入的http请求运行筛选，包括图像，css等。

If you want a middleware function to run for all you should include it at the top of your `config/routes.js` as a wildcard route.  for your controller (both HTTP and virtual) requests
-->






<docmeta name="displayName" value="Middleware">
