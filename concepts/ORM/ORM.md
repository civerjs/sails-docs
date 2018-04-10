# Waterline: SQL/noSQL数据映射器 (ORM/ODM)


Sails安装了强大的[ORM / ODM](http://stackoverflow.com/questions/12261866/what-is-the-difference-between-an-orm-and-an-odm) 名字叫做[Waterline](https://github.com/balderdashy/waterline), 一个数据存储不可知论的工具，可以显着简化与一个或多个[数据库](http://www.cs.umb.edu/cs630/hd1.pdf). 它在底层数据库之上提供抽象层，使您能够轻松查询和操作数据，而无需编写特定依赖的集成代码。

### 数据库不可知论

在关系化数据库像[Postgres](http://www.postgresql.org/), [Oracle](https://www.oracle.com/database), 和 [MySQL](http://www.mysql.com),模型代表表格. 在[MongoDB](http://www.mongodb.org)中, 他们代表Mongo的"collections". 在[Redis](http://redis.io)中, 它们使用键/值对来表示. 每个数据库都有自己独特的查询方言，在某些情况下，甚至需要安装和编译特定的本地模块才能连接到服务器。这涉及相当多的开销，并且将[依赖锁定]（https://en.wikipedia.org/wiki/Vendor_lock-in）到特定数据库; 例如如果你的应用使用了一堆SQL查询，那么以后很难切换到Mongo，或Redis，反之亦然。

Waterline查询语法最重要的是着重于创建新记录，获取/搜索现有记录，更新记录或销毁记录等业务逻辑。无论您与哪个数据库联系，使用情况都完全相同。 此外，Waterline允许您在模型之间[`.populate（）`](https://sailsjs.com/documentation/reference/waterline-orm/queries/populate)关联，即使每个模型的数据存在于不同数据库。这意味着您可以将应用程序的模型从Mongo切换到Postgres，切换到MySQL，再切换到Redis，而无需更改任何代码。需要low-level数据库功能的时候，Waterline提供了一个查询界面，可让您直接与模型的底层数据库驱动程序通讯(请参阅[.query()](https://sailsjs.com/documentation/reference/waterline-orm/models/query)和 [.native()](https://sailsjs.com/documentation/reference/waterline-orm/models/native).)


### 脚本

假设您正在建立一个电子商务网站，并附带一个移动应用程序。 用户按类别浏览产品或按关键字搜索产品，然后购买它们。如此，你的应用程序的某些部分很普通; 您有用于登录，注册，订单/付款处理，重置密码等的API驱动流程。但是，您知道潜藏在您的路线图中的一些常见功能可能会涉及更多。 果然:

##### 灵活性

_你问企业他们想使用什么数据库:_

> “数据 ......什么？我们不要急，不想做出错误的选择，我先咨询一下IT部门，然后继续。”

为Web应用程序/ API选择单一数据库的传统方法对于许多生产用例来说实际上是禁止的。 通常，应用程序需要与一个或多个现有数据集保持兼容，或者出于性能原因需要使用几种不同类型的数据库。

由于Sails默认使用`sails-disk`，所以您可以使用本地临时文件存储来构建零配置的应用程序。当你准备好切换到真实的东西时（当每个人都知道甚至是什么时候），只需改变你的应用程序的[datastore configuration](https://sailsjs.com/documentation/reference/configuration/sails-config-datastores).


##### 兼容性

_产品所有者/利益相关者走向你并说:_

> “哦，嘿，顺便说一句，这些产品实际上已经在我们的销售系统中存在了，可能是在ERP中，比如”DB2“，不管怎样，我相信你会发现它 - 听起来很简单吧？

许多企业应用程序必须与现有数据库集成。 如果幸运的话，来一次性数据迁移就可以了。但更常见的是，现有数据集仍在被其他应用程序修改。 为了构建您的应用程序，您可能需要结合来自多个旧系统的数据，或与其他地方存储的单独数据集结合使用。 这些数据集可以分布在世界各地的5个不同的服务器上！ 一个共享数据库服务器可能包含关系数据的SQL数据库，而另一个云服务器可能包含少数Mongo或Redis集合。

Sails/Waterline可让您将不同的模型连接到不同的数据存储区; 本地或互联网上的任何地方。 您可以构建映射到旧数据库中的自定义MySQL表的用户模型（它使用奇怪的疯狂列名称）。 同样的事情还有，映射到DB2中的表的产品模型或映射到MongoDB集合的Order模型。 最重要的是，你可以在这些不同的数据存储区和适配器上使用`.populate()`，所以如果你配置一个模型来存放在不同的数据库中，你的控制器/模型代码不需要改变(注意，你需要手动迁移重要的产品数据）


##### 性能

_你在深夜坐在你的笔记本电脑前，你意识到:_
> “我怎样才能做关键字搜索？产品数据没有任何关键字，企业希望根据n-gram单词序列排列搜索结果，我也不知道这个推荐引擎是如何工作的。 今晚我再听到“大数据”这句话，我正要离开，然后回到咖啡店工作。“

那么“大数据”呢？ 通常当你听到博客和分析师使用这个流行语时，你会想到数据挖掘和商业智能。 您可以想象一个从多个来源获取数据，处理/索引/分析数据的流程，然后将提取的信息写入其他位置，并保留原始数据或将其丢弃。

但是，还有一些比较常见的挑战，诸如“驾驶方向-亲密度”搜索的特征，或相关产品的推荐引擎。 幸运的是，许多数据库简化了特定的大数据使用情况（例如MongoDB提供了地理空间索引，ElasticSearch为全文搜索索引数据提供了极好的支持）。


### 适配器

Like most MVC frameworks, Sails supports [multiple databases](https://sailsjs.com/features).  That means the syntax to query and manipulate our data is always the same, whether we're using MongoDB, MySQL, or any other supported database.

Waterline builds on this flexibility with its concept of adapters.  An adapter is a bit of code that maps methods like `find()` and `create()` to a lower-level syntax like `SELECT * FROM` and `INSERT INTO`.  The Sails core team maintains open-source adapters for a handful of the [most popular databases](https://sailsjs.com/features), and a wealth of [community adapters](https://github.com/balderdashy/sails-docs/blob/0.9/Database-Support.md) are also available.

Custom Waterline adapters are actually [pretty simple to build](https://github.com/balderdashy/sails-generate-adapter), and can make for more maintainable integrations; anything from a proprietary enterprise account system, to a cache, to a traditional database.


### 数据存储

A **datastore** represents a particular database configuration.  This configuration object includes an adapter to use, as well as information like the host, port, username, password, and so forth.  Datastores are defined in the Sails config [`config/datastores.js`](https://sailsjs.com/documentation/reference/configuration/sails-config-datastores).

```javascript
// in config/datastores.js
// ...
{
  adapter: 'sails-mysql',
  host: 'localhost',
  port: 3306,
  user: 'root',
  password: 'g3tInCr4zee&stUfF'
}
// ...
```


### 比喻Analogy

Imagine a file cabinet full of completed pen-and-ink forms. All of the forms have the same fields (e.g. "name", "birthdate", "maritalStatus"), but for each form, the _values_ written in the fields vary.  For example, one form might contain "Lara", "2000-03-16T21:16:15.127Z", "single", while another form contains "Larry", "1974-01-16T21:16:15.127Z", "married".

Now imagine you're running a hotdog business.  If you were _very_ organized, you might set up your file cabinets as follows:

+ **Employee** (contains your employee records)
  + `fullName`
  + `hourlyWage`
  + `phoneNumber`
+ **Location** (contains a record for each location you operate)
  + `streetAddress`
  + `city`
  + `state`
  + `zipcode`
  + `purchases`
    + a list of all the purchases made at this location
  + `manager`
    + the employee who manages this location
+ **Purchase** (contains a record for each purchase made by one of your customers)
  + `madeAtLocation`
  + `productsPurchased`
  + `createdAt`
+ **Product** (contains a record for each of your various product offerings)
  + `nameOnMenu`
  + `price`
  + `numCalories`
  + `percentRealMeat`
  + `availableAt`
    + a list of the locations where this product offering is available.


In your Sails app, a **model** is like one of the file cabinets.  It contains **records**, which are like the forms.  `Attributes` are like the fields in each form.



### 注意
+ This documentation on models is not applicable if you are overriding the built-in ORM, [Waterline](https://github.com/balderdashy/waterline).  In that case, your models will follow whatever convention you set up, on top of whatever ORM library you're using (e.g. Mongoose.)





<docmeta name="displayName" value="Models and ORM">
<docmeta name="nextUpLink" value="/documentation/concepts/models-and-orm/models">
<docmeta name="nextUpName" value="Models">
