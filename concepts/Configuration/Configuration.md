# 配置

### 概述

Sails忠实遵守[convention-over-configuration](http://en.wikipedia.org/wiki/Convention_over_configuration)的理念，对于如何定义方便使用的默认配置非常重要。几乎Sails中的每个惯例，都有一组配置选项，您可以调整或覆盖以适应您的需求。

> 需要寻找某个设置？ 请参阅[参考>配置](sailsjs.com/documentation/reference/configuration)以查看Sails中所有可用配置选项的完整指南。

Sails应用程序可以[以编程方式配置](https://github.com/mikermcneil/sails-generate-new-but-like-express/blob/master/templates/app.js#L15)，通过指定[环境变量](http://en.wikipedia.org/wiki/Environment_variable)或命令行参数，通过更改本地或全局[`.sailsrc`文件](https://sailsjs.com/documentation/anatomy/.sailsrc) 或（最常见的）使用通常位于新项目的[`config/`](https://sailsjs.com/documentation/anatomy/config)文件夹中的样板文件。在运行时可以在sails全局中作为sails.config使用。


### 标准配置文件 (`config/*`)

默认情况下，新的Sails应用程序中会生成大量配置文件。 这些样板包含大量内嵌声明，旨在提供快速，实时的参考，而无需在文档和文本编辑器之间来回切换。

在大多数情况下，`sails.config`对象上的顶级key（例如`sails.config.views`）对应于您的应用中的特定配置文件（例如`config/views.js`）; 然后，你可以通过你的`config/`目录中的文件来配置设置。 配置key的名称非常重要(i.e. key)-而不是它指向哪个的文件。

例如，假设您添加一个新文件， `config/foo.js`:

```js
// config/foo.js
// The object below will be merged into `sails.config.blueprints`:
module.exports.blueprints = {
  shortcuts: false
};
```
有关各个配置选项以及默认情况下它们所在的文件的详尽参考，请查看本节中的参考页面，或查看[The Anatomy of a Sails App](https://sailsjs.com/documentation/anatomy)中的更多章节["`config /`"](https://sailsjs.com/documentation/anatomy/config)。


### 特定环境文件 (`config/env/*`)

在标准配置文件中指定的设置通常可用于所有环境（即开发，生产，测试等）。 如果您希望某些设置仅在特定环境中生效，则可以使用特定环境特定的文件和文件夹:

* Any files saved under the `/config/env/<environment-name>` folder will be loaded *only* when Sails is lifted in the `<environment-name>` environment.  For example, files saved under `config/env/production` will only be loaded when Sails is lifted in production mode.
* Any files saved as `config/env/<environment-name>.js` will be loaded *only* when Sails is lifted in the `<environment-name>` environment, and will be merged on top of any settings loaded from the environment-specific subfolder.  For example, settings in `config/env/production.js` will take precedence over those in the files in the  `config/env/production` folder.

By default, your app runs in the "development" environment.  The recommended approach for changing your app's environment is by using the `NODE_ENV` environment variable:
```
NODE_ENV=production node app.js
```

> The `production` environment is special-- depending on your configuration, it enables compression, caching, minification, etc.
>
> Also note that if you are using `config/local.js`, the configuration exported in that file takes precedence over environment-specific configuration files.


### `config/local.js`文件

您可以使用`config/local.js`文件为您的本地环境（例如您的笔记本电脑）配置Sails应用程序。 此文件中的设置优先于除[.sailsrc]之外的所有其他配置文件(https://sailsjs.com/documentation/concepts/configuration/using-sailsrc-files)。由于它们仅用于本地使用，因此不应将其置于版本控制之下（因此，它包含在默认的`.gitignore`文件中）。使用 `local.js`存储本地数据库设置, 更改本地调试端口,等待.

查看[Concepts > Configuration > The local.js file](https://sailsjs.com/documentation/concepts/configuration/the-local-js-file) 获得更多内容.


### 在您的应用中访问`sails.config`

`config`对象在Sails应用程序实例（`sails`）上可用。 默认情况下，会在升级过程中暴露在[global scope(https://sailsjs.com/documentation/concepts/globals) 上，因此可以在应用程序的任何位置使用。

##### 示例
```javascript
// 此示例检查是否处于生产模式，否则启用csrf。
// It throws an error and crashes the app otherwise.
if (sails.config.environment === 'production' && !sails.config.security.csrf) {
  throw new Error('STOP IMMEDIATELY ! CSRF should always be enabled in a production deployment!');
}
```

### 直接使用环境变量设置`sails.config`值

In addition to using configuration _files_, you can set individual configuration values on the command line when you lift Sails by prefixing the config key names with `sails_`, and separating nested key names with double-underscores (`__`).  Any environment variable formatted this way will be parsed as JSON (if possible). For example, you could do the following to set the [allowed CORS origins](https://sailsjs.com/documentation/concepts/security/cors) (`sails.config.security.cors.allowOrigins`) to `["http://somedomain.com","https://anotherdomain.com:1337"]` on the command line:

```javascript
sails_security__cors__allowOrigins='["http://somedomain.com","https://anotherdomain.com:1337"]' sails console
```

> Note the use of double-quotes to indicate strings within the JSON-encoded value, and the single quotes surrounding the whole value so that it is passed correctly from to Sails from the console.

This value will be in effect _only_ for the lifetime of this particular Sails instance, and will override any values in the configuration files.

Also note that configuration specified using environment variables does _not_ automatically apply to Sails instances that are started [programmatically](https://sailsjs.com/documentation/concepts/programmatic-usage).

> There are a couple of special exceptions to the above rule: `NODE_ENV` and `PORT`.
> + `NODE_ENV` is a convention for any Node.js app.  When set to `'production'`, it sets [`sails.config.environment`](https://sailsjs.com/documentation/reference/configuration/sails-config#?sailsconfigenvironment).
> + Similarly, `PORT` is just another way to set [`sails.config.port`](https://sailsjs.com/documentation/reference/configuration/sails-config#?sailsconfigport).  This is strictly for convenience and backwards compatibility.
>
> Here's a relatively common example where you might use both of these environment variables at the same time:
>
> ```bash
> PORT=443 NODE_ENV=production sails lift
> ```
>
> When present in the current process environment, `NODE_ENV` and `PORT` will apply to any Sails app that is started via the command line or programmatically, unless explicitly overridden.

Environment variables are one of the most powerful ways to configure your Sails app.  Since you can customize just about any setting (as long as it's JSON-serializable), this approach solves a number of problems, and is our core team's recommended strategy for production deployments.  Here are a few:

+ Using environment variables means you don't have to worry about checking in your production database credentials, API tokens, etc.
+ This makes changing Postgresql hosts, Mailgun accounts, S3 credentials, and other maintenance straightforward, fast, and easy; plus you don't need to change any code or worry about merging in downstream commits from other people on your team
+ Depending on your hosting situation, you may be able to manage your production configuration through a UI (most PaaS providers like [Heroku](http://heroku.com) or [Modulus](https://modulus.io) support this, as does [Azure Cloud](https://azure.microsoft.com/en-us/).)


### Setting `sails.config` values using command-line arguments

For situations where setting an environment variable on the command line may not be practical (such as some Windows systems), you can use regular command-line arguments to set configuration options.  To do so, specify the name of the option prefixed by two dashes (`--`), with nested key names separated by dots.  Command-line arguments are parsed using [minimist](https://github.com/substack/minimist/tree/0.0.10), which does _not_ parse JSON values like arrays or dictionaries, but will handle strings, numbers and booleans (using a special syntax).  Some examples:

```javascript
// Set the port to 1338
sails lift --port=1338

// Set a custom "email" value to "foo@bar.com":
sails lift --custom.email='foo@bar.com'

// Turn on CSRF support
sails lift --security.csrf

// Turn off CSRF support
sails lift --no-security.csrf

// This won't work; it'll just try to set the value to the string "[1,2,3]"
sails lift --custom.array='[1,2,3]'
```

### Custom configuration

You can also leverage Sails's configuration loader to manage your own custom settings.  See [sails.config.custom](https://sailsjs.com/config/sails-config-custom) for more information.



### Configuring the command-line interface

When it comes to configuration, most of the time you'll be focused on managing the runtime settings for a particular app: the port, database setup, and so forth.  However it can also be useful to customize the Sails CLI itself; to simplify your workflow, reduce repetitive tasks, perform custom build automation, etc.  Thankfully, Sails v0.10 added a powerful new tool to do just that.

The [`.sailsrc` file](https://sailsjs.com/documentation/anatomy/.sailsrc) is unique from other configuration sources in Sails in that it may also be used to configure the Sails CLI-- either system-wide, for a group of directories, or only when you are `cd`'ed into a particular folder.  The main reason to do this is to customize the [generators](https://sailsjs.com/documentation/concepts/extending-sails/Generators) that are used when `sails generate` and `sails new` are run, but it can also be useful to install your own custom generators or apply hard-coded config overrides.

And since Sails will look for the "nearest" `.sailsrc` in the ancestor directories of the current working directory, you can safely use this file to configure sensitive settings you can't check in to your cloud-hosted code repository (_like your **database password**_.)  Just include a `.sailsrc` file in your "$HOME" directory.  See [the docs on `.sailsrc`](https://sailsjs.com/documentation/anatomy/.sailsrc) files for more information.


### Order of precedence for configuration

Depending on whether you're starting a Sails app from the command line using `sails lift` or `node app.js`, or programmatically using [`sails.lift()`](https://sailsjs.com/documentation/reference/application/advanced-usage/sails-lift) or [`sails.load()`](https://sailsjs.com/documentation/reference/application/advanced-usage/sails-load), Sails will draw its configuration from a number of sources, in a certain order.

##### Order of precedence when starting via `sails lift` or `node app.js` (in order from highest to lowest priority):

+ command-line options parsed by [minimist](https://github.com/substack/minimist/tree/0.0.10); e.g. `sails lift --custom.mailgun.apiToken='foo'` becomes `sails.config.custom.mailgun.apiToken`.
+ [environment variables](https://en.wikipedia.org/wiki/Environment_variable) prefixed with `sails_`, and using double underlines to indicate dots; e.g.: `sails_port=1492 sails lift` ([A few more examples](https://gist.github.com/mikermcneil/92769de1e6c10f0159f97d575e18c6cf)).
+ a [`.sailsrc` file](https://sailsjs.com/documentation/concepts/configuration/using-sailsrc-files) in your app's directory, or the first found looking in `../`, `../../` etc.
+ a global `.sailsrc` file in your home folder (e.g. `~/.sailsrc`).
+ any existing `config/local.js` file in your app.
+ any existing `config/env/*` files in your app that match the name of your current NODE_ENV environment (defaulting to `development`).
+ any other files in your app's `config/` directory (if one exists).

##### Order of precedence when starting programmatically (in order from highest to lowest priority):

+ an optional dictionary (`{}`) of configuration overrides passed in as the first argument to `.lift()` or `.load()`.
+ any existing `config/local.js` file in your app.
+ any existing `config/env/*` files in your app that match the name of your current NODE_ENV environment (defaulting to `development`).
+ any other files in your app's `config/` directory (if one exists).


### Notes
> The built-in meaning of the settings in `sails.config` are, in some cases, only interpreted by Sails during the "lift" process.  In other words, changing some options at runtime will have no effect.  To change the port your app is running on, for instance, you can't just change `sails.config.port`-- you'll need to change or override the setting in a configuration file or as a command-line argument, etc., then restart the server.



<docmeta name="displayName" value="Configuration">
<docmeta name="nextUpLink" value="/documentation/concepts/policies">
<docmeta name="nextUpName" value="Policies">
