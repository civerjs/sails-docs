# 脚本和控制器

### 简介

_Actions_ 是您的Sails应用程序中的首要对象，它们负责响应来自Web浏览器的*请求*, 同时应用于移动APP或任何其他能够与服务器通信的系统.  他们作为模型和视图的中间件。 [models](https://sailsjs.com/documentation/concepts/models-and-orm) and [views](https://sailsjs.com/documentation/concepts/views). 除了少数例外, actions负责项目中的大部分操作&rsquo;s [business logic](http://en.wikipedia.org/wiki/Business_logic).

Actions和路由绑定[routes](https://sailsjs.com/documentation/concepts/Routes), 以便当用户请求特定的URL时，会执行绑定的action对业务逻辑发送响应。  例如, 在项目中访问 `GET /hello` 路由，他绑定action是:

```javascript
async function (req, res) {
  return res.send('Hi there!');
}
```

任何适合当用户访问 `/hello` 这个URL, 页面会显示: &ldquo;Hi there!&rdquo;.

### actions在哪里定义?
Actions 在 `api/controllers/` 文件夹和子文件夹 (we&rsquo;ll talk more about _controllers_ in a bit). 为了在文件中识别action, action的格式必须是_kebab-cased_（仅包含小写字母，数字和破折号）.  当引用Sails中的action(例如, 引用[binding it to a route](https://sailsjs.com/documentation/concepts/routes/custom-routes#?action-target-syntax)), 使用相对路径 `api/controllers`,不需要文件扩展名.  例如, 文件 `api/controllers/user/find.js` 可以直接访问 `user/find`.

##### actions 扩展名

action 可以使用任意后缀，除了 `.md` (Markdown) 和 `.txt` (text). 默认情况下, Sails只解析 `.js` 文件, 但你可以自定项目APP使用 [CoffeeScript](https://sailsjs.com/documentation/tutorials/using-coffee-script) 或者 [TypeScript](https://sailsjs.com/documentation/tutorials/using-type-script).

### action 文件的命名规则？

Action 文件可以使用两种命名方式: _classic_ or _actions2_.

##### 经典 actions

创建Sails action的传统方式是将其声明为函数。 当客户端请求绑定该操作的路由时， 函数将调用 [传入请求](https://sailsjs.com/documentation/reference/request-req) 作为第一个对象 (通常名为 `req`), 或者[传出请求](https://sailsjs.com/documentation/reference/response-res) 作为第二个对象 (通常名为 `res`).  以下是一个示例，它根据ID查找用户，如果无法找到用户，则显示“欢迎”视图或重定向到注册页面：

```javascript
module.exports = async function welcomeUser (req, res) {

  // Get the `userId` parameter from the request.
  // This could have been set on the querystring, in
  // the request body, or as part of the URL used to
  // make the request.
  var userId = req.param('userId');

   // If no `userId` was specified, or it wasn't a number, return an error.
  if (!_.isNumeric(userId)) {
    return res.badRequest(new Error('No user ID specified!'));
  }

  // Look up the user whose ID was specified in the request.
  var user = await User.findOne({ id: userId });

  // If no user was found, redirect to signup.
  if (!user) { return res.redirect('/signup' );

  // Display the welcome view, setting the view variable
  // named "name" to the value of the user's name.
  return res.view('welcome', {name: user.name});

}
```

##### actions2

另一种更结构化的创建action的方法是将其写入更现代的（“actions2”）语法中。 与Sails [helpers](https://sailsjs.com/documentation/concepts/helpers) 的工作方式大致相同, 通过用声明来定义你的动作 ("_machine_"), 本质上是自我记录和自我验证.  这是与上面相同的操作，使用actions2格式重写:

```javascript
module.exports = {

   friendlyName: 'Welcome user',

   description: 'Look up the specified user and welcome them, or redirect to a signup page if no user was found.',

   inputs: {
      userId: {
        description: 'The ID of the user to look up.',
        // By declaring a numeric example, Sails will automatically respond with `res.badRequest`
        // if the `userId` parameter is not a number.
        type: 'number',
        // By making the `userId` parameter required, Sails will automatically respond with
        // `res.badRequest` if it's left out.
        required: true
      }
   },

   exits: {
      success: {
        responseType: 'view',
        viewTemplatePath: 'pages/welcome'
      },
      notFound: {
        description: 'No user with the specified ID was found in the database.',
        responseType: 'notFound'
      }
   },

   fn: async function (inputs, exits) {

      // Look up the user whose ID was specified in the request.
      // Note that we don't have to validate that `userId` is a number;
      // the machine runner does this for us and returns `badRequest`
      // if validation fails.
      var user = await User.findOne({ id: inputs.userId });

      // If no user was found, respond "notFound" (like calling `res.notFound()`)
      if (!user) { return exits.notFound(); }

      // Display the welcome view.
      return exits.success({name: user.name});
   }
};
```

Sails使用模块[machine-as-action](https://github.com/treelinehq/machine-as-action)自动创建路由功能，就像上面的例子一样。 查看 [machine-as-action docs](https://github.com/treelinehq/machine-as-action#customizing-the-response) 以了解更多信息.

> 请注意machine-as-action提供actions访问 [request object] (https://sailsjs.com/documentation/reference/request-req) as `this.req`.

<!--
Removed in order to reduce the amount of information:  (Mike nov 14, 2017)

and to the Sails application object (in case you don&rsquo;t have [globals](https://sailsjs.com/documentation/concepts/globals) turned on) as `this.sails`.
-->

虽然在actions中使用经典函数 `req, res` 是非常简洁的用法，但是使用actions2有如下几个优点:

 * 你的代码不直接依赖于 `req` 或 `res`, 使得代码更容易重用，或者抽象为[helper](https://sailsjs.com/documentation/concepts/helpers).
 * 你可以快速确定请求的参数和名称，并且你可以使action在运行之前进行自动验证。
 * 可以直观的看到action所有可能的运行结果，而不用去分析代码。

简而言之，您的代码将以一种形式进行标准化，以便稍后重新使用和修改。 由于您会提前声明action参数，因此您将不太可能暴露封包和安全漏洞。

###### Exit 操作

在动作、助手或脚本中，默认情况下，抛出任何东西都会触发`error`退出。 如果想出发其他退出方式,可以通过抛出一个 "special exit signal". 可以采用一个字符串 (the name of the exit), 或者以出口名称作为key并输出数据值的object。
例如，可以这样代替通常的语法:

```javascript
return exits.hasConflictingCourses();
```

可以采用抛出:

```javascript
throw 'hasConflictingCourses';
```

或者，包含输出数据:

```javascript
throw { hasConflictingCourses: ['CS 301', 'M 402'] };
```

除了易于读写, 退出信号在 `for` 循环中也特别有用, 还有`forEach`等等, 都可以通过这种方式退出.

### Controllers 控制器

编写Sails应用程序的最快方法是将您的action组织到_controller files_中。  控制器文件使用帕斯卡命名规则 [_PascalCased_](https://en.wikipedia.org/wiki/PascalCase) 最后必须以 `Controller`结束,并包含一个动作字典. 例如, 一个"用户控制器"创建 `api/controllers/UserController.js`文件并包含:

```javascript
module.exports = {
  login: function (req, res) { ... },
  logout: function (req, res) { ... },
  signup: function (req, res) { ... },
};
```

你可以使用 [`sails generate controller`] (https://sailsjs.com/documentation/reference/command-line-interface/sails-generate#?sails-generate-controller-foo-action-1-action-2) 快速创建控制器文件.

##### controllers 控制器的后缀名

控制器可以使用除了 `.md` (Markdown) and `.txt` (text)之外的任意后缀名.默认情况下, Sails 只解析`.js` 文件, 但你可以自定义后缀名如 [CoffeeScript](https://sailsjs.com/documentation/tutorials/using-coffee-script) 或者 [TypeScript](https://sailsjs.com/documentation/tutorials/using-type-script).


### 独立 actions

对于大型、更成熟的APP, _standalone actions_(独立 actions) 可能是比控制器更好的方法. 在这个方案中，不是每个动作都存在一个文件中，而是每个动作都在它自己的`api / controllers`的适当子文件夹中. 以下文件结构等同于 `UserController.js` 文件:

```
api/
 controllers/
  user/
   login.js
   logout.js
   signup.js
```

这三个Javascript文件中的每一个都导出`req，res`函数或者一个actions2定义。

使用独立actions与控制器controllers相比有几个优点:

* 只需查看文件夹中包含的文件，而不是分析控制器文件中的代码，就可以更轻松地跟踪应用中包含的操作。
* 每个action文件都很小，易于维护，而控制器文件往往会随着您的应用程序的增长而增长。
* [路由指向独立 actions](https://sailsjs.com/documentation/concepts/routes/custom-routes#?action-target-syntax) 指向子文件夹比嵌套控制器文件更为直观 (`foo/bar/baz.js` vs. `foo/BarController.baz`).

* Blueprint index 路由适用于顶级独立actions，因此您可以创建一个`api / controllers / index.js`文件并将其自动绑定到您的app的&rsquo;s `/` 路由（而不必创建任意控制器文件来指向根目录）。


### 精益求精

在大多数MVC框架的传统中，成熟的Sails app 通常具有“瘦”的控制器 - 也就是说，由于可重用的代码已被移入 [helpers] (https://sailsjs.com/documentation/concepts/helpers)，或偶尔提取到单独的节点模块中。这种方法绝对可以让您的应用程序更容易维护，因为程序的复杂性日益增长。

但与此同时，过早地将代码推广到可重复使用的helpers中会导致维护问题，浪费时间和降低生产力。 所以要在两者之间寻找平衡。

Sails建议采用以下一般经验法则：**等到你第三次使用相同的代码时，再把它放到一个单独的助手中。**但和任何习惯一样，需要你自己判断！ 如果代码很长或很复杂，那么将它添加到helpers中可能是有意义的。 相反，如果您知道自己构建的是快速，一次性原型，则可以复制并粘贴代码以节省时间。

> 无论您是为激情还是利润而开发，在一天结束时，作为工程师，您的目标都是尽可能地利用您的时间。 有些日子需要编写代码，其他日子要寻找项目的长期可维护性。 如果你不确定这些目标中的哪一个在你目前的发展阶段中更重要，你可以退后一步思考一下。  (更好的是和自己的团队沟通或者 [other folks building apps on Node.js/Sails](https://sailsjs.com/support).)

<docmeta name="displayName" value="Actions and controllers">
<docmeta name="nextUpLink" value="/documentation/concepts/views">
<docmeta name="nextUpName" value="Views">
