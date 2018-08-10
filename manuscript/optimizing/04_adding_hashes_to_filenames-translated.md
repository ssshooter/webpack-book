# 文件名中加入hash

即使生成的构建工作，它使用的文件名也是有问题的。它不允许有效利用客户端级别缓存，因为无法判断文件是否已更改。可以通过在文件名中包含hash来实现缓存失效。
Even though the generated build works the file names it uses is problematic. It doesn't allow to leverage client level cache efficiently as there's no way tell whether or not a file has changed. Cache invalidation can be achieved by including a hash to the filenames.

## 占位符
## Placeholders

Webpack 为此提供**占位符**。这些字符串用于将特定信息附加到 webpack 输出。其中最有价值的是：

* `[id]` - 返回 chunk id。
* `[path]` - 返回文件路径。
* `[name]` - 返回文件名。
* `[ext]` - 返回扩展名。 `[ext]`适用于大多数可用字段。`MiniCssExtractPlugin`是一个值得注意的例外。
* `[hash]` - 返回构建hash。如果构建的任何部分发生变化，hash值也会发生变化。
* `[chunkhash]` - 返回一个 entry chunk-specific hash。配置中定义的每个 `entry` 都会收到自己的hash值。如果条目的任何部分发生更改，则hash值也会更改。根据定义，`[chunkhash]` 比 `[hash]` 更精细。
* `[contenthash]` - 返回根据内容生成的hash。

`hash` 和 `chunkhash` 用在生产环境是极好的，但hash在开发环境没有太大作用。

T> 可以使用特定语法对 `hash` 和 `chunkhash` 进行分割：`[chunkhash：4]`。这样会生成 `8c4c`，而不是像 `8c4cbfdb91ff93f3f3c5` 这样的hash。

T>有更多可用选项，你甚至可以修改hash和摘要类型，如[loader-utils]（https://www.npmjs.com/package/loader-utils#interpolatename）文档中所述。
T> There are more options available, and you can even modify the hashing and digest type as discussed at [loader-utils](https://www.npmjs.com/package/loader-utils#interpolatename) documentation.

### 占位符的一个例子

假设有以下配置：

```javascript
{
  output: {
    path: PATHS.build,
    filename: "[name].[chunkhash].js",
  },
},
```

Webpack 会基于这个配置生成文件名：

```bash
main.d587bbd6e38337f5accd.js
vendor.dc746a5db4ed650296e1.js
```

如果文件内容改变了，则 hash 也会随之而变，缓存也因此无效。准确地说，浏览器请求新文件，如果只更新了 `main` 包，则只需要再次请求 `main`。

通过生成静态文件名并通过查询字符串使缓存无效（即 `main.js?d587bbd6e38337f5accd`）可以实现相同的结果。问号后的部分使缓存无效。[Steve Souders](http://www.stevesouders.com/blog/2008/08/23/revving-filenames-dont-use-querystring/) 指出，将 hash 值附加到文件名上是最高性能的选择。

{pagebreak}

## 设置 hash

构建需要调整以生成适当的hash。图像和字体应该接收`hash`，而块应该在名称中使用`chunkhash`来正确地使它们无效：
The build needs tweaking to generate proper hashes. Images and fonts should receive `hash` while chunks should use `chunkhash` in their names to invalidate them correctly:

**webpack.config.js**

```javascript
const productionConfig = merge([
leanpub-start-insert
  {
    output: {
      chunkFilename: "[name].[chunkhash:4].js",
      filename: "[name].[chunkhash:4].js",
    },
  },
leanpub-end-insert
  ...
  parts.loadImages({
    options: {
      limit: 15000,
leanpub-start-delete
      name: "[name].[ext]",
leanpub-end-delete
leanpub-start-insert
      name: "[name].[hash:4].[ext]",
leanpub-end-insert
    },
  }),
  ...
]);
```


W> `[hash]` is defined differently for *file-loader* than for the rest of webpack. It's calculated based on file **content**. See [file-loader documentation](https://www.npmjs.com/package/file-loader#placeholders) for further information.

如果你对提取的CSS也使用`chunkhash`，这会导致问题，因为代码通过JavaScript将CSS指向同一个条目。这意味着如果应用程序代码或CSS发生了变化，它将使两者无效。
If you used `chunkhash` for the extracted CSS as well, this would lead to problems as the code points to the CSS through JavaScript bringing it to the same entry. That means if the application code or CSS changed, it would invalidate both.

{pagebreak}

因此，你可以使用基于提取的内容生成的`contenthash`而不是`chunkhash`：
Therefore, instead of `chunkhash`, you can use `contenthash` that is generated based on the extracted content:

**webpack.parts.js**

```javascript
exports.extractCSS = ({ include, exclude, use }) => {
  // Output extracted CSS to a file
  const plugin = new MiniCssExtractPlugin({
leanpub-start-delete
    filename: "[name].css",
leanpub-end-delete
leanpub-start-insert
    filename: "[name].[contenthash:4].css",
leanpub-end-insert
  });

  ...
};
```

W> hash已被切片，以使输出在书中更好地适应。在实践中，你可以跳过切片。
W> The hashes have been sliced to make the output fit better in the book. In practice, you can skip slicing them.

如果你现在生成一个构建（`npm run build`），你应该看到一些东西：
If you generate a build now (`npm run build`), you should see something:

```bash
Hash: fb67c5fd35454da1d6ff
Version: webpack 4.1.1
Time: 3034ms
Built at: 3/16/2018 6:18:07 PM
                   Asset       Size  Chunks             Chunk Names
               0.0847.js  161 bytes       0  [emitted]
    vendors~main.d2f1.js   96.8 KiB       1  [emitted]  vendors~main
            main.745c.js   2.25 KiB       2  [emitted]  main
           main.5524.css    1.2 KiB       2  [emitted]  main
   vendors~main.3dd5.css   1.32 KiB       1  [emitted]  vendors~main
           0.0847.js.map  203 bytes       0  [emitted]
vendors~main.d2f1.js.map    235 KiB       1  [emitted]  vendors~main
        main.745c.js.map   11.4 KiB       2  [emitted]  main
              index.html  349 bytes          [emitted]
Entrypoint main = vendors~main.d2f1.js ...
...
```

文件现在有整齐的hash。为了证明它适用于样式，你可以尝试改变 *src/main.css* 并查看重建时hash会发生什么。
The files have neat hashes now. To prove that it works for styling, you could try altering *src/main.css* and see what happens to the hashes when you rebuild.

但是有一个问题。如果更改应用程序代码，它也会使供应商文件无效！解决这个问题需要提取**清单**，但在此之前，你可以改进生产构建处理模块ID的方式。
There's one problem, though. If you change the application code, it invalidates the vendor file as well! Solving this requires extracting a **manifest**, but before that, you can improve the way the production build handles module IDs.

## 结论

将与文件内容相关的hash包括在其名称中允许在客户端使它们无效。如果hash值已更改，则会强制客户端再次下载该资产。
Including hashes related to the file contents to their names allows to invalidate them on the client side. If a hash has changed, the client is forced to download the asset again.

回顾一下：

* Webpack 的**占位符**允许你设置文件名格式，在其中加入 hash。
* 最实用的占位符是 `[name]`，`[chunkhash]` 和 `[ext]`。chunk hash 基于它所属的入口生成。
* 如果你使用的是 `MiniCssExtractPlugin`，你应该使用 `[contenthash]`。这样，只有在它们的内容发生更改时，资源才会失效。

即使该项目现在产生hash，输出也不完美。问题是如果应用程序发生更改，它也会使供应商包无效。下一章将深入探讨该主题，并向你展示如何提取**清单**以解决问题。
Even though the project generates hashes now, the output isn't flawless. The problem is that if the application changes, it invalidates the vendor bundle as well. The next chapter digs deeper into the topic and shows you how to extract a **manifest** to resolve the issue.

