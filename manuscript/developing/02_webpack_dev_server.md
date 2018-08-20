# webpack-dev-server

[LiveReload](http://livereload.com/) 或 [Browsersync](http://www.browsersync.io/) 等工具允许在开发应用程序时自动刷新浏览器，避免 CSS 更改时整页刷新。webpack 可以通过 [browser-sync-webpack-plugin](https://www.npmjs.com/package/browser-sync-webpack-plugin) 设置 Browsersync，但还有更多其他方法。

## Webpack `watch` Mode 和 *webpack-dev-server*

提高开发体验的第一步是开启 webpack **watch** 模式。你可以通过将 `--watch` 传递给 webpack 来激活它。示例：`npm run build -- --watch`。

启用后，监视模式会检测对文件是否更改，并自动重新编译。*webpack-dev-server*（WDS）进一步加强了 watch 模式。

WDS 是一个运行于**内存**的开发服务器，这意味着 bundle 内容不会写入文件而会存储在内存中。在调试代码和样式时，在速度上区别显著。

默认情况下，WDS 会在你开发应用程序时**自动**在浏览器中刷新内容。同时，它也支持 webpack 的高级特性，**热模块替换**（HMR）。

HMR 可以在不完全刷新的情况下 patch 应用。因为页面的刷新会破坏应用程序状态，所以这个特性使 React 这样的库特别方便。*Hot Module Replacement* 附录详细介绍了该功能。

WDS 提供了一个可以动态 patch 代码的接口，但你必须在代码实现此接口。这个接口对 JavaScript 框架和库很实用，但是因为 CSS 是无状态的，所以作用不大。

## WDS 生成文件

为了达到最高性能，WDS 默认运行在内存中，但如果你要集成其他服务器，可能就需要生成文件了。[write-file-webpack-plugin](https://www.npmjs.com/package/write-file-webpack-plugin) 可以实现此功能。

W> WDS 应只用与开发。如果要发布应用程序，请考虑其他标准解决方案，例如 Apache 或 Nginx。

## 开始使用 WDS

安装 WDS：

```bash
npm install webpack-dev-server --save-dev
```

和之前一样，这个命令在 `npm bin` 目录下面生成一个命令，你可以从那里运行 **webpack-dev-server**。运行 WDS 后，你有一个运行在 `http://localhost:8080` 的开发服务器。现在，最基本的浏览器自动刷新功能已经具备。

{pagebreak}

## 将 WDS 添加到项目

将 WDS 集成到项目中，定义一个 npm 启动脚本。一般我们将其命名为 *start*：

**package.json**

```json
"scripts": {
leanpub-start-insert
  "start": "webpack-dev-server --mode development",
leanpub-end-insert
  "build": "webpack --mode production"
},
```

T> WDS 使用 webpack 本身的配置，适用相同的规则。

现在执行 **npm run start** 或 **npm start**，你可以在终端中看到：

```bash
> webpack-dev-server --mode development

ℹ ｢wds｣: Project is running at http://localhost:8080/
ℹ ｢wds｣: webpack output is served from /
ℹ ｢wdm｣: Hash: eb06816060088d633767
Version: webpack 4.1.1
Time: 608ms
Built at: 3/16/2018 3:44:04 PM
     Asset       Size  Chunks                    Chunk Names
   main.js    338 KiB    main  [emitted]  [big]  main
index.html  181 bytes          [emitted]
Entrypoint main [big] = main.js
...
```

{pagebreak}

服务器运行中，在浏览器中打开 `http://localhost:8080/`，你应该能看到你的网页：

![Hello world](images/hello_01.png)

修改代码后，在终端能看到新的输出，浏览器还应该对更改执行整页刷新。

T> WDS 在默认端口被使用的情况下会尝试在另一个端口运行。终端的输出会告诉你的它运行端口。你可以使用 `netstat -na | grep 8080` 命令查看端口情况，如果端口 8080 上正在运行某些东西，应该会有对应的信息。

T> 除了 `production` 和 `development`，还有第三种模式 `none`，它禁用所有内容，这类似于 webpack 4 之前版本中的行为。

## 通过 Webpack 配置设定 WDS

可以通过 webpack 配置中的 `devServer` 字段自定义 WDS 功能。你也可以通过 CLI 设置大多数选项，但通过 webpack 管理配置显然更优雅。

{pagebreak}

配置方法如下所示：

**webpack.config.js**

```javascript
...

module.exports = {
leanpub-start-insert
  devServer: {
    // Display only errors to reduce the amount of output.
    stats: "errors-only",

    // Parse host and port from env to allow customization.
    //
    // If you use Docker, Vagrant or Cloud9, set
    // host: options.host || "0.0.0.0";
    //
    // 0.0.0.0 is available to all network devices
    // unlike default `localhost`.
    host: process.env.HOST, // Defaults to `localhost`
    port: process.env.PORT, // Defaults to 8080
    open: true, // Open the page in browser
  },
leanpub-end-insert
  ...
};
```

完成此更改后，你可以通过环境参数配置服务器 host 和 port 选项（例如：`PORT=3000 npm start`）。

T> [dotenv](https://www.npmjs.com/package/dotenv) 允许你通过 *.env* 文件定义环境变量。这样可以快速控制和设置的 host 和 port。

T> 如果你使用基于 HTML5 History API 的路由，请启用 `devServer.historyApiFallback`。

## 启用错误覆盖

WDS 提供错误覆盖，可以直接在网页显示捕获到的警告和错误：

**webpack.config.js**

```javascript
module.exports = {
  devServer: {
    ...
leanpub-start-insert
    overlay: true,
leanpub-end-insert
  },
  ...
};
```

运行服务器（`npm start`），随便写些错误代码，再看看浏览器：

![Error overlay](images/error-overlay.png)

T> 可以考虑使用 [error-overlay-webpack-plugin](https://www.npmjs.com/package/error-overlay-webpack-plugin)，它更好地显示了错误的来源。

W> WDS 错误覆盖**不**显示应用程序的运行时错误。

## 启用热模块更换

热模块替换是 webpack 与众不同的功能之一。实现它需要在服务器端和客户端上进行额外的配置。*Hot Module Replacement* 附录更详细地讨论了该主题。如果要将 HMR 集成到项目中，可以了解一下。不过完成本教程不需要它。

## 从网络访问开发服务器

可以自定义主机和端口设置（Unix上的 `export PORT=3000`，Windows上的 `SET PORT=3000`）。在大多数平台上，默认设置已足够。

要访问你的服务器，需要知道你的 IP。在Unix上，这可以使用 `ifconfig | grep inet`。在Windows上，可以使用 `ipconfig`。 npm 包，例如 [node-ip](https://www.npmjs.com/package/node-ip) 也能派上用场。在 Windows 上，需要 `HOST` 匹配你的 IP 才能正常访问。

{pagebreak}

## 快速开发配置

在修改程序代码时，WDS 会自动重新启动服务器，但是当你编辑 webpack 配置时呢？每次进行更改都要重启开发服务器就很让人纳闷了。该过程可以通过 [nodemon](https://www.npmjs.com/package/nodemon) 自动执行，请看 [GitHub 中讨论](https://github.com/webpack/webpack-dev-server/issues/440#issuecomment-205757892)。

首先安装 `npm install nodemon --save-dev`，然后写以下脚本，你就可以在 webpack 配置更新后自动重启 WDS 了：

**package.json**

```json
"scripts": {
  "start": "nodemon --watch webpack.config.js --exec \"webpack-dev-server --mode development\"",
  "build": "webpack --mode production"
},
```

另外，未来 WDS 可能会[自带此功能](https://github.com/webpack/webpack-cli/issues/15)。

{pagebreak}

## 使用轮询而不是监视文件变化

有时，WDS 提供的文件监视无法在你的系统上运行。在旧版本的 Windows，Ubuntu，Vagrant 和 Docker 上可能会出现问题。启用轮询是一个很好的选择：

**webpack.config.js**

```javascript
const path = require("path");
const webpack = require("webpack");

module.exports = {
  devServer: {
    watchOptions: {
      // Delay the rebuild after the first change
      aggregateTimeout: 300,

      // Poll using interval (in ms, accepts boolean too)
      poll: 1000,
    },
  },
  plugins: [
    // Ignore node_modules so CPU usage with poll
    // watching drops significantly.
    new webpack.WatchIgnorePlugin([
      path.join(__dirname, "node_modules")
    ]),
  ],
};
```

这样比默认设置更耗费资源，但某些情况下值得一用。

{pagebreak}

## *webpack-dev-server* 的替代工具

你可以设置 Express 服务器并使用中间件。这里有几种选择：

* [官方 WDS 中间件](https://webpack.js.org/guides/development/#using-webpack-dev-middleware)
* [webpack-hot-middleware](https://www.npmjs.com/package/webpack-hot-middleware)
* [webpack-isomorphic-dev-middleware](https://www.npmjs.com/package/webpack-isomorphic-dev-middleware)

如果你需要更灵活的控制，webpack 还提供一个 [Node API](https://webpack.js.org/configuration/dev-server/)。

W> CLI 和 Node API [略有差异](https://github.com/webpack/webpack-dev-server/issues/616)。

## *webpack-dev-server* 的其他特性

WDS 以下功能超出本文介绍范围，但你仍然可以了解以下：

* `devServer.contentBase`--假设你不想动态生成 *index.html* 而是自己在特定目录中维护它，你需要将 WDS 指向它。`contentBase` 接受路径（例如，`"build"`）或路径数组（例如，`["build", "images"]`）。默认值为项目根目录。
* `devServer.proxy`--如果你使用多个服务器，你需要把 WDS 代理到他们。代理设置接受一个代理映射对象（例如, `{ "/api": "http://localhost:3000/api" }`），这个对象匹配到对应路径会请求转发到其他服务器。代理设定默认不启用。
* `devServer.headers`--你的请求添加自定义 header。

T> [官方文档](https://webpack.js.org/configuration/dev-server/)涵盖更多选项。

## 开发插件

webpack 插件生态系统是多种多样的，这些插件都有针对性地有助于某些开发情况：

* [case-sensitive-paths-webpack-plugin](https://www.npmjs.com/package/case-sensitive-paths-webpack-plugin) 可以方便你在大小写不敏感的环境（如 macOS 或 Windows）强制大小写敏感（便于和 Linux 统一）。
* [npm-install-webpack-plugin](https://www.npmjs.com/package/npm-install-webpack-plugin) 可以在你 import 软件包的时候自动安装或链接已经在 **package.json** 安装的软件包。
* [react-dev-utils](https://www.npmjs.com/package/react-dev-utils) 包含了为 [Create React App](https://www.npmjs.com/package/create-react-app) 开发的 webpack 实用程序。虽然名字带 React，但也可以用于 React 之外。如果你只想要 webpack 消息格式化，请考虑[webpack-format-messages](https://www.npmjs.com/package/webpack-format-messages)。
* [start-server-webpack-plugin](https://www.npmjs.com/package/start-server-webpack-plugin) 能够在 webpack 构建完成后启动服务器。
{pagebreak}

## 关于输出的插件

还有一些插件可以让 webpack 的终端输出更容易被注意和理解：

* [system-bell-webpack-plugin](https://www.npmjs.com/package/system-bell-webpack-plugin) 在遇到错误时调用系统级警报，而​​不是让 webpack 静默失败。
* [webpack-notifier](https://www.npmjs.com/package/webpack-notifier) 使用系统通知让你了解 webpack 状态。
* [nyan-progress-webpack-plugin](https://www.npmjs.com/package/nyan-progress-webpack-plugin) 可用于在构建过程中获得更整洁的输出。但是如果你使用像 Travis 等持续集成（CI）系统，可能会破坏输出。 Webpack 的 `ProgressPlugin` 提供相似功能。不过没有 nyan 哦。
* [friendly-errors-webpack-plugin](https://www.npmjs.com/package/friendly-errors-webpack-plugin) 改进了 webpack 的错误报告，使常见错误并以更友好的方式显示。
* [webpack-dashboard](https://www.npmjs.com/package/webpack-dashboard) 在标准 webpack 输出上，提供了一个完整的基于终端的仪表板。如果你喜欢清晰明白的图形化输出，它就能派上用场了。

{pagebreak}

## 总结

WDS 补充了 webpack，并提供了“面向开发”的功能，对开发人员更友好。

回顾一下：

* Webpack 的 `watch` 模式是提高开发体验的第一步。你在编辑源代码后可以自动更新 webpack 编译包。
* WDS 可以在代码更改时自动刷新浏览器。同时还可以实现**热模块替换**。
* 默认 WDS 设置在特定系统上可能存在问题。因此，轮询是另一种选择。
* WDS 可以使用中间件集成到现有的 Node 服务器中。这样做比 CLI 更容易控制。
* WDS 远不止自动刷新和 HMR。例如，使用代理连接到其他服务器。

在下一章中，你将学习如何更优雅地写配置，这是进一步开发的必备知识。

