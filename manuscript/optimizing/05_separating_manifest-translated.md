# 分离 Manifest

当 webpack 输出 bundle 时，还会维护一个 **manifest**（清单）。你可以在此项目中生成的 *vendor* 包中找到它。manifest 描述了 webpack 需要加载的文件。可以将其拆分，以更快地开始加载项目文件，而不必等待加载 *vendor* bundle 完毕。

如果 webpack 生成的 hash 更改，则清单也会随之更改。结果，vendor bundle 的内容也会发生变化，缓存因此无效。通过将 manifest 提取到单独文件或将其内联写入项目的 **index.html**，可以解决这个问题。

## 提取 Manifest

在 **Bundle Splitting** 章节中设置 `extractBundles` 时，大部分工作已经完成。要提取 Manifest，请按如下方式定义 `optimization.runtimeChunk`：

**webpack.config.js**

```javascript
const productionConfig = merge([
  ...
  {
    optimization: {
      splitChunks: {
        ...
      },
leanpub-start-insert
      runtimeChunk: {
        name: "manifest",
      },
leanpub-end-insert
    },
  },
  ...
]);
```

命名为 `manifest` 是一个惯例。你可以使用任何其他名称，它仍然可以正常工作。

如果你现在构建项目（`npm run build`），你应该可以看到：

```bash
Hash: 2e1c61341de0fd7e0e5c
Version: webpack 4.1.1
Time: 3347ms
Built at: 3/16/2018 6:24:51 PM
                   Asset       Size  Chunks             Chunk Names
leanpub-start-insert
       manifest.d41d.css    0 bytes       1  [emitted]  manifest
leanpub-end-insert
               0.73a8.js  160 bytes       0  [emitted]
    vendors~main.3af5.js   96.8 KiB       2  [emitted]  vendors~main
            main.8da2.js  546 bytes       3  [emitted]  main
           main.5524.css    1.2 KiB       3  [emitted]  main
   vendors~main.3dd5.css   1.32 KiB       2  [emitted]  vendors~main
leanpub-start-insert
        manifest.8cac.js   1.81 KiB       1  [emitted]  manifest
leanpub-end-insert
           0.73a8.js.map  203 bytes       0  [emitted]
leanpub-start-insert
    manifest.8cac.js.map     10 KiB       1  [emitted]  manifest
leanpub-end-insert
vendors~main.3af5.js.map    235 KiB       2  [emitted]  vendors~main
        main.8da2.js.map   1.45 KiB       3  [emitted]  main
              index.html  460 bytes          [emitted]
...
```

此更改提供了一个包含清单的单独文件。在上面的输出中，它已被标记为`manifest`块名称。因为设置是使用`HtmlWebpackPlugin`，所以不需要担心自己加载清单，因为插件添加了对* index.html *的引用。
This change gave a separate file that contains the manifest. In the output above it has been marked with `manifest` chunk name. Because the setup is using `HtmlWebpackPlugin`, there is no need to worry about loading the manifest ourselves as the plugin adds a reference to *index.html*.

插件，例如[inline-manifest-webpack-plugin]（https://www.npmjs.com/package/inline-manifest-webpack-plugin）和[html-webpack-inline-chunk-plugin]（https：/ /www.npmjs.com/package/html-webpack-inline-chunk-plugin），[assets-webpack-plugin]（https://www.npmjs.com/package/assets-webpack-plugin），使用` HtmlWebpackPlugin`并允许你在* index.html *中编写清单以避免请求。
Plugins, such as [inline-manifest-webpack-plugin](https://www.npmjs.com/package/inline-manifest-webpack-plugin) and [html-webpack-inline-chunk-plugin](https://www.npmjs.com/package/html-webpack-inline-chunk-plugin), [assets-webpack-plugin](https://www.npmjs.com/package/assets-webpack-plugin), work with `HtmlWebpackPlugin` and allow you to write the manifest within *index.html* to avoid a request.

尝试调整 **src/index.js** 并查看哈希值如何变化。这次它应该**不**使供应商包无效，只有清单和应用程序包名称应该变得不同。
Try adjusting *src/index.js* and see how the hashes change. This time around it should **not** invalidate the vendor bundle, and only the manifest and app bundle names should become different.

T> 要更好地了解 manifest 内容，请在开发模式下运行构建，或通过配置将 `none` 传递给模式。你应该看到熟悉的东西。
T> To get a better idea of the manifest contents, run the build in development mode or pass `none` to mode through configuration. You should see something familiar there.

T>要与资产管道集成，你可以考虑使用[chunk-manifest-webpack-plugin]（https://www.npmjs.com/package/chunk-manifest-webpack-plugin），[webpack-manifest-]等插件插件]（https://www.npmjs.com/package/webpack-manifest-plugin），[webpack-assets-manifest]（https://www.npmjs.com/package/webpack-assets-manifest），或者[的WebPack护栏舱单-插件]（https://www.npmjs.com/package/webpack-rails-manifest-plugin）。这些解决方案发出JSON，将原始资产路径映射到新资产路径。
T> To integrate with asset pipelines, you can consider using plugins like [chunk-manifest-webpack-plugin](https://www.npmjs.com/package/chunk-manifest-webpack-plugin), [webpack-manifest-plugin](https://www.npmjs.com/package/webpack-manifest-plugin), [webpack-assets-manifest](https://www.npmjs.com/package/webpack-assets-manifest), or [webpack-rails-manifest-plugin](https://www.npmjs.com/package/webpack-rails-manifest-plugin). These solutions emit JSON that maps the original asset path to the new one.

T> 可以通过CDN加载流行的依赖项（如React）来进一步改进构建。这将进一步减少供应商包的大小，同时在项目中添加外部依赖项。这个想法是，如果用户之前已经点击了CDN，那么缓存可以像这里一样启动。
T> The build can be improved further by loading popular dependencies, such as React, through a CDN. That would decrease the size of the vendor bundle even further while adding an external dependency on the project. The idea is that if the user has hit the CDN earlier, caching can kick in like here.

## 使用 Records

正如 **Bundle Splitting** 章节中所提到的，如 `AggressiveSplittingPlugin` 之类的插件使用 **Records** 来实现缓存。Records 会比上述方法更优秀。
As mentioned in the *Bundle Splitting* chapter, plugins such as `AggressiveSplittingPlugin` use **records** to implement caching. The approaches discussed above are still valid, but records go one step further.

Records 用于跨单独的构建存储模块ID。问题是你需要保存此文件。如果你在本地构建，则可以选择将其包含在版本控制中。
Records are used for storing module IDs across separate builds. The problem is that you need to save this file. If you build locally, one option is to include it in your version control.

要生成 **records.json** 文件，请按如下方式调整配置：

**webpack.config.js**

```javascript
const productionConfig = merge([
  {
    ...
leanpub-start-insert
    recordsPath: path.join(__dirname, "records.json"),
leanpub-end-insert
  },
  ...
]);
```

现在构建项目（`npm run build`），应该可以在项目根目录下看到一个新文件 **records.json**。下一次 webpack 构建时，它会获取信息并重写文件（如果出现修改）。

如果你有一个复杂的代码拆分设置并希望确保拆分部分获得正确的缓存行为，则 records 特别有用。最大的问题是维护记录文件。
Records are particularly valuable if you have a complicated setup with code splitting and want to make sure the split parts gain correct caching behavior. The biggest problem is maintaining the record file.

T> `recordsInputPath` 和 `recordsOutputPath` 对输入和输出提供了更精细的控制，但通常只设置 `recordsPath` 就足够了。

W>如果你改变webpack处理模块ID的方式（即删除`HashedModuleIdsPlugin`），仍然会考虑可能的现有记录！如果要使用新模块ID方案，则还必须删除记录文件。
W> If you change the way webpack handles module IDs (i.e., remove `HashedModuleIdsPlugin`), possible existing records are still taken into account! If you want to use the new module ID scheme, you have to delete your records file as well.

{pagebreak}

## 总结

该项目现在具有基本的缓存能力。如果你尝试修改 **index.js** 或 **component.js**，则 vendor bundle 应保持不变。

回顾一下：

* Webpack 会维护一个 **manifest**，其中包含运行该应用程序所需的信息。
* 如果 manifest 发生更改，这会使包含的 bundle 无效。
* 一些插件可以把 manifest 写入生成的 *index.html*。也可以将信息提取到 JSON 文件中。 JSON在**服务器渲染**中可以派上用场。
* **Records** 允许你跨 builds 存储模块 ID。但缺点是你必须跟踪 records 文件。

你将在下一章中学习如何分析构建得到的文件，它对于理解和改进构建至关重要。

