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

像大多数MVC框架一样，Sails支持[多个数据库](https://sailsjs.com/features). 这意味着无论我们使用的是MongoDB，MySQL还是其他支持的数据库，查询和操作数据的语法都是相同的。

Waterline构建在灵活的适配器之上。 适配器是将将`find()`和`create()`方法映射到`SELECT * FROM`和`INSERT INTO`等更低级语法的代码。 Sails核心团队为部分[流行数据库](https://sailsjs.com/features)和大量[community adapters](https://github.com/balderdashy/sails-docs/blob/0.9/Database-Support.md)维护开源适配器。

自定义Waterline适配器实际上[相当简单](https://github.com/balderdashy/sails-generate-adapter)，并且更容易维护和集成; 从专有企业系统到高速缓存，再到传统数据库。


### 数据存储

**数据存储**表示特定的数据库配置。 此配置对象包含要使用的适配器以及主机，端口，用户名，密码等信息。 数据存储在Sails配置[`config / datastores.js`](https://sailsjs.com/documentation/reference/configuration/sails-config-datastores)中定义.

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

想象一个文件柜里装满各种表单形式。 所有表格都具有相同的字段（例如“姓名”，“出生日期”，“婚姻状态”），但是对于每种表格，字段中写入的_values_都不相同。 例如，一个表单可能包含“Lara”，“2000-03-16T21：16：15.127Z”，“single”，而另一个表单包含“Larry”，“1974-01-16T21：16：15.127Z”，“ 已婚”。

现在想象你正在经营热狗业务。 如果你善于管理，你可以按如下步骤设置你的文件柜:

+ **雇员** (contains your employee records)
  + `fullName`
  + `hourlyWage`
  + `phoneNumber`
+ **位置** (contains a record for each location you operate)
  + `streetAddress`
  + `city`
  + `state`
  + `zipcode`
  + `purchases`
    + a list of all the purchases made at this location
  + `manager`
    + the employee who manages this location
+ **采购** (contains a record for each purchase made by one of your customers)
  + `madeAtLocation`
  + `productsPurchased`
  + `createdAt`
+ **产品** (contains a record for each of your various product offerings)
  + `nameOnMenu`
  + `price`
  + `numCalories`
  + `percentRealMeat`
  + `availableAt`
    + a list of the locations where this product offering is available.

在您的Sails应用程序中，**模型**就像其中一个文件柜。 它包含**records**，它们就像表格一样。 `Attributes`就像每个表单中的字段一样。


### 注意
+ 如果您覆盖内置的ORM，[Waterline](https://github.com/balderdashy/waterline)，则此模型文档不再适用。 这种情况下，除了您使用其他ORM库之外，模型将遵循您设置的约定（例如，Mongoose）。





<docmeta name="displayName" value="Models and ORM">
<docmeta name="nextUpLink" value="/documentation/concepts/models-and-orm/models">
<docmeta name="nextUpName" value="Models">
