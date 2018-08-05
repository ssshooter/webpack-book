＃加载JavaScript
# Loading JavaScript

Webpack默认处理ES2015模块定义并将其转换为代码。但它不会**转换特定的语法，例如`const`。生成的代码可能会出现问题，尤其是在旧版浏览器中。
Webpack processes ES2015 module definitions by default and transforms them into code. It does **not** transform specific syntax, such as `const`, though. The resulting code can be problematic especially in the older browsers.

为了更好地了解默认转换，请考虑下面的示例输出（`npm run build --devtool false --mode development`）：
To get a better idea of the default transform, consider the example output below (`npm run build -- --devtool false --mode development`):

** DIST / main.js **
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

通过[Babel]（https://babeljs.io/）处理代码可以解决这个问题，这是一个支持ES2015 +功能的着名JavaScript编译器等等。它类似于ESLint，因为它建立在预设和插件之上。预设是插件的集合，您也可以定义自己的插件。
The problem can be worked around by processing the code through [Babel](https://babeljs.io/), a famous JavaScript compiler that supports ES2015+ features and more. It resembles ESLint in that it's built on top of presets and plugins. Presets are collections of plugins, and you can define your own as well.

T>鉴于有时扩展现有的预设是不够的，[modify-babel-preset]（https://www.npmjs.com/package/modify-babel-preset）允许您更进一步，并配置基本预设一种更灵活的方式。
T> Given sometimes extending existing presets is not enough, [modify-babel-preset](https://www.npmjs.com/package/modify-babel-preset) allows you to go a step further and configure the base preset in a more flexible way.

##将Babel与Webpack配置一起使用
## Using Babel with Webpack Configuration

尽管Babel可以单独使用，正如您在* SurviveJS  -  Maintenance *一书中所看到的，您也可以将其与webpack联系起来。在开发过程中，如果您使用浏览器支持的语言功能，则跳过处理是有意义的。
Even though Babel can be used standalone, as you can see in the *SurviveJS - Maintenance* book, you can hook it up with webpack as well. During development, it can make sense to skip processing if you are using language features supported by your browser.

如果您不依赖任何自定义语言功能并使用现代浏览器工作，则跳过处理是一个不错的选择。但是，当您编译生产代码时，通过Babel处理几乎是必需的。
Skipping processing is a good option primarily if you don't rely on any custom language features and work using a modern browser. Processing through Babel becomes almost a necessity when you compile your code for production, though.

您可以通过[babel-loader]（https://www.npmjs.com/package/babel-loader）将Babel与webpack一起使用。它可以获取项目级别的Babel配置，或者您可以在webpack加载器本身进行配置。 [babel-webpack-plugin]（https://www.npmjs.com/package/babel-webpack-plugin）是另一个鲜为人知的选择。
You can use Babel with webpack through [babel-loader](https://www.npmjs.com/package/babel-loader). It can pick up project level Babel configuration, or you can configure it at the webpack loader itself. [babel-webpack-plugin](https://www.npmjs.com/package/babel-webpack-plugin) is another lesser-known option.

将Babel与项目连接允许您通过它处理webpack配置。要实现此目的，请使用* webpack.config.babel.js *约定命名您的webpack配置。 [解释]（https://www.npmjs.com/package/interpret）包支持这个，它也支持其他编译器。
Connecting Babel with a project allows you to process webpack configuration through it. To achieve this, name your webpack configuration using the *webpack.config.babel.js* convention. [interpret](https://www.npmjs.com/package/interpret) package enables this, and it supports other compilers as well.

T>鉴于[Node支持ES2015规范]（http://node.green/）这些天，您可以使用许多ES2015功能，而无需通过Babel处理配置。
T> Given that [Node supports the ES2015 specification well](http://node.green/) these days, you can use a lot of ES2015 features without having to process configuration through Babel.

W>如果你使用* webpack.config.babel.js *，请注意``modules'：false，`setting。如果要使用ES2015模块，可以跳过全局Babel配置中的设置，然后按照下面的讨论为每个环境配置它。
W> If you use *webpack.config.babel.js*, take care with the `"modules": false,` setting. If you want to use ES2015 modules, you could skip the setting in your global Babel configuration and then configure it per environment as discussed below.

{pagebreak}

###设置* babel-loader *
### Setting Up *babel-loader*

配置Babel以使用webpack的第一步是设置[babel-loader]（https://www.npmjs.com/package/babel-loader）。它需要代码并将其转换为旧浏览器可以理解的格式。安装* babel-loader *并包含其对等依赖* babel-core *：
The first step towards configuring Babel to work with webpack is to set up [babel-loader](https://www.npmjs.com/package/babel-loader). It takes the code and turns it into a format older browsers can understand. Install *babel-loader* and include its peer dependency *babel-core*:

```bash
npm install babel-loader babel-core --save-dev
```

像往常一样，让我们​​为Babel定义一个函数：
As usual, let's define a function for Babel:

** ** webpack.parts.js
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

接下来，您需要将其连接到主配置。如果您使用现代浏览器进行开发，则可以考虑通过Babel仅处理生产代码。在这种情况下，它用于生产和开发环境。此外，只有应用程序代码通过Babel处理。
Next, you need to connect this to the main configuration. If you are using a modern browser for development, you can consider processing only the production code through Babel. It's used for both production and development environments in this case. Also, only application code is processed through Babel.

{pagebreak}

调整如下：
Adjust as below:

** ** webpack.config.js
**webpack.config.js**

```javascript
const commonConfig = merge([
  ...
leanpub-start-insert
  parts.loadJavaScript({ include: PATHS.app }),
leanpub-end-insert
]);
```

即使您安装了Babel并进行了设置，您仍然缺少一点：Babel配置。可以使用* .babelrc * dotfile设置配置，因为其他工具可以使用相同的配置。
Even though you have Babel installed and set up, you are still missing one bit: Babel configuration. The configuration can be set up using a *.babelrc* dotfile as then other tooling can use the same.

W>如果您尝试导入配置根目录**之外的文件**然后通过* babel-loader *处理它们，则会失败。这是[已知问题]（https://github.com/babel/babel-loader/issues/313），并且有一些解决方法，包括在项目的更高级别维护* .babelrc *并通过`解决Babel预设在webpack配置中的require.resolve`。
W> If you try to import files **outside** of your configuration root directory and then process them through *babel-loader*, this fails. It's [a known issue](https://github.com/babel/babel-loader/issues/313), and there are workarounds including maintaining *.babelrc* at a higher level in the project and resolving against Babel presets through `require.resolve` at webpack configuration.

###设置* .babelrc *
### Setting Up *.babelrc*

至少，您需要[babel-preset-env]（https://www.npmjs.com/package/babel-preset-env）。它是一个Babel预设，可根据您传递给它的可选环境定义启用所需的插件。
At a minimum, you need [babel-preset-env](https://www.npmjs.com/package/babel-preset-env). It's a Babel preset that enables the required plugins based on the optional environment definition you pass to it.

首先安装预设：
Install the preset first:

```bash
npm install babel-preset-env --save-dev
```

要让Babel知道预设，你需要写一个* .babelrc *。鉴于webpack支持开箱即用的ES2015模块，您可以告诉Babel跳过处理它们。跳过这一步将破坏webpack的HMR机制，尽管生产构建仍然有效。您还可以限制构建输出仅在最新版本的Chrome中有效。
To make Babel aware of the preset, you need to write a *.babelrc*. Given webpack supports ES2015 modules out of the box, you can tell Babel to skip processing them. Jumping over this step would break webpack's HMR mechanism although the production build would still work. You can also constrain the build output to work only in recent versions of Chrome.

根据需要调整目标定义。只要您按照[browserslist]（https://www.npmjs.com/package/browserslist）进行操作即可。这是一个示例配置：
Adjust the target definition as you like. As long as you follow [browserslist](https://www.npmjs.com/package/browserslist), it should work. Here's a sample configuration:

**。** babelrc
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

如果你现在执行`npm run build --devtool false --mode development`并检查* dist / main.js *，你会看到基于你的`.browserslistrc`文件的不同内容。
If you execute `npm run build -- --devtool false --mode development` now and examine *dist/main.js*, you will see something different based on your `.browserslistrc` file.

{pagebreak}

尝试在那里只包含“IE 8”这样的定义，代码应该相应地改变：
Try to include only a definition like `IE 8` there, and the code should change accordingly:

** DIST / main.js **
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

特别注意函数是如何转换的。您可以尝试不同的浏览器定义和语言功能，以查看输出如何根据选择进行更改。
Note especially how the function was transformed. You can try out different browser definitions and language features to see how the output changes based on the selection.

## Polyfilling功能
## Polyfilling Features

* babel-preset-env *允许您为旧版浏览器填充某些语言功能。为此，你应该启用它的`useBuiltIns`选项（`“useBuiltIns”：true`）并安装[babel-polyfill]（https://babeljs.io/docs/usage/polyfill/）。您必须通过导入或条目将其包含在项目中（`app：[“babel-polyfill”，PATHS.app]`）。 * babel-preset-env *根据您的浏览器定义重写导入，并仅加载所需的polyfill。
*babel-preset-env* allows you to polyfill certain language features for older browsers. For this to work, you should enable its `useBuiltIns` option (`"useBuiltIns": true`) and install [babel-polyfill](https://babeljs.io/docs/usage/polyfill/). You have to include it in your project either through an import or an entry (`app: ["babel-polyfill", PATHS.app]`). *babel-preset-env* rewrites the import based on your browser definition and loads only the polyfills that are needed.

* babel-polyfill *使用“Promise”等对象污染全局范围。鉴于这对图书馆作者来说可能有问题，那就是[transform-runtime]（https://babeljs.io/docs/plugins/transform-runtime/）选项。它可以作为Babel插件启用，它通过以不需要它们的方式重写代码来避免全局变量的问题。
*babel-polyfill* pollutes the global scope with objects like `Promise`. Given this can be problematic for library authors, there's [transform-runtime](https://babeljs.io/docs/plugins/transform-runtime/) option. It can be enabled as a Babel plugin, and it avoids the problem of globals by rewriting the code in such way that they aren't be needed.

W>某些webpack功能，例如* Code Splitting *，在webpack处理加载器之后，将基于“Promise”的代码写入webpack的引导程序。在执行应用程序代码之前应用填充程序可以解决该问题。示例：`entry：{app：[“core-js / es6 / promise”，PATHS.app]}`。
W> Certain webpack features, such as *Code Splitting*, write `Promise` based code to webpack's bootstrap after webpack has processed loaders. The problem can be solved by applying a shim before your application code is executed. Example: `entry: { app: ["core-js/es6/promise", PATHS.app] }`.

##巴比伦小贴士
## Babel Tips

除此处介绍的内容之外，还有其他可能的[* .babelrc *选项]（https://babeljs.io/docs/usage/options/）。与ESLint一样，* .babelrc *支持[JSON5]（https://www.npmjs.com/package/json5）作为其配置格式，这意味着您可以在源代码中包含注释，使用单引号字符串等。
There are other possible [*.babelrc* options](https://babeljs.io/docs/usage/options/) beyond the ones covered here. Like ESLint, *.babelrc* supports [JSON5](https://www.npmjs.com/package/json5) as its configuration format meaning you can include comments in your source, use single quoted strings, and so on.

有时您希望使用适合您项目的实验性功能。虽然你可以在所谓的舞台预设中找到很多它们，但最好逐个启用它们，甚至将它们组织成自己的预设，除非你正在进行一次性项目。如果您希望项目能够存活很长时间，那么最好记录您正在使用的功能。
Sometimes you want to use experimental features that fit your project. Although you can find a lot of them within so-called stage presets, it's a good idea to enable them one by one and even organize them to a preset of their own unless you are working on a throwaway project. If you expect your project to live a long time, it's better to document the features you are using well.

Babel并不是唯一的选择，尽管它是最受欢迎的选择。 Rich Harris的[Buble]（https://buble.surge.sh）是另一个值得一试的编译器。有实验[buble-loader]（https://www.npmjs.com/package/buble-loader）允许你在webpack中使用它。 Buble不支持ES2015模块，但这不是问题，因为webpack提供了这种功能。
Babel isn't the only option although it's the most popular one. [Buble](https://buble.surge.sh) by Rich Harris is another compiler worth checking out. There's experimental [buble-loader](https://www.npmjs.com/package/buble-loader) that allows you to use it with webpack. Buble doesn't support ES2015 modules, but that's not a problem as webpack provides that functionality.

{pagebreak}

## Babel插件
## Babel Plugins

关于Babel最棒的事情可能就是可以使用插件进行扩展：
Perhaps the greatest thing about Babel is that it's possible to extend with plugins:

* [babel-plugin-import]（https://www.npmjs.com/package/babel-plugin-import）重写模块导入，以便您可以使用“antd”中的`import {Button}等形式;而不是通过精确的路径指向模块。
* [babel-plugin-import](https://www.npmjs.com/package/babel-plugin-import) rewrites module imports so that you can use a form such as `import { Button } from "antd";` instead of pointing to the module through an exact path.
* [babel-plugin-import-asserts]（https://www.npmjs.com/package/babel-plugin-import-asserts）断言您的导入已定义。
* [babel-plugin-import-asserts](https://www.npmjs.com/package/babel-plugin-import-asserts) asserts that your imports have been defined.
* [babel-plugin-jsdoc-to-assert]（https://www.npmjs.com/package/babel-plugin-jsdoc-to-assert）转换[JSDoc]（http://usejsdoc.org/）注释可运行的断言。
* [babel-plugin-jsdoc-to-assert](https://www.npmjs.com/package/babel-plugin-jsdoc-to-assert) converts [JSDoc](http://usejsdoc.org/) annotations to runnable assertions.
* [babel-plugin-log-deprecated]（https://www.npmjs.com/package/babel-plugin-log-deprecated）将`console.warn`添加到注释中带有@@ deprecate`注释的函数中。
* [babel-plugin-log-deprecated](https://www.npmjs.com/package/babel-plugin-log-deprecated) adds `console.warn` to functions that have `@deprecate` annotation in their comment.
* [babel-plugin-annotate-console-log]（https://www.npmjs.com/package/babel-plugin-annotate-console-log）使用有关调用上下文的信息注释`console.log`调用，所以它是更容易看到他们记录的位置。
* [babel-plugin-annotate-console-log](https://www.npmjs.com/package/babel-plugin-annotate-console-log) annotates `console.log` calls with information about invocation context, so it's easier to see where they logged.
* [babel-plugin-sitrep]（https://www.npmjs.com/package/babel-plugin-sitrep）记录函数的所有赋值并打印它们。
* [babel-plugin-sitrep](https://www.npmjs.com/package/babel-plugin-sitrep) logs all assignments of a function and prints them.
* [babel-plugin-webpack-loaders]（https://www.npmjs.com/package/babel-plugin-webpack-loaders）允许您通过Babel使用某些webpack加载器。
* [babel-plugin-webpack-loaders](https://www.npmjs.com/package/babel-plugin-webpack-loaders) allows you to use certain webpack loaders through Babel.
* [babel-plugin-syntax-trailing-function-commas]（https://www.npmjs.com/package/babel-plugin-syntax-trailing-function-commas）为函数添加尾随逗号支持。
* [babel-plugin-syntax-trailing-function-commas](https://www.npmjs.com/package/babel-plugin-syntax-trailing-function-commas) adds trailing comma support for functions.
* [babel-plugin-transform-react-remove-prop-types]（https://www.npmjs.com/package/babel-plugin-transform-react-remove-prop-types）允许你删除`propType`生产版本中的相关代码。它还允许组件作者生成包装的代码，以便“DefinePlugin”的设置环境可以按照书中的讨论启动。
* [babel-plugin-transform-react-remove-prop-types](https://www.npmjs.com/package/babel-plugin-transform-react-remove-prop-types) allows you to remove `propType` related code from your production build. It also allows component authors to generate code that's wrapped so that setting environment at `DefinePlugin` can kick in as discussed in the book.

T>可以通过[babel-register]（https://www.npmjs.com/package/babel-register）或[babel-cli]（https://www.npmjs.com/package）将Babel与Node连接起来/巴别-CLI）。如果您想在不使用webpack的情况下通过Babel执行代码，这些包可以很方便。
T> It's possible to connect Babel with Node through [babel-register](https://www.npmjs.com/package/babel-register) or [babel-cli](https://www.npmjs.com/package/babel-cli). These packages can be handy if you want to execute your code through Babel without using webpack.

##启用每个环境的预设和插件
## Enabling Presets and Plugins per Environment

Babel允许您通过其[env选项]（https://babeljs.io/docs/usage/babelrc/#env-option）控制每个环境使用哪些预设和插件。您可以通过这种方式管理每个构建目标的Babel行为。
Babel allows you to control which presets and plugins are used per environment through its [env option](https://babeljs.io/docs/usage/babelrc/#env-option). You can manage Babel's behavior per build target this way.

`env`检查`NODE_ENV`和`BABEL_ENV`并根据它为你的构建添加功能。如果设置了'BABEL_ENV`，它将覆盖任何可能的`NODE_ENV`。
`env` checks both `NODE_ENV` and `BABEL_ENV` and adds functionality to your build based on that. If `BABEL_ENV` is set, it overrides any possible `NODE_ENV`.

考虑以下示例：
Consider the example below:

**。** babelrc
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

任何共享预设和插件仍可供所有目标使用。 `env`允许您进一步专门化您的Babel配置。
Any shared presets and plugins are available to all targets still. `env` allows you to specialize your Babel configuration further.

{pagebreak}

可以通过调整将webpack环境传递给Babel：
It's possible to pass the webpack environment to Babel with a tweak:

** ** webpack.config.js
**webpack.config.js**

```javascript
module.exports = mode => {
leanpub-start-insert
  process.env.BABEL_ENV = mode;
leanpub-end-insert

  ...
};
```

T>`env`的工作方式很微妙。考虑记录`env`并确保它与您的Babel配置匹配，否则您期望的功能不会应用于您的构建。
T> The way `env` works is subtle. Consider logging `env` and make sure it matches your Babel configuration or otherwise the functionality you expect is not applied to your build.

##设置TypeScript
## Setting Up TypeScript

Microsoft的[TypeScript]（http://www.typescriptlang.org/）是一种编译语言，遵循与Babel类似的设置。整洁的是，除了JavaScript之外，它还可以发出类型定义。一个好的编辑可以选择那些并提供增强的编辑体验。更强的打字对于开发很有价值，因为更容易说明您的类型合同。
Microsoft's [TypeScript](http://www.typescriptlang.org/) is a compiled language that follows a similar setup as Babel. The neat thing is that in addition to JavaScript, it can emit type definitions. A good editor can pick those up and provide enhanced editing experience. Stronger typing is valuable for development as it becomes easier to state your type contracts.

与Facebook的类型检查器Flow相比，TypeScript是一种更安全的选择。因此，您可以找到更多预制的类型定义，总体而言，支持的质量应该更好。
Compared to Facebook's type checker Flow, TypeScript is a more secure option. As a result, you find more premade type definitions for it, and overall, the quality of support should be better.

您可以使用以下加载器将TypeScript与webpack一起使用：
You can use TypeScript with webpack using the following loaders:

* [ts-loader]（https://www.npmjs.com/package/ts-loader）
* [ts-loader](https://www.npmjs.com/package/ts-loader)
* [awesome-typescript-loader]（https://www.npmjs.com/package/awesome-typescript-loader）
* [awesome-typescript-loader](https://www.npmjs.com/package/awesome-typescript-loader)

T>有一个[ESLint的TypeScript解析器]（https://www.npmjs.com/package/typescript-eslint-parser）。它也可以通过[tslint]（https://www.npmjs.com/package/tslint）来提取它。
T> There's a [TypeScript parser for ESLint](https://www.npmjs.com/package/typescript-eslint-parser). It's also possible to lint it through [tslint](https://www.npmjs.com/package/tslint).

##设置流程
## Setting Up Flow

[Flow]（https://flow.org/）根据您的代码及其类型注释执行静态分析。您必须将其作为单独的工具安装，然后针对您的代码运行它。有[flow-status-webpack-plugin]（https://www.npmjs.com/package/flow-status-webpack-plugin），允许您在开发期间通过webpack运行它。
[Flow](https://flow.org/) performs static analysis based on your code and its type annotations. You have to install it as a separate tool and then run it against your code. There's [flow-status-webpack-plugin](https://www.npmjs.com/package/flow-status-webpack-plugin) that allows you to run it through webpack during development.

如果您使用React，则React特定Babel预设通过[babel-plugin-syntax-flow]（https://www.npmjs.com/package/babel-plugin-syntax-flow）完成大部分工作。它可以删除Flow注释并将您的代码转换为可以进一步转换的格式。
If you use React, the React specific Babel preset does most of the work through [babel-plugin-syntax-flow](https://www.npmjs.com/package/babel-plugin-syntax-flow). It can strip Flow annotations and convert your code into a format that is possible to transpile further.

还有[babel-plugin-typecheck]（https://www.npmjs.com/package/babel-plugin-typecheck），它允许您根据Flow注释执行运行时检查。 [flow-runtime]（https://codemix.github.io/flow-runtime/）进一步提升并提供更多功能。这些方法补充了Flow静态检查程序，使您可以捕获更多问题。
There's also [babel-plugin-typecheck](https://www.npmjs.com/package/babel-plugin-typecheck) that allows you to perform runtime checks based on your Flow annotations. [flow-runtime](https://codemix.github.io/flow-runtime/) goes a notch further and provides more functionality. These approaches complement Flow static checker and allow you to catch even more issues.

T> [flow-coverage-report]（https://www.npmjs.com/package/flow-coverage-report）显示Flow类型注释涵盖了多少代码。
T> [flow-coverage-report](https://www.npmjs.com/package/flow-coverage-report) shows how much of your code is covered by Flow type annotations.

{pagebreak}

##结论
## Conclusion

Babel已经成为开发人员不可或缺的工具，因为它标准化了旧版浏览器的标准。即使您针对现代浏览器，通过Babel进行转换也是一种选择。
Babel has become an indispensable tool for developers given it bridges the standard with older browsers. Even if you targeted modern browsers, transforming through Babel is an option.

回顾一下：
To recap:

* Babel让您可以控制要支持的浏览器。它可以将ES2015 +功能编译为旧浏览器所理解的形式。 * babel-preset-env *很有价值，因为它可以根据您的浏览器定义选择要编译的功能和要启用的polyfill。
* Babel gives you control over what browsers to support. It can compile ES2015+ features to a form the older browser understand. *babel-preset-env* is valuable as it can choose which features to compile and which polyfills to enable based on your browser definition.
* Babel允许您使用实验语言功能。您可以找到许多通过优化改进开发体验和生产构建的插件。
* Babel allows you to use experimental language features. You can find numerous plugins that improve development experience and the production build through optimizations.
*可以为每个开发目标启用Babel功能。这样，您可以确保在正确的位置使用正确的插件。
* Babel functionality can be enabled per development target. This way you can be sure you are using the correct plugins at the right place.
*除了Babel之外，webpack还支持TypeScript或Flow等其他解决方案。 Flow可以补充Babel，而TypeScript代表编译为JavaScript的整个语言。
* Besides Babel, webpack supports other solutions like TypeScript or Flow. Flow can complement Babel while TypeScript represents an entire language compiling to JavaScript.

