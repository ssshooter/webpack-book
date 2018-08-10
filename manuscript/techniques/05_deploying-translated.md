＃部署应用程序
# Deploying Applications

使用webpack构建的项目可以部署到各种环境中。可以使用* gh-pages *包将不依赖后端的公共项目推送到GitHub页面。此外，还有各种webpack插件可以针对其他环境，例如S3。
A project built with webpack can be deployed to a variety of environments. A public project that doesn't rely on a backend can be pushed to GitHub Pages using the *gh-pages* package. Also, there are a variety of webpack plugins that can target other environments, such as S3.

##使用* gh-pages进行部署*
## Deploying with *gh-pages*

[gh-pages]（https://www.npmjs.com/package/gh-pages）允许你轻松地在GitHub页面上托管独立应用程序。首先必须指向构建目录。它拾取内容并将它们推送到`gh-pages`分支。
[gh-pages](https://www.npmjs.com/package/gh-pages) allows you to host stand-alone applications on GitHub Pages easily. It has to be pointed to a build directory first. It picks up the contents and pushes them to the `gh-pages` branch.

尽管它的名称，该包也可以与支持Git存储库托管的其他服务一起使用。但鉴于GitHub如此受欢迎，它可以用来证明这个想法。在实践中，你可能会有更复杂的设置，通过持续集成系统将结果推送到另一个服务。
Despite its name, the package works with other services that support hosting from a Git repository as well. But given GitHub is so popular, it can be used to demonstrate the idea. In practice, you would likely have more complicated setup in place that would push the result to another service through a Continuous Integration system.

###设置* gh-pages *
### Setting Up *gh-pages*

要开始，请执行
To get started, execute

```bash
npm install gh-pages --save-dev
```

{pagebreak}

你还需要* package.json *中的脚本：
You are also going to need a script in *package.json*:

** **的package.json
**package.json**

```json
"scripts": {
leanpub-start-insert
  "deploy": "gh-pages -d build",
leanpub-end-insert
  ...
},
```

要使资产路径在GitHub页面上运行，必须调整`output.publicPath`字段。否则，资产路径最终指向根，除非你直接在域根（例如“survivaljs.com`”）之后托管，否则这不起作用。
To make the asset paths work on GitHub Pages, `output.publicPath` field has to be adjusted. Otherwise, the asset paths end up pointing at the root, and that doesn't work unless you are hosting behind a domain root (say `survivejs.com`) directly.

例如，`publicPath`控制你在* index.html *看到的结果url。如果你在CDN上托管资产，这将是调整的地方。
`publicPath` gives control over the resulting urls you see at *index.html* for instance. If you are hosting your assets on a CDN, this would be the place to tweak.

在这种情况下，将其设置为指向GitHub项目就足够了，如下所示：
In this case, it's enough to set it to point the GitHub project as below:

** ** webpack.config.js
**webpack.config.js**

```javascript
const productionConfig = merge([
  {
    ...
    output: {
      ...
leanpub-start-delete
      // Needed for code splitting to work in nested paths
      publicPath: "/",
leanpub-end-delete
leanpub-start-insert
      // Tweak this to match your GitHub project name
      publicPath: "/webpack-demo/",
leanpub-end-insert
    },
  },
  ...
]);
```

在构建（`npm run build`）和部署（`npm run deploy`）之后，你应该从GitHub Pages上托管的`build /`目录中获得你的应用程序。你应该在`https：// <name> .github.io / <project>`找到它，假设一切正常。
After building (`npm run build`) and deploying (`npm run deploy`), you should have your application from the `build/` directory hosted on GitHub Pages. You should find it at `https://<name>.github.io/<project>` assuming everything went fine.

T>如果你需要更精细的设置，请使用* gh-pages *提供的Node API。但是，它提供的默认命令行工具足以满足基本目的。
T> If you need a more elaborate setup, use the Node API that *gh-pages* provides. The default command line tool it gives is enough for essential purposes, though.

T> GitHub Pages允许你选择部署的分支。即使对于不需要捆绑的最小站点来说，它也可以使用`master`分支。你还可以指向“master”分支中的*。/ docs *目录下方并维护你的站点。
T> GitHub Pages allows you to choose the branch where you deploy. It's possible to use the `master` branch even as it's enough for minimal sites that don't need bundling. You can also point below the *./docs* directory within your `master` branch and maintain your site.

###存档旧版本
### Archiving Old Versions

* gh-pages *为存档目的提供了`add`选项。这个想法如下：
*gh-pages* provides an `add` option for archival purposes. The idea goes as follows:

1.将旧版本的站点复制到临时目录中，并从中删除* archive *目录。你可以根据需要命名存档目录。
1. Copy the old version of the site in a temporary directory and remove *archive* directory from it. You can name the archival directory as you want.
2.清理并构建项目。
2. Clean and build the project.
3.复制* build / archive / <version> *下面的旧版本*
3. Copy the old version below *build/archive/<version>*
4.设置脚本以通过Node调用* gh-pages *，如下所示，并捕获回调中可能的错误：
4. Set up a script to call *gh-pages* through Node as below and capture possible errors in the callback:

```javascript
ghpages.publish(path.join(__dirname, "build"), { add: true }, cb);
```

##部署到其他环境
## Deploying to Other Environments

即使你可以将部署问题推到webpack之外，但有一些特定于webpack的实用程序可以派上用场：
Even though you can push the problem of deployment outside of webpack, there are a couple of webpack specific utilities that come in handy:

* [webpack-deploy]（https://www.npmjs.com/package/webpack-deploy）是部署实用程序的集合，甚至可以在webpack之外工作。
* [webpack-deploy](https://www.npmjs.com/package/webpack-deploy) is a collection of deployment utilities and works even outside of webpack.
* [webpack-s3-sync-plugin]（https://www.npmjs.com/package/webpack-s3-sync-plugin）和[webpack-s3-plugin]（https://www.npmjs.com/ package / webpack-s3-plugin）将资产同步到亚马逊。
* [webpack-s3-sync-plugin](https://www.npmjs.com/package/webpack-s3-sync-plugin) and [webpack-s3-plugin](https://www.npmjs.com/package/webpack-s3-plugin) sync the assets to Amazon.
* [ssh-webpack-plugin]（https://www.npmjs.com/package/ssh-webpack-plugin）专为通过SSH部署而设计。
* [ssh-webpack-plugin](https://www.npmjs.com/package/ssh-webpack-plugin) has been designed for deployments over SSH.
* [now-loader]（https://www.npmjs.com/package/now-loader）在资源级别运行，允许你将特定资源部署到Now托管服务。
* [now-loader](https://www.npmjs.com/package/now-loader) operates on resource level and allows you to deploy specific resources to Now hosting service.

T>要访问生成的文件及其路径，请考虑使用[assets-webpack-plugin]（https://www.npmjs.com/package/assets-webpack-plugin）。路径信息允许你在部署时将webpack与其他环境集成。
T> To get access to the generated files and their paths, consider using [assets-webpack-plugin](https://www.npmjs.com/package/assets-webpack-plugin). The path information allows you to integrate webpack with other environments while deploying.

W>为确保在部署新版本后依赖旧版捆绑包的客户端仍然有效，请不要删除旧文件，直到它们足够大。你可以对部署时要删除的内容执行特定检查，而不是删除每个旧资产。
W> To make sure clients relying on the older bundles still work after deploying a new version, do **not** remove the old files until they are old enough. You can perform a specific check on what to remove when deploying instead of removing every old asset.

##动态解析`output.publicPath`
## Resolving `output.publicPath` Dynamically

如果你事先不知道`publicPath`，则可以通过以下步骤根据环境解决它：
If you don't know `publicPath` beforehand, it's possible to resolve it based on the environment by following these steps:

1.在应用程序入口点设置`__webpack_public_path__ = window.myDynamicPublicPath;`，并根据需要解决它。
1. Set `__webpack_public_path__ = window.myDynamicPublicPath;` in the application entry point and resolve it as you see fit.
2.从webpack配置中删除`output.publicPath`设置。
2. Remove `output.publicPath` setting from your webpack configuration.
3.如果你正在使用ESLint，请将其设置为通过`globals .__ webpack_public_path__：true`忽略全局。
3. If you are using ESLint, set it to ignore the global through `globals.__webpack_public_path__: true`.

编译时，webpack选择`__webpack_public_path__`并重写它，使其指向webpack逻辑。
When you compile, webpack picks up `__webpack_public_path__` and rewrites it so that it points to webpack logic.

##结论
## Conclusion

即使webpack不是部署工具，你也可以找到它的插件。
Even though webpack isn't a deployment tool, you can find plugins for it.

回顾一下：
To recap:

*可以处理webpack之外的部署问题。例如，你可以在npm脚本中实现此目的。
* It's possible to handle the problem of deployment outside of webpack. You can achieve this in an npm script for example.
*你可以动态配置webpack的`output.publicPath`。如果你不知道编译时并希望稍后决定，这种技术很有用。这可以通过`__webpack_public_path__` global来实现。
* You can configure webpack's `output.publicPath` dynamically. This technique is valuable if you don't know it compile-time and want to decide it later. This is possible through the `__webpack_public_path__ ` global.

