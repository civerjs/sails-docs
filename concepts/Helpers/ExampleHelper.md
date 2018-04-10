# 一个helper的例子

helper的常见用法是封装一些重复的数据库查询。 例如，假设我们的应用程序有一个`User`模型，其中包含一个字段`lastActiveAt`，用于跟踪用户上次登录的时间。 这种程序的常见任务是检索最近在线用户的列表。 我们可以编写一个helper,而不是将这个查询硬编码到多个位置：

```javascript
// api/helpers/get-recent-users.js
module.exports = {


  friendlyName: 'Get recent users',


  description: 'Retrieve a list of users who were online most recently.',


  extendedDescription: 'Use `activeSince` to only retrieve users who logged in since a certain date/time.'


  inputs: {

    numUsers: {
      friendlyName: 'Number of users',
      description: 'The maximum number of users to retrieve.',
      type: 'number',
      defaultsTo: 5
    },

    activeSince: {
      description: 'Cut-off time to look for logins after, expressed as a JS timestamp.',
      extendedDescription: 'Remember: A _JS timestamp_ is the number of **milliseconds** since [that fateful night in 1970](https://en.wikipedia.org/wiki/Unix_time).',
      type: 'number'
      defaultsTo: 0
    }

  },


  exits: {

    success: {
      outputFriendlyName: 'Recent users',
      outputDescription: 'An array of users who recently logged in.',
    },

    noUsersFound: {
      description: 'Could not find any users who logged in during the specified time frame.'
    }

  },


  fn: async function (inputs, exits) {

    // Run the query
    var users = await User.find({
      active: true,
      lastLogin: { '>': inputs.activeSince }
    })
    .sort('lastLogin DESC')
    .limit(inputs.numUsers);

    // If no users were found, trigger the `noUsersFound` exit.
    if (users.length === 0) {
      throw 'noUsersFound';
    }

    // Otherwise return the records through the `success` exit.
    return exits.success(users);

  }

};
```

### 用法

从应用程序代码中调用此帮助程序使用默认选项（例如，在一个action中），我们将使用:

```javascript
var users = await sails.helpers.getRecentUsers();
```

要改变返回用户的用法，我们可以传入一些值:

```javascript
var users = await sails.helpers.getRecentUsers(50);
```

或者，例如，获取自2017年圣帕特里克节以来已登录的10位最新用户:

```javascript
await sails.helpers.getRecentUsers(10, (new Date('2017-03-17')).getTime());
```

> 注意：在运行时传递给helper的这些值被称为**argins**或options，它们与helper声明的inputs的键顺序（例如`numUsers`和`activeSince`）相对应。

再次，链入`.with()`以使用命名参数:

```javascript
await sails.helpers.getRecentUsers.with({
  numUsers: 10,
  activeSince: (new Date('2017-03-17')).getTime()
});
```


##### 例外

最后，为了明确地处理`noUsersFound`出口，而不是将它看作简单地其他错误，我们可以使用 [`.intercept()`](https://sailsjs.com/documentation/reference/waterline-orm/queries/intercept) 或者 [`.tolerate()`](https://sailsjs.com/documentation/reference/waterline-orm/queries/tolerate):

```javascript
var users = await sails.helpers.getRecentUsers(10)
.tolerate('noUsersFound', ()=>{
  // ... handle the case where no users were found. For example:
  sails.log.verbose(
    'Worth noting: Just handled a request for active users during a time frame '+
    'where no users were found.  Anyway, I didn\'t think this was possible, because '+
    'our app is so cool and popular.  But there you have it.'
  );
});
```

```javascript
var users = await sails.helpers.getRecentUsers(10)
.intercept('noUsersFound', ()=>{
  return new Error('Inconceivably, no active users were found for that timeframe.');
});
```
使用助手的一个关键优势来自于，能够更新功能并涉及应用程序许多部分，这些都是通过在一个地方更改代码来实现的。 例如，通过将`numUsers`的默认值从`5`更改为`15`，我们更新使用helper的_any_地方返回的默认列表的大小。 另外，通过使用像`numUsers`和`activeSince`这样定义明确的输入，我们保证如果我们意外地使用了无效（即非数字）值，我们会收到有效的错误。


### 注意

关于上面的示例`getRecentUsers()`helper的更多注意事项:

> * 许多字段，例如`description`和`friendlyName`并不是必须的，但在保持代码可维护性方面非常有用，特别是跨多个应用程序共享helper时。
> * noUsersFound退出可能有用，也可能没有用，具体取决于您的应用程序。 如果您希望一直在没有反馈的情况下执行某些特定操作（例如，重定向到不同的页面），exits将是一个好主意。 另一方面，如果您只是想根据是否返回来调整视图中的某些文本，更好的方法是使用`success`退出并检查您的action中返回数组的长度或查看代码。

<docmeta name="displayName" value="Example helper">
