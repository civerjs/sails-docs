# CSRF

跨站请求伪造（[CSRF](https://www.owasp.org/index.php/Cross-Site_Request_Forgery)）是一种攻击类型，他/她通过身份验证，迫使最终用户在Web应用程序后端执行不需要的操作。 换句话说，无需保护，无论用户目前是在访问Chase.com还是Horrible-Hacker-Site.com，都可以使用存储在Google Chrome等浏览器中的Cookie向Chase.com发送请求。

### 关于CSRF tokens

CSRF tokens像限量版藏品。虽然session告诉服务器用户“他们是谁”，但csrf令牌告诉服务器他们“他们在哪里”。 在Sails应用程序中启用CSRF保护时，所有到服务器的非GET请求都必须附带一个特殊的“CSRF标记”，该标记可以作为'_csrf'参数或'X-CSRF-Token'标头包含。

使用令牌可以保护您的Sails应用程序免受跨站点请求伪造（或CSRF）攻击。 一个潜在的攻击者不仅需要用户的会话cookie，还需要这个时间戳，秘密的CSRF令牌，当用户访问你应用的域名时刷新/授权。 这使您可以确定用户的请求没有被劫持，并且他们提出的请求是有意和合法的。

启用CSRF保护需要在您的前端应用程序中管理令牌。 在传统的表单提交中，通过将CSRF标记作为隐藏输入发送到<form>中，可以轻松完成。 或者当发送AJAX请求时，请将CSRF令牌作为请求参数或headers包含在内。 要做到这一点，您可以通过向安装`security/grant-csrf-token`的路由发送请求来获取令牌，或者更好的方法是使用`exposeLocalsToBrowser`部分从视图本地获取令牌。

这里有些例子:

#### (a) 适用于AJAX提交表单的视图混合应用程序:
使用`exposeLocalsToBrowser`部分来提供对令牌的访问
您的客户端JavaScript，例如:
```html
<%- exposeLocalsToBrowser() %>
<script>
  $.post({
    foo: 'bar',
    _csrf: window.SAILS_LOCALS._csrf
  })
</script>
```

#### (b) 适用于带有静态HTML的单页应用程序:
Fetch the token by sending a GET request to the route where you mounted
the `security/grant-csrf-token`.  It will respond with JSON, e.g.:
```js
{ _csrf: 'ajg4JD(JGdajhLJALHDa' }
```

#### (c) 对于传统的HTML表单提交:
Render the token directly into a hidden form input element in your HTML, e.g.:
```html
<form>
  <input type="hidden" name="_csrf" value="<%= _csrf %>" />
</form>
```

### 启用CSRF保护

开箱即装可选的CSRF保护。 要启用只需对[sails.config.security.csrf]进行以下调整即可(https://sailsjs.com/docs/reference/configuration/sails-config-security-csrf) (conventionally located in your project's [`config/security.js`](https://sailsjs.com/anatomy/config/security-js) file):

```js
csrf: true
```

You can also turn CSRF protection on or off on a per-route basis by adding `csrf: true` or `csrf: false` to any route in your [`config/routes.js`](https://sailsjs.com/anatomy/config/routes-js) file.

Note that if you have existing code that communicates with your Sails backend via POST, PUT, or DELETE requests, you'll need to acquire a CSRF token and include it as a parameter or header in those requests.  More on that in a sec.



### CSRF令牌

Like most Node applications, Sails and Express are compatibile with Connect's [CSRF protection middleware](http://www.senchalabs.org/connect/csrf.html) for guarding against such attacks.  This middleware implements the [Synchronizer Token Pattern](https://www.owasp.org/index.php/Cross-Site_Request_Forgery_%28CSRF%29_Prevention_Cheat_Sheet#General_Recommendation:_Synchronizer_Token_Pattern).  When CSRF protection is enabled, all non-GET requests to the Sails server must be accompanied by a special token, identified by either a header or a parameter in the query string or HTTP body.

CSRF tokens are temporary and session-specific; e.g. Imagine Mary and Muhammad are both shoppers accessing our e-commerce site running on Sails, and CSRF protection is enabled.  Let's say that on Monday, Mary and Muhammad both make purchases.  In order to do so, our site needed to dispense at least two different CSRF tokens- one for Mary and one for Muhammad.  From then on, if our web backend received a request with a missing or incorrect token, that request will be rejected. So now we can rest assured that when Mary navigates away to play online poker, the 3rd party website cannot trick the browser into sending malicious requests to our site using her cookies.

### 分配CSRF令牌

To get a CSRF token, you should either bootstrap it in your view using [locals](https://sailsjs.com/documentation/concepts/views/locals) (good for traditional multi-page web applications) or fetch it using AJAX from a special protected JSON endpoint (handy for single-page-applications (SPAs).)


##### 使用本地视图:

For old-school form submissions, it's as easy as passing the data from a view into a form action.  You can grab hold of the token in your view, where it may be accessed as a view local: `<%= _csrf %>`

e.g.:
```html
<form action="/signup" method="POST">
 <input type="text" name="emailaddress"/>
 <input type='hidden' name='_csrf' value='<%= _csrf %>'>
 <input type='submit'>
</form>
```
If you are doing a `multipart/form-data` upload with the form, be sure to place the `_csrf` field before the `file` input, otherwise you run the risk of a timeout and a 403 firing before the file finishes uploading.





##### 使用AJAX/WebSockets

In AJAX/Socket-heavy apps, you might prefer to get the CSRF token dynamically rather than having it bootstrapped on the page.  You can do so by setting up a route in your [`config/routes.js`](https://sailsjs.com/anatomy/config/routes-js) file pointing to the `security/grant-csrf-token` action:

```json
{
  "GET /csrfToken": { action: "security/grant-csrf-token" }
}
```

Then send a GET request to the route you defined, and you'll get CSRF token returned as JSON, e.g.:

```json
{
  "_csrf": "ajg4JD(JGdajhLJALHDa"
}
```

> For security reasons, you can&rsquo;t retrieve a CSRF token via a socket request.  You can however _spend_ CSRF tokens (see below) via socket requests.
> The `security/grant-csrf-token` action is not intended to be used in cross-origin requests, since some browsers block third-party cookies by default.  See the [CORS documentation](https://sailsjs.com/documentation/concepts/security/cors) for more info about cross-origin requests.



### 消耗CSRF令牌

Once you've enabled CSRF protection, any POST, PUT, or DELETE requests (**including** virtual requests, e.g. from Socket.io) made to your Sails app will need to send an accompanying CSRF token as a header or parameter.  Otherwise, they'll be rejected with a 403 (Forbidden) response.

For example, if you're sending an AJAX request from a webpage with jQuery:
```js
$.post('/checkout', {
  order: '8abfe13491afe',
  electronicReceiptOK: true,
  _csrf: 'USER_CSRF_TOKEN'
}, function andThen(){ ... });
```

With some client-side modules, you may not have access to the AJAX request itself. In this case, you can consider sending the CSRF token directly in the URL of your query. However, if you do so, remember to URL-encode the token before spending it:
```js
..., {
  checkoutAction: '/checkout?_csrf='+encodeURIComponent('USER_CSRF_TOKEN')
}
```



### 注意

> + You can choose to send the CSRF token as the `X-CSRF-Token` header instead of the `_csrf` parameter.
> + For most developers and organizations, CSRF attacks need only be a concern if you allow users to log into/securely access your Sails backend _from the browser_ (i.e. from your HTML/CSS/JavaScript front-end code). If you _don't_ (e.g. users only access the secured sections from your native iOS or Android app), it is possible you don't need to enable CSRF protection.  Why?  Because technically, the common CSRF attack discussed on this page is only _possible_ in scenarios where users use the _same client application_ (e.g. Chrome) to access different web services (e.g. Chase.com, Horrible-Hacker-Site.com.)
> + For more information on CSRF, check out [Wikipedia](http://en.wikipedia.org/wiki/Cross-site_request_forgery)
> + For "spending" CSRF tokens in a traditional form submission, refer to the example above (under "Using view locals".)


<docmeta name="displayName" value="CSRF">
