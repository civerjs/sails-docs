# 布局layouts

在构建多页面的应用程序时，将多个HTML文件共享的标记放置到layouts会有帮助。这会[减少项目中的代码总量](http://en.wikipedia.org/wiki/Don't_repeat_yourself)，并避免在多个文件中进行相同的更改。

在Sails和Express中，layout由视图引擎自己实现。例如，`jade`有它自己的layout系统，有自己的语法。

为方便起见，当使用默认视图引擎EJS时，**Sails捆绑了对layout**的特殊支持。 如果您想使用具有不同视图引擎的layout，请查看[该视图引擎的文档](https://sailsjs.com/documentation/concepts/concepts/views/view-engine)以查找适当的语法。


### 创建布局

Sails layout是应用的`views/`文件夹中的`.ejs`文件，您可以用它来“包含”或“植入”其他视图。 layout通常包含前导码（例如`<！DOCTYPE html> <html> <head> .... </ head> <body>`）和结尾（`</ body> </ html>`）。然后使用`<％- body ％>`包含原始视图文件。 布局绝不会在没有视图的情况下使用。

您可以在[`config / views.js`](https://sailsjs.com/documentation/anatomy/config/views.js)配置或禁用布局支持，并且可通过设置一个名为`layout`的特殊[local](https://sailsjs.com/documentation/concepts/views/locals)覆盖特定路由或操作。默认情况下，Sails将使用位于`views/layouts/layout.ejs`的布局来编译所有视图。

要指定视图使用的layout，请参见下面的示例[routes](https://sailsjs.com/documentation/concepts/routes).

下面的示例路由使用位于`./views/users.ejs`的布局内的`./views/users/privacy.ejs`视图。

```javascript
'get /privacy': {
    view: 'users/privacy',
    locals: {
      layout: 'users'
    }
  },
```

下面的示例控制器action将使用位于`./views/users.ejs`的布局内的位于`./views/users/privacy.ejs`的视图。

```javascript
privacy: function (req, res) {
  res.view('users/privacy', {layout: 'users'})
}
```

### 注意

> #### 为什么布局只适用于EJS？
> A couple of years ago, built-in support for layouts/partials was deprecated in Express. Instead, developers were expected to rely on the view engines themselves to implement this features. (See https://github.com/balderdashy/sails/issues/494 for more info on that.)
>
> Sails supports the legacy `layouts` feature for convenience, backwards compatibility with Express 2.x and Sails 0.8.x apps, and in particular, familiarity for new community members coming from other MVC frameworks. As a result, layouts have only been tested with the default view engine (ejs).
>
> If layouts aren&rsquo;t your thing, or (for now) if you&rsquo;re using a server-side view engine other than ejs, (e.g. Jade, handlebars, haml, dust) you&rsquo;ll want to set `layout:false` in [`sails.config.views`](https://sailsjs.com/documentation/reference/configuration/sails-config-views), then rely on your view engine&rsquo;s custom layout/partial support.





<docmeta name="displayName" value="Layouts">
