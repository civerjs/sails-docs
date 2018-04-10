### 翻译动态内容

如果您的后端存储多语言数据（例如，产品数据通过CMS以多种语言输入），则不应该依赖简单的JSON语言环境文件，除非您以某种计划动态地编辑语言翻译。 一种选择是以编程方式编辑语言翻译，可以使用自定义实现或通过翻译服务。 Sails/node-i18n JSON字符串文件与[webtranslateit.com](https://webtranslateit.com/en)使用的格式兼容.

另一方面，您可将这些类型的动态翻译串存储在数据库中。 如果这样，只要确保并相应地构建数据模型，以便可以通过本地ID（例如“en”，“es”，“de”等）存储和检索相关动态数据。这样，您就可以利用[`req.getLocale（）`](https://github.com/jeresig/i18n-node-2/tree/9c77e01a772bfa0b86fab8716619860098d90d6f#getlocale)方法。帮助您找出在给定响应中使用哪些翻译的内容，在您的应用中其他地方使用的约定保持一致。


<docmeta name="displayName" value="Translating dynamic content">
