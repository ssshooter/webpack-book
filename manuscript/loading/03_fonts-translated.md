＃加载字体
# Loading Fonts

加载字体与加载图像类似。但它确实带来了独特的挑战。如何知道支持哪种字体格式？如果要为每个浏览器提供一流的支持，最多可以担心四种字体格式。
Loading fonts is similar to loading images. It does come with unique challenges, though. How to know what font formats to support? There can be up to four font formats to worry about if you want to provide first class support to each browser.

通过决定应该接收一流服务的一组浏览器和平台可以解决该问题。其余的可以使用系统字体。
The problem can be solved by deciding a set of browsers and platforms that should receive first class service. The rest can use system fonts.

你可以通过webpack以多种方式解决问题。你仍然可以使用* url-loader *和* file-loader *作为图像。但是，字体`test`模式往往更复杂，你不得不担心与字体文件相关的查找。
You can approach the problem in several ways through webpack. You can still use *url-loader* and *file-loader* as with images. Font `test` patterns tend to be more complicated, though, and you have to worry about font file related lookups.

T> [canifont]（https://www.npmjs.com/package/canifont）可帮助你确定应支持的字体格式。它接受**。browserslistrc **定义，然后根据定义检查每个浏览器的字体支持。
T> [canifont](https://www.npmjs.com/package/canifont) helps you to figure out which font formats you should support. It accepts a **.browserslistrc** definition and then checks font support of each browser based on the definition.

##选择一种格式
## Choosing One Format

如果排除Opera Mini，则所有浏览器都支持* .woff *格式。它的新版本* .woff2 *得到了现代浏览器的广泛支持，可以作为一个很好的选择。
If you exclude Opera Mini, all browsers support the *.woff* format. Its newer version, *.woff2*, is widely supported by modern browsers and can be a good alternative.

{pagebreak}

使用一种格式，你可以使用与图像类似的设置，并在使用限制选项时依赖于* file-loader *和* url-loader *：
Going with one format, you can use a similar setup as for images and rely on both *file-loader* and *url-loader* while using the limit option:

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

一个更精细的方法来实现类似的结果，包括* .woff2 *和其他将最终得到如下代码：
A more elaborate approach to achieve a similar result that includes *.woff2* and others would be to end up with the code as below:

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

##支持多种格式
## Supporting Multiple Formats

如果你想确保网站在最大数量的浏览器上看起来很好，你可以使用* file-loader *而忘记内联。再次，这是一个权衡，因为你得到额外的请求，但也许这是正确的举措。在这里你可能最终得到一个加载器配置：
In case you want to make sure the site looks good on a maximum amount of browsers, you can use *file-loader* and forget about inlining. Again, it's a trade-off as you get extra requests, but perhaps it's the right move. Here you could end up with a loader configuration:

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

编写CSS定义的方式很重要。为了确保你从较新的格式中获益，它们应该成为定义中的第一个。这样浏览器会选择它们。
The way you write your CSS definition matters. To make sure you are getting the benefit from the newer formats, they should become first in the definition. This way the browser picks them up.

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

T> [MDN详细讨论了字体系列规则]（https://developer.mozilla.org/en/docs/Web/CSS/@font-face）。
T> [MDN discusses the font-family rule](https://developer.mozilla.org/en/docs/Web/CSS/@font-face) in detail.

{pagebreak}

##操作* file-loader *输出路径和`publicPath`
## Manipulating *file-loader* Output Path and `publicPath`

如上所述和[webpack issue tracker]（https://github.com/webpack/file-loader/issues/32#issuecomment-250622904）所述，* file-loader *允许整形输出。这样你就可以使用root输出`fonts /`下的字体，`images /`下的图像等等。
As discussed above and in [webpack issue tracker](https://github.com/webpack/file-loader/issues/32#issuecomment-250622904), *file-loader* allows shaping the output. This way you can output your fonts below `fonts/`, images below `images/`, and so on over using the root.

此外，可以操作`publicPath`并覆盖默认的每个加载器定义。以下示例将这些技术结合在一起：
Furthermore, it's possible to manipulate `publicPath` and override the default per loader definition. The following example illustrates these techniques together:

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

##基于SVG生成字体文件
## Generating Font Files Based on SVGs

如果你更喜欢使用基于SVG的字体，可以使用[webfonts-loader]（https://www.npmjs.com/package/webfonts-loader）将它们捆绑为单个字体文件。
If you prefer to use SVG based fonts, they can be bundled as a single font file by using [webfonts-loader](https://www.npmjs.com/package/webfonts-loader).

W>如果已经安装了SVG特定图像设置，请注意SVG图像。如果要以不同方式处理字体SVG，请仔细设置其定义。 * Loader Definitions *章节涵盖了替代方案。
W> Take care with SVG images if you have SVG specific image setup in place already. If you want to process font SVGs differently, set their definitions carefully. The *Loader Definitions* chapter covers alternatives.

##使用Google字体
## Using Google Fonts

[google-fonts-webpack-plugin]（https://www.npmjs.com/package/google-fonts-webpack-plugin）可以将Google字体下载到webpack构建目录或使用CDN连接到它们。
[google-fonts-webpack-plugin](https://www.npmjs.com/package/google-fonts-webpack-plugin) can download Google Fonts to webpack build directory or connect to them using a CDN.

##使用图标字体
## Using Icon Fonts

[iconfont-webpack-plugin]（https://www.npmjs.com/package/iconfont-webpack-plugin）旨在简化基于图标的加载字体。它在CSS文件中内联SVG引用。
[iconfont-webpack-plugin](https://www.npmjs.com/package/iconfont-webpack-plugin) was designed to simplify loading icon based fonts. It inlines SVG references within CSS files.

## 总结


加载字体与加载其他资源类似。你必须考虑要支持的浏览器，并根据该选择加载策略。
Loading fonts is similar to loading other assets. You have to consider the browsers you want to support and choose the loading strategy based on that.

回顾一下：


*加载字体时，应用与图像相同的技术。你可以选择内联小字体，而较大字体则作为单独资源。
* When loading fonts, the same techniques as for images apply. You can choose to inline small fonts while bigger ones are served as separate assets.
*如果你决定仅为现代浏览器提供一流支持，则只能选择一种或两种字体格式，并让旧版浏览器使用系统级字体。
* If you decide to provide first class support to only modern browsers, you can select only a font format or two and let the older browsers to use system level fonts.

在下一章中，你将学习使用Babel和webpack加载JavaScript。 Webpack默认加载JavaScript，但主题还有更多内容，因为你必须考虑要支持的浏览器。
In the next chapter, you'll learn to load JavaScript using Babel and webpack. Webpack loads JavaScript by default, but there's more to the topic as you have to consider what browsers you want to support.

