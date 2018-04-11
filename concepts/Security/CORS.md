# 跨源资源共享 (CORS)

<!--
Every Sails app comes ready to handle AJAX requests from a web page on the same domain.  But what if you need to handle AJAX requests
originating from other domains?
-->

[CORS](http://en.wikipedia.org/wiki/Cross-origin_resource_sharing) is a mechanism that allows browser scripts on pages served from other domains (e.g. myothersite.com) to talk to your server (e.g. api.mysite.com).  Like [JSONP](https://en.wikipedia.org/wiki/JSONP), the goal of CORS is to circumvent the [same-origin policy](http://en.wikipedia.org/wiki/Same-origin_policy); allowing your Sails server to successfully respond to requests from client-side JavaScript code running on a page hosted from some other domain.  But unlike JSONP, it works with more than just GET requests.  And it allows you to whitelist particular origins (`staging.yoursite.com` or `yourothersite.net`) and prevent requests from others (`evil.com`).

Sails can be configured to allow cross-origin requests from a list of domains you specify, or from every domain.  This can be done on a per-route basis, or globally for every route in your app.

### 启用CORS

出于安全原因，Sails中默认禁用CORS。 但启用它非常简单。

要允许"可信域白名单"的"跨域请求"在您的应用中使用路由，只需启用“allRoutes”并提供“origin”，请设置[`config/cors.js`](https://sailsjs.com/docs/reference/configuration/sails-config-cors):

```javascript
allRoutes: true,
allowOrigins: ['http://example.com','https://api.example.com','http://blog.example.com:1337','https://foo.com:8888']
```

要在应用中允许跨域请求，请使用`allowOrigins：'*'`:

```javascript
allRoutes: true,
allowOrigins: '*',
allowCredentials: false
```
请注意，在使用`allowOrigins：'*'`时，`credentials`设置_必须为`false`，这意味着包含cookie的请求将被阻止。 此限制防止第三方网站诱骗您的登录用户向应用程序发出未经授权的请求。 您可以使用[`allowAnyOriginWithCredentialsUnsafe`](https://sailsjs.com/docs/reference/configuration/sails-config-security-cors)设置取消此限制（风险自负！）。


请参阅 [`sails.config.security.cors`](https://sailsjs.com/documentation/reference/configuration/sails-config-security-cors)以获取所有可用选项的全面参考。


### 配置CORS对于单个路由
除`config / security.js`中的全局CORS配置外，您还可以在[`config / routes.js`]中按照根据路由配置这些设置(https://sailsjs.com/anatomy/config/routes-js).

如果你在`config / cors.js`中设置了`allRoutes：true`，但你想豁免一个特定的路由，那么在路由的目标中设置`cors：false`:

```javascript
'POST /signup': {
   action: 'user/signup',
   cors: false
}
```

要启用或覆盖特定路由的全局CORS配置，请将`cors`作为字典:

```javascript
'GET /videos': {
   action: 'video/find',
   cors: {
     allowOrigins: ['http://example.com','https://api.example.com','http://blog.example.com:1337','https://foo.com:8888'],
     allowCredentials: false
   }
}
```

### 注意

> + CORS支持仅与HTTP请求相关。 通过socket进行的请求不受限制。为了确保您的应用程序通过socket是安全的，请配置[`onlyAllowOrigins`](https://sailsjs.com/documentation/reference/configuration/sails-config-sockets)设置（通常在[`config/env/production.js`](https://sailsjs.com/documentation/anatomy/config/env/production-js)。
> + Internet Explorer 7不支持CORS。在IE8及更高版本以及所有其他现代浏览器中均受支持。
> + Read [more about CORS from MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS)
> + Read the [CORS spec](https://www.w3.org/TR/cors/)

<docmeta name="displayName" value="CORS">
