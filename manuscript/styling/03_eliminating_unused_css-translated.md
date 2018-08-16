# 清理未使用 CSS

像 [Bootstrap](https://getbootstrap.com/) 这样的框架往往带有很多 CSS。一般你只会使用到它的一小部分。不做任何设置的话，你会把未使用的 CSS 也打包到输出。不过，有方法可以从打包结果中清除你不使用的部分。

[PurifyCSS](https://www.npmjs.com/package/purifycss) 是一个可以通过分析文件来实现这一目标的工具。它遍历你的代码并确定正在使用的 CSS class。这个步骤之后通常有足够的信息可以判定项目中未使用的 CSS，并将其删除。它也适用于单页应用程序。

[uncss](https://www.npmjs.com/package/uncss) 是 PurifyCSS 的一个很好的替代品。它通过 PhantomJS 运行，以不同的方式执行其工作。你可以作为 PostCSS 插件使用 uncss。

W> 如果使用CSS模块，必须要小心。你必须按照[purifycss-webpack readme]（https://github.com/webpack-contrib/purifycss-webpack#usage-with-css-modules）中的讨论，将相关类列入白名单**。
W> You have to be careful if you are using CSS Modules. You have to **whitelist** the related classes as discussed in [purifycss-webpack readme](https://github.com/webpack-contrib/purifycss-webpack#usage-with-css-modules).

## 使用 Pure.css

为了让例子更贴近实践，我们安装一个小型 CSS 框架 [Pure.css](http://purecss.io/)，并从项目中引用它，以便你可以看到 PurifyCSS 的运行情况。注意注意，尽管它们的名字很像，但这两个项目确实没有任何关联。

```bash
npm install purecss --save
```

{pagebreak}

在项目中 `import` Pure.css：

**src/index.js**

```javascript
leanpub-start-insert
import "purecss";
leanpub-end-insert
...
```

T> `import` 是有效的，因为webpack将解析`"browser": "build/pure-min.css",`，由于[resolve.mainFields]（https），Pure.css的* package.json *文件中的字段：//webpack.js.org/configuration/resolve/#resolve-mainfields）。在查看`main`之前，Webpack将尝试解析可能的`browser`和`module`字段。
T> The `import` works because webpack will resolve against `"browser": "build/pure-min.css",` field in the *package.json* file of Pure.css due to [resolve.mainFields](https://webpack.js.org/configuration/resolve/#resolve-mainfields). Webpack will try to resolve possible `browser` and `module` fields before looking into `main`.

在演示组件添加 Pure.css class：

**src/component.js**

```javascript
export default (text = "Hello world") => {
  const element = document.createElement("div");

leanpub-start-insert
  element.className = "pure-button";
leanpub-end-insert
  element.innerHTML = text;

  return element;
};
```

运行应用程序（`npm start`），"Hello world" 看起来应该像一个按钮。

![Styled hello](images/styled-button.png)

构建应用程序（`npm run build`）产生的输出：

```bash
Hash: 36bff4e71a3f746d46fa
Version: webpack 4.1.1
Time: 739ms
Built at: 3/16/2018 4:26:49 PM
     Asset       Size  Chunks             Chunk Names
   main.js  747 bytes       0  [emitted]  main
  main.css   16.1 KiB       0  [emitted]  main
index.html  220 bytes          [emitted]
...
```

正如你所见，CSS 文件的体积增加了，这个问题可以通过 PurifyCSS 来解决。

## 启用 PurifyCSS

使用 PurifyCSS 可以节省大量成本。在该项目的示例中，他们使用约40%的选择器在一个应用程序中纯化和缩小Bootstrap（140 kB），仅为~35 kB。大有不同。
Using PurifyCSS can lead to significant savings. In the example of the project, they purify and minify Bootstrap (140 kB) in an application using ~40% of its selectors to mere ~35 kB. That's a big difference.

{pagebreak}

[purifycss-webpack](https://www.npmjs.com/package/purifycss-webpack) 可以实现类似的结果。你应该配合使用 `MiniCssExtractPlugin` 来获得最佳效果。首先安装它和[glob](https://www.npmjs.org/package/glob)：

```bash
npm install glob purifycss-webpack purify-css --save-dev
```

你还需要配置 PurifyCSS：

**webpack.parts.js**

```javascript
const PurifyCSSPlugin = require("purifycss-webpack");

exports.purifyCSS = ({ paths }) => ({
  plugins: [new PurifyCSSPlugin({ paths })],
});
```

接下来添加在主配置中。插件需要在 `MiniCssExtractPlugin` **之后**使用，否则会不起作用：

**webpack.config.js**

```javascript
...
leanpub-start-insert
const path = require("path");
const glob = require("glob");
leanpub-end-insert

const parts = require("./webpack.parts");

leanpub-start-insert
const PATHS = {
  app: path.join(__dirname, "src"),
};
leanpub-end-insert

...

const productionConfig = merge([
  ...
leanpub-start-insert
  parts.purifyCSS({
    paths: glob.sync(`${PATHS.app}/**/*.js`, { nodir: true }),
  }),
leanpub-end-insert
]);
```

W> 顺序很重要，CSS 提取必须在 purify 之前进行。

现在执行 `npm run build`，可以看到：

```bash
Hash: 36bff4e71a3f746d46fa
Version: webpack 4.1.1
Time: 695ms
Built at: 3/16/2018 4:29:54 PM
     Asset       Size  Chunks             Chunk Names
   main.js  747 bytes       0  [emitted]  main
  main.css   2.07 KiB       0  [emitted]  main
index.html  220 bytes          [emitted]
...
```

CSS 文件的体积从原来的 16k 减小到现在的 2k。对于更大规模的 CSS 框架而言，差异将更加显着。

PurifyCSS 支持[附加选项](https://github.com/purifycss/purifycss#the-optional-options-argument)，包括`minify`。你可以在实例化插件时通过 `purifyOptions` 字段启用它们。鉴于PurifyCSS无法选择你一直使用的所有类，你应该使用`purifyOptions.whitelist`数组来定义选择器，它应该留在结果中，无论如何。
PurifyCSS supports [additional options](https://github.com/purifycss/purifycss#the-optional-options-argument) including `minify`. You can enable these through the `purifyOptions` field when instantiating the plugin. Given PurifyCSS cannot pick all of the classes you are always using, you should use `purifyOptions.whitelist` array to define selectors which it should leave in the result no matter what.

W> 受 PurifyCSS 的底层原理所限，即使你在 loader 启用了 source maps，也无法正常使用。

{pagebreak}

### 关键路径渲染
### Critical Path Rendering

[关键路径渲染]（https://developers.google.com/web/fundamentals/performance/critical-rendering-path/）的想法从不同的角度来看待CSS性能。它不是优化大小，而是优化渲染顺序，并强调**首屏** CSS。通过渲染页面然后确定获得所示结果所需的规则来实现结果。
The idea of [critical path rendering](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/) takes a look at CSS performance from a different angle. Instead of optimizing for size, it optimizes for render order and emphasizes **above-the-fold** CSS. The result is achieved by rendering the page and then figuring out which rules are required to obtain the shown result.

[webpack-critical]（https://www.npmjs.com/package/webpack-critical）和[html-critical-webpack-plugin]（https://www.npmjs.com/package/html-critical-webpack -plugin）将该技术实现为`HtmlWebpackPlugin`插件。 [isomorphic-style-loader]（https://www.npmjs.com/package/isomorphic-style-loader）使用webpack和React实现了相同的功能。
[webpack-critical](https://www.npmjs.com/package/webpack-critical) and [html-critical-webpack-plugin](https://www.npmjs.com/package/html-critical-webpack-plugin) implement the technique as a `HtmlWebpackPlugin` plugin. [isomorphic-style-loader](https://www.npmjs.com/package/isomorphic-style-loader) achieves the same using webpack and React.

Addy Osmani 的 [critical-path-css-tools](https://github.com/addyosmani/critical-path-css-tools) 列出了其他相关工具。

## 总结

使用 PurifyCSS 可以显着减少样式文件的体积。它主要用于依赖大量 CSS 框架的**静态站点**。站点或应用越是“动态”，就越难可靠地进行分析。

回顾一下：

* PurifyCSS 会对源执行静态分析，可以清除未使用的 CSS。
* 功能可以通过* purifycss-webpack *启用，插件应该在*`MiniCssExtractPlugin`之后应用*。
* The functionality can be enabled through *purifycss-webpack*, and the plugin should be applied *after* `MiniCssExtractPlugin`.
*充其量，PurifyCSS可以消除大多数（如果不是全部）未使用的CSS规则。
* At best, PurifyCSS can eliminate most, if not all, unused CSS rules.
*关键路径渲染是另一种CSS技术，它强调首先渲染上层CSS。我们的想法是尽可能快地呈现某些内容，而不是等待加载所有CSS。
* Critical path rendering is another CSS technique that emphasizes rendering the above-the-fold CSS first. The idea is to render something as fast as possible instead of waiting for all CSS to load.

在下一章中，你将学习 **autoprefix**。启用该功能可以更方便地开发适用于老旧浏览器的复杂 CSS。

