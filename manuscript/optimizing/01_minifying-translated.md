# Minifying

从 webpack 4 开始，生产环境默认使用 UglifyJS 压缩输出。所以，了解这项技术和进一步的可能性是很好的。

## 压缩 JavaScript

**压缩**的目的是将代码转换为更小的形式。安全**转换**通过重写代码而不会失去任何意义。例如：重命名变量，甚至在判定确实无法访问后删除整个代码块（`if（false）`）。

不安全的转换可能会破坏代码，因为它们可能会丢失底层代码所依赖的隐含内容。例如，Angular 1 在使用模块时需要特定的函数参数命名。这种情况下需要采取预防措施，否则重写参数会破坏代码。

### 修改 JavaScript 压缩处理

在 webpack 4 中，通过两个配置字段控制压缩处理：`optimization.minimize` 用于开关此功能；`optimization.minimizer` 数组用于配置项。

为了调整默认值，我们将 [uglifyjs-webpack-plugin](https://www.npmjs.com/package/uglifyjs-webpack-plugin) 添加到项目中，以便可以对其进行调整。

第一步，先将插件包含在项目中：

```bash
npm install uglifyjs-webpack-plugin --save-dev
```

{pagebreak}

要将其附加到配置，首先为其定义一个部件：

**webpack.parts.js**

```javascript
const UglifyWebpackPlugin = require("uglifyjs-webpack-plugin");

exports.minifyJavaScript = () => ({
  optimization: {
    minimizer: [new UglifyWebpackPlugin({ sourceMap: true })],
  },
});
```

将其连接到配置：

**webpack.config.js**

```javascript
const productionConfig = merge([
  parts.clean(PATHS.build),
leanpub-start-insert
  parts.minifyJavaScript(),
leanpub-end-insert
  ...
]);
```

如果你现在执行 `npm run build`，你应该看到与之前相同的结果。结果可能略好一些，因为你可能会以这种方式使用较新版本的 UglifyJS。
If you execute `npm run build` now, you should see result close to the same as before. The outcome may be a slightly better as you are likely using a newer version of UglifyJS this way.

T> 默认情况下禁用 Source maps。你可以通过 `sourceMap` 属性启用它们。可以查看 *uglifyjs-webpack-plugin* 以获取更多可选参数。

T> 要从源中删除 `console.log` 调用，可以将 `uglifyOptions.compress.drop_console` 设置为 `true`，如 [discussed on Stack Overflow](https://stackoverflow.com/questions/49101152/webpack-v4-remove-console-logs-with-webpack-uglify) 提到。

{pagebreak}

## 压缩 JavaScript 的其他方法

虽然默认值和 *uglifyjs-webpack-plugin* 适用于此用例，但你可以考虑更多选项：
Although the defaults and *uglifyjs-webpack-plugin* works for this use case, there are more options you can consider:

* [babel-minify-webpack-plugin](https://www.npmjs.com/package/babel-minify-webpack-plugin) 依赖于 [babel-preset-minify](https://www.npmjs.com/package/babel-preset-minify)。它由Babel团队开发的。比UglifyJS要慢一点。
* [webpack-closure-compiler](https://www.npmjs.com/package/webpack-closure-compiler) 并行运行，有时甚至比 *babel-minify-webpack-plugin* 的输出更小。[closure-webpack-plugin](https://www.npmjs.com/package/closure-webpack-plugin) 是另一种选择。
* [butternut-webpack-plugin](https://www.npmjs.com/package/butternut-webpack-plugin) 使用 Rich Harris 的实验性[butternut](https://www.npmjs.com/package/butternut) 压缩器。

## 加快 JavaScript 执行速度

特定的解决方案允许你预处理代码，以便它运行得更快。它们是对压缩技术的补充，可以分为 **scope hoisting**（作用域提升），**pre-evaluation**（预计算）和 **improving parsing**。这些技术有时可能会增加 bundle 的大小，但会加快运行速度。

### Scope Hoisting（作用域提升）

从 webpack 4 开始，它默认在生产模式下应用 Scope Hoisting。它将所有模块提升到单个作用于，而不是为每个模块编写单独的闭包。这样做会减慢构建速度，但会为你提供执行速度更快的 bundle。可以在 webpack 博客上[阅读有关 Scope Hoisting 的更多信息](https://medium.com/webpack/brief-introduction-to-scope-hoisting-in-webpack-8435084c171f)。

T> 将 `--display-optimization-bailout` flag 传入 webpack，以获取与提升结果相关的调试信息。

### Pre-evaluation（预计算）

[prepack-webpack-plugin](https://www.npmjs.com/package/prepack-webpack-plugin) 使用[Prepack]（https://prepack.io/），这是一个局部 JavaScript 运算程序。它重写可以在编译时完成的计算，从而加快了代码执行速度。替代方案请参考 [val-loader](https://www.npmjs.com/package/val-loader) 和 [babel-plugin-preval](https://www.npmjs.com/package/babel-plugin-preval)。

### Improving Parsing（编译优化）

[optimize-js-plugin](https://www.npmjs.com/package/optimize-js-plugin) 通过包装即时运行函数来加速 JavaScript 执行速度，并且它增强了 JavaScript 代码初次编译。该插件依赖于 Nolan Lawson 的 [optimize-js](https://github.com/nolanlawson/optimize-js)。

## 压缩 HTML

如果你使用[html-loader]（https://www.npmjs.com/package/html-loader）通过代码使用HTML模板，则可以通过[posthtml]（https://www.npmjs.com）对其进行预处理。 / package / posthtml）与[posthtml-loader]（https://www.npmjs.com/package/posthtml-loader）。你可以使用[posthtml-minifier]（https://www.npmjs.com/package/posthtml-minifier）通过它缩小HTML。
If you consume HTML templates through your code using [html-loader](https://www.npmjs.com/package/html-loader), you can preprocess it through [posthtml](https://www.npmjs.com/package/posthtml) with [posthtml-loader](https://www.npmjs.com/package/posthtml-loader). You can use [posthtml-minifier](https://www.npmjs.com/package/posthtml-minifier) to minify your HTML through it.

## 压缩 CSS

* css-loader *允许通过[cssnano]（http://cssnano.co/）缩小CSS。需要使用`minimize`选项显式启用缩小。你还可以将[cssnano特定选项]（http://cssnano.co/optimisations/）传递给查询以进一步自定义行为。
*css-loader* allows minifying CSS through [cssnano](http://cssnano.co/). Minification needs to be enabled explicitly using the `minimize` option. You can also pass [cssnano specific options](http://cssnano.co/optimisations/) to the query to customize the behavior further.

[clean-css-loader]（https://www.npmjs.com/package/clean-css-loader）允许你使用流行的CSS缩小器[clean-css]（https://www.npmjs.com/包/清洁CSS）。
[clean-css-loader](https://www.npmjs.com/package/clean-css-loader) allows you to use a popular CSS minifier [clean-css](https://www.npmjs.com/package/clean-css).

[optimize-css-assets-webpack-plugin]（https://www.npmjs.com/package/optimize-css-assets-webpack-plugin）是一个基于插件的选项，可以在CSS资产上应用选定的minifier。使用`MiniCssExtractPlugin`可以导致重复的CSS，因为它只合并文本块。 `OptimizeCSSAssetsPlugin`通过对生成的结果进行操作来避免这个问题，从而可以产生更好的结果。
[optimize-css-assets-webpack-plugin](https://www.npmjs.com/package/optimize-css-assets-webpack-plugin) is a plugin based option that applies a chosen minifier on CSS assets. Using `MiniCssExtractPlugin` can lead to duplicated CSS given it only merges text chunks. `OptimizeCSSAssetsPlugin` avoids this problem by operating on the generated result and thus can lead to a better result.

### 设置 CSS 压缩

在可用的解决方案中，`OptimizeCSSAssetsPlugin`构成了最好的。要将其附加到设置，请先安装它并[cssnano]（http://cssnano.co/）：
Out of the available solutions, `OptimizeCSSAssetsPlugin` composes the best. To attach it to the setup, install it and [cssnano](http://cssnano.co/) first:

```bash
npm install optimize-css-assets-webpack-plugin cssnano --save-dev
```

{pagebreak}

与JavaScript一样，你可以将这个想法包含在配置部分中：
Like for JavaScript, you can wrap the idea in a configuration part:

**webpack.parts.js**

```javascript
const OptimizeCSSAssetsPlugin = require(
  "optimize-css-assets-webpack-plugin"
);
const cssnano = require("cssnano");

exports.minifyCSS = ({ options }) => ({
  plugins: [
    new OptimizeCSSAssetsPlugin({
      cssProcessor: cssnano,
      cssProcessorOptions: options,
      canPrint: false,
    }),
  ],
});
```

W> 如果你在* Build Analysis *章节中讨论过使用webpack的`--json`输出，你应该为插件设置`canPrint：false`。
W> If you use `--json` output with webpack as discussed in the *Build Analysis* chapter, you should set `canPrint: false` for the plugin.

{pagebreak}

然后，添加到主配置：

**webpack.config.js**

```javascript
const productionConfig = merge([
  ...
  parts.minifyJavaScript(),
leanpub-start-insert
  parts.minifyCSS({
    options: {
      discardComments: {
        removeAll: true,
      },
      // Run cssnano in safe mode to avoid
      // potentially unsafe transformations.
      safe: true,
    },
  }),
leanpub-end-insert
  ...
]);
```

如果你现在构建项目（`npm run build`），可以看到 CSS 文件已经被压缩，删除了注释：

```bash
Hash: f51ecf99e0da4db99834
Version: webpack 4.1.1
Time: 3125ms
Built at: 3/16/2018 5:32:55 PM
           Asset       Size  Chunks             Chunk Names
      chunk.0.js  162 bytes       0  [emitted]
      chunk.1.js   96.8 KiB       1  [emitted]  vendors~main
         main.js   2.19 KiB       2  [emitted]  main
leanpub-start-insert
        main.css    1.2 KiB       2  [emitted]  main
vendors~main.css   1.32 KiB       1  [emitted]  vendors~main
leanpub-end-insert
  chunk.0.js.map  204 bytes       0  [emitted]
  chunk.1.js.map    235 KiB       1  [emitted]  vendors~main
...
```

T> [compression-webpack-plugin]（https://www.npmjs.com/package/compression-webpack-plugin）允许你将生成压缩文件的问题推送到webpack，从而可能节省服务器上的处理时间。
T> [compression-webpack-plugin](https://www.npmjs.com/package/compression-webpack-plugin) allows you to push the problem of generating compressed files to webpack to potentially save processing time on the server.

## 压缩图片

使用[img-loader]（https://www.npmjs.com/package/img-loader），[imagemin-webpack]（https://www.npmjs.com/package/imagemin--可以减少图像大小webpack）和[imagemin-webpack-plugin]（https://www.npmjs.com/package/imagemin-webpack-plugin）。这些包使用下面的图像优化器。
Image size can be reduced by using [img-loader](https://www.npmjs.com/package/img-loader), [imagemin-webpack](https://www.npmjs.com/package/imagemin-webpack), and [imagemin-webpack-plugin](https://www.npmjs.com/package/imagemin-webpack-plugin). The packages use image optimizers underneath.

使用* cache-loader *和* thread-loader *可以是一个好主意，如* Performance *章节中所述，因为它们可以是实质性操作。
It can be a good idea to use *cache-loader* and *thread-loader* with these as discussed in the *Performance* chapter given they can be substantial operations.

## 总结

压缩是最舒服的步骤了，只要配置一下就可以让你的项目构建更小。回顾一下：

* **压缩**过程分析你的源代码并将其转换为具有相同含义的较小形式（如果你使用安全转换）。特定的不安全转换允许你达到更小的结果，同时可能破坏依赖于精确参数命名的代码。
* 在生产模式下，Webpack 使用 UglifyJS 默认进行压缩。其他解决方案，例如 *babel-minify-webpack-plugin*，提供功能，不过有时会有其他代价。
* 除了 JavaScript 之外，还可以压缩其他资源，例如 CSS，HTML 和图片。压缩它们需要特定的技术，这些技术必须通过各自的 loader 和插件来应用。

你将在下一章中学习如何对代码应用 tree shaking。

