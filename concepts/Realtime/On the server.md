# 将实时消息从服务器发送到一个或多个客户端

### 概述

Sails公开了两个用于与连接的socket客户端通信的API：高级的[resourceful pubsub API](https://sailsjs.com/documentation/reference/web-sockets/resourceful-pub-sub)和底层的[sails.sockets API](https://sailsjs.com/documentation/reference/web-sockets/sails-sockets)。


### 丰富的PubSub

Resourceful PubSub（发布/订阅）API提供了一种高级的方式来订阅sockets模型类和实例的socket。 仅使用此API完全有可能创建丰富的实时体验（例如，聊天应用程序）。Sails蓝图使用Resourceful PubSub自动发送有关新模型实例的通知以及对现有实例的更改，但您也可以在自定义控制器action中使用它们。

##### 示例

创建一个新的用户模型实例并通知所有感兴趣的客户

```javascript
// Create the new user
User.create({
  name: 'johnny five'
}).exec(function(err, newUser) {
  if (err) {
    // Handle errors here!
    return;
  }
  // Tell any socket watching the User model class
  // that a new User has been created!
  User.publishCreate(newUser);
});
```

### `sails.sockets`

`sails.sockets`API允许使用诸如[`sails.sockets.join（）`](https://sailsjs.com/documentation/reference/web-sockets/sails-sockets/sails-sockets-join)之类的方法直接与socket进行底层通信(订阅所有发送到特定“房间”的消息的socket），[`sails.sockets.leave()`](https://sailsjs.com/documentation/reference/web-sockets/sails-sockets/sails-sockets-leave)（取消订阅来自房间的套接字）和[`sails.sockets.broadcast()`](https://sailsjs.com/documentation/reference/web-sockets/sails-sockets/sails-sockets-broadcast)（向一个或多个房间中的所有用户广播消息）。


##### 示例

给房间添加一个socket “funSockets”

```javascript
sails.sockets.join(someSocket, "funSockets");
```
向“funSockets”房间广播“hello”消息。 
（1）通过sails.sockets.join()连接到服务器上的“funSockets”空间中的所有客户端socket都会收到此消息
（2）为服务器上的“hello”事件添加了一个侦听器，客户端使用[`socket.on('hello', ...)`](https://sailsjs.com/documentation/reference/web-sockets/socket-client/io-socket-on)。

```javascript
sails.sockets.broadcast("funSockets", "hello", "Hello to all my fun sockets!");
```

### Reference

* View the full [sails.sockets](https://sailsjs.com/documentation/reference/web-sockets/sails-sockets) API reference
* See the reference for the [sails.io.js library](https://sailsjs.com/documentation/reference/web-sockets/socket-client) to learn how to use sockets on the client side to communicate with your Sails app.
* See the [resourceful pub-sub](https://sailsjs.com/documentation/reference/web-sockets/resourceful-pub-sub) reference to learn how to use Sails blueprints to automatically send realtime messages about changes to your [models](https://sailsjs.com/documentation/concepts/models-and-orm/models).
* Visit the [Socket.io](http://socket.io) website to learn more about the underlying library Sails uses for realtime communication

<docmeta name="displayName" value="On the server">
