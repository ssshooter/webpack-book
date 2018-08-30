# Bundle 拆分

目前，应用程序的生产版本是单个 JavaScript 文件。如果应用程序发生更改，客户端必须重新下载供应商依赖项（引入的其他 npm 包）。

最好只下载更改的部分。如果供应商依赖项发生更改，则客户端应仅获取供应商依赖项。实际的应用程序代码也是如此。 **使用`optimization.splitChunks.cacheGroups`可以实现捆绑拆分**。在生产模式下运行时，[webpack 4可以开箱即可执行一系列拆分]（https://gist.github.com/sokra/1522d586b8e5c0f5072d7565c2bee693）但在这种情况下，我们会手动执行某些操作。
It would be better to download only the changed portion. If the vendor dependencies change, then the client should fetch only the vendor dependencies. The same goes for actual application code. **Bundle splitting** can be achieved using `optimization.splitChunks.cacheGroups`. When running in production mode, [webpack 4 can perform a series of splits out of the box](https://gist.github.com/sokra/1522d586b8e5c0f5072d7565c2bee693) but in this case, we'll do something manually.

T> 要使旧的 bundle 无效，必须将 hash 添加到生成的 bundle 中，之后的 *Adding Hashes to Filenames* 章节将会提到。

## Bundle 拆分的思路

通过 bundle 拆分，你可以将依赖的其他 npm 包打包到自己的 bundle，这对客户端缓存大有脾益。如此处理后，整个应用的包大小基本不变，但浏览器请求会增加，以此为代价，好处是你可以缓存一些不常变化的文件。（译者注：这里是重点，拆分不为别的，就是为了部分缓存）

为了给你一个快速的例子，你可以得到 **main.js**（10 kB）和 **vendor.js**（90 kB），而不是 **main.js**（100 kB）。这么做对于二次访问该应用会很方便。

缓存也会带来一些问题，其中之一是缓存失效。相关问题会在 *Adding Hashes to Filenames* 章节中讨论。

Bundle 拆分不是唯一的出路。**Code Splitting** 章节中会讨论另一种更细粒度的方法。

## 添加要拆分的东西

现在还没什么可以拆分出去，所以应该先添加一些代码。首先将 React 添加到项目中：

```bash
npm install react react-dom --save
```

然后让项目依赖于它：

**src/index.js**

```
leanpub-start-insert
import "react";
import "react-dom";
leanpub-end-insert
...
```

执行 `npm run build` 获取基础构建。你应该得到如下的东西：

```bash
Hash: 80f9bb6fc04c54949644
Version: webpack 4.1.1
Time: 3276ms
Built at: 3/16/2018 4:59:25 PM
       Asset       Size  Chunks             Chunk Names
leanpub-start-insert
     main.js   97.5 KiB       0  [emitted]  main
leanpub-end-insert
    main.css   3.49 KiB       0  [emitted]  main
 main.js.map    240 KiB       0  [emitted]  main
main.css.map   85 bytes       0  [emitted]  main
  index.html  220 bytes          [emitted]
Entrypoint main = main.js main.css main.js.map main.css.map
...
```

如你所见，**main.js** 的体积很大，这是接下来要解决的问题。

## 设置 `vendor` bundle

在 webpack 4 之前，`CommonsChunkPlugin` 可以用于管理 bundle 拆分。该插件已被自动化和配置所取代。要从 **node_modules** 目录中提取第三方 bundle，可以如此调整代码：

**webpack.config.js**

```javascript
const productionConfig = merge([
  ...
leanpub-start-insert
  {
    optimization: {
      splitChunks: {
        chunks: "initial",
      },
    },
  },
leanpub-end-insert
]);
```

如果你现在构建（`npm run build`），你应该看到以下内容：

```bash
Hash: 6c499f10237fdbb07378
Version: webpack 4.1.1
Time: 3172ms
Built at: 3/16/2018 5:00:03 PM
               Asset       Size  Chunks             Chunk Names
leanpub-start-insert
     vendors~main.js   96.8 KiB       0  [emitted]  vendors~main
leanpub-end-insert
             main.js   1.35 KiB       1  [emitted]  main
            main.css   1.27 KiB       1  [emitted]  main
leanpub-start-insert
    vendors~main.css   2.27 KiB       0  [emitted]  vendors~main
 vendors~main.js.map    235 KiB       0  [emitted]  vendors~main
vendors~main.css.map   93 bytes       0  [emitted]  vendors~main
leanpub-end-insert
         main.js.map   7.11 KiB       1  [emitted]  main
        main.css.map   85 bytes       1  [emitted]  main
          index.html  329 bytes          [emitted]
Entrypoint main = vendors~main.js vendors~main.css ...
...
```

（译者注：chunk 和 bundle 大概可以理解为一个东西）
现在的 bundle 看起来应如下图所示：

![Main and vendor bundles after applying configuration](images/bundle_02.png)

{pagebreak}

## bundle 拆分的控制

可以调整一下上面的配置，让拆分只针对 **node_modules**，如下所示：

**webpack.config.js**

```javascript
const productionConfig = merge([
  ...
leanpub-start-insert
  {
    optimization: {
      splitChunks: {
        cacheGroups: {
          commons: {
            test: /[\\/]node_modules[\\/]/,
            name: "vendor",
            chunks: "initial",
          },
        },
      },
    },
  },
leanpub-end-insert
]);
```

如果你不希望依赖自动配置，可以使用此格式可以更好地控制拆分处理。

## 拆分和合并 Chunk

Webpack 通过两个插件提供对生成的 chunk 作更多控制：`AggressiveSplittingPlugin` 和 `AggressiveMergingPlugin`。前者允许你发出更多和更小的 bundle，这对 HTTP/2 来说十分方便。

{pagebreak}

以下是积极拆分（aggressive splitting）的基本思路：

```javascript
{
  plugins: [
    new webpack.optimize.AggressiveSplittingPlugin({
        minSize: 10000,
        maxSize: 30000,
    }),
  ],
},
```

如果你分成多个小 bundle，那么在缓存中会失败，这是一种权衡。你还会在HTTP / 1环境中获得请求开销。目前，由于[插件中的一个错误]启用`HtmlWebpackPlugin`（https://github.com/ampedandwired/html-webpack-plugin/issues/446），该方法不起作用。
There's a trade-off as you lose out in caching if you split to multiple small bundles. You also get request overhead in HTTP/1 environment. For now, the approach doesn't work when `HtmlWebpackPlugin` is enabled due to [a bug in the plugin](https://github.com/ampedandwired/html-webpack-plugin/issues/446).

积极合并（aggressive merging）插件以相反的方式工作，允许你将小 bundle 组合成更大的：

```javascript
{
  plugins: [
    new AggressiveMergingPlugin({
        minSizeReduce: 2,
        moveToParents: true,
    }),
  ],
},
```

如果使用 webpack **记录**，则可以使用这些插件获得良好的缓存行为。这个想法在 *Adding Hashes to Filenames* 章节中有详细讨论。
It's possible to get good caching behavior with these plugins if a webpack **records** are used. The idea is discussed in detail in the *Adding Hashes to Filenames* chapter.

`webpack.optimize` 的 `LimitChunkCountPlugin` 和 `MinChunkSizePlugin`，它们可以进一步控制 chunk 大小。

T> Tobias Koppers 在 webpack的官方博客上讨论[积极合并（aggressive merging）](https://medium.com/webpack/webpack-http-2-7083ec3f3ce6)。

## Webpack 中的 Chunk 类型

在上面的示例中，你使用了不同类型的 webpack chunk。 Webpack 把 chunk 分为三种类型：

* **Entry chunks** - Entry chunks 包括 webpack 运行时和它即将加载的模块。
* **Normal chunks** - Normal chunks **不** 包含 webpack 运行时。它们可以在应用运行时被动态加载。webpack 会为其生成一个合适的包裹器（例如 JSONP）。下一章做代码分割生成的便是 normal chunk。
* **Initial chunks** - Initial chunks 是一种 normal chunks，但是它的加载算进初始加载中。作为用户不用太关注这类，知道它们介于两者之间就好了。（It's the split between entry chunks and normal chunks that is important）

## 总结

与之前相比，情况好多了，`main` bundle 在拆分后比 `vendor` bundle 小多了。不过要从此受益，请在看本书下一部分的 *Adding Hashes to Filenames* 章节，了解缓存相关设置。

回顾一下：

* Webpack允许你通过 `optimization.splitChunks.cacheGroups` 字段在配置中拆分 bundle。在生产模式下，会默认执行 bundle 拆分。
* vendor bundle 包含项目的第三方代码。可以通过匹配模块的导入位置来检测第三方依赖。
* Webpack 通过特定的插件提供对 chunk 分割的更多控制，例如 `AggressiveSplittingPlugin` 和 `AggressiveMergingPlugin`。拆分插件主要是在面向 HTTP/2 的设置中可以方便。
* 内部 webpack 依赖于三种 chunk 类型：入口 chunk，正常 chunk 和初始 chunk。

在下一章中，你将了解代码拆分和按需加载代码。

