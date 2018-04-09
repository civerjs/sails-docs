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

action 可以使用任意后缀，比如 `.md` (Markdown) 或者 `.txt` (text). 默认情况下, Sails只解析 `.js` 文件, 但你可以自定项目APP使用 [CoffeeScript](https://sailsjs.com/documentation/tutorials/using-coffee-script) 或者 [TypeScript](https://sailsjs.com/documentation/tutorials/using-type-script).

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

另一种更结构化的创建action的方法是将其写入更现代的（“actions2”）语法中。 与Sails [helpers](https://sailsjs.com/documentation/concepts/helpers)的工作方式大致相同, 通过用声明来定义你的动作 ("_machine_"), 本质上是自我记录和自我验证.  这是与上面相同的操作，使用actions2格式重写:

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

Sails使用模块[machine-as-action](https://github.com/treelinehq/machine-as-action)自动创建路由处理功能，就像上面的例子一样。 查看 [machine-as-action docs](https://github.com/treelinehq/machine-as-action#customizing-the-response) 以了解更多信息.

> 请注意machine-as-action提供actions访问 [request object](https://sailsjs.com/documentation/reference/request-req) as `this.req`.

<!--
Removed in order to reduce the amount of information:  (Mike nov 14, 2017)

and to the Sails application object (in case you don&rsquo;t have [globals](https://sailsjs.com/documentation/concepts/globals) turned on) as `this.sails`.
-->

Using classic `req, res` functions for your actions is technically less typing.  However, using actions2 provides several advantages:

 * The code you write is not directly dependent on `req` and `res`, making it easier to re-use or abstract into a [helper](https://sailsjs.com/documentation/concepts/helpers).
 * You guarantee that you&rsquo;ll be able to quickly determine the names and types of the request parameters the action expects, and you'll know that they will be automatically validated before the action is run.
 * You&rsquo;ll be able to see all of the possible outcomes from running the action without having to dissect the code.

In a nutshell, your code will be standardized in a way that makes it easier to re-use and modify later.  And since you'll declare the action's parameters ahead of time, you'll be much less likely to expose edge cases and security holes.

###### Exit signals

In an action, helper, or script, throwing anything will trigger the `error` exit by default. If you want to trigger any other exit, you can do so by throwing a "special exit signal". This will either be a string (the name of the exit), or an object with the name of the exit as the key and the output data as the value.
For example, instead of the usual syntax:

```javascript
return exits.hasConflictingCourses();
```

You could use the shorthand:

```javascript
throw 'hasConflictingCourses';
```

Or, to include output data:

```javascript
throw { hasConflictingCourses: ['CS 301', 'M 402'] };
```

Aside from being an easy-to-read shorthand, exit signals are especially useful if you're inside of a `for` loop, `forEach`, etc., but still want to exit through a particular exit.

### Controllers

The quickest way to get started writing Sails apps is to organize your actions into _controller files_.  A controller file is a [_PascalCased_](https://en.wikipedia.org/wiki/PascalCase) file whose name must end in `Controller`, containing a dictionary of actions.  For example, a  "User controller" could be created at `api/controllers/UserController.js` file containing:

```javascript
module.exports = {
  login: function (req, res) { ... },
  logout: function (req, res) { ... },
  signup: function (req, res) { ... },
};
```

You can use [`sails generate controller`](https://sailsjs.com/documentation/reference/command-line-interface/sails-generate#?sails-generate-controller-foo-action-1-action-2) to quickly create a controller file.

##### File extensions for controllers

A controller can have any file extension besides `.md` (Markdown) and `.txt` (text).  By default, Sails only knows how to interpret `.js` files, but you can customize your app to use things like [CoffeeScript](https://sailsjs.com/documentation/tutorials/using-coffee-script) or [TypeScript](https://sailsjs.com/documentation/tutorials/using-type-script) as well.


### Standalone actions

For larger, more mature apps, _standalone actions_ may be a better approach than controller files.  In this scheme, rather than having multiple actions living in a single file, each action is in its own file in an appropriate subfolder of `api/controllers`.  For example, the following file structure would be equivalent to the  `UserController.js` file:

```
api/
 controllers/
  user/
   login.js
   logout.js
   signup.js
```

where each of the three Javascript files exports a `req, res` function or an actions2 definition.

Using standalone actions has several advantages over controller files:

* It's easier to keep track of the actions that your app contains, by simply looking at the files contained in a folder rather than scanning through the code in a controller file.
* Each action file is small and easy to maintain, whereas controller files tend to grow as your app grows.
* [Routing to standalone actions](https://sailsjs.com/documentation/concepts/routes/custom-routes#?action-target-syntax) in nested subfolders is more intuitive than with nested controller files (`foo/bar/baz.js` vs. `foo/BarController.baz`).

* Blueprint index routes apply to top-level standalone actions, so you can create an `api/controllers/index.js` file and have it automatically bound to your app&rsquo;s `/` route (as opposed to having to create an arbitrary controller file to hold the root action).


### Keeping it lean

In the tradition of most MVC frameworks, mature Sails apps usually have "thin" controllers -- that is, your action code ends up lean, because reusable code has been moved into [helpers](https://sailsjs.com/documentation/concepts/helpers) or occasionally even extracted into separate node modules.  This approach can definitely make your app easier to maintain as it grows in complexity.

But at the same time, extrapolating code into reusable helpers _too early_ can cause maintainence issues that waste time and productivity.  So the right answer lies somewhere in the middle.

Sails recommends this general rule of thumb:  **Wait until you're about to use the same piece of code for the _third_ time before you extrapolate it into a separate helper.**  But as with any dogma, use your judgement!  If the code in question is very long or complex, then it might make sense to pull it out into helper a helper much sooner.  Conversely, if you know what you're building is a quick, throwaway prototype, you might just copy and paste the code to save time.

> Whether you're developing for passion or profit, at the end of the day, the goal is to make the best possible use of your time as an engineer.  Some days that means getting more code written, and other days it means looking out for the long-term maintainability of the project.  If you're not sure which of these goals is more important at your current stage of development, you might take a step back and give it some thought.  (Better yet, have a chat with the rest of your team or [other folks building apps on Node.js/Sails](https://sailsjs.com/support).)

<docmeta name="displayName" value="Actions and controllers">
<docmeta name="nextUpLink" value="/documentation/concepts/views">
<docmeta name="nextUpName" value="Views">
