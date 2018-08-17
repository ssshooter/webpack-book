# Loader 的定义 

Webpack 提供了多种配置模块 loader 的方法。 Webpack 2 开始通过引入了 `use` 字段，简化了 loader 使用。在这里选择绝对路径是一个好主意，因为它们允许你在不破坏假设的情况下移动配置。
Webpack provides multiple ways to set up module loaders. Webpack 2 simplified the situation by introducing the `use` field. It can be a good idea to prefer absolute paths here as they allow you to move configuration without breaking assumptions.

另一种方法是设置`context`字段，因为这会产生类似的效果并影响入口点和loader的解析方式。但是它对输出没有影响，你仍然需要使用绝对路径或`/`。
The other way is to set `context` field as this gives a similar effect and affects the way entry points and loaders are resolved. It doesn't have an impact on the output, though, and you still need to use an absolute path or `/` there.

假设你设置了`include`或`exclude`规则，从* node_modules *加载的包仍然可以正常工作，因为它们的编译方式使它们能够开箱即用。如果他们不这样做，那么你必须应用 **Consuming Packages** 章节中涵盖的技术。
Assuming you set an `include` or `exclude` rule, packages loaded from *node_modules* still work as the assumption is that they have been compiled in such a way that they work out of the box. If they don't, then you have to apply techniques covered in the *Consuming Packages* chapter.

T> `include`/`exclude` 在使用* node_modules *时非常方便，因为当你将JavaScript文件导入项目时，webpack会默认处理并遍历已安装的包。因此，你需要配置它以避免该行为。其他文件类型不会遇到此问题。
T> `include`/`exclude` is handy with *node_modules* as webpack processes and traverses the installed packages by default when you import JavaScript files to your project. Therefore you need to configure it to avoid that behavior. Other file types don't suffer from this issue.

## 剖析 Loader

Webpack 通过**loaders**支持多种格式。此外，它支持一些开箱即用的JavaScript模块格式。这个想法是一样的。你总是设置一个loader或loader，并将它们与你的目录结构连接起来。
Webpack supports a large variety of formats through *loaders*. Also, it supports a couple of JavaScript module formats out of the box. The idea is the same. You always set up a loader, or loaders, and connect those with your directory structure.

{pagebreak}

考虑下面的例子，webpack通过Babel处理JavaScript：
Consider the example below where webpack processes JavaScript through Babel:

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

如果你仔细地将与'test`相关的其他loader链接到声明，那么可以在没有`enforce`的情况下编写相同的配置。使用`enforce`删除了它的必要性，并允许你将加载程序执行拆分为更容易编写的单独阶段。
It would be possible to write the same configuration without `enforce` if you chained the declaration with other loaders related to the `test` carefully. Using `enforce` removes the necessity for that and allows you to split loader execution into separate stages that are easier to compose.

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

这种配置风格也适用于条目和源代码导入，因为webpack会选择它。在某些个别情况下，该格式会派上用场，但通常你最好使用更具可读性的替代方案。
This style of configuration works in entries and source imports too as webpack picks it up. The format comes in handy in certain individual cases, but often you are better off using more readable alternatives.

{pagebreak}

最好通过 `use`：
It's preferable to go through `use`:

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

如果你想使用多个loader，你可以将一个数组传递给`use`并从那里扩展：
If you wanted to use more than one loader, you could pass an array to `use` and expand from there:

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

本书中，你在更高级别上进行配置。实现类似结果的另一个选择是在 `use` 处分支，因为 webpack 的 loader 定义接受允许根据环境进行分支的函数。思考以下示例：
In the book setup, you compose configuration on a higher level. Another option to achieve similar results would be to branch at `use` as webpack's loader definitions accept functions that allow you to branch depending on the environment. Consider the example below:

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

仔细应用，这种技术允许不同的组成方式。
Carefully applied, this technique allows different means of composition.

## 内联式定义

尽管配置级 loader 定义更可取，但可以内联编写 loader 定义：
Even though configuration level loader definitions are preferable, it's possible to write loader definitions inline:

```javascript
// Process foo.png through url-loader and other
// possible matches.
import "url-loader!./foo.png";

// Override possible higher level match completely
import "!!url-loader!./bar.png";
```

这种方法的问题在于它将你的源与webpack相结合。尽管如此，它仍然是一个很好的形式。由于配置条目通过相同的机制，因此相同的表单也在那里工作：
The problem with this approach is that it couples your source with webpack. Nonetheless, it's still an excellent form to know. Since configuration entries go through the same mechanism, the same forms work there as well:

```javascript
{
  entry: {
    app: "babel-loader!./app",
  },
},
```

## 匹配文件的备选方法

`test`结合`include`或`exclude`来约束匹配是匹配文件的最常用方法。这些接受下面列出的数据类型：
`test` combined with `include` or `exclude` to constrain the match is the most common approach to match files. These accept the data types as listed below:

* `test`——匹配 RegExp，字符串，函数，对象或数组。
* `include`——同上。
* `exclude`——同上，输出与 `include` 相反。
* `resource: /inline/`——匹配包含查询的资源路径。示例：`/path/foo.inline.js`, `/path/bar.png?inline`。
*`issuer：/ bar.js /` - 匹配匹配请求的资源。示例：如果从`/ path / bar.js`请求，`/ path / foo.png`将匹配。
* `issuer: /bar.js/` - Match against a resource requested from the match. Example: `/path/foo.png` would match if it was requested from `/path/bar.js`.
*`resourcePath：/ inline /` - 在没有查询的情况下匹配资源路径。示例：`/ path / foo.inline.png`。
* `resourcePath: /inline/` - Match against a resource path without its query. Example: `/path/foo.inline.png`.
*`resourceQuery：/ inline /` - 根据资源的查询匹配资源。示例：`/ path / foo.png？inline`。
* `resourceQuery: /inline/` - Match against a resource based on its query. Example: `/path/foo.png?inline`.

基于布尔的字段可用于进一步约束这些匹配器：
Boolean based fields can be used to constrain these matchers further:

* `not`  -  ** **不符合条件（参见`test`表示接受的值）。
* `not` - Do **not** match against a condition (see `test` for accepted values).
* `和` - 匹配一系列条件。一切都必须匹配。
* `and` - Match against an array of conditions. All must match.
*`或者 - 与任何必须匹配的数组匹配。
* `or` - Match against an array while any must match.

## 基于 `resourceQuery` 加载
## Loading Based on `resourceQuery`

`oneOf` 字段可以根据资源相关匹配将 webpack 路由到特定的 loader：
`oneOf` field makes it possible to route webpack to a specific loader based on a resource related match:

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

如果要将上下文信息嵌入到文件名中，则规则可以使用`resourcePath`而不是`resourceQuery`。
If you wanted to embed the context information to the filename, the rule could use `resourcePath` over `resourceQuery`.

{pagebreak}

## 基于 `issuer` 加载

`issuer` 基于资源的导入位置进行操作。在下面改编自 [css-loader issue 287](https://github.com/webpack-contrib/css-loader/pull/287#issuecomment-261269199) 的示例中，*style-loader* 在 webpack 捕获时应用 JavaScript 导入的 CSS 文件：
`issuer` can be used to control behavior based on where a resource was imported. In the example below adapted from [css-loader issue 287](https://github.com/webpack-contrib/css-loader/pull/287#issuecomment-261269199), *style-loader* is applied when webpack captures a CSS file from a JavaScript import:

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

另一种方法是混合 `issuer` 和 `not`：
Another approach would be to mix `issuer` and `not`:

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

[inspect-loader](https://www.npmjs.com/package/inspect-loader) 可以监视 loader 之间传递的内容。你可以将此加载程序附加到你的配置并检查其中的流，而不必在* node_modules *中插入`console.log`。
[inspect-loader](https://www.npmjs.com/package/inspect-loader) allows you to inspect what's being passed between loaders. Instead of having to insert `console.log`s within *node_modules*, you can attach this loader to your configuration and inspect the flow there.

## 总结

Webpack 提供了多种设置 loader 的方法，但在 webpack 4 中用好 `use` 就足够了。注意 loader 的处理顺序，这是很多常见的问题来源。

回顾一下：

* **Loaders** 决定了 webpack 的模块解析机制匹配到文件时应该作何处理。
* loader 定义包括用于匹配的**条件（conditions）**，以及匹配成功需要进行的**动作（actions）**。
* Webpack 2 引入了`use`字段。它将以前的 `loader` 和 `loaders` 字段结合到了一起。
* Webpack 4 提供了多种匹配和改变 loader 行为的方法。例如，你可以在匹配 loader 后基于**资源查询进行匹配，并将loader路由到特定操作。
* Webpack 4 provides multiple ways to match and alter loader behavior. You can, for example, match based on a **resource query** after a loader has been matched and route the loader to specific actions.

在下一章中，你将学习使用 webpack 加载图片。

