# 模型设置

在Sails中，模型定义的顶级属性称为**模型设置**。 这包括模型将使用的[属性定义](https://sailsjs.com/documentation/concepts/models-and-orm/model-settings#?attributes)到[数据库设置](https://sailsjs.com/documentation/concepts/models-and-orm/model-settings#?datastore)，以及一些其他选项。

本页面的大部分内容专门介绍Sails支持的模型设置。但在开始之前，让我们看看如何在Sails中实际应用这些设置。


### 概述

模型设置允许您在Sails中自定义模型的行为。可以通过在[模型定义](https://sailsjs.com/documentation/concepts/models-and-orm/models)中设置顶级属性，或者默认值在[`sails.config.models`](https://sailsjs.com/documentation/reference/configuration/sails-config-models)中。

##### 更改默认模型设置

要修改应用中所有模型共享的[默认模型设置](https://sailsjs.com/documentation/reference/configuration/sails-config-models)，请编辑[`config/models.js`](https://sailsjs.com/documentation/anatomy/my-app/config/models-js)。

例如，当您生成一个新的应用程序时，Sails会在`config/models.js`文件中自动包含三个不同的默认属性：`id`，`createdAt`和`updatedAt`。 比方说，对于所有的模型，你都想使用一个稍微不同的自定义`id`属性。 要做到这一点，你可以在`config / models.js`定义中覆盖`attributes：{id：{...}}`。


##### 覆盖特定模型的设置

要进一步为特定模型自定义这些设置，可以指定为该模型定义文件中的顶级属性（例如，`api/models/ User.js`）。 这将覆盖具有相同名称的默认模型设置。

例如，如果您为某个模型定义（`api/models/UploadedFile.js`）添加`fetchRecordsOnUpdate：true`，那么该模型现在将返回已更新的记录。 但其他模型将不受影响; 他们仍然会使用默认设置（除非你已经改变了`fetchRecordsOnUpdate：false`）。


##### 选择一种方法

在日常开发过程中，您最常与之交互的模型设置是“属性”。 几乎每个模型定义中都使用了属性，它的一些默认属性包含在`config/models.js`中。 为了将来参考，这里有一些额外的提示:

+ 如果你指定一个`tableName`，你应该总是以模型为基础。（应用程序的表名是没有用的！）
+ 别去指定应用程序范围的datastore，因为您已经有一个开箱即用的（名为“default”）。 但你可能想要覆盖特定模型的`datastore`; 例如，如果您的默认数据存储是PostgreSQL，但是您有一个想要在Redis中存在的`CachedBloodworkReport`模型。
+ 为了清楚起见，最好仅将`migrate`和`schema`设置指定为应用程序范围的默认值; 从不以每个模型为基础。


现在你已经知道一般的模型设置，以及如何配置它们，让我们来看看。

--------------------




### 属性

模型的一组属性定义。

```
attributes: { /* ... */ }
```

| Type           | Example                 | Default       |
| -------------- |:------------------------|:--------------|
| ((dictionary)) | _See below._            | `{}`          |

大多数情况下，您将在各个模型定义中定义属性（`api/models/`）。但是你也可以在`config/models.js`中指定**默认属性**。 这使您可以在一个地方定义一组全局属性，然后依靠Sails使其隐含在所有模型中，而无需重复。在相关模型定义中定义同名属性，也可以基于每个模型覆盖默认属性。


```js
attributes: {
  id: { type: 'number', autoIncrement: true },
  createdAt: { type: 'number', autoCreatedAt: true },
  updatedAt: { type: 'number', autoUpdatedAt: true },
}
```

有关模型属性的完整介绍（包括如何在Sails应用程序中定义和使用它们），请参阅[Concepts > ORM > Attributes](https://sailsjs.com/documentation/concepts/orm/attributes).

### customToJSON

使用一个函数，将自定义模型记录序列化为JSON的方式。

```
customToJSON: function() { /*...*/ }
```

| Type         | Example                 | Default       |
| ------------ |:------------------------|:--------------|
| ((function)) | _See below._            | _n/a_         |

将`customToJSON`设置添加到模型会改变模型记录的方式。换句话说，它允许您插入自定义逻辑，这些逻辑将这些记录传递到`JSON.stringify()`中。这用于实现故障安全，确保敏感数据（如用户密码）不会意外地包含在响应中(使 [`res.send()`](https://sailsjs.com/documentation/reference/response-res/res-send)和actions2可以在发送之前使数据串联）。

customToJSON函数没有参数，但提供了对`this`变量的访问。这允许您忽略敏感数据并返回处理结果，这是JSON.stringify()在生成JSON字符串时实际使用的结果。例如:

```js
customToJSON: function() {
  // Return a shallow copy of this record with the password and ssn removed.
  return _.omit(this, ['password', 'ssn'])
}
```

> 请注意，`customToJSON`中提供的`this`变量是对实际记录object_的direct引用，因此请不要修改它。 换句话说，避免编写像`delete this.password`这样的代码。可以使用`_.omit（）`或`_.pick（）`等方法来获取记录的_copy_。 或者建立一个新的字典并返回（例如`return {foo：this.foo}`）。

### 表名tableName

SQL表或MongoDB集合的名称，模型以行或MongoDB文档的形式存储和检索其记录。

```
tableName: 'some_preexisting_table'
```

| Type        | Example                    | Default       |
| ----------- |:---------------------------|:--------------|
| ((string))  | `'some_preexisting_table'` | _Same as model's identity._

默认情况下，这与模型的[身份](https://sailsjs.com/documentation/concepts/models-and-orm/model-settings?identity)相同。 但是，如果使用共享/旧数据库集成在，以这种方式自定义`tableName`的功能可以为您节省大量时间。

**tableName**设置能够自定义特定模型所使用的基础物理模型的名称。换句话说，它允许您控制模型在数据库中存储和检索记录的位置，不会影响控制器contorl/helper程序中的代码。

> 但是，名字是什么？在MySQL和PostgreSQL这样的数据库中，这个设置引用了一个“表”。 在MongoDB和Redis等其他数据库中，它指的是一个“集合”。

默认情况下，当没有指定tableName时，Waterline使用模型的 [identity]（＃？身份）（例如“user”）:

```js
await User.find();
// => SELECT * FROM user;
```

这是推荐的惯例，在大多数情况下不需要改变。但是，如果您要与现有的PHP或Java应用程序共享数据库，或者如果您想遵守不同的命名约定，那么自定义此映射可能会很有用。 回到上面的例子，如果你在`api/models/User.js`中修改了模型定义，并设置了`tableName'foo_bar'`，那么你会看到稍微不同的结果:

```js
await User.find();
// => SELECT * FROM foo_bar;
```


### 迁移

Sails将在您每次加载应用时运行**自动迁移策略**。

```
migrate: 'alter'
```

| Type        | Example                 | Default       |
| ----------- |:------------------------|:--------------|
| ((string))  | `'alter'`               | _You'll be prompted._<br/><br/>_**Note**: In production, this is always `'safe'`._

“迁移”设置控制您应用的自动迁移策略。这告诉Sails你是否希望在数据库中自动重建tables/collections/sets等。

##### 数据库迁移

在开发应用程序的过程中，您几乎总是需要对数据库的结构进行至少一次或两次**重大更改**。 确切地说_什么构成“breaking change”取决于您使用的数据库：例如，假设您为其中一个模型定义添加了新属性。 如果该模型配置为使用MongoDB，那么这没什么大不了的。 但是，如果该模型被配置为使用MySQL，那么还有一个额外的步骤：必须将一列添加到相应的表（否则模型方法，如`.create（）`将停止工作。）因此，对于使用MySQL的模型，添加属性是一个重大改变。

> 即使您的所有模型都使用MongoDB，仍然有一些需要注意。例如，如果您为其中一个属性添加“unique：true”，则必须在MongoDB中创建[unique index](https://docs.mongodb.com/manual/core/index-unique/)。


在Sails中，当涉及[数据库迁移]时，有两种不同的[操作模式](https://en.wikipedia.org/wiki/Schema_migration):

1. **手动迁移** - The art of updating your database tables/collections/sets/etc. by hand.  For example, writing a SQL query to [add a new column](http://dev.mysql.com/doc/refman/5.7/en/alter-table.html), or sending a [Mongo command to create a unique index](https://docs.mongodb.com/manual/core/index-unique/).  If the database contains data you care about (in production, for example), you must carefully consider whether that data needs to change to fit the new schema, and, if necessary, write scripts to migrate it.  A [number of](https://www.npmjs.com/package/sails-migrations) great [open-source tools](http://knexjs.org/#Migrations-CLI) exist for managing manual migration scripts, as well as hosted products like the [database migration service on AWS](https://aws.amazon.com/blogs/aws/aws-database-migration-service/).
2. **自动迁移** - A convenient, built-in feature in Sails that allows you to make iterative changes to your model definitions during development, without worrying about the reprecussions.  Auto-migrations should _never_ be enabled when connecting to a database with data you care about.  Instead, use auto-migrations with fake data, or with cached data that you can easily recreate.


无论何时您需要对您的_产品数据库_应用重大更改，都应该使用手动数据库迁移。但除此之外，当您在笔记本电脑上开发或运行自动化测试时，自动迁移可为您节省大量时间。


##### 自动迁移如何工作

When you lift your Sails app in a development environment (e.g. running `sails lift` in a brand new Sails app), the configured auto-migration strategy will run.  If you are using `migrate: 'safe'`, then nothing extra will happen at all.  But if you are using `drop` or `alter`, Sails will load every record in your development database into memory, then drop and recreate the physical layer representation of the data (i.e. tables/collections/sets/etc.)  This allows any breaking changes you've made in your model definitions, like removing a uniqueness constraint, to be automatically applied to your development database.  Finally, if you are using `alter`, Sails will then attempt to re-seed the freshly generated tables/collections/sets with the records it saved earlier.


| Auto-migration strategy  | Description |
|:-------------------------|:---------------------------------------------|
| `safe`                    | never auto-migrate my database(s). I will do it myself, by hand.
| `alter`                   | auto-migrate columns/fields, but attempt to keep my existing data (experimental)
| `drop`                    | wipe/drop ALL my data and rebuild models every time I lift Sails


> Keep in mind that when using the `alter` or `drop` strategies, any manual changes you have made to your database since the last time you lifted your app may be lost.  This includes things like custom indexes, foreign key constraints, column order and comments.  In general, tables created by auto-migrations are not guaranteed to be consistent regarding any details of your physical database columns besides setting the column name, type (including character set / encoding if specified) and uniqueness.

##### 我可以在产品中使用自动迁移吗？

The `drop` and `alter` auto-migration strategies in Sails exist as a feature for your convenience during development, and when running automated tests.  **They are not designed to be used with data you care about.**  Please take care to never use `drop` or `alter` with a production dataset.  In fact, as a failsafe to help protect you from doing this inadvertently, any time you lift your app [in a production environment](https://sailsjs.com/documentation/reference/configuration/sails-config#?sailsconfigenvironment), Sails _always_ uses `migrate: 'safe'`, no matter what you have configured.

In many cases, hosting providers automatically set the `NODE_ENV` environment variable to "production" when they detect a Node.js app.  Even so, please don't rely only on that failsafe, and take the usual precautions to keep your users' data safe.  Any time you connect Sails (or any other tool or framework) to a database with pre-existing production data, **do a dry run**.  Especially the very first time.  Production data is sensitive, valuable, and in many cases irreplaceable.  Customers, users, and their lawyers are not cool with it getting flushed.

As a best practice, make sure to never lift or [deploy](https://sailsjs.com/documentation/concepts/deployment) your app with production database credentials unless you are 100% sure you are running in a production environment.  A popular approach for solving this organization-wide is simply to _never_ push up production database credentials to your source code repository in the first place, and instead relying on [environment variables](https://sailsjs.com/documentation/reference/configuration) for all sensitive credentials.  (This is an especially good idea if your app is subject to regulatory requirements, or if a large number of people have access to your code base.)


##### Are auto-migrations slow?

If you are working with a relatively large amount of development/test data, the `alter` auto-migration strategy may take a long time to complete at startup.  If you notice that a command like `npm test`, `sails console`, or `sails lift` appears to hang, consider decreasing the size of your development dataset.  (Remember: Sails auto-migrations should only be used on your local laptop/desktop computer, and only with small, development datasets.)



### 关系型

模型是否期望记录符合特定的一组属性。

```
schema: true
```

| Type        | Example                 | Default       |
| ----------- |:------------------------|:--------------|
| ((boolean)) | `true`                  | _Depends on the adapter._


`schema`设置允许您在“非关系型”或“关系型”模式之间切换模型。 更具体地说，它支配像`.create（）`和`.update（）`这样的方法的行为。 通常情况下，只要您使用的适配器支持它，就可以将任意数据存储在记录中。 但是，如果启用`schema：true`，则只会存储与模型的“属性”相对应的属性。

> 此设置仅适用于使用非关系型数据库（如MongoDB）的模型。当连接到像MySQL或PostgreSQL这样的关系型数据库时，模型总是有效的`schema：true`（因为底层数据库只能将数据存储在提前设置的表和列中）。



### 数据存储

模型将用于查找记录，创建记录等的[数据存储配置](https://sailsjs.com/documentation/reference/sails-config/sails-config-datastores)的名称等等。


```
datastore: 'legacyECommerceDb'
```

| Type       | Example                 | Default       |
| ---------- |:------------------------|:--------------|
| ((string)) | `'legacyECommerceDb'`   | `'default'`   |

这表明该模型将获取并保存其数据的数据库。 除非另有说明，否则应用中的每个模型都会使用名为“default”的内置数据存储，该数据存储包含在每个新的Sails应用程序中。 这样可以轻松配置应用程序的主数据库，同时仍然允许您覆盖任何特定模型的`datastore`设置。

For more about configuring your app's datastores, see [Reference > Configuration > Datastores](https://sailsjs.com/documentation/reference/sails-config/sails-config-datastores).


### 数据加密密钥

解密数据时使用的一组密钥。 除非另行配置，否则“默认”密钥（或“DEK”）始终用于加密。


```javascript
dataEncryptionKeys: {
  default: 'tVdQbq2JptoPp4oXGT94kKqF72iV0VKY/cnp7SjL7Ik='
}
```

> 除非你需要流动秘钥，否则`default`键就是你所需要的。 除“default”以外的任何其他数据加密密钥都可以解密用它们加密的旧数据。

##### Key rotation

如果让数据加密密钥荣归故里，您需要给它一个新的密钥ID（如`2028`），然后创建一个新的“deault”密钥用于任何新的加密。 例如，如果您在2028年发布Sails应用程序，并且您的密钥每年都轮换出来，那么在下一年您的`dataEncryptionKeys`可能看起来像这样:

```javascript
dataEncryptionKeys: {
  default: 'DZ7MslaooGub3pS/0O734yeyPTAeZtd0Lrgeswwlt0s=',
  '2028': 'C5QAkA46HD9pK0m7293V2CzEVlJeSUXgwmxBAQVj+xU='
}
```

在2030年1月之后更改默认密钥后，您可能会有:

```javascript
dataEncryptionKeys: {
  default: 'tVdQbq2JptoPp4oXGT94kKqF72iV0VKY/cnp7SjL7Ik=',
  '2029': 'DZ7MslaooGub3pS/0O734yeyPTAeZtd0Lrgeswwlt0s=',
  '2028': 'C5QAkA46HD9pK0m7293V2CzEVlJeSUXgwmxBAQVj+xU='
}
```


### 连续销毁

Whether or not to _always_ act like you set `cascade: true` any time you call `.destroy()` using this model.

```
cascadeOnDestroy: true
```

| Type        | Example                 | Default       |
| ----------- |:------------------------|:--------------|
| ((boolean)) | `true`                  | `false`

出于性能原因，这是默认禁用的。 您可以使用此模型设置启用它，或者使用[`.meta（{cascade：true}）`](https://sailsjs.com/documentation/reference/waterline-orm/queries/meta)在每个查询的基础上启用它.



### 不使用ObjectIds

> ##### _ **此功能只用于[`sails-mongo`适配器](https://sailsjs.com/documentation/concepts/extending-sails/adapters/available-adapters#?sailsmongo)**_

如果设置为“true”，模型将不使用自动生成的MongoDB ObjectID对象作为其主键。 这允许你使用`sails-mongo`适配器创建模型，其中主键是任意字符串或数字，而不仅仅是很长的UUID。请注意，将此设置为“true”意味着您必须在每次调用[`.create（）`]或[`.createEach（）`]时提供`id`的值或。


| Type        | Example                 | Default       |
| ----------- |:------------------------|:--------------|
| ((boolean)) | `true`                  | `false`

出于性能原因，这是默认禁用的。 您可以使用此模型设置启用它，或者使用[`.meta({cascade: true})`](https://sailsjs.com/documentation/reference/waterline-orm/queries/meta).



### 很少使用的设置

以下低级设置它们很少（如果曾经）被改变。


##### 主键

模型主键属性的名称。

> **你不需要改变这个设置，在“id”属性上设置了一个自定义的`columnName`。**

```javascript
primaryKey: 'id'
```

| Type       | Example       | Default       |
| ---------- |:--------------|:--------------|
| ((string)) | `'id'`        | `'id'`        |

通常，这是“id”，这是一个默认属性，自动包含在由Sails v1.0生成的新应用程序的config/models.js文件中。改变模型主键的最好方法就是自定义该默认属性的`columnName`。

例如，假设您有一个用户模型需要与预先存在的MySQL数据库中的表集成。 该表可能会有一个名称不是“id”（如“email_address”）的列作为其主键。 为了让你的模型尊重这个主键，你需要在模型定义中为`id`属性指定一个override:

```js
id: {
  type: 'string',
  columnName: 'email_address',
  required: true
}
```

然后，在您应用的代码中，您将能够通过主键查找用户，并生成的SQL查询:

```js
await User.find({ id: req.param('emailAddress' });
```

> MongoDB忠实粉丝，在您新的Sails应用程序中，您将在`config/models.js`的默认“id”属性中设置`columnName：'_id'`开始。 然后你可以像平常一样使用Sails和Waterline，一切都可以正常工作。
>
> 但是如果你发现自己希望你可以改变“id”属性本身的名称。例如，当您在代码中调用内置模型方法时，而不是通常的“id”时，可以使用`.destroy（{_id：'ba8319abd-13810-ab31815'}）`的语法。
>
> 这就是该模型设置可能出现的位置。您只需编辑`config/models.js`，以便它包含`primaryKey：'_id'`，然后将默认的“id”属性重命名为“_id”。  But there are some [good reasons to reconsider](https://gist.github.com/mikermcneil/9247a420488d86f09be342038e114a08).

##### 模型identity

小写的每个模型唯一的标识符。

>  **模型的“identity”是只读的。 它是自动派生的，不应该由手工设置。**

```
Something.identity;
```

| Type       | Example       |
| ---------- |:--------------|
| ((string)) | `'purchase'`  |

在Sails中，通过小写文件名来自动推断模型的“identity”。 例如，`api/models/Purchase.js`的标识就是`purchase`。 它可以作为`sails.models.purchase`访问，并且如果蓝图路由已启用，则可以通过GET/purchase和PATCH/purchase/1等请求到达它。


```javascript
assert(Purchase.identity === 'purchase');
assert(sails.models.purchase.identity === 'purchase');
assert(Purchase === sails.models.purchase);
```



##### 全局IDglobalId

模型的唯一全局标识符，它也确定相应全局变量的名称。

> **模型的`globalId`是只读的。 它是自动派生的，不应该由手工设置。**

```
Something.globalId;
```

| Type       | Example       |
| ---------- |:--------------|
| ((string)) | `'Purchase'`  |

模型globalId的主要目的是确定Sails自动公开到全局变量的名称 - 也就是说，除非模型的全局化已被禁用(https://sailsjs.com/documentation/concepts/globals?q=disabling-globals)。在Sails中，模型的`globalId`是通过文件名自动推断的。 例如，`api/models/Purchase.js`的globalId就是`Purchase`。


```javascript
assert(Purchase.globalId === 'Purchase');
assert(sails.models.purchase.globalId === 'Purchase');
if (sails.config.globals.models) {
  assert(sails.models.purchase === Purchase);
}
else {
  assert(typeof Purchase === 'undefined');
}
```


<docmeta name="displayName" value="Model settings">
