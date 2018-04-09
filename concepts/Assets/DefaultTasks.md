# 默认任务

### 概述

Sails的asset管理通道是一组默认设置的Grunt任务，旨在是您的项目更加的一直和高效. 整个前端asset工作流程是可以定制的，他提供了一些开箱即用的默认任务。Sails可以更简单的[configure new tasks](https://sailsjs.com/documentation/concepts/assets/task-automation#?task-configuration)满足你的需求.


这里有一些Sails默认的Grunt配置可以帮助你:
- LESS自动编译
- JST自动编译
- Coffeescript自动编译
- 破坏缓存
- 可选资源自动注入, 压缩和连接
- 创建一个web预备公开目录
- 文件监视和同步
- 转换client-side JavaScript 使用 >=ES6 并保持浏览器兼容性
- 资源优化


### 默认Grunt tasks

下面是新Sails项目默认包含的Grunt任务列表:

##### clean

> 这个 grunt task任务清除 `.tmp/public/`中的内容.

> [usage docs](https://github.com/gruntjs/grunt-contrib-clean)

##### coffee

> 从 `assets/js/` 编译coffeeScript文件并放置到`.tmp/public/js/`目录.

> [usage docs](https://github.com/gruntjs/grunt-contrib-coffee)

##### hash

> 为需要清除缓存的文件添加一个随机hash.

> [usage docs](https://github.com/jgallen23/grunt-hash/tree/0.5.0#grunt-hash)

##### concat

> 连接JavaScript和css文件,并保存在`.tmp/public/concat/`目录.

> [usage docs](https://github.com/gruntjs/grunt-contrib-concat)

##### copy

> **dev task config**
> 除coffeescript和less之外的所有文件和目录,从sails assets文件夹放入`.tmp/public/`目录.

> **build task config**
> 将所有文件和目录.tmp/public directory放入www目录.

> [usage docs](https://github.com/gruntjs/grunt-contrib-copy)

##### cssmin

> 压缩css文件然后放入`.tmp/public/min/`目录.

> [usage docs](https://github.com/gruntjs/grunt-contrib-cssmin)

##### jst

> 预编译Underscore模板为 `.jst`文件. (i.e. it takes HTML template files and turns them into tiny JavaScript functions). 减少带宽使用并提高模板加载速度.

> [usage docs](https://github.com/gruntjs/grunt-contrib-jst)

##### less

> 编译LESS文件为CSS. 只有`assets/styles/importer.less`被编译. 这允许您自己控制排序，在其他样式表之前导入您的依赖关系，混入、变量、重置等。

> [usage docs](https://github.com/gruntjs/grunt-contrib-less)

##### sails-linker

> 为JavaCcript文件自动插入`<script>`标签和为css文件插入`<link>`标签. 还会使用`<script>`标签自动链接包含预编译模板的输出文件。 更详细的描述在这里 [here](https://github.com/balderdashy/sails-generate-frontend/blob/master/docs/overview.md#a-litte-bit-more-about-sails-linking), but the big takeaway is that script and stylesheet injection is *only* done in files containing `<!--SCRIPTS--><!--SCRIPTS END-->` and/or `<!--STYLES--><!--STYLES END-->` tags.  These are included in the default **views/layouts/layout.ejs** file in a new Sails project.  If you don't want to use the linker for your project, you can simply remove those tags.

> [usage docs](https://github.com/Zolmeister/grunt-sails-linker)

##### sync

> 让目录保持同步的Grunk任务。它与grunt-contrib-copy非常相似，但是只会尝试复制更改过的文件。它会将`assets/`文件夹中的文件同步到`.tmp/public /`，并覆盖已经存在的内容。

> [usage docs](https://github.com/tomusdrw/grunt-sync)

##### babel

> 这个grunt任务将任何>=ES6语法的Javascript文件编译为旧版以兼容浏览器.

> [usage docs](https://github.com/babel/grunt-babel)

##### uglify

> 压缩客户端JavaScript资源.  请注意，默认情况下，此任务将“改变”所有的函数和变量名称 (either by changing them to a much shorter name, or stripping them entirely).  This is usually desirable as it makes your code significantly smaller, but in some cases can lead to unexpected results (particularly when you expect an object's constructor to have a certain name).  To turn off or modify this behavior, [use the `mangle` option](https://www.npmjs.com/package/uglify-es#mangle-properties-options) when setting up this task.

> [usage docs](https://github.com/gruntjs/grunt-contrib-uglify/tree/harmony)

##### watch

> 运行一个监视任务，在添加修改或删除后执行。监视`assets/`文件夹的文件变化,并重新运行任务 (e.g. less and jst compilation).  This allows you to see changes to your assets reflected in your app without having to restart the Sails server.

> [usage docs](https://github.com/gruntjs/grunt-contrib-watch)


<docmeta name="displayName" value="Default tasks">
