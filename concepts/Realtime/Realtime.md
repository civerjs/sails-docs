# 实时通信 (aka Sockets)

### 概述

Sails应用程序能够在客户端和服务器之间进行全双工实时通信。这意味着客户端（例如浏览器选项卡，Raspberry Pi等）任何时候可以保持与Sails后端的持久连接，并且消息可以从客户端发送到服务器（例如AJAX）或从服务器发送到客户端（例如“comet”） 。 实时通信的两种常见用途是实时聊天和多人游戏。 Sails使用[socket.io](http://socket.io)库在服务器上实现实时通讯，在客户端使用[sails.io.js](https://sailsjs.com/documentation/reference/web-sockets/socket-client/io-socket-on)库。 在整个Sails文档中，术语**socket**和**websocket**通常指Sails应用程序和客户端之间的双向持续通信通道。

通过socket与Sails应用进行通信类似于使用AJAX，因为这两种方法都允许网页与服务器进行无刷新交换。 但是，socket与AJAX在两个重要方面有所不同：首先，只要网页打开，socket就可以保持连接到服务器，从而允许它维护_state_（AJAX请求，如所有HTTP请求，均为_stateless_）。 其次，由于连接始终在线，Sails应用程序可以随时将数据发送到socket（“realtime”），而AJAX只允许服务器在发出请求时作出响应。


### 实时模型更新与丰富的pub-sub

向Sails的[blueprint actions](https://sailsjs.com/documentation/reference/blueprint-api)发出请求的socket会自动订阅关于他们通过[resourceful pub-sub API](https://sailsjs.com/documentation/reference/web-sockets/resourceful-pub-sub)检索模型的实时消息。 您还可以在自定义控制器action中使用此API将消息发送给对此模型感兴趣的客户。


##### 示例

将客户端套接字连接到服务器，订阅`user`事件，并请求`/user`获得当前和将来的用户模型的实例。

```html
<!-- Simply include the sails.io.js script, and a client socket will be created for you -->
<script type="text/javascript" src="/js/dependencies/sails.io.js"></script>
<script type="text/javascript">
// 自动创建的socket显示为io.socket。
// 使用.on()订阅客户端上的“用户”事件。
// 这个事件由Sails发送"create", "update",
// "delete", "add" and "remove" blueprints to any socket that
// 订阅了一个或多个用户模型实例。
io.socket.on('user', function gotHelloMessage (data) {
  console.log('User alert!', data);
});
// 使用.get（'/ user'）检索当前用户模型的列表，
// 订阅此socket到这些模型，并订阅此socket
// 通知有关新用户模型的创建时间。
io.socket.get('/user', function gotResponse(body, response) {
  console.log('Current users: ', body);
})
</script>
```

### 使用`sails.sockets`定制实时通信

Sails在客户端和服务器上公开了一个丰富的API，用于发送自定义实时消息。

##### 示例

以下是将socket连接到Sails/Node.js服务器，并监听名为“hello”的socket事件的客户端代码:

```html
<!-- Simply include the sails.io.js script, and a client socket will be created and auto-connected for you -->
<script type="text/javascript" src="/js/dependencies/sails.io.js"></script>
<script type="text/javascript">

// 自动创建的socket显示为io.socket。

// Use `io.socket.on()` to listen for the 'hello' event:
io.socket.on('hello', function (data) {
  console.log('Socket `' + data.id + '` joined the party!');
});
</script>
```

然后，在客户端，我们也可以发送一个_socket请求_。这种情况下，我们连接浏览器并在单击按钮时发送socket请求:

```js
$('button#say-hello').click(function (){

  // And use `io.socket.get()` to send a request to the server:
  io.socket.get('/say/hello', function gotResponse(data, jwRes) {
    console.log('Server responded with status code ' + jwRes.statusCode + ' and data: ', data);
  });

});

```


同时，在服务器上...

为了响应对GET /say/hello的请求，我们使用一个action。 在我们的action中，我们将订阅请求socket到“funSockets”房间，然后向该房间中的所有socket（不包括新socket）广播“hello”消息。

```javascript
// In /api/controllers/SayController.js
module.exports = {

  hello: function(req, res) {

    // 确保这是一个socket请求（不是传统的HTTP）
    if (!req.isSocket) {
      return res.badRequest();
    }

    // 有请求加入“funSockets”房间的socket。
    sails.sockets.join(req, 'funSockets');

    // 向所有加入的socket广播通知
    // “funSockets”房间，不包括新增的socket:
    sails.sockets.broadcast('funSockets', 'hello', { howdy: 'hi there!'}, req);

    // ^^^
    // 此时，我们已经向所有拥有socket的socket消息发出了广播,xx加入了“funSockets”的房间。 但这不表示他们在_listening_。 换句话说，要实际处理socket消息，连接的socket需要监听这个事件（在这里我们用事件名称“hello”广播了我们的消息）。客户端像这样:
    // ```
    // io.socket.on('hello', function (broadcastedData){
    //   console.log(data.howdy);
    //   // => 'hi there!'
    // }
    // ```

    //现在我们已经广播了socket消息，我们需要继续
    //在action中需要使用其他逻辑，然后发送一个
    //响应。在示例中，我们只是想结束本次连接，所以我们会继续

    // 以200 响应请求。
    // 这里返回的数据就是我们在客户端以`data`的形式接收到的数据:
    // `io.socket.get('/say/hello', function gotResponse(data, jwRes) { /* ... */ });`
    return res.json({
      anyData: 'we want to send back'
    });

  }
}
```

### 参考

* 请参阅[sails.io.js库](https://sailsjs.com/documentation/reference/web-sockets/socket-client/io-socket-on)的完整参考以了解如何在客户端上使用socket端与您的Sails应用程序进行通信。
* 请参阅[sails.sockets](https://sailsjs.com/documentation/reference/web-sockets/sails-sockets)参考，了解如何将自定义消息从服务器发送到连接的socket。
* 请参阅[resourceful pub-sub](https://sailsjs.com/documentation/reference/web-sockets/resourceful-pub-sub) 参考，了解Sails的蓝图API如何自动发送有关[模型]更改的实时消息。
* 访问[Socket.io](http://socket.io)网站，了解有关Sails实时通信的底层库的更多信息

<docmeta name="displayName" value="Realtime">
