# 国际化

### 概述

如果您的app会涉及来自世界各地的人员或系统，则国际化和本地化（也称为“i18n”）可能是您国际战略的重要组成部分。 这对于主用户群分散于不同语言的应用程序尤其重要：例如提供西班牙文和英文内容的教程站点，或者遍布魁北克省和不列颠哥伦比亚省的客户的在线商店。

幸运的是，Sails为检测用户语言偏好和翻译静态单词/句子提供了内置支持。 从Sails v1开始，使用轻量级[`i18n-node-2`包](https://www.npmjs.com/package/i18n-2)实现。 除了这里介绍的内容外，这个包还提供了几个额外的选项，您可以在README文件中阅读这些选项。对于许多具有基本国际化要求的Node.js/Sails.js 应用程序，下面的简单用法就是您所需要的。


### 用法

在Sails中，使用request header指定的区域设置很容易翻译单词和短语:

来自一个视图:
```ejs
<h1> <%= __('Hello') %> </h1>
<h1> <%= __('Hello %s, how are you today?', 'Mike') %> </h1>
<p> <%= i18n('That\'s right-- you can use either i18n() or __()') %> </p>
```


##### 覆盖language headers

有时，重写浏览器/设备语言头文件非常有用 - 例如，如果您想允许用户设置自己的语言首选项。 无论这种偏好是基于session的还是与其在数据库中的帐户相关联的，使用[`req.setLocale（）`](https://sailsjs.com/documentation/reference/request-req/req-set-locale)完成这项工作是非常简单的.


##### 国际化一个shell script

最后，如果您使用Sails构建[命令行脚本](https://sailsjs.com/documentation/concepts/shell-scripts) 或者追求其他高级用例，则还可以将abritrary字符串转换为[ 配置默认语言环境](https://sailsjs.com/documentation/reference/configuration/sails-config-i-18-n)从几乎任何地方使用`sails.__`您的应用程序:

```javascript
sails.__('Welcome');
// => 'Bienvenido'

sails.__('Welcome, %s', 'Mary');
// => 'Bienvenido, Mary'
```

<!--

  FUTURE: See https://trello.com/c/7GusjTTX

-->

### 语言环境

查看 [**Concepts > Internationalization > Locales**](https://sailsjs.com/documentation/concepts/internationalization/locales)了解更多关于创建您的语言环境文件（又名“stringfiles”）的信息。


### 其他选项

本地化/国际化设置可以在[`config/i18n.js`]（https://sailsjs.com/documentation/reference/configuration/sails-config-i-18-n）中配置。 您需要修改这些设置的最常见原因是编辑应用支持的语言环境列表。

有关配置Node.js / Sails.js应用程序国际化设置的更多信息，请参阅[sails.config.i18n](https://sailsjs.com/documentation/reference/configuration/sails-config-i-18-n)。


### 禁用或自定义Sails的默认国际化支持

您可以随时在您的项目的任何位置 `require()`任何您喜欢的节点模块，并使用您所需的任何国际化策略。

但值得注意的是，由于Sails在[i18n hook](https://sailsjs.com/documentation/concepts/Internationalization)中实现了[node-i18n-2](https://github.com/jeresig/i18n-node-2)，您可以使用[`loadHooks`](https://github.com/balderdashy/sails-docs/blob/master/PAGE_NEEDED.md)和/或[`hooks`](https://github.com/balderdashy/sails-docs/blob/master/PAGE_NEEDED.md)完全禁用或覆盖它的配置选项。


### 翻译动态内容

See [**Concepts > Internationalization > Translating dynamic content**](https://sailsjs.com/documentation/concepts/internationalization/translating-dynamic-content).


### 关于客户端的i18n呢？

上述技术对于服务器端视图来说非常有用。 但是，从CDN或静态主机提供静态HTML模板的胖客户端应用程序呢？ <!-- (e.g. performance-sensitive SPAs, Chrome extensions, or webview apps built with tools like Ionic, PhoneGap, etc.) -->

那么，最简单的选择就是保持服务器渲染视图的国际化。 但是如果你不想这样做，还有[很多不同的选项可用](http://stackoverflow.com/questions/9640630/javascript-i18n-internationalization-frameworks-libraries-for-client-side-use)。 像其他客户端技术一样，将它们中任何一个与Sails集成都没有问题。

> 如果您不想使用外部国际化库，您实际上可以重新使用Sails的i18n支持来帮助您将翻译后的模板发送到浏览器。 如果您想使用Sails将浏览器端模板国际化，请将您的前端模板放在应用程序`/views`文件夹的子目录中。
> + 在开发模式下，每次使用grunt-contrib-watch更改相关字符串文件或模板时，应该重新编译和预编译模板，grunt-contrib-watch默认情况下已安装在新的Sails项目中。
> + 在产品模式下，您需要翻译并预编译lift()中的所有模板。 在加载时间很重要的情况下（例如移动网络应用程序），您甚至可以将已翻译的，预编译的缩小模板上载到像Cloudfront这样的CDN，以获得进一步的性能提升。


<docmeta name="displayName" value="Internationalization">
