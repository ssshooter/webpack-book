＃表现
# Performance

Webpack的开箱即用性能通常足以满足小型项目的需求。也就是说，随着项目规模的扩大，它开始达到极限。这是webpack问题跟踪器中的常见主题。 [问题1905]（https://github.com/webpack/webpack/issues/1905）就是一个很好的例子。
Webpack's performance out of the box is often enough for small projects. That said, it begins to hit limits as your project grows in scale. It's a frequent topic in webpack's issue tracker. [Issue 1905](https://github.com/webpack/webpack/issues/1905) is a good example.

在优化方面有几个基本规则：
There are a couple of ground rules when it comes to optimization:

1.知道要优化的内容。
1. Know what to optimize.
2.首先快速执行调整。
2. Perform fast to implement tweaks first.
3.之后进行更多涉及的调整。
3. Perform more involved tweaks after.
4.衡量影响。
4. Measure impact.

有时优化需要成本。它们可以使你的配置更难理解或将其与特定解决方案联系起来。通常，最好的优化是减少工作量或更聪明地做。基本说明将在下一节中介绍，以便你知道在何时处理性能。
Sometimes optimizations come with a cost. They can make your configuration harder to understand or tie it to a particular solution. Often the best optimization is to do less work or do it more smartly. The basic directions are covered in the next sections, so you know where to look when it's time to work on performance.

##衡量影响力
## Measuring Impact

如前一章所述，生成统计数据可用于衡量构建时间。 [speed-measure-webpack-plugin]（https://www.npmjs.com/package/speed-measure-webpack-plugin）为每个插件和加载器提供更详细的信息，以便你知道哪些过程大部分时间都在你的过程中。
As discussed in the previous chapter, generating stats can be used to measure build time. [speed-measure-webpack-plugin](https://www.npmjs.com/package/speed-measure-webpack-plugin) gives more granular information per plugin and loader so you know which take most of the time in your process.

{pagebreak}

##高级优化
## High-Level Optimizations

默认情况下，Webpack仅使用单个实例，这意味着你无需额外工作就无法从多核处理器中受益。这是第三方解决方案，例如[parallel-webpack]（https://www.npmjs.com/package/parallel-webpack）和[HappyPack]（https://www.npmjs.com/package/happypack）进来。
Webpack uses only a single instance by default meaning you aren't able to benefit from a multi-core processor without extra effort. This where third-party solutions, such as [parallel-webpack](https://www.npmjs.com/package/parallel-webpack) and [HappyPack](https://www.npmjs.com/package/happypack) come in.

### parallel-webpack  - 并行运行多个Webpack实例
### parallel-webpack - Run Multiple Webpack Instances in Parallel

* parallel-webpack *允许你以两种方式并行化webpack配置。假设你已将webpack配置定义为数组，它可以并行运行配置。除此之外，* parallel-webpack *可以基于给定的**变体**生成构建。
*parallel-webpack* allows you to parallelize webpack configuration in two ways. Assuming you have defined your webpack configuration as an array, it can run the configurations in parallel. In addition to this, *parallel-webpack* can generate builds based on given **variants**.

使用变体可以同时生成生产和开发构建。变体还允许你生成具有不同目标的包，以便根据环境更容易使用它们。如'EnvironmentPlugin`结合使用时，变量可用于实现功能标志，如*环境变量*章节中所述。
Using variants allows you to generate both production and development builds at once. Variants also allow you to generate bundles with different targets to make them easier to consume depending on the environment. Variants can be used to implement feature flags when combined with `DefinePlugin` as discussed in the *Environment Variables* chapter.

可以使用[worker-farm]（https://www.npmjs.com/package/worker-farm）实现基本思想。事实上，* parallel-webpack *依赖于* worker-farm *。
The underlying idea can be implemented using a [worker-farm](https://www.npmjs.com/package/worker-farm). In fact, *parallel-webpack* relies on *worker-farm* underneath.

* parallel-webpack *可以通过将它作为开发依赖项安装到项目中，然后用`parallel-webpack`替换`webpack`命令来使用。
*parallel-webpack* can be used by installing it to your project as a development dependency and then replacing `webpack` command with `parallel-webpack`.

{pagebreak}

### HappyPack  - 文件级并行
### HappyPack - File Level Parallelism

与* parallel-webpack *相比，HappyPack是一个更为复杂的选择。我们的想法是，HappyPack拦截你指定的加载程序调用，然后并行运行它们。你必须先设置插件：
Compared to *parallel-webpack*, HappyPack is a more involved option. The idea is that HappyPack intercepts the loader calls you specify and then runs them in parallel. You have to set up the plugin first:

** ** webpack.config.js
**webpack.config.js**

```javascript
...
const HappyPack = require("happypack");

...

const commonConfig = merge([{
  {
    plugins: [
      new HappyPack({
        loaders: [
          // Capture Babel loader
          "babel-loader"
        ],
      }),
    ],
  },
}];
```

{pagebreak}

要完成连接，你必须使用HappyPack替换原始Babel加载程序定义：
To complete the connection, you have to replace the original Babel loader definition with a HappyPack one:

```javascript
exports.loadJavaScript = ({ include, exclude }) => ({
  module: {
    rules: [
      {
        ...
leanpub-start-delete
        loader: "babel-loader",
leanpub-end-delete
leanpub-start-insert
        loader: "happypack/loader",
leanpub-end-insert
        ...
      },
    ],
  },
});
```

上面的示例包含足够的信息，以便webpack运行给定的loader并行。 HappyPack带有更高级的选项，但应用这个想法就足以开始了。
The example above contains enough information for webpack to run the given loader parallel. HappyPack comes with more advanced options, but applying this idea is enough to get started.

也许HappyPack的问题在于它将你的配置与它结合在一起。有可能通过设计克服这个问题并使注入更容易。一种选择是构建更高级别的抽象，可以在vanilla配置之上执行替换。
Perhaps the problem with HappyPack is that it couples your configuration with it. It would be possible to overcome this issue by design and make it easier to inject. One option would be to build a higher level abstraction that can perform the replacement on top of vanilla configuration.

##低级优化
## Low-Level Optimizations

特定的低级优化可以很好地了解。关键是允许webpack执行更少的工作。你已经实现了其中的几个，但枚举它们是个好主意：
Specific lower-level optimizations can be good to know. The key is to allow webpack to perform less work. You have already implemented a couple of these, but it's a good idea to enumerate them:

*考虑在开发过程中使用更快的源地图变体或跳过它们。如果你不以任何方式处理代码，则可以跳过。
* Consider using faster source map variants during development or skip them. Skipping is possible if you don't process the code in any way.
*在开发期间使用[babel-preset-env]（https://www.npmjs.com/package/babel-preset-env）代替源地图，以便为现代浏览器转换更少的功能，并使代码更具可读性和更舒适调试。
* Use [babel-preset-env](https://www.npmjs.com/package/babel-preset-env) during development instead of source maps to transpile fewer features for modern browsers and make the code more readable and more comfortable to debug.
*在开发过程中跳过polyfill。将[babel-polyfill]（https://www.npmjs.com/package/babel-polyfill）等软件包附加到应用程序的开发版本会增加开销。
* Skip polyfills during development. Attaching a package, such as [babel-polyfill](https://www.npmjs.com/package/babel-polyfill), to the development version of an application adds to the overhead.
*禁用开发期间不需要的应用程序部分。编译你正在处理的一小部分可能是一个有效的想法，因为你可以减少捆绑。
* Disable the portions of the application you don't need during development. It can be a valid idea to compile only a small fraction you are working on as then you have less to bundle.
* Polyfill less of Node并且不提供任何内容。例如，一个包可以使用Node`process`，这反过来会使你的包膨胀。要禁用它，请将`node.process`设置为`false`。要完全禁用polyfilling，请将`node`直接设置为`false`。 [请参阅webpack文档]（https://webpack.js.org/configuration/node/）获取默认值。
* Polyfill less of Node and provide nothing instead. For example, a package could use Node `process` which in turn will bloat your bundle. To disable it, set `node.process` to `false`. To disable polyfilling entirely, set `node` to `false` directly. [See webpack documentation](https://webpack.js.org/configuration/node/) for the default values.
*将更少的包推送到**动态加载库**（DLL）以避免不必要的处理。 [官方webpack示例]（https://github.com/webpack/webpack/tree/master/examples/dll-user）在[Rob Knight的博客文章]（https://robertknight.me.uk）中找到了重点。 / posts / webpack-dll-plugins /）进一步解释了这个想法。 [autodll-webpack-plugin]（https://www.npmjs.com/package/autodll-webpack-plugin）可以自动完成此过程。
* Push bundles that change less to **Dynamically Loaded Libraries** (DLL) to avoid unnecessary processing. The [official webpack example](https://github.com/webpack/webpack/tree/master/examples/dll-user) gets to the point while [Rob Knight's blog post](https://robertknight.me.uk/posts/webpack-dll-plugins/) explains the idea further. [autodll-webpack-plugin](https://www.npmjs.com/package/autodll-webpack-plugin) can automate the process.

###插件特定优化
### Plugin Specific Optimizations

有一系列特定于插件的优化需要考虑：
There are a series of plugin specific optimizations to consider:

*利用[hard-source-webpack-plugin]（https://www.npmjs.com/package/hard-source-webpack-plugin）等插件进行缓存，以避免不必要的工作。
* Utilize caching through plugins like [hard-source-webpack-plugin](https://www.npmjs.com/package/hard-source-webpack-plugin) to avoid unnecessary work.
*在开发过程中使用相同但更轻的插件和装载器替代品。用[HtmlPlugin]（https://gist.github.com/bebraw/5bd5ebbb2a06936e052886f5eb1e6874）替换`HtmlWebpackPlugin`的方法要少得多。
* Use equivalent, but lighter alternatives, of plugins and loaders during development. Replacing `HtmlWebpackPlugin` with a [HtmlPlugin](https://gist.github.com/bebraw/5bd5ebbb2a06936e052886f5eb1e6874) that does far less is one direction.

{pagebreak}

### Loader特定优化
### Loader Specific Optimizations

加载器也有它们的优化：
Loaders have their optimizations as well:

*通过在开发期间跳过装载机来执行较少的处理。特别是如果你使用的是现代浏览器，则可以跳过使用* babel-loader *或完全相同的内容。
* Perform less processing by skipping loaders during development. Especially if you are using a modern browser, you can skip using *babel-loader* or equivalent altogether.
*对JavaScript特定的加载器使用`include`或`exclude`。 Webpack默认遍历* node_modules *并对文件执行* babel-loader *，除非它已正确配置。
* Use either `include` or `exclude` with JavaScript specific loaders. Webpack traverses *node_modules* by default and executes *babel-loader* over the files unless it has been configured correctly.
*使用[cache-loader]（https://www.npmjs.com/package/cache-loader）将昂贵的加载器（例如图像处理）的结果缓存到磁盘。
* Cache the results of expensive loaders (e.g., image manipulation) to the disk using the [cache-loader](https://www.npmjs.com/package/cache-loader).
*使用[thread-loader]（https://www.npmjs.com/package/thread-loader）并行执行昂贵的加载器。鉴于工作人员在Node中有开销，使用* thread-loader *只有在并行操作很重的情况下才值得。
* Parallelize the execution of expensive loaders using [thread-loader](https://www.npmjs.com/package/thread-loader). Given workers come with an overhead in Node, using *thread-loader* is worth it only if the parallelized operation is heavy.

##优化开发过程中的重新绑定速度
## Optimizing Rebundling Speed During Development

通过将开发设置指向库的缩小版本（例如React），可以在开发期间优化重新绑定时间。在React的情况下，你将失去基于“propType”的验证。如果速度很重要，这种技术是值得的。
It's possible to optimize rebundling times during development by pointing the development setup to a minified version of a library, such as React. In React's case, you lose `propType`-based validation. If speed is important, this technique is worth it.

`module.noParse`接受RegExp或RegExps数组。除了告诉webpack不要解析你想要使用的缩小文件之外，你还必须使用`resolve.alias`指向它`react`。别名的想法将在* Consuming Packages *章节中详细讨论。
`module.noParse` accepts a RegExp or an array of RegExps. In addition to telling webpack not to parse the minified file you want to use, you also have to point `react` to it by using `resolve.alias`. The aliasing idea is discussed in detail in the *Consuming Packages* chapter.

{pagebreak}

可以将核心思想封装在一个函数中：
It's possible to encapsulate the core idea within a function:

```javascript
exports.dontParse = ({ name, path }) => {
  const alias = {};
  alias[name] = path;

  return {
    module: {
      noParse: [new RegExp(path)],
    },
    resolve: {
      alias,
    },
  };
};
```

要使用该功能，你可以按如下方式调用它：
To use the function, you would call it as follows:

```javascript
dontParse({
  name: "react",
  path: path.resolve(
    __dirname, "node_modules/react/cjs/react.production.min.js",
  ),
}),
```

在此更改之后，应用程序应该更快地重建，具体取决于底层实现。该技术也可以应用于生产。
After this change, the application should be faster to rebuild depending on the underlying implementation. The technique can also be applied to production.

如果你想忽略所有`* .min.js`文件，那么`module.noParse`接受一个正则表达式，你可以将它设置为`/ \ .min \ .js /`。
Given `module.noParse` accepts a regular expression if you wanted to ignore all `*.min.js` files, you could set it to `/\.min\.js/`.

W>并非所有模块都支持`module.noParse`。它们不应该引用`require`，`define`或类似的东西，因为它会导致`Uncaught ReferenceError：require is not defined` error。
W> Not all modules support `module.noParse`. They should not have a reference to `require`, `define`, or similar, as that leads to an `Uncaught ReferenceError: require is not defined` error.

##结论
## Conclusion

你可以通过多种方式优化webpack的性能。在转向涉及更多的技术之前，通常最好先使用更易于理解的技术。你必须使用的确切方法取决于项目。
You can optimize webpack's performance in multiple ways. Often it's a good idea to start with more accessible techniques before moving to more involved ones. The exact methods you have to use, depend on the project.

回顾一下：
To recap:

*从最先实施的高级技术开始。
* Start with higher level techniques that are fast to implement first.
*较低级别的技术更多涉及但是获得胜利。
* Lower level techniques are more involved but come with their wins.
*由于webpack默认使用单个实例运行，因此并行化是值得的。
* Since webpack runs using a single instance by default, parallelizing is worthwhile.
*特别是在开发过程中，由于现代浏览器，跳过工作是可以接受的。
* Especially during development, skipping work can be acceptable thanks to modern browsers.

T> [官方构建性能指南]（https://webpack.js.org/guides/build-performance/）有更多提示。另请参阅[保持webpack Fast：现场指南以获得更好的构建性能]（https://slack.engineering/keep-webpack-fast-a-field-guide-for-better-build-performance-f56a5995e8f1），[
T> [The official build performance guide](https://webpack.js.org/guides/build-performance/) has more tips. See also [Keep webpack Fast: A Field Guide for Better Build Performance](https://slack.engineering/keep-webpack-fast-a-field-guide-for-better-build-performance-f56a5995e8f1), [
我们如何将webpack构建性能提高95％]（https://blog.box.com/blog/how-we-improved-webpack-build-performance-95/），[webpack优化 - 案例研究]（https： //medium.com/walmartlabs/webpack-optimization-a-case-study-92b130334b6c）和[Web基础知识]（https://developers.google.com/web/fundamentals/performance/webpack/）。
How we improved webpack build performance by 95%](https://blog.box.com/blog/how-we-improved-webpack-build-performance-95/), [webpack optimization — A Case Study](https://medium.com/walmartlabs/webpack-optimization-a-case-study-92b130334b6c), and [Web Fundamentals by Google](https://developers.google.com/web/fundamentals/performance/webpack/).

