# 载入图片

HTTP/1 应用程序加载大量小文件会变得缓慢，因为每个请求都会产生开销。HTTP/2 在这方面有所改善，在此之前，你可以使用 Webpack 提供的优化方法。

Webpack 可以使用 [url-loader](https://www.npmjs.com/package/url-loader) 内联资源。它会在 JavaScript 包中将你的图片作为 base64 字符串生成。该过程减少了所需的请求数，增加了 bundle 大小。在开发过程中使用 *url-loader* 就足够了。但在生产构建需要其他替代方案。

Webpack 可以控制内联过程，并可以将加载延迟到 [file-loader](https://www.npmjs.com/package/url-loader)。 * file-loader *输出图片文件并返回它们的路径而不是内联。此技术适用于其他资源类型，例如字体，如后面章节中所示。
Webpack gives control over the inlining process and can defer loading to [file-loader](https://www.npmjs.com/package/file-loader). *file-loader* outputs image files and returns paths to them instead of inlining. This technique works with other assets types, such as fonts, as you see in the later chapters.

## 配置 *url-loader*

*url-loader* 是开发环境的完美选择，因为你不必关心 bundle 的大小。它带有一个 *limit* 选项，可用于把超过限制的图片交给 *file-loader*。这样，你可以将小文件内联到 JavaScript 包中，同时为较大的文件生成单独的文件。

如果使用limit选项，则需要在项目中同时安装* url-loader *和* file-loader *。假设你已正确配置样式，webpack将解析样式包含的任何`url（）`语句。你也可以通过JavaScript代码指向图片资源。
If you use the limit option, you need to install both *url-loader* and *file-loader* to your project. Assuming you have configured your styles correctly, webpack resolves any `url()` statements your styling contains. You can point to the image assets through your JavaScript code as well.

如果使用`limit`选项，* url-loader *会将可能的附加选项传递给* file-loader *，从而可以进一步配置其行为。
In case the `limit` option is used, *url-loader* passes possible additional options to *file-loader* making it possible to configure its behavior further.

要在内联25kB以下的文件时加载 *.jpg* 和 *.png* 文件，你必须设置一个加载器：
To load *.jpg* and *.png* files while inlining files below 25kB, you would have to set up a loader:

```javascript
{
  test: /\.(jpg|png)$/,
  use: {
    loader: "url-loader",
    options: {
      limit: 25000,
    },
  },
},
```

T> 如果你希望在超过 *limit* 使用另一个 loader 而不是 *file-loader*，请设置 `fallback: "some-loader"`。

## 配置 *file-loader*

如果要完全跳过内联，直接使用 *file-loader* 就可以了。以下设置自定义生成的文件名。默认情况下，*file-loader* 返回文件内容的 MD5 和原始扩展名：

```javascript
{
  test: /\.(jpg|png)$/,
  use: {
    loader: "file-loader",
    options: {
      name: "[path][name].[hash].[ext]",
    },
  },
},
```

T> 如果要将图片输出到特定目录下，请将其设置为 `name: "./images/[hash].[ext]"`。

W> 注意不要同时在图片上同时使用两个 loader！如果 *url-loader* `limit` 不足以区分，请使用 `include` 字段进一步控制。

## 将图片集成到项目中

安装依赖项：

```bash
npm install file-loader url-loader --save-dev
```

配置功能：

**webpack.parts.js**

```javascript
exports.loadImages = ({ include, exclude, options } = {}) => ({
  module: {
    rules: [
      {
        test: /\.(png|jpg)$/,
        include,
        exclude,
        use: {
          loader: "url-loader",
          options,
        },
      },
    ],
  },
});
```

要将其附加到配置，请按如下方式进行调整。配置在开发期间默认为 *url-loader*，并在生产中使用 *url-loader* 和 *file-loader* 来维护较小的包大小。*url-loader* 在设置`limit`时隐式使用*file-loader*，并且必须安装两者才能使设置生效。
To attach it to the configuration, adjust as follows. The configuration defaults to *url-loader* during development and uses both *url-loader* and *file-loader* in production to maintain smaller bundle sizes. *url-loader* uses *file-loader* implicitly when `limit` is set, and both have to be installed for the setup to work.

**webpack.config.js**

```javascript
const productionConfig = merge([
  ...
leanpub-start-insert
  parts.loadImages({
    options: {
      limit: 15000,
      name: "[name].[ext]",
    },
  }),
leanpub-end-insert
]);

const developmentConfig = merge([
  ...
leanpub-start-insert
  parts.loadImages(),
leanpub-end-insert
]);
```

要测试设置是否有效，请下载图片或生成图片（`convert -size 100x100 gradient：blue logo.png`）并从项目中引用它：
To test that the setup works, download an image or generate it (`convert -size 100x100 gradient:blue logo.png`) and refer to it from the project:

**src/main.css**

```css
body {
  background: cornsilk;
leanpub-start-insert
  background-image: url("./logo.png");
  background-repeat: no-repeat;
  background-position: center;
leanpub-end-insert
}
```

行为会根据你设置的“限制”而改变。低于限制，它应该内联图片，而它应该发出一个单独的资源和一个路径。 CSS查找因* css-loader *而起作用。你还可以尝试从JavaScript代码导入图片，看看会发生什么。
The behavior changes depending on the `limit` you set. Below the limit, it should inline the image while above it should emit a separate asset and a path to it. The CSS lookup works because of *css-loader*. You can also try importing the image from JavaScript code and see what happens.

## 加载 SVG

Webpack 有[几种方式](https://github.com/webpack/webpack/issues/595)加载 SVG。最简单的方法是通过 *file-loader* 加载：

```javascript
{
  test: /\.svg$/,
  use: "file-loader",
},
```

下面的示例 SVG 路径是相对于 CSS 文件：
Assuming you have set up your styling correctly, you can refer to your SVG files as below. The example SVG path below is relative to the CSS file:

```css
.icon {
   background-image: url("../assets/icon.svg");
}
```

还要考虑以下 loader：
Consider also the following loaders:

* [raw-loader](https://www.npmjs.com/package/raw-loader) 可以访问原始 SVG 内容。
* [svg-inline-loader](https://www.npmjs.com/package/svg-inline-loader) 清理了一些 SVG 中不必要的标记.
* [svg-sprite-loader](https://www.npmjs.com/package/svg-sprite-loader) 可以将单独的 SVG 文件合并为雪碧图，这样可以避免请求开销，加载更高效。此 loader 也支持光栅图片（*.jpg*, *.png*）。
* [svg-url-loader](https://www.npmjs.com/package/svg-url-loader) 将 SVG 加载为 UTF-8 编码的数据 url，结果比 base64 更小，解析更快。
* [react-svg-loader](https://www.npmjs.com/package/react-svg-loader) 将 SVG 生成为 React 组件，最终可能得到像 `<Image width={50} height={50}/>` 这样的代码。在导入代码后渲染 SVG。

T> 也可以跟图片一样用 *url-loader* 处理 SVG。

## 压缩图片

如果你想压缩图片，请使用 [image-webpack-loader](https://www.npmjs.com/package/image-webpack-loader)，[svgo-loader](https://www.npmjs.com/package/svgo-loader)（SVG 专用），或[imagemin-webpack-plugin](https://www.npmjs.com/package/imagemin-webpack-plugin)。这些 loader 应该优先处理数据，因此请记住把它们放在 `use` 的最后。

压缩在生产环境尤其有用，可以使你的站点或应用程序高速加载。

## 使用 `srcset`
## Utilizing `srcset`

[resize-image-loader](https://www.npmjs.com/package/resize-image-loader) 和 [responsive-loader](https://www.npmjs.com/package/responsive-loader) 允许你为现代浏览器生成`srcset`兼容的图片集合。 `srcset` 为浏览器提供了更多控制权，可以加载哪些图片以及何时提高性能。
[resize-image-loader](https://www.npmjs.com/package/resize-image-loader) and [responsive-loader](https://www.npmjs.com/package/responsive-loader) allow you to generate `srcset` compatible collections of images for modern browsers. `srcset` gives more control to the browsers over what images to load and when resulting in higher performance.

## 动态加载图片

Webpack 可以根据条件动态加载图片。 *Code Splitting* 和 *Dynamic Loading* 章节中涵盖的技术可以实现此目的。这样做可以节省带宽，仅在需要时或空闲时间加载图片。

## 加载雪碧图

** Spriting **技术允许你将多个较小的图片组合成单个图片。它已经被用于游戏来描述动画，它对于Web开发很有价值，同时也避免了请求开销。
**Spriting** technique allows you to combine multiple smaller images into a single image. It has been used for games to describe animations and it's valuable for web development as well as you avoid request overhead.

[webpack-spritesmith]（https://www.npmjs.com/package/webpack-spritesmith）将提供的图片转换为精灵表和Sass / Less / Stylus mixins。你必须设置一个`SpritesmithPlugin`，将其指向目标图片，并设置生成的mixin的名称。之后，你的造型可以拿起它：
[webpack-spritesmith](https://www.npmjs.com/package/webpack-spritesmith) converts provided images into a sprite sheet and Sass/Less/Stylus mixins. You have to set up a `SpritesmithPlugin`, point it to target images, and set the name of the generated mixin. After that, your styling can pick it up:

```scss
@import "~sprite.sass";

.close-button {
  sprite($close);
}

.open-button {
  sprite($open);
}
```

## 使用占位符

[image-trace-loader]（https://www.npmjs.com/package/image-trace-loader）加载图片并将结果公开为`image / svg + xml` URL编码数据。它可以与* file-loader *和* url-loader *一起使用，用于在加载实际图片时显示占位符。
[image-trace-loader](https://www.npmjs.com/package/image-trace-loader) loads images and exposes the results as `image/svg+xml` URL encoded data. It can be used in conjunction with *file-loader* and *url-loader* for showing a placeholder while the actual image is being loaded.

[lqip-loader]（https://www.npmjs.com/package/lqip-loader）实现了类似的想法。它不是跟踪，而是提供模糊图片而不是跟踪图片。
[lqip-loader](https://www.npmjs.com/package/lqip-loader) implements a similar idea. Instead of tracing, it provides a blurred image instead of a traced one.

## 获取图片尺寸

有时只获得对图片的引用是不够的。 [image-size-loader]（https://www.npmjs.com/package/image-size-loader）除了对图片本身的引用外，还会发出图片尺寸，类型和大小。
Sometimes getting the only reference to an image isn't enough. [image-size-loader](https://www.npmjs.com/package/image-size-loader) emits image dimensions, type, and size in addition to the reference to the image itself.

## 参考图片
## Referencing to Images

假设* css-loader *已经配置，Webpack可以通过`@ import`和`url（）`从样式表中获取图片。你还可以在代码中引用你的图片。在这种情况下，你必须显式导入文件：
Webpack can pick up images from style sheets through `@import` and `url()` assuming *css-loader* has been configured. You can also refer to your images within the code. In this case, you have to import the files explicitly:

```javascript
import src from "./avatar.png";

// Use the image in your code somehow now
const Profile = () => <img src={src} />;
```

如果你使用React，那么你使用[babel-plugin-transform-react-jsx-img-import]（https://www.npmjs.com/package/babel-plugin-transform-react-jsx-img-import ）自动生成`require`。在这种情况下，你最终会得到代码：
If you are using React, then you use [babel-plugin-transform-react-jsx-img-import](https://www.npmjs.com/package/babel-plugin-transform-react-jsx-img-import) to generate the `require` automatically. In that case, you would end up with code:

```javascript
const Profile = () => <img src="avatar.png" />;
```

也可以像 *Code Splitting* 章节中讨论的那样设置动态导入。这是一个小例子：
It's also possible to set up dynamic imports as discussed in the *Code Splitting* chapter. Here's a small example:

```javascript
const src = require(`./avatars/${avatar}`);`.
```

## 图片和 *css-loader* Source Map 的问题
## Images and *css-loader* Source Map Gotcha

如果你正在使用带有`sourceMap`选项的图片和* css-loader *，那么将`output.publicPath`设置为指向开发服务器的绝对值非常重要。否则，图片无法正常工作。有关详细说明，请参阅[相关的webpack问题]（https://github.com/webpack/style-loader/issues/55）。
If you are using images and *css-loader* with the `sourceMap` option enabled, it's important that you set `output.publicPath` to an absolute value pointing to your development server. Otherwise, images aren't going to work. See [the relevant webpack issue](https://github.com/webpack/style-loader/issues/55) for further explanation.

## 总结

Webpack 可以把图片内联到 bundle 中。你需要评估图片的内联大小限制，在 bundle 大小和请求数之间取得平衡。

回顾一下：

* * url-loader *内嵌JavaScript中的资源。它带有一个`limit`选项，允许你将它上面的资源推迟到* file-loader *。
* *url-loader* inlines the assets within JavaScript. It comes with a `limit` option that allows you to defer assets above it to *file-loader*.
* *file-loader* 发出图片资源并将它们返回到代码的路径。它允许散列资源名称。
* *file-loader* emits image assets and returns paths to them to the code. It allows hashing the asset names.
* 你可以找到与图片优化相关的加载器和插件，以便你进一步调整其大小。
* You can find image optimization related loaders and plugins that allow you to tune their size further.
* 可以组合较小的图片生成**雪碧图**。
* It's possible to generate **sprite sheets** out of smaller images to combine them into a single request.
* Webpack 允许你根据给定条件动态加载图片。
* Webpack allows you to load images dynamically based on a given condition.
* 如果你使用 source maps，你应该记住将 `output.publicPath` 设置为要显示的图片的绝对值。
* If you are using source maps, you should remember to set `output.publicPath` to an absolute value for the images to show up.

你将学习在下一章中使用 webpack 加载字体文件。

