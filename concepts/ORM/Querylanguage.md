# Waterline查询语言

Sails的模型方法支持的语法称为Waterline查询语言。 Waterline知道如何解释这种语法，并从任何支持的数据库中检索或转译记录。Waterline使用安装在项目中的数据库适配器将该语言翻译为本地查询，然后将这些查询发送到适当的数据库。 这意味着您可以像使用Redis或MongoDb一样对MySQL使用相同的查询。 它允许你用最少的更改来更换你的数据库。



### 查询语言基础

标准对象使用四种类型的对象键之一形成。这些是最高级别的查询对象中使用的键。 它基于MongoDB中使用的标准进行简单的改动。
查询可以使用`where`键来指定属性，这将允许您也使用查询选项，例如`limit`和`skip`，或者如果不带有`where`，整个对象还会被视为标准`where`。


```javascript

var peopleNamedMary = await Model.find({
  name: 'mary'
});

// OR

var thirdPageOfRecentPeopleNamedMary = await Model.find({
  where: { name: 'mary' },
  skip: 20,
  limit: 10,
  sort: 'createdAt DESC'
});
```

#### Key对

一个Key对可用于搜索记录中与指定内容完全匹配的值。这是一个标准对象的基础，其中键表示模型上的属性，并且该值是匹配值的记录的严格相等检查。

```javascript
var peopleNamedLyra = await Model.find({
  name: 'lyra'
});
```

它们可以一起使用来搜索多个属性.

```javascript
var waltersFromNewMexico = await Model.find({
  name: 'walter',
  state: 'new mexico'
});
```

#### 复杂约束

复杂约束还具有键的模型属性，但它们使用条件修饰符来执行严格相等性检查。


```javascript
var peoplePossiblyNamedLyra = await Model.find({
  name : {
    'contains' : 'yra'
  }
})
```

#### 在调节器中

提供一个数组来查找，其属性值与指定搜索项_任何_完全匹配的记录。

> 这或多或少等同于SQL中的“IN”查询和MongoDB中的“$in”运算符。

```javascript
var waltersAndSkylers = await Model.find({
  name : ['walter', 'skyler']
});
```

#### 不在调节器中

使用`！`键（如`{'！'：[...]}`）提供包装在字典中的数组，以查找其属性_是 NOT_与任何指定搜索项完全匹配的记录。


> 这或多或少等同于SQL中的“NOT IN”查询和MongoDB中的“$nin”运算符。

```javascript
var everyoneExceptWaltersAndSkylers = await Model.find({
  name: { '!' : ['walter', 'skyler'] }
});
```

#### Or语法

使用`or`修饰符来作为查询。 对于匹配`or`查询的记录，它们必须至少匹配`or`数组中的一个。

```javascript
var waltersAndTeachers = await Model.find({
  or : [
    { name: 'walter' },
    { occupation: 'teacher' }
  ]
});
```

### 标准修饰符

构建查询时可使用以下修饰符。

* `'<'`
* `'<='`
* `'>'`
* `'>='`
* `'!='`
* `'nin'`
* `'in'`
* `'contains'`
* `'startsWith'`
* `'endsWith'`

> 请注意，与[JSON属性](https://sailsjs.com/documentation/concepts/models-and-orm/validations#?builtin-data-types)的属性进行匹配时，条件修饰符的可用性和行为可能会有所不同。 例如，虽然`sails-postgresql`会将您的JSON属性映射到JSON 列类型，您需要[发送本地查询](https://sailsjs.com/documentation/reference/waterline-orm/datastores/send-native-query)以便直接查询这些属性。 另一方面，`sails-mongo`支持针对JSON类型属性的查询，但是您应该知道，如果一个字段包含一个数组，则将针对数组中的每个_item_运行查询条件，而不是数组本身（ 这是基于MongoDB本身的行为）。

#### '<'

搜索值小于指定值的记录。

```usage
Model.find({ age: { '<': 30 }})
```

#### '<='

搜索值小于等于指定值的记录。

```usage
Model.find({ age: { '<=': 20 }})
```

#### '>'

搜索值大于指定值的记录。

```usage
Model.find({ age: { '>': 18 }})
```

#### '>='

搜索值大于等于指定值的记录。

```usage
Model.find({ age: { '>=': 21 }})
```

#### '!='

搜索值不等于指定值的记录。

```usage
Model.find({
  name: { '!=': 'foo' }
})
```

#### in

搜索包含记录。

```usage
Model.find({
  name: { in: ['foo', 'bar'] }
})
```

#### nin

搜索不包含记录。

```usage
Model.find({
  name: { nin: ['foo', 'bar'] }
})
```

#### contains

搜索此属性的值包含给定字符串的记录。

```usage
var musicCourses = await Course.find({
  subject: { contains: 'music' }
});
```

_For performance reasons, case-sensitivity of `contains` depends on the database adapter._

#### startsWith

搜索此属性的值_开始位置_给定字符串的记录。

```usage
var coursesAboutAmerica = await Course.find({
  subject: { startsWith: 'american' }
});
```

_For performance reasons, case-sensitivity of `startsWith` depends on the database adapter._

#### endsWith

搜索此属性的值_结束位置_给定字符串的记录。

```usage
var historyCourses = await Course.find({
  subject: { endsWith: 'history' }
});
```

_For performance reasons, case-sensitivity of `endsWith` depends on the database adapter._


### 查询选项

查询选项允许您优化查询返回的结果。 目前的选项
可用的:

* `limit`
* `skip`
* `sort`

#### Limit

限制查询返回的结果数量。

```usage
Model.find({ where: { name: 'foo' }, limit: 20 })
```

> Note: if you set `limit` to 0, the query will always return an empty array.

#### Skip

返回除跳过项目数量外的所有结果。

```usage
Model.find({ where: { name: 'foo' }, skip: 10 });
```

##### Pagination

可以同时使用“skip”和“limit”来构建分页系统。

```usage
Model.find({ where: { name: 'foo' }, limit: 10, skip: 10 });
```

> **Waterline**
>
> 您可以在下面找到更多关于Waterline API的信息:
> * [Sails.js Documentation](https://sailsjs.com/documentation/reference/waterline-orm/queries)
> * [Waterline README](https://github.com/balderdashy/waterline/blob/master/README.md)
> * [Waterline Reference Docs](https://sailsjs.com/documentation/reference/waterline-orm)
> * [Waterline Github Repository](https://github.com/balderdashy/waterline)


#### 排序

结果可以按属性名称排序。 只需指定自然属性名称（升序）排序或分别为升序或降序指定“ASC”或“DESC”标志。

```usage
// Sort by name in ascending order
Model.find({ where: { name: 'foo' }, sort: 'name' });

// Sort by name in descending order
Model.find({ where: { name: 'foo' }, sort: 'name DESC' });

// Sort by name in ascending order
Model.find({ where: { name: 'foo' }, sort: 'name ASC' });

// Sort by object notation
Model.find({ where: { name: 'foo' }, sort: [{ 'name': 'ASC' }});

// Sort by multiple attributes
Model.find({ where: { name: 'foo' }, sort: [{ name:  'ASC'}, { age: 'DESC' }]);
```


<docmeta name="displayName" value="Query language">
