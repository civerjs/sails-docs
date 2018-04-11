# 策略Policies
### 概述

Sails中的策略是用于授权和访问控制的多功能工具，它们让您在运行操作前执行一些逻辑，以确定是否继续处理请求。 策略最常见的用例是将某些操作限制为仅限于用户登录。

> 注意：策略仅应用于控制器和操作，而不应用于视图。 如果您在[routes.js配置文件](https://sailsjs.com/documentation/reference/configuration/sails-config-routes)中定义了直接指向视图的路由，则不会应用策略。为了确保应用策略，您可以定义显示视图的action，并将您的路由指向该action。


### 何时使用策略

最好避免在您的应用中实施大量或复杂的策略。在实现基于角色的权限等功能时，请依靠您的[action](https://sailsjs.com/documentation/concepts/actions-and-controllers)来拒绝不需要的访问。并提供相应的本地视图或者JSON响应数据。

例如，如果您需要实现用户级别或角色权限，最直接的方法是在控制器操作开始处理相关检测 - 内置或通过call helper 遵循此做法将显着提高代码的可维护性。


### 使用策略守护action和控制器

Sails有一个内置的ACL（访问控制列表），位于`config/policies.js`中。 该文件用于将策略映射到您的action和控制器。

这个文件是*一个描述*，它描述了*你的应用程序的权限应该是什么样子而不是*他们应该如何工作。 这使得新开发人员可以更轻松地了解正在发生的事情，并且可以让您的程序更加灵活，因为需求随着时间的推移发生变化。

`config/policies.js`文件是一个字典，其属性和值根据您是否将策略应用于[controllers](https://sailsjs.com/documentation/concepts/actions-and-controllers#?controllers)或[独立操作](https://sailsjs.com/documentation/concepts/actions-and-controllers#?standalone-actions)而有所不同。

##### 将策略应用于控制器

要将策略应用于控制器，请使用控制器名称作为`config/policies.js`字典中属性的名称，并将其值设置为映射到该控制器中action的字典。 使用`*`来表示“所有未映射的动作”。 策略的_名称_与其文件名相同，减去文件扩展名。

```js
module.exports.policies = {
  UserController: {
    // By default, require requests to come from a logged-in user
    // (runs the policy in api/policies/isLoggedIn.js)
    '*': 'isLoggedIn',

    // Only allow admin users to delete other users
    // (runs the policy in api/policies/isAdmin.js)
    'delete': 'isAdmin',

    // Allow anyone to access the login action, even if they're not logged in.
    'login': true
  }
};
```

##### 将策略应用于独立操作

要将策略应用于一个或多个独立操作，请在`config/policies.js`字典中使用操作路径（相对于`api/controllers`）作为属性名称，并将该值设置为该应用的策略的action。通过在操作路径末尾使用通配符"/*",可以将策略应用于以该路径开头的所有action。这是与上面相同的一组策略，重写为适用于独立操作:

```js
module.exports.policies = {
  'user/*': 'isLoggedIn',
  'user/delete': 'isAdmin',
  'user/login': true
}
```

> 请注意，此示例与基于控制器的策略略有不同，因为`isLoggedIn`策略将应用于`api/controllers/user`文件夹中的所有action_(或子文件夹`api/controllers/user`) _除`delete`和`login`外_ ，如下一节所述）。

##### 策略排序和优先顺序

值得注意的是，策略不会“cascade”。在上面的示例中，`isLoggedIn`策略将应用于`UserController.js`文件中的所有操作（或者在`api / controllers/user`下的独立操作），`delete`和`login`除外。 例如，如果您希望将多个策略应用于某个操作，请列出数组中的策略:

```javascript
'getEncryptedData': ['isLoggedIn', 'isInValidRegion']
```

##### 对蓝图操作使用策略

Sails的内置[blueprint API](https://sailsjs.com/documentation/concepts/blueprints)是使用常规Sails操作实现的。 唯一的区别是蓝图行动是隐含的。

要将您的策略应用于蓝图行动，请设置您的策略映射，就像我们在上面的示例中所做的一样，但指出了在您的控制器的相关隐式[蓝图action](https://sailsjs.com/documentation/concepts/blueprints/blueprint-actions)的名称（或一个独立action）。例如:
```js
module.exports.policies = {
  UserController: {
    // Apply the 'isLoggedIn' policy to the 'update' action of 'UserController'
    update: 'isLoggedIn'
  }
};
```
or
```js
module.exports.policies = {
  'user/update': 'isLoggedIn'
};
```

##### 全局策略

您可以将策略应用于通过使用`*`属性未明确映射的_全部_动作。 例如:

```js
module.exports.policies = {
  '*': 'isLoggedIn',
  'user/login': true
};
```
除了`api /controllers/user/login.js`（或`api/controllers/UserController.js`中的`login`动作），这会将`isLoggedIn`策略应用于每个动作。

### 内置策略
Sails提供了两个内置的策略，可以应用于全局或特定的控制器或操作。
  + `true`: public access  (允许任何人访问映射的控制器/操作)
  + `false`: **NO** access (不允许允许访问映射的控制器/动作)

 `'*': true` is the default policy for all controllers and actions.  In production, it's good practice to set this to `false` to prevent access to any logic you might have inadvertently exposed.


### 编写你的第一个策略

这是一个简单的`isLoggedIn`策略，用于防止未经身份验证的用户访问。 它检查会话是否有`userId`属性，如果它找不到，则发送默认的[`forbidden`响应](https://sailsjs.com/documentation/concepts/extending-sails/custom-responses/default-responses#?resforbidden)。 （对于许多应用程序，这是必须需要的策略。）以下示例假定，在用于验证用户的控制器action中，您将`req.session.userId`的值设置为[truthy](https://developer.mozilla.org/en-US/docs/Glossary/Truthy)。


```javascript
// policies/isLoggedIn.js
module.exports = async function (req, res, proceed) {

  // If `req.me` is set, then we know that this request originated
  // from a logged-in user.  So we can safely proceed to the next policy--
  // or, if this is the last policy, the relevant action.
  // > For more about where `req.me` comes from, check out this app's
  // > custom hook (`api/hooks/custom/index.js`).
  if (req.me) {
    return proceed();
  }

  //--•
  // Otherwise, this request did not come from a logged-in user.
  return res.forbidden();

};
```




<docmeta name="displayName" value="Policies">
<docmeta name="nextUpLink" value="/documentation/concepts/helpers">
<docmeta name="nextUpName" value="Helpers">
