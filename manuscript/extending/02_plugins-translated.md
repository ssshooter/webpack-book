＃使用插件扩展
# Extending with Plugins

与加载器相比，插件是一种更灵活的扩展webpack的方法。您可以访问webpack的**编译器**和**编译**进程。可以运行子编译器，插件可以与加载器一起工作，如“MiniCssExtractPlugin”所示。
Compared to loaders, plugins are a more flexible means to extend webpack. You have access to webpack's **compiler** and **compilation** processes. It's possible to run child compilers, and plugins can work in tandem with loaders as `MiniCssExtractPlugin` shows.

插件允许您通过挂钩拦截webpack的执行。 Webpack本身已经实现为插件集合。它下面依赖于[tapable]（https://www.npmjs.com/package/tapable）插件接口，允许webpack以不同的方式应用插件。
Plugins allow you to intercept webpack's execution through hooks. Webpack itself has been implemented as a collection of plugins. Underneath it relies on [tapable](https://www.npmjs.com/package/tapable) plugin interface that allows webpack to apply plugins in different ways.

接下来你将学会开发几个小插件。与加载器不同，没有单独的环境可以运行插件，因此您必须针对webpack本身运行它们。但是，可以将较小的逻辑推到面向webpack的部分之外，因为这允许您单独对其进行单元测试。
You'll learn to develop a couple of small plugins next. Unlike for loaders, there is no separate environment where you can run plugins, so you have to run them against webpack itself. It's possible to push smaller logic outside of the webpack facing portion, though, as this allows you to unit test it in isolation.

## Webpack插件的基本流程
## The Basic Flow of Webpack Plugins

Webpack插件应该公开`apply（编译器）`方法。 JavaScript允许多种方式来执行此操作。你可以使用一个函数，然后将方法附加到它的`prototype`。要遵循最新的语法，您可以使用`class`来建模相同的想法。
A webpack plugin is expected to expose an `apply(compiler)` method. JavaScript allows multiple ways to do this. You could use a function and then attach methods to its `prototype`. To follow the newest syntax, you could use a `class` to model the same idea.

无论您采用何种方法，都应捕获用户在构造函数中传递的可能选项。声明一个模式以将它们传达给用户是个好主意。 [schema-utils]（https://www.npmjs.com/package/schema-utils）允许验证并与加载器一起使用。
Regardless of your approach, you should capture possible options passed by a user at the constructor. It's a good idea to declare a schema to communicate them to the user. [schema-utils](https://www.npmjs.com/package/schema-utils) allows validation and works with loaders too.

当插件连接到webpack配置时，webpack将运行其构造函数并使用传递给它的编译器对象调用`apply`。该对象公开了webpack的插件API，并允许您使用[官方编译器参考]（https://webpack.js.org/api/plugins/compiler/）列出的钩子。
When the plugin is connected to webpack configuration, webpack will run its constructor and call `apply` with a compiler object passed to it. The object exposes webpack's plugin API and allows you to use its hooks as listed by [the official compiler reference](https://webpack.js.org/api/plugins/compiler/).

T> [webpack-defaults]（https://www.npmjs.com/package/webpack-defaults）是webpack插件的起点。它包含用于开发官方webpack加载器和插件的基础结构。
T> [webpack-defaults](https://www.npmjs.com/package/webpack-defaults) works as a starting point for webpack plugins. It contains the infrastructure used to develop official webpack loaders and plugins.

##设置开发环境
## Setting Up a Development Environment

由于插件必须针对webpack运行，因此您必须设置一个以运行将进一步开发的演示插件：
Since plugins have to be run against webpack, you have to set up one to run a demo plugin that will be developed further:

** ** webpack.plugin.js
**webpack.plugin.js**

```javascript
const path = require("path");
const DemoPlugin = require("./plugins/demo-plugin.js");

const PATHS = {
  lib: path.join(__dirname, "app", "shake.js"),
  build: path.join(__dirname, "build"),
};

module.exports = {
  entry: {
    lib: PATHS.lib,
  },
  output: {
    path: PATHS.build,
    filename: "[name].js",
  },
  plugins: [new DemoPlugin()],
};
```

T>如果您还没有设置`lib`条目文件，请写一个。只要是webpack可以解析的JavaScript，内容就无所谓了。
T> If you don't have a `lib` entry file set up yet, write one. The contents don't matter as long as it's JavaScript that webpack can parse.

为方便运行，请设置构建快捷方式：
To make it convenient to run, set up a build shortcut:

** **的package.json
**package.json**

```json
"scripts": {
leanpub-start-insert
  "build:plugin": "webpack --config webpack.plugin.js",
leanpub-end-insert
  ...
},
```

执行它应该导致“错误：无法找到模块”失败，因为实际的插件仍然缺失。
Executing it should result in an `Error: Cannot find module` failure as the actual plugin is still missing.

T>如果您想要一个交互式开发环境，请考虑针对构建设置[nodemon]（https://www.npmjs.com/package/nodemon）。在这种情况下，Webpack的观察者将无法工作。
T> If you want an interactive development environment, consider setting up [nodemon](https://www.npmjs.com/package/nodemon) against the build. Webpack's watcher won't work in this case.

##实现基本插件
## Implementing a Basic Plugin

最简单的插件应该做两件事：捕获选项并提供`apply`方法：
The simplest plugin should do two things: capture options and provide `apply` method:

**插件/演示plugin.js **
**plugins/demo-plugin.js**

```javascript
module.exports = class DemoPlugin {
  apply() {
    console.log("applying");
  }
};
```

如果你运行插件（`npm run build：plugin`），你应该在控制台看到`applied`消息。鉴于大多数插件都接受选项，捕获它们并将它们传递给“apply”是个好主意。
If you run the plugin (`npm run build:plugin`), you should see `applying` message at the console. Given most plugins accept options, it's a good idea to capture those and pass them to `apply`.

{pagebreak}

##捕获选项
## Capturing Options

可以通过`constructor`捕获选项：
Options can be captured through a `constructor`:

**插件/演示plugin.js **
**plugins/demo-plugin.js**

```javascript
module.exports = class DemoPlugin {
  constructor(options) {
    this.options = options;
  }
  apply() {
    console.log("apply", this.options);
  }
};
```

现在运行插件将导致“apply undefined”类型的消息，因为没有传递任何选项。
Running the plugin now would result in `apply undefined` kind of message given no options were passed.

调整配置以传递选项：
Adjust the configuration to pass an option:

** ** webpack.plugin.js
**webpack.plugin.js**

```javascript
module.exports = {
  ...
leanpub-start-delete
  plugins: [new DemoPlugin()],
leanpub-end-delete
leanpub-start-insert
  plugins: [new DemoPlugin({ name: "demo" })],
leanpub-end-insert
  ],
};
```

现在你应该在运行后看到`apply {name：'demo'}`。
Now you should see `apply { name: 'demo' }` after running.

{pagebreak}

##理解编译器和编译
## Understanding Compiler and Compilation

`apply`接收webpack的编译器作为参数。调整如下：
`apply` receives webpack's compiler as a parameter. Adjust as below:

**插件/演示plugin.js **
**plugins/demo-plugin.js**

```javascript
module.exports = class DemoPlugin {
  constructor(options) {
    this.options = options;
  }
  apply(compiler) {
    console.log(compiler);
  }
};
```

运行后，您应该会看到很多数据。特别是`options`应该看起来很熟悉，因为它包含webpack配置。您还可以看到熟悉的名称，如`records`。
After running, you should see a lot of data. Especially `options` should look familiar as it contains webpack configuration. You can also see familiar names like `records`.

如果你浏览webpack的[插件开发文档]（https://webpack.js.org/api/plugins/），你会发现编译器提供了大量的钩子。每个钩子对应一个特定的阶段。例如，要发出文件，您可以监听`emit`事件然后写。
If you go through webpack's [plugin development documentation](https://webpack.js.org/api/plugins/), you'll see a compiler provides a large number of hooks. Each hook corresponds to a specific stage. For example, to emit files, you could listen to the `emit` event and then write.

更改实现以侦听和捕获`编译`：
Change the implementation to listen and capture `compilation`:

**插件/演示plugin.js **
**plugins/demo-plugin.js**

```javascript
module.exports = class DemoPlugin {
  constructor(options) {
    this.options = options;
  }
  apply(compiler) {
leanpub-start-delete
    console.log(compiler);
leanpub-end-delete
leanpub-start-insert
    compiler.plugin("emit", (compilation, cb) => {
      console.log(compilation);

      cb();
    });
leanpub-end-insert
  }
};
```

W>忘记回调并运行插件会让webpack无声地失败！
W> Forgetting the callback and running the plugin makes webpack fail silently!

运行构建应该显示比以前更多的信息，因为编译对象包含webpack遍历的整个依赖关系图。您可以访问与此相关的所有内容，包括条目，块，模块，资产等。
Running the build should show more information than before because a compilation object contains whole dependency graph traversed by webpack. You have access to everything related to it here including entries, chunks, modules, assets, and more.

T>许多可用的钩子暴露了编译，但有时它们揭示了更具体的结构，并且需要更具体的研究才能理解这些。
T> Many of the available hooks expose compilation, but sometimes they reveal a more specific structure, and it takes a more particular study to understand those.

T> Loaders通过下划线（`this._compiler` /`this._compilation`）对`compiler`和`compilation`进行脏访问。
T> Loaders have dirty access to `compiler` and `compilation` through underscore (`this._compiler`/`this._compilation`).

##通过编译编写文件
## Writing Files Through Compilation

编译的`assets`对象可用于编写新文件。您还可以捕获已创建的资产，对其进行操作并将其写回。
The `assets` object of compilation can be used for writing new files. You can also capture already created assets, manipulate them, and write them back.

要编写资产，您必须使用[webpack-sources]（https://www.npmjs.com/package/webpack-sources）文件抽象。首先安装它：
To write an asset, you have to use [webpack-sources](https://www.npmjs.com/package/webpack-sources) file abstraction. Install it first:

```bash
npm install webpack-sources --save-dev
```

{pagebreak}

调整代码如下，以通过`RawSource`写入：
Adjust the code as follows to write through `RawSource`:

**插件/演示plugin.js **
**plugins/demo-plugin.js**

```javascript
leanpub-start-insert
const { RawSource } = require("webpack-sources");
leanpub-end-insert

module.exports = class DemoPlugin {
  constructor(options) {
    this.options = options;
  }
  apply(compiler) {
leanpub-start-insert
    const { name } = this.options;
leanpub-end-insert

    compiler.plugin("emit", (compilation, cb) => {
leanpub-start-delete
      console.log(compilation);
leanpub-end-delete
leanpub-start-insert
      compilation.assets[name] = new RawSource("demo");
leanpub-end-insert

      cb();
    });
  }
};
```

构建之后，您应该看到输出：
After building, you should see output:

```bash
Hash: d698e1dab6472ba42525
Version: webpack 3.8.1
Time: 51ms
 Asset     Size  Chunks             Chunk Names
lib.js   2.9 kB       0  [emitted]  lib
  demo  4 bytes          [emitted]
   [0] ./app/shake.js 107 bytes {0} [built]
```

如果您检查* build / demo *文件，您将看到它包含* demo *，如上面的代码所示。
If you examine *build/demo* file, you'll see it contains the word *demo* as per code above.

T> Compilation有一组自己的钩子，如[官方编译参考]（https://webpack.js.org/api/plugins/compiler/）所述。
T> Compilation has a set of hooks of its own as covered in [the official compilation reference](https://webpack.js.org/api/plugins/compiler/).

##管理警告和错误
## Managing Warnings and Errors

可以通过抛出（`throw new Error（“Message”）`）使插件执行失败。如果验证选项，则可以使用此方法。
Plugin execution can be caused to fail by throwing (`throw new Error("Message")`). If you validate options, you can use this method.

如果您想在编译期间向用户发出警告或错误消息，您应该使用`compilation.warnings`和`compilation.errors`。例：
In case you want to give the user a warning or an error message during compilation, you should use `compilation.warnings` and `compilation.errors`. Example:

```javascript
compilation.warnings.push("warning");
compilation.errors.push("error");
```

虽然有[日志提案]（https://github.com/webpack/webpack/issues/3996），但是没有办法将信息消息传递给webpack。如果你想为此使用`console.log`，请将它推到`verbose`标志后面。问题是`console.log`将打印到stdout，结果将最终出现在webpack的`--json`输出中。标志将允许用户解决此问题。
There is no way pass information messages to webpack yet although there is [a logging proposal](https://github.com/webpack/webpack/issues/3996). If you want to use `console.log` for this purpose, push it behind a `verbose` flag. The problem is that `console.log` will print to stdout and it will end up in webpack's `--json` output as a result. A flag will allow the user to work around this problem.

##插件可以有插件
## Plugins Can Have Plugins

插件可以提供自己的钩子。 [html-webpack-plugin]（https://www.npmjs.com/package/html-webpack-plugin）使用插件进行扩展，如*入门*章节中所述。
A plugin can provide hooks of its own. [html-webpack-plugin](https://www.npmjs.com/package/html-webpack-plugin) uses plugins to extend itself as discussed in the *Getting Started* chapter.

##插件可以运行自己的编译器
## Plugins Can Run Compilers of Their Own

在特殊情况下，如[offline-plugin]（https://www.npmjs.com/package/offline-plugin），运行子编译器是有意义的。它完全控制相关的条目和输出。该插件的作者Arthur Stolyar解释了[Stack Overflow中的子编译器的想法]（https://stackoverflow.com/questions/38276028/webpack-child-compiler-change-configuration）。
In special cases, like [offline-plugin](https://www.npmjs.com/package/offline-plugin), it makes sense to run a child compiler. It gives full control over related entries and output. Arthur Stolyar, the author of the plugin, has explained [the idea of child compilers at Stack Overflow](https://stackoverflow.com/questions/38276028/webpack-child-compiler-change-configuration).

##结论
## Conclusion

当您开始设计插件时，花时间研究足够接近的现有插件。分段开发插件，以便您一次验证一件。研究webpack源代码可以提供更多的洞察力，因为它是一组插件本身。
When you begin to design a plugin, spend time studying existing plugins that are close enough. Develop plugins piece-wise so that you validate one piece at a time. Studying webpack source can give more insight given it's a collection of plugins itself.

回顾一下：
To recap:

* **插件**可以拦截webpack的执行并扩展它，使它们比装载机更灵活。
* **Plugins** can intercept webpack's execution and extend it making them more flexible than loaders.
*插件可与装载机组合使用。 `MiniCssExtractPlugin`就是这样的。随附的加载程序用于标记要提取的资产。
* Plugins can be combined with loaders. `MiniCssExtractPlugin` works this way. The accompanying loader is used to mark assets to extract.
*插件可以访问webpack的**编译器**和**编译**进程。两者都为webpack的执行流程的不同阶段提供了钩子，并允许您操作它。 Webpack本身就是这样运作的。
* Plugins have access to webpack's **compiler** and **compilation** processes. Both provide hooks for different stages of webpack's execution flow and allow you to manipulate it. Webpack itself works this way.
*插件可以发出新资产并塑造现有资产。
* Plugins can emit new assets and shape existing assets.
*插件可以实现自己的插件系统。 `HtmlWebpackPlugin`就是这种插件的一个例子。
* Plugins can implement plugin systems of their own. `HtmlWebpackPlugin` is an example of such plugin.
*插件可以自己运行编译器。隔离提供了更多控制，并允许编写像* offline-plugin *这样的插件。
* Plugins can run compilers on their own. The isolation gives more control and allows plugins like *offline-plugin* to be written.

