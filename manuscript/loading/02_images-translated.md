＃载入图片
# Loading Images

通过加载大量小资产可以减慢HTTP / 1应用程序，因为每个请求都会产生开销。 HTTP / 2在这方面有所帮助，并且在某种程度上改变了这种情况。直到那时你才会遇到不同的方法。 Webpack允许其中一些。
HTTP/1 application can be made slow by loading a lot of small assets as each request comes with an overhead. HTTP/2 helps in this regard and changes the situation somewhat drastically. Till then you are stuck with different approaches. Webpack allows a few of these.

Webpack可以使用[url-loader]（https://www.npmjs.com/package/url-loader）内联资产。它会在JavaScript包中将你的图像作为base64字符串发出。该过程减少了增加捆绑包大小时所需的请求数。在开发过程中使用* url-loader *就足够了。但是，你想要考虑生产构建的其他替代方案。
Webpack can inline assets by using [url-loader](https://www.npmjs.com/package/url-loader). It emits your images as base64 strings within your JavaScript bundles. The process decreases the number of requests needed while growing the bundle size. It's enough to use *url-loader* during development. You want to consider other alternatives for the production build, though.

Webpack可以控制内联过程，并可以将加载延迟到[file-loader]（https://www.npmjs.com/package/file-loader）。 * file-loader *输出图像文件并返回它们的路径而不是内联。此技术适用于其他资源类型，例如字体，如后面章节中所示。
Webpack gives control over the inlining process and can defer loading to [file-loader](https://www.npmjs.com/package/file-loader). *file-loader* outputs image files and returns paths to them instead of inlining. This technique works with other assets types, such as fonts, as you see in the later chapters.

##设置* url-loader *
## Setting Up *url-loader*

* url-loader *是一个很好的起点，它是开发目的的完美选择，因为你不必关心生成的包的大小。它带有一个* limit *选项，可用于在达到绝对限制后将图像生成推迟到* file-loader *。这样，你可以将小文件内联到JavaScript包中，同时为较大的文件生成单独的文件。
*url-loader* is a good starting point and it's the perfect option for development purposes, as you don't have to care about the size of the resulting bundle. It comes with a *limit* option that can be used to defer image generation to *file-loader* after an absolute limit is reached. This way you can inline small files to your JavaScript bundles while generating separate files for the bigger ones.

如果使用limit选项，则需要在项目中同时安装* url-loader *和* file-loader *。假设你已正确配置样式，webpack将解析样式包含的任何`url（）`语句。你也可以通过JavaScript代码指向图像资源。
If you use the limit option, you need to install both *url-loader* and *file-loader* to your project. Assuming you have configured your styles correctly, webpack resolves any `url()` statements your styling contains. You can point to the image assets through your JavaScript code as well.

如果使用`limit`选项，* url-loader *会将可能的附加选项传递给* file-loader *，从而可以进一步配置其行为。
In case the `limit` option is used, *url-loader* passes possible additional options to *file-loader* making it possible to configure its behavior further.

要在内联25kB以下的文件时加载* .jpg *和* .png *文件，你必须设置一个加载器：
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

T>如果你希望使用另一个加载器而不是* file-loader *作为* limit *，请设置`fallback：“some-loader”`。然后webpack将解析为而不是默认值。
T> If you prefer to use another loader than *file-loader* as the *limit* is reached, set `fallback: "some-loader"`. Then webpack will resolve to that instead of the default.

##设置*文件加载器*
## Setting Up *file-loader*

如果要完全跳过内联，可以直接使用* file-loader *。以下设置自定义生成的文件名。默认情况下，* file-loader *返回文件内容的MD5哈希值和原始扩展名：
If you want to skip inlining altogether, you can use *file-loader* directly. The following setup customizes the resulting filename. By default, *file-loader* returns the MD5 hash of the file's contents with the original extension:

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

T>如果要将图像输出到特定目录下，请将其设置为“name：”。/ images / [hash]。[ext]“`。
T> If you want to output your images below a particular directory, set it up with `name: "./images/[hash].[ext]"`.

W>注意不要同时在图像上同时使用两个装载机！如果* url-loader *`limit`不够，请使用`include`字段进一步控制。
W> Be careful not to apply both loaders on images at the same time! Use the `include` field for further control if *url-loader* `limit` isn't enough.

##将图像集成到项目中
## Integrating Images to the Project

上面的想法可以包含在一个小帮手中，可以合并到书籍项目中。要开始，请安装依赖项：
The ideas above can be wrapped in a small helper that can be incorporated into the book project. To get started, install the dependencies:

```bash
npm install file-loader url-loader --save-dev
```

设置如下功能：
Set up a function as below:

** ** webpack.parts.js
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

要将其附加到配置，请按如下方式进行调整。配置在开发期间默认为* url-loader *，并在生产中使用* url-loader *和* file-loader *来维护较小的包大小。 * url-loader *在设置`limit`时隐式使用* file-loader *，并且必须安装两者才能使设置生效。
To attach it to the configuration, adjust as follows. The configuration defaults to *url-loader* during development and uses both *url-loader* and *file-loader* in production to maintain smaller bundle sizes. *url-loader* uses *file-loader* implicitly when `limit` is set, and both have to be installed for the setup to work.

** ** webpack.config.js
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

要测试设置是否有效，请下载图像或生成图像（`convert -size 100x100 gradient：blue logo.png`）并从项目中引用它：
To test that the setup works, download an image or generate it (`convert -size 100x100 gradient:blue logo.png`) and refer to it from the project:

** SRC / **的main.css
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

行为会根据你设置的“限制”而改变。低于限制，它应该内联图像，而它应该发出一个单独的资产和一个路径。 CSS查找因* css-loader *而起作用。你还可以尝试从JavaScript代码导入图像，看看会发生什么。
The behavior changes depending on the `limit` you set. Below the limit, it should inline the image while above it should emit a separate asset and a path to it. The CSS lookup works because of *css-loader*. You can also try importing the image from JavaScript code and see what happens.

##加载SVG
## Loading SVGs

Webpack允许[几种方式]（https://github.com/webpack/webpack/issues/595）加载SVG。但是，最简单的方法是通过* file-loader *如下：
Webpack allows a [couple ways](https://github.com/webpack/webpack/issues/595) to load SVGs. However, the easiest way is through *file-loader* as follows:

```javascript
{
  test: /\.svg$/,
  use: "file-loader",
},
```

假设你已正确设置样式，可以参考下面的SVG文件。下面的示例SVG路径是相对于CSS文件：
Assuming you have set up your styling correctly, you can refer to your SVG files as below. The example SVG path below is relative to the CSS file:

```css
.icon {
   background-image: url("../assets/icon.svg");
}
```

还要考虑以下加载器：
Consider also the following loaders:

* [raw-loader]（https://www.npmjs.com/package/raw-loader）可以访问原始SVG内容。
* [raw-loader](https://www.npmjs.com/package/raw-loader) gives access to the raw SVG content.
* [svg-inline-loader]（https://www.npmjs.com/package/svg-inline-loader）更进一步，消除了SVG中不必要的标记。
* [svg-inline-loader](https://www.npmjs.com/package/svg-inline-loader) goes a step further and eliminates unnecessary markup from your SVGs.
* [svg-sprite-loader]（https://www.npmjs.com/package/svg-sprite-loader）可以将单独的SVG文件合并为一个精灵，从而可以更有效地加载，因为你可以避免请求开销。它也支持光栅图像（* .jpg *，* .png *）。
* [svg-sprite-loader](https://www.npmjs.com/package/svg-sprite-loader) can merge separate SVG files into a single sprite, making it potentially more efficient to load as you avoid request overhead. It supports raster images (*.jpg*, *.png*) as well.
* [svg-url-loader]（https://www.npmjs.com/package/svg-url-loader）将SVG加载为UTF-8编码数据网址。结果比base64更小，更快解析。
* [svg-url-loader](https://www.npmjs.com/package/svg-url-loader) loads SVGs as UTF-8 encoded data urls. The result is smaller and faster to parse than base64.
* [react-svg-loader]（https://www.npmjs.com/package/react-svg-loader）将SVG作为React组件发出，这意味着你最终可能得到像`<Image width = {50} height =这样的代码{50} />`在导入代码后在代码中呈现SVG。
* [react-svg-loader](https://www.npmjs.com/package/react-svg-loader) emits SVGs as React components meaning you could end up with code like `<Image width={50} height={50}/>` to render a SVG in your code after importing it.

T>你仍然可以使用* url-loader *以及上面的SVG提示。
T> You can still use *url-loader* and the tips above with SVGs too.

##优化图像
## Optimizing Images

如果你想压缩图像，请使用[image-webpack-loader]（https://www.npmjs.com/package/image-webpack-loader），[svgo-loader]（https：//www.npmjs .com / package / svgo-loader）（特定于SVG），或[imagemin-webpack-plugin]（https://www.npmjs.com/package/imagemin-webpack-plugin）。这种类型的加载器应首先应用于数据，因此请记住将其作为`use`列表中的最后一个。
In case you want to compress your images, use [image-webpack-loader](https://www.npmjs.com/package/image-webpack-loader), [svgo-loader](https://www.npmjs.com/package/svgo-loader) (SVG specific), or [imagemin-webpack-plugin](https://www.npmjs.com/package/imagemin-webpack-plugin). This type of loader should be applied first to the data, so remember to place it as the last within `use` listing.

压缩对于生产构建尤其有用，因为它可以减少下载图像资产所需的带宽量，从而加快你的站点或应用程序的速度。
Compression is particularly valuable for production builds as it decreases the amount of bandwidth required to download your image assets and speed up your site or application as a result.

##使用`srcset`
## Utilizing `srcset`

[resize-image-loader]（https://www.npmjs.com/package/resize-image-loader）和[responsive-loader]（https://www.npmjs.com/package/responsive-loader）允许你为现代浏览器生成`srcset`兼容的图像集合。 `srcset`为浏览器提供了更多控制权，可以加载哪些图像以及何时提高性能。
[resize-image-loader](https://www.npmjs.com/package/resize-image-loader) and [responsive-loader](https://www.npmjs.com/package/responsive-loader) allow you to generate `srcset` compatible collections of images for modern browsers. `srcset` gives more control to the browsers over what images to load and when resulting in higher performance.

##动态加载图像
## Loading Images Dynamically

Webpack允许你根据条件动态加载图像。 * Code Splitting *和* Dynamic Loading *章节中涵盖的技术足以实现此目的。这样做可以节省带宽并仅在需要时加载图像或在有时间的情况下预加载图像。
Webpack allows you to load images dynamically based on a condition. The techniques covered in the *Code Splitting* and *Dynamic Loading* chapters are enough for this purpose. Doing this can save bandwidth and load images only when you need them or preload them while you have time.

##加载精灵
## Loading Sprites

** Spriting **技术允许你将多个较小的图像组合成单个图像。它已经被用于游戏来描述动画，它对于Web开发很有价值，同时也避免了请求开销。
**Spriting** technique allows you to combine multiple smaller images into a single image. It has been used for games to describe animations and it's valuable for web development as well as you avoid request overhead.

[webpack-spritesmith]（https://www.npmjs.com/package/webpack-spritesmith）将提供的图像转换为精灵表和Sass / Less / Stylus mixins。你必须设置一个`SpritesmithPlugin`，将其指向目标图像，并设置生成的mixin的名称。之后，你的造型可以拿起它：
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

##使用占位符
## Using Placeholders

[image-trace-loader]（https://www.npmjs.com/package/image-trace-loader）加载图像并将结果公开为`image / svg + xml` URL编码数据。它可以与* file-loader *和* url-loader *一起使用，用于在加载实际图像时显示占位符。
[image-trace-loader](https://www.npmjs.com/package/image-trace-loader) loads images and exposes the results as `image/svg+xml` URL encoded data. It can be used in conjunction with *file-loader* and *url-loader* for showing a placeholder while the actual image is being loaded.

[lqip-loader]（https://www.npmjs.com/package/lqip-loader）实现了类似的想法。它不是跟踪，而是提供模糊图像而不是跟踪图像。
[lqip-loader](https://www.npmjs.com/package/lqip-loader) implements a similar idea. Instead of tracing, it provides a blurred image instead of a traced one.

##获取图像尺寸
## Getting Image Dimensions

有时只获得对图像的引用是不够的。 [image-size-loader]（https://www.npmjs.com/package/image-size-loader）除了对图像本身的引用外，还会发出图像尺寸，类型和大小。
Sometimes getting the only reference to an image isn't enough. [image-size-loader](https://www.npmjs.com/package/image-size-loader) emits image dimensions, type, and size in addition to the reference to the image itself.

##参考图像
## Referencing to Images

假设* css-loader *已经配置，Webpack可以通过`@ import`和`url（）`从样式表中获取图像。你还可以在代码中引用你的图像。在这种情况下，你必须显式导入文件：
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

也可以像* Code Splitting *章节中讨论的那样设置动态导入。这是一个小例子：
It's also possible to set up dynamic imports as discussed in the *Code Splitting* chapter. Here's a small example:

```javascript
const src = require(`./avatars/${avatar}`);`.
```

## Images和* css-loader *源地图陷阱
## Images and *css-loader* Source Map Gotcha

如果你正在使用带有`sourceMap`选项的图像和* css-loader *，那么将`output.publicPath`设置为指向开发服务器的绝对值非常重要。否则，图像无法正常工作。有关详细说明，请参阅[相关的webpack问题]（https://github.com/webpack/style-loader/issues/55）。
If you are using images and *css-loader* with the `sourceMap` option enabled, it's important that you set `output.publicPath` to an absolute value pointing to your development server. Otherwise, images aren't going to work. See [the relevant webpack issue](https://github.com/webpack/style-loader/issues/55) for further explanation.

##总结
## Conclusion

Webpack允许你在需要时在捆绑中内联图像。找出适合你图像的内联限制需要进行实验。你必须在包大小和请求数之间取得平衡。
Webpack allows you to inline images within your bundles when needed. Figuring out proper inlining limits for your images requires experimentation. You have to balance between bundle sizes and the number of requests.

回顾一下：
To recap:

* * url-loader *内嵌JavaScript中的资产。它带有一个`limit`选项，允许你将它上面的资产推迟到* file-loader *。
* *url-loader* inlines the assets within JavaScript. It comes with a `limit` option that allows you to defer assets above it to *file-loader*.
* * file-loader *发出图像资源并将它们返回到代码的路径。它允许散列资产名称。
* *file-loader* emits image assets and returns paths to them to the code. It allows hashing the asset names.
*你可以找到与图像优化相关的加载器和插件，以便你进一步调整其大小。
* You can find image optimization related loaders and plugins that allow you to tune their size further.
*可以从较小的图像中生成**精灵表**，将它们组合成一个请求。
* It's possible to generate **sprite sheets** out of smaller images to combine them into a single request.
* Webpack允许你根据给定条件动态加载图像。
* Webpack allows you to load images dynamically based on a given condition.
*如果你使用源地图，你应该记住将`output.publicPath`设置为要显示的图像的绝对值。
* If you are using source maps, you should remember to set `output.publicPath` to an absolute value for the images to show up.

你将学习在下一章中使用webpack加载字体。
You'll learn to load fonts using webpack in the next chapter.

