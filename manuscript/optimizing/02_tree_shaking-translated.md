＃树摇晃
# Tree Shaking

**Tree Shaking**是ES2015模块定义启用的功能。这个想法是，如果可以静态地分析模块定义而不运行它，webpack可以告诉代码的哪些部分正在使用，哪些部分不正确。可以通过扩展应用程序并在那里添加应该删除的代码来验证此行为。
**Tree shaking** is a feature enabled by the ES2015 module definition. The idea is that given it's possible to analyze the module definition statically without running it, webpack can tell which parts of the code are being used and which are not. It's possible to verify this behavior by expanding the application and adding code there that should be eliminated.

T> Tree shake在某种程度上通过[webpack-common-shake]（https://www.npmjs.com/package/webpack-common-shake）对CommonJS模块定义起作用。由于大多数npm软件包都是使用旧版定义编写的，因此该插件具有价值。
T> Tree shaking works to an extent through [webpack-common-shake](https://www.npmjs.com/package/webpack-common-shake) against CommonJS module definition. As a majority of npm packages have been authored using the older definition, the plugin has value.

## 演示 Tree Shaking
## Demonstrating Tree Shaking

要动摇代码，你必须定义一个模块并仅使用其部分代码。设置一个：
To shake code, you have to define a module and use only a part of its code. Set one up:

**src/shake.js**

```javascript
const shake = () => console.log("shake");
const bake = () => console.log("bake");

export { shake, bake };
```

{pagebreak}

要确保使用部分代码，请更改应用程序入口点：
To make sure you use a part of the code, alter the application entry point:

**src/index.js**

```javascript
...
import { bake } from "./shake";

bake();

...
```

如果再次构建项目（`npm run build`）并检查构建结果（*dist/main.js*），它应该包含 `console.log（“bake”）`，不包含 `console.log（“shake” “）`。Tree Shaking 正常工作。

为了更好地了解 webpack 为 Tree Shaking 做了什么，可以通过 `npm run build -- --display-used-exports` 运行它。你应该在终端中看到额外的输出，如 `[no exports used]` 或 `[only some exports used: bake]`。

T>如果你使用的是“UglifyJsPlugin”，请启用类似效果的警告。除了其他消息之外，你应该看到诸如`Dropping unused variable treeShakingDemo [./src/component.js:17,6]`之类的行。
T> If you are using `UglifyJsPlugin`, enable warnings for a similar effect. In addition to other messages, you should see lines like `Dropping unused variable treeShakingDemo [./src/component.js:17,6]`.

T>在[dead-css-loader]（https://github.com/simlrh/dead-css-loader）中有一个与CSS模块相关的Tree Shaking概念验证。
T> There is a CSS Modules related tree shaking proof of concept at [dead-css-loader](https://github.com/simlrh/dead-css-loader).

## 包级别上的 Tree Shaking
## Tree Shaking on Package Level

同样的想法适用于使用ES2015模块定义的依赖项。鉴于相关的包装，标准仍在出现，在使用此类包装时必须小心。出于这个原因，Webpack尝试解析* package.json *`module`字段。
The same idea works with dependencies that use the ES2015 module definition. Given the related packaging, standards are still emerging, you have to be careful when consuming such packages. Webpack tries to resolve *package.json* `module` field for this reason.

对于像webpack这样的工具来允许树摇npm包，你应该生成一个构建，它已经转换了除ES2015模块定义之外的所有其他内容，然后通过* package.json *`module`字段指向它。在Babel术语中，你必须通过设置`“modules”：false`让webpack来管理ES2015模块。
For tools like webpack to allow tree shake npm packages, you should generate a build that has transpiled everything else except the ES2015 module definitions and then point to it through *package.json* `module` field. In Babel terms, you have to let webpack to manage ES2015 modules by setting `"modules": false`.

为了最大限度地利用外部软件包摇动树，你必须使用[babel-plugin-transform-imports]（https://www.npmjs.com/package/babel-plugin-transform-imports）重写导入，以便他们使用webpack的Tree Shaking逻辑。有关详细信息，请参阅[webpack issue＃2867]（https://github.com/webpack/webpack/issues/2867）。
To get most out of tree shaking with external packages, you have to use [babel-plugin-transform-imports](https://www.npmjs.com/package/babel-plugin-transform-imports) to rewrite imports so that they work with webpack's tree shaking logic. See [webpack issue #2867](https://github.com/webpack/webpack/issues/2867) for more information.

T> [SurviveJS - Maintenance](https://survivejs.com/maintenance/packaging/building/) 介绍了如何编写软件包，以便可以对它们应用 Tree Shaking。

## 结论

Tree Shaking 是一项潜力很大的技术。为了让源代码受益于 Tree Shaking，必须使用 ES2015 模块语法实现 npm 包，并且必须通过*package.json* `module` 这样的字段工具公开ES2015版本，例如webpack可以接收。
Tree shaking is a potentially powerful technique. For the source to benefit from tree shaking, npm packages have to be implemented using the ES2015 module syntax, and they have to expose the ES2015 version through *package.json* `module` field tools like webpack can pick up.

回顾一下：

* **Tree Shaking** 根据静态代码分析丢弃未使用的代码片段。Webpack 会在遍历依赖关系图时执行此过程。
* 要使用 tree shaking，你必须使用 ES2015 模块规范。
* 作为 npm 包的作者，你可以提供包含 ES2015 模块的版本，其余版本可转换为 ES5。

你将在下一章学习如何使用 webpack 管理环境变量。

