# 报错Errors

当对任何模型方法或帮助器的调用失败时，Sails抛出一个[JavaScript错误实例](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error)，有助于诊断出错的地方。


Waterline将这些错误实例标准化，用一致的“err.name”值对它们进行分类，并且在适用的情况下使用“err.code”:

```js
try {
  await Something.create({…});
} catch (err) {
  // err.name
  // err.code
  // …
}
```


### Negotiating errors

错误处理往往是不够的。 （“这不是一个有效的用户名”和“我们无法立即创建新用户”之间存在很大差异）。为了适当地协商各种错误，您需要能够以更深入的方式对它们进行检查。

幸运的是，Sails提供了一些开箱即用的语法糖，无需诉诸try ... catch：[.intercept()](https://sailsjs.com/documentation/reference/waterline-orm/queries/intercept)和[.tolerate()](https://sailsjs.com/documentation/reference/waterline-orm/queries/tolerate).


```javascript
await Something.create({…})
.intercept((err)=>{
 // Return a modified error here (or a special exit signal)
 // and .create() will throw that instead
 err.message = 'Uh oh: '+err.message;
 return err;
});
```


| Property       | Type          | Details            |
|:---------------|---------------|:-------------------|
| name           | ((string))    | The broad classification of the error. <br/><br/> e.g.`'UsageError'`     |
| message        | ((string))    | <em>See [.message](https://nodejs.org/dist/latest-v7.x/docs/api/errors.html#errors_error_message).</em> |
| stack          | ((string))    | <em>See [.stack](https://nodejs.org/dist/latest-v7.x/docs/api/errors.html#errors_error_stack).<em>     |
| _code_         | ((string?))   | A narrower classification of the error that is sometimes included.<br/><br/>e.g. `'E_UNIQUE'`       |

当使用与Waterline交互的代码时（通常通过模型方法），可能会遇到几种不同类型的错误。


### 使用errors

当一个error有`name：UsageError'`时，这表明Waterline方法被错误地使用，或者被用无效的选项执行（例如，试图创建一个违反模型的[高级验证规则之一]

这种错误可能来自任何模型方法。

```
err.name === 'UsageError'
```

### 适配器errors

适配器错误通常表示底层适配器有问题，而不是请求本身。 当数据库脱机，出现权限问题，某些特定于数据库的溢出案例，或者适配器错误时，可能会发生这种情况。这种错误将具有`name：'AdapterError'`。

这种错误可能来自任何模型方法。

```
err.name === 'AdapterError'
```


##### E_UNIQUE

当唯一值与数据库中另一条记录的值冲突时，会发生唯一性错误。虽然这被认为是一个适配器错误，但它有自己的`code`以区别于正常的适配器错误: `code: 'E_UNIQUE'`.

这种错误只能来自于`.create()`, `.update()`, `.addToCollection()`, 和 `.replaceCollection()`.

```
err.code === 'E_UNIQUE'
```

### 例子

在Sails应用中使用方法取决于您是使用“await”，promises还是callbacks。


##### 通过`await`协商errors

处理尝试在action中创建新用户时可能发生的不同错误:

```javascript
await User.create({ emailAddress: inputs.emailAddress })
// Uniqueness constraint violation
.intercept('E_UNIQUE', (err)=> {
  return 'emailAlreadyInUse';
})
// Some other kind of usage / validation error
.intercept('UsageError', (err)=> {
  return 'invalid';
});
// If something completely unexpected happened, the error will be thrown as-is.

return exits.success();
```

##### 用callbacks或promises链来协商错误

如果因为使用Node.js <= v7.9而无法使用`await`，则准备好：错误处理在使用callbacks或[promises时有点不同](https://github.com/mikermcneil/parley/tree/49c06ee9ed32d9c55c24e8a0e767666a6b60b7e8#flow-control) instead of `await`.

> 如果可能，请使用“await”！ 对你的应用程序来说更安全，你的代码会更干净，而且你会更快乐。

例如，如果您使用的是promises，那么尝试创建新用户时可能发生的不同错误:

```javascript
User.create({
  emailAddress: req.param('emailAddress')
})
.then(function (){
  res.ok();
})
// Uniqueness constraint violation
.catch({ code: 'E_UNIQUE' }, function (err) {
  res.sendStatus(409);
})
// Some other kind of usage / validation error
.catch({ name: 'UsageError' }, function (err) {
  res.badRequest();
})
// If something completely unexpected happened.
.catch(function (err) {
  res.serverError(err);
});
```

这里是同样的例子，但是用传统的Node.js回调代替promise编写:

```javascript
User.create({
  emailAddress: req.param('emailAddress')
})
.exec(function (err){
  if (err && err.code === 'E_UNIQUE') {
    return res.sendStatus(409);
  } else if (err && err.name === 'UsageError') {
    return res.badRequest();
  } else if (err) {
    return res.serverError(err);
  }

  return res.ok();
});
```

> 但要小心[未捕获的例外](https://github.com/mikermcneil/parley/tree/49c06ee9ed32d9c55c24e8a0e767666a6b60b7e8#handling-uncaught-exceptions)!


<docmeta name="displayName" value="Errors">
