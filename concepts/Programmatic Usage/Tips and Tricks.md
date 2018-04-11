# 编程方式启动使用的技巧和窍门

以编程方式加载Sails应用程序时，通常需要关闭某些未被有效使用的钩子，都是出于优化原因，并确保Sails应用程序和包含它的Node脚本之间的干扰最小。 要关闭一个钩子，把它作为`.load（）`或`.lift（）`的第一个参数的一部分发送到`hooks`字典中的`false`中。

此外，您需要关闭Sails[globals](https://sailsjs.com/documentation/concepts/globals)，尤其是在同时加载多个Sails应用时。 由于同一进程中的所有Node应用程序共享相同的全局变量。



```javascript
// Turn off globala and commonly unused hooks in programmatic apps
mySailsApp.load({
  hooks: {
     grunt: false,
     sockets: false,
     pubsub: false
  },
  globals: false
})
```

最后，请注意，虽然您可以使用Sails构造函数按程序创建和启动尽可能多的Sails应用程序，但每个应用程序只能启动一次。 一旦你在应用上调用`.lower()`，它就不能再次启动。

<docmeta name="displayName" value="Tips and tricks">
