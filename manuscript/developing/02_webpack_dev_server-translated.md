# webpack-dev-server

[LiveReload](http://livereload.com/) 或 [Browsersync](http://www.browsersync.io/) 等工具允许在开发应用程序时自动刷新浏览器，避免 CSS 更改时整页刷新。webpack 可以通过 [browser-sync-webpack-plugin](https://www.npmjs.com/package/browser-sync-webpack-plugin) 设置Browsersync，但还有更多其他方法。

## Webpack `watch` Mode 和 *webpack-dev-server*

迈向更好的开发环境的第一步是在** watch **模式下使用webpack。你可以通过将`--watch`传递给webpack来激活它。示例：`npm run build  -  --watch`。
A good first step towards a better development environment is to use webpack in its **watch** mode. You can activate it by passing the `--watch` to webpack. Example: `npm run build -- --watch`.

启用后，监视模式会检测对文件所做的更改并自动重新编译。 * webpack-dev-server *（WDS）实现了监视模式，甚至更进一步。
Once enabled, the watch mode detects changes made to your files and recompiles automatically. *webpack-dev-server* (WDS) implements a watch mode and goes even further.

WDS是一个运行** in-memory **的开发服务器，这意味着bundle内容不会写入文件但会存储在内存中。在尝试调试代码和样式时，区别很重要。
WDS is a development server running **in-memory**, meaning the bundle contents aren't written out to files but stored in memory. The distinction is important when trying to debug code and styles.

默认情况下，WDS会在你开发应用程序时自动在浏览器中刷新内容，因此你无需亲自执行此操作。但它也支持高级webpack功能，**热模块替换**（HMR）。
By default, WDS refreshes content automatically in the browser while you develop your application, so you don't have to do it yourself. However it also supports an advanced webpack feature, **Hot Module Replacement** (HMR).

HMR允许在没有完全刷新的情况下修补浏览器状态，这使得像React这样的库特别方便，其中更新会破坏应用程序状态。 *热模块更换*附录详细介绍了该功能。
HMR allows patching the browser state without a full refresh making it particularly handy with libraries like React where an update blows away the application state. The *Hot Module Replacement* appendix covers the feature in detail.

WDS提供了一个可以动态修补代码的接口，但为了有效地工作，你必须为客户端代码实现此接口。对CSS之类的东西来说这是微不足道的，因为它是无状态的，但JavaScript框架和库的问题更难。
WDS provides an interface that makes it possible to patch code on the fly, however for this to work efficiently you have to implement this interface for the client-side code. It's trivial for something like CSS because it's stateless, but the problem is harder with JavaScript frameworks and libraries.

##从WDS发送文件
## Emitting Files from WDS

尽管出于性能原因默认WDS在内存中运行是好的，但有时将文件发送到文件系统会很好。如果要与期望查找文件的其他服务器集成，则这变得至关重要。 [write-file-webpack-plugin]（https://www.npmjs.com/package/write-file-webpack-plugin）允许你这样做。
Even though it's good that WDS operates in-memory by default for performance reasons, sometimes it can be good to emit files to the file system. If you are integrating with another server that expects to find the files, this becomes essential. [write-file-webpack-plugin](https://www.npmjs.com/package/write-file-webpack-plugin) allows you to do this.

W>你应该严格使用WDS进行开发。如果要托管应用程序，请考虑其他标准解决方案，例如Apache或Nginx。
W> You should use WDS strictly for development. If you want to host your application, consider other standard solutions, such as Apache or Nginx.

## WDS入门
## Getting Started with WDS

要开始使用WDS，请先安装它：
To get started with WDS, install it first:

```bash
npm install webpack-dev-server --save-dev
```

和以前一样，这个命令在`npm bin`目录下面生成一个命令，你可以从那里运行* webpack-dev-server *。运行WDS后，你有一个运行在`http：// localhost：8080`的开发服务器。现在，自动浏览器刷新已经到位，但基本水平。
As before, this command generates a command below the `npm bin` directory, and you could run *webpack-dev-server* from there. After running the WDS, you have a development server running at `http://localhost:8080`. Automatic browser refresh is in place now, although at a fundamental level.

{pagebreak}

##将WDS附加到项目
## Attaching WDS to the Project

要将WDS集成到项目中，请定义用于启动它的npm脚本。要遵循npm约定，请将其称为* start *，如下所示：
To integrate WDS to the project, define an npm script for launching it. To follow npm conventions, call it as *start* like below:

** **的package.json
**package.json**

```json
"scripts": {
leanpub-start-insert
  "start": "webpack-dev-server --mode development",
leanpub-end-insert
  "build": "webpack --mode production"
},
```

T> WDS选择webpack本身的配置。适用相同的规则。
T> WDS picks up configuration like webpack itself. The same rules apply.

如果你执行* npm run start *或* npm start * now，你应该在终端中看到一些内容：
If you execute either *npm run start* or *npm start* now, you should see something in the terminal:

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

服务器正在运行，如果你在浏览器中打开`http：// localhost：8080 /`，你应该看到熟悉的东西：
The server is running, and if you open `http://localhost:8080/` at your browser, you should see something familiar:

![Hello world](images/hello_01.png)

如果你尝试修改代码，你应该在终端中看到输出。浏览器还应该对更改执行硬刷新。
If you try modifying the code, you should see the output in your terminal. The browser should also perform a hard refresh on change.

T> WDS尝试在另一个端口运行，以防使用默认端口。终端输出告诉你它最终运行的位置。你可以使用`netstat -na |这样的命令调试情况grep 8080`。如果端口8080上正在运行某些东西，它应该在Unix上显示一条消息。
T> WDS tries to run in another port in case the default one is being used. The terminal output tells you where it ends up running. You can debug the situation with a command like `netstat -na | grep 8080`. If something is running on the port 8080, it should display a message on Unix.

T>除了`production`和`development`之外，还有第三种模式，`none`，它禁用所有内容，并且接近webpack 4之前版本中的行为。
T> In addition to `production` and `development`, there's a third mode, `none`, which disables everything and is close to the behavior you had in versions before webpack 4.

##通过Webpack配置配置WDS
## Configuring WDS Through Webpack Configuration

可以通过webpack配置中的`devServer`字段自定义WDS功能。你也可以通过CLI设置大多数这些选项，但通过webpack管理它们是一种不错的方法。
WDS functionality can be customized through the `devServer` field in the webpack configuration. You can set most of these options through the CLI as well, but managing them through webpack is a decent approach.

{pagebreak}

启用其他功能，如下所示：
Enable additional functionality as below:

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

完成此更改后，你可以通过环境参数配置服务器主机和端口选项（例如：`PORT = 3000 npm start`）。
After this change, you can configure the server host and port options through environment parameters (example: `PORT=3000 npm start`).

T> [dotenv]（https://www.npmjs.com/package/dotenv）允许你通过* .env *文件定义环境变量。 * dotenv *允许你快速控制设置的主机和端口设置。
T> [dotenv](https://www.npmjs.com/package/dotenv) allows you to define environment variables through a *.env* file. *dotenv* allows you to control the host and port setting of the setup quickly.

如果你使用基于HTML5历史记录API的路由，请启用`devServer.historyApiFallback`。
T> Enable `devServer.historyApiFallback` if you are using HTML5 History API based routing.

##启用错误覆盖
## Enabling Error Overlay

WDS提供了一个覆盖，用于捕获与编译相关的警告和错误：
WDS provides an overlay for capturing compilation related warnings and errors:

** ** webpack.config.js
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

立即运行服务器（`npm start`）并中断代码以在浏览器中查看叠加层：
Run the server now (`npm start`) and break the code to see an overlay in the browser:

![Error overlay](images/error-overlay.png)

T>如果你想要更好的输出，请考虑[error-overlay-webpack-plugin]（https://www.npmjs.com/package/error-overlay-webpack-plugin），因为它更好地显示了错误的来源。
T> If you want even better output, consider [error-overlay-webpack-plugin](https://www.npmjs.com/package/error-overlay-webpack-plugin) as it shows the origin of the error better.

W> WDS覆盖确实*不*捕获应用程序的运行时错误。
W> WDS overlay does *not* capture runtime errors of the application.

##启用热模块更换
## Enabling Hot Module Replacement

热模块替换是将webpack与众不同的功能之一。实现它需要在服务器端和客户端上进行额外的工作。 *热模块替换*附录更详细地讨论了该主题。如果要将HMR集成到项目中，请查看。但是，完成本教程不需要它。
Hot Module Replacement is one of those features that set webpack apart. Implementing it requires additional effort on both server and client-side. The *Hot Module Replacement* appendix discusses the topic in greater detail. If you want to integrate HMR to your project, give it a look. It won't be needed to complete the tutorial, though.

##从网络访问开发服务器
## Accessing the Development Server from Network

可以通过设置中的环境自定义主机和端口设置（即Unix上的`export PORT = 3000`或Windows上的`SET PORT = 3000`）。在大多数平台上，默认设置已足够。
It's possible to customize host and port settings through the environment in the setup (i.e., `export PORT=3000` on Unix or `SET PORT=3000` on Windows). The default settings are enough on most platforms.

要访问你的服务器，你需要弄清楚你的计算机的IP。在Unix上，这可以使用`ifconfig |来实现grep inet`。在Windows上，可以使用`ipconfig`。 npm包，例如[node-ip]（https://www.npmjs.com/package/node-ip）也派上用场。特别是在Windows上，你需要设置`HOST`以匹配你的IP以使其可访问。
To access your server, you need to figure out the ip of your machine. On Unix, this can be achieved using `ifconfig | grep inet`. On Windows, `ipconfig` can be utilized. An npm package, such as [node-ip](https://www.npmjs.com/package/node-ip) come in handy as well. Especially on Windows, you need to set your `HOST` to match your ip to make it accessible.

{pagebreak}

##快速开发配置
## Making It Faster to Develop Configuration

当你更改捆绑文件时，WDS将处理重新启动服务器，但是当你编辑webpack配置时呢？每次进行更改时重新启动开发服务器都会在一段时间后变得无聊。该过程可以通过[nodemon]（https：//www.npmjs）自动执行[在GitHub中讨论]（https://github.com/webpack/webpack-dev-server/issues/440#issuecomment-205757892）。 com / package / nodemon）监控工具。
WDS will handle restarting the server when you change a bundled file, but what about when you edit the webpack config? Restarting the development server each time you make a change tends to get boring after a while. The process can be automated as [discussed in GitHub](https://github.com/webpack/webpack-dev-server/issues/440#issuecomment-205757892) by using [nodemon](https://www.npmjs.com/package/nodemon) monitoring tool.

要使它工作，你必须首先通过`npm install nodemon --save-dev`安装它。之后，你可以让它观看webpack配置并在更改时重新启动WDS。这是脚本，如果你想试一试：
To get it to work, you have to install it first through `npm install nodemon --save-dev`. After that, you can make it watch webpack config and restart WDS on change. Here's the script if you want to give it a go:

** **的package.json
**package.json**

```json
"scripts": {
  "start": "nodemon --watch webpack.config.js --exec \"webpack-dev-server --mode development\"",
  "build": "webpack --mode production"
},
```

未来WDS [将支持功能]（https://github.com/webpack/webpack-cli/issues/15）本身是可能的。如果你想让它在更改时重新加载，你现在应该实现此解决方法。
It's possible WDS [will support the functionality](https://github.com/webpack/webpack-cli/issues/15) itself in the future. If you want to make it reload itself on change, you should implement this workaround for now.

{pagebreak}

##轮询而不是看文件
## Polling Instead of Watching Files

有时，WDS提供的文件观看设置将无法在你的系统上运行。在旧版本的Windows，Ubuntu，Vagrant和Docker上可能会出现问题。启用轮询是一个很好的选择：
Sometimes the file watching setup provided by WDS won't work on your system. It can be problematic on older versions of Windows, Ubuntu, Vagrant, and Docker. Enabling polling is a good option then:

** ** webpack.config.js
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

设置比默认设置更耗费资源，但值得尝试。
The setup is more resource intensive than the default, but it's worth trying out.

{pagebreak}

##替代方法* webpack-dev-server *
## Alternate Ways to Use *webpack-dev-server*

你可以通过终端传递WDS选项。管理webpack配置中的选项更清楚，因为这有助于保持* package.json *漂亮和整洁。由于你不需要从webpack源代码中挖掘出答案，因此也更容易理解正在发生的事情。
You could have passed the WDS options through a terminal. It's clearer to manage the options within webpack configuration as that helps to keep *package.json* nice and tidy. It's also easier to understand what's going on as you don't need to dig out the answers from the webpack source.

或者，你可以设置Express服务器并使用中间件。有几种选择：
Alternately, you could have set up an Express server and use a middleware. There are a couple of options:

* [官方WDS中间件]（https://webpack.js.org/guides/development/#using-webpack-dev-middleware）
* [The official WDS middleware](https://webpack.js.org/guides/development/#using-webpack-dev-middleware)
* [webpack-hot-middleware]（https://www.npmjs.com/package/webpack-hot-middleware）
* [webpack-hot-middleware](https://www.npmjs.com/package/webpack-hot-middleware)
* [webpack-isomorphic-dev-middleware]（https://www.npmjs.com/package/webpack-isomorphic-dev-middleware）
* [webpack-isomorphic-dev-middleware](https://www.npmjs.com/package/webpack-isomorphic-dev-middleware)

如果你想要更多的控制和灵活性，还有一个[Node API]（https://webpack.js.org/configuration/dev-server/）。
There's also a [Node API](https://webpack.js.org/configuration/dev-server/) if you want more control and flexibility.

W> CLI和Node API之间存在[略有差异]（https://github.com/webpack/webpack-dev-server/issues/616）。
W> There are [slight differences](https://github.com/webpack/webpack-dev-server/issues/616) between the CLI and the Node API.

## webpack-dev-server *的其他功能
## Other Features of *webpack-dev-server*

WDS提供的功能超出了上述范围。你应该注意几个相关领域：
WDS provides functionality beyond what was covered above. There are a couple of relevant fields that you should be aware of:

*`devServer.contentBase`  - 假设你没有动态生成* index.html *并且喜欢自己在特定目录中维护它，你需要将WDS指向它。 `contentBase`接受路径（例如，“build”）或路径数组（例如，`[“build”，“images”]`）。该值默认为项目根目录。
* `devServer.contentBase` - Assuming you don't generate *index.html* dynamically and prefer to maintain it yourself in a specific directory, you need to point WDS to it. `contentBase` accepts either a path (e.g., `"build"`) or an array of paths (e.g., `["build", "images"]`). The value defaults to the project root.
*`devServer.proxy`  - 如果你使用多个服务器，则必须向它们代理WDS。代理设置接受代理映射的对象（例如，`{“/ api”：“http：// localhost：3000 / api”}`），它将匹配的查询解析到另一个服务器。默认情况下禁用代理设置。
* `devServer.proxy` - If you are using multiple servers, you have to proxy WDS to them. The proxy setting accepts an object of proxy mappings (e.g., `{ "/api": "http://localhost:3000/api" }`) that resolve matching queries to another server. Proxy settings are disabled by default.
*`devServer.headers`  - 在此处为你的请求附加自定义标头。
* `devServer.headers` - Attach custom headers to your requests here.

T> [官方文档]（https://webpack.js.org/configuration/dev-server/）涵盖更多选项。
T> [The official documentation](https://webpack.js.org/configuration/dev-server/) covers more options.

##开发插件
## Development Plugins

webpack插件生态系统是多种多样的，有很多插件可以帮助专门开发：
The webpack plugin ecosystem is diverse, and there are a lot of plugins that can help specifically with development:

* [case-sensitive-paths-webpack-plugin]（https://www.npmjs.com/package/case-sensitive-paths-webpack-plugin）在开发不区分大小写的环境（如macOS）时非常方便Windows但使用像Linux这样的区分大小写的环境进行生产。
* [case-sensitive-paths-webpack-plugin](https://www.npmjs.com/package/case-sensitive-paths-webpack-plugin) can be handy when you are developing on case-insensitive environments like macOS or Windows but using case-sensitive environment like Linux for production.
* [npm-install-webpack-plugin]（https://www.npmjs.com/package/npm-install-webpack-plugin）允许webpack在导入时使用* package.json *安装和连接已安装的软件包新项目的包。
* [npm-install-webpack-plugin](https://www.npmjs.com/package/npm-install-webpack-plugin) allows webpack to install and wire the installed packages with your *package.json* as you import new packages to your project.
* [react-dev-utils]（https://www.npmjs.com/package/react-dev-utils）包含为[Create React App]开发的webpack实用程序（https://www.npmjs.com/package/创建反应的应用内）。尽管它的名字，他们可以找到除React以外的用途。如果你只想要webpack消息格式，请考虑[webpack-format-messages]（https://www.npmjs.com/package/webpack-format-messages）。
* [react-dev-utils](https://www.npmjs.com/package/react-dev-utils) contains webpack utilities developed for [Create React App](https://www.npmjs.com/package/create-react-app). Despite its name, they can find use beyond React. If you want only webpack message formatting, consider [webpack-format-messages](https://www.npmjs.com/package/webpack-format-messages).
* [start-server-webpack-plugin]（https://www.npmjs.com/package/start-server-webpack-plugin）能够在webpack构建完成后启动你的服务器。
* [start-server-webpack-plugin](https://www.npmjs.com/package/start-server-webpack-plugin) is able to start your server after webpack build completes.

{pagebreak}

##输出插件
## Output Plugins

还有一些插件可以让webpack输出更容易被注意和理解：
There are also plugins that make the webpack output easier to notice and understand:

* [system-bell-webpack-plugin]（https://www.npmjs.com/package/system-bell-webpack-plugin）在系统响铃失败时响铃，而​​不是让webpack无声地失败。
* [system-bell-webpack-plugin](https://www.npmjs.com/package/system-bell-webpack-plugin) rings the system bell on failure instead of letting webpack fail silently.
* [webpack-notifier]（https://www.npmjs.com/package/webpack-notifier）使用系统通知让你了解webpack状态。
* [webpack-notifier](https://www.npmjs.com/package/webpack-notifier) uses system notifications to let you know of webpack status.
* [nyan-progress-webpack-plugin]（https://www.npmjs.com/package/nyan-progress-webpack-plugin）可用于在构建过程中获得更整洁的输出。如果你使用像Travis这样的持续集成（CI）系统，请小心，因为它们可以破坏输出。 Webpack为同一目的提供了“ProgressPlugin”。不过那里没有nyan。
* [nyan-progress-webpack-plugin](https://www.npmjs.com/package/nyan-progress-webpack-plugin) can be used to get tidier output during the build process. Take care if you are using Continuous Integration (CI) systems like Travis as they can clobber the output. Webpack provides `ProgressPlugin` for the same purpose. No nyan there, though.
* [friendly-errors-webpack-plugin]（https://www.npmjs.com/package/friendly-errors-webpack-plugin）改进了webpack的错误报告。它捕获常见错误并以更友好的方式显示它们。
* [friendly-errors-webpack-plugin](https://www.npmjs.com/package/friendly-errors-webpack-plugin) improves on error reporting of webpack. It captures common errors and displays them in a friendlier manner.
* [webpack-dashboard]（https://www.npmjs.com/package/webpack-dashboard）在标准webpack输出上提供了一个完整的基于终端的仪表板。如果你喜欢清晰的视觉输出，这个就派上用场了。
* [webpack-dashboard](https://www.npmjs.com/package/webpack-dashboard) gives an entire terminal based dashboard over the standard webpack output. If you prefer clear visual output, this one comes in handy.

{pagebreak}

##总结
## Conclusion

WDS补充了webpack，并通过提供面向开发的功能使其对开发人员更友好。
WDS complements webpack and makes it more friendly for developers by providing development oriented functionality.

回顾一下：
To recap:

* Webpack的“watch”模式是迈向更好的开发体验的第一步。你可以在编辑源时使用webpack编译包。
* Webpack's `watch` mode is the first step towards a better development experience. You can have webpack compile bundles as you edit your source.
* WDS可以在更改时刷新浏览器。它还实现**热模块替换**。
* WDS can refresh the browser on change. It also implements **Hot Module Replacement**.
*默认WDS设置在特定系统上可能存在问题。因此，更多的资源密集型轮询是另一种选择。
* The default WDS setup can be problematic on specific systems. For this reason, more resource intensive polling is an alternative.
* WDS可以使用中间件集成到现有的Node服务器中。这样做比依赖命令行界面更容易控制。
* WDS can be integrated into an existing Node server using a middleware. Doing this gives you more control than relying on the command line interface.
* WDS远不止刷新和HMR。例如，代理允许你将其连接到其他服务器。
* WDS does far more than refreshing and HMR. For example proxying allows you to connect it to other servers.

在下一章中，你将学习如何组合配置，以便可以在本书后面进一步开发。
In the next chapter, you learn to compose configuration so that it can be developed further later in the book.

