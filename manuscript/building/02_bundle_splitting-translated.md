#Bundle Splitting
# Bundle Splitting

目前，应用程序的生产版本是单个JavaScript文件。如果应用程序已更改，则客户端也必须下载供应商依赖项。
Currently, the production version of the application is a single JavaScript file. If the application is changed, the client must download vendor dependencies as well.

最好只下载更改的部分。如果供应商依赖项发生更改，则客户端应仅获取供应商依赖项。实际的应用程序代码也是如此。 **使用`optimization.splitChunks.cacheGroups`可以实现捆绑拆分**。在生产模式下运行时，[webpack 4可以开箱即可执行一系列拆分]（https://gist.github.com/sokra/1522d586b8e5c0f5072d7565c2bee693）但在这种情况下，我们会手动执行某些操作。
It would be better to download only the changed portion. If the vendor dependencies change, then the client should fetch only the vendor dependencies. The same goes for actual application code. **Bundle splitting** can be achieved using `optimization.splitChunks.cacheGroups`. When running in production mode, [webpack 4 can perform a series of splits out of the box](https://gist.github.com/sokra/1522d586b8e5c0f5072d7565c2bee693) but in this case, we'll do something manually.

T>要正确使bundle无效，必须将哈希附加到生成的bundle中，如* Add Hashes to Filenames *章节中所述。
T> To invalidate the bundles correctly, you have to attach hashes to the generated bundles as discussed in the *Adding Hashes to Filenames* chapter.

##捆绑拆分的想法
## The Idea of Bundle Splitting

通过捆绑拆分，您可以将供应商依赖项推送到自己的捆绑包，并从客户端级别缓存中受益。该过程可以以应用程序的整个大小保持不变的方式完成。鉴于需要执行的请求越多，就会产生轻微的开销。但缓存的好处弥补了这一成本。
With bundle splitting, you can push the vendor dependencies to a bundle of their own and benefit from client level caching. The process can be done in such a way that the whole size of the application remains the same. Given there are more requests to perform, there's a slight overhead. But the benefit of caching makes up for this cost.

为了给你一个快速的例子，你可以得到* main.js *（10 kB）和* vendor.js *（90 kB），而不是* main.js *（100 kB）。现在，对于已经使用过该应用程序的客户端，对应用程序所做的更改很便宜。
To give you a quick example, instead of having *main.js* (100 kB), you could end up with *main.js* (10 kB) and *vendor.js* (90 kB). Now changes made to the application are cheap for the clients that have already used the application earlier.

缓存伴随着它的问题。其中之一是缓存失效。与此相关的潜在方法在*添加散列到文件名*章节中讨论。
Caching comes with its problems. One of those is cache invalidation. A potential approach related to that is discussed in the *Adding Hashes to Filenames* chapter.

捆绑分裂不是唯一的出路。 * Code Splitting *章节讨论了另一种更细粒度的方法。
Bundle splitting isn't the only way out. The *Code Splitting* chapter discusses another, more granular way.

##添加要拆分的东西
## Adding Something to Split

鉴于暂时没有拆分供应商包，你应该在那里添加一些东西。首先将React添加到项目中：
Given there's not much to split into the vendor bundle yet, you should add something there. Add React to the project first:

```bash
npm install react react-dom --save
```

然后让项目依赖于它：
Then make the project depend on it:

** SRC / index.js **
**src/index.js**

```
leanpub-start-insert
import "react";
import "react-dom";
leanpub-end-insert
...
```

执行`npm run build`以获得基线构建。你应该得到如下的东西：
Execute `npm run build` to get a baseline build. You should end up with something as below:

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

如您所见，* main.js *很大。这是接下来要解决的问题。
As you can see, *main.js* is big. That is something to fix next.

##设置`vendor` Bundle
## Setting Up a `vendor` Bundle

在webpack 4之前，曾经有`CommonsChunkPlugin`来管理bundle拆分。该插件已被自动化和配置所取代。要从* node_modules *目录中提取供应商包，请按如下方式调整代码：
Before webpack 4, there used to be `CommonsChunkPlugin` for managing bundle splitting. The plugin has been replaced with automation and configuration. To extract a vendor bundle from the *node_modules* directory, adjust the code as follows:

** ** webpack.config.js
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

如果您现在尝试生成构建（`npm run build`），您应该看到以下内容：
If you try to generate a build now (`npm run build`), you should see something along this:

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

现在捆绑看起来应该如此。下图说明了当前的情况。
Now the bundles look the way they should. The image below illustrates the current situation.

![Main and vendor bundles after applying configuration](images/bundle_02.png)

{pagebreak}

##控制捆绑分裂
## Controlling Bundle Splitting

可以使用针对* node_modules *的显式测试重写上面的配置，如下所示：
The configuration above can be rewritten with an explicit test against *node_modules* as below:

** ** webpack.config.js
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

如果您不希望依赖自动化，则使用此格式可以更好地控制拆分过程。
Following this format gives you more control over the splitting process if you don't prefer to rely on automation.

##拆分和合并块
## Splitting and Merging Chunks

Webpack通过两个插件提供对生成的块的更多控制：`AggressiveSplittingPlugin`和`AggressiveMergingPlugin`。前者允许您发出更多和更小的包。由于新标准的工作方式，HTTP / 2的行为很方便。
Webpack provides more control over the generated chunks by two plugins: `AggressiveSplittingPlugin` and `AggressiveMergingPlugin`. The former allows you to emit more and smaller bundles. The behavior is handy with HTTP/2 due to the way the new standard works.

{pagebreak}

以下是激进分裂的基本思路：
Here's the basic idea of aggressive splitting:

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

如果您分成多个小捆绑，那么在缓存中会失败，这是一种权衡。您还会在HTTP / 1环境中获得请求开销。目前，由于[插件中的一个错误]启用`HtmlWebpackPlugin`（https://github.com/ampedandwired/html-webpack-plugin/issues/446），该方法不起作用。
There's a trade-off as you lose out in caching if you split to multiple small bundles. You also get request overhead in HTTP/1 environment. For now, the approach doesn't work when `HtmlWebpackPlugin` is enabled due to [a bug in the plugin](https://github.com/ampedandwired/html-webpack-plugin/issues/446).

积极的合并插件以相反的方式工作，并允许您将小包组合成更大的：
The aggressive merging plugin works the opposite way and allows you to combine small bundles into bigger ones:

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

如果使用webpack **记录**，则可以使用这些插件获得良好的缓存行为。这个想法在*添加哈希到文件名*章节中有详细讨论。
It's possible to get good caching behavior with these plugins if a webpack **records** are used. The idea is discussed in detail in the *Adding Hashes to Filenames* chapter.

`webpack.optimize`包含`LimitChunkCountPlugin`和`MinChunkSizePlugin`，它们可以进一步控制块大小。
`webpack.optimize` contains `LimitChunkCountPlugin` and `MinChunkSizePlugin` which give further control over chunk size.

T> Tobias Koppers在webpack的官方博客上讨论[积极合并]（https://medium.com/webpack/webpack-http-2-7083ec3f3ce6）。
T> Tobias Koppers discusses [aggressive merging in detail at the official blog of webpack](https://medium.com/webpack/webpack-http-2-7083ec3f3ce6).

Webpack中的##块类型
## Chunk Types in Webpack

在上面的示例中，您使用了不同类型的webpack块。 Webpack处理三种类型的块：
In the example above, you used different types of webpack chunks. Webpack treats chunks in three types:

* **条目块**  - 条目块包含webpack运行时和它随后加载的模块。
* **Entry chunks** - Entry chunks contain webpack runtime and modules it then loads.
* **正常块**  - 正常块**不包含webpack运行时。相反，这些可以在应用程序运行时动态加载。为这些生成合适的包装器（例如JSONP）。在设置代码拆分时，您将在下一章中生成正常的块。
* **Normal chunks** - Normal chunks **don't** contain webpack runtime. Instead, these can be loaded dynamically while the application is running. A suitable wrapper (JSONP for example) is generated for these. You generate a normal chunk in the next chapter as you set up code splitting.
* **初始块**  - 初始块是正常的块，计入应用程序的初始加载时间。作为用户，您不必关心这些。这是入口块和正常块之间的分离，这很重要。
* **Initial chunks** - Initial chunks are normal chunks that count towards initial loading time of the application. As a user, you don't have to care about these. It's the split between entry chunks and normal chunks that is important.

##结论
## Conclusion

与之前相比，现在情况好转。注意小`main`包与`vendor`包相比如何。要从此拆分中受益，请在本书的下一部分的“添加散列到文件名*”一章中设置缓存。
The situation is better now compared to the earlier. Note how small `main` bundle compared to the `vendor` bundle. To benefit from this split, you set up caching in the next part of this book in the *Adding Hashes to Filenames* chapter.

回顾一下：
To recap:

* Webpack允许您通过`optimization.splitChunks.cacheGroups`字段从配置条目中拆分包。它在生产模式下默认执行捆绑拆分。
* Webpack allows you to split bundles from configuration entries through the `optimization.splitChunks.cacheGroups` field. It performs bundle splitting by default in production mode as well.
*供应商捆绑包含项目的第三方代码。可以通过检查模块的导入位置来检测供应商依赖性。
* A vendor bundle contains the third party code of your project. The vendor dependencies can be detected by inspecting where the modules are imported.
* Webpack通过特定的插件提供对块分的更多控制，例如`AggressiveSplittingPlugin`和`AggressiveMergingPlugin`。主要是拆分插件在面向HTTP / 2的设置中可以很方便。
* Webpack offers more control over chunking through specific plugins, such as `AggressiveSplittingPlugin` and `AggressiveMergingPlugin`. Mainly the splitting plugin can be handy in HTTP/2 oriented setups.
*内部webpack依赖于三种块类型：入口块，正常块和初始块。
* Internally webpack relies on three chunk types: entry, normal, and initial chunks.

在下一章中，您将了解代码拆分和按需加载代码。
In the next chapter, you'll learn about code splitting and loading code on demand.

