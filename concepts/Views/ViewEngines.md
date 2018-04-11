# 视图引擎

Sails的默认视图引擎是[EJS](https://github.com/mde/ejs).

##### 更换视图引擎

要使用不同的视图引擎，您应该使用npm将其安装到您的项目中，然后在[`config/views.js`]中设置`sails.config.views.extension`到你想要的文件扩展名和`sails.config.views.getRenderFn`到一个返回你的视图引擎的渲染函数的函数。

如果您的视图引擎受[Consolidate]）支持，您可以在`getRenderFn`中使用它来轻松访问渲染功能。首先，你需要使用npm在项目中安装`consolidate`，如果不存在的话:

```bash
npm install consolidate --save
```
安装完成后，您已经安装了您的视图引擎包，然后您可以设置视图配置。 例如，要使用[Swig]模板，您需要`npm install swig --save`，然后将以下内容添加到[`config / views.js`]:

```javascript
'extension': 'swig',
'getRenderFn': function() {
  // Import `consolidate`.
  var cons = require('consolidate');
  // Return the rendering function for Swig.
  return cons.swig;
}
```

`getRenderFn`允许您在将其插入Sails之前配置您的视图引擎:

```javascript
'extension': 'swig',
'getRenderFn': function() {
  // Import `consolidate`.
  var cons = require('consolidate');
  // Import `swig`.
  var swig = require('swig');
  // Configure `swig`.
  swig.setDefaults({tagControls: ['{?', '?}']});
  // Set the module that Consolidate uses for Swig.
  cons.requires.swig = swig;
  // Return the rendering function for Swig.
  return cons.swig;
}
```

<docmeta name="displayName" value="View engines">
