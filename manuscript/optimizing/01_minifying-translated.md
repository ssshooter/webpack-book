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

如果你现在执行 `npm run build`，你应该看到与之前相同的结果。结果可能略好一些，因为您可能会以这种方式使用较新版本的 UglifyJS。
If you execute `npm run build` now, you should see result close to the same as before. The outcome may be a slightly better as you are likely using a newer version of UglifyJS this way.

T> 默认情况下禁用源映射。您可以通过 `sourceMap` 标志启用它们。您应该查看 *uglifyjs-webpack-plugin* 以获取更多选项。
T> Source maps are disabled by default. You can enable them through the `sourceMap` flag. You should check *uglifyjs-webpack-plugin* for more options.

T> 要从源中删除 `console.log` 调用，可以将 `uglifyOptions.compress.drop_console` 设置为 `true`，如[discussed on Stack Overflow](https://stackoverflow.com/questions/49101152/webpack-v4-remove-console-logs-with-webpack-uglify) 提到。

{pagebreak}

##其他方法来缩小JavaScript
## Other Ways to Minify JavaScript

虽然默认值和* uglifyjs-webpack-plugin *适用于此用例，但您可以考虑更多选项：
Although the defaults and *uglifyjs-webpack-plugin* works for this use case, there are more options you can consider:

* [babel-minify-webpack-plugin]（https://www.npmjs.com/package/babel-minify-webpack-plugin）依赖于[babel-preset-minify]（https://www.npmjs.com下面是/ package / babel-preset-minify）它是由Babel团队开发的。但它比UglifyJS要慢。
* [babel-minify-webpack-plugin](https://www.npmjs.com/package/babel-minify-webpack-plugin) relies on [babel-preset-minify](https://www.npmjs.com/package/babel-preset-minify) underneath and it has been developed by the Babel team. It's slower than UglifyJS, though.
* [webpack-closure-compiler]（https://www.npmjs.com/package/webpack-closure-compiler）并行运行，有时甚至比* babel-minify-webpack-plugin *的结果更小。 [closure-webpack-plugin]（https://www.npmjs.com/package/closure-webpack-plugin）是另一种选择。
* [webpack-closure-compiler](https://www.npmjs.com/package/webpack-closure-compiler) runs parallel and gives even smaller result than *babel-minify-webpack-plugin* at times. [closure-webpack-plugin](https://www.npmjs.com/package/closure-webpack-plugin) is another option.
* [butternut-webpack-plugin]（https://www.npmjs.com/package/butternut-webpack-plugin）使用Rich Harris的实验[butternut]（https://www.npmjs.com/package/butternut）下面的minifier。
* [butternut-webpack-plugin](https://www.npmjs.com/package/butternut-webpack-plugin) uses Rich Harris' experimental [butternut](https://www.npmjs.com/package/butternut) minifier underneath.

##加快JavaScript执行速度
## Speeding Up JavaScript Execution

特定的解决方案允许您预处理代码，以便它运行得更快。它们是对缩小技术的补充，可以分为**范围提升**，**预评估**和**改进解析**。这些技术有时可能会增加整体包大小，同时允许更快的执行速度。
Specific solutions allow you to preprocess code so that it will run faster. They complement the minification technique and can be split into **scope hoisting**, **pre-evaluation**, and **improving parsing**. It's possible these techniques grow overall bundle size sometimes while allowing faster execution.

###范围吊装
### Scope Hoisting

从webpack 4开始，它默认在生产模式下应用范围提升。它将所有模块提升到单个范围，而不是为每个模块编写单独的闭包。这样做会减慢构建速度，但会为您提供执行速度更快的捆绑包。 [阅读有关范围提升的更多信息]（https://medium.com/webpack/brief-introduction-to-scope-hoisting-in-webpack-8435084c171f）在webpack博客上。
Since webpack 4, it applies scope hoisting in production mode by default. It hoists all modules to a single scope instead of writing a separate closure for each. Doing this slows down the build but gives you bundles that are faster to execute. [Read more about scope hoisting](https://medium.com/webpack/brief-introduction-to-scope-hoisting-in-webpack-8435084c171f) at the webpack blog.

T>将`--display-optimization-bailout`标志传递给webpack，以获取与提升结果相关的调试信息。
T>  Pass `--display-optimization-bailout` flag to webpack to gain debugging information related to hoisting results.

###预评估
### Pre-evaluation

[prepack-webpack-plugin]（https://www.npmjs.com/package/prepack-webpack-plugin）使用[Prepack]（https://prepack.io/），这是一个部分JavaScript评估程序。它重写了可以在编译时完成的计算，从而加快了代码执行速度。另请参阅[val-loader]（https://www.npmjs.com/package/val-loader）和[babel-plugin-preval]（https://www.npmjs.com/package/babel-plugin-preval ）替代解决方案。
[prepack-webpack-plugin](https://www.npmjs.com/package/prepack-webpack-plugin) uses [Prepack](https://prepack.io/), a partial JavaScript evaluator. It rewrites computations that can be done compile-time and therefore speeds up code execution. See also [val-loader](https://www.npmjs.com/package/val-loader) and [babel-plugin-preval](https://www.npmjs.com/package/babel-plugin-preval) for alternative solutions.

###改进解析
### Improving Parsing

[optimize-js-plugin]（https://www.npmjs.com/package/optimize-js-plugin）通过包装eager函数来补充其他解决方案，并且它增强了最初解析JavaScript代码的方式。该插件依赖于Nolan Lawson的[optimize-js]（https://github.com/nolanlawson/optimize-js）。
[optimize-js-plugin](https://www.npmjs.com/package/optimize-js-plugin) complements the other solutions by wrapping eager functions, and it enhances the way your JavaScript code gets parsed initially. The plugin relies on [optimize-js](https://github.com/nolanlawson/optimize-js) by Nolan Lawson.

##缩小HTML
## Minifying HTML

如果您使用[html-loader]（https://www.npmjs.com/package/html-loader）通过代码使用HTML模板，则可以通过[posthtml]（https://www.npmjs.com）对其进行预处理。 / package / posthtml）与[posthtml-loader]（https://www.npmjs.com/package/posthtml-loader）。您可以使用[posthtml-minifier]（https://www.npmjs.com/package/posthtml-minifier）通过它缩小HTML。
If you consume HTML templates through your code using [html-loader](https://www.npmjs.com/package/html-loader), you can preprocess it through [posthtml](https://www.npmjs.com/package/posthtml) with [posthtml-loader](https://www.npmjs.com/package/posthtml-loader). You can use [posthtml-minifier](https://www.npmjs.com/package/posthtml-minifier) to minify your HTML through it.

##缩小CSS
## Minifying CSS

* css-loader *允许通过[cssnano]（http://cssnano.co/）缩小CSS。需要使用`minimize`选项显式启用缩小。您还可以将[cssnano特定选项]（http://cssnano.co/optimisations/）传递给查询以进一步自定义行为。
*css-loader* allows minifying CSS through [cssnano](http://cssnano.co/). Minification needs to be enabled explicitly using the `minimize` option. You can also pass [cssnano specific options](http://cssnano.co/optimisations/) to the query to customize the behavior further.

[clean-css-loader]（https://www.npmjs.com/package/clean-css-loader）允许您使用流行的CSS缩小器[clean-css]（https://www.npmjs.com/包/清洁CSS）。
[clean-css-loader](https://www.npmjs.com/package/clean-css-loader) allows you to use a popular CSS minifier [clean-css](https://www.npmjs.com/package/clean-css).

[optimize-css-assets-webpack-plugin]（https://www.npmjs.com/package/optimize-css-assets-webpack-plugin）是一个基于插件的选项，可以在CSS资产上应用选定的minifier。使用`MiniCssExtractPlugin`可以导致重复的CSS，因为它只合并文本块。 `OptimizeCSSAssetsPlugin`通过对生成的结果进行操作来避免这个问题，从而可以产生更好的结果。
[optimize-css-assets-webpack-plugin](https://www.npmjs.com/package/optimize-css-assets-webpack-plugin) is a plugin based option that applies a chosen minifier on CSS assets. Using `MiniCssExtractPlugin` can lead to duplicated CSS given it only merges text chunks. `OptimizeCSSAssetsPlugin` avoids this problem by operating on the generated result and thus can lead to a better result.

###设置CSS缩小
### Setting Up CSS Minification

在可用的解决方案中，`OptimizeCSSAssetsPlugin`构成了最好的。要将其附加到设置，请先安装它并[cssnano]（http://cssnano.co/）：
Out of the available solutions, `OptimizeCSSAssetsPlugin` composes the best. To attach it to the setup, install it and [cssnano](http://cssnano.co/) first:

```bash
npm install optimize-css-assets-webpack-plugin cssnano --save-dev
```

{pagebreak}

与JavaScript一样，您可以将这个想法包含在配置部分中：
Like for JavaScript, you can wrap the idea in a configuration part:

** ** webpack.parts.js
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

W>如果你在* Build Analysis *章节中讨论过使用webpack的`--json`输出，你应该为插件设置`canPrint：false`。
W> If you use `--json` output with webpack as discussed in the *Build Analysis* chapter, you should set `canPrint: false` for the plugin.

{pagebreak}

然后，连接主配置：
Then, connect with the main configuration:

** ** webpack.config.js
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

如果您现在构建项目（`npm run build`），您应该注意到CSS已经变得更小，因为它缺少注释：
If you build the project now (`npm run build`), you should notice that CSS has become smaller as it's missing comments:

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

T> [compression-webpack-plugin]（https://www.npmjs.com/package/compression-webpack-plugin）允许您将生成压缩文件的问题推送到webpack，从而可能节省服务器上的处理时间。
T> [compression-webpack-plugin](https://www.npmjs.com/package/compression-webpack-plugin) allows you to push the problem of generating compressed files to webpack to potentially save processing time on the server.

##缩小图像
## Minifying Images

使用[img-loader]（https://www.npmjs.com/package/img-loader），[imagemin-webpack]（https://www.npmjs.com/package/imagemin--可以减少图像大小webpack）和[imagemin-webpack-plugin]（https://www.npmjs.com/package/imagemin-webpack-plugin）。这些包使用下面的图像优化器。
Image size can be reduced by using [img-loader](https://www.npmjs.com/package/img-loader), [imagemin-webpack](https://www.npmjs.com/package/imagemin-webpack), and [imagemin-webpack-plugin](https://www.npmjs.com/package/imagemin-webpack-plugin). The packages use image optimizers underneath.

使用* cache-loader *和* thread-loader *可以是一个好主意，如* Performance *章节中所述，因为它们可以是实质性操作。
It can be a good idea to use *cache-loader* and *thread-loader* with these as discussed in the *Performance* chapter given they can be substantial operations.

##结论
## Conclusion

缩小是您可以采取的最舒适的步骤，以使您的构建更小。回顾一下：
Minification is the most comfortable step you can take to make your build smaller. To recap:

* **缩小**过程分析您的源代码并将其转换为具有相同含义的较小形式（如果您使用安全转换）。特定的不安全转换允许您达到更小的结果，同时可能破坏依赖于精确参数命名的代码。
* **Minification** process analyzes your source code and turns it into a smaller form with the same meaning if you use safe transformations. Specific unsafe transformations allow you to reach even smaller results while potentially breaking code that relies, for example, on exact parameter naming.
* Webpack默认使用UglifyJS在生产模式下执行缩小。其他解决方案，例如* babel-minify-webpack-plugin *，提供与其自身成本相似的功能。
* Webpack performs minification in production mode using UglifyJS by default. Other solutions, such as *babel-minify-webpack-plugin*, provide similar functionality with costs of their own.
*除了JavaScript之外，还可以缩小其他资产，例如CSS，HTML和图像。缩小这些需要特定的技术，这些技术必须通过自己的加载器和插件来应用。
* Besides JavaScript, it's possible to minify other assets, such as CSS, HTML, and images, too. Minifying these requires specific technologies that have to be applied through loaders and plugins of their own.

您将学习如何在下一章中对代码应用树摇动。
You'll learn to apply tree shaking against code in the next chapter.

