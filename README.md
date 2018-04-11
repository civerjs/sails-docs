![Squiddy reads the docs](https://sailsjs.com/images/squidford_swimming.png)

# Sails.js 中文文档

Sails当前稳定版本的官方文档位于这里[master branch]（github.com/balderdashy/sails-docs）。 [官方Sails网站]（https://sailsjs.com）



> 我们现在使用branches来跟踪不同版本的Sails文档。 在开始一个新的翻译项目之前，请查看下面的[更新信息]（＃how-can-i-help-translate-the-documentation）。



## 贡献Sails文档

欢迎您的帮助！ 请发送pull request给** master **，并加以修改/添加，我们会尽快对它进行检查和合并。

我们愿意接受，如何改善管理文档的流程以及与社区合作的建议。 请将您的想法发布到Google Group，或者如果您有兴趣直接提供帮助，请在Twitter上联系@fancydoilies，@rudeboot或@mikermcneil。

#### 我应该编辑哪个分支？

<!-- As we get closer to releasing a newer version of Sails, we ask that all pull requests be made to the `1.0` branch, since that content will soon replace the 0.12 docs on the main website. The only exception is if you are documenting something that isn't relevant for Sails v1. -->

要编辑与最新的Sails稳定版（即[NPM](npmjs.org/package/sails)上的版本）的文档，您可以编辑本分支 请参阅sails-docs pull request）。 Sails核心团队将把它合并到最新的Sails稳定版本的相应分支中，大约一周时间将其部署到sailsjs.com。

<!-- That depends on what kind of edit you are making.  Most often, you'll be making an edit that is relevant for the latest stable version of Sails (i.e. the version on [NPM](npmjs.org/package/sails)) and so you'll want to edit the `master` branch of _this_ repo (what you see in the sails-docs repo by default).  The docs team merges master into the appropriate branch for the latest stable release of Sails, and then deploys that to sailsjs.com about once per week.

另外，如果您编辑即将发布的版本中未发布的功能， 比如作为一个功能建议或对Sails或相关项目的开放式请求，那么您需要编辑Sails的下一个未发布版本（有时称为“边缘”）的分支。
 -->

| Branch (in `sails-docs`)                                          | Documentation for Sails Version...                                                     | Accessible At...   |
|:------------------------------------------------------------------|:---------------------------------------------------------------------------------------|:-------------------|
| [`master`](https://github.com/balderdashy/sails-docs/tree/master) | _Bleeding edge_                                                                        | [`next.sailsjs.com`](https://next.sailsjs.com)
| [`1.0`](https://github.com/balderdashy/sails-docs/tree/1.0)       | [![NPM version](https://badge.fury.io/js/sails.png)](http://badge.fury.io/js/sails)    | [`sailsjs.com`](https://sailsjs.com)
| [`0.12`](https://github.com/balderdashy/sails-docs/tree/0.12)     | Sails v0.12.x                                                                          | [`0.12.sailsjs.com`](https://0.12.sailsjs.com)
| [`0.11`](https://github.com/balderdashy/sails-docs/tree/0.11)     | Sails v0.11.x                                                                          | [`0.11.sailsjs.com`](http://0.11.sailsjs.com)


#### 这些文档如何编译并推送到网站？

我们使用`doc-templater`模块将.md文件转换为网站的html。 您可以在 [doc-templater repo](https://github.com/uncletammy/doc-templater)中了解更多。

每个.md文件在网站上都有自己的页面（即所有的参考，概念和解析文件），并且应该包含一个特殊的<docmeta name =“displayName”>“标签，其中带有`value`属性，用于指定当前页。 这会影响文档页面在搜索引擎结果中的显示方式，并且还会在sailsjs.com上的导航菜单中作为显示名称。 例如：

```markdown
<docmeta name="displayName" value="Building Custom Homemade Puddings">
```


#### 我如何帮助翻译文档？

这是帮助Sails项目发展的一个好事，如果你自己说英语以外的语言，并自愿翻译Sails文档。 请使用该分支的自述文件中的说明与翻译项目的维护人员联系。

如果您的语言未在上表中列出，并且您有兴趣开始翻译项目，请按以下步骤操作：

+ Fork this repo (`balderdashy/sails-docs`) and change the name of your fork to be `sails-docs-{{IETF}}` where {{IETF}} is the [IETF language tag](https://en.wikipedia.org/wiki/IETF_language_tag) for your language.
+ Edit the README to summarize your progress so far, provide any other information you think would be helpful for others reading your translation, and let interested contributors know how to contact you.
+ Send a pull request editing the table above to add a link to your fork.
+ When you are satisfied with the first complete version of your translation, open an issue and someone from our docs team will be happy to help you get preview it in the context of the Sails website, get it live on a domain (yours, or a subdomain of sailsjs.com, whichever makes the most sense), and share it with the rest of the Sails community.


#### 我还能怎么帮忙？

有关贡献Sails的更多信息，请参阅[贡献指南]（sailsjs.com/contributing）。


## License

[MIT](./LICENSE.md)

The [Sails framework](https://sailsjs.com) is free and open-source under the [MIT License](https://sailsjs.com/license).

_(And the files in this repo are too.)_

