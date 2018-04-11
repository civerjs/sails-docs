# 模型

模型表示一组结构化数据，称为记录。模型通常对应于数据库中的表/集合，属性对应于列/字段，记录对应于行/文档。

### 定义模型

按照惯例，模型是通过在Sails应用程序的`api/models/`文件夹中创建一个文件来定义的:

```javascript
// api/models/Product.js
module.exports = {
  attributes: {
    nameOnMenu: { type: 'string', required: true },
    price: { type: 'string', required: true },
    percentRealMeat: { type: 'number', defaultsTo: 20, columnType: 'FLOAT' },
    numCalories: { type: 'number' },
  },
};
```

有关设置模型定义时可用选项的完整操作，请参阅 [Model Settings](https://sailsjs.com/documentation/concepts/models-and-orm/model-settings), [Attributes](https://sailsjs.com/documentation/concepts/models-and-orm/attributes), and [Associations](https://sailsjs.com/documentation/concepts/models-and-orm/associations).

<!--
commented-out content at: https://gist.github.com/rachaelshaw/1d7a989f6685f11134de3a5c47b2ebb8#1


commented-out content at: https://gist.github.com/rachaelshaw/1d7a989f6685f11134de3a5c47b2ebb8#2
-->



### 使用模型

一旦Sails应用程序正在运行，它的模型可以从控制器action、helpers、tests以及后端代码的任何地方访问。 这使代码可以调用模型方法与您的数据库（甚至是多个数据库）进行通信。

模型中有许多内置方法，其中最重要的是模型方法，如[.find()](https://sailsjs.com/documentation/reference/waterline/models/find)和 [.create()](https://sailsjs.com/documentation/reference/waterline/models/create)。 您可以在[参考>Waterline（ORM>Models](https://sailsjs.com/documentation/reference/waterline-orm/models)中找到这些方法的详细使用文档。



### 查询方法

Sails中的每个模型都有一组暴露的方法，允许您以规范化的方式与数据库进行交互。这是与应用数据交互的主要方式。

由于他们必须向数据库发送查询并等待响应，因此大多数模型方法都是**异步**的。 也就是说，他们不会马上返回结果。像其他JavaScript中的异步逻辑（例如`setTimeout()`）一样，这意味着我们需要一些其他方式来确定它们何时执行完毕，它们是否成功，如果不成功，是什么样的错误（或其他异常情况 ） 发生。


在Node.js，Sails和JavaScript中，推荐的方法是使用[`async/await`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await).

有关处理查询的更多信息，请参阅 [Reference > Waterline (ORM) > Queries](https://sailsjs.com/documentation/reference/waterline-orm/queries).

### 丰富的pubsub方法

Sails还提供了一些其他的“丰富的pubsub”（或“RPS”）方法，专门用于使用动态执行简单的实时操作。 有关这些方法的更多信息，请参阅[Reference > WebSockets > Resourceful PubSub](https://sailsjs.com/documentation/reference/web-sockets/resourceful-pub-sub).


### 自定义模型方法

除了Sails提供的内置方法外，您还可以定义自己的模型方法。 自定义模型方法对于控制器代码原型设计是非常有用的; 即这允许你从你的控制器中抽成代码，并将它们变成可以从任何地方调用的可重用函数（即，不依赖于`req`或`res`）。


> 模型无法自动识别自定义模型，因此您必须小心使用，以免无意中重写内置方法（不要定义名为“create”的方法等）
> 如果你不确定用法，可以写一个[helper](https://sailsjs.com/documentation/concepts/helpers) instead.

自定义模型方法可以是同步或异步函数，但更多的时候，它们是_异步的_。 按照惯例，异步模型方法应该是`async`函数，它接受一个`options`字典作为它们的参数。

For example:

```js
// in api/models/Monkey.js...

// Find monkeys with the same name as the specified person
findWithSameNameAsPerson: async function (opts) {
	var person = await Person.findOne(person);
	
	if (!person) {
		let err = new Error(require('util').format('Cannot find monkeys with the same name as the person w/ id=%s because that person does not exist.', person));
		err.code = 'E_UNKNOWN_PERSON';
		throw err;
	}
	
	return await Monkey.find({ name: person.name });
}
```
> 请注意，该函数内没有任何`try/catch`的代码，这是留给谁来调用它的函数（例如一个动作）。

接下来可以:

```js
var monkeys = await Monkey.findWithSameNameAsPerson(37);
```

> 有关更多提示，请阅读有关涉及的事件 [Timothy the Monkey]().

##### 实例方法呢？

从Sails v1.0开始，实例方法已从Sails和Waterline中移除。 尽管像.save()和.destroy()这样的实例方法在应用代码中有时很方便，但至少在Node.js中，许多用户发现它们导致了意想不到的后果和设计缺陷。


例如，管理婚礼记录的应用程序。在Person模型上编写一个实例方法来更新数据库中两个个体的`spouse`属性似乎是个好主意。 您编写类似的控制器代码:

```js
personA.marry(personB, function (err) {
  if (err) { return res.serverError(err); }
  return res.ok();
})
```

这看起来不错，直到实现一个稍微不同的动作，相同的逻辑，但唯一可用的数据是“personA”（而不是整个记录）的ID。在这种情况下，你卡住了。无论如何，重写实例方法是一种静态方法！

更好的策略是从一开始就编写一个自定义（静态）模型方法。 这使得你的函数更具可重用性/多功能性，因为无论你是否有实际的记录实例，它都可以被访问。 你可能会重构前面例子中的代码:

```js
Person.marry(personA.id, personB.id, function (err) {
  if (err) { return res.serverError(err); }
  return res.ok();
})
```

### 区分大小写

无论数据库如何处理查询，Sails 1.0中的查询不再被强制为不区分大小写。 这会大大提高查询性能并提高索引利用率。 大多数数据库在默认情况下都是大小写敏感的，极少数情况下不是这样，如果您想要更改该行为，必须修改数据库才能执行此操作。

例如，默认情况下，MySQL将使用大小写不敏感的数据库归类，这与sails-disk不同，因此您可能会遇到从开发到生产的不同结果。 为了解决这个问题，你可以将你的MySQL数据库中的表设置为一个大小写敏感的排序规则，比如`utf8_bin`。



<!--
commented-out content at: https://gist.github.com/rachaelshaw/1d7a989f6685f11134de3a5c47b2ebb8#3


commented-out content at: https://gist.github.com/rachaelshaw/1d7a989f6685f11134de3a5c47b2ebb8#4

commented-out content at: https://gist.github.com/rachaelshaw/1d7a989f6685f11134de3a5c47b2ebb8#5

commented-out content at: https://gist.github.com/rachaelshaw/1d7a989f6685f11134de3a5c47b2ebb8#6
-->

<docmeta name="displayName" value="Models">
<docmeta name="nextUpLink" value="/documentation/concepts/configuration">
<docmeta name="nextUpName" value="Configuration">
