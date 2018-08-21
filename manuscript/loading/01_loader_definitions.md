# Loader 的定义 

Webpack 提供了多种配置模块 loader 的方法。 Webpack 2 开始通过引入 `use` 字段，简化了 loader 使用。在这里使用绝对路径是一个好主意，因为它们允许你在不影响构建的情况下移动配置。

另一种方法是设置 `context` 字段，因为这会产生类似的效果并影响 entry 和 loader 的路径解析。但是它对输出没有影响，你仍然需要使用绝对路径或 `/`。

即使你设置了 `include` 或 `exclude` 规则，从 **node_modules** 加载的包仍然可以正常工作，因为它们已经被编译为开箱即用的代码。如果它们没这么做，那么你必须应用 **Consuming Packages** 章节中涵盖的技术。

T> `include`/`exclude` 在处理 **node_modules** 问题时非常方便，因为当你将 JavaScript 文件导入项目时，webpack 会默认处理并遍历已安装的包。为了让 webpack 不处理 **node_modules**，你需要使用 `exclude`。其他文件类型不会遇到此问题。

## 剖析 Loader

Webpack 通过 **loader** 支持多种格式的文件。此外，它支持一些开箱即用的 JavaScript 模块规范。文件格式不同，但思路都是一致的，你必须设置一个或多个 loader，并将它们与你的目录结构连接起来。

{pagebreak}

下例中 webpack 通过 Babel 处理 JavaScript：

**webpack.config.js**

```javascript
module.exports = {
  ...
  module: {
    rules: [
      {
        // **Conditions** to match files using RegExp, function.
        test: /\.js$/,

        // **Restrictions**
        // Restrict matching to a directory. This
        // also accepts an array of paths or a function.
        // The same applies to `exclude`.
        include: path.join(__dirname, "app"),
        exclude(path) {
          // You can perform more complicated checks  as well.
          return path.match(/node_modules/);
        },

        // **Actions** to apply loaders to the matched files.
        use: "babel-loader",
      },
    ],
  },
};
```

T> 如果你对 RegExp 的匹配不熟悉，可以使用在线工具，例如 [regex101](https://regex101.com/)，[RegExr](http://regexr.com/) 或 [Regexper](https://regexper.com)。

## Loader 的运算顺序

一定要记住 loader 总是从右到左，从下到上（拆开写的时候）进行运算的。把它看成函数比较容易理解所谓“从右到左运行”。你可以把 `use: ["style-loader", "css-loader"]` 看作 `style(css(input))`。

要查看规则，请看以下示例：

```javascript
{
  test: /\.css$/,
  use: ["style-loader", "css-loader"],
},
```

根据从右到左的规则，可以等效拆分为：

```javascript
{
  test: /\.css$/,
  use: "style-loader",
},
{
  test: /\.css$/,
  use: "css-loader",
},
```

### 强制执行顺序

尽管可以使用上述规则配置，但是也可以强制在常规规则**之前**或**之后**应用特定规则。`enforce` 字段在这里可以派上用场。把他设置为 `pre` or `post` 以在其他 loader 之前或之后进行处理。

Lint 是一个很好的例子，因为 Lint 必须先于任何其他行为。`enforce: "post"` 倒是很少用到，这多是你想对构建结果进行检查时使用的。

{pagebreak}

基本语法如下：

```javascript
{
  // Conditions
  test: /\.js$/,
  enforce: "pre", // "post" too

  // Actions
  use: "eslint-loader",
},
```

如果你可以保证 `test` 中的 loader 顺序无误，那么可以不使用 `enforce`。不过使用 `enforce` 方便你把不同步骤的 loader 分离开来，更容易组织。

## Loader 的传参

可通过 query 把参数传到 loader：

```javascript
{
  // Conditions
  test: /\.js$/,
  include: PATHS.app,

  // Actions
  use: "babel-loader?presets[]=env",
},
```

这种配置风格也适用于 entry 和 import，webpack 会处理他们。在某些个别情况下，这个写法能派上用场，但通常情况下最好使用以下更具可读性的方案。

{pagebreak}

传入对象到 `use`：

```javascript
{
  // Conditions
  test: /\.js$/,
  include: PATHS.app,

  // Actions
  use: {
    loader: "babel-loader",
    options: {
      presets: ["env"],
    },
  },
},
```

如果你想使用多个 loader，你可以将一个对象数组传递给 `use`：

```javascript
{
  test: /\.js$/,
  include: PATHS.app,

  use: [
    {
      loader: "babel-loader",
      options: {
        presets: ["env"],
      },
    },
    // Add more loaders here
  ],
},
```

{pagebreak}

## 使用函数在 `use` 字段添加分支

本书中，你在更高级别上进行环境配置。实现类似结果的另一个选择是在 `use` 处使用分支，因为 webpack 的 loader 定义接受函数作为参数，你可以通过此函数区分环境：

```javascript
{
  test: /\.css$/,

  // `resource` refers to the resource path matched.
  // `resourceQuery` contains possible query passed to it
  // `issuer` tells about match context path
  use: ({ resource, resourceQuery, issuer }) => {
    // You have to return something falsy, object, or a
    // string (i.e., "style-loader") from here.
    //
    // Returning an array fails! Nest rules instead.
    if (env === "development") {
      return {
        use: {
          loader: "css-loader", // css-loader first
          rules: [
            "style-loader", // style-loader after
          ],
        },
      };
    }
  },
},
```

用心感受，这是组合配置的另一种手段。

## 内联式定义

尽管配置级 loader 定义更可取，但可以内联编写 loader 定义：

```javascript
// Process foo.png through url-loader and other
// possible matches.
import "url-loader!./foo.png";

// Override possible higher level match completely
import "!!url-loader!./bar.png";
```

这种方法的问题在你的源代码会与 webpack 耦合。相同的机制还适用于 entry：

```javascript
{
  entry: {
    app: "babel-loader!./app",
  },
},
```

## 匹配文件的备选方法

`test` 结合 `include` 或 `exclude` 是匹配文件的最常用方法。这些字段接受以下数据类型：

* `test`——匹配 RegExp，字符串，函数，对象或数组。
* `include`——同上。
* `exclude`——同上，输出与 `include` 相反。
* `resource: /inline/`——匹配包含查询内容的资源路径。示例：`/path/foo.inline.js`, `/path/bar.png?inline`。
* `issuer: /bar.js/`——匹配从某处请求的资源。示例：如果 `/path/foo.png` 从 `/path/bar.js` 请求，那么 `/path/foo.png` 将匹配。
* `resourcePath: /inline/`——匹配包含查询内容的资源路径（不包括 query）。示例：`/path/foo.inline.png`。
* `resourceQuery: /inline/`——匹配包含查询内容的 query（不包括 query）。示例：`/path/foo.png?inline`。

基于布尔值的字段可用于进一步进行约束：
Boolean based fields can be used to constrain these matchers further:

* `not`——**不**匹配给定条件（参见`test`表示接受的值）。
* `and`——同时匹配一系列条件。
* `or`——与数组中其中一个条件匹配。

## 基于 `resourceQuery` 加载

`oneOf` 字段可以根据资源相关匹配将 webpack 路由到特定的 loader：

```javascript
{
  test: /\.png$/,
  oneOf: [
    {
      resourceQuery: /inline/,
      use: "url-loader",
    },
    {
      resourceQuery: /external/,
      use: "file-loader",
    },
  ],
},
```

如果你需要在文件名中查询，应该使用 `resourcePath` 而不是 `resourceQuery`。

{pagebreak}

## 基于 `issuer` 加载

`issuer` 基于资源的导入位置进行操作。以下示例改编自 [css-loader issue 287](https://github.com/webpack-contrib/css-loader/pull/287#issuecomment-261269199)，**style-loader** 将应用于 JavaScript 导入的 CSS 文件：

```javascript
{
  test: /\.css$/,

  rules: [
    {
      issuer: /\.js$/,
      use: "style-loader",
    },
    {
      use: "css-loader",
    },
  ],
},
```

另一种方法结合了 `issuer` 和 `not`：

```javascript
{
  test: /\.css$/,

  rules: [
    // CSS imported from other modules is added to the DOM
    {
      issuer: { not: /\.css$/ },
      use: "style-loader",
    },
    // Apply css-loader against CSS imports to return CSS
    {
      use: "css-loader",
    },
  ],
}
```

## 了解 loader 行为

通过观察 loader 行为可以更深入地理解它们。 [loader-runner](https://www.npmjs.com/package/loader-runner) 允许你在没有 webpack 的情况下单独运行它们。Webpack 在底层也是使用此软件包，*Extending with Loaders* 章节将会详细介绍它。

[inspect-loader](https://www.npmjs.com/package/inspect-loader) 可以监视 loader 之间传递的内容。将此 loader 添加到你的配置即可检查其中的数据流，而不必在 *node_modules* 中插入 `console.log`。

## 总结

Webpack 提供了多种设置 loader 的方法，但在 webpack 4 中用好 `use` 就足够了。注意 loader 的处理顺序，这是很多常见的问题来源。

回顾一下：

* **Loaders** 决定了 webpack 的模块解析机制匹配到文件时应该作何处理。
* loader 定义包括用于匹配的**条件（conditions）**，以及匹配成功需要进行的**动作（actions）**。
* Webpack 2 引入了`use`字段。它将以前的 `loader` 和 `loaders` 字段结合到了一起。
* Webpack 4 提供了多种匹配和改变 loader 行为的方法。例如，你可以在匹配 loader 后进行 **resource query** 匹配，指引 loader 进行特定操作。

在下一章中，你将学习使用 webpack 加载图片。

