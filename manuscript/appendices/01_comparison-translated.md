＃构建工具的比较
# Comparison of Build Tools

在当天，将脚本连接在一起就足够了。但是，时代已经发生了变化，现在分发你的JavaScript代码可能是一项复杂的工作。随着单页应用程序（SPA）的兴起，这个问题已经升级。他们倾向于依赖许多庞大的图书馆。
Back in the day, it was enough to concatenate scripts together. Times have changed, though, and now distributing your JavaScript code can be a complicated endeavor. This problem has escalated with the rise of single-page applications (SPAs). They tend to rely on many hefty libraries.

出于这个原因，有多种策略可以加载它们。你可以一次加载它们或考虑根据需要加载库。 Webpack支持许多这样的策略。
For this reason, there are multiple strategies on how to load them. You could load them all at once or consider loading libraries as you need them. Webpack supports many of these sorts of strategies.

Node和[npm]（https://www.npmjs.com/）（它的包管理器）的流行提供了更多的上下文。在npm开始流行之前，很难消耗依赖关系。有一段时间人们开发了前端特定的包裹经理，但最终赢了。现在依赖管理比以前更舒服，尽管仍有挑战需要克服。
The popularity of Node and [npm](https://www.npmjs.com/), its package manager, provide more context. Before npm became popular, it was hard to consume dependencies. There was a period when people developed frontend specific package managers, but npm won in the end. Now dependency management is more comfortable than before, although there are still challenges to overcome.

##任务跑步者
## Task Runners

从历史上看，已经有很多构建工具。 * Make *可能是最知名的，它仍然是一个可行的选择。专门的*任务运行者*，例如Grunt和Gulp，是专门针对JavaScript开发人员而创建的。通过npm提供的插件使得任务运行器功能强大且可扩展。甚至可以使用npm`scrip`作为任务运行器。这很常见，特别是对于webpack。
Historically speaking, there have been many build tools. *Make* is perhaps the best known, and it's still a viable option. Specialized *task runners*, such as Grunt and Gulp were created particularly with JavaScript developers in mind. Plugins available through npm made both task runners powerful and extendable. It's possible to use even npm `scripts` as a task runner. That's common, particularly with webpack.

{pagebreak}

### Make
### Make

[Make]（https://en.wikipedia.org/wiki/Make_%28software%29）可以追溯到最近，因为它最初是在1977年发布的。即使它是一个旧工具，它仍然具有相关性。 Make允许你为各种目的编写单独的任务。例如，你可以使用不同的任务来创建生成构建，缩小JavaScript或运行测试。你可以在许多其他工具中找到相同的想法。
[Make](https://en.wikipedia.org/wiki/Make_%28software%29) goes way back, as it was initially released in 1977. Even though it's an old tool, it has remained relevant. Make allows you to write separate tasks for various purposes. For instance, you could have different tasks for creating a production build, minifying your JavaScript or running tests. You can find the same idea in many other tools.

尽管Make主要用于C项目，但它并没有以任何方式与C相关联。 James Coglan详细讨论了[如何使用Make with JavaScript]（https://blog.jcoglan.com/2014/02/05/building-javascript-projects-with-make/）。考虑基于James的帖子的缩写代码：
Even though Make is mostly used with C projects, it's not tied to C in any way. James Coglan discusses in detail [how to use Make with JavaScript](https://blog.jcoglan.com/2014/02/05/building-javascript-projects-with-make/). Consider the abbreviated code based on James' post below:

**Makefile**

```makefile
PATH  := node_modules/.bin:$(PATH)
SHELL := /bin/bash

source_files := $(wildcard lib/*.coffee)
build_files  := $(source_files:%.coffee=build/%.js)
app_bundle   := build/app.js
spec_coffee  := $(wildcard spec/*.coffee)
spec_js      := $(spec_coffee:%.coffee=build/%.js)

libraries    := vendor/jquery.js

.PHONY: all clean test

all: $(app_bundle)

build/%.js: %.coffee
    coffee -co $(dir $@) $<

$(app_bundle): $(libraries) $(build_files)
    uglifyjs -cmo $@ $^

test: $(app_bundle) $(spec_js)
    phantomjs phantom.js

clean:
    rm -rf build
```

使用Make，你可以使用特定于Make的语法和终端命令对任务进行建模，从而可以与webpack集成。
With Make, you model your tasks using Make-specific syntax and terminal commands making it possible to integrate with webpack.

### npm`scripts`作为任务运行器
### npm `scripts` as a Task Runner

尽管npm CLI主要不是用作任务运行器，但是由于* package.json *`scripts`字段，它可以正常工作。考虑以下示例：
Even though npm CLI wasn't primarily designed to be used as a task runner, it works as such thanks to *package.json* `scripts` field. Consider the example below:

** **的package.json
**package.json**

```json
"scripts": {
  "start": "webpack-dev-server --env development",
  "build": "webpack --env production",
  "build:stats": "webpack --env production --json > stats.json"
},
```

这些脚本可以使用`npm run`列出，然后使用`npm run <script>`执行。你还可以使用`test：watch`之类的约定命名脚本。这种方法的问题在于它需要保持跨平台。
These scripts can be listed using `npm run` and then executed using `npm run <script>`. You can also namespace your scripts using a convention like `test:watch`. The problem with this approach is that it takes care to keep it cross-platform.

你可能希望使用[rimraf]（https://www.npmjs.com/package/rimraf）等实用程序而不是`rm -rf`。可以在此处调用其他任务运行程序来隐藏你正在使用的任务。这样，你可以在保持界面相同的同时重构工具。
Instead of `rm -rf`, you likely want to use utilities such as [rimraf](https://www.npmjs.com/package/rimraf) and so on. It's possible to invoke other tasks runners here to hide the fact that you are using one. This way you can refactor your tooling while keeping the interface as the same.

### Grunt

![Grunt](images/grunt.png)

[Grunt]（http://gruntjs.com/）是前端开发人员的第一个着名任务选手。它的插件架构有助于它的流行。插件通常很复杂。因此，当配置增长时，很难理解发生了什么。
[Grunt](http://gruntjs.com/) was the first famous task runner for frontend developers. Its plugin architecture contributed towards its popularity. Plugins are often complicated by themselves. As a result, when configuration grows, it can become difficult to understand what's going on.

以下是[Grunt文档]（http://gruntjs.com/sample-gruntfile）中的示例。在此配置中，你可以定义linting和watcher任务。当* watch *任务运行时，它也会触发* lint *任务。这样，当你运行Grunt时，你在编辑源代码时会在终端中实时收到警告。
Here's an example from [Grunt documentation](http://gruntjs.com/sample-gruntfile). In this configuration, you define a linting and watcher tasks. When the *watch* task gets run, it triggers the *lint* task as well. This way, as you run Grunt, you get warnings in real-time in the terminal as you edit the source code.

{pagebreak}

** ** Gruntfile.js
**Gruntfile.js**

```javascript
module.exports = grunt => {
  grunt.initConfig({
    lint: {
      files: ["Gruntfile.js", "src/**/*.js", "test/**/*.js"],
      options: {
        globals: {
          jQuery: true,
        },
      },
    },
    watch: {
      files: ["<%= lint.files %>"],
      tasks: ["lint"],
    },
  });

  grunt.loadNpmTasks("grunt-contrib-jshint");
  grunt.loadNpmTasks("grunt-contrib-watch");

  grunt.registerTask("default", ["lint"]);
};
```

在实践中，你将有许多小任务用于特定目的，例如构建项目。 Grunt强大功能的一个重要组成部分就是隐藏了很多线路。
In practice, you would have many small tasks for specific purposes, such as building the project. An essential part of the power of Grunt is that it hides a lot of the wiring from you.

太过分了，这可能会有问题。很难理解幕后发生了什么。这是Grunt的建筑课程。
Taken too far, this can get problematic. It can become hard to understand what's going on under the hood. That's the architectural lesson to take from Grunt.

T> [grunt-webpack]（https://www.npmjs.com/package/grunt-webpack）插件允许你在Grunt环境中使用webpack，同时将繁重的工作留给webpack。
T> [grunt-webpack](https://www.npmjs.com/package/grunt-webpack) plugin allows you to use webpack in a Grunt environment while you leave the heavy lifting to webpack.

### Gulp
### Gulp

![Gulp](images/gulp.png)

[Gulp]（http://gulpjs.com/）采取了不同的方法。你不必依赖每个插件的配置，而是处理实际代码。如果你熟悉Unix和管道，你会喜欢Gulp。你有* sources *来匹配文件，*过滤器*来操作这些源，* sinks *来管道构建结果。
[Gulp](http://gulpjs.com/) takes a different approach. Instead of relying on configuration per plugin, you deal with actual code. If you are familiar with Unix and piping, you'll like Gulp. You have *sources* to match files, *filters* to operate on these sources, and *sinks* to pipe the build results.

这是一个缩写的示例* Gulpfile *，它改编自项目的自述文件，让你更好地了解该方法：
Here's an abbreviated sample *Gulpfile* adapted from the project's README to give you a better idea of the approach:

** ** Gulpfile.js
**Gulpfile.js**

```javascript
const gulp = require("gulp");
const coffee = require("gulp-coffee");
const concat = require("gulp-concat");
const uglify = require("gulp-uglify");
const sourcemaps = require("gulp-sourcemaps");
const del = require("del");

const paths = {
  scripts: ["client/js/**/*.coffee", "!client/external/**/*.coffee"],
};

// Not all tasks need to use streams.
// A gulpfile is another node program
// and you can use all packages available on npm.
gulp.task("clean", () => del(["build"]));
gulp.task("scripts", ["clean"], () =>
  // Minify and copy all JavaScript (except vendor scripts)
  // with source maps all the way down.
  gulp
    .src(paths.scripts)
    // Pipeline within pipeline
    .pipe(sourcemaps.init())
    .pipe(coffee())
    .pipe(uglify())
    .pipe(concat("all.min.js"))
    .pipe(sourcemaps.write())
    .pipe(gulp.dest("build/js"))
);
gulp.task("watch", () => gulp.watch(paths.scripts, ["scripts"]));

// The default task (called when you run `gulp` from CLI).
gulp.task("default", ["watch", "scripts"]);
```

鉴于配置是代码，如果遇到麻烦，你总是可以破解它。你可以将现有Node包打包为Gulp插件，依此类推。与Grunt相比，你可以更清楚地知道发生了什么。不过，你仍然会为休闲任务编写大量样板文件。这就是新方法的用武之地。
Given the configuration is code, you can always hack it if you run into troubles. You can wrap existing Node packages as Gulp plugins, and so on. Compared to Grunt, you have a clearer idea of what's going on. You still end up writing a lot of boilerplate for casual tasks, though. That is where newer approaches come in.

T> [webpack-stream]（https://www.npmjs.com/package/webpack-stream）允许你在Gulp环境中使用webpack。
T> [webpack-stream](https://www.npmjs.com/package/webpack-stream) allows you to use webpack in a Gulp environment.

##脚本加载器
## Script Loaders

有一段时间，[RequireJS]（http://requirejs.org/），一个脚本加载器，很受欢迎。我们的想法是提供异步模块定义并在此基础上构建。幸运的是，标准已经赶上了，而且现在看来，RequireJS似乎更像是一种好奇心。
For a while, [RequireJS](http://requirejs.org/), a script loader, was popular. The idea was to provide an asynchronous module definition and build on top of that. Fortunately, the standards have caught up, and RequireJS seems more like a curiosity now.

### RequireJS
### RequireJS

[RequireJS]（http://requirejs.org/）可能是第一个真正受欢迎的脚本加载器。它首次正确地了解了Web上的模块化JavaScript。最吸引人的是AMD。它引入了一个`define`包装器：
[RequireJS](http://requirejs.org/) was perhaps the first script loader that became genuinely popular. It gave the first proper look at what modular JavaScript on the web could be. Its greatest attraction was AMD. It introduced a `define` wrapper:

```javascript
define(["./MyModule.js"], function (MyModule) {
  return function() {}; // Export at module root
});

// or
define(["./MyModule.js"], function (MyModule) {
  return {
    hello: function() {...}, // Export as a module function
  };
});
```

顺便说一下，可以在包装器中使用`require`：
Incidentally, it's possible to use `require` within the wrapper:

```javascript
define(["require"], function (require) {
  var MyModule = require("./MyModule.js");

  return function() {...};
});
```

后一种方法消除了一部分杂乱。你仍然会遇到感觉多余的代码。 ES2015和其他标准解决了这个问题。
This latter approach eliminates a part of the clutter. You still end up with code that feels redundant. ES2015 and other standards solve this.

T> Jamund Ferguson写了一篇关于如何从[RequireJS到webpack]（https://gist.github.com/xjamundx/b1c800e9282e16a6a18e）移植的优秀博客系列。
T> Jamund Ferguson has written an excellent blog series on how to port from [RequireJS to webpack](https://gist.github.com/xjamundx/b1c800e9282e16a6a18e).

### JSPM
### JSPM

![JSPM](images/jspm.png)

使用[JSPM]（http://jspm.io/）与以前的工具完全不同。它附带了一个自己的命令行工具，用于将新包安装到项目中，创建生产包等等。它支持[SystemJS插件]（https://github.com/systemjs/systemjs#plugins），允许你为项目加载各种格式。
Using [JSPM](http://jspm.io/) is entirely different than previous tools. It comes with a command line tool of its own that is used to install new packages to the project, create a production bundle, and so on. It supports [SystemJS plugins](https://github.com/systemjs/systemjs#plugins) that allow you to load various formats to your project.

## Bundlers
## Bundlers

任务跑步者是高水平的伟大工具。它们允许你以跨平台方式执行操作。当你需要将各种资产拼接在一起并生成捆绑包时，问题就开始了。 * bundlers *，例如Br​​owserify，Brunch或webpack，因此存在，并且它们在较低的抽象级别上运行。它们不是对文件进行操作，而是对模块和资产进行操作。
Task runners are great tools on a high level. They allow you to perform operations in a cross-platform manner. The problems begin when you need to splice various assets together and produce bundles. *bundlers*, such as Browserify, Brunch, or webpack, exist for this reason and they operate on a lower level of abstraction. Instead of operating on files, they operate on modules and assets.

### Browserify
### Browserify

![Browserify](images/browserify.png)

处理JavaScript模块一直是个问题。在ES2015之前，语言本身没有模块的概念。因此，当涉及到浏览器环境时，语言被困在90年代。已经提出了各种解决方案，包括[AMD]（http://requirejs.org/docs/whyamd.html）。
Dealing with JavaScript modules has always been a bit of a problem. The language itself didn't have the concept of modules till ES2015. Ergo, the language was stuck in the '90s when it comes to browser environments. Various solutions, including [AMD](http://requirejs.org/docs/whyamd.html), have been proposed.

[Browserify]（http://browserify.org/）是模块问题的一种解决方案。它允许将CommonJS模块捆绑在一起。你可以使用Gulp进行连接，你可以找到更小的转换工具，使你可以超越基本用途。例如，[watchify]（https://www.npmjs.com/package/watchify）提供了一个文件监视器，可在开发节省期间为你创建捆绑包。
[Browserify](http://browserify.org/) is one solution to the module problem. It allows CommonJS modules to be bundled together. You can hook it up with Gulp, and you can find smaller transformation tools that allow you to move beyond the basic usage. For example, [watchify](https://www.npmjs.com/package/watchify) provides a file watcher that creates bundles for you during development saving effort.

Browserify生态系统由许多小模块组成。通过这种方式，Browserify坚持Unix哲学。采用Browserify比使用webpack更加舒适，事实上，它是一个很好的替代方案。
The Browserify ecosystem is composed of a lot of small modules. In this way, Browserify adheres to the Unix philosophy. Browserify is more comfortable to adopt than webpack, and is, in fact, a good alternative to it.

T> [Splittable]（https://www.npmjs.com/package/splittable）是一个Browserify包装器，允许代码分割，支持开箱即用的ES2015，树摇动等等。 [bankai]（https://www.npmjs.com/package/bankai）是另一个需要考虑的选择。
T> [Splittable](https://www.npmjs.com/package/splittable) is a Browserify wrapper that allows code splitting, supports ES2015 out of the box, tree shaking, and more. [bankai](https://www.npmjs.com/package/bankai) is another option to consider.

T> [ify-loader]（https://www.npmjs.com/package/ify-loader）和[transform-loader]（https://www.npmjs.com/package/transform-loader）允许你使用Browserify转换为webpack。
T> [ify-loader](https://www.npmjs.com/package/ify-loader) and [transform-loader](https://www.npmjs.com/package/transform-loader) allow you to use Browserify transforms with webpack.

＃＃＃ 早午餐
### Brunch

![Brunch](images/brunch.png)

与Gulp相比，[Brunch]（http://brunch.io/）在更高的抽象层次上运行。它使用类似于webpack的声明方法。举个例子，考虑从Brunch网站改编的以下配置：
Compared to Gulp, [Brunch](http://brunch.io/) operates on a higher level of abstraction. It uses a declarative approach similar to webpack's. To give you an example, consider the following configuration adapted from the Brunch site:

```javascript
module.exports = {
  files: {
    javascripts: {
      joinTo: {
        "vendor.js": /^(?!app)/,
        "app.js": /^app/,
      },
    },
    stylesheets: {
      joinTo: "app.css",
    },
  },
  plugins: {
    babel: {
      presets: ["react", "env"],
    },
    postcss: {
      processors: [require("autoprefixer")],
    },
  },
};
```

早午餐带有“brunch new”，“brunch watch --server”和“brunch build --production”等命令。它包含很多开箱即用，可以使用插件进行扩展。
Brunch comes with commands like `brunch new`, `brunch watch --server`, and `brunch build --production`. It contains a lot out of the box and can be extended using plugins.

T> Brunch有一个实验性的[Hot Module Replacement runtime]（https://www.npmjs.com/package/hmr-brunch）。
T> There is an experimental [Hot Module Replacement runtime](https://www.npmjs.com/package/hmr-brunch) for Brunch.

###包裹
### Parcel

![Parcel](images/parcel.png)

[Parcel]（https://parceljs.org/）是一个高性能的捆绑器，与其前身不同，不需要配置。 *零配置*方法使其在社区中流行。我们的想法是你设置一个* index.html *，Parcel将根据它开始捆绑过程。它支持开箱即用的热模块更换。
[Parcel](https://parceljs.org/) is a performant bundler, that unlike its predecessors, doesn't require configuration. The *zero configuration* approach has made it popular within the community. The idea is that you set up an *index.html* and Parcel will begin the bundling process based on that. It supports Hot Module Replacement out of the box.

T>有一整套零配置捆绑包，如Parcel。这些工具包括[微网]（https://www.npmjs.com/package/microbundle），[sfo]（https://www.npmjs.com/package/sfo), [bili]（https：// www .npmjs.com / package / bili），[ovi]（https://www.npmjs.com/package/ovi）和[asbundle]（https://www.npmjs.com/package/asbundle）。
T> There's a whole category of zero configuration bundlers like Parcel. These tools include [microbundle](https://www.npmjs.com/package/microbundle), [sfo](https://www.npmjs.com/package/sfo), [bili](https://www.npmjs.com/package/bili), [ovi](https://www.npmjs.com/package/ovi), and [asbundle](https://www.npmjs.com/package/asbundle).

### Webpack
### Webpack

![webpack](images/webpack.png)

你可以说[webpack]（https://webpack.js.org/）采用比Browserify更统一的方法。虽然Browserify包含多个小工具，但webpack附带的核心提供了大量开箱即用的功能。
You could say [webpack](https://webpack.js.org/) takes a more unified approach than Browserify. Whereas Browserify consists of multiple small tools, webpack comes with a core that provides a lot of functionality out of the box.

Webpack核心可以使用特定的* loaders *和* plugins *进行扩展。它可以控制*解析模块的方式，从而可以调整你的构建以匹配特定情况和解决方案无法正常使用的软件包。
Webpack core can be extended using specific *loaders* and *plugins*. It gives control over how it *resolves* the modules, making it possible to adapt your build to match specific situations and workaround packages that don't work correctly out of the box.

与其他工具相比，webpack具有最初的复杂性，但它通过其广泛的功能集弥补了这一点。这是一个需要耐心的高级工具。但是一旦你理解了它背后的基本思想，webpack就会变得强大。
Compared to the other tools, webpack comes with initial complexity, but it makes up for this through its broad feature set. It's an advanced tool that requires patience. But once you understand the basic ideas behind it, webpack becomes powerful.

{pagebreak}

##其他选项
## Other Options

你可以找到更多替代方案，如下所示：
You can find more alternatives as listed below:

* [pundle]（https://www.npmjs.com/package/pundle）宣称自己是下一代捆绑商，并特别注意其性能。
* [pundle](https://www.npmjs.com/package/pundle) advertises itself as a next-generation bundler and notes particularly its performance.
* [Rollup]（https://www.npmjs.com/package/rollup）专注于捆绑ES2015代码。 *树摇*是其卖点之一，它也支持代码分割。你可以通过[rollup-loader]（https://www.npmjs.com/package/rollup-loader）将Rollup与webpack一起使用。
* [Rollup](https://www.npmjs.com/package/rollup) focuses on bundling ES2015 code. *Tree shaking* is one of its selling points and it supports code splitting as well. You can use Rollup with webpack through [rollup-loader](https://www.npmjs.com/package/rollup-loader).
* [AssetGraph]（https://www.npmjs.com/package/assetgraph）采用完全不同的方法，并建立在HTML语义之上，使其成为[超级分析]的理想选择（https://www.npmjs.com/包/超链接）或[结构分析]（https://www.npmjs.com/package/assetviz）。 [webpack-assetgraph-plugin]（https://www.npmjs.com/package/webpack-assetgraph-plugin）将webpack和AssetGraph连接在一起。
* [AssetGraph](https://www.npmjs.com/package/assetgraph) takes an entirely different approach and builds on top of HTML semantics making it ideal for [hyperlink analysis](https://www.npmjs.com/package/hyperlink) or [structural analysis](https://www.npmjs.com/package/assetviz). [webpack-assetgraph-plugin](https://www.npmjs.com/package/webpack-assetgraph-plugin) bridges webpack and AssetGraph together.
* [FuseBox]（https://www.npmjs.com/package/fuse-box）是一个专注于速度的捆绑商。它采用零配置方法，旨在开箱即用。
* [FuseBox](https://www.npmjs.com/package/fuse-box) is a bundler focusing on speed. It uses a zero-configuration approach and aims to be usable out of the box.
* [StealJS]（https://stealjs.com/）是一个依赖性加载器和一个构建工具，它专注于性能和易用性。
* [StealJS](https://stealjs.com/) is a dependency loader and a build tool which has focused on performance and ease of use.
* [Flipbox]（https://www.npmjs.com/package/flipbox）将许多捆绑包包装在统一界面后面。
* [Flipbox](https://www.npmjs.com/package/flipbox) wraps many bundlers behind a uniform interface.
* [Blendid]（https://www.npmjs.com/package/blendid）是Gulp和捆绑商的混合体，形成资产管道。
* [Blendid](https://www.npmjs.com/package/blendid) is a blend of Gulp and bundlers to form an asset pipeline.

{pagebreak}

## 总结


从历史上看，有很多用于JavaScript的构建工具。每个人都试图以其方式解决特定问题。标准已经开始迎头赶上，并且围绕基本语义需要更少的努力。相反，工具可以在更高层次上竞争并推动更好的用户体验。通常，你可以一起使用几个单独的解决方案。
Historically there have been a lot of build tools for JavaScript. Each has tried to solve a specific problem in its way. The standards have begun to catch up, and less effort is required around basic semantics. Instead, tools can compete on a higher level and push towards better user experience. Often you can use a couple of separate solutions together.

回顾一下：


* **任务跑步者**和**捆绑者**解决不同的问题。你可以使用两者获得类似的结果，但通常最好将它们一起使用以相互补充。
* **Task runners** and **bundlers** solve different problems. You can achieve similar results with both, but often it's best to use them together to complement each other.
*旧的工具，例如Make或RequireJS，即使它们在Web开发中不像以前那样流行，它们仍然具有影响力。
* Older tools, such as Make or RequireJS, still have influence even if they aren't as popular in web development as they once were.
* Browserify或webpack等捆绑包解决了一个重要问题，可帮助你管理复杂的Web应用程序。
* Bundlers like Browserify or webpack solve an important problem and help you to manage complex web applications.
*新兴技术从不同角度解决问题。有时它们构建在其他工具之上，有时它们可​​以一起使用。
* Emerging technologies approach the problem from different angles. Sometimes they build on top of other tools, and at times they can be used together.

