# 上传到Mongo GridFS

由于Mongo的GridFS文件系统特点，可以将文件上传到MongoDB。 使用Sails，你可以使用[MongoDB的GridFS]的Skipper适配器(https://github.com/willhuang85/skipper-gridfs)，使用很少的附加配置来完成此任务。

安装:

```sh
$ npm install skipper-gridfs --save
```

然后在一个控制器中使用它:

```javascript
  uploadFile: function (req, res) {
    req.file('avatar').upload({
      adapter: require('skipper-gridfs'),
      uri: 'mongodb://[username:password@]host1[:port1][/[database[.bucket]]'
    }, function (err, filesUploaded) {
      if (err) return res.serverError(err);
      return res.ok();
    });
  }
```

<docmeta name="displayName" value="Uploading to GridFS">
