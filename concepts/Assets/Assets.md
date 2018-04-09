# Assets

### 概述

Assets请参考 [static files](http://en.wikipedia.org/wiki/Static_web_page) (js, css, images, etc) 这一些在服务器上提供外界访问的文件.  在Sails中, 这些文件被放置在 [`assets/`](https://sailsjs.com/documentation/anatomy/assets) 文件夹.  当你升级你的App, 或修改现有assets, Sails' 内置asset管理进程和程序将把这些文件放入隐藏文件夹(`.tmp/public/`).

> 这个中间步骤（将文件从`assets/`移动到`.tmp/ public/`）允许Sails进行预处理，客户端使用的 assets - LESS，CoffeeScript，SASS，Spritesheets，Jade模板等。

`.tmp/public`文件夹内容是Sails在实际运行时提供的内容。大致相当于[express]（https://github.com/expressjs）中的“public”文件夹，或者您可能熟悉的Web服务器（如Apache）的 `www/` 文件夹。


### 静态中间件

在后台, Sails使用Express的[serve-static middleware](https://www.npmjs.com/package/serve-static) 为assets提供服务. 你可以配置这个中间件 (e.g. to change cache settings)在 [`/config/http.js`](https://sailsjs.com/documentation/reference/configuration/sails-config-http).

##### `index.html`
和大多数web服务器一样, Sails支持 `index.html`约定.例如, 如果你在一个新项目中创建 `assets/foo.html`, 可以使用 `http://localhost:1337/foo.html`访问. 如果创建 `assets/foo/index.html`, 可以通过下面两种方式访问 `http://localhost:1337/foo/index.html` 和 `http://localhost:1337/foo`.

##### 优先权
It is important to note that the static [middleware](http://stephensugden.com/middleware_guide/) is installed **after** the Sails router.  So if you define a [custom route](https://sailsjs.com/documentation/concepts/Routes?q=custom-routes), but also have a file in your assets directory with a conflicting path, the custom route will intercept the request before it reaches the static middleware. For example, if you create `assets/index.html`, with no routes defined in your [`config/routes.js`](https://sailsjs.com/documentation/reference/configuration/sails-config-routes) file, it will be served as your home page.  But if you define a custom route, `'/': 'FooController.bar'`, that route will take precedence.



<docmeta name="displayName" value="Assets">
<docmeta name="nextUpLink" value="/documentation/concepts/shell-scripts">
<docmeta name="nextUpName" value="Shell Scripts">

