＃ 入门
# Getting Started

在开始之前，请确保您使用的是[Node]（http://nodejs.org/）的最新版本。您应该至少使用最新的LTS（长期支持）版本。本书的配置考虑了LTS Node功能。你的终端应该有`node`和`npm`命令。 [Yarn]（https://yarnpkg.com/）是npm的一个很好的替代品，也适用于本教程。
Before getting started, make sure you are using a recent version of [Node](http://nodejs.org/). You should use at least the most current LTS (long-term support) version. The configuration of the book has been written with the LTS Node features in mind. You should have `node` and `npm` commands available at your terminal. [Yarn](https://yarnpkg.com/) is a good alternative to npm and works for the tutorial as well.

通过使用[Docker]（https://www.docker.com/），[Vagrant]（https://www.vagrantup.com/）或[nvm]等解决方案，可以获得更加可控的环境（ https://www.npmjs.com/package/nvm）。 Vagrant因依赖虚拟机而受到性能损失。 Vagrant在团队中很有价值：每个开发人员都可以拥有通常接近生产的相同环境。
It's possible to get a more controlled environment by using a solution such as [Docker](https://www.docker.com/), [Vagrant](https://www.vagrantup.com/) or [nvm](https://www.npmjs.com/package/nvm). Vagrant comes with a performance penalty as it relies on a virtual machine. Vagrant is valuable in a team: each developer can have the same environment that is usually close to production.

T>完成的配置可在[GitHub]（https://github.com/survivejs-demos/webpack-demo）上找到。
T> The completed configuration is available at [GitHub](https://github.com/survivejs-demos/webpack-demo).

{pagebreak}

##设置项目
## Setting Up the Project

要获得起点，您应该为项目创建一个目录并在那里设置* package.json *。 npm使用它来管理项目依赖项。以下是基本命令：
To get a starting point, you should create a directory for the project and set up a *package.json* there. npm uses that to manage project dependencies. Here are the basic commands:

```bash
mkdir webpack-demo
cd webpack-demo
npm init -y # -y generates *package.json*, skip for more control
```

您可以手动调整生成的* package.json *以对其进行进一步更改，即使部分操作会自动为您修改文件。官方文档更详细地解释了[package.json选项]（https://docs.npmjs.com/files/package.json）。
You can tweak the generated *package.json* manually to make further changes to it even though a part of the operations modify the file automatically for you. The official documentation explains [package.json options](https://docs.npmjs.com/files/package.json) in more detail.

T>您可以在*〜/ .npmrc *中设置那些`npm init`默认值。
T> You can set those `npm init` defaults at *~/.npmrc*.

T>这是使用[Git]（https://git-scm.com/）设置版本控制的绝佳机会。您可以为每个步骤创建一个提交和每章标记，因此如果您愿意，可以更轻松地来回移动。
T> This is an excellent chance to set up version control using [Git](https://git-scm.com/). You can create a commit per step and tag per chapter, so it's easier to move back and forth if you want.

T>本书的例子已经使用[Prettier]（https://www.npmjs.com/package/prettier）格式化了“trailingComma”：“es5”，`和`“printWidth”：68`选项可以使差异清理并适合页面。
T> The book examples have been formatted using [Prettier](https://www.npmjs.com/package/prettier) with `"trailingComma": "es5",` and `"printWidth": 68` options enabled to make the diffs clean and fit the page.

##安装Webpack
## Installing Webpack

即使webpack可以全局安装（`npm install webpack -g`），最好将它作为项目的依赖项来维护，以避免出现问题，因为您可以控制正在运行的确切版本。该方法也适用于**持续集成**（CI）设置。 CI系统可以安装本地依赖项，使用它们编译项目，然后将结果推送到服务器。
Even though webpack can be installed globally (`npm install webpack -g`), it's a good idea to maintain it as a dependency of your project to avoid issues, as then you have control over the exact version you are running. The approach works nicely in **Continuous Integration** (CI) setups as well. A CI system can install your local dependencies, compile your project using them, and then push the result to a server.

要将webpack添加到项目中，请执行：
To add webpack to the project, execute:

```bash
npm install webpack webpack-cli --save-dev # -D to type less
```

你应该在你的* package.json *`devDependencies`部分看到webpack。除了在* node_modules *目录下本地安装软件包之外，npm还会为可执行文件生成一个条目。
You should see webpack at your *package.json* `devDependencies` section after this. In addition to installing the package locally below the *node_modules* directory, npm also generates an entry for the executable.

T>您可以使用`--save`和`--save-dev`来分离应用程序和开发依赖项。前者安装并写入* package.json *`dependencies`字段，而后者则写入`devDependencies`。
T> You can use `--save` and `--save-dev` to separate application and development dependencies. The former installs and writes to *package.json* `dependencies` field whereas the latter writes to `devDependencies` instead.

T> [webpack-cli]（https://www.npmjs.com/package/webpack-cli）附带了其他功能，包括`init`和`migrate`命令，允许您快速创建新的webpack配置并从旧版本更新版本。
T> [webpack-cli](https://www.npmjs.com/package/webpack-cli) comes with additional functionality including `init` and `migrate` commands that allow you to create new webpack configuration fast and update from an older version to a newer one.

##执行Webpack
## Executing Webpack

您可以使用`npm bin`显示可执行文件的确切路径。最有可能的是它指向*。/ node_modules / .bin *。尝试使用`node_modules / .bin / webpack`或类似命令从终端通过终端运行webpack。
You can display the exact path of the executables using `npm bin`. Most likely it points at *./node_modules/.bin*. Try running webpack from there through the terminal using `node_modules/.bin/webpack` or a similar command.

运行后，您应该看到一个版本，一个指向命令行界面指南的链接以及一个广泛的选项列表。大多数都没有在这个项目中使用，但是很高兴知道这个工具包含了其他功能。
After running, you should see a version, a link to the command line interface guide and an extensive list of options. Most aren't used in this project, but it's good to know that this tool is packed with functionality if nothing else.

```bash
$ node_modules/.bin/webpack
Hash: 6736210d3313db05db58
Version: webpack 4.1.1
Time: 88ms
Built at: 3/16/2018 3:35:07 PM

WARNING in configuration
The 'mode' option has not been set. Set 'mode' option to 'development' or 'production' to enable defaults for this environment.

ERROR in Entry module not found: Error: Can't resolve './src' in '.../webpack-demo'
```

输出告诉webpack找不到要编译的源代码。它还缺少一个`mode`参数来应用开发或生产特定的默认值。
The output tells that webpack cannot find the source to compile. It's also missing a `mode` parameter to apply development or production specific defaults.

为了快速了解webpack输出，我们应该解决这两个问题：
To get a quick idea of webpack output, we should fix both:

1.设置* src / index.js *，使其包含`console.log（“Hello world”）;`。
1. Set up *src/index.js* so that it contains `console.log("Hello world");`.
2.执行`node_modules / .bin / webpack --mode development`。 Webpack将按节点约定发现源文件。
2. Execute `node_modules/.bin/webpack --mode development`. Webpack will discover the source file by Node convention.
3.检查* dist / main.js *。您应该看到开始执行代码的webpack引导代码。在引导程序下面，你应该找到熟悉的东西。
3. Examine *dist/main.js*. You should see webpack bootstrap code that begins executing the code. Below the bootstrap, you should find something familiar.

T>尝试`--mode production`并比较输出。
T> Try also `--mode production` and compare the output.

{pagebreak}

##设置资产
## Setting Up Assets

为了使构建更加复杂，我们可以向项目添加另一个模块并开始开发一个小应用程序：
To make the build more involved, we can add another module to the project and start developing a small application:

** SRC / component.js **
**src/component.js**

```javascript
export default (text = "Hello world") => {
  const element = document.createElement("div");

  element.innerHTML = text;

  return element;
};
```

我们还必须修改原始文件以导入新文件并通过DOM呈现应用程序：
We also have to modify the original file to import the new file and render the application through the DOM:

** SRC / index.js **
**src/index.js**

```javascript
import component from "./component";

document.body.appendChild(component());
```

构建后检查输出（`node_modules / .bin / webpack --mode development`）。你应该看到webpack写入`dist`目录的bundle中的两个模块。
Examine the output after building (`node_modules/.bin/webpack --mode development`). You should see both modules in the bundle that webpack wrote to the `dist` directory.

要使输出更清晰，请将`--devtool false`参数传递给webpack。默认情况下，Webpack将生成基于“eval”的源映射，这样做会禁用该行为。有关详细信息，请参阅*源地图*章节。
To make the output clearer to examine, pass `--devtool false` parameter to webpack. Webpack will generate `eval` based source maps by default and doing this will disable the behavior. See the *Source Maps* chapter for more information.

但问题仍然存在。我们如何在浏览器中测试应用程序？
One problem remains, though. How can we test the application in the browser?

##配置* html-webpack-plugin *
## Configuring *html-webpack-plugin*

可以通过编写指向生成的文件的* index.html *文件来解决该问题。我们可以使用插件和webpack配置来完成此操作，而不是单独执行此操作。
The problem can be solved by writing an *index.html* file that points to the generated file. Instead of doing that on our own, we can use a plugin and webpack configuration to do this.

要开始使用，请安装* html-webpack-plugin *：
To get started, install *html-webpack-plugin*:

```bash
npm install html-webpack-plugin --save-dev
```

要将插件与webpack连接，请按如下所示设置配置：
To connect the plugin with webpack, set up configuration as below:

** ** webpack.config.js
**webpack.config.js**

```javascript
const HtmlWebpackPlugin = require("html-webpack-plugin");

module.exports = {
  plugins: [
    new HtmlWebpackPlugin({
      title: "Webpack demo",
    }),
  ],
};
```

现在配置已完成，您应该尝试以下操作：
Now that the configuration is done, you should try the following:

1.使用`node_modules / .bin / webpack --mode production`构建项目。你也可以试试`development`模式。
1. Build the project using `node_modules/.bin/webpack --mode production`. You can try the `development` mode too.
2.使用`cd dist`输入构建目录。
2. Enter the build directory using `cd dist`.
3.使用`serve`（`npm i serve -g`）或类似命令运行服务器。
3. Run the server using `serve` (`npm i serve -g`) or a similar command.
4.通过Web浏览器检查结果。你应该看到熟悉的东西。
4. Examine the result through a web browser. You should see something familiar there.

![Hello world](images/hello_01.png)

T> **尾随逗号**有意用于本书示例中，因为它为代码示例提供了更清晰的差异。
T> **Trailing commas** are used in the book examples on purpose as it gives cleaner diffs for the code examples.

##检查输出
## Examining the Output

如果你执行`node_modules / .bin / webpack --mode production`，你应该看到输出：
If you execute `node_modules/.bin/webpack --mode production`, you should see output:

```bash
Hash: aafe36ba210b0fbb7073
Version: webpack 4.1.1
Time: 338ms
Built at: 3/16/2018 3:40:14 PM
     Asset       Size  Chunks             Chunk Names
   main.js  679 bytes       0  [emitted]  main
index.html  181 bytes          [emitted]
Entrypoint main = main.js
   [0] ./src/index.js + 1 modules 219 bytes {0} [built]
       | ./src/index.js 77 bytes [built]
       | ./src/component.js 142 bytes [built]
Child html-webpack-plugin for "index.html":
     1 asset
    Entrypoint undefined = index.html
       [0] (webpack)/buildin/module.js 519 bytes {0} [built]
       [1] (webpack)/buildin/global.js 509 bytes {0} [built]
        + 2 hidden modules
```

输出说明了很多：
The output tells a lot:

*`哈希：aafe36ba210b0fbb7073`  - 构建的哈希。您可以使用它来通过`[hash]`占位符使资产无效。 Hashing将在* Add Hashes to Filenames *章节中详细讨论。
* `Hash: aafe36ba210b0fbb7073` - The hash of the build. You can use this to invalidate assets through `[hash]` placeholder. Hashing is discussed in detail in the *Adding Hashes to Filenames* chapter.
*`版本：webpack 4.1.1`  -  Webpack版本。
* `Version: webpack 4.1.1` - Webpack version.
*`时间：338ms`  - 执行构建所花费的时间。
* `Time: 338ms` - Time it took to execute the build.
*`main.js 679字节0 [emit] main`  - 生成的资产的名称，大小，与其相关的**块的ID，状态信息，告知它是如何生成的，块的名称。
* `main.js  679 bytes       0  [emitted]  main` - Name of the generated asset, size, the IDs of the **chunks** into which it's related, status information telling how it was generated, the name of the chunk.
*`index.html 181字节[emit]` - 进程发出的另一个生成的资产。
* `index.html  181 bytes          [emitted]` - Another generated asset that was emitted by the process.
*`[0] ./src/index.js + 1个模块219个字节{0} [built]` - 条目资产的ID，名称，大小，条目块ID，生成方式。
* `[0] ./src/index.js + 1 modules 219 bytes {0} [built]` - The ID of the entry asset, name, size, entry chunk ID, the way it was generated.
*`index html-webpack-plugin for“index.html”：` - 这是与插件相关的输出。在这种情况下，* html-webpack-plugin *会自行创建此输出。
* `Child html-webpack-plugin for "index.html":` - This is plugin-related output. In this case *html-webpack-plugin* is creating this output on its own.

检查`dist /`目录下面的输出。如果仔细观察，可以在源代码中看到相同的ID。
Examine the output below the `dist/` directory. If you look closely, you can see the same IDs within the source.

T>除配置对象外，webpack还接受一系列配置。你也可以返回一个`Promise`并最终将`resolve`返回给一个配置。
T> In addition to a configuration object, webpack accepts an array of configurations. You can also return a `Promise` and eventually `resolve` to a configuration for example.

T>如果您想要替代* html-webpack-plugin *，请参阅[mini-html-webpack-plugin]（https://www.npmjs.com/package/mini-html-webpack-plugin）。它做得少，但理解起来也更简单。
T> If you want a light alternative to *html-webpack-plugin*, see [mini-html-webpack-plugin](https://www.npmjs.com/package/mini-html-webpack-plugin). It does less but it's also simpler to understand.

{pagebreak}

##添加构建快捷方式
## Adding a Build Shortcut

鉴于执行`node_modules / .bin / webpack`是详细的，你应该对它做点什么。调整* package.json *以运行如下任务：
Given executing `node_modules/.bin/webpack` is verbose, you should do something about it. Adjust *package.json* to run tasks as below:

** **的package.json
**package.json**

```json
"scripts": {
  "build": "webpack --mode production"
},
```

运行`npm run build`以查看与之前相同的输出。 npm将* node_modules / .bin *临时添加到启用此功能的路径中。因此，您不必编写“build”：“node_modules / .bin / webpack”，而是可以执行“build”：“webpack”`。
Run `npm run build` to see the same output as before. npm adds *node_modules/.bin* temporarily to the path enabling this. As a result, rather than having to write `"build": "node_modules/.bin/webpack"`, you can do `"build": "webpack"`.

您可以通过* npm run *执行此类脚本，并且可以在项目中的任何位置使用* npm run *。如果按原样运行该命令，它将为您提供可用脚本的列表。
You can execute this kind of scripts through *npm run* and you can use *npm run* anywhere within your project. If you run the command as is, it gives you the listing of available scripts.

T>有像* npm start *和* npm test *这样的快捷方式。你可以在没有* npm run *的情况下直接运行它们，尽管它也可以。对于那些赶时间的人，你可以使用* npm t *来运行你的测试。
T> There are shortcuts like *npm start* and *npm test*. You can run these directly without *npm run* although that works too. For those in a hurry, you can use *npm t* to run your tests.

T>要更进一步，请使用终端配置中的`alias`命令设置系统级别名。例如，您可以将`nrb`映射到`npm run build`。
T> To go one step further, set up system level aliases using the `alias` command in your terminal configuration. You could map `nrb` to `npm run build` for instance.

{pagebreak}

##`HtmlWebpackPlugin`扩展
## `HtmlWebpackPlugin` Extensions

虽然您可以用自己的模板替换`HtmlWebpackPlugin`模板，但有预制的模板，如[html-webpack-template]（https://www.npmjs.com/package/html-webpack-template）或[html-webpack-template -pug]（https://www.npmjs.com/package/html-webpack-template-pug）。
Although you can replace `HtmlWebpackPlugin` template with your own, there are premade ones like [html-webpack-template](https://www.npmjs.com/package/html-webpack-template) or [html-webpack-template-pug](https://www.npmjs.com/package/html-webpack-template-pug).

还有一些特定的插件可以扩展`HtmlWebpackPlugin`的功能：
There are also specific plugins that extend `HtmlWebpackPlugin`'s functionality:

* [favicons-webpack-plugin]（https://www.npmjs.com/package/favicons-webpack-plugin）能够生成favicon。
* [favicons-webpack-plugin](https://www.npmjs.com/package/favicons-webpack-plugin) is able to generate favicons.
* [script-ext-html-webpack-plugin]（https://www.npmjs.com/package/script-ext-html-webpack-plugin）使您可以更好地控制脚本标记，并允许您进一步调整脚本加载。
* [script-ext-html-webpack-plugin](https://www.npmjs.com/package/script-ext-html-webpack-plugin) gives you more control over script tags and allows you to tune script loading further.
* [style-ext-html-webpack-plugin]（https://www.npmjs.com/package/style-ext-html-webpack-plugin）将CSS引用转换为内联CSS。作为初始有效负载的一部分，该技术可用于快速向客户端提供关键CSS。
* [style-ext-html-webpack-plugin](https://www.npmjs.com/package/style-ext-html-webpack-plugin) converts CSS references to inlined CSS. The technique can be used to serve critical CSS to the client fast as a part of the initial payload.
* [resource-hints-webpack-plugin]（https://www.npmjs.com/package/resource-hints-webpack-plugin）添加[资源提示]（https://www.w3.org/TR/resource -hints /）到你的HTML文件，以加快加载时间。
* [resource-hints-webpack-plugin](https://www.npmjs.com/package/resource-hints-webpack-plugin) adds [resource hints](https://www.w3.org/TR/resource-hints/) to your HTML files to speed up loading time.
* [preload-webpack-plugin]（https://www.npmjs.com/package/preload-webpack-plugin）为脚本启用`rel = preload`功能，并有助于延迟加载，并且它与在本书* Building *的一部分。
* [preload-webpack-plugin](https://www.npmjs.com/package/preload-webpack-plugin) enables `rel=preload` capabilities for scripts and helps with lazy loading, and it combines well with techniques discussed in the *Building* part of this book.
* [webpack-cdn-plugin]（https://www.npmjs.com/package/webpack-cdn-plugin）允许您指定通过内容交付网络（CDN）加载的依赖项。这种常用技术用于加速流行库的加载。
* [webpack-cdn-plugin](https://www.npmjs.com/package/webpack-cdn-plugin) allows you to specify which dependencies to load through a Content Delivery Network (CDN). This common technique is used for speeding up loading of popular libraries.
* [dynamic-cdn-webpack-plugin]（https://www.npmjs.com/package/dynamic-cdn-webpack-plugin）实现了类似的结果。
* [dynamic-cdn-webpack-plugin](https://www.npmjs.com/package/dynamic-cdn-webpack-plugin) achieves a similar result.

{pagebreak}

##结论
## Conclusion

即使你已经设法启动并运行webpack，但它还没有那么多。反对它的发展将是痛苦的。每次要查看应用程序时，都必须使用`npm run build`手动构建它，然后刷新浏览器。这就是webpack更高级功能的用武之地。
Even though you have managed to get webpack up and running, it does not do that much yet. Developing against it would be painful. Each time you wanted to check out the application, you would have to build it manually using `npm run build` and then refresh the browser. That's where webpack's more advanced features come in.

回顾一下：
To recap:

*在全局安装的webpack上使用本地安装的webpack版本是个好主意。这样您就可以确定您使用的是哪个版本。本地依赖项也适用于持续集成环境。
* It's a good idea to use a locally installed version of webpack over a globally installed one. This way you can be sure of what version you are using. The local dependency also works in a Continuous Integration environment.
* Webpack通过* webpack-cli *包提供命令行界面。即使没有配置，您也可以使用它，但任何高级用法都需要配置。
* Webpack provides a command line interface through the *webpack-cli* package. You can use it even without configuration, but any advanced usage requires configuration.
*要编写更复杂的设置，您很可能必须编写单独的* webpack.config.js *文件。
* To write more complicated setups, you most likely have to write a separate *webpack.config.js* file.
*`HtmlWebpackPlugin`可用于生成应用程序的HTML入口点。在* Multiple Pages *一章中，您将了解如何使用它生成多个单独的页面。
* `HtmlWebpackPlugin` can be used to generate an HTML entry point to your application. In the *Multiple Pages* chapter you will see how to generate multiple separate pages using it.
*使用npm * package.json *脚本来管理webpack很方便。您可以将它用作轻型任务运行器并使用webpack之外的系统功能。
* It's handy to use npm *package.json* scripts to manage webpack. You can use it as a light task runner and use system features outside of webpack.

在下一章中，您将学习如何通过启用自动浏览器刷新来改善开发人员体验。
In the next chapter, you will learn how to improve the developer experience by enabling automatic browser refresh.

