# 客户端和服务器之间的实时通信

将实时消息从客户端发送到Sails应用程序的最简单方法是使用[sails.io.js](https://sailsjs.com/documentation/reference/web-sockets/sails-io-js)库。 该库允许您轻松地将socket连接到正在运行的Sails应用程序，并提供用于向[Sails routes](https://sailsjs.com/documentation/concepts/routes)发出请求的方法，这些方法的处理方式很像HTTP请求。

使用`<script>`标签将sails.io.js库自动添加到新Sails应用的[布局模板](https://sailsjs.com/documentation/concepts/views/layouts)中。 当网页加载`sails.io.js`脚本时，它会尝试创建一个新的[client socket](https://sailsjs.com/documentation/reference/web-sockets/socket-client/sails-socket) 并将其连接到Sails应用程序，将其公开为全局变量`io.socket`。

### 示例

首先在代码中包含`sails.io.js`库, 并使用自动连接的socket向Sails应用的`/hello`路由发出请求:

```html
<script type="text/javascript" src="/js/dependencies/sails.io.js"></script>
<script type="text/javascript">
io.socket.get('/hello', function responseFromServer (body, response) {
  console.log("The server responded with status " + response.statusCode + " and said: ", body);
});
</script>
```

更高级的用例：禁用eager（自动连接）socket，手动创建一个新的客户端socket。当它成功连接到服务器时，我们会让它显示一条消息:
```html
<script type="text/javascript" src="/js/dependencies/sails.io.js" autoConnect="false"></script>
<script type="text/javascript">
var mySocket = io.sails.connect();
mySocket.on('connect', function onConnect () {
  console.log("Socket connected!");
});
</script>
```

### Socket请求与传统的AJAX请求

您可能已经注意到客户端socket`.get()`与创建AJAX请求非常相似，例如使用jQuery的`.get()`方法。这是故意的 - 无论请求来自何处，您的目标都希望从Sails获得相同的响应。 使用客户端socket进行请求的好处在于，Sails应用中的[controller action](https://sailsjs.com/documentation/concepts/controllers#?actions)）将可以访问发出请求的socket， 允许它将socket订阅到实时通知（请参阅[从服务器发送实时消息](https://sailsjs.com/documentation/concepts/realtime/on-the-server)）。


### 参考

* View the full [sails.io.js library](https://sailsjs.com/documentation/reference/web-sockets/socket-client) reference.
* See the [sails.sockets](https://sailsjs.com/documentation/reference/web-sockets/sails-sockets) reference to learn how to send messages from the server to connected sockets
* See the [resourceful pub-sub](https://sailsjs.com/documentation/reference/web-sockets/resourceful-pub-sub) reference to learn how to use Sails blueprints to automatically send realtime messages about changes to your [models](https://sailsjs.com/documentation/concepts/models-and-orm/models).
* Visit the [Socket.io](http://socket.io) website to learn more about the underlying library Sails uses for realtime communication

<docmeta name="displayName" value="On the client">
