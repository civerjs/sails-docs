# 文件上传

在Sails中上传文件类似于为Node.js vanilla或Express应用程序上传文件。 但是，如果您使用不同的服务器端（如PHP，.NET，Python，Ruby或Java），那么与您的习惯可能不同。 但不要害怕：核心团队已经竭尽全力让文件上传变得更容易，同时保持其可扩展性和安全性。

Sails提供了一个名为[Skipper]”(https://github.com/balderdashy/skipper)的强大的“body parser，它可以很容易地实现streaming file uploads - 不仅服务器文件系统（如硬盘），也适用于Amazon S3，MongoDB的网格或其他支持的文件系统。


### 上传一个文件

文件以_file parameters_的形式上传到HTTP Web服务器。 以同样的方式，您可以将表单POST发送带有“name”，“email”和“password”等文本参数的URL，您可以将文件作为文件参数发送，如“avatar”或“newSong”。

以这个简单的例子:

```javascript
req.file('avatar').upload(function (err, uploadedFiles) {
  // ...
});
```

文件需要在某个控制器的`action`里面上传。 这是进阶的例子，演示了如何让用户上传头像图片，并与他的帐户关联。 假设您已经加入了权限检测，并已将登录用户的ID存储在`req.session.userId`中。


```javascript
// api/controllers/UserController.js
//
// ...


/**
 * Upload avatar for currently logged-in user
 *
 * (POST /user/avatar)
 */
uploadAvatar: function (req, res) {

  req.file('avatar').upload({
    // don't allow the total upload size to exceed ~10MB
    maxBytes: 10000000
  },function whenDone(err, uploadedFiles) {
    if (err) {
      return res.serverError(err);
    }

    // If no files were uploaded, respond with an error.
    if (uploadedFiles.length === 0){
      return res.badRequest('No file was uploaded');
    }

    // Get the base URL for our deployed application from our custom config
    // (e.g. this might be "http://foobar.example.com:1339" or "https://example.com")
    var baseUrl = sails.config.custom.baseUrl;

    // Save the "fd" and the url where the avatar for a user can be accessed
    User.update(req.session.userId, {

      // Generate a unique URL where the avatar can be downloaded.
      avatarUrl: require('util').format('%s/user/avatar/%s', baseUrl, req.session.userId),

      // Grab the first file and use it's `fd` (file descriptor)
      avatarFd: uploadedFiles[0].fd
    })
    .exec(function (err){
      if (err) return res.serverError(err);
      return res.ok();
    });
  });
},


/**
 * Download avatar of the user with the specified id
 *
 * (GET /user/avatar/:id)
 */
avatar: function (req, res){

  User.findOne(req.param('id')).exec(function (err, user){
    if (err) return res.serverError(err);
    if (!user) return res.notFound();

    // User has no avatar image uploaded.
    // (should have never have hit this endpoint and used the default image)
    if (!user.avatarFd) {
      return res.notFound();
    }

    var SkipperDisk = require('skipper-disk');
    var fileAdapter = SkipperDisk(/* optional opts */);

    // set the filename to the same file as the user uploaded
    res.set("Content-disposition", "attachment; filename='" + file.name + "'");

    // Stream the file down
    fileAdapter.read(user.avatarFd)
    .on('error', function (err){
      return res.serverError(err);
    })
    .pipe(res);
  });
}

//
// ...
```




#### 这些文件在哪里？

当使用默认的`receiver`时，文件上传将进入`myApp/.tmp/uploads/`目录。 你可以使用`dirname`选项覆盖设置。 请注意，在调用`.upload()`函数时以及在调用文件系统时，您都需要提供此选项（以便可以上传到同一位置并从同一位置下载）。


#### 上传到自定义文件夹

在上面的例子中，我们将文件上传到.tmp/uploads。 那么，我们如何配置一个自定义文件夹，比如'assets/images'。
我们可以通过添加选项来实现上传功能，如下所示。

```javascript
req.file('avatar').upload({
  dirname: require('path').resolve(sails.config.appPath, 'assets/images')
},function (err, uploadedFiles) {
  if (err) return res.serverError(err);

  return res.json({
    message: uploadedFiles.length + ' file(s) uploaded successfully!'
  });
});
```

### 文件上传同时发送文本字段

如上所述，您可以将文本字段（如“姓名”和“电子邮件”）与文件上传字段一起发送到Sails action。 但是，文本字段必须出现在文件字段_fields_之前才能处理它们。 这对Sails在文件上传时运行您的action代码（而不必等待它们完成）的性能至关重要。 有关更多信息，请参阅[Skipper文档](https://github.com/balderdashy/skipper#text-parameters)。


### 示例

#### 创建一个 `api`

首先，我们需要为服务/存储文件生成一个新的`api`。请使用sails命令行工具执行此操作。

```sh
$ sails generate api file

debug: Generated a new controller `file` at api/controllers/FileController.js!
debug: Generated a new model `File` at api/models/File.js!

info: REST API generated @ http://localhost:1337/file
info: and will be available the next time you run `sails lift`.
```

#### 编写控制器操作

我们做一个`index`动作来启动文件上传，和一个`upload`动作来接收文件。

```javascript

// myApp/api/controllers/FileController.js

module.exports = {

  index: function (req,res){

    res.writeHead(200, {'content-type': 'text/html'});
    res.end(
    '<form action="http://localhost:1337/file/upload" enctype="multipart/form-data" method="post">'+
    '<input type="text" name="title"><br>'+
    '<input type="file" name="avatar" multiple="multiple"><br>'+
    '<input type="submit" value="Upload">'+
    '</form>'
    )
  },
  upload: function  (req, res) {
    req.file('avatar').upload(function (err, files) {
      if (err)
        return res.serverError(err);

      return res.json({
        message: files.length + ' file(s) uploaded successfully!',
        files: files
      });
    });
  }

};
```

## 阅读更多

+ [Skipper docs](https://github.com/balderdashy/skipper)
+ [Uploading to Amazon S3](https://sailsjs.com/documentation/concepts/file-uploads/uploading-to-s-3)
+ [Uploading to Mongo GridFS](https://sailsjs.com/documentation/concepts/file-uploads/uploading-to-grid-fs)



<docmeta name="displayName" value="File uploads">
