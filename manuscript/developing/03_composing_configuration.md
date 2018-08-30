# 撰写配置

webpack 还没做什么，配置量就开始变得很大。现在你必须要小心你的配置方式，毕竟项目中有不同的生产和开发两套环境。随着功能增多，情况只会变得更糟。

使用单个配置文件会影响理解,并且无法复用。随着项目需求的增长，你必须找到更有效地管理 webpack 配置的方法。

## 管理配置的方法

你可以通过以下方式管理 webpack 配置：

* 为每个环境维护多个配置，并通过 `--config` 参数指引 webpack，通过模块导入共享配置。
* 将配置推送到库，然后使用该库。示例：[hjs-webpack](https://www.npmjs.com/package/hjs-webpack)，[Neutrino](https://neutrino.js.org/)，[webpack-blocks](https：/ /www.npmjs.com/package/webpack-blocks)。
* 将配置推送到工具。例如：[create-react-app](https://www.npmjs.com/package/create-react-app), [kyt](https://www.npmjs.com/package/kyt), [nwb](https://www.npmjs.com/package/nwb)。
* 在单个文件中维护所有配置，然后依赖 `--env` 参数建立分支，稍后会详细介绍该方法。

可以组合这些方法，抽象到更高级别的配置。然后可以将这些配置打包上传，通过 npm 使用它们，从而方便地在多个项目中使用相同的配置。

## 使用 merge 合并构建配置

如果配置文件被拆分为几个部分，则必须以某种方式再次组合它们。这通常意味着合并对象和数组。为了消除处理 `Object.assign` 和 `Array.concat` 的问题，我（原文作者）开发了 [webpack-merge](https://www.npmjs.org/package/webpack-merge)。

**webpack-merge** 做了两件事：连接数组，合并（而不是覆盖）对象。以下示例详细显示了行为：

```bash
> merge = require("webpack-merge")
...
> merge(
... { a: [1], b: 5, c: 20 },
... { a: [2], b: 10, d: 421 }
... )
{ a: [ 1, 2 ], b: 10, c: 20, d: 421 }
```

**webpack-merge** 提供更多控制方式，使你能够控制每个字段的行为。可以强制 append/prepend/replace 内容。

尽管 **webpack-merge** 是为本书设计的，但现在已经远远不止如此。你可以将其视为一种学习工具，如果你觉得它很方便，可以用于学习或是在工作中使用。

T> [webpack-chain](https://www.npmjs.com/package/webpack-chain) 为配置 webpack 提供了一个流畅的 API，它可以更直观地组织配置文件。

{pagebreak}

## 设置 **webpack-merge**

首先，将 **webpack-merge** 添加到项目中：

```bash
npm install webpack-merge --save-dev
```

作为一层抽象，可以先定义一个顶层配置 **webpack.config.js**，然后为其他配置定义 **webpack.parts.js**。以下是从现有代码中提取的，基于功能区分的其中一个接口：

**webpack.parts.js**

```javascript
exports.devServer = ({ host, port } = {}) => ({
  devServer: {
    stats: "errors-only",
    host, // Defaults to `localhost`
    port, // Defaults to 8080
    open: true,
    overlay: true,
  },
});
```

T> `stats` 也适用于生产配置，所有可用选项请参阅[官方文档](https://webpack.js.org/configuration/stats/)。

{pagebreak}

将这部分配置加入 **webpack.config.js**，如下面的代码示例所示：

**webpack.config.js**

```javascript
const merge = require("webpack-merge");
const HtmlWebpackPlugin = require("html-webpack-plugin");

const parts = require("./webpack.parts");

// 通用配置
const commonConfig = merge([
  {
    plugins: [
      new HtmlWebpackPlugin({
        title: "Webpack demo",
      }),
    ],
  },
]);

// 生产配置
const productionConfig = merge([]);

// 开发配置
const developmentConfig = merge([
  parts.devServer({
    // Customize host/port here if needed
    host: process.env.HOST,
    port: process.env.PORT,
  }),
]);

// 配置导出
module.exports = mode => {
  if (mode === "production") {
    return merge(commonConfig, productionConfig, { mode });
  }

  return merge(commonConfig, developmentConfig, { mode });
};
```

不是直接返回配置，而是返回接受 `env` 参数的函数。该函数返回基于它的配置，并将 webpack `mode` 映射到它。这样做意味着 **package.json** 需要修改：

**package.json**

```json
"scripts": {
leanpub-start-insert
  "start": "webpack-dev-server --env development",
  "build": "webpack --env production"
leanpub-end-insert
leanpub-start-delete
  "start": "webpack-dev-server --mode development",
  "build": "webpack --mode production"
leanpub-end-delete
},
```

作以上更改后，构建还是跟以前一样。但是现在你有了扩展的空间，不必再迷惑于配置的管理。

通过扩展 **package.json** 脚本，在 **webpack.config.js** 建立分支。**webpack.parts.js** 在之后也会添加各种配置。

T> `productionConfig` 现在还没用上，但先把结构写出来，我们进一步扩展配置时它会被改写。

T> 代码中使用的 [process](https://nodejs.org/api/process.html) 模块是 Node 的全局 API。除了 `env` 之外，它还提供了许多其他功能，如获得系统的相关信息。

{pagebreak}

### 理解 `--env`

`--env` 不只能传入字符串传，还可以传入对象：

**package.json**

```json
"scripts": {
  "start": "webpack-dev-server --env development",
  "build": "webpack --env.target production"
},
```

现在在配置中应该能收到 `{target：“production”}` 对象而不是字符串。你可以传递更多的键值对，它们会传入 `env` 对象。如果在设置 `--env.target` 的同时设置 `--env foo`，则会以字符串优先。Webpack 底层处理依赖于[yargs](http://yargs.js.org/docs/#parsing-tricks-dot-notation)。

## “组合”配置的好处

配置拆分利于扩展配置。最重要是你可以提取不同目标之间的共性。你还可以找出一些公共设置，把他们发布成 npm 包以跨项目使用。

你不必再在多个项目之间复制配置，而是将配置作为依赖项进行管理。在你优化这个配置之后，你的所有项目都会收益。

每种方法都有其优点和缺点。拆分配置后，每个文件都会变得精简，可以看看别人是怎么写的，你总能找到最适合你的方法。

也许最大的问题是，你需要知道如何“组合”，而且你可能不会在第一时间得到正确的“组合”方式。毕竟这是一个超越 webpack 范畴的软件工程问题。

你可以随时迭代接口并找到更好的接口。把对象而不是多个参数传入配置，可以在不影响其 API 的情况下修改这个配置。

## 配置布局

此项目中，配置都写在两个文件中：**webpack.config.js** 和 **webpack.parts.js**。前者包含顶层配置，而后者包含各种细节。这个方法可以使用更丰富的布局方式。

### 按配置环境拆分

如果你按环境拆分配置，最​​终可能会得到如下文件结构：

```bash
.
└── config
    ├── webpack.common.js
    ├── webpack.development.js
    ├── webpack.parts.js
    └── webpack.production.js
```

在这种情况下，你可以通过 webpack `--config` 参数确定环境，通过 `module.exports = merge(common, config);` 来 `merge` 通用配置。

{pagebreak}

### 按目的拆分 parts 文件

要对管理配置添加层次结构，你可以按类别拆分 **webpack.parts.js**：

```bash
.
└── config
    ├── parts
    │   ├── devserver.js
    ...
    │   ├── index.js
    │   └── javascript.js
    └── ...
```

这种处理可以让你更快地找到与分类相关的配置。你也可以把他们放在一个文件里，使用注释将其拆分。

### 打包 parts 文件

鉴于所有配置都是 JavaScript，你理所当然地可以使用包。你可以打包需要共享的配置，以便可以跨项目使用它们。相关实现的更多信息请参阅 [SurviveJS - Maintenance](https://survivejs.com/maintenance/) 一书。

{pagebreak}

## 总结

虽然配置的输出还是跟以前还是一样，但现在它有充分的拓展空间。

回顾一下：

* webpack 配置是 JavaScript 代码构成的, 有很多方法可以管理它。
* 你应该根据实际情况选择配置方法。[webpack-merge](https://www.npmjs.com/package/webpack-merge)为了提供一种轻松的合成方法，但还有许多其他选择。
* Webpack 的 `--env` 参数可以通过终端配置。然后通过一个函数接口收到传递的 `env`。
* “组合”方便配置共享。你无需在每个仓库维护自定义配置，而是可以通过这种方式在仓库之间共享它。可以通过 npm 包做到这点。开发配置其实跟真正的程序开发差不多了。现在，你可以把你的实践打包上传。

本书的下一部分将介绍不同的技术，对 *webpack.parts.js* 有比较多的修改。幸运的是，*webpack.config.js* 的更改仍然很少。

