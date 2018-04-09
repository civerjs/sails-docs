# 路由指向actions

### 手动路由指向

默认情况下，用户将无法访问Sails app中控制器的actions，除非您将它们绑定到[`config/routes.js`](https://sailsjs.com/documentation/reference/configuration/sails-config-routes).  绑定路由, 指定一个用户可以访问该action的URL, 诸如此类像 [CORS security settings](https://sailsjs.com/documentation/concepts/security/cors#?configuring-cors-for-individual-routes).

绑定action到路由在 `config/routes.js` 文件中, 你可以使用HTTP verb或者路径 (i.e. the **route address**) 作为关键字, 并将动作作为值 (i.e. the **route target**).

例如，下面的手动路由，当应用接收到一个POST请求`/make/a/sandwich`就会触发 `api/controllers/SandwichController.js`中的`makeIt`动作：

```js
  'POST /make/a/sandwich': 'SandwichController.make'
```

如果你使用独立actions, 你会有一个 `api/controllers/sandwich/make.js` 文件, 使用action路径更直接的语法是 (relative to `api/controllers`):

```js
  'POST /make/a/sandwich': 'sandwich/make'
```

有关路由的完整讨论，请参阅 [routes documentation](https://sailsjs.com/documentation/concepts/Routes).

### 自动路由指向

Sails也可以自动绑定路由到你的控制器动作，发送到`/：actionIdentity`的`GET`请求就会触发这个action。

Sails can also automatically bind routes to your controller actions so that a `GET` request to `/:actionIdentity` will trigger the action.  我们称它为 _blueprint action routing_, 他可以通过设置 `actions` 为 `true`进行激活，请参考 [`config/blueprints.js`](https://sailsjs.com/documentation/reference/configuration/sails-config-blueprints) 文件.  例如, 打开 blueprint action routing 后, `signup` action 存储在 `api/controllers/UserController.js` or `api/controllers/user/signup.js` 会自动绑定到 `/user/signup` 路由.  参考 [blueprints documentation](https://sailsjs.com/documentation/reference/blueprint-api) 获得更多信息关于Sails&rsquo; 自动路由指向.


<docmeta name="displayName" value="Routing to actions">
