#Loading Styles
# Loading Styles

Webpack不能处理开箱即用的样式，您必须使用加载器和插件来允许加载样式文件。在本章中，您将使用项目设置CSS，并通过自动浏览器刷新来了解它的工作原理。当您对CSS webpack进行更改时，不必强制进行完全刷新。相反，它可以在没有一个CSS的情况下修补CSS。
Webpack doesn't handle styling out of the box, and you will have to use loaders and plugins to allow loading style files. In this chapter, you will set up CSS with the project and see how it works out with automatic browser refreshing. When you make a change to the CSS webpack doesn't have to force a full refresh. Instead, it can patch the CSS without one.

##加载CSS
## Loading CSS

要加载CSS，您需要使用[css-loader]（https://www.npmjs.com/package/css-loader）和[style-loader]（https://www.npmjs.com/package/style -loader）。 * css-loader *在匹配的文件中经历可能的`@ import`和`url（）`查找，并将它们视为常规的ES2015`import`。如果`@ import`指向外部资源，* css-loader *会跳过它，因为webpack只会进一步处理内部资源。
To load CSS, you need to use [css-loader](https://www.npmjs.com/package/css-loader) and [style-loader](https://www.npmjs.com/package/style-loader). *css-loader* goes through possible `@import` and `url()` lookups within the matched files and treats them as a regular ES2015 `import`. If an `@import` points to an external resource, *css-loader* skips it as only internal resources get processed further by webpack.

* style-loader *通过`style`元素注入样式。它的方式可以定制。它还实现了* Hot Module Replacement *界面，提供了愉快的开发体验。
*style-loader* injects the styling through a `style` element. The way it does this can be customized. It also implements the *Hot Module Replacement* interface providing for a pleasant development experience.

匹配的文件可以通过[file-loader]（https://www.npmjs.com/package/file-loader）或[url-loader]（https://www.npmjs.com/package/）等加载器进行处理。 url-loader），这些可能性在本书的* Loading Assets *部分中讨论。
The matched files can be processed through loaders like [file-loader](https://www.npmjs.com/package/file-loader) or [url-loader](https://www.npmjs.com/package/url-loader), and these possibilities are discussed in the *Loading Assets* part of the book.

由于内联CSS不适合生产使用，因此使用“MiniCssExtractPlugin”生成单独的CSS文件是有意义的。您将在下一章中完成此操作。
Since inlining CSS isn't a good idea for production usage, it makes sense to use `MiniCssExtractPlugin` to generate a separate CSS file. You will do this in the next chapter.

{pagebreak}

要开始，请调用
To get started, invoke

```bash
npm install css-loader style-loader --save-dev
```

现在让我们确保webpack知道它们。在零件定义的末尾添加一个新函数：
Now let's make sure webpack is aware of them. Add a new function at the end of the part definition:

** ** webpack.parts.js
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

您还需要将片段连接到主要配置：
You also need to connect the fragment to the primary configuration:

** ** webpack.config.js
**webpack.config.js**

```javascript
const commonConfig = merge([
  ...
leanpub-start-insert
  parts.loadCSS(),
leanpub-end-insert
]);
```

添加的配置意味着以`.css`结尾的文件应该调用给定的加载器。 `test`匹配JavaScript风格的正则表达式。
The added configuration means that files ending with `.css` should invoke the given loaders. `test` matches against a JavaScript-style regular expression.

加载器是应用于源文件的转换，并返回新的源，并且可以像Unix中的管道一样链接在一起。他们从右到左进行评估。这意味着`loaders：[“style-loader”，“css-loader”]`可以读作`styleLoader（cssLoader（input））`。
Loaders are transformations that are applied to source files, and return the new source and can be chained together like a pipe in Unix. They evaluated from right to left. This means that `loaders: ["style-loader", "css-loader"]` can be read as `styleLoader(cssLoader(input))`.

T>如果要禁用* css-loader *`url`解析集`url：false`。同样的想法适用于`@ import`。要禁用解析导入，可以通过加载程序选项设置`import：false`。
T> If you want to disable *css-loader* `url` parsing set `url: false`. The same idea applies to `@import`. To disable parsing imports you can set `import: false` through the loader options.

T>如果您不需要HMR功能，支持旧版Internet Explorer和源地图，请考虑使用[micro-style-loader]（https://www.npmjs.com/package/micro-style-loader）而不是* style-loader *。
T> In case you don't need HMR capability, support for old Internet Explorer, and source maps, consider using [micro-style-loader](https://www.npmjs.com/package/micro-style-loader) instead of *style-loader*.

##设置初始CSS
## Setting Up the Initial CSS

您仍然缺少CSS：
You are missing the CSS still:

** SRC / **的main.css
**src/main.css**

```css
body {
  background: cornsilk;
}
```

此外，您需要让webpack知道它。没有以某种方式指向它的条目，webpack无法找到该文件：
Also, you need to make webpack aware of it. Without having an entry pointing to it somehow, webpack is not able to find the file:

** SRC / index.js **
**src/index.js**

```javascript
leanpub-start-insert
import "./main.css";
leanpub-end-insert
...
```

执行`npm start`并浏览到`http：// localhost：8080`如果你使用默认端口并打开* main.css *并将背景颜色更改为`lime`（`background：lime`） 。
Execute `npm start` and browse to `http://localhost:8080` if you are using the default port and open up *main.css* and change the background color to something like `lime` (`background: lime`).

你将在下一章继续。不过，在此之前，您将了解与样式相关的技术。
You continue from here in the next chapter. Before that, though, you'll learn about styling-related techniques.

![Hello cornsilk world](images/hello_02.png)

T> * CSS Modules *附录讨论了一种允许您默认处理本地文件的方法。它避免了CSS的范围问题。
T> The *CSS Modules* appendix discusses an approach that allows you to treat local to files by default. It avoids the scoping problem of CSS.

## Loading Less
## Loading Less

![Less](images/less.png)

[Less]（http://lesscss.org/）是一个包含功能的CSS处理器。使用Less不需要花费很多精力通过webpack，因为[less-loader]（https://www.npmjs.com/package/less-loader）处理繁重的工作。您应该安装[less]（https://www.npmjs.com/package/less），因为它是* less-loader *的对等依赖项。
[Less](http://lesscss.org/) is a CSS processor packed with functionality. Using Less doesn't take a lot of effort through webpack as [less-loader](https://www.npmjs.com/package/less-loader) deals with the heavy lifting. You should install [less](https://www.npmjs.com/package/less) as well given it's a peer dependency of *less-loader*.

考虑以下最小设置：
Consider the following minimal setup:

```javascript
{
  test: /\.less$/,
  use: ["style-loader", "css-loader", "less-loader"],
},
```

加载器支持更少的插件，源映射等。要了解这些工作原理，您应该查看项目本身。
The loader supports Less plugins, source maps, and so on. To understand how those work you should check out the project itself.

##加载Sass
## Loading Sass

![Sass](images/sass.png)

[Sass]（http://sass-lang.com/）是一种广泛使用的CSS预处理器。您应该使用[sass-loader]（https://www.npmjs.com/package/sass-loader）。请记住将[node-sass]（https://www.npmjs.com/package/node-sass）安装到您的项目中，因为它是对等的依赖项。
[Sass](http://sass-lang.com/) is a widely used CSS preprocessor. You should use [sass-loader](https://www.npmjs.com/package/sass-loader) with it. Remember to install [node-sass](https://www.npmjs.com/package/node-sass) to your project as it's a peer dependency.

Webpack不需要太多配置：
Webpack doesn't need much configuration:

```javascript
{
  test: /\.scss$/,
  use: ["style-loader", "css-loader", "sass-loader"],
},
```

T>如果你想要更高的性能，特别是在开发过程中，请查看[fast-sass-loader]（https://www.npmjs.com/package/fast-sass-loader）。
T> If you want more performance, especially during development, check out [fast-sass-loader](https://www.npmjs.com/package/fast-sass-loader).

##加载Stylus和Yeticss
## Loading Stylus and Yeticss

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

要开始在Stylus中使用nowicss，您必须将其导入到应用程序的* .styl *文件之一：
To start using yeticss with Stylus, you must import it to one of your app's *.styl* files:

```javascript
@import "yeticss"
//or
@import "yeticss/components/type"
```

## PostCSS
## PostCSS

![PostCSS](images/postcss.png)

[PostCSS]（http://postcss.org/）允许您通过JavaScript插件执行CSS转换。您甚至可以找到为您提供类似Sass功能的插件。 PostCSS相当于Babel的样式。 [postcss-loader]（https://www.npmjs.com/package/postcss-loader）允许将其与webpack一起使用。
[PostCSS](http://postcss.org/) allows you to perform transformations over CSS through JavaScript plugins. You can even find plugins that provide you Sass-like features. PostCSS is the equivalent of Babel for styling. [postcss-loader](https://www.npmjs.com/package/postcss-loader) allows using it with webpack.

下面的示例说明了如何使用PostCSS设置自动修复。它还设置了[precss]（https://www.npmjs.com/package/precss），这是一个PostCSS插件，允许您在CSS中使用类似Sass的标记。您可以将此技术与其他加载器混合以在那里启用自动修复。
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

您必须记住将[autoprefixer]（https://www.npmjs.com/package/autoprefixer）和[precss]（https://www.npmjs.com/package/precss）包含在您的项目中以使其工作。该技术将在* Autoprefixing *章节中详细讨论。
You have to remember to include [autoprefixer](https://www.npmjs.com/package/autoprefixer) and [precss](https://www.npmjs.com/package/precss) to your project for this to work. The technique is discussed in detail in the *Autoprefixing* chapter.

T> PostCSS支持基于* postcss.config.js *的配置。它内部依赖于[cosmiconfig]（https://www.npmjs.com/package/cosmiconfig）用于其他格式。
T> PostCSS supports *postcss.config.js* based configuration. It relies on [cosmiconfig](https://www.npmjs.com/package/cosmiconfig) internally for other formats.

### cssnext
### cssnext

[cssnext]（http://cssnext.io/）是一个PostCSS插件，允许在某些限制条件下体验未来。您可以通过[postcss-cssnext]（https://www.npmjs.com/package/postcss-cssnext）使用它并按如下方式启用它：
[cssnext](http://cssnext.io/) is a PostCSS plugin that allows experiencing the future now with certain restrictions. You can use it through [postcss-cssnext](https://www.npmjs.com/package/postcss-cssnext) and enable it as follows:

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

有关可用选项，请参阅[使用说明书]（http://cssnext.io/usage/）。
See [the usage documentation](http://cssnext.io/usage/) for available options.

T> cssnext包括* autoprefixer *！您不必单独配置自动修复，以便在这种情况下工作。
T> cssnext includes *autoprefixer*! You don't have to configure autoprefixing separately for it to work in this case.

##了解查找
## Understanding Lookups

为了充分利用* css-loader *，您应该了解它如何执行查找。即使* css-loader *默认处理相对导入，它也不会触及绝对导入（`url（“/ static / img / demo.png”）`）。如果您依赖此类导入，则必须将文件复制到项目中。
To get most out of *css-loader*, you should understand how it performs its lookups. Even though *css-loader* handles relative imports by default, it doesn't touch absolute imports (`url("/static/img/demo.png")`). If you rely on this kind of imports, you have to copy the files to your project.

[copy-webpack-plugin]（https://www.npmjs.com/package/copy-webpack-plugin）可以用于此目的，但您也可以将文件复制到webpack之外。前一种方法的好处是webpack-dev-server可以选择它。
[copy-webpack-plugin](https://www.npmjs.com/package/copy-webpack-plugin) works for this purpose, but you can also copy the files outside of webpack. The benefit of the former approach is that webpack-dev-server can pick that up.

如果您使用Sass或更少，T> [resolve-url-loader]（https://www.npmjs.com/package/resolve-url-loader）会派上用场。它增加了对环境相对导入的支持。
T> [resolve-url-loader](https://www.npmjs.com/package/resolve-url-loader) comes in handy if you use Sass or Less. It adds support for relative imports to the environments.

###处理* css-loader *导入
### Processing *css-loader* Imports

如果要以特定方式处理* css-loader *导入，则应将`importLoaders`选项设置为一个数字，该数字告诉加载器在找到的导入之前应该对* css-loader *执行多少个加载器。如果您通过`@ import`语句从CSS导入其他CSS文件，并希望通过特定的加载器处理导入，则此技术至关重要。
If you want to process *css-loader* imports in a specific way, you should set up `importLoaders` option to a number that tells the loader how many loaders before the *css-loader* should be executed against the imports found. If you import other CSS files from your CSS through the `@import` statement and want to process the imports through specific loaders, this technique is essential.

{pagebreak}

请考虑从CSS文件导入以下内容：
Consider the following import from a CSS file:

```css
@import "./variables.sass";
```

要处理Sass文件，您必须编写配置：
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

如果您向链中添加了更多的加载器，例如* postcss-loader *，则必须相应地调整`importLoaders`选项。
If you added more loaders, such as *postcss-loader*, to the chain, you would have to adjust the `importLoaders` option accordingly.

###从* node_modules *目录加载
### Loading from *node_modules* Directory

您可以直接从node_modules目录加载文件。考虑Bootstrap及其用法，例如：
You can load files directly from your node_modules directory. Consider Bootstrap and its usage for example:

```less
@import "~bootstrap/less/bootstrap";
```

波形符（`~`）告诉webpack默认情况下它不是相对导入。如果包含tilde，它会对`node_modules`（默认设置）执行查找，尽管可以通过[resolve.modules]（https://webpack.js.org/configuration/resolve/#resolve-modules）字段进行配置。
The tilde character (`~`) tells webpack that it's not a relative import as by default. If tilde is included, it performs a lookup against `node_modules` (default setting) although this is configurable through the [resolve.modules](https://webpack.js.org/configuration/resolve/#resolve-modules) field.

W>如果你正在使用* postcss-loader *，你可以跳过使用`@`，如[postcss-loader issue tracker]（https://github.com/postcss/postcss-loader/issues/166）中所述。 * postcss-loader *可以在没有波形符号的情况下解析导入。
W> If you are using *postcss-loader*, you can skip using `~` as discussed in [postcss-loader issue tracker](https://github.com/postcss/postcss-loader/issues/166). *postcss-loader* can resolve the imports without a tilde.

##启用源地图
## Enabling Source Maps

如果要为CSS启用源映射，则应为* css-loader *启用`sourceMap`选项，并将`output.publicPath`设置为指向开发服务器的绝对URL。如果链中有多个加载器，则必须分别为每个加载器启用源映射。 * css-loader * [issue 29]（https://github.com/webpack/css-loader/issues/29）进一步讨论了这个问题。
If you want to enable source maps for CSS, you should enable `sourceMap` option for *css-loader* and set `output.publicPath` to an absolute url pointing to your development server. If you have multiple loaders in a chain, you have to enable source maps separately for each. *css-loader* [issue 29](https://github.com/webpack/css-loader/issues/29) discusses this problem further.

##将CSS转换为字符串
## Converting CSS to Strings

特别是对于Angular 2，如果你能够以字符串格式获得CSS，可以将其推送到组件，这将非常方便。 [css-to-string-loader]（https://www.npmjs.com/package/css-to-string-loader）正是如此。
Especially with Angular 2, it can be convenient if you can get CSS in a string format that can be pushed to components. [css-to-string-loader](https://www.npmjs.com/package/css-to-string-loader) achieves exactly this.

##使用Bootstrap
## Using Bootstrap

有几种方法可以通过webpack使用[Bootstrap]（https://getbootstrap.com/）。一种选择是指向[npm版本]（https://www.npmjs.com/package/bootstrap）并执行如上所述的加载程序配置。
There are a couple of ways to use [Bootstrap](https://getbootstrap.com/) through webpack. One option is to point to the [npm version](https://www.npmjs.com/package/bootstrap) and perform loader configuration as above.

[Sass版本]（https://www.npmjs.com/package/bootstrap-sass）是另一种选择。在这种情况下，你应该将* sass-loader *的`precision`选项设置为至少8.这是[已知问题]（https://www.npmjs.com/package/bootstrap-sass#sass-number-精度）在* bootstrap-sass *解释。
The [Sass version](https://www.npmjs.com/package/bootstrap-sass) is another option. In this case, you should set `precision` option of *sass-loader* to at least 8. This is [a known issue](https://www.npmjs.com/package/bootstrap-sass#sass-number-precision) explained at *bootstrap-sass*.

第三种选择是通过[bootstrap-loader]（https://www.npmjs.com/package/bootstrap-loader）。它做了很多，但允许自定义。
The third option is to go through [bootstrap-loader](https://www.npmjs.com/package/bootstrap-loader). It does a lot more but allows customization.

##结论
## Conclusion

Webpack可以加载各种样式格式。这里介绍的方法默认将样式编写到JavaScript包中。
Webpack can load a variety of style formats. The approaches covered here write the styling to JavaScript bundles by default.

回顾一下：
To recap:

* * css-loader *评估样式的`@ import`和`url（）`定义。 * style-loader *将其转换为JavaScript并实现webpack的* Hot Module Replacement *界面。
* *css-loader* evaluates the `@import` and `url()` definitions of your styling. *style-loader* converts it to JavaScript and implements webpack's *Hot Module Replacement* interface.
* Webpack支持通过加载器编译为CSS的各种格式。这些包括Sass，Less和Stylus。
* Webpack supports a large variety of formats compiling to CSS through loaders. These include Sass, Less, and Stylus.
* PostCSS允许您通过其插件系统向CSS注入功能。 cssnext是PostCSS插件集合的一个示例，它实现了CSS的未来功能。
* PostCSS allows you to inject functionality to CSS in through its plugin system. cssnext is an example of a collection of plugins for PostCSS that implements future features of CSS.
* * css-loader *默认不触及绝对导入。它允许通过`importLoaders`选项自定义加载行为。您可以通过在导入前添加波形符（`~`）字符来对* node_modules *执行查找。
* *css-loader* doesn't touch absolute imports by default. It allows customization of loading behavior through the `importLoaders` option. You can perform lookups against *node_modules* by prefixing your imports with a tilde (`~`) character.
*要使用源映射，您必须通过正在使用的每个样式加载器启用`sourceMap`布尔值，但* style-loader *除外。您还应该将`output.publicPath`设置为指向开发服务器的绝对URL。
* To use source maps, you have to enable `sourceMap` boolean through each style loader you are using except for *style-loader*. You should also set `output.publicPath` to an absolute url that points to your development server.
*在webpack中使用Bootstrap需要特别小心。您可以通过通用加载器或引导程序特定的加载程序来获得更多自定义选项。
* Using Bootstrap with webpack requires special care. You can either go through generic loaders or a bootstrap specific loader for more customization options.

虽然这里介绍的加载方法足以用于开发目的，但它并不适合生产。通过将CSS与源分离，您将在下一章中了解为什么以及如何解决这个问题。
Although the loading approach covered here is enough for development purposes, it's not ideal for production. You'll learn why and how to solve this in the next chapter by separating CSS from the source.

