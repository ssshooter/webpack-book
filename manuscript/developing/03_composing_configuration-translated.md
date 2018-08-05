# 撰写配置项

webpack 还没做什么，配置量就开始变得很大。现在你必须要小心你的配置方式，毕竟项目中有不同的生产和开发两套环境。随着功能增多，情况只会变得更糟。

使用单个配置文件会影响理解,并且无法复用。随着项目需求的增长，您必须找到更有效地管理 webpack 配置的方法。

## 管理配置的可能方法

你可以通过以下方式管理 webpack 配置：

*在每个环境的多个文件中维护配置，并通过`--config`参数将webpack指向每个文件，通过模块导入共享配置。
* Maintain configuration within multiple files for each environment and point webpack to each through the `--config` parameter, sharing configuration through module imports.
*将配置推送到库，然后使用该库。示例：[hjs-webpack]（https://www.npmjs.com/package/hjs-webpack），[Neutrino]（https://neutrino.js.org/），[webpack-blocks]（https：/ /www.npmjs.com/package/webpack-blocks）。
* Push configuration to a library, which you then consume. Examples: [hjs-webpack](https://www.npmjs.com/package/hjs-webpack), [Neutrino](https://neutrino.js.org/), [webpack-blocks](https://www.npmjs.com/package/webpack-blocks).
*将配置推送到工具。示例：[create-react-app]（https://www.npmjs.com/package/create-react-app），[kyt]（https://www.npmjs.com/package/kyt），[nwb ]（https://www.npmjs.com/package/nwb）。
* Push configuration to a tool. Examples: [create-react-app](https://www.npmjs.com/package/create-react-app), [kyt](https://www.npmjs.com/package/kyt), [nwb](https://www.npmjs.com/package/nwb).
*维护单个文件中的所有配置并在那里进行分支并依赖于`--env`参数。本章后面将详细介绍该方法。
* Maintain all configuration within a single file and branch there and rely on the `--env` parameter. The approach is explained in detail later in this chapter.

可以组合这些方法以创建更高级别的配置，然后由更小的部分组成。然后可以将这些部分添加到库中，然后通过npm使用它，从而可以在多个项目中使用相同的配置。
These approaches can be combined to create a higher level configuration that is then composed of smaller parts. Those parts could then be added to a library which you then use through npm making it possible to consume the same configuration across multiple projects.

## 使用 merge 合并构建配置

如果配置文件被分成不同的部分，则必须以某种方式再次组合它们。通常这意味着合并对象和数组。为了消除处理`Object.assign`和`Array.concat`的问题，开发了[webpack-merge]（https://www.npmjs.org/package/webpack-merge）。
If the configuration file is broken into separate pieces, they have to be combined again somehow. Normally this means merging objects and arrays. To eliminate the problem of dealing with `Object.assign` and `Array.concat`, [webpack-merge](https://www.npmjs.org/package/webpack-merge) was developed.

*webpack-merge* 做了两件事：连接数组，合并（而不是覆盖）对象。以下示例详细显示了行为：

```bash
> merge = require("webpack-merge")
...
> merge(
... { a: [1], b: 5, c: 20 },
... { a: [2], b: 10, d: 421 }
... )
{ a: [ 1, 2 ], b: 10, c: 20, d: 421 }
```

*webpack-merge* 提供更多控制方式，使你能够控制每个字段的行为。你可以强制 append/prepend/replace 内容。

尽管 *webpack-merge* 是为本书设计的，但现在已经远远不止如此。您可以将其视为一种学习工具，如果您觉得它很方便，可以用于学习或是在工作中使用。

T> [webpack-chain]（https://www.npmjs.com/package/webpack-chain）提供了一个流畅的API，用于配置webpack，允许您在启用组合时避免与配置形状相关的问题。
T> [webpack-chain](https://www.npmjs.com/package/webpack-chain) provides a fluent API for configuring webpack allowing you to avoid configuration shape-related problems while enabling composition.

{pagebreak}

## 设置 *webpack-merge*

首先，将* webpack-merge *添加到项目中：
To get started, add *webpack-merge* to the project:

```bash
npm install webpack-merge --save-dev
```

要给出一定程度的抽象，可以为更高级别的配置定义* webpack.config.js *，为要使用的配置部分定义* webpack.parts.js *。以下是从现有代码中提取的具有基于函数的小接口的部分：
To give a degree of abstraction, you can define *webpack.config.js* for higher level configuration and *webpack.parts.js* for configuration parts to consume. Here are the parts with small function-based interfaces extracted from the existing code:

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

T>同样的`stats`理念也适用于生产配置。有关所有可用选项，请参阅[官方文档]（https://webpack.js.org/configuration/stats/）。
T> The same `stats` idea works for production configuration as well. See [the official documentation](https://webpack.js.org/configuration/stats/) for all the available options.

{pagebreak}

要连接此配置部分，请设置* webpack.config.js *，如下面的代码示例所示：
To connect this configuration part, set up *webpack.config.js* as in the code example below:

**webpack.config.js**

```javascript
const merge = require("webpack-merge");
const HtmlWebpackPlugin = require("html-webpack-plugin");

const parts = require("./webpack.parts");

const commonConfig = merge([
  {
    plugins: [
      new HtmlWebpackPlugin({
        title: "Webpack demo",
      }),
    ],
  },
]);

const productionConfig = merge([]);

const developmentConfig = merge([
  parts.devServer({
    // Customize host/port here if needed
    host: process.env.HOST,
    port: process.env.PORT,
  }),
]);

module.exports = mode => {
  if (mode === "production") {
    return merge(commonConfig, productionConfig, { mode });
  }

  return merge(commonConfig, developmentConfig, { mode });
};
```

不是直接返回配置，而是返回捕获传递的`env`的函数。该函数返回基于它的配置，并将webpack`mode`映射到它。这样做意味着* package.json *需要修改：
Instead of returning a configuration directly, a function capturing the passed `env` is returned. The function returns configuration based on it and also maps webpack `mode` to it. Doing this means *package.json* needs a modification:

** **的package.json
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

在这些更改之后，构建应该像以前一样运行。但是，这次您有扩展的空间，您不必担心如何组合配置的不同部分。
After these changes, the build should behave the same way as before. This time, however, you have room to expand, and you don't have to worry about how to combine different parts of the configuration.

您可以通过扩展* package.json *定义并根据需要在* webpack.config.js *分支来添加更多目标。 * webpack.parts.js *增长以包含您可以用来组成配置的特定技术。
You can add more targets by expanding the *package.json* definition and branching at *webpack.config.js* based on the need. *webpack.parts.js* grows to contain specific techniques you can then use to compose the configuration.

T>`productionConfig`现在是一个存根，随着我们进一步扩展配置，它将会增长。
T> `productionConfig` is a stub for now and it will grow later as we expand the configuration further.

T>代码中使用的[process]（https://nodejs.org/api/process.html）模块由Node作为全局公开。除了`env`之外，它还提供了许多其他功能，使您可以获得有关主机系统的更多信息。
T> The [process](https://nodejs.org/api/process.html) module used in the code is exposed by Node as a global. In addition to `env`, it provides plenty of other functionality that allows you to get more information of the host system.

{pagebreak}

### 理解 `--env`

即使`--env`允许将字符串传递给配置，它也可以做更多。请考虑以下示例：
Even though `--env` allows to pass strings to the configuration, it can do a bit more. Consider the following example:

**package.json**

```json
"scripts": {
  "start": "webpack-dev-server --env development",
  "build": "webpack --env.target production"
},
```

而不是字符串，你现在应该在配置时收到一个对象`{target：“production”}`。你可以传递更多的键值对，它们会转到`env`对象。如果在设置`--env.target`时设置`--env foo`，则字符串获胜。 Webpack依赖于[yargs]（http://yargs.js.org/docs/#parsing-tricks-dot-notation）进行解析。
Instead of a string, you should receive an object `{ target: "production" }` at configuration now. You could pass more key-value pairs, and they would go to the `env` object. If you set `--env foo` while setting `--env.target`, the string wins. Webpack relies on [yargs](http://yargs.js.org/docs/#parsing-tricks-dot-notation) for parsing underneath.

## “组合”配置的好处

配置拆分允许您继续扩展设置。最重要的胜利是你可以提取不同目标之间的共性。您还可以识别要组成的较小配置部件。这些配置部件可以推送到自己的软件包以跨项目使用。
Configuration splitting allows you to keep on expanding the setup. The most significant win is the fact that you can extract commonalities between different targets. You can also identify smaller configuration parts to compose. These configuration parts can be pushed to packages of their own to consume across projects.

您可以将配置作为依赖项进行管理，而不是在多个项目之间复制类似的配置。当您找到更好的方法来执行任务时，您的所有项目都会收到改进。
Instead of duplicating similar configuration across multiple projects, you can manage configuration as a dependency now. As you figure out better ways to perform tasks, all your projects receive the improvements.

每种方法都有其优点和缺点。基于构图的方法是一个很好的起点。除了合成之外，它还为您提供了有限的代码进行扫描，但最好还要了解其他人是如何进行扫描的。你可以根据自己的喜好找到最好的东西。
Each approach comes with its pros and cons. The composition-based approach is a good starting point. In addition to composition, it gives you a limited amount of code to scan through, but it's a good idea to check out how other people do it too. You can find something that works the best based on your tastes.

也许最大的问题是，你需要知道自己在做什么，而且你可能不会在第一时间得到正确的构图。但这是一个超越webpack的软件工程问题。
Perhaps the biggest problem is that with composition you need to know what you are doing, and it's possible you aren't going to get the composition right the first time around. But that's a software engineering problem that goes beyond webpack.

您可以随时迭代接口并找到更好的接口。通过传入配置对象而不是多个参数，您可以在不影响其API的情况下更改零件的行为，从而根据需要有效地公开API。
You can always iterate on the interfaces and find better ones. By passing in a configuration object instead of multiple arguments you can change the behavior of a part without affecting its API, effectively exposing the API as you need it.

##配置布局
## Configuration Layouts

在book项目中，您将所有配置推送到两个文件中：* webpack.config.js *和* webpack.parts.js *。前者包含更高级别的配置，而较低级别将您与webpack特定信息隔离开来。所选择的方法允许比我们拥有的文件布局更多的文件布局。
In the book project, you will push all of the configuration into two files: *webpack.config.js* and *webpack.parts.js*. The former contains higher level configuration while the lower level isolates you from webpack specifics. The chosen approach allows more file layouts than the one we have.

### 按配置环境拆分

如果您按环境拆分配置，最​​终可能会得到如下文件结构：

```bash
.
└── config
    ├── webpack.common.js
    ├── webpack.development.js
    ├── webpack.parts.js
    └── webpack.production.js
```

在这种情况下，您可以通过webpack` --config`参数指向目标，通过`module.exports = merge（common，config）;`指向`merge`通用配置。
In this case, you would point to the targets through webpack `--config` parameter and `merge` common configuration through `module.exports = merge(common, config);`.

{pagebreak}

### 按目的拆分 parts 文件

要在管理配置部件的方式中添加层次结构，您可以按类别分解* webpack.parts.js *：
To add hierarchy to the way configuration parts are managed, you could decompose *webpack.parts.js* per category:

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

这种安排可以更快地找到与类别相关的配置。一个很好的选择是将部件安排在一个文件中，并使用注释将其拆分。
This arrangement would make it faster to find configuration related to a category. A good option would be to arrange the parts within a single file and use comments to split it up.

### 打包 parts 文件

鉴于所有配置都是JavaScript，没有什么可以阻止您将其作为包或包使用。可以打包共享配置，以便您可以跨多个项目使用它。有关如何实现此目的的更多信息，请参阅[SurviveJS  - 维护]（https://survivejs.com/maintenance/）一书。
Given all configuration is JavaScript, nothing prevents you from consuming it as a package or packages. It would be possible to package the shared configuration so that you can consume it across multiple projects. See the [SurviveJS - Maintenance](https://survivejs.com/maintenance/) book for further information on how to achieve this.

{pagebreak}

## 结论

即使配置在技术上与以前相同，但现在您还有空间来扩展它。
Even though the configuration is technically the same as before, now you have room to grow it.

回顾一下：

* webpack 配置是 JavaScript 代码构成的, 有很多方法可以管理它。
* 你应该根据实际情况选择配置方法。[webpack-merge](https://www.npmjs.com/package/webpack-merge)为了提供一种轻松的合成方法，但还有许多其他选择。
* Webpack 的 `--env` 参数允许你通过终端控制配置。你通过一个函数接口收到传递的`env`。
* “组合”方便配置共享。您无需在每个仓库维护自定义配置，而是可以通过这种方式在仓库之间共享它。可以通过 npm 包做到这点。开发配置其实跟真正的程序开发差不多了。现在，你可以把你的实践打包上传。

本书的下一部分介绍了不同的技术，* webpack.parts.js *会看到很多动作。幸运的是，* webpack.config.js *的更改仍然很少。
The next parts of this book cover different techniques, and *webpack.parts.js* sees a lot of action as a result. The changes to *webpack.config.js*, fortunately, remain minimal.

