# Shell脚本

Sails与[Whelk](https://github.com/sailshq/whelk)捆绑在一起，它允许您将JavaScript函数作为shell脚本运行。这对于运行预定作业（cron，Heroku调度程序），工作进程以及任何其他需要访问Sails应用程序的模型，配置和帮助程序的自定义一次性脚本都很有用。


### 你的第一个脚本

要添加新脚本，只需在应用程序的`scripts /`文件夹中创建一个文件即可。

```bash
sails generate script hello
```

Then, to run it, use:

```bash
sails run hello
```

> If you need to run a script without global access to the `sails` command-line interface (in a Procfile, for example), use `node ./node_modules/sails/bin/sails run hello`.

### Example

Here's a more complex example that you'd be more likely to see in a real-world app:

```js
// scripts/send-email-proof-reminders.js
module.exports = {

  description: 'Send a reminder to any recent users who haven\'t confirmed their email address yet.',

  inputs: {
    template: {
      description: 'The name of another email template to use as an optional override.',
      type: 'string',
      defaultsTo: 'reminder-to-confirm-email'
    }
  },

  fn: async function (inputs, exits) {

    await User.stream({
      emailStatus: 'pending',
      emailConfirmationReminderAlreadySent: false,
      createdAt: { '>': Date.now() - 1000*60*60*24*3 }
    })
    .eachRecord(async (user, proceed)=>{
      await sails.helpers.sendTemplateEmail.with({
        template: 'reminder-to-confirm-email',
        templateData: {
          user: user
        },
        to: user.emailAddress
      });
      return proceed();
    });//∞

    return exits.success();

  }
};
```

Then you can run:

```bash
sails run send-email-proof-reminders
```

For more detailed information on usage, see the [`whelk` README](https://github.com/sailshq/whelk/blob/master/README.md).

<docmeta name="displayName" value="Shell scripts">
<docmeta name="nextUpLink" value="/documentation/concepts/models-and-orm">
<docmeta name="nextUpName" value="Models and ORM">
