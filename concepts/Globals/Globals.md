# 全局

### 概述

为了方便，Sails公开了一些全局变量。 默认情况下，您应用的[models](https://sailsjs.com/documentation/reference/Models)，[services](https://sailsjs.com/documentation/reference/Services)和全局`sails`对象都在全局提供; 这意味着，你可以通过你的后端代码中的任何地方引用它们（只要Sails[已经加载](https://github.com/balderdashy/sails/tree/master/lib/app)）。

Sails核心中没有任何内核依赖于这些全局变量 - Sails中暴露的每个全局对象都可以在`sails.config.globals`中被禁用（通常在`config/globals.js`中配置）。


### 应用程序对象 (`sails`)
在大多数情况下，您会维持`sails`对象的全局可访问性-它使您的应用程序代码更加清洁。 但是，如果你禁用了所有全局变量，包括`sails`，则需要访问请求对象（`req`）上的`sails`。


### 模型和服务
你的APP的[模型](https://sailsjs.com/documentation/reference/Models) and [services](https://sailsjs.com/documentation/reference/Services) 使用它们的`globalId`作为全局变量公开. 例如，在`api/models/Foo.js`文件中定义的模型将作为`Foo`全局访问，并且在`api/services/Baz.js`中定义的服务将以`Baz`的形式提供。

### Async (`async`) and Lodash (`_`)
Sails还将[lodash](http://lodash.com)的实例公开为`_`，并将[async](https://github.com/caolan/async)的实例公开为`async`。 默认提供这些常用的实用程序，因此您不必在每个新项目中都“安装”它们。 像sails中其他全局变量一样，它们可以被禁用。



<docmeta name="displayName" value="Globals">
