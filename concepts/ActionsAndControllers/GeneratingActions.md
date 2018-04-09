# 生成controllers或者独立actions

你可以使用 [`sails-generate`](https://sailsjs.com/documentation/reference/command-line-interface/sails-generate) 从Sails命令行工具快速生成控制器，甚至只是一个独立actions。


### 生成控制器 controllers

实例, 创建一个控制器:

```sh
$ sails generate controller user
```

Sails 会创建 `api/controllers/UserController.js`:

```javascript
/**
 * UserController.js
 *
 * @description :: Server-side controller action for manging users.
 * @help        :: See https://sailsjs.com/documentation/concepts/controllers
 */
module.exports = {

}
```

### 创建独立actions

运行以下命令以生成独立action:

```sh
$ sails generate action user/signup
info: Created an action!
Using "actions2"...
[?] https://sailsjs.com/docs/concepts/actions
```

Sails会创建 `api/controllers/user/sign-up.js`:

```javascript
/**
 * user/sign-up.js
 *
 * @description :: Server-side controller action for handling incoming requests.
 * @help        :: See https://sailsjs.com/documentation/concepts/controllers
 */
module.exports = {


  friendlyName: 'Sign up',


  description: '',


  inputs: {

  },


  exits: {

  },


  fn: function (inputs, exits) {

    return exits.success();

  }


};

```


或者，使用 [classic actions](https://sailsjs.com/documentation/concepts/actions-and-controllers#?classic-actions) 界面:


```sh
$ sails generate action user/signup --no-actions2
info: Created a traditional (req,res) controller action, but as a standalone file
```

Sails会创建`api/controllers/user/sign-up.js`:

```javascript
/**
 * Module dependencies
 */

// ...


/**
 * user/signup.js
 *
 * Signup user.
 */
module.exports = function signup(req, res) {

  sails.log.debug('TODO: implement');
  return res.ok();

};
```



<docmeta name="displayName" value="Generating actions and controllers">
