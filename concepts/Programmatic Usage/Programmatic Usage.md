# 以编程方式创建Sails应用程序

### 概述

大多数情况下，您将通过[命令行界面](https://sailsjs.com/documentation/reference/command-line-interface)与Sails进行交互，使用[`sails lift`](https://sailsjs.com/documentation/reference/command-line-interface/sails-lift)启动服务器。但是，Sails应用程序也可以使用[编程接口](https://sailsjs.com/documentation/reference/application)从其他Node程序中启动和操作。此接口的主要用途之一是在自动测试中运行Sails应用程序。


### 以编程方式创建Sails应用程序

要在Node.js脚本中创建新的Sails应用程序，请使用Sails构造函数。 您可以使用相同的构造函数来创建任意多个不同的Sails应用程序:

```javascript
var Sails = require('sails').constructor;
var mySailsApp = new Sails();
var myOtherSailsApp = new Sails();
```

### 以编程方式配置，启动和停止Sails应用程序

一旦你有了一个新的Sails应用程序的引用，你可以使用[`.load()`](https://sailsjs.com/documentation/reference/application/sails-load)或[`.lift()`] (https://sailsjs.com/documentation/reference/application/sails-lift)来启动它。 两种方法都有两个参数：一个选项配置字典和一个将在Sails应用程序启动后运行的回调函数。

> 当以编程方式启动Sails时，它仍将使用当前工作目录下的`api`，`config`和其他文件夹来加载控制器，模型和配置选项。 一个值得注意的例外是，以这种方式启动应用程序时，不会加载`.sailsrc`文件。

> 任何作为参数发送到`.load()`或`.lift()`的配置选项将优先于从其他地方加载的选项。

> 通过环境变量设置的配置选项将不会自动应用于以编程方式启动的Sails应用程序，但“NODE_ENV”和“PORT”除外。

> 要从`.sailsrc`文件和环境变量加载配置选项，可以使用sails通过`require（'sails / accessible / rc'）`提供的`rc`模块。

The difference between `.load()` and `.lift()` is that `.lift()` takes the additional steps of (1) running the app's [bootstrap](https://sailsjs.com/documentation/reference/configuration/sails-config-bootstrap), if any, and (2) starting an HTTP server on the port configured via `sails.config.port` (1337 by default).  This allows you to make HTTP requests to the lifted app.  To make requests to an app started with `.load()`, you can use the [`.request()`](https://sailsjs.com/documentation/reference/application/sails-request) method of the loaded app.


##### .lift()

在端口1338上使用`.lift（）`启动应用程序并通过HTTP发送POST请求:

```javascript
var request = require('request');
var Sails = require('sails').constructor;

var mySailsApp = new Sails();
mySailsApp.lift({
  port: 1338
  // Optionally pass in any other programmatic config overrides you like here.
}, function(err) {
  if (err) {
    console.error('Failed to lift app.  Details:', err);
    return;
  }

  // --•
  // Make a request using the "request" library and display the response.
  // Note that you still must have an `api/controllers/FooController.js` file
  // under the current working directory, with an `index` action,
  // or a `/foo` or `POST /foo` route set up in `config/routes.js`.
  request.post('/foo', function (err, response) {
    if (err) {
      console.log('Could not send HTTP request.  Details:', err);
    }
    else {
      console.log('Got response:', response);
    }

    // >--
    // In any case, whether the request worked or not, now we need to call `.lower()`.
    mySailsApp.lower(function (err) {
      if (err) {
        console.log('Could not lower Sails app.  Details:',err);
        return;
      }

      // --•
      console.log('Successfully lowered Sails app.');

    });//</lower sails app>
  });//</request.post() :: send http request>
});//</lift sails app>
```

使用当前环境和.sailsrc设置使用`.lift（）`启动应用程序:

```javascript
var Sails = require('sails').constructor;

var rc = require('sails/accessible/rc');

var mySailsApp = new Sails();
mySailsApp.lift(rc('sails'), function(err) {

});
```

##### .load()

下面是前一个例子的一个替代方法：使用`.load（）`启动一个Sails应用程序并发送同样的POST请求 - 但是这次，我们将使用虚拟请求代替HTTP:

```javascript
mySailsApp.load({
  // Optionally pass in any programmatic config overrides you like here.
}, function(err) {
  if (err) {
    console.error('Failed to load app.  Details:', err);
    return;
  }

  // --•
  // Make a request using the "request" method and display the response.
  // Note that you still must have an `api/controllers/FooController.js` file
  // under the current working directory, with an `index` action,
  // or a `/foo` or `POST /foo` route set up in `config/routes.js`.
  mySailsApp.request({url:'/foo', method: 'post'}, function (err, response) {
    if (err) {
      console.log('Could not send virtual request.  Details:', err);
    }
    else {
      console.log('Got response:', response);
    }

    // >--
    // In any case, whether the request worked or not, now we need to call `.lower()`.
    mySailsApp.lower(function (err) {
      if (err) {
        console.log('Could not lower Sails app.  Details:',err);
        return;
      }

      // --•
      console.log('Successfully lowered Sails app.');

    });//</lower sails app>
  });//</send virtual request to sails app>
});//</load sails app (but not lift!)>
```

##### .lower()

要以编程方式停止应用程序，请使用`.lower()`:

```javascript
mySailsApp.lower(function(err) {
  if (err) {
     console.log('An error occured when attempting to stop app:', err);
     return;
  }

  // --•
  console.log('Lowered app successfully.');

});
```

##### 使用`moduleDefinitions`来添加actions，模型等等

> **Warning:**  Declarative loading of modules with the `moduleDefinitions` setting is **currently experimental**, and may undergo breaking changes _even between major version releases_.  Before using this setting, be sure your project's Sails dependency is pinned to an exact version (i.e. no `^`).

Whenever a Sails app starts, it typically loads and initializes all modules stored in `api/*` (e.g. models from `api/models`, policies from `api/policies`, etc.).  You can add _additional_ modules by specifying them in the runtime configuration passed in as the first argument to `.load()` or `.lift()`, using the `moduleDefinitions` key.  This is mainly useful when running tests.

The following Sails modules can be added programmatically:

  Module type          | Config key        | Details
 :------------------   |:----------        |:-------
 Actions | `controllers.moduleDefinitions` | A dictionary mapping [standalone action](https://sailsjs.com/documentation/concepts/actions-and-controllers#?standalone-actions) paths to action definitions ([classic](https://sailsjs.com/documentation/concepts/actions-and-controllers#?classic-actions) or [Actions2](https://sailsjs.com/documentation/concepts/actions-and-controllers#?actions-2)).
 Helpers | `helpers.moduleDefinitions` | A dictionary mapping helper names to helper definitions.
 Models  | `orm.moduleDefinitions.models` | A dictionary mapping model identities (lower-cased model names) to model definitions.
 Policies | `policies.moduleDefinitions` | A dictionary mapping policy names (e.g. `isAdmin`) to policy functions.


### 参考

Sails程序界面的完整参考文献可在 [**Reference > Application**](https://sailsjs.com/documentation/reference/application).

<docmeta name="displayName" value="Programmatic usage">
