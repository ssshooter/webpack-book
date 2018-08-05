＃来源地图
# Source Maps

![Source maps in Chrome](images/sourcemaps.png)

当您的源代码经历了转换时，调试就成了问题。在浏览器中调试时，如何判断原始代码在哪里？ **源映射**通过提供原始代码和转换后的源代码之间的映射来解决此问题。除了源代码编译JavaScript之外，这也适用于样式。
When your source code has gone through transformations, debugging becomes a problem. When debugging in a browser, how to tell where the original code is? **Source maps** solve this problem by providing a mapping between the original and the transformed source code. In addition to source compiling to JavaScript, this works for styling as well.

一种方法是在开发期间跳过源映射，并依赖于浏览器对语言功能的支持。如果你使用没有任何扩展的ES2015并使用现代浏览器进行开发，这可以工作。这样做的好处是可以避免与源映射相关的所有问题，同时获得更好的性能。
One approach is to skip source maps during development and rely on browser support of language features. If you use ES2015 without any extensions and develop using a modern browser, this can work. The advantage of doing this is that you avoid all the problems related to source maps while gaining better performance.

如果您使用webpack 4和新的`mode`选项，该工具将在`development`模式下自动为您生成源映射。但是，生产使用需要注意。
If you are using webpack 4 and the new `mode` option, the tool will generate source maps automatically for you in `development` mode. Production usage requires attention, though.

T>如果您想更详细地了解源地图背后的想法，[阅读Ryan Seddon对该主题的介绍]（https://www.html5rocks.com/en/tutorials/developertools/sourcemaps/）。
T> If you want to understand the ideas behind source maps in greater detail, [read Ryan Seddon's introduction to the topic](https://www.html5rocks.com/en/tutorials/developertools/sourcemaps/).

T>要了解webpack如何处理源映射，请参阅该工具的作者的[source-map-visualization]（https://sokra.github.io/source-map-visualization/）。
T> To see how webpack handles source maps, see [source-map-visualization](https://sokra.github.io/source-map-visualization/) by the author of the tool.

##内联源映射和单独的源映射
## Inline Source Maps and Separate Source Maps

Webpack可以生成内联或单独的源映射文件。由于更好的性能，内置的在生产过程中是有价值的，而单独的那些在生产中使用非常方便，因为它可以保持束尺寸小。在这种情况下，加载源映射是可选的。
Webpack can generate both inline or separate source map files. The inline ones are valuable during development due to better performance while the separate ones are handy for production use as it keeps the bundle size small. In this case, loading source maps is optional.

您可能**不希望为生产包生成源映射，因为这样可以轻松检查您的应用程序。通过禁用源映射，您正在执行某种混淆。无论您是否要为生产启用源映射，它们都可以方便地进行分段。跳过源映射会加快构建速度，因为生成最佳质量的源映射可能是一项复杂的操作。
It's possible you **don't** want to generate a source map for your production bundle as this makes it effortless to inspect your application. By disabling source maps, you are performing a sort of obfuscation. Whether or not you want to enable source maps for production, they are handy for staging. Skipping source maps speeds up your build as generating source maps at the best quality can be a complicated operation.

**隐藏的源映射**仅提供堆栈跟踪信息。您可以将它们与监视服务连接，以便在应用程序崩溃时获取跟踪，从而允许您修复有问题的情况。虽然这不是理想的，但最好还是了解可能出现的问题。
**Hidden source maps** give stack trace information only. You can connect them with a monitoring service to get traces as the application crashes allowing you to fix the problematic situations. While this isn't ideal, it's better to know about possible problems than not.

T>研究您正在使用的加载器的文档以查看加载器特定提示是个好主意。例如，使用TypeScript，您必须设置特定标志以使其按预期工作。
T> It's a good idea to study the documentation of the loaders you are using to see loader specific tips. For example, with TypeScript, you have to set a particular flag to make it work as you expect.

##启用源地图
## Enabling Source Maps

Webpack提供了两种启用源映射的方法。有一个`devtool`快捷方式字段。您还可以找到两个插件，提供更多调整选项。插件将在本章末尾简要讨论。除了webpack之外，您还必须在用于开发的浏览器上启用对源映射的支持。
Webpack provides two ways to enable source maps. There's a `devtool` shortcut field. You can also find two plugins that give more options to tweak. The plugins are going to be discussed briefly at the end of this chapter. Beyond webpack, you also have to enable support for source maps at the browsers you are using for development.

###在Webpack中启用源映射
### Enabling Source Maps in Webpack

首先，您可以将核心思想包含在配置部分中。如果需要，您可以将其转换为稍后使用插件：
To get started, you can wrap the core idea within a configuration part. You can convert this to use the plugins later if you want:

** ** webpack.parts.js
**webpack.parts.js**

```javascript
exports.generateSourceMaps = ({ type }) => ({
  devtool: type,
});
```

Webpack支持各种源映射类型。这些因质量和构建速度而异。现在，您启用`source-map`进行生产，让webpack使用默认开发。设置如下：
Webpack supports a wide variety of source map types. These vary based on quality and build speed. For now, you enable `source-map` for production and let webpack use the default for development. Set it up as follows:

** ** webpack.config.js
**webpack.config.js**

```javascript
const productionConfig = merge([
leanpub-start-insert
  parts.generateSourceMaps({ type: "source-map" }),
leanpub-end-insert
  ...
]);
```

`source-map`是他们所有人中最慢和最高质量的选择，但这对于生产构建来说很好。
`source-map` is the slowest and highest quality option of them all, but that's fine for a production build.

{pagebreak}

如果你现在构建项目（`npm run build`），你应该在输出中看到源映射：
If you build the project now (`npm run build`), you should see source maps in the output:

```bash
Hash: b59445cb2b9ae4cea11b
Version: webpack 4.1.1
Time: 1347ms
Built at: 3/16/2018 4:58:14 PM
       Asset       Size  Chunks             Chunk Names
     main.js  838 bytes       0  [emitted]  main
    main.css   3.49 KiB       0  [emitted]  main
 main.js.map   3.75 KiB       0  [emitted]  main
main.css.map   85 bytes       0  [emitted]  main
  index.html  220 bytes          [emitted]
Entrypoint main = main.js main.css main.js.map main.css.map
...
```

仔细看看那些* .map *文件。这就是生成和源之间的映射发生的地方。在开发期间，它将映射信息写入捆绑包中。
Take a good look at those *.map* files. That's where the mapping between the generated and the source happens. During development, it writes the mapping information in the bundle.

###在浏览器中启用源映射
### Enabling Source Maps in Browsers

要在浏览器中使用源映射，必须根据特定于浏览器的说明显式启用源映射：
To use source maps within a browser, you have to enable source maps explicitly as per browser-specific instructions:

* [Chrome]（https://developer.chrome.com/devtools/docs/javascript-debugging）。有时源地图[不会在Chrome检查器中更新]（https://github.com/webpack/webpack/issues/2478）。目前，临时修复是强制检查员使用* alt-r *重新加载自身。
* [Chrome](https://developer.chrome.com/devtools/docs/javascript-debugging). Sometimes source maps [will not update in Chrome inspector](https://github.com/webpack/webpack/issues/2478). For now, the temporary fix is to force the inspector to reload itself by using *alt-r*.
* [Firefox]（https://developer.mozilla.org/en-US/docs/Tools/Debugger/How_to/Use_a_source_map）
* [Firefox](https://developer.mozilla.org/en-US/docs/Tools/Debugger/How_to/Use_a_source_map)
* [IE Edge]（https://developer.microsoft.com/en-us/microsoft-edge/platform/documentation/f12-devtools-guide/debugger/#source-maps）
* [IE Edge](https://developer.microsoft.com/en-us/microsoft-edge/platform/documentation/f12-devtools-guide/debugger/#source-maps)
* [Safari]（https://support.apple.com/guide/safari/use-the-safari-develop-menu-sfri20948/mac）
* [Safari](https://support.apple.com/guide/safari/use-the-safari-develop-menu-sfri20948/mac)

W>如果你想使用断点（即``debugger;`或通过浏览器设置的语句），基于`eval`的选项在Chrome中不起作用！
W> If you want to use breakpoints (i.e., a `debugger;` statement or ones set through the browser), the `eval`-based options won't work in Chrome!

Webpack支持的##源映射类型
## Source Map Types Supported by Webpack

webpack支持的源映射类型可以分为两类：
Source map types supported by webpack can be split into two categories:

* **内联**源映射将映射数据直接添加到生成的文件中。
* **Inline** source maps add the mapping data directly to the generated files.
* **单独的**源映射将映射数据发送到单独的源映射文件，并使用注释将源链接到它们。隐藏的源映射会故意省略注释。
* **Separate** source maps emit the mapping data to separate source map files and link the source to them using a comment. Hidden source maps omit the comment on purpose.

由于速度快，内联源图是开发的理想选择。鉴于它们使捆绑包很大，单独的源映射是生产的首选解决方案。如果性能开销可以接受，则在开发期间单独的源映射也可以工作。
Thanks to their speed, inline source maps are ideal for development. Given they make the bundles big, separate source maps are the preferred solution for production. Separate source maps work during development as well if the performance overhead is acceptable.

##内联源映射类型
## Inline Source Map Types

Webpack提供多个内联源映射变体。通常`eval`是起点，[webpack问题＃2145]（https://github.com/webpack/webpack/issues/2145#issuecomment-294361203）推荐`cheap-module-eval-source-map`，因为它是在Chrome和Firefox浏览器中可靠地工作时，速度和质量之间的良好折衷。
Webpack provides multiple inline source map variants. Often `eval` is the starting point and [webpack issue #2145](https://github.com/webpack/webpack/issues/2145#issuecomment-294361203) recommends `cheap-module-eval-source-map` as it's a good compromise between speed and quality while working reliably in Chrome and Firefox browsers.

为了更好地了解可用选项，下面列出了它们，同时为每个选项提供了一个小例子。源代码只包含一个`console.log（'Hello world'）`和`webpack.NamedModulesPlugin`用于保持输出更容易理解。实际上，您会看到更多代码来处理映射。
To get a better idea of the available options, they are listed below while providing a small example for each. The source code contains only a single `console.log('Hello world')` and `webpack.NamedModulesPlugin` is used to keep the output easier to understand. In practice, you would see a lot more code to handle the mapping.

###`devtool：“eval”`
### `devtool: "eval"`

`eval`生成代码，其中每个模块都包含在`eval`函数中：
`eval` generates code in which each module is wrapped within an `eval` function:

```javascript
webpackJsonp([1, 2], {
  "./src/index.js": function(module, exports) {
    eval("console.log('Hello world');\n\n//////////////////\n// WEBPACK FOOTER\n// ./src/index.js\n// module id = ./src/index.js\n// module chunks = 1\n\n//# sourceURL=webpack:///./src/index.js?")
  }
}, ["./src/index.js"]);
```

###`devtool：“cheap-eval-source-map”`
### `devtool: "cheap-eval-source-map"`

`cheap-eval-source-map`更进一步，它包含base64编码版本的代码作为数据网址。结果仅包含行数据而丢失列映射。
`cheap-eval-source-map` goes a step further and it includes base64 encoded version of the code as a data url. The result contains only line data while losing column mappings.

```javascript
webpackJsonp([1, 2], {
  "./src/index.js": function(module, exports) {
    eval("console.log('Hello world');//# sourceMappingURL=data:application/json;charset=utf-8;base64,eyJ2ZXJzaW9uIjozLCJmaWxlIjoiLi9hcHAvaW5kZXguanMuanMiLCJzb3VyY2VzIjpbIndlYnBhY2s6Ly8vLi9hcHAvaW5kZXguanM/MGUwNCJdLCJzb3VyY2VzQ29udGVudCI6WyJjb25zb2xlLmxvZygnSGVsbG8gd29ybGQnKTtcblxuXG4vLy8vLy8vLy8vLy8vLy8vLy9cbi8vIFdFQlBBQ0sgRk9PVEVSXG4vLyAuL2FwcC9pbmRleC5qc1xuLy8gbW9kdWxlIGlkID0gLi9hcHAvaW5kZXguanNcbi8vIG1vZHVsZSBjaHVua3MgPSAxIl0sIm1hcHBpbmdzIjoiQUFBQSIsInNvdXJjZVJvb3QiOiIifQ==")
  }
}, ["./src/index.js"]);
```

{pagebreak}

如果解码该base64字符串，则会获得包含映射的输出：
If you decode that base64 string, you get output containing the mapping:

```json
{
  "file": "./src/index.js",
  "mappings": "AAAA",
  "sourceRoot": "",
  "sources": [
    "webpack:///./src/index.js?0e04"
  ],
  "sourcesContent": [
    "console.log('Hello world');\n\n\n//////////////////\n// WEBPACK FOOTER\n// ./src/index.js\n// module id = ./src/index.js\n// module chunks = 1"
  ],
  "version": 3
}
```

###`devtool：“cheap-module-eval-source-map”`
### `devtool: "cheap-module-eval-source-map"`

`cheap-module-eval-source-map`是一个相同的想法，除了更高的质量和更低的性能：
`cheap-module-eval-source-map` is the same idea, except with higher quality and lower performance:

```javascript
webpackJsonp([1, 2], {
  "./src/index.js": function(module, exports) {
    eval("console.log('Hello world');//# sourceMappingURL=data:application/json;charset=utf-8;base64,eyJ2ZXJzaW9uIjozLCJmaWxlIjoiLi9hcHAvaW5kZXguanMuanMiLCJzb3VyY2VzIjpbIndlYnBhY2s6Ly8vYXBwL2luZGV4LmpzPzIwMTgiXSwic291cmNlc0NvbnRlbnQiOlsiY29uc29sZS5sb2coJ0hlbGxvIHdvcmxkJyk7XG5cblxuLy8gV0VCUEFDSyBGT09URVIgLy9cbi8vIGFwcC9pbmRleC5qcyJdLCJtYXBwaW5ncyI6IkFBQUEiLCJzb3VyY2VSb290IjoiIn0=")
  }
}, ["./src/index.js"]);
```

{pagebreak}

再次，解码数据显示更多：
Again, decoding the data reveals more:

```json
{
  "file": "./src/index.js",
  "mappings": "AAAA",
  "sourceRoot": "",
  "sources": [
    "webpack:///src/index.js?2018"
  ],
  "sourcesContent": [
    "console.log('Hello world');\n\n\n// WEBPACK FOOTER //\n// src/index.js"
  ],
  "version": 3
}
```

在这种特殊情况下，选项之间的差异很小。
In this particular case, the difference between the options is minimal.

###`devtool：“eval-source-map”`
### `devtool: "eval-source-map"`

`eval-source-map`是内联选项的最高质量选项。它也是最慢的，因为它发出的数据最多：
`eval-source-map` is the highest quality option of the inline options. It's also the slowest one as it emits the most data:

```javascript
webpackJsonp([1, 2], {
  "./src/index.js": function(module, exports) {
    eval("console.log('Hello world');//# sourceMappingURL=data:application/json;charset=utf-8;base64,eyJ2ZXJzaW9uIjozLCJzb3VyY2VzIjpbIndlYnBhY2s6Ly8vLi9hcHAvaW5kZXguanM/ZGFkYyJdLCJuYW1lcyI6WyJjb25zb2xlIiwibG9nIl0sIm1hcHBpbmdzIjoiQUFBQUEsUUFBUUMsR0FBUixDQUFZLGFBQVoiLCJmaWxlIjoiLi9hcHAvaW5kZXguanMuanMiLCJzb3VyY2VzQ29udGVudCI6WyJjb25zb2xlLmxvZygnSGVsbG8gd29ybGQnKTtcblxuXG4vLyBXRUJQQUNLIEZPT1RFUiAvL1xuLy8gLi9hcHAvaW5kZXguanMiXSwic291cmNlUm9vdCI6IiJ9")
  }
}, ["./src/index.js"]);
```

{pagebreak}

这次可以为浏览器提供更多的映射数据：
This time around there's more mapping data available for the browser:

```json
{
  "file": "./src/index.js",
  "mappings": "AAAAA,QAAQC,GAAR,CAAY,aAAZ",
  "names": [
    "console",
    "log"
  ],
  "sourceRoot": "",
  "sources": [
    "webpack:///./src/index.js?dadc"
  ],
  "sourcesContent": [
    "console.log('Hello world');\n\n\n// WEBPACK FOOTER //\n// ./src/index.js"
  ],
  "version": 3
}
```

##单独的源映射类型
## Separate Source Map Types

Webpack还可以生成生产使用友好的源映射。它们最终以单独的文件结尾，以。.map`扩展名结尾，仅在需要时由浏览器加载。这样您的用户就可以获得良好的性能，同时您可以更轻松地调试应用程序。
Webpack can also generate production usage friendly source maps. These end up in separate files ending with `.map` extension and are loaded by the browser only when required. This way your users get good performance while it's easier for you to debug the application.

`source-map`是一个合理的默认值。即使以这种方式生成源地图需要更长的时间，您也可以获得最佳质量。如果您不关心生产源地图，则可以跳过该设置并获得更好的性能。
`source-map` is a reasonable default here. Even though it takes longer to generate the source maps this way, you get the best quality. If you don't care about production source maps, you can skip the setting there and get better performance in return.

{pagebreak}

###`devtool：“cheap-source-map”`
### `devtool: "cheap-source-map"`

`cheap-source-map`类似于上面的便宜选项。结果将错过列映射。此外，不会使用来自加载器的源映射，例如* css-loader *。
`cheap-source-map` is similar to the cheap options above. The result is going to miss column mappings. Also, source maps from loaders, such as *css-loader*, are not going to be used.

在这种情况下，检查`.map`文件会显示以下输出：
Examining the `.map` file reveals the following output in this case:

```json
{
  "file": "main.9aff3b1eced1f089ef18.js",
  "mappings": "AAAA",
  "sourceRoot": "",
  "sources": [
    "webpack:///main.9aff3b1eced1f089ef18.js"
  ],
  "sourcesContent": [
    "webpackJsonp([1,2],{\"./src/index.js\":function(o,n){console.log(\"Hello world\")}},[\"./src/index.js\"]);\n\n\n// WEBPACK FOOTER //\n// main.9aff3b1eced1f089ef18.js"
  ],
  "version": 3
}
```

源在其末尾包含`//＃sourceMappingURL = main.9a ... 18.js.map`类型的注释以映射到此文件。
The source contains `//# sourceMappingURL=main.9a...18.js.map` kind of comment at its end to map to this file.

{pagebreak}

###`devtool：“cheap-module-source-map”`
### `devtool: "cheap-module-source-map"`

`cheap-module-source-map`与之前的相同，只是来自加载器的源映射被简化为每行一个映射。在这种情况下，它会产生以下输出：
`cheap-module-source-map` is the same as previous except source maps from loaders are simplified to a single mapping per line. It yields the following output in this case:

```json
{
  "file": "main.9aff3b1eced1f089ef18.js",
  "mappings": "AAAA",
  "sourceRoot": "",
  "sources": [
    "webpack:///main.9aff3b1eced1f089ef18.js"
  ],
  "version": 3
}
```

W>`cheap-module-source-map`是[目前在使用缩小时被破坏]（https://github.com/webpack/webpack/issues/4176），这是避免此选项的一个很好的理由。
W> `cheap-module-source-map` is [currently broken if minification is used](https://github.com/webpack/webpack/issues/4176) and this is an excellent reason to avoid the option for now.

###`devtool：“hidden-source-map”`
### `devtool: "hidden-source-map"`

`hidden-source-map`与`source-map`相同，只是它不会将对源映射的引用写入源文件。如果您不希望在希望使用正确的堆栈跟踪时直接将源映射公开给开发工具，这很方便。
`hidden-source-map` is the same as `source-map` except it doesn't write references to the source maps to the source files. If you don't want to expose source maps to development tools directly while you wish proper stack traces, this is handy.

###`devtool：“nosources-source-map”`
### `devtool: "nosources-source-map"`

`nosources-source-map`创建一个没有`sourcesContent`的源映射。不过，你仍然可以获得堆栈跟踪。如果您不希望将源代码公开给客户端，则该选项很有用。
`nosources-source-map` creates a source map without `sourcesContent` in it. You still get stack traces, though. The option is useful if you don't want to expose your source code to the client.

T> [官方文档]（https://webpack.js.org/configuration/devtool/#devtool）包含有关`devtool`选项的更多信息。
T> [The official documentation](https://webpack.js.org/configuration/devtool/#devtool) contains more information about `devtool` options.

{pagebreak}

###`devtool：“source-map”`
### `devtool: "source-map"`

`source-map`提供最好的质量和完整的结果，但它也是最慢的选择。输出反映了这一点：
`source-map` provides the best quality with the complete result, but it's also the slowest option. The output reflects this:

```json
{
  "file": "main.9aff3b1eced1f089ef18.js",
  "mappings": "AAAAA,cAAc,EAAE,IAEVC,iBACA,SAAUC,EAAQC,GCHxBC,QAAQC,IAAI,kBDST",
  "names": [
    "webpackJsonp",
    "./src/index.js",
    "module",
    "exports",
    "console",
    "log"
  ],
  "sourceRoot": "",
  "sources": [
    "webpack:///main.9aff3b1eced1f089ef18.js",
    "webpack:///./src/index.js"
  ],
  "sourcesContent": [
    "webpackJsonp([1,2],{\n\n/***/ \"./src/index.js\":\n/***/ (function(module, exports) {\n\nconsole.log('Hello world');\n\n/***/ })\n\n},[\"./src/index.js\"]);\n\n\n// WEBPACK FOOTER //\n// main.9aff3b1eced1f089ef18.js",
    "console.log('Hello world');\n\n\n// WEBPACK FOOTER //\n// ./src/index.js"
  ],
  "version": 3
}
```

{pagebreak}

##其他源地图选项
## Other Source Map Options

还有一些其他选项会影响源地图生成：
There are a couple of other options that affect source map generation:

```javascript
{
  output: {
    // Modify the name of the generated source map file.
    // You can use [file], [id], and [hash] replacements here.
    // The default option is enough for most use cases.
    sourceMapFilename: '[file].map', // Default

    // This is the source map filename template. It's default
    // format depends on the devtool option used. You don't
    // need to modify this often.
    devtoolModuleFilenameTemplate:
      'webpack:///[resource-path]?[loaders]'
  },
}
```

T> [官方文档]（https://webpack.js.org/configuration/output/#output-sourcemapfilename）深入研究`output`细节。
T> The [official documentation](https://webpack.js.org/configuration/output/#output-sourcemapfilename) digs into `output` specifics.

W>如果您正在使用`UglifyJsPlugin`并且仍然想要源映射，则需要为插件启用`sourceMap：true`。否则，结果不是您所期望的，因为UglifyJS将执行代码的进一步转换，从而打破映射。其他插件和加载器执行更改时也必须这样做。 * css-loader *和相关的加载器就是一个很好的例子。
W> If you are using `UglifyJsPlugin` and still want source maps, you need to enable `sourceMap: true` for the plugin. Otherwise, the result isn't what you expect because UglifyJS will perform a further transformation of the code, breaking the mapping. The same has to be done with other plugins and loaders performing changes. *css-loader* and related loaders are a good example.

##`SourceMapDevToolPlugin`和`EvalSourceMapDevToolPlugin`
## `SourceMapDevToolPlugin` and `EvalSourceMapDevToolPlugin`

如果您想要更多地控制源地图生成，可以使用[SourceMapDevToolPlugin]（https://webpack.js.org/plugins/source-map-dev-tool-plugin/）或`EvalSourceMapDevToolPlugin`代替。后者是一种更有限的替代方案，正如其名称所述，它可以方便地生成基于“eval”的源地图。
If you want more control over source map generation, it's possible to use the [SourceMapDevToolPlugin](https://webpack.js.org/plugins/source-map-dev-tool-plugin/) or `EvalSourceMapDevToolPlugin` instead. The latter is a more limited alternative, and as stated by its name, it's handy for generating `eval` based source maps.

这两个插件可以允许更精细地控制要为其生成源映射的代码部分，同时还使用`SourceMapDevToolPlugin`严格控制结果。使用任一插件都可以完全跳过`devtool`选项。
Both plugins can allow more granular control over which portions of the code you want to generate source maps for, while also having strict control over the result with `SourceMapDevToolPlugin`. Using either plugin allows you to skip the `devtool` option altogether.

鉴于webpack默认情况下只为源映射匹配`.js`和`.css`文件，你可以使用`SourceMapDevToolPlugin`来克服这个问题。这可以通过传递`test`模式来实现，如`/ \。（js | jsx | css）（$ | \？）/ i`。
Given webpack matches only `.js` and `.css` files by default for source maps, you can use `SourceMapDevToolPlugin` to overcome this issue. This can be achieved by passing a `test` pattern like `/\.(js|jsx|css)($|\?)/i`.

如上所述，`EvalSourceMapDevToolPlugin`仅接受`module`和`lineToLine`选项。因此，它可以被视为`devtool：“eval”的别名，同时允许更多的灵活性。
`EvalSourceMapDevToolPlugin` accepts only `module` and `lineToLine` options as described above. Therefore it can be considered as an alias to `devtool: "eval"` while allowing a notch more flexibility.

##更改源映射前缀
## Changing Source Map Prefix

您可以使用** pragma **字符为源映射选项添加前缀，该字符将注入到源映射引用中。默认情况下，Webpack使用现代浏览器支持的`＃`，因此您无需进行设置。
You can prefix a source map option with a **pragma** character that gets injected into the source map reference. Webpack uses `#` by default that is supported by modern browsers, so you don't have to set it.

要覆盖它，您必须在其前面添加源映射选项（例如，`@ source-map`）。在更改之后，假设使用了单独的源映射类型，您应该在JavaScript文件中看到`// @`对源映射的引用`//＃`。
To override this, you have to prefix your source map option with it (e.g., `@source-map`). After the change, you should see `//@` kind of reference to the source map over `//#` in your JavaScript files assuming a separate source map type was used.

##使用依赖关系源映射
## Using Dependency Source Maps

假设您使用的是在其发行版中使用内联源映射的包，您可以使用[source-map-loader]（https://www.npmjs.com/package/source-map-loader）使webpack知道它们。如果不对包进行设置，您将获得最小化的调试输出。通常，您可以跳过此步骤，因为这是一个特殊情况。
Assuming you are using a package that uses inline source maps in its distribution, you can use [source-map-loader](https://www.npmjs.com/package/source-map-loader) to make webpack aware of them. Without setting it up against the package, you get minified debug output. Often you can skip this step as it's a special case.

## Source Maps for Styling
## Source Maps for Styling

如果要为样式文件启用源映射，可以通过启用`sourceMap`选项来实现。同样的想法适用于样式加载器，例如* css-loader *，* sass-loader *和* less-loader *。
If you want to enable source maps for styling files, you can achieve this by enabling the `sourceMap` option. The same idea works with style loaders such as *css-loader*, *sass-loader*, and *less-loader*.

当你在导入中使用相对路径时，* css-loader * [已知有问题]（https://github.com/webpack-contrib/css-loader/issues/232）。要解决此问题，您应该设置`output.publicPath`来解析服务器URL。
The *css-loader* is [known to have issues](https://github.com/webpack-contrib/css-loader/issues/232) when you are using relative paths in imports. To overcome this problem, you should set `output.publicPath` to resolve the server url.

{pagebreak}

##结论
## Conclusion

源地图在开发过程中很方便。它们提供了更好的调试应用程序的方法，因为您仍然可以检查生成的原始代码。它们甚至可以用于生产用途，并且允许您在提供客户端友好版本的应用程序时调试问题。
Source maps can be convenient during development. They provide better means to debug applications as you can still examine the original code over a generated one. They can be valuable even for production usage and allow you to debug issues while serving a client-friendly version of your application.

回顾一下：
To recap:

* **源地图**在开发和生产过程中都很有用。它们提供有关正在发生的事情的更准确信息，并使调试可能出现的问题变得更快。
* **Source maps** can be helpful both during development and production. They provide more accurate information about what's going on and make it faster to debug possible problems.
* Webpack支持各种源映射变体。它们可以根据生成位置拆分为内联和单独的源映射。由于速度的原因，内联源映射在开发过程中很方便。单独的源映射适用于生产，然后加载它们变为可选。
* Webpack supports a large variety of source map variants. They can be split into inline and separate source maps based on where they are generated. Inline source maps are handy during development due to their speed. Separate source maps work for production as then loading them becomes optional.
*`devtool：“source-map”`是最高质量的选择，使其对生产有价值。
* `devtool: "source-map"` is the highest quality option making it valuable for production.
*`cheap-module-eval-source-map`是一个很好的开发起点。
* `cheap-module-eval-source-map` is a good starting point for development.
*如果你想在生产过程中只获得堆栈跟踪，请使用`devtool：“hidden-source-map”`。您可以捕获输出并将其发送到第三方服务供您检查。这样您就可以捕获错误并修复它们。
* If you want to get only stack traces during production, use `devtool: "hidden-source-map"`. You can capture the output and send it to a third party service for you to examine. This way you can capture errors and fix them.
*`SourceMapDevToolPlugin`和`EvalSourceMapDevToolPlugin`提供了比`devtool`快捷方式更多的结果控制。
* `SourceMapDevToolPlugin` and `EvalSourceMapDevToolPlugin` provide more control over the result than the `devtool` shortcut.
如果您的依赖项提供源映射，* source-map-loader *可以派上用场。
* *source-map-loader* can come in handy if your dependencies provide source maps.
*启用样式的源地图需要额外的努力。您必须为正在使用的与样式相关的加载器启用`sourceMap`选项。
* Enabling source maps for styling requires additional effort. You have to enable `sourceMap` option per styling related loader you are using.

在下一章中，您将学习拆分捆绑包并将当前捆绑包分为应用程序和供应商捆绑包。
In the next chapter, you'll learn to split bundles and separate the current one into application and vendor bundles.

