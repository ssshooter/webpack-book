# 加载 JavaScript

Webpack 默认处理 ES2015 模块并将其转换为浏览器可用代码。但它**不会**转换大部分语法，例如 `const`，在旧版浏览器中可能会出现问题。

为了更好地了解 Webpack 默认转换，请看以下输出（`npm run build -- --devtool false --mode development`）：

**dist/main.js**

```javascript
...
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
__webpack_require__.r(__webpack_exports__);
/* harmony default export */ __webpack_exports__["default"] = ((text = "Hello world") => {
  const element = document.createElement("div");

  element.className = "pure-button";
  element.innerHTML = text;

  return element;
});
...
```

[Babel](https://babeljs.io/) 是一个以支持 ES2015+ 功能着名的 JavaScript 编译器，它处理上述问题。它类似于 ESLint，建立在预设和插件之上。预设是一系列插件的集合，你也可以定义自己的插件。

T> [modify-babel-preset](https://www.npmjs.com/package/modify-babel-preset) 能以更灵活的方式配置基本预设。

## 在 Webpack 中使用 Babel

尽管 Babel 可以单独使用，正如你在 **SurviveJS - Maintenance** 一书中所看到的，你也可以将其与 webpack 一起使用。在开发过程中，如果你使用浏览器本身就支持你使用的新语法，那么完全可以跳过 Babel 处理。

但是，当你编译生产代码时，为了你的大多数用户能正常使用，Babel 处理几乎是必需的。

你可以通过 [babel-loader](https://www.npmjs.com/package/babel-loader) 在 webpack 中使用 Babel。它可以获取项目级别的 Babel 配置（译者注：dotfile），或者你可以在 webpack loader 中进行配置。[babel-webpack-plugin](https://www.npmjs.com/package/babel-webpack-plugin) 是另一个鲜为人知的选择。

Babel 还可以处理 webpack 配置（译者注：配置本身，区别于需要打包的内容）。要实现此目的，请把你的 webpack 配置命名为 *webpack.config.babel.js*。这依赖于 [interpret](https://www.npmjs.com/package/interpret)，它也支持其他编译器。

T> 最近 [Node 支持了 ES2015 规范](http://node.green/)，你可以无需通过 Babel 处理配置在 Node 使用许多 ES2015 功能。

W> 如果你使用 **webpack.config.babel.js**，请注意 `"modules": false,` 这一项配置。如果要使用 ES2015 模块，可以跳过全局 Babel 配置中的设置，然后按照下面的讨论为每个环境配置它。

{pagebreak}

### 配置 **babel-loader**

在 webpack 使用 Babel 的第一步是设置 [babel-loader](https://www.npmjs.com/package/babel-loader)。babel-loader 会把代码转换为旧浏览器可以运行的格式。安装 **babel-loader** 并包含其同版本依赖（Peer Dependencies） **babel-core**：

```bash
npm install babel-loader babel-core --save-dev
```

与之前一样，先定义一个 Babel 配置函数：

**webpack.parts.js**

```javascript
exports.loadJavaScript = ({ include, exclude } = {}) => ({
  module: {
    rules: [
      {
        test: /\.js$/,
        include,
        exclude,
        use: "babel-loader",
      },
    ],
  },
});
```

接下来，你需要将其添加到主配置。如果你使用现代浏览器进行开发，则可以考虑仅在生产环境使用 Babel。以下配置是两个环境都使用 Babel，并且只有特定路径接受 Babel 处理。

{pagebreak}

作如下调整：

**webpack.config.js**

```javascript
const commonConfig = merge([
  ...
leanpub-start-insert
  parts.loadJavaScript({ include: PATHS.app }),
leanpub-end-insert
]);
```

这样你用上了 Babel，但还缺少一点东西：Babel 配置。我们可以使用 *.babelrc* dotfile，这个设置文件也适用于其他工具。

W> 如果你尝试导入配置根目录**之外的文件**然后通过 **babel-loader** 处理它们，则会失败。这是[已知问题](https://github.com/babel/babel-loader/issues/313)，解决方法是，包括在项目的更高级别维护 **.babelrc** 并通过解决Babel预设在webpack配置中的 `require.resolve`。
W> If you try to import files **outside** of your configuration root directory and then process them through *babel-loader*, this fails. It's [a known issue](https://github.com/babel/babel-loader/issues/313), and there are workarounds including maintaining *.babelrc* at a higher level in the project and resolving against Babel presets through `require.resolve` at webpack configuration.

### 关于 **.babelrc**

至少，你需要 [babel-preset-env](https://www.npmjs.com/package/babel-preset-env)。它是一个 Babel 预设，可根据你传递给它的可选环境定义启用所需的插件。

首先安装预设：

```bash
npm install babel-preset-env --save-dev
```

要让 Babel 使用预设，需要写一个 **.babelrc**。鉴于 webpack 支持开箱即用的 ES2015 模块，你可以让 Babel 跳过处理它们。跳过这一步将破坏 webpack 的 HMR 机制，尽管生产构建仍然有效。你还可以限制构建输出仅在最新版本的 Chrome 中有效。

根据需要调整目标浏览器，只要你按照 [browserslist](https://www.npmjs.com/package/browserslist) 进行操作即可。这是一个示例配置：

**.babelrc**

```json
{
  "presets": [
    [
      "env",
      {
        "modules": false,
      }
    ]
  ]
}
```

现在执行 `npm run build -- --devtool false --mode development` 并查看 **dist/main.js**，你会看到基于你的 `.browserslistrc` 文件的不同内容。

{pagebreak}

尝试在那里只包含 `IE 8` 这样的定义，代码应该相应地改变：
Try to include only a definition like `IE 8` there, and the code should change accordingly:

**dist/main.js**

```javascript
...
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
__webpack_require__.r(__webpack_exports__);
/* harmony default export */ __webpack_exports__["default"] = (function () {
  var text = arguments.length > 0 && arguments[0] !== undefined ? arguments[0] : "Hello world";

  var element = document.createElement("div");

  element.className = "pure-button";
  element.innerHTML = text;

  return element;
});
...
```

特别注意函数是如何转换的。你可以尝试不同的浏览器定义和语言功能，以查看输出如何根据选择进行更改。
Note especially how the function was transformed. You can try out different browser definitions and language features to see how the output changes based on the selection.

## Polyfill

**babel-preset-env** 为旧版浏览器“填充”（polyfill）某些语言功能。为此，你应该启用它的 `useBuiltIns` 选项（`"useBuiltIns": true`）并安装[babel-polyfill](https://babeljs.io/docs/usage/polyfill/)。你必须通过导入或条目将其包含在项目中（`app: ["babel-polyfill", PATHS.app]`）。*babel-preset-env* 根据你的“浏览器文件”重写导入，并仅加载所需的 polyfill。
*babel-preset-env* allows you to polyfill certain language features for older browsers. For this to work, you should enable its `useBuiltIns` option (`"useBuiltIns": true`) and install [babel-polyfill](https://babeljs.io/docs/usage/polyfill/). You have to include it in your project either through an import or an entry (`app: ["babel-polyfill", PATHS.app]`). *babel-preset-env* rewrites the import based on your browser definition and loads only the polyfills that are needed.

**babel-polyfill** 使用“Promise”等对象污染全局范围。鉴于这对图书馆作者来说可能有问题，那就是[transform-runtime]（https://babeljs.io/docs/plugins/transform-runtime/）选项。它可以作为Babel插件启用，它通过以不需要它们的方式重写代码来避免全局变量的问题。
*babel-polyfill* pollutes the global scope with objects like `Promise`. Given this can be problematic for library authors, there's [transform-runtime](https://babeljs.io/docs/plugins/transform-runtime/) option. It can be enabled as a Babel plugin, and it avoids the problem of globals by rewriting the code in such way that they aren't be needed.

W> 某些webpack功能，例如**代码拆分**，在webpack处理加载器之后，将基于“Promise”的代码写入webpack的引导程序。在执行应用程序代码之前应用填充程序可以解决该问题。示例：`entry: { app: ["core-js/es6/promise", PATHS.app] }`。
W> Certain webpack features, such as *Code Splitting*, write `Promise` based code to webpack's bootstrap after webpack has processed loaders. The problem can be solved by applying a shim before your application code is executed. Example: `entry: { app: ["core-js/es6/promise", PATHS.app] }`.

## Babel 小贴士

除此处介绍的内容之外，还有其他可能的 [**.babelrc** 选项](https://babeljs.io/docs/usage/options/)。与 ESLint 一样，**.babelrc** 支持 [JSON5](https://www.npmjs.com/package/json5) 作为其配置格式，这意味着文件中可以包含注释，使用单引号字符串等。

有时你希望使用适合你项目的实验性功能。虽然你可以在所谓的舞台预设中找到很多它们，但最好逐个启用它们，甚至将它们组织成自己的预设，除非你正在进行一次性项目。如果你希望项目能够存活很长时间，那么最好记录你正在使用的功能。
Sometimes you want to use experimental features that fit your project. Although you can find a lot of them within so-called stage presets, it's a good idea to enable them one by one and even organize them to a preset of their own unless you are working on a throwaway project. If you expect your project to live a long time, it's better to document the features you are using well.

尽管 Babel 很受欢迎，但它并非唯一选择。Rich Harris 的 [Buble](https://buble.surge.sh) 是另一个值得一试的编译器。借助实验性的 [buble-loader](https://www.npmjs.com/package/buble-loader) 可以在 webpack 中使用它。Buble 不支持 ES2015 模块，但问题不大，因为 webpack 提供了这个功能。

{pagebreak}

## Babel 插件

Babel 的最大优势之一是可以使用插件进行扩展：

* [babel-plugin-import]（https://www.npmjs.com/package/babel-plugin-import）重写模块导入，以便你可以使用“antd”中的`import {Button}等形式;而不是通过精确的路径指向模块。
* [babel-plugin-import](https://www.npmjs.com/package/babel-plugin-import) rewrites module imports so that you can use a form such as `import { Button } from "antd";` instead of pointing to the module through an exact path.
* [babel-plugin-import-asserts]（https://www.npmjs.com/package/babel-plugin-import-asserts）断言你的导入已定义。
* [babel-plugin-import-asserts](https://www.npmjs.com/package/babel-plugin-import-asserts) asserts that your imports have been defined.
* [babel-plugin-jsdoc-to-assert]（https://www.npmjs.com/package/babel-plugin-jsdoc-to-assert）转换[JSDoc]（http://usejsdoc.org/）注释可运行的断言。
* [babel-plugin-jsdoc-to-assert](https://www.npmjs.com/package/babel-plugin-jsdoc-to-assert) converts [JSDoc](http://usejsdoc.org/) annotations to runnable assertions.
* [babel-plugin-log-deprecated]（https://www.npmjs.com/package/babel-plugin-log-deprecated）将`console.warn`添加到注释中带有@@ deprecate`注释的函数中。
* [babel-plugin-log-deprecated](https://www.npmjs.com/package/babel-plugin-log-deprecated) adds `console.warn` to functions that have `@deprecate` annotation in their comment.
* [babel-plugin-annotate-console-log]（https://www.npmjs.com/package/babel-plugin-annotate-console-log）使用有关调用上下文的信息注释`console.log`调用，所以它是更容易看到他们记录的位置。
* [babel-plugin-annotate-console-log](https://www.npmjs.com/package/babel-plugin-annotate-console-log) annotates `console.log` calls with information about invocation context, so it's easier to see where they logged.
* [babel-plugin-sitrep]（https://www.npmjs.com/package/babel-plugin-sitrep）记录函数的所有赋值并打印它们。
* [babel-plugin-sitrep](https://www.npmjs.com/package/babel-plugin-sitrep) logs all assignments of a function and prints them.
* [babel-plugin-webpack-loaders]（https://www.npmjs.com/package/babel-plugin-webpack-loaders）允许你通过Babel使用某些webpack加载器。
* [babel-plugin-webpack-loaders](https://www.npmjs.com/package/babel-plugin-webpack-loaders) allows you to use certain webpack loaders through Babel.
* [babel-plugin-syntax-trailing-function-commas]（https://www.npmjs.com/package/babel-plugin-syntax-trailing-function-commas）为函数添加尾随逗号支持。
* [babel-plugin-syntax-trailing-function-commas](https://www.npmjs.com/package/babel-plugin-syntax-trailing-function-commas) adds trailing comma support for functions.
* [babel-plugin-transform-react-remove-prop-types]（https://www.npmjs.com/package/babel-plugin-transform-react-remove-prop-types）允许你删除`propType`生产版本中的相关代码。它还允许组件作者生成包装的代码，以便“DefinePlugin”的设置环境可以按照书中的讨论启动。
* [babel-plugin-transform-react-remove-prop-types](https://www.npmjs.com/package/babel-plugin-transform-react-remove-prop-types) allows you to remove `propType` related code from your production build. It also allows component authors to generate code that's wrapped so that setting environment at `DefinePlugin` can kick in as discussed in the book.

T> 可以通过[babel-register]（https://www.npmjs.com/package/babel-register）或[babel-cli]（https://www.npmjs.com/package）将Babel与Node连接起来/巴别-CLI）。如果你想在不使用webpack的情况下通过Babel执行代码，这些包可以很方便。
T> It's possible to connect Babel with Node through [babel-register](https://www.npmjs.com/package/babel-register) or [babel-cli](https://www.npmjs.com/package/babel-cli). These packages can be handy if you want to execute your code through Babel without using webpack.

## 根据环境启用预设和插件

Babel 可以通过其 [env 选项](https://babeljs.io/docs/usage/babelrc/#env-option) 控制每个环境使用哪些预设和插件。你可以通过这种方式管理不同构建中 Babel 的行为。

`env` 检查 `NODE_ENV` 和 `BABEL_ENV` 并根据它为你的构建添加功能。如果设置了 `BABEL_ENV`，它将覆盖任何可能的`NODE_ENV`。
`env` checks both `NODE_ENV` and `BABEL_ENV` and adds functionality to your build based on that. If `BABEL_ENV` is set, it overrides any possible `NODE_ENV`.

思考以下代码：

**.babelrc**

```json
{
  ...
  "env": {
    "development": {
      "plugins": [
        "annotate-console-log"
      ]
    }
  }
}
```

任何共享预设和插件仍可供所有目标使用。 `env`允许你进一步专门化你的Babel配置。
Any shared presets and plugins are available to all targets still. `env` allows you to specialize your Babel configuration further.

{pagebreak}

可以将 webpack 的环境传递给 Babel：

**webpack.config.js**

```javascript
module.exports = mode => {
leanpub-start-insert
  process.env.BABEL_ENV = mode;
leanpub-end-insert

  ...
};
```

T> `env` 的工作方式很微妙。考虑记录`env`并确保它与你的Babel配置匹配，否则你期望的功能不会应用于你的构建。
T> The way `env` works is subtle. Consider logging `env` and make sure it matches your Babel configuration or otherwise the functionality you expect is not applied to your build.

## 使用 TypeScript

微软的 [TypeScript](http://www.typescriptlang.org/) 是一种编译语言，遵循与Babel类似的设置。整洁的是，除了JavaScript之外，它还可以发出类型定义。一个好的编辑可以选择那些并提供增强的编辑体验。更强的打字对于开发很有价值，因为更容易说明你的类型合同。
Microsoft's [TypeScript](http://www.typescriptlang.org/) is a compiled language that follows a similar setup as Babel. The neat thing is that in addition to JavaScript, it can emit type definitions. A good editor can pick those up and provide enhanced editing experience. Stronger typing is valuable for development as it becomes easier to state your type contracts.

与 Facebook 的类型检查器 Flow 相比，TypeScript 是一种更安全的选择。因此，你可以找到更多预制的类型定义，总体而言，支持的质量应该更好。
Compared to Facebook's type checker Flow, TypeScript is a more secure option. As a result, you find more premade type definitions for it, and overall, the quality of support should be better.

你可以使用以下 loader 在 webpack 中使用 TypeScript：

* [ts-loader](https://www.npmjs.com/package/ts-loader)
* [awesome-typescript-loader](https://www.npmjs.com/package/awesome-typescript-loader)

T> 有一个[ESLint的TypeScript解析器]（https://www.npmjs.com/package/typescript-eslint-parser）。它也可以通过[tslint]（https://www.npmjs.com/package/tslint）来提取它。
T> There's a [TypeScript parser for ESLint](https://www.npmjs.com/package/typescript-eslint-parser). It's also possible to lint it through [tslint](https://www.npmjs.com/package/tslint).

## 使用 Flow

[Flow]（https://flow.org/）根据你的代码及其类型注释执行静态分析。你必须将其作为单独的工具安装，然后针对你的代码运行它。有[flow-status-webpack-plugin]（https://www.npmjs.com/package/flow-status-webpack-plugin），允许你在开发期间通过webpack运行它。
[Flow](https://flow.org/) performs static analysis based on your code and its type annotations. You have to install it as a separate tool and then run it against your code. There's [flow-status-webpack-plugin](https://www.npmjs.com/package/flow-status-webpack-plugin) that allows you to run it through webpack during development.

如果你使用React，则React特定Babel预设通过[babel-plugin-syntax-flow]（https://www.npmjs.com/package/babel-plugin-syntax-flow）完成大部分工作。它可以删除Flow注释并将你的代码转换为可以进一步转换的格式。
If you use React, the React specific Babel preset does most of the work through [babel-plugin-syntax-flow](https://www.npmjs.com/package/babel-plugin-syntax-flow). It can strip Flow annotations and convert your code into a format that is possible to transpile further.

还有[babel-plugin-typecheck]（https://www.npmjs.com/package/babel-plugin-typecheck），它允许你根据Flow注释执行运行时检查。 [flow-runtime]（https://codemix.github.io/flow-runtime/）进一步提升并提供更多功能。这些方法补充了Flow静态检查程序，使你可以捕获更多问题。
There's also [babel-plugin-typecheck](https://www.npmjs.com/package/babel-plugin-typecheck) that allows you to perform runtime checks based on your Flow annotations. [flow-runtime](https://codemix.github.io/flow-runtime/) goes a notch further and provides more functionality. These approaches complement Flow static checker and allow you to catch even more issues.

T> [flow-coverage-report](https://www.npmjs.com/package/flow-coverage-report) 可以检测 Flow 类型的覆盖率。

{pagebreak}

## 总结

Babel 已经成为开发人员不可或缺的工具，它可以转换 JavaScript 代码，使其在老旧浏览器正常运行。即使针对现代浏览器，通过 Babel 进行转换也是一种选择。

回顾一下：

* Babel 可以让你选择需要支持的浏览器。它可以将 ES2015+ 特性编译为可以在旧浏览器运行的形识。**babel-preset-env** 很有价值，因为它可以根据你的 `.browserslistrc` 文件选择需要编译的新特性和要启用的 polyfill。
* Babel 允许你使用实验性 feature。你可以找到许多通过优化改进开发体验和生产构建的插件。
* Babel allows you to use experimental language features. You can find numerous plugins that improve development experience and the production build through optimizations.
* 可以为每个开发目标启用 Babel 功能。这样，你可以确保在正确的位置使用正确的插件。
* Babel functionality can be enabled per development target. This way you can be sure you are using the correct plugins at the right place.
* 除了 Babel 之外，webpack 还支持 TypeScript 或 Flow 等其他解决方案。 Flow 可以补充 Babel，而 TypeScript 代表编译为 JavaScript 的整个语言。
* Besides Babel, webpack supports other solutions like TypeScript or Flow. Flow can complement Babel while TypeScript represents an entire language compiling to JavaScript.

