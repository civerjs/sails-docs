# 蓝图

### 概述

与任何良好的Web框架一样，Sails的目标是减少您编写的代码量，并减少启动运行应用程序所需的时间。 _Blueprints_ 是Sails快速生成API [路由](https://sailsjs.com/documentation/concepts/routes)和 [actions](https://sailsjs.com/documentation/concepts/controllers#?actions)的方式，以您的应用程序设计为基础。

[蓝图路由](https://sailsjs.com/documentation/concepts/blueprints/blueprint-routes)和[蓝图actions](https://sailsjs.com/documentation/concepts/blueprints/blueprint-actions)组成**blueprint API**，这为创建[RESTful JSON API](http://en.wikipedia.org/wiki/Representational_state_transfer)格式的model和控制器提供动力的内置逻辑。


例如, 在项目中创建一个文件名为`User.js`的model, 在启用蓝图的情况下，你可以访问`/user/create?name=joe`来创建用户, 访问`/user`查看app所有用户.  这些都不需要写一行代码!

蓝图是用于原型制作的强大工具，但在许多情况下也可用于生产,因为它们可以被overridden, protected, extended或者完全禁用.

### 接下来

+ [了解更多](https://sailsjs.com/documentation/concepts/blueprints/blueprint-actions) 关于内置的蓝图actions
+ [了解更多](https://sailsjs.com/documentation/concepts/blueprints/blueprint-routes) 怎么样重写或者配置隐式 "影子" 路由

<docmeta name="displayName" value="Blueprints">
