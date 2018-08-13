# 什么是 Webpack

> 原文链接：https://survivejs.com/webpack/what-is-webpack/

**涉及到的未翻译单词**
- input 输入
- output 输出
- entry 入口文件
- bundle 包（打包结果）

Webpack 是**模块打包器**。它可以在打包的同时使用任务运行器。然而，由于社区开发的 webpack 插件，打包器和任务运行器之间的界限变得模糊。有时，这些插件甚至独立于 webpack 使用，例如清理构建目录或部署构建使用的插件。
（译者注：grunt 之类的任务运行器，就是自动帮你走一次如“组合两个 js 文件--压缩 js--压缩 css”这样流程的工具）

Webpack 也可以在其他环境中的使用，例如 [Ruby on Rails](https://github.com/rails/webpacker)。尽管它的名字带有 web，但 webpack 并不仅限于 web。它也可以打包其他东西，这点在 *Build Targets* 章节中会提到。

T> 如果你想更详细地了解构建工具及其历史，请查看附录 *Comparison of Build Tools*。

## Webpack 基于模块

使用 webpack 构建工程，至少包括 **input** 和 **output**。打包处理从用户定义的 **entry** 开始。entry 本身就是**模块**，它可以通过 **import** 指向其他模块。

当你使用 webpack 打包项目时，它会遍历 import，构建项目的**依赖关系图**，然后根据配置文件中的设定生成 **output**。你还可以定义**分割点**，以在项目代码内拆分出单独的 bundle（包）。

Webpack 支持开箱即用的 ES2015，CommonJS 和 AMD 模块标准。loader 机制也适用于 CSS，通过 *css-loader* 在 css 文件中使用 `@import` 和 `url()`。你还可以找到某些实现特定功能的插件，例如压缩，国际化，HMR等。

T> 依赖图是描述节点如何相互关联的有向图。这个图是通过文件之间的引用（`require`，`import`）构建的。Webpack 会在不执行资源的情况下静态遍历这些资源，并生成创建 bundle 所需的依赖图。

## Webpack 的执行流程

![Webpack's execution process](https://raw.githubusercontent.com/ssshooter/webpack-book/dev/manuscript/images/webpack-process.png)

Webpack 从 **entry** 开始运行。entry 通常是 JavaScript 模块，webpack 从这里开始遍历处理。在此过程中，webpack 根据 **loader** 配置转换每个匹配到的模块。


### 模块解析

Entry 本身就是一个模块。当 webpack 遇到 entry，webpack 会在文件系统匹配相关文件。除了 *node_modules* 之外，webpack 还可以对特定目录执行查找。也可以调整 webpack 匹配文件扩展名的方式，也可以为目录定义 aliases（别名）。这方面在 *Consuming Packages* 章节有更详细的介绍。

如果 webpack 正确解析文件，对应 loader 会处理匹配的文件，不同的 loader 对模块内容应用的转换各不相同。如果解析失败，webpack 会报运行时错误。

loader 可以通过多种方式匹配待处理文件，如文件类型和文件的位置。 Webpack 甚至可以让你按 import 位置分类，不同位置 import 的文件采用不同的转换方法。

对 webpack loader 执行相同的解析过程。 你可以在选择 loader 时使用相同的逻辑。由于这个原因，Loader 已经解析了自己的配置。如果 webpack 查找 loader 失败，则会引发运行时错误。

T> Webpack 的解释底层依赖于 [enhanced-resolve](https://www.npmjs.com/package/enhanced-resolve) 包。

### Webpack 可以解析任何类型的文件

Webpack 将在构造依赖图时解析它遇到的每个模块。如果 entry 包含依赖项，则将针对每个依赖项递归执行该过程，直到遍历完成为止。 Webpack 可以针对任何文件类型执行此过程，这与 Babel 或 Sass 编译器等专用工具不同。

Webpack 可以控制对不同资源的处理方式。例如，可以把资源**内联**到 JavaScript 包以避免过多的网络请求（译者注：例如图片转 base64）。Webpack 还允许你使用 CSS 模块等技术将样式与组件结合，并避免标准 CSS 样式问题。这种灵活性是 webpack 价值的体现。

尽管 webpack 主要用于打包 JavaScript，但它可以捕获图像或字体等资源，并为它们抽取为单独的文件。Entry 只是打包处理的起点。 webpack 生成的内容完全取决于你配置它的方式。

### 处理流程

Webpack 会下到上、从右到左地（`styleLoader(cssLoader('./main.css'))`）处理匹配成功的加载器，模块会依次通过 loader 的处理。最后，你将获得 webpack 输出的包。*Loader Definitions* 章节详细介绍了该主题。

如果所有 loader 都成功运行，则 webpack 会在最后一个包中包含源。 **Plugins** 可以在打包过程的不同阶段拦截**运行时事件**。

虽然 loader 可以做很多事情，但它们不能为高级任务提供足够的动力。Plugins 可以拦截 webpack 提供的**运行时事件**。一个很好的例子是由 `MiniCssExtractPlugin` 执行的包提取，当与 loader 一起使用时，从包中提取 CSS 文件并将其提取到单独的文件中。如果没有这一步，CSS 将在生成的 JavaScript 中内联，因为 webpack 默认将所有代码视为 JavaScript。CSS 提取将在 *Separating CSS* 一章中讨论。

### 完成

每个模块都经过处理之后，webpack 生成 **output**。output 包括一个引导脚本，其中包含一个指引浏览器执行该项目的 manifest 文件。可以将 manifest 提取到单独的文件中，本书后面会有相关介绍。output 会根据你使用的 build target 而有所不同（Web 不是唯一选择）。

这并不是打包过程的全部内容。例如，你可以做代码拆分，webpack 会在程序运行到所需功能时才加载的单独包。这个话题会在 *Code Splitting* 章节中讨论。

## 配置驱动的 Webpack

webpack 的核心依赖于配置。以下是根据[官方 webpack 教程](https://webpack.js.org/get-started/)改编的配置示例证明：

**webpack.config.js**

```javascript
const webpack = require("webpack");

module.exports = {
  // Where to start bundling
  entry: {
    app: "./entry.js",
  },

  // Where to output
  output: {
    // Output to the same directory
    path: __dirname,

    // Capture name from the entry using a pattern
    filename: "[name].js",
  },

  // How to resolve encountered imports
  module: {
    rules: [
      {
        test: /\.css$/,
        use: ["style-loader", "css-loader"],
      },
      {
        test: /\.js$/,
        use: "babel-loader",
        exclude: /node_modules/,
      },
    ],
  },

  // What extra processing to perform
  plugins: [
    new webpack.DefinePlugin({ ... }),
  ],

  // Adjust module resolution algorithm
  resolve: {
    alias: { ... },
  },
};
```

Webpack 的配置模型有时候会让人感觉雾里看花，因为配置文件太庞大，属性太多。除非你理解每一个属性的意义，否则很难理解 webpack 在做什么。让你完全理解 webpack 配置的使用，是本书存在的主要目的之一。

## 资源哈希编码

使用 webpack 可以为每个包的名称注入一个哈希值（例如，*app.d587bbd6.js*），以便在版本更新后使客户端上旧版本的包无效（重新下载）。包（bundle）拆分可以让客户端在理想情况下仅重新加载一小部分数据。

## 热模块更换（HMR）

你可能使用过 [LiveReload](http://livereload.com/) 或 [BrowserSync](http://www.browsersync.io/) 等工具。这些工具会在你进行更改时自动刷新浏览器。**热模块更换**（HMR）则更先进，在使用 React 的情况下，应用程序可以在不强制刷新页面的情况下更新应用。虽然这听起来没什么特别，但它可以在实践中大有不同。

HMR 不是webpack独有的功能，通过 [livereactload](https://github.com/milankinen/livereactload) 也可以在 Browserify 中使用 HMR。

## 代码拆分

Webpack 可以以多种方式拆分代码。你甚至可以在应用程序执行时动态加载代码。因为依赖可以根据需要即时加载，所以延迟加载特别适用于体积庞大的应用。

即使是小型应用也可以得益于代码拆分，因为它允许用户更快地获得可用的东西。毕竟，性能是评价一个应用的重要标准，了解这项技术是值得的。

## 总结

Webpack 学习曲线比较陡峭。但是有了它，项目的长期维护可以节省多少时间和精力，所以这是一个非常值得学习的工具。为了更好地了解 Webpack 与其他工具的比较，请查看[官网上与其他工具的比较](https://webpack.js.org/comparison/)。

Webpack 不是万能的。但它确实很好地解决了打包问题。在开发过程中需要担心的问题又少了一件。活用 *package.json* 和 webpack，走遍天下都不怕。

总结一下：

* Webpack 是**模块打包器**，但你也可以使用它运行任务（译者注：也就是顺序运行一系列操作）。
* Webpack 底层基于**依赖图**。Webpack 遍历源文件构建依赖图，并以依赖图和配置为基础生成 bundle。
* Webpack 依赖于 **loader** 和 **plugin**。loader 在模块级别上运行，而 plugin 依赖于 webpack 提供的钩子，并且可以很好地访问其执行过程。
* Webpack 的**配置**描述了如何转换“依赖图”中的资源以及它应该生成什么样的输出。如果要使用**代码拆分**等功能，则可以将拆分指令写在源代码中。
* Webpack 如此受人喜爱的原因之一是**热模块更换**（HMR）。这个功能可以不刷新整页而更新应用代码，开发体验极好。
* Webpack 可以为文件名生成**哈希值**，在内容更改时，可以作废浏览器缓存中上个版本的包。

在本书的下一部分中，你将学习使用 webpack 构建开发配置，同时了解更多相关概念。

T> 如果你对 webpack 仍有疑问或者不明白我们为什么需要一个打包器，请阅读[Why would I use a Webpack?](http://tinselcity.net/whys/packers)。
