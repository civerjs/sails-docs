# 常问问题


### 我可以使用环境变量吗？

是的! 像任何Node应用程序一样，您可以使用环境变量`process.env`。

Sails还内置创建自定义设置的支持，该设置将暴露在`sails.config`上。无论是自定义设置还是内置设置，sails.config中的任何配置属性都可以使用环境变量进行覆盖。 请参阅有关[配置](https://sailsjs.com/documentation/concepts/configuration)的概念性文档.


### 在哪里放置我的生产数据库凭据？ 还有其他设置？

将配置添加到Sails应用程序的最简单方法是修改`config /`中的文件或添加新的文件。 Sails支持开箱即用的环境设置，使用`config /env/production.js`进行设置。 有关详细信息，请参阅[配置](https://sailsjs.com/documentation/concepts/configuration)上的概念性文档。

但有时候，你会遗忘存储库中的某些配置信息。 **这种配置的最佳位置是在环境变量中。**

也就是说，使用环境变量进行开发（例如，在您的笔记本电脑上）有时可能会有些尴尬。 因此，用于部署特定的设置，想要保密的证书，您可以使用`config/local.js`文件。该文件默认包含在`.gitignore`文件中-这有助于防止您无意中将凭据提交到代码库。


**config/local.js**
```javascript
// Local configuration
// 
// Included in the .gitignore by default,
// this is where you include configuration overrides for your local system
// or for a production deployment.
//
// For example, to use port 80 on the local machine, override the `port` config
module.exports = {
    port: 80,
    environment: 'production',
    adapters: {
        mysql: {
            user: 'root',
            password: '12345'
        }
    }
}
```



### 我如何在服务器上获得我的Sails应用程序？

如果你使用的是Heroku或Modulus这样的Paas，这很简单：只需根据他们的提示操作.

否则，请将你的服务器和`ssh`的IP地址放到服务器上。 然后`npm install -g sails`和`npm install -g forever`首次在服务器上从NPM全局安装Sails和`forever`。 最后`git clone`你的项目（如果它不在git仓库中，将其'scp'到服务器上）到服务器上的一个新文件夹中，`cd`它，然后运行`forever start app.js`。


### 我对性能应该有什么期待？

Sails的基础性能与从标准 Node.js/Express 应用程序中获得的性能相当。一个字，快！ 我们已经在Sails核心中做了一些优化，我们重点不会乱从我们的依赖关系中获得的东西。有关性能测试，请参阅[性能测试](http://serdardogruyol.com/sails-vs-rails-a-quick-and-dirty-benchmark）。

Sails应用程序最常见的性能瓶颈是数据库。一个不断增长用户的应用程序的整个生命周期中，为您的表设置好的索引以及使用分页查询变得越来越重要。 最终，随着数据库增长到数千万条记录，您将开始手动查找和优化缓存，通过调用[`.query（）`])https://sailsjs.com/documentation/reference/waterline-orm/models/query)或[`.native（）`](https://sailsjs.com/documentation/reference/waterline-orm/models/native)，或者使用NPM的底层数据库驱动程序）。



### 在连接中session内存警告是什么？

如果您在Sails应用程序中使用session，则不应使用内存session存储。内存session存储是一种仅限于开发的工具，不能扩展到多个服务器; 即使你只有一台服务器，它也不是特别有效（参见[＃3099](https://github.com/balderdashy/sails/issues/3099) and [#2779](https://github.com/balderdashy/sails/issues/2779))。


有关配置产品session存储的说明，请参见[sails.config.session](https://sailsjs.com/documentation/reference/configuration/sails-config-session)。如果您想完全禁用session支持，请关闭`.sailsrc`文件中的`session`钩子:

```javascript
"hooks": {
  "session": false
}
```


<docmeta name="displayName" value="FAQ">

