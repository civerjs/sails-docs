# 生命周期回调

### 概述

生命周期回调(lifecycle)是在某些_模型_操作之前或之后自动调用的函数。 例如，我们有时使用lifecycle在创建或更新“Account”模型之前自动加密密码。

Sails在默认情况下暴露了一些lifecycle。

使用`createEach`批量插入的数据不使用lifecycle。


##### 在`create`后调用

`afterCreate` 生命周期回调(lifecycle)只会在`fetch` meta标志设置为`true`的查询中运行。 有关使用`meta`标志的更多信息，请参见[Waterline Queries](https://sailsjs.com/documentation/reference/waterline-orm/queries/meta).

  - beforeCreate: fn(recordToInsert, cb)
  - afterCreate: fn(newlyInsertedRecord, cb)

##### `update`后调用

`afterUpdate` 生命周期回调(lifecycle)只会在`fetch` meta标志设置为`true`的查询中运行。 有关使用`meta`标志的更多信息，请参见 [Waterline Queries](https://sailsjs.com/documentation/reference/waterline-orm/queries/meta).

  - beforeUpdate: fn(valuesToUpdate, cb)
  - afterUpdate: fn(updatedRecord, cb)

##### `destroy`后调用

`afterDestroy` 生命周期回调(lifecycle)只会在`fetch` meta标志设置为`true`的查询中运行。 有关使用`meta`标志的更多信息，请参见[Waterline Queries](https://sailsjs.com/documentation/reference/waterline-orm/queries/meta).

  - beforeDestroy: fn(criteria, cb)
  - afterDestroy: fn(destroyedRecord, cb)


### 例子

如果要在保存到数据库之前对密码进行加密处理，可以使用`beforeCreate`生命周期回调。

```javascript
var bcrypt = require('bcrypt');

module.exports = {

  attributes: {

    username: {
      type: 'string',
      required: true
    },

    password: {
      type: 'string',
      minLength: 6,
      required: true,
      columnName: 'hashed_password'
    }

  },


  // Lifecycle Callbacks
  beforeCreate: function (values, cb) {

    // Hash password
    bcrypt.hash(values.password, 10, function(err, hash) {
      if(err) return cb(err);
      values.password = hash;
      //calling cb() with an argument returns an error. Useful for canceling the entire operation if some criteria fails.
      cb();
    });
  }
};
```



<docmeta name="displayName" value="Lifecycle callbacks">
