# 验证

Sails bundle支持自动验证模型的属性。任何时候更新记录或创建新记录时，都会根据所有预定义验证规则来进行监测。 这提供了一个故障保护，以确保无效条目不会进入您数据库。

除“唯一(unique)”（数据库级约束执行; [请参阅“unique”](https://sailsjs.com/documentation/concepts/models-and-orm/validations#?unique)）外的所有验证。下面的代码是在JavaScript中实现的，并且在与Sails相同的Node.js服务器进程中运行。 还要记住，无论使用哪种验证，属性都必须指定一种内置数据类型（`string`，`number`，`json`等）。


```javascript
// User
module.exports = {
  attributes: {
    emailAddress: {
      type: 'string',
      unique: true,
      required: true
    }
  }
};
```

### 内置数据类型

在Sails / Waterline中，模型属性总是具有某种数据类型。这为开发人员提供一种方法，以便对特定模型中出现或出来的数据保持合理的假设。

该数据类型用于逻辑验证和结果和标准的强制。 以下是Sails和Waterline支持的数据类型列表:

| Data Type        | Usage                         | Description                                                  |
|:----------------:|:----------------------------- |:------------------------------------------------------------ |
| ((string))       | `type: 'string'`              | Any string.
| ((number))       | `type: 'number'`              | Any number.
| ((boolean))      | `type: 'boolean'`             | `true` or `false`.
| ((json))         | `type: 'json'`                | Any JSON-serializable value, including numbers, booleans, strings, arrays, dictionaries (plain JavaScript objects), and `null`.
| ((ref))          | `type: 'ref'`                 | Any JavaScript value except `undefined`. (Should only be used when taking advantage of adapter-specific behavior.)    |

Sails的ORM（Waterline）及其适配器执行宽松验证，以确保字典中提供的值以及`.create()`或`.update()`的值与预期的数据类型相匹配。

**注意：**在本身不支持（json）类型的适配器中，适配器必须以其他方式支持它。 例如，在MySQL中，写入（json）属性的数据获得调用的JSON.stringify()，然后存储在类型设置为text的列中。 每次返回记录时，数据都会调用JSON.parse()。注意性能和与其他应用程序或数据库中现有数据的兼容性。 官方的postgresql和mongodb适配器可以本地读取和写入（json）数据。



##### Null和空字符串

当创建或更新记录时，`string`，`number`和`boolean`数据类型不会接受`null`作为值。 为了允许设置'null'值，您可以在属性上切换'allowNull`标志。 'allowNull`标志只对这些数据类型有效。 对于类型为“json”或“ref”的属性，任何关联或任何主键属性，它都是无效的。

由于空字符串（“”）是一个字符串，它通常由`type：'string'`属性支持。 例外：主键（因为主键从不支持空字符串）和任何具有`required：true`的属性。


##### 必填

如果一个属性是`required：true`，调用`.create()`时必须指定一个值。它还可以防止创建（或更新）该值为'null'或空字符串（“”），

### 验证规则

验证无法对`null`进行 _额外_ 限制。也就是说，如果是`null`将被允许，那么启用`isEmail`验证规则将不会导致`null`无效。

同样，以下验证规则中的大部分都不会对空字符串（“”）施加任何其他限制。 有一些例外（`isNotEmptyString`，以及像`isBoolean`，`isNumber`，`max`和`min`这样的非字符串相关的规则），否则，空字符串（“”） 通常被允许，添加验证规则不会导致它被拒绝。

在下表中，显示哪些数据类型（即属性定义的`type`属性）适的验证规则。 在很多情况下，验证规则可以用于多种类型。 请注意，下面的表格采用了一个快捷方式：如果兼容（字符串），（数字）或（布尔）。



| Name of Rule      | What It Checks For                                                                                                  | Notes On Usage                                         | Compatible Attribute Type(s) |
|:------------------|:--------------------------------------------------------------------------------------------------------------------|:--------------------------------------------------------|:----------------------------:|
| custom            | A value such that when it is provided as the first argument to the custom function, the function returns `true`.                          | [Example](https://sailsjs.com/documentation/concepts/models-and-orm/validations#?custom-validation-rules)            |  _Any_   |
| isAfter           | A value that, when parsed as a date, refers to a moment _after_ the configured JavaScript `Date` instance.          | `isAfter: new Date('Sat Nov 05 1605 00:00:00 GMT-0000')`  | ((string)), ((number))       |
| isBefore          | A value that, when parsed as a date, refers to a moment _before_ the configured JavaScript `Date` instance.         | `isBefore: new Date('Sat Nov 05 1605 00:00:00 GMT-0000')` | ((string)), ((number))       |
| isBoolean         | A value that is `true` or `false` | isBoolean: true | ((json)), ((ref)) |
| isCreditCard      | A value that is a credit card number.                                                                               | **Do not store credit card numbers in your database unless your app is PCI compliant!**  If you want to allow users to store credit card information, a safe alternative is to use a payment API like [Stripe](https://stripe.com). | ((string)) |
| isEmail           | A value that looks like an email address.                                                                           | `isEmail: true`                                         | ((string)) |
| isHexColor        | A string that is a hexadecimal color.                                                                               | `isHexColor: true`                                      | ((string)) |
| isIn              | A value that is in the specified array of allowed strings.                                                          | `isIn: ['paid', 'delinquent']`                          | ((string)) |
| isInteger         | A number that is an integer (a whole number)                                                                        | `isInteger: true`                                       | ((number)) |
| isIP              | A value that is a valid IP address (v4 or v6)                                                                       | `isIP: true`                                              | ((string)) |
| isNotEmptyString  | A value that is _not_ an empty string | `isNotEmptyString: true` | ((json)), ((ref))
| isNotIn           | A value that **is not in** the configured array.                                                                    | `isNotIn: ['profanity1', 'profanity2']`                   | ((string)) |
| isNumber          | A value that is a Javascript number | `isNumber: true` | ((json)), ((ref))
| isString          | A value that is a string (i.e. `typeof(value) === 'string'`) | `isString: true` | ((json)), ((ref))
| isURL             | A value that looks like a URL. | `isURL: true` | ((string)) |
| isUUID            | A value that looks like a UUID (v3, v4 or v5) | `isUUID: true` | ((string))
| max               | A number that is less than or equal to the configured number. | `max: 10000` | ((number)) |
| min               | A number that is greater than or equal to the configured number. | `min: 0` | ((number)) |
| maxLength         | A string that has no more than the configured number of characters. |  `maxLength: 144` | ((string)) |
| minLength         | A string that has at least the configured number of characters. | `minLength: 8` | ((string)) |
| regex             | A string that matches the configured regular expression. | `regex: /^[a-z0-9]$/i` | ((string)) |


##### 示例：可选电子邮件地址

例如，假设您有一个如下定义的属性:

```javascript
workEmail: {
  type: 'string',
  isEmail: true,
}
```

然后当你调用`.create()`或者`.update()`，这个值可以设置为任何有效的电子邮件地址（如“santa@clause.com”）或者空字符串（“”）。 但是，您将无法将其设置为“null”，因为这会违反`type：'string'`强加的类型安全限制。


> 要想使它接受'null'（例如，如果你正在使用已有数据库），把它改为`type：'json'`。 你也可以添加`isString：true`--但由于在这个例子中，我们已经强制执行`isEmail：true`，所以不需要这样做。
> 更高级的是，根据数据库的不同，还可以选择利用(https://sailsjs.com/documentation/concepts/models-and-orm/attributes#?columntype)通知Sails/Waterline在自动迁移期间定义哪种列类型。



##### 例子: 评价星级所需

如果我们想表明一个属性支持某些数字，比如星级，我们可能会做类似以下的事情:

```javascript
starRating: {
  type: 'number',
  min: 1,
  max: 5,
  required: true,
}
```


##### 例子:可选星级

如果我们想让我们的星级评分为可选，最简单的方法就是删除`required：true`标志。 如果省略，starRating将默认为零。


##### 示例：可选星级 (w/ `null`)

但是，如果星级不能成为一个数字呢？ 例如，我们可能需要一个我们需要整合的遗留数据库，其中数据库中已有的星级可能是一个数字或特殊的'null'字面值。 对于这种情况，我们需要一种方法来定义这个属性，以便它可以支持某些数字或空值。

要做到这一点，只需使用 `allowNull`:

```javascript
starRating: {
  type: 'number',
  allowNull: true
  min: 1,
  max: 5,
}
```

> 为方便起见，Sails和Waterline属性支持`allowNull`。 但另一个解决方案是将`starRating`从`type：'number`更改为`type：json`。 由于`json`类型允许其他数据，如布尔值，数组等。由于我们可能防止这种情况，所以要添加`isNumber：true`验证规则:
>
>
> ```javascript
> starRating: {
>   type: 'json',
>   isNumber: true,
>   min: 1,
>   max: 5,
> }
> ```



### 唯一值Unique

`unique`与上面列出的所有验证规则不同。 事实上，它不是一个验证：它是一个**数据库级约束**。 更多关于这一点。

如果一个属性声明自己是`unique：true`，那么Sails确保不会有两个记录被允许具有相同的值。例子：`User`模型上的`emailAddress`属性:

```javascript
// api/models/User.js
module.exports = {

  attributes: {
    emailAddress: {
      type: 'string',
      unique: true,
      required: true
    }
  }

};
```

##### 为什么“unique”与其他验证不同？

想象一下，您的数据库中有1,000,000条用户记录。 如果“unique”与其他验证一样实施，则每当新用户注册您的应用程序时，Sails都需要搜索现有的一千万条记录，以确保没有其他人重复使用。这不仅会很慢，而且在完成这些记录的搜索之后，其他人可能已经注册了！

幸运的是，这种唯一性检查是_any_数据库的最普遍特征。为了利用它，Sails依靠[database adapter](https://sailsjs.com/documentation/concepts/models-and-orm#?adapters)来实现对“unique”的支持 - 具体来说，通过添加一个 **唯一性约束** [自动迁移]期间数据库本身中的相关字段/列/属性(https://sailsjs.com/documentation/concepts/models-and-orm/model-settings#?migrate)。也就是说，当您的应用程序设置为`migrate：alter'`时，Sails将自动生成基础数据库中的表/集合，并且内置唯一性约束。


##### 怎么使用索引?

当你开始使用你的产品数据库时，设置索引来提高数据库的性能总是一个好主意。设置索引的具体过程和最佳实践因数据库而异，超出了本文档的范围。 这就是说，如果你以前从未这样做过，不要担心 - 这比你想象的要容易得多，

就像其他，一旦您将应用设置为使用'migrate：'safe''，Sails将完全由您自己决定数据库索引。

> 请注意，这意味着在执行[手动迁移](https://github.com/BlueHotDog/sails-migrations)时，您应确保更新您的索引以及唯一性约束。.


### 何时使用验证

验证可以节省时间，防止您编写数百行重复代码。 但请记住，验证在针对每个create或update运行。在您的某个属性定义中使用验证规则之前，请确保您可以在应用程序调用`.create（）`或`.update（）`保存新值时应用它。如果不是这种情况，那么编写一个action来验证控制器内嵌的传入值; 或者在您的[服务](https://sailsjs.com/documentation/concepts/services)或[模型类方法]之一中调用自定义函数(https://sailsjs.com/documentation/concepts/models-and-orm/models#?model-methods-aka-static-or-class-methods)。

例如，假设您的Sails应用允许用户通过
（A）输入电子邮件地址和密码，然后确认该电子邮件地址或
（B）调用LinkedIn登录验证，注册一个帐户。
现在我们假设你的`User`模型有一个名为`linkedInEmail`的属性和另一个名为`manuallyEnteredEmail`的属性。取决于用户所选注册的方式。 因此，在这种情况下，您的`User`模型不能使用`required：true`验证 - 相反，您需要通过在相关的`.create`之前手动验证电子邮件或另一个电子邮件，例如:

```javascript
if ( !_.isString( req.param('email') ) ) {
  return res.badRequest();
}
```

为了更进一步，现在让我们假设您的应用程序接受付款。在注册流程中，如果用户注册了付费计划，他或她也必须提供一个用于计费的电子邮件地址（`billingEmail`）。如果用户使用免费注册，则跳过该步骤。 在帐户设置页面上，付费计划的用户确实可以看到一个“帐单电子邮件”表单字段，他们可以在其中自定义帐单地址。 这与免费计划中的用户有所不同，他们看到链接到“Upgrade Plan”页面的声明。

即使这些要求看起来非常具体，但还有一些尚未解决的问题:

- 当其他默认的电子邮件地址发生变化时，我们是否会自动更新帐单邮件？
- 如果结算电子邮件已被更改，会怎么样？
- 用户降级到免费计划后，结算电邮会发生什么变化？如果用户再次升级到付费计划，我们是否推测他的账单电子邮件地址或使用旧的？
- 当现有用户连接他或她的LinkedIn帐户并保存新的`linkedInEmail`时，结算电子邮件会发生什么？
- 如果无法发送每月发票，那么结算电子邮件会发生什么情况？
- 如果您的团队成员登录到管理界面并手动更改，那么结算电子邮件会发生什么变化？
- 如果在我们提供给LinkedIn API的回调URL上收到POST请求，以便通知我们的应用用户在http://linkedin.com上更改了她的电子邮件地址，那么结算电子邮件会发生什么情况，因此新的`linkedInEmail`得获批吗？
- 当现有用户断开其LinkedIn账户时，账单邮件会发生什么变化？
- 数据库中的两个用户帐户是否允许使用相同的结算电子邮件？ LinkedIn的电子邮件怎么样？ 或者他们手动输入的那个？

根据这些问题的答案，我们最终可能会对`billingEmail`进行`required`验证，添加新的属性（`hasBillingEmailBeenChangedManually`），甚至改变是否使用`unique`约束。

### 最佳实践

最后，这里有一些提示:

+ 决定是否对某个属性使用验证，应该取决于应用程序的要求，以及您如何调用`.update（）`和`.create（）`。 不要害怕放弃内置验证支持，并在控制器或辅助功能中手动检查值。
+ 随着应用程序的发展，从模型中添加或删除验证没有任何问题。 但是一旦你投入生产，有一个**非常重要的例外**：`unique`。 在开发过程中，当您的应用配置为使用[`migrate：'alter'`](https://sailsjs.com/documentation/concepts/models-and-orm/model-settings#?migrate)时，您可以添加或 随意删除`unique`验证。 但是，如果您正在使用`migrate：safe`（例如使用产品数据库），则需要更新数据库中的约束/索引，以及[手动迁移数据](https://github.com/BlueHotDog/sails-migrations)。

> 尽可能多地充实自己的应用程序前端设计，在您花费大量时间实现_any_后端代码之前。 当然，这并不总是可行的 - 这就是[blueprint API](https://sailsjs.com/documentation/concepts/blueprints)的用途。 以UI为中心或“前端第一”理念构建的应用程序更容易维护，缺陷更少，而且由于它们是在完全了解用户界面的基础上构建而成的，因此它们通常会更加优雅。



### 自定义验证规则

您可以通过在属性中指定一个“自定义”函数来定义自己的自定义验证规则。

```javascript
// api/models/User.js
module.exports = {

  // Values passed for creates or updates of the User model must obey the following rules:
  attributes: {

    firstName: {
      // Note that a base type (in this case "string") still has to be defined, even though validation rules are in use.
      type: 'string',
      required: true,
      minLength: 5,
      maxLength: 15
    },

    location: {
      type: 'json',
      custom: function(value) {
        return _.isObject(value) &&
        _.isNumber(value.x) && _.isNumber(value.y) &&
        value.x !== Infinity && value.x !== -Infinity &&
        value.y !== Infinity && value.y !== -Infinity;
      }
    },

    password: {
      type: 'string',
      custom: function(value) {
        // • be a string
        // • be at least 6 characters long
        // • contain at least one number
        // • contain at least one letter
        return _.isString(value) && value.length >= 6 && value.match(/[a-z]/i) && value.match(/[0-9]/);
      }
    }

  }

}
```

自定义验证函数接收被验证的传入值作为它们的第一个参数，并且如果有效则返回“true”，否则返回“false”。



##### 自定义验证消息

Sails.js不支持自定义验证消息。相反，你的代码应该在你的`create()`或`update()`的回调中查看验证错误并采取适当的操作; 无论是在您的JSON响应中发送特定的错误代码，还是在HTML错误页面中呈现。


<docmeta name="displayName" value="Validations">
