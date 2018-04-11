# 属性
### 概述

模型属性是模型的基本信息。 例如，名为`Person`的模型可能具有名为`firstName`，`lastName`，`phoneNumber`，`age`，`birthDate`和`emailAddress`的属性。

<!---
FUTURE: address sql vs. no sql and stuff like:
"""
In most cases, this data is _homogenous_, meaning each record has the same attributes,
"""
-->



### 定义属性

模型的`attributes`[配置功能](https://sailsjs.com/documentation/concepts/models-and-orm/model-settings)允许您设置一组属性，每个属性定义为一个字典（aka plain JavaScript object）:

```javascript
// api/models/User.js
{
  attributes: {
    emailAddress: { type: 'string', required: true, },
    karma: { type: 'number', },
    isSubscribedToNewsletter: { type: 'boolean', defaultsTo: true, },
  },
}
```
在每个属性中，有一个或多个键、选项用于为Sails和Waterline作为提示。 这些属性键告诉模型如何去确保类型安全，执行高级验证规则，以及（如果您启用了automigrations），怎么样设置数据库中的表或集合。



##### 默认属性

您还可以定义将出现在模型中的所有的默认属性，将`attributes`定义为[默认模型设置](https://sailsjs.com/documentation/concepts/models-and-orm/model-settings)就可以了（例如`config/models.js`中）。新的Sails应用程序带有三个默认属性：`id`，`createdAt`和`updatedAt`。

这些属性将在所有模型中可用，除非它们被覆盖或禁用。 要覆盖默认属性，请在模型定义中定义一个具有相同名称的属性。 为了金融默认属性，将其定义为“false”。 例如，要为特定模型禁用默认`updatedAt`属性:

```javascript
// api/models/ProductCategory.js
module.exports = {
  attributes: {
    updatedAt: false,
    label: { type: 'string', required: true },
  }
}
```



### 类型安全


##### 类型

除[关联(associations)](https://sailsjs.com/documentation/concepts/models-and-orm/associations)外，每个属性都必须声明一个`type`。

这是可以在`type`中存储的数据类型 - 用于对查询及结果的逻辑类型进行安全检查。 以下是Sails和Waterline支持的数据类型列表:

- string
- number
- boolean
- json
- ref



##### 必填字段Required

如果一个属性是`required：true`，那么在调用`.create()`时必须指定一个值。 它防止显式创建（或更新）该值为'null'或空字符串（""），


##### 默认值

除了以上五种数据类型之外，还有属性定义的其他部分，包括默认值功能。

一个属性的默认值（`defaultsTo`）仅适用于`.create()`，当字段不输入时生效。


```javascript
attributes: {
  phoneNumber: {
    type: 'string',
    defaultsTo: '111-222-3333'
  }
}
```

##### 允许为空

当创建或更新记录时，`string`，`number`和`boolean`数据类型不接受`null`值。为了允许设置`null`值，您可以在属性设置切换`allowNull`标志。 `allowNull`标志只对这些数据类型有效。 对于类型为"json"或"ref"的属性，任何关联或主键设置都是无效的。

```javascript
attributes: {
  phoneNumber: {
    type: 'string',
    allowNull: true
  }
}
```

### 验证

除对基础类型安全检查外，Sails还提供了几种不同的高级验证规则。 例如，`isIn`规则，该字段内容必须包含在设置的数组之内:

```javascript
unsubscribeReason: {
  type: 'string',
  isIn: ['boring', 'too many emails', 'recipes too difficult', 'other'],
  required: true
}
```

有关高级验证规则的完整列表，请参阅 [Validations](https://sailsjs.com/documentation/concepts/models-and-orm/validations).



<!--

FUTURE: need ot move primary key out to the top-level (it's a model setting now)

commented-out content at: https://gist.github.com/rachaelshaw/f10d70c73780d5087d4c936cdefd5648#1
-->


### 列的名称

在属性定义中，您可以指定一个`columnName`来强制Sails/Waterline将该属性的数据存储在配置的数据存储（即数据库）中的特定列中。 请注意，这不是SQL专属 - 它也适用于MongoDB字段等等。

虽然`columnName`属性主要用于处理现有/旧数据库，但在您的数据库被其他应用程序共享或您没有访问权限来更改架构的情况下，它可能很有用。


要将数字模型`Wheels`属性存储到`number_of_round_rotating_things`列中或从`number_of_round_rotating_things`列中存储/读取
:
```javascript
  // An attribute in one of your models:
  // ...
  numberOfWheels: {
    type: 'number',
    columnName: 'number_of_round_rotating_things'
  }
  // ...
```


现在是一个更彻底的例子。

假设您的Sails应用中有一个`User`模型，看起来像这样:

```javascript
// api/models/User.js
module.exports = {
  datastore: 'shinyNewMySQLDatabase',
  attributes: {
    name: {
      type: 'string'
    },
    password: {
      type: 'string'
    },
    email: {
      type: 'string',
      unique: true
    }
  }
};
```

假如一切都很好，但不使用现有的MySQL数据库。服务器上坐落在某个地方，这个服务器恰好容纳了您的应用程序的用户:

```javascript
// config/datastores.js
module.exports = {
  // ...

  // Existing users are in here!
  rustyOldMySQLDatabase: {
    adapter: 'sails-mysql',
    url: 'mysql://ofh:Gh19R!?@db.eleven.sameness.foo/jonas'
  },
  // ...
};
```

假设在旧的MySQL数据库中有一个名为`our_users`的表，看起来像这样:

| the_primary_key | email_address | full_name | seriously_encrypted_password|
|------|---|----|---|
| 7 | mike@sameness.foo | Mike McNeil | ranchdressing |
| 14 | nick@sameness.foo | Nick Crumrine | thousandisland |


为了在Sails中使用它，你需要将`User`模型改为这个样子:

```javascript
// api/models/User.js
module.exports = {
  datastore: 'rustyOldMySQLDatabase',
  tableName: 'our_users',
  attributes: {
    id: {
      type: 'number',
      unique: true,
      columnName: 'the_primary_key'
    },
    name: {
      type: 'string',
      columnName: 'full_name'
    },
    password: {
      type: 'string',
      columnName: 'seriously_encrypted_password'
    },
    email: {
      type: 'string',
      unique: true,
      columnName: 'email_address'
    }
  }
};
```

> 您可能已经注意到，本例中我们也使用了[`tableName`](https://sailsjs.com/documentation/concepts/models-and-orm/model-settings#?tablename)属性。 这使我们能够控制用于容纳我们数据的表的名称。



### 加密Encryption at rest

##### 加密

是否自动加密此属性。 如果设置为“true”，那么当检索到记录时，它将显示此属性的加密值，除非使用[`.decrypt()`](https://sailsjs.com/documentation/reference/waterline-orm/queries/decrypt)。


```javascript
attributes: {
  ssn: {
    type: number,
    encrypt: true
  }
}
```






### 自动迁移

自动迁移设置，在应用程序发布时指导Sails，应如何为属性创建物理级别（例如PostgreSQL，MySQL或MongoDB）数据库字段。


> 当模型的migrate属性设置为safe时，这些设置将被忽略，数据库列将保持不变。

##### 列的类型

指示Sails，在创建数据库表时,属性对应的物理级别-列的数据类型。 这使您可以直接指定基础数据库如何创建它们的类型。 例如，您可能有一个属性将其`type`属性设置为`number`，并将其存储在要使用列类型`float`的数据库中。 你的属性定义看起来像:

```javascript
attributes: {
  placeInLine: {
    type: 'number',
    columnType: 'float'
  }
}
```

> * 列类型完全依赖于数据库。 确保您选择的`columnType`对应于您数据库有效的数据类型！ 如果您不指定`columnType`，则适配器会根据属性的`type`为您选择一个.
> * `columnType`值在创建数据库列的语句中执行，因此您可以使用它来指定其他选项，例如，`varchar（255）CHARACTER SET utf8mb4`.
> * 如果你打算在Sails模型中存储二进制数据，你需要将属性的`type`设置为`ref`，然后为你选择的数据库使用适当的`columnType`（例如`mediumblob` for MySQL或者 PostgreSQL的`bytea`）。 请记住，无论您尝试存储哪些内容，都必须先将其存入内存，然后才能将数据传输到数据库 - 目前在Sails中没有将二进制数据流式传输到数据存储适配器的机制。 作为在数据库中存储blob的替代方案，您可以考虑使用[`.upload()` 方法](https://sailsjs.com/documentation/concepts/file-uploads)将它们流式传输到磁盘或S3等远程文件系统。
> * 请记住，像MySQL中的`CHARACTER SET utf8mb4`这样的自定义列选项会影响列的存储大小。 这与`unique`属性结合使用时尤其重要，因为您必须指定列大小以避免错误。有关更多信息，请参见下面的[`unique`属性](https://sailsjs.com/documentation/concepts/models-and-orm/attributes#?unique)文档。


##### 自动递增

该属性设置为自动递增。将新记录添加到模型时，如果未指定此属性的值，则会通过将最新记录的值递增1来生成该记录。 注意：被指定`autoIncrement`的字段属性应该总是`type：'number`。 另外，请记住，这个支持因不同数据库而异。 例如，MySQL不允许每个表有多个自动递增列。

```javascript
attributes: {
  placeInLine: {
    type: 'number',
    autoIncrement: true
  }
}
```

##### 唯一值unique

适配器级约束，确保不允许两个记录具有相同的属性值。大多数情况下，这使基础数据存储中创建的属性具有唯一索引。

```javascript
attributes: {
  username: {
    type: 'string',
    unique: true
  }
}
```
> 当在MySQL数据库中使用`utf8mb4`字符集的属性上使用`unique：true`时，您需要通过[`columnType` property](https://sailsjs.com/documentation/concepts/models-and-orm/attributes#?columntype)来避免可能的'index too long'错误。 例如：`columnType：varchar（100）CHARACTER SET utf8mb4`。



<!--

commented-out content at: https://gist.github.com/rachaelshaw/f10d70c73780d5087d4c936cdefd5648#2


commented-out content at: https://gist.github.com/rachaelshaw/f10d70c73780d5087d4c936cdefd5648#3


FUTURE: move enum to validations page


commented-out content at: https://gist.github.com/rachaelshaw/f10d70c73780d5087d4c936cdefd5648#4


commented-out content at: https://gist.github.com/rachaelshaw/f10d70c73780d5087d4c936cdefd5648#5


-->







<docmeta name="displayName" value="Attributes">
