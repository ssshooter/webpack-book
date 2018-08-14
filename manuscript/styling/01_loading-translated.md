# 加载样式

Webpack不能处理开箱即用的样式，你必须使用加载器和插件来允许加载样式文件。在本章中，你将使用项目设置CSS，并通过自动浏览器刷新来了解它的工作原理。当你对CSS webpack进行更改时，不必强制进行完全刷新。相反，它可以在没有一个CSS的情况下修补CSS。
Webpack doesn't handle styling out of the box, and you will have to use loaders and plugins to allow loading style files. In this chapter, you will set up CSS with the project and see how it works out with automatic browser refreshing. When you make a change to the CSS webpack doesn't have to force a full refresh. Instead, it can patch the CSS without one.

## 加载 CSS

加载 CSS 需要使用 [css-loader](https://www.npmjs.com/package/css-loader) 和 [style-loader](https://www.npmjs.com/package/style-loader)。*css-loader* 在匹配的文件遍历 `@import` 和 `url()`，并将它们视为常规的ES2015 `import`。如果 `@import` 指向外部资源（外链），*css-loader* 会跳过它，因为 webpack 只会进一步处理内部资源。

*style-loader* 通过 `style` 标签注入样式，并且这个行为是可定制的。它还实现了 *Hot Module Replacement* 接口，提供了愉悦的开发体验。

匹配的文件可以通过[file-loader]（https://www.npmjs.com/package/file-loader）或[url-loader]（https://www.npmjs.com/package/）等加载器进行处理。 url-loader），这些可能性在本书的* Loading Assets *部分中讨论。
The matched files can be processed through loaders like [file-loader](https://www.npmjs.com/package/file-loader) or [url-loader](https://www.npmjs.com/package/url-loader), and these possibilities are discussed in the *Loading Assets* part of the book.

由于内联 CSS 不适合生产环境使用，借助 `MiniCssExtractPlugin` 生成独立的 CSS 文件是有必要的。你将在下一章中完成此操作。

{pagebreak}

安装 css-loader 和 style-loader：

```bash
npm install css-loader style-loader --save-dev
```

在 **webpack.parts.js** 文件的末尾追加定义一个加载 CSS 的函数：

**webpack.parts.js**

```javascript
exports.loadCSS = ({ include, exclude } = {}) => ({
  module: {
    rules: [
      {
        test: /\.css$/,
        include,
        exclude,

        use: ["style-loader", "css-loader"],
      },
    ],
  },
});
```

并将这个代码片段引入到主配置文件：

**webpack.config.js**

```javascript
const commonConfig = merge([
  ...
leanpub-start-insert
  parts.loadCSS(),
leanpub-end-insert
]);
```

这段代码的作用是对 `.css` 文件调用给定的 loader。`test` 匹配的是 JavaScript 风格的正则表达式。

loader 是应用于源文件的转换，并返回新的资源，并且可以像 Unix 中的 pipe 一样链接在一起。他们从右到左进行处理。这意味着 `loaders: ["style-loader", "css-loader"]` 可以看作 `styleLoader(cssLoader(input))`。

T> 如果要禁用 *css-loader* `url` 解析，可以设置 `url：false`。同理，禁用 `@import` 解析，设置 `import：false`。

T> 如果你不需要 HMR 功能和 source maps，可以使用 [micro-style-loader](https://www.npmjs.com/package/micro-style-loader) 代替 *style-loader*。

## 新建一个 CSS

我们还没有 CSS 文件呢，赶紧写一个：

**src/main.css**

```css
body {
  background: cornsilk;
}
```

此外，你需要让 webpack 知道它的存在。如果它没被引用，webpack 是不知道有这个文件的：

**src/index.js**

```javascript
leanpub-start-insert
import "./main.css";
leanpub-end-insert
...
```

执行 `npm start`，如果你使用默认端口，打开 `http://localhost:8080`，在 *main.css* 将背景颜色更改为`lime`（`background：lime`）。

这段代码先放下，下一章继续。在此之前，你将了解与样式相关的技术。

![Hello cornsilk world](images/hello_02.png)

T> *CSS Modules* 附录讨论了一种允许你默认处理本地文件的方法。它避免了CSS的范围问题。
T> The *CSS Modules* appendix discusses an approach that allows you to treat local to files by default. It avoids the scoping problem of CSS.

## 加载 Less

![Less](images/less.png)

[Less]（http://lesscss.org/）是一个包含功能的CSS处理器。使用Less不需要花费很多精力通过webpack，因为[less-loader]（https://www.npmjs.com/package/less-loader）处理繁重的工作。你应该安装[less]（https://www.npmjs.com/package/less），因为它是* less-loader *的对等依赖项。
[Less](http://lesscss.org/) is a CSS processor packed with functionality. Using Less doesn't take a lot of effort through webpack as [less-loader](https://www.npmjs.com/package/less-loader) deals with the heavy lifting. You should install [less](https://www.npmjs.com/package/less) as well given it's a peer dependency of *less-loader*.

最小设置如下：

```javascript
{
  test: /\.less$/,
  use: ["style-loader", "css-loader", "less-loader"],
},
```

加载器支持更少的插件，source maps等。要了解这些工作原理，你应该查看项目本身。
The loader supports Less plugins, source maps, and so on. To understand how those work you should check out the project itself.

## 加载 Sass

![Sass](images/sass.png)

[Sass]（http://sass-lang.com/）是一种广泛使用的CSS预处理器。你应该使用[sass-loader]（https://www.npmjs.com/package/sass-loader）。请记住将[node-sass]（https://www.npmjs.com/package/node-sass）安装到你的项目中，因为它是对等的依赖项。
[Sass](http://sass-lang.com/) is a widely used CSS preprocessor. You should use [sass-loader](https://www.npmjs.com/package/sass-loader) with it. Remember to install [node-sass](https://www.npmjs.com/package/node-sass) to your project as it's a peer dependency.

最小设置如下：

```javascript
{
  test: /\.scss$/,
  use: ["style-loader", "css-loader", "sass-loader"],
},
```

T>如果你想要更高的性能，特别是在开发过程中，请查看[fast-sass-loader]（https://www.npmjs.com/package/fast-sass-loader）。
T> If you want more performance, especially during development, check out [fast-sass-loader](https://www.npmjs.com/package/fast-sass-loader).

## 加载 Stylus 和 Yeticss

![Stylus](images/stylus.png)

[Stylus]（http://stylus-lang.com/）是CSS处理器的另一个例子。它通过[stylus-loader]（https://www.npmjs.com/package/stylus-loader）很好地工作。 [yeticss]（https://www.npmjs.com/package/yeticss）是一个适用于它的模式库。
[Stylus](http://stylus-lang.com/) is yet another example of a CSS processor. It works well through [stylus-loader](https://www.npmjs.com/package/stylus-loader). [yeticss](https://www.npmjs.com/package/yeticss) is a pattern library that works well with it.

{pagebreak}

请考虑以下配置：
Consider the following configuration:

```javascript
{
  ...
  module: {
    rules: [
      {
        test: /\.styl$/,
        use: [
          "style-loader",
          "css-loader",
          {
            loader: "stylus-loader",
            options: {
              use: [require("yeticss")],
            },
          },
        ],
      },
    ],
  },
},
```

要使用 yeticss，你必须应用中的其中一个 *.styl* 文件引入它：

```javascript
@import "yeticss"
//or
@import "yeticss/components/type"
```

## PostCSS

![PostCSS](images/postcss.png)

[PostCSS](http://postcss.org/) 可以通过 JavaScript 插件转换 CSS。你甚至可以找到提供 Sass-like 特性的插件。其实 PostCSS 就相当于样式界的 Babel。可以借助 [postcss-loader](https://www.npmjs.com/package/postcss-loader) 在 webpack 中使用 PostCSS。

下面的示例说明了如何使用 PostCSS 设置 autoprefix。它还设置了 [precss](https://www.npmjs.com/package/precss)，这是一个PostCSS 插件，允许你在 CSS 中使用类似 Sass 的标记。你可以将此技术与其他 loader 混合以在那里启用 autoprefix。
The example below illustrates how to set up autoprefixing using PostCSS. It also sets up [precss](https://www.npmjs.com/package/precss), a PostCSS plugin that allows you to use Sass-like markup in your CSS. You can mix this technique with other loaders to enable autoprefixing there.

```javascript
{
  test: /\.css$/,
  use: [
    "style-loader",
    "css-loader",
    {
      loader: "postcss-loader",
      options: {
        plugins: () => ([
          require("autoprefixer"),
          require("precss"),
        ]),
      },
    },
  ],
},
```

记住引入 [autoprefixer](https://www.npmjs.com/package/autoprefixer) 和 [precss](https://www.npmjs.com/package/precss) 以正常工作。该技术将在 *Autoprefixing* 章节中详细讨论。

T> PostCSS 支持基于 *postcss.config.js* 的配置。它底层依赖于 [cosmiconfig](https://www.npmjs.com/package/cosmiconfig)。

### cssnext

[cssnext](http://cssnext.io/) 是一个 PostCSS 插件，允许在某些限制条件下体验未来的 CSS 特性。你可以通过 [postcss-cssnext](https://www.npmjs.com/package/postcss-cssnext) 使用它：

```javascript
{
  use: {
    loader: "postcss-loader",
    options: {
      plugins: () => [require("postcss-cssnext")()],
    },
  },
},
```

相关可用选项，请参阅[使用说明书](http://cssnext.io/usage/)。

T> cssnext 包含 *autoprefixer*！你不必单独配置 autoprefix。

## 了解查找
## Understanding Lookups

为了充分利用 *css-loader*，你应该了解它如何执行查找。即使 *css-loader* 默认处理相对导入，它会完全无视绝对导入（`url("/static/img/demo.png")`）。如果你依赖此类导入，必须将文件复制到项目中。

[copy-webpack-plugin](https://www.npmjs.com/package/copy-webpack-plugin) 可以用于此目的，但你也可以将文件复制到webpack之外。前一种方法的好处是webpack-dev-server可以选择它。
[copy-webpack-plugin](https://www.npmjs.com/package/copy-webpack-plugin) works for this purpose, but you can also copy the files outside of webpack. The benefit of the former approach is that webpack-dev-server can pick that up.

T> 如果你使用 Sass 或 Less，[resolve-url-loader](https://www.npmjs.com/package/resolve-url-loader) 会更方便。It adds support for relative imports to the environments.

### 处理 autoprefix 导入
### Processing autoprefix Imports

如果要以特定方式处理*css-loader*导入，则应将`importLoaders`选项设置为一个数字，该数字告诉加载器在找到的导入之前应该对*css-loader*执行多少个加载器。如果你通过`@ import`语句从CSS导入其他CSS文件，并希望通过特定的加载器处理导入，则此技术至关重要。
If you want to process *css-loader* imports in a specific way, you should set up `importLoaders` option to a number that tells the loader how many loaders before the *css-loader* should be executed against the imports found. If you import other CSS files from your CSS through the `@import` statement and want to process the imports through specific loaders, this technique is essential.

{pagebreak}

请考虑从CSS文件导入以下内容：
Consider the following import from a CSS file:

```css
@import "./variables.sass";
```

要处理Sass文件，你必须编写配置：
To process the Sass file, you would have to write configuration:

```javascript
{
  test: /\.css$/,
  use: [
    "style-loader",
    {
      loader: "css-loader",
      options: {
        importLoaders: 1,
      },
    },
    "sass-loader",
  ],
},
```

如果你向链中添加了更多的加载器，例如* postcss-loader *，则必须相应地调整`importLoaders`选项。
If you added more loaders, such as *postcss-loader*, to the chain, you would have to adjust the `importLoaders` option accordingly.

### 从 *node_modules* 目录加载

你可以直接从 node_modules 目录加载文件。加载 Bootstrap 的例子如下：

```less
@import "~bootstrap/less/bootstrap";
```

波形符（`~`）告诉 webpack 默认情况下它不是相对导入。如果包含 `~`，它会对 `node_modules`（默认设置）执行查找，当然也可以通过 [resolve.modules](https://webpack.js.org/configuration/resolve/#resolve-modules) 字段进行配置。

W> 如果你正在使用 *postcss-loader*，你不使用 `~`，如 [postcss-loader issue](https://github.com/postcss/postcss-loader/issues/166) 中所述。*postcss-loader* 可以在没有 `~` 的情况下解析导入。

## 启用 Source Maps

如果要为 CSS 启用 source maps，则应为 *css-loader* 启用 `sourceMap` 选项，并将 `output.publicPath` 设置为指向开发服务器的绝对URL。如果 loader 链中有多个 loader，则必须分别为每个 loader 启用 source maps。*css-loader* [issue 29](https://github.com/webpack/css-loader/issues/29) 进一步讨论了这个问题。

## 将 CSS 转换为字符串

特别是对于 Angular 2，如果你能够以字符串格式获得 CSS，可以将其推送到组件，这将非常方便。[css-to-string-loader](https://www.npmjs.com/package/css-to-string-loader) 实现了这个功能。

## 使用 Bootstrap

有几种方法可以通过webpack使用[Bootstrap]（https://getbootstrap.com/）。一种选择是指向[npm版本]（https://www.npmjs.com/package/bootstrap）并执行如上所述的加载程序配置。
There are a couple of ways to use [Bootstrap](https://getbootstrap.com/) through webpack. One option is to point to the [npm version](https://www.npmjs.com/package/bootstrap) and perform loader configuration as above.

[Sass版本]（https://www.npmjs.com/package/bootstrap-sass）是另一种选择。在这种情况下，你应该将* sass-loader *的`precision`选项设置为至少8.这是[已知问题]（https://www.npmjs.com/package/bootstrap-sass#sass-number-精度）在* bootstrap-sass *解释。
The [Sass version](https://www.npmjs.com/package/bootstrap-sass) is another option. In this case, you should set `precision` option of *sass-loader* to at least 8. This is [a known issue](https://www.npmjs.com/package/bootstrap-sass#sass-number-precision) explained at *bootstrap-sass*.

第三种选择是通过[bootstrap-loader]（https://www.npmjs.com/package/bootstrap-loader）。它做了很多，但允许自定义。
The third option is to go through [bootstrap-loader](https://www.npmjs.com/package/bootstrap-loader). It does a lot more but allows customization.

## 总结

Webpack 可以加载各种格式的样式表。这里介绍的方法默认将样式编写到 JavaScript 包中。

回顾一下：

* *css-loader* 处理样式文件里的 `@import` 和 `url()`。*style-loader* 将其 *css-loader* 的处理结果转换为 JavaScript 并实现webpack 的 *HMR* 接口。
* Webpack 支持通过 loader 将各种格式编译为 CSS。包括 Sass，Less 和 Stylus。
* PostCSS 允许你通过其插件系统向 CSS 注入功能。 cssnext 是 PostCSS 插件的其中一个例子，它实现了 CSS 的未来功能。
* *css-loader* 默认不处理绝对导入。它允许通过 `importLoaders` 选项自定义加载行为。你可以通过在导入前添加波形符（`~`）字符来对 *node_modules* 执行查找。
* 要使用 source maps，你必须对除 *style-loader* 以外每个样式 loader 启用 `sourceMap`。你还应该将 `output.publicPath` 设置为指向开发服务器的绝对 URL。
* 在 webpack 中使用 Bootstrap 需要格外小心。你可以通过通用 loader 或 bootstrap 专用 loader 来获得更多自定义选项。

虽然这里介绍的加载方法足以用于开发环境，但它并不适合于生产环境。你将在下一章中通过将 CSS 与 JavaScript 分离，了解为什么以及如何解决这个问题。

