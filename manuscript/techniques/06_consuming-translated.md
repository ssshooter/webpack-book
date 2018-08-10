＃消费包
# Consuming Packages

有时包没有按照你期望的方式打包，你必须调整webpack解释它们的方式。 Webpack提供了多种方法来实现这一目标。
Sometimes packages have not been packaged the way you expect, and you have to tweak the way webpack interprets them. Webpack provides multiple ways to achieve this.

##`resolve.alias`
## `resolve.alias`

有时包不符合标准规则，它们的* package.json *包含一个错误的`main`字段。它可以完全丢失。 `resolve.alias`是这里使用的字段，如下例所示：
Sometimes packages do not follow the standard rules and their *package.json* contains a faulty `main` field. It can be missing altogether. `resolve.alias` is the field to use here as in the example below:

```javascript
{
  resolve: {
    alias: {
      demo: path.resolve(
        __dirname,
        "node_modules/demo/dist/demo.js"
      ),
    },
  },
},
```

这个想法是，如果webpack解析器在开始时匹配`demo`，它将从目标解析。你可以使用类似`demo $`的模式将进程约束为确切的名称。
The idea is that if webpack resolver matches `demo` in the beginning, it resolves from the target. You can constrain the process to an exact name by using a pattern like `demo$`.

Light React替代品，例如[Preact]（https://www.npmjs.com/package/preact），[react-lite]（https://www.npmjs.com/package/react-lite），或[ Inferno]（https://www.npmjs.com/package/inferno）提供更小的尺寸，同时权衡诸如“propTypes”和合成事件处理之类的功能。用更轻的替代品替换React可以节省大量空间，但如果你这样做，你应该测试得好。
Light React alternatives, such as [Preact](https://www.npmjs.com/package/preact), [react-lite](https://www.npmjs.com/package/react-lite), or [Inferno](https://www.npmjs.com/package/inferno), offer smaller size while trading off functionality like `propTypes` and synthetic event handling. Replacing React with a lighter alternative can save a significant amount of space, but you should test well if you do this.

如果你使用* react-lite *，请将其配置如下：
If you are using *react-lite*, configure it as below:

```javascript
{
  resolve: {
    alias: {
      // Swap the target based on your need
      react: "react-lite",
      "react-dom": "react-lite",
    },
  },
},
```

T>同样的技术也适用于装载机。你可以类似地使用`resolveLoader.alias`。你可以使用该方法调整RequireJS项目以使用webpack。
T> The same technique works with loaders too. You can use `resolveLoader.alias` similarly. You can use the method to adapt a RequireJS project to work with webpack.

##`resolve.extensions`
## `resolve.extensions`

默认情况下，webpack只会在没有扩展名的情况下解析`.js`和`.json`文件，将其调整为包含JSX文件，调整如下：
By default, webpack will resolve only against `.js` and `.json` files while importing without an extension, to tune this to include JSX files, adjust as below:

```javascript
{
  resolve: {
    extensions: [".js", ".json", ".jsx"],
  },
},
```

##`resolve.modules`
## `resolve.modules`

可以通过更改webpack查找模块的位置来更改模块解析过程。默认情况下，它仅在* node_modules *目录中查找。如果你想覆盖那里的包，你可以告诉webpack首先查看其他目录：
The module resolution process can be altered by changing where webpack looks for modules. By default, it will look only within the *node_modules* directory. If you want to override packages there, you could tell webpack to look into other directories first:

```javascript
{
  resolve: {
    modules: ["my_modules", "node_modules"],
  },
},
```

更改后，webpack将首先尝试查看* my_modules *目录。该方法适用于要自定义行为的大型项目。
After the change, webpack will try to look into the *my_modules* directory first. The method can be applicable in large projects where you want to customize behavior.

##`resolve.plugins`
## `resolve.plugins`

Webpack允许你使用`resolve.plugins`字段自定义模块解析行为。请考虑以下插件示例：
Webpack allows you to customize the module resolution behavior using the `resolve.plugins` field. Consider the following plugin examples:

* [directory-named-webpack-plugin]（https://www.npmjs.com/package/directory-named-webpack-plugin）将针对目录的导入映射到与目录名匹配的文件。例如，它会将`import foo从'./foo“;`映射到`import foo from”./foo/foo.js“;`。该模式在React中很受欢迎，使用该插件可以简化代码。 [babel-plugin-module-resolver]（https://www.npmjs.com/package/babel-plugin-module-resolver）通过Babel实现了相同的行为。
* [directory-named-webpack-plugin](https://www.npmjs.com/package/directory-named-webpack-plugin) maps imports made against directories to files matching the directory name. For example, it would map `import foo from "./foo";` to `import foo from "./foo/foo.js";`. The pattern is popular with React and using the plugin will allow you to simplify your code. [babel-plugin-module-resolver](https://www.npmjs.com/package/babel-plugin-module-resolver) achieves the same behavior through Babel.
* [webpack-resolve-short-path-plugin]（https://www.npmjs.com/package/webpack-resolve-short-path-plugin）旨在避免深度嵌套导入，例如`import foo from ... /../../ foo“;`通过添加对tilde（`~`）语法的支持。如果使用插件，`~foo“`的import foo将解析项目根目录。
* [webpack-resolve-short-path-plugin](https://www.npmjs.com/package/webpack-resolve-short-path-plugin) was designed to avoid deeply nested imports like `import foo from "../../../foo";` by adding support for tilde (`~`) syntax. `import foo from "~foo"` would resolve against the project root if the plugin is used.

##在Webpack之外使用包
## Consuming Packages Outside of Webpack

浏览器依赖性（如jQuery）通常通过公共可用的内容交付网络（CDN）提供。 CDN允许你解决在其他地方加载流行软件包的问题。如果已经从CDN加载了一个包并且它在用户缓存中，则无需加载它。
Browser dependencies, like jQuery, are often served through publicly available Content Delivery Networks (CDN). CDNs allow you to push the problem of loading popular packages elsewhere. If a package has been already loaded from a CDN and it's in the user cache, there is no need to load it.

要使用此技术，你应首先将所涉及的依赖项标记为外部：
To use this technique, you should first mark the dependency in question as an external:

```javascript
externals: {
  jquery: "jquery",
},
```

你仍然必须指向CDN并且理想情况下提供本地回退，因此如果CDN不适用于客户端，则需要加载：
You still have to point to a CDN and ideally provide a local fallback, so there is something to load if the CDN does not work for the client:

```html
<script src="//ajax.googleapis.com/ajax/libs/jquery/3.1.1/jquery.min.js"></script>
<script>
    window.jQuery || document.write('<script src="js/jquery-3.1.1.min.js"><\/script>')
</script>
```

T> [html-webpack-cdn-plugin]（https://www.npmjs.com/package/html-webpack-cdn-plugin）是一个选项，如果你使用`HtmlWebpackPlugin`并想要注入`脚本'自动标记。
T> [html-webpack-cdn-plugin](https://www.npmjs.com/package/html-webpack-cdn-plugin) is one option if you are using `HtmlWebpackPlugin` and want to inject a `script` tag automatically.

##处理Globals
## Dealing with Globals

有时模块依赖于全局变量。 jQuery提供的`$`就是一个很好的例子。 Webpack提供了一些允许你处理它们的方法。
Sometimes modules depend on globals. `$` provided by jQuery is a good example. Webpack offers a few ways that allow you to handle them.

{pagebreak}

### Global Injecting
### Injecting Globals

[imports-loader]（https://www.npmjs.com/package/imports-loader）允许你注入全局变量，如下所示：
[imports-loader](https://www.npmjs.com/package/imports-loader) allows you to inject globals as below:

```javascript
{
  module: {
    rules: [
      {
        // Resolve against package path.
        // require.resolve returns a path to it.
        test: require.resolve("jquery-plugin"),
        loader: "imports-loader?$=jquery",
      },
    ],
  },
},
```

###解决Globals
### Resolving Globals

Webpack的`ProvidePlugin`允许webpack在遇到全局变量时解析它们：
Webpack's `ProvidePlugin` allows webpack to resolve globals as it encounters them:

```javascript
{
  plugins: [
    new webpack.ProvidePlugin({
      $: "jquery",
    }),
  ],
},
```

{pagebreak}

###将Globals暴露给浏览器
### Exposing Globals to the Browser

有时你必须将包公开给第三方脚本。 [expose-loader]（https://www.npmjs.com/package/expose-loader）允许如下：
Sometimes you have to expose packages to third-party scripts. [expose-loader](https://www.npmjs.com/package/expose-loader) allows this as follows:

```javascript
{
  test: require.resolve("react"),
  use: "expose-loader?React",
},
```

通过小额外调整，该技术可用于通过`React.Perf` global将React性能实用程序公开给浏览器。你必须将以下代码插入到应用程序入口点才能使其正常工作：
With the small extra tweak, the technique can be used to expose React performance utilities to the browser through `React.Perf` global. You have to insert the following code to your application entry point for this to work:

```javascript
if (process.env.NODE_ENV !== "production") {
  React.Perf = require("react-addons-perf");
}
```

T>最好将[React Developer Tools]（https://github.com/facebook/react-devtools）安装到Chrome以获取更多信息，因为它允许你检查* props *和* state * of你的申请。
T> It can be a good idea to install [React Developer Tools](https://github.com/facebook/react-devtools) to Chrome for even more information as it allows you to inspect *props* and *state* of your application.

T> [script-loader]（https://www.npmjs.com/package/script-loader）允许你在全局上下文中执行脚本。如果你使用的脚本依赖于全局注册设置，则必须执行此操作。
T> [script-loader](https://www.npmjs.com/package/script-loader) allows you to execute scripts in a global context. You have to do this if the scripts you are using rely on a global registration setup.

{pagebreak}

##删除未使用的模块
## Removing Unused Modules

尽管软件包可以很好地开箱即用，但它们有时会为你的项目带来太多代码。 [Moment.js]（https://www.npmjs.com/package/moment）是一个很受欢迎的例子。默认情况下，它会将区域设置数据带到项目中。
Even though packages can work well out of the box, they bring too much code to your project sometimes. [Moment.js](https://www.npmjs.com/package/moment) is a popular example. It brings locale data to your project by default.

禁用该行为的最简单方法是使用`IgnorePlugin`忽略语言环境：
The easiest method to disable that behavior is to use `IgnorePlugin` to ignore locales:

```javascript
{
  plugins: [new webpack.IgnorePlugin(/^\.\/locale$/, /moment$/)],
},
```

T>你可以使用相同的机制来解决有问题的依赖项。示例：`new webpack.IgnorePlugin（/ ^（buffertools）$ /）`。
T> You can use the same mechanism to work around problematic dependencies. Example: `new webpack.IgnorePlugin(/^(buffertools)$/)`.

要将特定的语言环境引入项目，你应该使用`ContextReplacementPlugin`：
To bring specific locales to your project, you should use `ContextReplacementPlugin`:

```javascript
{
  plugins: [
    new webpack.ContextReplacementPlugin(
      /moment[\/\\]locale$/,
      /de|fi/
    ),
  ],
},
```

T>有一个[Stack Overflow问题]（https://stackoverflow.com/questions/25384360/how-to-prevent-moment-js-from-loading-locales-with-webpack/25426019）详细介绍了这些想法。另见[Ivan Akulov对“ContextReplacementPlugin”的解释]（https://iamakulov.com/notes/webpack-contextreplacementplugin/）。
T> There's a [Stack Overflow question](https://stackoverflow.com/questions/25384360/how-to-prevent-moment-js-from-loading-locales-with-webpack/25426019) that covers these ideas in detail. See also [Ivan Akulov's explanation of `ContextReplacementPlugin`](https://iamakulov.com/notes/webpack-contextreplacementplugin/).

{pagebreak}

##管理预构建的依赖项
## Managing Pre-built Dependencies

webpack可能会给出以下警告以及某些依赖项：
It's possible webpack gives the following warning with certain dependencies:

```bash
WARNING in ../~/jasmine-promises/dist/jasmine-promises.js
Critical dependencies:
1:113-120 This seems to be a pre-built javascript file. Though this is possible, it's not recommended. Try to require the original source to get better results.
 @ ../~/jasmine-promises/dist/jasmine-promises.js 1:113-120
```

如果包指向预先构建（即，缩小且已经处理）的文件，则可能发生警告。 Webpack检测到这种情况并发出警告。
The warning can happen if a package points at a pre-built (i.e., minified and already processed) file. Webpack detects this case and warns against it.

如上所述，可以通过将包别名化为源版本来消除警告。鉴于有时源不可用，另一个选择是告诉webpack跳过通过`module.noParse`解析文件。它接受RegExp或RegExps数组，可以配置如下：
The warning can be eliminated by aliasing the package to a source version as discussed above. Given sometimes the source is not available, another option is to tell webpack to skip parsing the files through `module.noParse`. It accepts either a RegExp or an array of RegExps and can be configured as below:

```javascript
{
  module: {
    noParse: /node_modules\/demo-package\/dist\/demo-package.js/,
  },
},
```

W>在禁用警告时要小心，因为它可以隐藏潜在的问题。先考虑替代方案。有一个[webpack问题]（https://github.com/webpack/webpack/issues/1617）详细讨论了这个问题。
W> Take care when disabling warnings as it can hide underlying issues. Consider alternatives first. There's a [webpack issue](https://github.com/webpack/webpack/issues/1617) that discusses the problem in detail.

{pagebreak}

##管理符号链接
## Managing Symbolic Links

符号链接或符号链接是一种操作系统级功能，允许你通过文件系统指向其他文件而无需复制它们。你可以使用`npm link`为正在开发的包创建全局符号链接，然后使用`npm unlink`删除链接。
Symbolic links, or symlinks, are an operating system level feature that allows you to point to other files through a file system without copying them. You can use `npm link` to create global symlinks for packages under development and then use `npm unlink` to remove the links.

Webpack将符号链接解析为Node的完整路径。问题是，如果你不知道这个事实，这种行为会让你感到惊讶，特别是如果你依赖webpack处理。可以解决webpack问题[＃1643]（https://github.com/webpack/webpack/issues/1643）和[＃985]（https://github.com/webpack/webpack）中讨论的行为。 /问题/ 985）。 Webpack核心行为将来可能会有所改进，使这些变通方法变得不必要。
Webpack resolves symlinks to their full path as Node does. The problem is that if you are unaware of this fact, the behavior can surprise you especially if you rely on webpack processing. It's possible to work around the behavior as discussed in webpack issues [#1643](https://github.com/webpack/webpack/issues/1643) and [#985](https://github.com/webpack/webpack/issues/985). Webpack core behavior may improve in the future to make these workarounds unnecessary.

T>你可以通过将`resolve.symlinks`设置为`false`来禁用webpack的符号链接处理。
T> You can disable webpack's symlink handling by setting `resolve.symlinks` as `false`.

##获取有关包的见解
## Getting Insights on Packages

为了获得更多信息，npm为基本查询提供了`npm info <package>`命令。你可以使用它来检查与包关联的元数据，同时确定版本相关信息。请考虑以下工具：
To get more information, npm provides `npm info <package>` command for basic queries. You can use it to check the metadata associated with packages while figuring out version related information. Consider the following tools as well:

* [package-config-checker]（https://www.npmjs.com/package/package-config-checker）更进了一步。它允许你更好地了解项目的哪些软件包最近更新，并提供了深入了解依赖项的方法。例如，它可以揭示哪些包可以使用与下载大小相关的改进。
* [package-config-checker](https://www.npmjs.com/package/package-config-checker) goes a step further. It allows you to understand better which packages of your project have updated recently and it provides means to get insight into your dependencies. It can reveal which packages could use download size related improvements for example.
* [slow-deps]（https://www.npmjs.com/package/slow-deps）可以揭示项目的哪些依赖项安装速度最慢。
* [slow-deps](https://www.npmjs.com/package/slow-deps) can reveal which dependencies of a project are the slowest to install.
* [称量]（https://www.npmjs.com/package/weigh）可用于计算包以不同方式（未压缩，缩小，gzip）提供给浏览器时的大致大小。
* [weigh](https://www.npmjs.com/package/weigh) can be used figure out the approximate size of a package when it's served to a browser in different ways (uncompressed, minified, gzipped).

##结论
## Conclusion

Webpack可以毫无问题地使用大多数npm软件包。但有时候，使用webpack的解析机制需要修补。
Webpack can consume most npm packages without a problem. Sometimes, though, patching is required using webpack's resolution mechanism.

回顾一下：
To recap:

*使用webpack的模块分辨率为你带来好处。有时你可以通过调整分辨率来解决问题。不过，尝试将改进上游推进到项目本身通常是个好主意。
* Use webpack's module resolution to your benefit. Sometimes you can work around issues by tweaking resolution. Often it's a good idea to try to push improvements upstream to the projects themselves, though.
* Webpack允许你修补已解决的模块。给定特定的依赖关系期望全局变量，你可以注入它们。你还可以将模块公开为全局变量，因为这对于某些开发工具来说是必需的。
* Webpack allows you to patch resolved modules. Given specific dependencies expect globals, you can inject them. You can also expose modules as globals as this is necessary for certain development tooling to work.

