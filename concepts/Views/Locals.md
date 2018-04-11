# 本地变量

在特定视图中可访问的变量称为“本地”。代表你的视图可以访问的服务器端数据 - 除非你使用的视图引擎提供的特殊语法明确引用，否则本地变量实际上并不包含在编译的HTML中。


```ejs
<div>Logged in as <a><%= name %></a>.</div>
```

### 在你的视图中使用变量

视图引擎之间访问变量的符号各不相同。 在EJS中，您使用特殊模板标记（例如`<％=someValue％>`）在视图中包含变量。

EJS中有三种模板标签:
+ `<%= someValue %>`
  + HTML转义`someValue`变量，然后将其包含为一个字符串。
+ `<%- someRawHTML %>`
  + Includes the `someRawHTML` local verbatim, without escaping it.
  + 小心！ 如果您不知道自己在做什么，该标签可能会使您容易受到XSS攻击。
+ `<% if (!loggedIn) { %>  <a>Logout</a>  <% } %>`
  + 编译视图时，在`<％...％>`内运行JavaScript。
  + 用于条件（`if` /`else`）和循环数据（`for` /`each`）。


这是一个使用两个变量（user和corndogs）的视图（`views/backOffice/profile.ejs`）的例子:

```ejs
<div>
  <h1><%= user.name %>'s first view</h1>
  <h2>My corndog collection:</h2>
  <ul>
    <% _.each(corndogs, function (corndog) { %>
    <li><%= corndog.name %></li>
    <% }) %>
  </ul>
</div>
```

> 您可能已经注意到另一个变量“_”。 默认情况下，Sails会自动传递几个变量到你的视图，包括lodash（`_`）。

如果你想传递给这个视图的数据是完全静态的，你不需要一个控制器 - 你可以在你的`config/routes.js`文件中编码视图代码，即:

```javascript
  // ...
  'get /profile': {
    view: 'backOffice/profile',
    locals: {
      user: {
        name: 'Frank',
        emailAddress: 'frank@enfurter.com'
      },
      corndogs: [
        { name: 'beef corndog' },
        { name: 'chicken corndog' },
        { name: 'soy corndog' }
      ]
    }
  },
  // ...
```
另一方面，在更可能的情况下，这些数据是动态的，我们需要使用控制器action从我们的模型中加载它，然后使用[res.view()](https： //sailsjs.com/documentation/reference/response-res/res-view)方法。

假设我们将路由连接到控制器的一个action(模型已经建立），可以像这样发送请求:

```javascript
// in api/controllers/UserController.js...

  profile: function (req, res) {
    // ...
    return res.view('backOffice/profile', {
      user: theUser,
      corndogs: theUser.corndogCollection
    });
  },
  // ...
```

### 使用`exposeLocalsToBrowser`转义不受信任的数据

通常需要将“引导” 数据加载到页面上，以便在页面加载时通过Javascript调用，而不必在单独的AJAX或socket请求中获取数据。 像[Twitter and GitHub](https://blog.twitter.com/2012/improving-performance-on-twittercom)）这样的网站在很大程度上依赖于此方法，以便优化网页加载时间并改善用户体验。

过去，解决这个问题的常见方式是隐藏表单字段，或者通过手动刷新将服务器端变量注入到客户端脚本标记中。 但是，当数据来自_非信任_来源时，它可能包含HTML标记和Javascript代码，这些代码出现<a href =“https：// en .wikipedia.org / wiki / Cross-site_scripting“target =”_ blank“> XSS攻击</a>漏洞。 为了避免这些问题，Sails提供了一个名为`exposeLocalsToBrowser`的内置视图，您可以使用它来从您的视图变量中安全地注入数据，以便从客户端JavaScript访问。

要使用`exposeLocalsToBrowser`，只需使用模板语言的_non-escaping syntax_从视图内调用它。 例如，使用默认的EJS视图引擎，这样做:

```ejs
<%- exposeLocalsToBrowser() %>
```

默认情况下，这会将您的视图局部变量全部暴露为`window.SAILS_LOCALS`全局变量。例如，如果您的操作代码包含:

```javascript
res.view('myView', {
  someString: 'hello',
  someNumber: 123,
  someObject: { owl: 'hoot' },
  someArray: [1, 'boot', true],
  someBool: false
  someXSS: '<script>alert("all your credit cards is belongs to us");</script>'
});
```

那么如上所示使用`exposeLocalsToBrowser`会使变量安全地引导，使得`window.SAILS_LOCALS.someArray`包含数组'`1，'boot'，true]`和`window.SAILS_LOCALS。 someXSS`将包含字符串`<script>提醒（“您的所有信用卡属于我们”）; </ script>`，而不会导致该代码实际在页面上执行。

`exposeLocalsToBrowser`函数有一个`options`参数，可以用来配置输出什么数据。 `options`参数是一个包含以下属性的字典:

|&nbsp;   |     Property        | Type                                         | Default| Details                            |
|---|:--------------------|----------------------------------------------|:-----------------------------------|-----|
| 1 | _keys_     | ((array?))                              | `undefined` | A &ldquo;whitelist&rdquo; of locals to expose.  If left undefined, _all_ locals will be exposed.  If specified, this should be an array of property names from the locals dictionary.  For example, given the `res.view()` statement shown above, setting `keys: ['someString', 'someBool']` would cause `windows.SAILS_LOCALS` to be set to `{someString: 'hello', someBool: false}`.
| 2 | _namespace_ | ((string?)) | `SAILS_LOCALS` | The name of the global variable to assign the bootstrapped data to.
| 3| _dontUnescapeOnClient_ | ((boolean?)) | false | **Advanced. Not recommended for most apps.** If set to `true`, any string values that were escaped to avoid XSS attacks will _still be escaped_ when accessed from client-side JS, instead of being transformed back into the original value.  For example, given the `res.view()` statement from the example above, using `exposeLocalsToBrowser({dontUnescapeOnClient: true})` would cause `window.SAILS_LOCALS.someXSS` to be set to `&lt;script&gt;alert(&#39;hello!&#39;);`.


<docmeta name="displayName" value="Locals">
