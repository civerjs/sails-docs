# 在多服务器（又名“群集”）环境中进行实时通信

通过默认配置，Sails允许单个服务器与其所有连接的客户端进行实时通信。当[将您的Sails应用程序扩展到多个服务器](https://sailsjs.com/documentation/concepts/deployment/scaling)时，需要一些额外的设置，以便将实时消息可靠地传送到客户端。这种设置通常涉及:

1. 设置[Redis](http://redis.io/)的[托管](https://www.google.com/search?q=hosted+redis)实例。
2. 安装[@sailshq/socket.io-redis](https://npmjs.com/package/@sailshq/socket.io-redis)作为Sails应用程序的依赖项。
3.将[sails.config.sockets.adapter]（(https://sailsjs.com/documentation/reference/configuration/sails-config-sockets#?commonlyused-options)设置更新为`@sailshq /socket.io-redis`，并设置适当的“主机”，“密码”等字段指向您的托管Redis实例。

您的托管Redis安装不需要特殊设置; 只需将适当的主机地址和凭证插入到`/config/sockets.js`文件中，`@sailshq /socket.io-redis`适配器将为您处理所有事情。

> 注意：在多服务器环境中运行时，某些没有回调的socket方法是_变化的_，这意味着即使代码看起来立即执行，它们也需要不确定的时间才能完成。 在涉及代码时，记住这一点很好，例如，可以通过调用[`.addRoomMembersToRoom（）`](https://sailsjs.com/documentation/reference/web-sockets/sails-sockets/add-room-members-to-room)立即调用[`.broadcast（）`](https://sailsjs.com/documentation/reference/web-sockets/sails-sockets/sails-sockets-broadcast)。 不过这种情况下，新房间成员可能不会收到新广播的消息，因为当调用`.broadcast()`时，新加入成员不太可能已经传播到集群中的其他服务器。


### 参考

* 请参阅[sails.io.js库](https://sailsjs.com/documentation/reference/web-sockets/socket-client)的完整参考以了解如何使用客户端上的socket与您的Sails进行通信应用程序。
* 请参阅[sails.sockets](https://sailsjs.com/documentation/reference/web-sockets/sails-sockets)参考，了解如何将消息从服务器发送到连接的socket
* 请参阅[resourceful pub-sub](https://sailsjs.com/documentation/reference/web-sockets/resourceful-pub-sub)参考，了解如何使用Sails蓝图自动发送有关[模型的更改的实时消息](https://sailsjs.com/documentation/concepts/models-and-orm/models)。
* 访问[Socket.io](http://socket.io）网站，了解有关Sails用于实时通信的底层库的更多信息

<docmeta name="displayName" value="Multi-server environments">
