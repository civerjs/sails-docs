# 蓝图动作 Blueprint actions

蓝图actions(不要与包含 [blueprint "action" _routes_]混淆(https://sailsjs.com/documentation/concepts/blueprints/blueprint-routes#?action-routes)) 是设计用于models的通用action.将它们视为您的应用程序的默认行为. 例如,你有一个`User.js`model包含隐式action`find`, `create`, `update`, `destroy`, `populate`, `add` and `remove`, 你就可以不用写.

默认情况下, 蓝图[RESTful routes](https://sailsjs.com/documentation/concepts/blueprints/blueprint-routes#?restful-routes)和 [shortcut routes](https://sailsjs.com/documentation/concepts/blueprints/blueprint-routes#?shortcut-routes)都与他们相应的蓝图行动有关.  但是，通过在该控制器文件中创建自定义action，可以为该控制器覆盖蓝图action (e.g. `ParrotController.find`).

当前版本的Sails附带以下蓝图actions:

+ [find](https://sailsjs.com/documentation/reference/blueprint-api/find-where)
+ [findOne](https://sailsjs.com/documentation/reference/blueprint-api/find-one)
+ [create](https://sailsjs.com/documentation/reference/blueprint-api/create)
+ [update](https://sailsjs.com/documentation/reference/blueprint-api/update)
+ [destroy](https://sailsjs.com/documentation/reference/blueprint-api/destroy)
+ [populate](https://sailsjs.com/documentation/reference/blueprint-api/populate)
+ [add](https://sailsjs.com/documentation/reference/blueprint-api/add-to)
+ [remove](https://sailsjs.com/documentation/reference/blueprint-api/remove-from)
+ [replace](https://sailsjs.com/documentation/reference/blueprint-api/replace)

### Socket 通知

大多数蓝图actions都具有实时功能，如果您的应用程序启用了WebSocket，则这些功能才会生效。例如, 如果**find**蓝图action收到来自socket客户端的请求， 它会[订阅](https://sailsjs.com/documentation/reference/web-sockets/resourceful-pub-sub/subscribe)该socket将来的通知。 然后，使用蓝图action（如**update**）实时更新记录集，Sails将[推送](https://sailsjs.com/documentation/reference/web-sockets/resourceful-pub-sub/publish)某些通知.

理解蓝图行为的最好办法是阅读其[参考页面](https://sailsjs.com/documentation/reference/blueprint-api) (或查看上面的列表). 但是，如果您希望更多地了解Sails Blueprint API中实时功能的工作方式，查看[**Concepts > Realtime**](https://sailsjs.com/documentation/concepts/realtime).  (如果你认为某些资料已经过时，你可能想看看 [原版的 "Intro to Sails.js" video from 2013](https://www.youtube.com/watch?v=GK-tFvpIR7c).)

> 有关由Sails中的蓝图action推送的所有通知的更高级功能细分，请参阅:
> + [Chart A (scenarios vs. notification types)](https://docs.google.com/spreadsheets/d/10FV9plyHR4gE9xIomIZlF-YS1S54oHEdvH8ZmTC1Fnc/edit#gid=0)
> + [Chart B (actions vs. recipients)](https://docs.google.com/spreadsheets/d/1B6i8aOoLNLtxJ4aeiA8GQ2lUQSvLOrP89RSLr7IAImw/edit#gid=0)

### 重写蓝图actions

您也可以通过定义相同名称的[自定义action]来覆盖控制器的任何蓝图action(https://sailsjs.com/documentation/concepts/actions-and-controllers) .

```javascript
// api/controllers/user/UserController.js
module.exports = {

  /**
   * A custom action that overrides the built-in "findOne" blueprint action.
   * As a dummy example of customization, imagine we were working on something in our app
   * that demanded we tweak the format of the response data, and that we only populate two
   * associations: "company" and "friends".
   */
  findOne: function (req, res) {

    sails.log.debug('Running custom `findOne` action.  (Will look up user #'+req.param(\'id\')...');

    User.findOne({ id: req.param('id') }).omit(['password'])
    .populate('company', { select: ['profileImageUrl'] })
    .populate('top8', { omit: ['password'] })
    .exec(function(err, userRecord) {
      if (err) {
        switch (err.name) {
          case 'UsageError': return res.badRequest(err);
          default: return res.serverError(err);
        }
      }

      if (!userRecord) { return res.notFound(); }

      if (req.isSocket) {
        User.subscribe(req, [user.id]);
      }

      return res.ok({
        model: 'user',
        luckyCoolNumber: Math.ceil(10*Math.random()),
        record: userRecord
      });
    });
  }

}
```

> 抑或，我们可以在`api/controllers/user/find-one.js`中创建这个独立的action，或者使用[actions2](https://sailsjs.com/documentation/concepts/actions-and-controllers#?actions-2).

<docmeta name="displayName" value="Blueprint actions">
