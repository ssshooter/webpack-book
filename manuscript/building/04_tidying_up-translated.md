＃ 整理
# Tidying Up

当前安装程序不会清除构建之间的* build *目录。结果，它随着项目的变化不断累积文件。鉴于这可能会让人烦恼，你应该在两者之间进行清理。
The current setup doesn't clean the *build* directory between builds. As a result, it keeps on accumulating files as the project changes. Given this can get annoying, you should clean it up in between.

另一个不错的方法是将生成本身的构建信息包含在每个文件顶部的小注释中，至少包括版本信息。
Another nice touch would be to include information about the build itself to the generated bundles as a small comment at the top of each file including version information at least.

##清理构建目录
## Cleaning the Build Directory

可以通过使用webpack插件或在其外部解决此问题来解决此问题。您可以在npm脚本中触发`rm -rf ./build && webpack`或`rimraf ./build && webpack`以使其保持跨平台。任务运行员也可以为此目的而工作。
This issue can be resolved either by using a webpack plugin or solving it outside of it. You could trigger `rm -rf ./build && webpack` or `rimraf ./build && webpack` in an npm script to keep it cross-platform. A task runner could work for this purpose as well.

###设置`CleanWebpackPlugin`
### Setting Up `CleanWebpackPlugin`

首先安装[clean-webpack-plugin]（https://www.npmjs.com/package/clean-webpack-plugin）：
Install the [clean-webpack-plugin](https://www.npmjs.com/package/clean-webpack-plugin) first:

```bash
npm install clean-webpack-plugin --save-dev
```

{pagebreak}

接下来，您需要定义一个函数来包装基本思想。您可以直接使用该插件，但这感觉就像可以在项目中使用的东西，因此将其推送到库是有意义的：
Next, you need to define a function to wrap the basic idea. You could use the plugin directly, but this feels like something that could be used across projects, so it makes sense to push it to the library:

** ** webpack.parts.js
**webpack.parts.js**

```javascript
...
const CleanWebpackPlugin = require("clean-webpack-plugin");

exports.clean = path => ({
  plugins: [new CleanWebpackPlugin([path])],
});
```

将其与项目连接：
Connect it with the project:

** ** webpack.config.js
**webpack.config.js**

```javascript
const PATHS = {
  app: path.join(__dirname, "src"),
leanpub-start-insert
  build: path.join(__dirname, "dist"),
leanpub-end-insert
};

...

const productionConfig = merge([
leanpub-start-insert
  parts.clean(PATHS.build),
leanpub-end-insert
  ...
]);
```

在此更改之后，`build`目录在构建时应保持良好和整洁。您可以通过构建项目并确保输出目录中没有旧文件来验证这一点。
After this change, the `build` directory should remain nice and tidy while building. You can verify this by building the project and making sure no old files remained in the output directory.

{pagebreak}

##将修订附加到构建
## Attaching a Revision to the Build

将与当前构建版本相关的信息附加到构建文件本身可用于调试。 [webpack.BannerPlugin]（https://webpack.js.org/plugins/banner-plugin/）允许您实现此目的。它可以与[git-revision-webpack-plugin]（https://www.npmjs.com/package/git-revision-webpack-plugin）结合使用，在生成的文件的开头生成一个小注释。
Attaching information related to the current build revision to the build files themselves can be used for debugging. [webpack.BannerPlugin](https://webpack.js.org/plugins/banner-plugin/) allows you to achieve this. It can be used in combination with [git-revision-webpack-plugin](https://www.npmjs.com/package/git-revision-webpack-plugin) to generate a small comment at the beginning of the generated files.

###设置`BannerPlugin`和`GitRevisionPlugin`
### Setting Up `BannerPlugin` and `GitRevisionPlugin`

要开始，请安装修订插件：
To get started, install the revision plugin:

```bash
npm install git-revision-webpack-plugin --save-dev
```

然后定义一个部分来包装想法：
Then define a part to wrap the idea:

** ** webpack.parts.js
**webpack.parts.js**

```javascript
...
const webpack = require("webpack");
const GitRevisionPlugin = require("git-revision-webpack-plugin");

exports.attachRevision = () => ({
  plugins: [
    new webpack.BannerPlugin({
      banner: new GitRevisionPlugin().version(),
    }),
  ],
});
```

{pagebreak}

并将其连接到主配置：
And connect it to the main configuration:

** ** webpack.config.js
**webpack.config.js**

```javascript
const productionConfig = merge([
  ...
leanpub-start-insert
  parts.attachRevision(),
leanpub-end-insert
]);
```

如果你构建项目（`npm run build`），你应该注意到构建的文件包含像`/ *这样的注释！ 0b5bb05 * /`或`/ *！ v1.7.0-9-g5f82fe8 * /`开头。
If you build the project (`npm run build`), you should notice the built files contain comments like `/*! 0b5bb05 */` or `/*! v1.7.0-9-g5f82fe8 */` in the beginning.

可以通过调整横幅进一步自定义输出。您还可以使用`webpack.DefinePlugin`将修订信息传递给应用程序。在* Environment Variables *一章中详细讨论了这种技术。
The output can be customized further by adjusting the banner. You can also pass revision information to the application using `webpack.DefinePlugin`. This technique is discussed in detail in the *Environment Variables* chapter.

W> [插件在webpack 4中以生产模式中断]（https://github.com/webpack-contrib/uglifyjs-webpack-plugin/issues/222）！
W> [The plugin is broken in production mode in webpack 4](https://github.com/webpack-contrib/uglifyjs-webpack-plugin/issues/222)!

W>代码期望您在Git存储库中运行它！否则，你得到一个'致命的：不是git存储库（或任何父目录）：。git`错误。如果您不使用Git，则可以将横幅替换为其他数据。
W> The code expects you run it within a Git repository! Otherwise, you get a `fatal: Not a git repository (or any of the parent directories): .git` error. If you are not using Git, you can replace the banner with other data.

##复制文件
## Copying Files

复制文件是您可以使用webpack处理的另一个普通操作。 [copy-webpack-plugin]（https://www.npmjs.com/package/copy-webpack-plugin）如果你需要将外部数据带到你的构建中而不需要webpack直接指向它们，那么它们会很方便。
Copying files is another ordinary operation you can handle with webpack. [copy-webpack-plugin](https://www.npmjs.com/package/copy-webpack-plugin) can be handy if you need to bring external data to your build without having webpack pointing at them directly.

[cpy-cli]（https://www.npmjs.com/package/cpy-cli）是一个不错的选择，如果你想以跨平台的方式在webpack之外复制。根据定义，插件应该是跨平台的。
[cpy-cli](https://www.npmjs.com/package/cpy-cli) is a good option if you want to copy outside of webpack in a cross-platform way. Plugins should be cross-platforms by definition.

##结论
## Conclusion

通常，您通过识别问题然后找到解决问题的插件来处理webpack。在webpack之外解决这些类型的问题是完全可以接受的，但是webpack也可以经常处理它们。
Often, you work with webpack by identifying a problem and then finding a plugin to tackle it. It's entirely acceptable to solve these types of issues outside of webpack, but webpack can often handle them as well.

回顾一下：
To recap:

*您可以找到许多可用作任务的小插件，并将webpack推向任务运行器。
* You can find many small plugins that work as tasks and push webpack closer to a task runner.
*这些任务包括清理构建和部署。 *部署应用程序*章节详细讨论了后一主题。
* These tasks include cleaning the build and deployment. The *Deploying Applications* chapter discusses the latter topic in detail.
*向生产版本添加小注释以告知已部署的版本是一个好主意。这样您就可以更快地调试潜在问题。
* It can be a good idea to add small comments to the production build to tell what version has been deployed. This way you can debug potential issues faster.
*这些辅助任务可以在webpack之外执行。如果您使用*多页*章节中讨论的多页设置，则必须使用此功能。
* Secondary tasks like these can be performed outside of webpack. If you are using a multi-page setup as discussed in the *Multiple Pages* chapter, this becomes a necessity.

