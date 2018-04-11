# 访问控制和权限

Sails中的策略旨在控制二进制（"yes or no"）访问特定操作。 它们非常适合检查用户是否登录，或者用于其他简单的"yes or no"检查，例如登录用户是否为“超级管理员”。

要查看访问控制实例以及登录，身份验证和密码恢复，请初始化网站应用程序:

```bash
sails new foo

# Then choose "Web App"
```

### 动态权限

对于更复杂的权限方案，用户访问权限取决于他们是否正在尝试执行什么，您将需要涉及数据库。 虽然您可以使用策略来实现此目的，但通常使用[helper](https://sailsjs.com/documentation/concepts/helpers)更直接且更易于维护，


例如，你可能会创建`api/helpers/check-permissions.js`:

```javascript
module.exports = {


  friendlyName: 'Check permissions',


  description: 'Look up a user\'s "rights" within a particular organization.',


  inputs: {
    userId: { type: 'number', required: true },
    orgId: { type: 'number', required: true }
  },

  exits: {
    success: {
      outputFriendlyName: 'Rights',
      outputDescription: `A user's "rights" within an org.`,
      outputType: ['string']
    },
    orgNotFound: {
      description: 'No such organization exists.'
    }
  },

  fn: async function(inputs, exits) {
    var org = await Organization.findOne(inputs.orgId)
    .populate('adminUsers', { id: inputs.userId })
    .populate('regularUsers', { id: inputs.userId });

    if (!org) { throw 'orgNotFound'; }

    var rights = [];
    if (org.regularUsers.length !== 0) {
      rights = ['basicAccess', 'inviteRegularUsers'];
    } else if (org.adminUsers.length !== 0) {
      rights = ['basicAccess', 'inviteRegularUsers', 'removeRegularUsers', 'inviteOrgAdmins'];
    } else if (org.owner === inputs.userId) {
      rights = ['basicAccess', 'inviteRegularUsers', 'removeRegularUsers', 'inviteOrgAdmins', 'removeOrDemoteOrgAdmins'];
    }
    // ^^This could be as simple or as granular as you need, e.g.
    // ['basicAccess', 'inviteRegularUsers', 'inviteOrgAdmins', 'removeRegularUsers', 'removeOrDemoteOrgAdmins']

    return exits.success(rights);
  }

};
```


然后在你的action中，例如 `api/controllers/demote-org-admin.js`:

```javascript
//…
var rights = await checkPermissions(this.req.session.userId, inputs.orgId)
.intercept('orgNotFound', 'notFound');

if (!_.contains(rights, 'removeOrDemoteOrgAdmins')) {
  throw 'forbidden';
}

await Organization.removeFromCollection(inputs.orgId, 'adminUsers', inputs.targetUserId);
await Organization.addToCollection(inputs.orgId, 'regularUsers', inputs.targetUserId);

return exits.success();
```


> ### 注意
> 请记住，虽然我们在这里使用了`checkPermissions.with（...，...）`，但我们可以
> 也使用`.with()`并切换到命名参数:
>
> ```js
> await checkPermissions.with({
>   userId: this.req.session.userId,
>   orgId: inputs.orgId
> });
> ```
>
> The style you choose when calling a helper should depend on readability-- i.e.
> the number of different values you need to pass in, the complexity of those
> values, etc.  When in doubt, a good best practice is to optimize first for
> explicitness, then for readability, and only then for conciseness.  But the
> more confident/familiar you are with the usage of a helper, and the more frequently
> you use it, the more those priorities flip-flop.


<docmeta name="displayName" value="Access Control and Permissions">
