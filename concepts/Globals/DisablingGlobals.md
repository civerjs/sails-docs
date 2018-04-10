# 禁用globals

Sails通过检查[`sails.config.globals`](https://sailsjs.com/documentation/reference/configuration/sails-config-globals)来确定要暴露的全局变量，该变量通常在[`config / globals.js`](https://sailsjs.com/documentation/anatomy/config/globals.js)。


禁用全局变量, 只需要设置为 `false`:

```js
// config/globals.js
module.exports.globals = false;
```

禁用部分全局变量,需要指定一个object,例如:

```js
// config/globals.js
module.exports.globals = {
  _: false,
  async: false,
  models: false,
  services: false
};
```

### 注意

> + 请记住，在sails加载之前，全局变量、包括`sails`都不可访问。 另外，你不能在函数之外使用`sails.models.user`或`User`（因为`sails`还没有完成加载。）

<!-- not true anymore:
Most of this section of the docs focuses on the methods and properties of `sails`, the singleton object representing your app.
-->

<docmeta name="displayName" value="Disabling globals">
