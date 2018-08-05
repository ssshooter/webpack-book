＃什么是Webpack
# What is Webpack

Webpack是**模块捆绑器**。 Webpack可以与单独的任务运行器捆绑在一起。然而，由于社区开发的webpack插件，捆绑器和任务运行器之间的界限变得模糊。有时，这些插件用于执行通常在webpack之外完成的任务，例如清理构建目录或部署构建。
Webpack is a **module bundler**. Webpack can take care of bundling alongside a separate task runner. However, the line between bundler and task runner has become blurred thanks to community developed webpack plugins. Sometimes these plugins are used to perform tasks that are usually done outside of webpack, such as cleaning the build directory or deploying the build.

React和**热模块替换**（HMR）有助于推广webpack并导致其在其他环境中的使用，例如[Ruby on Rails]（https://github.com/rails/webpacker）。尽管它的名字，webpack并不仅限于网络。它也可以与其他目标捆绑在一起，如* Build Targets *章节中所述。
React, and **Hot Module Replacement** (HMR) helped to popularize webpack and led to its usage in other environments, such as [Ruby on Rails](https://github.com/rails/webpacker). Despite its name, webpack is not limited to the web alone. It can bundle with other targets as well, as discussed in the *Build Targets* chapter.

T>如果您想更详细地了解构建工具及其历史记录，请查看*构建工具比较*附录。
T> If you want to understand build tools and their history in better detail, check out the *Comparison of Build Tools* appendix.

## Webpack依赖于模块
## Webpack Relies on Modules

您可以使用webpack捆绑的最小项目包括**输入**和**输出**。捆绑过程从用户定义的**条目**开始。条目本身是**模块**，可以通过**导入**指向其他模块。
The smallest project you can bundle with webpack consists of **input** and **output**. The bundling process begins from user-defined **entries**. Entries themselves are **modules** and can point to other modules through **imports**.

当您使用webpack捆绑项目时，它会遍历导入，构建项目的**依赖关系图**，然后根据配置生成**输出**。此外，可以定义**分裂点**以在项目代码本身内创建单独的捆绑包。
When you bundle a project using webpack, it traverses the imports, constructing a **dependency graph** of the project and then generates **output** based on the configuration. Additionally, it's possible to define **split points** to create separate bundles within the project code itself.

Webpack支持开箱即用的ES2015，CommonJS和AMD模块格式。加载器机制也适用于CSS，通过* css-loader *支持`@ import`和`url（）`。您还可以找到特定任务的插件，例如缩小，国际化，HMR等。
Webpack supports ES2015, CommonJS, and AMD module formats out of the box. The loader mechanism works for CSS as well, with `@import` and `url()` support through *css-loader*. You can also find plugins for specific tasks, such as minification, internationalization, HMR, and so on.

T>依赖图是描述节点如何相互关联的有向图。在这种情况下，图形定义是通过文件之间的引用（`require`，`import`）定义的。 Webpack在不执行源的情况下静态遍历这些源，以生成创建bundle所需的图。
T> A dependency graph is a directed graph that describes how nodes relate to each other. In this case, the graph definition is defined through references (`require`, `import`) between files. Webpack statically traverses these without executing the source to generate the graph it needs to create bundles.

## Webpack的执行流程
## Webpack's Execution Process

![Webpack's execution process](images/webpack-process.png)

Webpack从**条目**开始工作。通常这些是JavaScript模块，其中webpack开始其遍历过程。在此过程中，webpack根据** loader **配置评估条目匹配，告诉webpack如何转换每个匹配。
Webpack begins its work from **entries**. Often these are JavaScript modules where webpack begins its traversal process. During this process, webpack evaluates entry matches against **loader** configurations that tell webpack how to transform each match.

{pagebreak}

###解决方案流程
### Resolution Process

条目本身就是一个模块。当webpack遇到一个时，webpack会尝试使用条目的`resolve`配置将条目与文件系统匹配。除了* node_modules *之外，您还可以告诉webpack对特定目录执行查找。也可以调整webpack与文件扩展名匹配的方式，并且可以为目录定义特定的别名。 *消费包*章节更详细地介绍了这些想法。
An entry itself is a module. When webpack encounters one, webpack tries to match the entry against the file system using the entry's `resolve` configuration. You can tell webpack to perform the lookup against specific directories in addition to *node_modules*. It's also possible to adjust the way webpack matches against file extensions, and you can define specific aliases for directories. The *Consuming Packages* chapter covers these ideas in greater detail.

如果解析通过失败，webpack会引发运行时错误。如果webpack设法正确解析文件，webpack将根据加载器定义对匹配的文件执行处理。每个加载器对模块内容应用特定的转换。
If the resolution pass failed, webpack raises a runtime error. If webpack managed to resolve a file correctly, webpack performs processing over the matched file based on the loader definition. Each loader applies a specific transformation against the module contents.

可以通过多种方式配置加载程序与已解析文件匹配的方式，包括文件类型和文件系统中的位置。 Webpack的灵活性甚至允许您根据*将*导入到项目中的特定转换应用于文件。
The way a loader gets matched against a resolved file can be configured in multiple ways, including by file type and by location within the file system. Webpack's flexibility even allows you to apply a specific transformation to a file based on *where* it was imported into the project.

对webpack的加载器执行相同的解析过程。 Webpack允许您在确定应使用哪个加载器时应用类似的逻辑。由于这个原因，装载程序已经解析了自己的配置。如果webpack无法执行加载程序查找，则会引发运行时错误。
The same resolution process is performed against webpack's loaders. Webpack allows you to apply similar logic when determining which loader it should use. Loaders have resolve configurations of their own for this reason. If webpack fails to perform a loader lookup, it will raise a runtime error.

T>要解决，webpack依赖于下面的[enhanced-resolve]（https://www.npmjs.com/package/enhanced-resolve）包。
T> To resolve, webpack relies on [enhanced-resolve](https://www.npmjs.com/package/enhanced-resolve) package underneath.

### Webpack解析任何文件类型
### Webpack Resolves Against Any File Type

Webpack将在构造依赖图时解析它遇到的每个模块。如果条目包含依赖项，则将针对每个依赖项递归执行该过程，直到遍历完成为止。 Webpack可以针对任何文件类型执行此过程，这与Babel或Sass编译器等专用工具不同。
Webpack will resolve each module it encounters while constructing the dependency graph. If an entry contains dependencies, the process will be performed recursively against each dependency until the traversal has completed. Webpack can perform this process against any file type, unlike specialized tools like the Babel or Sass compiler.

Webpack使您可以控制如何处理遇到的不同资产。例如，您可以决定**内联**资产到您的JavaScript包以避免请求。 Webpack还允许您使用CSS模块等技术将样式与组件结合，并避免标准CSS样式问题。这种灵活性使webpack非常有价值。
Webpack gives you control over how to treat different assets it encounters. For example, you can decide to **inline** assets to your JavaScript bundles to avoid requests. Webpack also allows you to use techniques like CSS Modules to couple styling with components, and to avoid issues of standard CSS styling. This flexibility is what makes webpack so valuable.

尽管webpack主要用于捆绑JavaScript，但它可以捕获图像或字体等资源，并为它们发出单独的文件。条目只是捆绑过程的起点。 webpack发出的内容完全取决于您配置它的方式。
Although webpack is used mainly to bundle JavaScript, it can capture assets like images or fonts and emit separate files for them. Entries are only a starting point of the bundling process. What webpack emits depends entirely on the way you configure it.

###评估流程
### Evaluation Process

假设找到所有加载器，webpack将从下到上和从右到左（`styleLoader（cssLoader（'。/ main.css'））`）评估匹配的加载器，同时依次通过每个加载器运行模块。因此，您将获得webpack将在结果**包中注入的输出**。 * Loader Definitions *章节详细介绍了该主题。
Assuming all loaders were found, webpack evaluates the matched loaders from bottom to top and right to left (`styleLoader(cssLoader('./main.css'))`) while running the module through each loader in turn. As a result, you get output which webpack will inject in the resulting **bundle**. The *Loader Definitions* chapter covers the topic in detail.

如果所有加载器评估都在没有运行时错误的情况下完成，则webpack会在最后一个包中包含源。 **插件**允许您在捆绑过程的不同阶段拦截**运行时事件**。
If all loader evaluation completed without a runtime error, webpack includes the source in the last bundle. **Plugins** allow you to intercept **runtime events** at different stages of the bundling process.

虽然装载机可以做很多事情，但它们不能为高级任务提供足够的动力。插件可以拦截webpack提供的**运行时事件**。一个很好的例子是由“MiniCssExtractPlugin”执行的包提取，当与加载器一起使用时，从包中提取CSS文件并将其提取到单独的文件中。如果没有这一步，CSS将在生成的JavaScript中内联，因为webpack默认将所有代码视为JavaScript。提取思想在* Separating CSS *一章中讨论。
Although loaders can do a lot, they don’t provide enough power for advanced tasks. Plugins can intercept **runtime events** supplied by webpack. A good example is bundle extraction performed by the `MiniCssExtractPlugin` which, when used with a loader, extracts CSS files out of the bundle and into a separate file. Without this step, CSS would be inlined in the resulting JavaScript, as webpack treats all code as JavaScript by default. The extraction idea is discussed in the *Separating CSS* chapter.

###整理
### Finishing

在评估每个模块之后，webpack写入**输出**。输出包括一个引导脚本，其中包含一个描述如何在浏览器中开始执行结果的清单。可以将清单提取到自己的文件中，如本书后面所述。输出根据您使用的构建目标而有所不同（定位Web不是唯一选项）。
After every module has been evaluated, webpack writes **output**. The output includes a bootstrap script with a manifest that describes how to begin executing the result in the browser. The manifest can be extracted to a file of its own, as discussed later in the book. The output differs based on the build target you are using (targeting web is not the only option).

这并不是捆绑过程的全部内容。例如，您可以定义特定的**拆分点**，其中webpack生成基于应用程序逻辑加载的单独包。这个想法在* Code Splitting *章节中讨论。
That’s not all there is to the bundling process. For example, you can define specific **split points** where webpack generates separate bundles that are loaded based on application logic. This idea is discussed in the *Code Splitting* chapter.

## Webpack是配置驱动的
## Webpack Is Configuration Driven

webpack的核心依赖于配置。以下是根据[官方webpack教程]（https://webpack.js.org/get-started/）改编的示例配置，其中包含要点：
At its core, webpack relies on configuration. Here is a sample configuration adapted from [the official webpack tutorial](https://webpack.js.org/get-started/) that covers the main points:

** ** webpack.config.js
**webpack.config.js**

```javascript
const webpack = require("webpack");

module.exports = {
  // Where to start bundling
  entry: {
    app: "./entry.js",
  },

  // Where to output
  output: {
    // Output to the same directory
    path: __dirname,

    // Capture name from the entry using a pattern
    filename: "[name].js",
  },

  // How to resolve encountered imports
  module: {
    rules: [
      {
        test: /\.css$/,
        use: ["style-loader", "css-loader"],
      },
      {
        test: /\.js$/,
        use: "babel-loader",
        exclude: /node_modules/,
      },
    ],
  },

  // What extra processing to perform
  plugins: [
    new webpack.DefinePlugin({ ... }),
  ],

  // Adjust module resolution algorithm
  resolve: {
    alias: { ... },
  },
};
```

Webpack的配置模型有时会感觉有点不透明，因为配置文件可能看起来是单片的。除非你知道背后的想法，否则很难理解webpack在做什么。提供驯服配置的方法是本书存在的主要目的之一。
Webpack's configuration model can feel a bit opaque at times as the configuration file can appear monolithic. It can be difficult to understand what webpack is doing unless you know the ideas behind it. Providing means to tame configuration is one of the primary purposes why this book exists.

##资产哈希
## Asset Hashing

使用webpack，您可以为每个包名称注入一个哈希值（例如，* app.d587bbd6.js *），以便在进行更改时使客户端上的包无效。捆绑分割允许客户端在理想情况下仅重新加载一小部分数据。
With webpack, you can inject a hash to each bundle name (e.g., *app.d587bbd6.js*) to invalidate bundles on the client side as changes are made. Bundle-splitting allows the client to reload only a small part of the data in the ideal case.

##热模块更换
## Hot Module Replacement

您可能已经熟悉了[LiveReload]（http://livereload.com/）或[BrowserSync]（http://www.browsersync.io/）等工具。这些工具会在您进行更改时自动刷新浏览器。 *热模块更换*（HMR）更进一步。在React的情况下，它允许应用程序在不强制刷新的情况下维持其状态。虽然这听起来不那么特别，但它可以在实践中产生很大的不同。
You are likely familiar with tools, such as [LiveReload](http://livereload.com/) or [BrowserSync](http://www.browsersync.io/), already. These tools refresh the browser automatically as you make changes. *Hot Module Replacement* (HMR) takes things one step further. In the case of React, it allows the application to maintain its state without forcing a refresh. While this does not sound all that special, it can make a big difference in practice.

通过[livereactload]（https://github.com/milankinen/livereactload）也可以在Browserify中使用HMR，因此它不是webpack独有的功能。
HMR is also available in Browserify via [livereactload](https://github.com/milankinen/livereactload), so it’s not a webpack exclusive feature.

## Code Sclitting
## Code Splitting

除HMR外，webpack的捆绑功能也非常广泛。 Webpack允许您以各种方式拆分代码。您甚至可以在应用程序执行时动态加载代码。这种延迟加载特别适用于更广泛的应用程序，因为依赖关系可以根据需要即时加载。
In addition to HMR, webpack’s bundling capabilities are extensive. Webpack allows you to split code in various ways. You can even load code dynamically as your application gets executed. This sort of lazy loading comes in handy especially for broader applications, as dependencies can be loaded on the fly as needed.

即使是小型应用程序也可以从代码分割中受益，因为它允许用户更快地获得可用的东西。毕竟，性能是一项功能。了解基本技术是值得的。
Even small applications can benefit from code splitting, as it allows the users to get something usable in their hands faster. Performance is a feature, after all. Knowing the basic techniques is worthwhile.

##结论
## Conclusion

Webpack带来了重要的学习曲线。然而，考虑到长期可以节省多少时间和精力，这是一个值得学习的工具。为了更好地了解它与其他工具的比较，请查看[官方比较]（https://webpack.js.org/comparison/）。
Webpack comes with a significant learning curve. However, it’s a tool worth learning, given how much time and effort it can save over the long term. To get a better idea how it compares to other tools, check out [the official comparison](https://webpack.js.org/comparison/).

Webpack不会解决所有问题。但是，它确实解决了捆绑问题。在开发过程中，这不用担心。单独使用* package.json *和webpack可以带你走远。
Webpack won’t solve everything. However, it does solve the problem of bundling. That’s one less worry during development. Using *package.json* and webpack alone can take you far.

总结一下：
To summarize:

* Webpack是**模块捆绑器**，但您也可以使用它运行任务。
* Webpack is a **module bundler**, but you can also use it running  tasks as well.
* Webpack依赖于下面的**依赖图**。 Webpack遍历源构建图，并使用此信息和配置生成包。
* Webpack relies on a **dependency graph** underneath. Webpack traverses through the source to construct the graph, and it uses this information and configuration to generate bundles.
* Webpack依赖于**加载器**和**插件**。加载器在模块级别上运行，而插件依赖于webpack提供的钩子，并且可以最好地访问其执行过程。
* Webpack relies on **loaders** and **plugins**. Loaders operate on a module level, while plugins rely on hooks provided by webpack and have the best access to its execution process.
* Webpack的**配置**描述了如何转换图形的资产以及它应该生成什么样的输出。如果使用**代码拆分**等功能，则可以将部分信息包含在源代码中。
* Webpack’s **configuration** describes how to transform assets of the graphs and what kind of output it should generate. Part of this information can be included in the source itself if features like **code splitting** are used.
* **热模块更换**（HMR）有助于推广webpack。这是一项功能，可以通过更新浏览器中的代码来增强开发体验，而无需刷新整页。
* **Hot Module Replacement** (HMR) helped to popularize webpack. It's a feature that can enhance the development experience by updating code in the browser without needing a full page refresh.
* Webpack可以为文件名生成**哈希**，允许您在内容更改时使过去的包无效。
* Webpack can generate **hashes** for filenames allowing you to invalidate past bundles as their contents change.

在本书的下一部分中，您将学习使用webpack构建开发配置，同时了解有关其基本概念的更多信息。
In the next part of the book, you'll learn to construct a development configuration using webpack while learning more about its basic concepts.

T>如果您仍然不确定webpack或者为什么需要捆绑包，请阅读[我为什么要使用Webpack？]（http://tinselcity.net/whys/packers）。
T> If you are still unsure of webpack or why bundlers are required, read [Why would I use a Webpack?](http://tinselcity.net/whys/packers).

