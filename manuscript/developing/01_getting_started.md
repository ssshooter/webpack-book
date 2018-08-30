# 从 0 开始构建 webpack 项目

在开始之前，请确保你使用的是 [Node](http://nodejs.org/) 的最新版本。至少是最新的 LTS（长期支持）版本，本书的配置基于 LTS 版本所写，你的终端需要有 `node` 和 `npm` 命令，[Yarn](https://yarnpkg.com/) 也是一个不错的选择，也适用于本教程。

通过使用 [Docker](https://www.docker.com/)，[Vagrant](https://www.vagrantup.com/) 或 [nvm](https://www.npmjs.com/package/nvm) 等方案，可以获得更加可控的环境。Vagrant 可以为团队中每个开发人员提供统一的环境，但它因依赖于虚拟机，性能稍处劣势。

T> 本书的完整配置可在 [GitHub](https://github.com/survivejs-demos/webpack-demo) 上找到。

{pagebreak}

## 建立项目

首先，你应该为项目创建一个目录并在那里新建 *package.json*。 npm 使用它来管理项目依赖。以下是基本命令：

```bash
mkdir webpack-demo
cd webpack-demo
npm init -y # -y generates *package.json*, skip for more control
```

生成 *package.json* 后，你可以手动对其进一步更改。官方文档更详细地解释了 [package.json选项](https://docs.npmjs.com/files/package.json)。

T> 你可以在 *~/.npmrc* 中设置 `npm init` 的默认值。

T> 这是使用 [Git](https://git-scm.com/) 进行版本控制的绝佳机会。你可以为每个步骤/每一章节创建一个 commit，这样你可以轻松做到回滚。

T> 本书的例子使用 [Prettier](https://www.npmjs.com/package/prettier) 进行格式化。使用选项为 `"trailingComma": "es5",` 和 `"printWidth": 68`，这样可以使 diff 看起来更舒服，更适合页面展示。

## 安装 Webpack

即使可以全局安装 webpack（`npm install webpack -g`），最好还是将它作为项目的依赖项来维护，以避免出现因为不同版本带来的意外问题。该方法也适用于**持续集成**（CI）设置。 CI 系统可以安装本地依赖项，使用它们编译项目，然后将结果推送到服务器。

要将 webpack 添加到项目中，请执行：

```bash
npm install webpack webpack-cli --save-dev # -D to type less
```

你应该可以在你的 **package.json** 文件中 `devDependencies` 部分看到 webpack。除了在 **node_modules** 目录下本地安装软件包之外，npm 还会为可执行文件生成一个入口。

T> 你可以使用 `--save` 和 `--save-dev` 来分离应用程序和开发依赖项。前者安装并写入 **package.json** `dependencies`字段，而后者则写入 `devDependencies`。

T> [webpack-cli](https://www.npmjs.com/package/webpack-cli) 附带了其他功能，包括 `init` 和 `migrate` 命令，可以快速创建新的 webpack 配置或从旧版本更新版本。

## 执行 Webpack

你可以使用 `npm bin` 显示可执行文件的确切路径。它很有可能指向 **./node_modules/.bin**。尝试使用 `node_modules/.bin/webpack` 或类似命令从终端运行 webpack。

运行后，你应该能看到版本号，一条指南链接以及一个选项列表。大多数的选项都没有在这个项目中使用，但了解一下也是极好的。

```bash
$ node_modules/.bin/webpack
Hash: 6736210d3313db05db58
Version: webpack 4.1.1
Time: 88ms
Built at: 3/16/2018 3:35:07 PM

WARNING in configuration
The 'mode' option has not been set. Set 'mode' option to 'development' or 'production' to enable defaults for this environment.

ERROR in Entry module not found: Error: Can't resolve './src' in '.../webpack-demo'
```

这段输出说明 webpack 找不到需要编译的源代码。同时它还缺少一个 `mode` 参数判定运行环境。

为了快速了解 webpack 输出，我们应该解决这两个问题：

1. 在 *src/index.js* 写入 `console.log("Hello world");`。
2. 执行 `node_modules/.bin/webpack --mode development`。Webpack 按设置查找源文件。
3. 查看 *dist/main.js*。你应该看到 webpack 的引导代码，在引导程序下面，你应该可以看到 `console.log("Hello world");`。

T> 尝试添加 `--mode production` 看返回值有什么不同。

{pagebreak}

## 准备资源

我们尝试做复杂一点的构建，在项目中添加另一个模块，逐步开发一个小应用：

**src/component.js**

```javascript
export default (text = "Hello world") => {
  const element = document.createElement("div");

  element.innerHTML = text;

  return element;
};
```

我们还必须修改入口文件，导入新文件，通过 DOM 渲染应用程序：

**src/index.js**

```javascript
import component from "./component";

document.body.appendChild(component());
```

构建后查看输出文件（`node_modules/.bin/webpack --mode development`）。你应该看到 webpack 已将两个模块写入 `dist` 目录的 bundle。

默认情况下，Webpack 将生成基于 `eval` 的 source maps，它会让输出显得很混乱，可以把 `--devtool false` 参数传递给 webpack 禁用该行为。详细信息请参阅 *Source Maps* 章节。

还有一个问题，我们如何在浏览器中测试应用程序？

## 配置 *html-webpack-plugin*

可以通过编写指向生成的文件的 *index.html* 文件来解决该问题。我们无需手动完成这件事，而是借助插件和 webpack 配置来完成此操作。

先安装 *html-webpack-plugin*：

```bash
npm install html-webpack-plugin --save-dev
```

在 webpack 中使用 plugins：

**webpack.config.js**

```javascript
const HtmlWebpackPlugin = require("html-webpack-plugin");

module.exports = {
  plugins: [
    new HtmlWebpackPlugin({
      title: "Webpack demo",
    }),
  ],
};
```

配置完成后，尝试以下操作：

1. 运行 `node_modules/.bin/webpack --mode production` 构建项目。你也可以试试 `development` 模式。
2. 运行 `cd dist` 进入构建目录。
3. 使用 `serve`（`npm i serve -g`）或类似工具运行服务器。
4. 通过浏览器查看结果。希望结果如你所愿~

![Hello world](images/hello_01.png)

T> 本书使用 **Trailing commas（尾逗号）**，这样做 diff 会更清晰简洁。

## 查看输出

现在执行 `node_modules/.bin/webpack --mode production`，应该看到如下输出：

```bash
Hash: aafe36ba210b0fbb7073
Version: webpack 4.1.1
Time: 338ms
Built at: 3/16/2018 3:40:14 PM
     Asset       Size  Chunks             Chunk Names
   main.js  679 bytes       0  [emitted]  main
index.html  181 bytes          [emitted]
Entrypoint main = main.js
   [0] ./src/index.js + 1 modules 219 bytes {0} [built]
       | ./src/index.js 77 bytes [built]
       | ./src/component.js 142 bytes [built]
Child html-webpack-plugin for "index.html":
     1 asset
    Entrypoint undefined = index.html
       [0] (webpack)/buildin/module.js 519 bytes {0} [built]
       [1] (webpack)/buildin/global.js 509 bytes {0} [built]
        + 2 hidden modules
```

这段输出的信息量很大：

* `Hash: aafe36ba210b0fbb7073`--当前构建的哈希。使用它来通过 `[hash]` 占位符使旧版本资源无效。 填充哈希值将在 *Adding Hashes to Filenames* 章节中详细讨论。
* `Version: webpack 4.1.1`--Webpack版本。
* `Time: 338ms`--执行构建所花费的时间。
* `main.js  679 bytes       0  [emitted]  main`--生成资源的名称，大小，与其相关的 **chunks** 的 ID，状态信息，生成方式以及名称。
* `index.html  181 bytes          [emitted]`--进程生成的另一个资源。
* `[0] ./src/index.js + 1 modules 219 bytes {0} [built]`--入口资源的 ID，名称，大小，entry chunk ID，生成方式。
* `Child html-webpack-plugin for "index.html":`--这是与插件相关的输出。此输出为 *html-webpack-plugin* 创建。

查看 `dist/` 目录下面的输出。如果仔细观察，可以在代码中看到相同的 ID。

T> webpack 配置除对象外，也可以通过返回一个 `Promise` 并最终 `resolve` 返回配置信息。

T> 如果不想使用 *html-webpack-plugin*，可以尝试功能较少但更好理解的 [mini-html-webpack-plugin](https://www.npmjs.com/package/mini-html-webpack-plugin)。

{pagebreak}

## 添加构建快捷方式

执行 `node_modules/.bin/webpack` 显得过于冗长，我们应该为此提供一个快捷入口。修改 *package.json*：

**package.json**

```json
"scripts": {
  "build": "webpack --mode production"
},
```

运行 `npm run build` 应该能看到与之前相同的输出。npm 会自动将 *node_modules/.bin* 添加到路径中。因此，你不必这么写 `"build": "node_modules/.bin/webpack"`，而直接是 `"build": "webpack"`。

你可以通过 *npm run* 执行此类脚本，并且可以在项目中的任何位置使用 *npm run*。如果直接运行 *npm run*，它将为你提供可用脚本的列表。

T> 还有像 *npm start* 和 *npm test* 这样的快捷方式。你可以在不使用 *npm run* 的情况下直接运行它们（尽管也可以）。如果你甚至可以懒到使用 *npm t* 来运行测试。

T> 更进一步，还可以使用终端配置中的 `alias` 命令设置系统级别名。例如将 `nrb` 映射到 `npm run build`。

{pagebreak}

## `HtmlWebpackPlugin` 扩展

你可以自己写 `HtmlWebpackPlugin` 的 template，但有这里也有一些预设模板，如 [html-webpack-template](https://www.npmjs.com/package/html-webpack-template) 或 [html-webpack-template-pug](https://www.npmjs.com/package/html-webpack-template-pug)。

还有一些插件可以扩展 `HtmlWebpackPlugin` 的功能：

* [favicons-webpack-plugin](https://www.npmjs.com/package/favicons-webpack-plugin) 能够生成favicon。
* [script-ext-html-webpack-plugin](https://www.npmjs.com/package/script-ext-html-webpack-plugin) 使你可以更好地控制 script 标记，并允许你进一步调整脚本加载。
* [style-ext-html-webpack-plugin](https://www.npmjs.com/package/style-ext-html-webpack-plugin) 将 CSS 引用转换为内联 CSS。该技术使 CSS 成为初始加载的一部分，可以快速向客户端提供关键 CSS。
* [resource-hints-webpack-plugin](https://www.npmjs.com/package/resource-hints-webpack-plugin) 添加 [resource hints](https://www.w3.org/TR/resource-hints/) 到你的HTML文件，以加快加载时间。
* [preload-webpack-plugin](https://www.npmjs.com/package/preload-webpack-plugin) 为脚本启用 `rel=preload` 功能，对懒加载比较使用，本书 *Building* 部分将会讨论到。
* [webpack-cdn-plugin](https://www.npmjs.com/package/webpack-cdn-plugin) 允许你指定通过内容交付网络（CDN）加载的依赖项。这常用于加速流行库的加载。
* [dynamic-cdn-webpack-plugin](https://www.npmjs.com/package/dynamic-cdn-webpack-plugin) 功能类似。

{pagebreak}

## 总结

虽然你成功运行了 webpack，但这还远远不够，后续开发仍需努力。现在每次要查看应用程序时，都必须使用 `npm run build` 手动构建，然后刷新浏览器。这个痛点是 webpack 高级功能的用武之地。

回顾一下：

* 本地安装 webpack 比全局安装 webpack 更有优势。这样你可以保证你正在使用的是哪个版本。本地依赖项也适用于持续集成环境。
* Webpack 通过 *webpack-cli* 包提供命令行界面。虽然没有配置也可以使用，但任何进阶使用都需要配置。
* 对于更复杂的设置，你很可能需要编写单独的 **webpack.config.js** 文件。
* `HtmlWebpackPlugin` 可用于生成应用程序的 HTML 入口。在 *Multiple Pages* 一章中，你将了解如何使用它生成多个独立的页面。
* 使用 npm *package.json* 脚本来管理 webpack 很方便。你可以将它用作轻型任务运行器，也可以使用于 webpack 之外的其他功能。

在下一章中，你将学习如何通过启用自动浏览器刷新来改善开发体验。

