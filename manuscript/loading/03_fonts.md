# 加载字体文件

加载字体与加载图片类似。但它确实带来了独特的挑战，如何知道浏览器支持哪种字体格式？如果要为每个浏览器提供最好的支持，最多需要四种字体格式。

你需要确定你的用户主要使用什么浏览器以及操作系统，尽量满足他们的字体显示要求，其余的浏览器可以使用系统字体。

你仍然可以像处理图片一样使用 **url-loader** 和 **file-loader**。但是，字体的 `test` 模式往往更复杂，你需要考虑字体文件的查找问题。

T> [canifont]（https://www.npmjs.com/package/canifont）收录了各大浏览器支持的字体格式。它根据 **.browserslistrc** 配置检查每个浏览器的字体支持情况。

## 选择字体格式

除了 Opera Mini，所有浏览器都支持 *.woff* 格式。另一个选择是它的新版本 *.woff2* 在现代浏览器得到广泛支持。

{pagebreak}

格式较少时你可以使用与图片处理类似的设置，在设置时 limit 选项时同时依赖于 *file-loader* 和 *url-loader*：

```javascript
{
  test: /\.woff$/,
  use: {
    loader: "url-loader",
    options: {
      limit: 50000,
    },
  },
},
```

以下是一种更精确的写法，包括 *.woff2*：

```javascript
{
  // Match woff2 in addition to patterns like .woff?v=1.1.1.
  test: /\.(woff|woff2)(\?v=\d+\.\d+\.\d+)?$/,
  use: {
    loader: "url-loader",
    options: {
      // Limit at 50k. Above that it emits separate files
      limit: 50000,

      // url-loader sets mimetype if it's passed.
      // Without this it derives it from the file extension
      mimetype: "application/font-woff",

      // Output below fonts directory
      name: "./fonts/[name].[ext]",
    }
  },
},
```

{pagebreak}

## 支持多种格式

如果需要确保你的网站在大多数浏览器上正常展示，你可以只使用 **file-loader**。这是也是一个权衡，因为这需要发送更多的请求：

```javascript
{
  test: /\.(ttf|eot|woff|woff2)$/,
  use: {
    loader: "file-loader",
    options: {
      name: "fonts/[name].[ext]",
    },
  },
},
```

在 CSS 中定义字体时，为了确保浏览器优先使用最先进的字体，应该把他们写在最前面。

```css
@font-face {
  font-family: "myfontfamily";
  src: url("./fonts/myfontfile.woff2") format("woff2"),
    url("./fonts/myfontfile.woff") format("woff"),
    url("./fonts/myfontfile.eot") format("embedded-opentype"),
    url("./fonts/myfontfile.ttf") format("truetype");
    /* Add other formats as you see fit */
}
```

T> [MDN 详细讨论了 font-family 规则](https://developer.mozilla.org/en/docs/Web/CSS/@font-face)。

{pagebreak}

## **file-loader** 输出路径与 `publicPath`

[webpack issue](https://github.com/webpack/file-loader/issues/32#issuecomment-250622904) 和本书前面也提到，**file-loader** 可以调整输出位置。这样你就可以在根目录的 `fonts/` 下输出字体，`images/` 下输出图片。

此外，可以使用 `publicPath` 覆盖单个 loader 的默认设置：

```javascript
{
  // Match woff2 and patterns like .woff?v=1.1.1.
  test: /\.woff2?(\?v=\d+\.\d+\.\d+)?$/,
  use: {
    loader: "url-loader",
    options: {
      limit: 50000,
      mimetype: "application/font-woff",
      name: "./fonts/[name].[ext]", // Output below ./fonts
      publicPath: "../", // Take the directory into account
    },
  },
},
```

## 基于 SVG 生成字体文件

如果你更喜欢使用基于 SVG 的字体，可以使用 [webfonts-loader](https://www.npmjs.com/package/webfonts-loader) 将它们打包为单个字体文件。

W> 如果已经对 SVG 配置了 webpack，对用于字体的 SVG 要做其他处理。**Loader Definitions** 章节提到了替代方案。

## 使用 Google Fonts

[google-fonts-webpack-plugin](https://www.npmjs.com/package/google-fonts-webpack-plugin) 可以将 Google 字体下载到 webpack 构建目录或通过 CDN 加载。

## 使用图标字体

[iconfont-webpack-plugin](https://www.npmjs.com/package/iconfont-webpack-plugin) 旨在简化字体图标的加载。它在 CSS 文件中内联为 SVG 引用。

## 总结

加载字体与加载其他资源类似。你必须考虑要支持的浏览器，根据浏览器选择加载策略。

回顾一下：

* 加载字体时可以使用对图片一样的处理，内联体积小的字体，而较大字体则作为单独资源。
* 如果你决定仅为现代浏览器提供一流支持，可以只选择一两种字体格式，让旧版浏览器使用系统字体。

在下一章中，你将学习使用 Babel 和 webpack 加载 JavaScript。虽然 Webpack 默认可以加载 JavaScript，但还有不少问题，因为你必须考虑要支持的浏览器。

