# 记录集

_记录_ 是你从`.find()`或`.findOne()`得到的。 每条记录都是唯一可识别的对象，与物理数据库条目一一对应; 例如 Oracle/MSSQL/ PostgreSQL/MySQL中的一行，MongoDB中的文档或Redis中的散列。

```js
var records = await Order.find();

console.log('Found %d records', records.length);
if (records.length > 0) {
  console.log('Found at least one record, and its `id` is:',records[0].id);
}
```


### JSON序列化

在Sails中，记录只是字典（普通的JavaScript对象），这意味着它们可以很容易地表示为JSON。 但您也可以使用[`customToJSON`模型设置]自定义记录来自特定模型的记录方式(https://sailsjs.com/documentation/concepts/models-and-orm/model-settings#?customtojson).


### 填充值

除了电子邮件地址，电话号码和生日等基本属性数据外，Waterline还可以使用[associations](https://sailsjs.com/documentation/concepts/models-and-orm/associations)动态存储和检索链接的记录集。 当在查询上调用[`.populate()`](https://sailsjs.com/documentation/reference/waterline-orm/queries/populate)时，每个结果记录将包含一个或多个填充值。 这些填充值中的每一个都是在查询时链接到该特定关联的记录（或记录数组）的快照。


填填充的类型取决于它关联什么类型:

+ `null`，或一个普通的JavaScript对象_（如果它对应于“models”关联）_or
+ 一个空数组或一组普通的JavaScript对象_（如果它对应于“collection”关联_



例如，假设我们正在处理可爱的小狼的订单:

```js
var orders = await Order.find()
.populate('buyers')  // a "collection" association
.populate('seller');  // a "model" association

// this array is a snapshot of the Customers who are associated with the first Order as "buyers"
console.log(orders[0].buyers);
// => [ {id: 1, name: 'Robb Stark'}, {id: 6, name: 'Arya Stark'} ]

// this object is a snapshot of the Company that is associated with the first Order as the "seller"
console.log(orders[0].seller);
// => { id: 42941, corporateName: 'WolvesRUs Inc.' }

// this array is empty because the second Order doesn't have any "buyers"
console.log(orders[1].buyers);
// => []

// this is `null` because there is no "seller" associated with the second Order
console.log(orders[1].seller);
// => null
```

##### 关联属性的预期 types/values

下表显示了在不同情况下从`.find（）`或`.findOne（）`调用返回的记录预期值。 

| &nbsp; |  without a `.populate()` added for the association | with `.populate()`, but no associated records found | with `.populate()`, with associated records found
|:--- |:--- | --- |:--- |
| Singular association (e.g. `seller`) | Whatever is in the database record for this attribute (typically `null` or a foreign key value) | `null` | A POJO representing a child record |
| Plural association (e.g. `buyers`) |  `undefined` (the key will not be present) | `[]` (an empty array) | An array of POJOs representing child records


##### 修改填充值

要修改特定记录或记录集的填充值，请调用 [.addToCollection()](https://sailsjs.com/documentation/reference/waterline-orm/models/add-to-collection)，[.removeFromCollection()](https://sailsjs.com/documentation/reference/waterline-orm/models/remove-from-collection)或[.replaceCollection()](https://sailsjs.com/documentation/reference/waterline-orm/models/replace-collection)模型方法。




<docmeta name="displayName" value="Records">
