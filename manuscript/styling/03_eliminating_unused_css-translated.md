＃消除未使用的CSS
# Eliminating Unused CSS

像[Bootstrap]（https://getbootstrap.com/）这样的框架往往带有很多CSS。通常你只使用它的一小部分。通常，你甚至捆绑未使用的CSS。但是，可以消除你不使用的部分。
Frameworks like [Bootstrap](https://getbootstrap.com/) tend to come with a lot of CSS. Often you use only a small part of it. Typically, you bundle even the unused CSS. It's possible, however, to eliminate the portions you aren't using.

[PurifyCSS]（https://www.npmjs.com/package/purifycss）是一个可以通过分析文件来实现这一目标的工具。它遍历你的代码并确定正在使用的CSS类。通常有足够的信息可以从项目中删除未使用的CSS。它也适用于单页应用程序。
[PurifyCSS](https://www.npmjs.com/package/purifycss) is a tool that can achieve this by analyzing files. It walks through your code and figures out which CSS classes are being used. Often there is enough information for it to strip unused CSS from your project. It also works with single page applications to an extent.

[uncss]（https://www.npmjs.com/package/uncss）是PurifyCSS的一个很好的替代品。它通过PhantomJS运行，并以不同的方式执行其工作。你可以使用uncss本身作为PostCSS插件。
[uncss](https://www.npmjs.com/package/uncss) is a good alternative to PurifyCSS. It operates through PhantomJS and performs its work differently. You can use uncss itself as a PostCSS plugin.

W>如果使用CSS模块，必须要小心。你必须按照[purifycss-webpack readme]（https://github.com/webpack-contrib/purifycss-webpack#usage-with-css-modules）中的讨论，将相关类列入白名单**。
W> You have to be careful if you are using CSS Modules. You have to **whitelist** the related classes as discussed in [purifycss-webpack readme](https://github.com/webpack-contrib/purifycss-webpack#usage-with-css-modules).

##设置Pure.css
## Setting Up Pure.css

为了使演示更加真实，让我们安装一个小的CSS框架[Pure.css]（http://purecss.io/），并从项目中引用它，以便你可以看到PurifyCSS的运行情况。尽管命名，但这两个项目没有任何关联。
To make the demo more realistic, let's install [Pure.css](http://purecss.io/), a small CSS framework, as well and refer to it from the project so that you can see PurifyCSS in action. These two projects aren't related in any way despite the naming.

```bash
npm install purecss --save
```

{pagebreak}

为了使项目了解Pure.css，`import`它：
To make the project aware of Pure.css, `import` it:

** SRC / index.js **
**src/index.js**

```javascript
leanpub-start-insert
import "purecss";
leanpub-end-insert
...
```

T>`import`是有效的，因为webpack将解析``browser'：“build / pure-min.css”，由于[resolve.mainFields]（https），Pure.css的* package.json *文件中的字段：//webpack.js.org/configuration/resolve/#resolve-mainfields）。在查看`main`之前，Webpack将尝试解析可能的`browser`和`module`字段。
T> The `import` works because webpack will resolve against `"browser": "build/pure-min.css",` field in the *package.json* file of Pure.css due to [resolve.mainFields](https://webpack.js.org/configuration/resolve/#resolve-mainfields). Webpack will try to resolve possible `browser` and `module` fields before looking into `main`.

你还应该使演示组件使用Pure.css类，因此可以使用以下内容：
You should also make the demo component use a Pure.css class, so there is something to work with:

** SRC / component.js **
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

如果你运行应用程序（`npm start`），“Hello world”应该看起来像一个按钮。
If you run the application (`npm start`), the "Hello world" should look like a button.

![Styled hello](images/styled-button.png)

构建应用程序（`npm run build`）应该产生输出：
Building the application (`npm run build`) should yield output:

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

正如你所看到的，CSS文件的大小增加了，这可以通过PurifyCSS来解决。
As you can see, the size of the CSS file grew, and this is something to fix with PurifyCSS.

##启用PurifyCSS
## Enabling PurifyCSS

使用PurifyCSS可以节省大量成本。在该项目的示例中，他们使用约40％的选择器在一个应用程序中纯化和缩小Bootstrap（140 kB），仅为~35 kB。这是一个很大的不同。
Using PurifyCSS can lead to significant savings. In the example of the project, they purify and minify Bootstrap (140 kB) in an application using ~40% of its selectors to mere ~35 kB. That's a big difference.

{pagebreak}

[purifycss-webpack]（https://www.npmjs.com/package/purifycss-webpack）允许实现类似的结果。你应该使用`MiniCssExtractPlugin`来获得最佳效果。首先安装它和[glob]（https://www.npmjs.org/package/glob）帮助器：
[purifycss-webpack](https://www.npmjs.com/package/purifycss-webpack) allows to achieve similar results. You should use the `MiniCssExtractPlugin` with it for the best results. Install it and a [glob](https://www.npmjs.org/package/glob) helper first:

```bash
npm install glob purifycss-webpack purify-css --save-dev
```

你还需要PurifyCSS配置如下：
You also need PurifyCSS configuration as below:

** ** webpack.parts.js
**webpack.parts.js**

```javascript
const PurifyCSSPlugin = require("purifycss-webpack");

exports.purifyCSS = ({ paths }) => ({
  plugins: [new PurifyCSSPlugin({ paths })],
});
```

接下来，必须将该部件与配置连接。在'MiniCssExtractPlugin`之后使用插件是必不可少的。否则，它不起作用：
Next, the part has to be connected with the configuration. It's essential the plugin is used *after* the `MiniCssExtractPlugin`; otherwise, it doesn't work:

** ** webpack.config.js
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

W>订单很重要。 CSS提取必须在净化之前进行。
W> The order matters. CSS extraction has to happen before purifying.

如果你现在执行`npm run build`，你会看到一些东西：
If you execute `npm run build` now, you should see something:

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

风格的大小明显减少。而不是16k，你现在大约有2k。对于更大规模的CSS框架而言，差异将更加显着。
The size of the style has decreased noticeably. Instead of 16k, you have roughly 2k now. The difference would be even more significant for more massive CSS frameworks.

PurifyCSS支持[附加选项]（https://github.com/purifycss/purifycss#the-optional-options-argument），包括`minify`。你可以在实例化插件时通过`purifyOptions`字段启用它们。鉴于PurifyCSS无法选择你一直使用的所有类，你应该使用`purifyOptions.whitelist`数组来定义选择器，它应该留在结果中，无论如何。
PurifyCSS supports [additional options](https://github.com/purifycss/purifycss#the-optional-options-argument) including `minify`. You can enable these through the `purifyOptions` field when instantiating the plugin. Given PurifyCSS cannot pick all of the classes you are always using, you should use `purifyOptions.whitelist` array to define selectors which it should leave in the result no matter what.

W>使用PurifyCSS会丢失CSS源映射，即使你已使用加载程序特定的配置启用它们，因为它在下面的工作方式。
W> Using PurifyCSS loses CSS source maps even if you have enabled them with loader specific configuration due to the way it works underneath.

{pagebreak}

###关键路径渲染
### Critical Path Rendering

[关键路径渲染]（https://developers.google.com/web/fundamentals/performance/critical-rendering-path/）的想法从不同的角度来看待CSS性能。它不是优化大小，而是优化渲染顺序，并强调**首屏** CSS。通过渲染页面然后确定获得所示结果所需的规则来实现结果。
The idea of [critical path rendering](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/) takes a look at CSS performance from a different angle. Instead of optimizing for size, it optimizes for render order and emphasizes **above-the-fold** CSS. The result is achieved by rendering the page and then figuring out which rules are required to obtain the shown result.

[webpack-critical]（https://www.npmjs.com/package/webpack-critical）和[html-critical-webpack-plugin]（https://www.npmjs.com/package/html-critical-webpack -plugin）将该技术实现为`HtmlWebpackPlugin`插件。 [isomorphic-style-loader]（https://www.npmjs.com/package/isomorphic-style-loader）使用webpack和React实现了相同的功能。
[webpack-critical](https://www.npmjs.com/package/webpack-critical) and [html-critical-webpack-plugin](https://www.npmjs.com/package/html-critical-webpack-plugin) implement the technique as a `HtmlWebpackPlugin` plugin. [isomorphic-style-loader](https://www.npmjs.com/package/isomorphic-style-loader) achieves the same using webpack and React.

Addy Osmani的[critical-path-css-tools]（https://github.com/addyosmani/critical-path-css-tools）列出了其他相关工具。
[critical-path-css-tools](https://github.com/addyosmani/critical-path-css-tools) by Addy Osmani lists other related tools.

## 总结


使用PurifyCSS可以显着减少文件大小。它主要用于依赖大量CSS框架的静态站点。站点或应用程序变得越动态，就越难以可靠地进行分析。
Using PurifyCSS can lead to a significant decrease in file size. It's mainly valuable for static sites that rely on a massive CSS framework. The more dynamic a site or an application becomes, the harder it becomes to analyze reliably.

回顾一下：


*使用PurifyCSS可以消除未使用的CSS。它对源执行静态分析。
* Eliminating unused CSS is possible using PurifyCSS. It performs static analysis against the source.
*功能可以通过* purifycss-webpack *启用，插件应该在*`MiniCssExtractPlugin`之后应用*。
* The functionality can be enabled through *purifycss-webpack*, and the plugin should be applied *after* `MiniCssExtractPlugin`.
*充其量，PurifyCSS可以消除大多数（如果不是全部）未使用的CSS规则。
* At best, PurifyCSS can eliminate most, if not all, unused CSS rules.
*关键路径渲染是另一种CSS技术，它强调首先渲染上层CSS。我们的想法是尽可能快地呈现某些内容，而不是等待加载所有CSS。
* Critical path rendering is another CSS technique that emphasizes rendering the above-the-fold CSS first. The idea is to render something as fast as possible instead of waiting for all CSS to load.

在下一章中，你将学习** autoprefix **。启用该功能可以更方便地开发适用于旧版浏览器的复杂CSS设置。
In the next chapter, you'll learn to **autoprefix**. Enabling the feature makes it more convenient to develop complicated CSS setups that work with older browsers as well.

