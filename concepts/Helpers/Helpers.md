# 助手Helpers

从版本1.0开始，所有Sails apps都内置了对**助手**的支持，这些简单的实用程序可让您在多个地方共享Node.js代码。 这可以帮助您避免重复劳动，并减少错误、减少重写来提高开发效率。 和actions2类似，这也使得为您的应用创建文档，变得更容易。

### 概述

在Sails中，helpers将重复代码放入单独文件，然后在各种[action](https://sailsjs.com/documentation/concepts/actions-and-controllers)，[自定义响应]中重用该代码(https://sailsjs.com/documentation/concepts/extending-sails/custom-responses），[命令行脚本]（https://www.npmjs.com/package/machine-as-script)，[unit 测试](https://sailsjs.com/documentation/concepts/testing)，或者其他可用项。 我不需要使用助手-可能一开始你就不需要它们。 但随着代码库的增长，helpers对于应用程序的可维护性将变得越来越重要。 （另外，他们真的很方便。）

例如，在创建Node.js/Sails 应用程序用于响应客户端请求的过程中，有时会发现自己在几个地方重复了代码。这可能会非常容易出错，更不用说烦人了。 幸运的是，有一个很好的解决方案：用一个自定义助手的调用替换重复的代码:

```javascript
var greeting = await sails.helpers.formatWelcomeMessage('Bubba');
sails.log(greeting);
// => "Hello, Bubba!"
```

> helpers可以从代码中的任何地方调用; 只要该地点可以访问[`sails`应用程序实例](https://sailsjs.com/documentation/reference/application).


### 如何定义助手

这是一个简单的示例, 明确定义helper:

```javascript
// api/helpers/format-welcome-message.js
module.exports = {

  friendlyName: 'Format welcome message',


  description: 'Return a personalized greeting based on the provided name.',


  inputs: {

    name: {
      type: 'string',
      example: 'Ami',
      description: 'The name of the person to greet.',
      required: true
    }

  },


  fn: async function (inputs, exits) {
    var result = `Hello, ${inputs.name}!`;
    return exits.success(result);
  }

};
```

虽然很简单，但该文件表现出了helper的优势：它以名称和描述开头。可以立即了解该实用程序的功能，并描述其使用方式。这以最简单的方式完成了一项离散任务。


> 看起来熟悉？ helpers遵循与 [shell scripts](https://sailsjs.com/documentation/concepts/shell-scripts) and [actions2](https://sailsjs.com/documentation/concepts/actions-and-controllers#?actions-2)相同的规范.

##### 函数`fn`

helper的核心是`fn`函数，它包含helper将运行的实际代码。 该函数有两个参数：`input`（a dictionary of input values或者 "argins"）和`exits`（a dictionary of callback functions）。 `fn`的工作是利用和处理argins，然后触发一个control back。 请注意，与使用`return`为调用者提供输出的典型Javascript函数相反，helpers通过control back传递给`exits.success()`来提供该结果值。


##### 输入inputs

在helper中声明的_inputs_类似于典型的Javascript函数的参数：它们定义了代码必须使用的值。 但是，与标准JavaScript函数参数不同，输入会自动验证。 如果使用错误类型的argins输入调用相应的助手，缺少所需输入的值时，将触发错误。 因此，助手是_self-validating_。

inputs字典中定义了助手的输入。 每个输入定义至少由一个`type`属性组成。 助手inputs支持类型:

* `string` - a string value
* `number` - a number value (both integers and floats are valid)
* `boolean` - the value `true` or `false`
* `ref` - a Javascript variable reference.  This can be _any_ value, including dictionaries, arrays, functions, streams, and more.

这里有一些[预定义模型属性](https://sailsjs.com/documentation/concepts/models-and-orm/attributes)的相同数据类型（和相关语义）。
正如你所期望的那样，你可以通过设置它的`defaultsTo`属性来为输入提供一个默认值。 或者通过设置`required：true`来设置它。 你甚至可以使用`allowNull`和进阶验证规则，如`isEmail`。

调用helper时传入的参数与该helper声明的“输入”中的键的顺序相对应。如果您希望按名称传递参数，请使用`.with（）`:

```javascript
var greeting = await sails.helpers.formatWelcomeMessage.with({ name: 'Bubba' });
```

##### 退出Exits

Exits描述了helper可能具有的所有不同可能的结果，无论好坏。每个助手都自动支持`error`和`success`退出。
当调用helper时，如果它的`fn`触发`success`，那么它将正常返回。 但是，如果它的`fn`触发其他退出，那么它会抛出一个错误（除非使用[`.tolerate()`](https://sailsjs.com/documentation/reference/waterline-orm/queries/tolerate)。）

必要时，您还可以开放其他自定义退出（称为“exceptions”），调用助手的用户级代码处理特定的例外情况。
这有助于确保您的代码的透明度和可维护性，因为它可以轻松的发布和沟通错误。

> 在`exits`字典中定义了助手的特殊情况（自定义退出）。可以使用显式的`description`属性提供所有自定义异常。

设想一个名为“邀请新用户”的helper， 其中开放了一个自定义的`emailAddressInUse`出口。 如果提供的电子邮件已存在，助手的`fn`可能会触发此自定义退出，从而允许您的用户级代码处理此特定场景-而不会混淆您的结果值或诉诸额外的try/catch结构。


例如，如果这个helper调用它的自定义出口“badRequest”:

```javascript
var newUserId = sails.helpers.inviteNewUser('bubba@hawtmail.com')
.intercept('emailAddressInUse', 'badRequest');
```

> 上面看起来很花哨的简写只是一个更快的写法:
>
> ```javascript
> .intercept('emailAddressInUse', (err)=>{
>   return 'badRequest';
> });
> ```
>
> 至于[.intercept()](https://sailsjs.com/documentation/reference/waterline-orm/queries/intercept)？
这是一个捷径，您不必强制编写自定义的try/catch块并手动协商这些错误。

在程序内部，助手的`fn`负责触发它的一个出口 - 或者通过抛出一个[特殊的退出信号]()或者通过调用一个退出回调函数（例如`exits.success('foo')`）。 如果你的帮手通过sucess退出（例如`'foo'`）并返回结果，那么这将是helper的返回值。

> 注意：对于非成功退出，如果需要，Sails将使用退出的预定义描述,并自动创建适合的JavaScript错误实例。

##### 同步助手

默认情况下，所有助手都被视为异步(_asynchronous_)。 尽管这是默认假设很不错，但事实还可以更好 - 当您确定不使用异步，您可以通过告诉Sails使用`sync：true`属性来优化性能。

如果你知道你的助手的`fn`中的所有代码都是同步的，你可以设置顶级`sync`属性为`true`，这允许userland代码[不需要'等待'调用助手](https：//sailsjs.com/documentation/concepts/helpers#?synchronous-usage)。
(您还必须记得将`fn：async function`更改为`fn：function`。)

> 注意：不使用“await”调用异步助手将不起作用。


##### 在助手中访问`req`
如果你正在设计一个帮助程序来分析request headers，特别是在actions使用，那么可以利用预制方法或[request object](https://sailsjs.com/documentation/reference/request-req)。 你的actions代码将`req`传递给你的helper的最简单的方法是定义一个`type：'ref'`输入:

```javascript
inputs: {

  req: {
    type: 'ref',
    description: 'The current incoming request (req).',
    required: true
  }

}
```


然后，为了在你的行为中使用你的助手，你可以编写这样的代码:

```javascript
var headers = await sails.helpers.parseMyHeaders(req);
```

### 创建一个helper

Sails提供了一个内置的生成器，您可以使用它自动创建一个新的helper:

```bash
sails generate helper foo-bar
```
这个命令将创建一个文件`api/helpers/ foo-bar.js`，可以在你的代码中使用`sails.helpers.fooBar`访问它。 该文件将是一个没有输入且只有默认退出（`success`和`error`）的通用helper，它在执行时立即触发其“success”退出。

### 调用helper
当Sails应用程序加载时，它会查找`api/helpers/`中的所有文件，将它们编译成函数，并使用驼峰文件名将它们存储在`sails.helpers`字典中。 然后可以从代码中调用，调用它只需使用`await`，并提供针对值:

```javascript
var result = await sails.helpers.formatWelcomeMessage('Dolly');
sails.log('Ok it worked!  The result is:', result);
```

> 这可能与您已经熟悉的[model methods](sailsjs.com/documentation/concepts/models-and-orm/models)中的`.create()`一样。

##### 同步使用

如果一个助手声明`sync`属性，你也可以这样调用它（没有`await`）:

```javascript
var greeting = sails.helpers.formatWelcomeMessage('Timothy');
```

但在删除`await`之前，需确保助手实际是同步的。 （否则，如果没有“await”，它将永远不会执行！）


### 处理异常
对于更细微的错误处理（以及那些不是_quite_错误的例外情况），您可能会习惯于设置某种错误代码，然后嗅探它。 这种方法可以正常工作，但可能很费时并且难以追踪。

幸运的是，Sails helper将这件事进一步推进了几步。 有关更多信息，请参阅[.tolerate()]()，[.intercept()]()和[special exit signals]()上的页面。


<!--
For future reference, see https://github.com/balderdashy/sails-docs/commit/61f0039d26021c8abf4873aa675c409372dc2f8f
for the original content of these docs.
-->

##### 或多或少需要
虽然这个例子的用法有些夸张，但很容易看到像依赖notUnique这样的自定义出口，非常有用的场景。 尽管如此，你不想随时处理每个自定义退出。 理想情况下，如果您真的需要，只需要处理用户级代码中的自定义退出：无论是实现某种功能，还是仅仅为了改善用户体验或提供更好的内部错误消息。

幸运的是，Sails助手支持“自动退出转发”。 这意味着，用户级代码可以根据具体情况，选择几个或多少个自定义退出集成。 换句话说，当你调用一个helper时，如果你不需要，完全忽略它的自定义`notUnique`出口是可以的。 这样，您的代码尽可能简洁直观。 如果事情发生变化，您可以随时回来并勾选一些代码以便稍后处理自定义退出。


### 下一步

+ [Explore a practical example](https://sailsjs.com/documentation/concepts/helpers/example-helper) of a helper in a Node.js/Sails app.
+ `sails-hook-organics` (which is bundled in the "Web App" template) comes with several free, open-source, and MIT-licensed helpers for many common use cases.  [Have a look!](https://npmjs.com/package/sails-hook-organics)
+ [Click here](https://sailsjs.com/support) if you're unsure about helpers, or if you want to see more tutorials and examples.

<docmeta name="displayName" value="Helpers">
<docmeta name="nextUpLink" value="/documentation/concepts/deployment">
<docmeta name="nextUpName" value="Deployment">
