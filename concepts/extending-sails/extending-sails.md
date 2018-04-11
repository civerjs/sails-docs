# 扩展Sails

为了与Node保持一致，Sails的目标是尽可能保持其核心，将除了最重要的功能以外的所有功能都分配给不同的模块[*]（./# foot1）。 目前，您可以为Sails制作三种类型的扩展程序:

+ [**Generators**](https://sailsjs.com/documentation/concepts/extending-sails/Generators) - 用于在Sails CLI中添加和覆盖功能。 *示例*：[sails-generate-model](https://www.npmjs.com/package/sails-generate-model)，它允许您使用`sails generate model foo`在命令行上创建模型。
+ [**适配器**](https://sailsjs.com/documentation/concepts/extending-sails/Adapters) - for integrating Waterline (Sails' ORM) with new data sources, including databases, APIs, or even hardware. *Example*: [sails-postgresql](https://www.npmjs.com/package/sails-postgresql), the official [PostgreSQL](http://www.postgresql.org/) adapter for Sails.
+ [**钩子**](https://sailsjs.com/documentation/concepts/extending-sails/Hooks) - for overriding or injecting new functionality in the Sails runtime.  *Example*: [sails-hook-autoreload](https://www.npmjs.com/package/sails-hook-autoreload), which adds auto-refreshing for a Sails project's API without having to manually restart the server.

如果您对开发Sails插件感兴趣，那么您通常会想要创建 [hook](https://sailsjs.com/documentation/concepts/extending-sails/Hooks).

<sub><a name="foot1">*</a> _Core hooks_, like `http`, `request`, etc. are hooks which are bundled with Sails out of the box.  They can be disabled by specifying a `hooks` configuration in your `.sailsrc` file, or when lifting Sails programatically.</sub>


<docmeta name="displayName" value="Extending Sails">
