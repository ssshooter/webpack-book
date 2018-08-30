# Build Analysis

分析构建统计信息可以更好地理解 webpack。可视化 webpack 输出可帮助你了解 bundle 的构成。

## 配置 Webpack

要获得合适的输出，你需要对配置进行一些调整。至少应该设置 `--json` flag，并将输出保存到文件中，如下所示：

**package.json**

```json
"scripts": {
leanpub-start-insert
  "build:stats": "webpack --env production --json > stats.json",
leanpub-end-insert
  ...
},
```

以上是你需要的基本设置。现在执行 `npm run build:stats`。过了一会儿，你应该在你的项目根目录找到 *stats.json*。可以通过各种工具推送此文件，以更好地了解正在发生的事情。
The above is the basic setup you need, regardless of your webpack configuration. Execute `npm run build:stats` now. After a while you should find *stats.json* at your project root. This file can be pushed through a variety of tools to understand better what's going on.

你还可以使用以下 flag：

* `--profile` 捕获与时间相关的信息。该选项非必要，但加上了也不错。
* `--progress` 显示 webpack 在构建的不同阶段花了多长时间。

T> 要理解为什么 webpack 在处理时把某些模块打包到构建结果，请使用 [whybundled](https://www.npmjs.com/package/whybundled) 或 [webpack-why](https://www.npmjs.com/package/webpack-why)。 `--display-reasons` flag 也提供了更多信息。示例：`npm run build -- --display-reasons`。

W>鉴于你在当前设置中捎带生产目标;这个过程清理构建目录！如果要避免这种情况，请设置一个不清理的单独目标。
W> Given you piggyback on the production target in the current setup; this process cleans the build directory! If you want to avoid that, set up a separate destination where you don't clean.

### Node API

可以通过Node捕获统计信息。由于统计信息可能包含错误，因此分开处理该情况是个好主意：
Stats can be captured through Node. Since stats can contain errors, so it's a good idea to handle that case separately:

```javascript
const webpack = require("webpack");
const config = require("./webpack.config.js")("production");

webpack(config, (err, stats) => {
  if (err) {
    return console.error(err);
  }

  if (stats.hasErrors()) {
    return console.error(stats.toString("errors-only"));
  }

  console.log(stats);
});
```

如果你想对统计数据进行进一步处理，这种技术可能很有价值，尽管其他解决方案通常已足够。
This technique can be valuable if you want to do further processing on stats although often the other solutions are enough.

T>如果你想从`stats`输出JSON，请使用`stats.toJson（）`。要获得* verbose *输出，请使用`stats.toJson（“verbose”）`。它遵循webpack支持的所有stat选项。
T> If you want JSON output from `stats`, use `stats.toJson()`. To get *verbose* output, use `stats.toJson("verbose")`. It follows all stat options webpack supports.

T>要模仿`--json`标志，请使用`console.log（JSON.stringify（stats.toJson（），null，2））;`。输出格式化为可读。
T> To mimic the `--json` flag, use `console.log(JSON.stringify(stats.toJson(), null, 2));`. The output is formatted to be readable.

### `StatsWebpackPlugin` 和 `WebpackStatsPlugin`

如果你想通过插件管理统计数据，请查看[stats-webpack-plugin]（https://www.npmjs.com/package/stats-webpack-plugin）。它使你可以更多地控制输出。你可以使用它从输出中排除特定的依赖项。
If you want to manage stats through a plugin, check out [stats-webpack-plugin](https://www.npmjs.com/package/stats-webpack-plugin). It gives you a bit more control over the output. You can use it to exclude specific dependencies from the output.

[webpack-stats-plugin]（https://www.npmjs.com/package/webpack-stats-plugin）是另一种选择。它允许你在输出数据之前转换数据。
[webpack-stats-plugin](https://www.npmjs.com/package/webpack-stats-plugin) is another option. It allows you to transform the data before outputting it.

##启用效果预算
## Enabling a Performance Budget

Webpack允许你定义**性能预算**。这个想法是它给出了你必须遵循的构建大小约束。默认情况下禁用该功能，并且计算包括提取的块到条目计算。如果未满足预算并且已将其配置为发出错误，则会终止整个构建。
Webpack allows you to define a **performance budget**. The idea is that it gives your build size constraint which it has to follow. The feature is disabled by default and the calculation includes extracted chunks to entry calculation. If a budget isn't met and it has been configured to emit an error, it would terminate the entire build.

{pagebreak}

要将该功能集成到项目中，请调整配置：
To integrate the feature into the project, adjust the configuration:

**webpack.config.js**

```javascript
const productionConfig = merge([
leanpub-start-insert
  {
    performance: {
      hints: "warning", // "error" or false are valid too
      maxEntrypointSize: 50000, // in bytes, default 250k
      maxAssetSize: 450000, // in bytes
    },
  },
leanpub-end-insert
  ...
]);
```

在实践中，你希望保持下限。目前这些足以进行此演示。如果你现在构建（`npm run build`），你应该看到一个警告：
In practice, you want to maintain lower limits. The current ones are enough for this demonstration. If you build now (`npm run build`), you should see a warning:

```bash
WARNING in entrypoint size limit: The following entrypoint(s) combined asset size exceeds the recommended limit (48.8 KiB). This can impact web performance.
Entrypoints:
  main (103 KiB)
      manifest.3fd9a1eb.js
      manifest.d41d8cd9.css
      vendor.0a4df2ff.js
      vendor.3dd53418.css
      main.9043ef51.js
      main.d5d711b1.css
```

你可以增加限制或删除配置以消除警告。一个有吸引力的选择是用*消费包*章节中讨论的更轻的替代品替换React。
You can increase the limit or remove the configuration to get rid of the warning. An attractive option would be to replace React with a lighter alternative as discussed in the *Consuming Packages* chapter.

{pagebreak}

## 分析工具

即使查看文件本身可以让你了解正在发生的事情，但通常最好使用特定的工具。考虑以下。
Even though having a look at the file itself gives you an idea of what's going on, often it's preferable to use a particular tool for that. Consider the following.

### 官方分析工具

![The Official Analyse Tool](images/analyse.png)

[官方分析工具](https://github.com/webpack/analyse) 为你提供建议，并对你的应用程序的依赖图有一个很好的了解。它也可以在本地运行。
[The official analyse tool](https://github.com/webpack/analyse) gives you recommendations and a good idea of your application's dependency graph. It can be run locally as well.

{pagebreak}

### Webpack Visualizer

![Webpack Visualizer](images/webpack-visualizer.png)

[Webpack Visualizer]（https://chrisbateman.github.io/webpack-visualizer/）提供了一个饼图，显示了你的包组合，可以了解哪些依赖关系有助于整体结果的大小。
[Webpack Visualizer](https://chrisbateman.github.io/webpack-visualizer/) provides a pie chart showing your bundle composition allowing to understand which dependencies contribute to the size of the overall result.

### `DuplicatePackageCheckerPlugin`

[duplicate-package-checker-webpack-plugin]（https://www.npmjs.com/package/duplicate-package-checker-webpack-plugin）会在你的构建中多次发现单个包时发出警告。否则这种情况很难发现。
[duplicate-package-checker-webpack-plugin](https://www.npmjs.com/package/duplicate-package-checker-webpack-plugin) warns you if it finds single package multiple times in your build. This situation can be hard to spot otherwise.

### Webpack Chart

![Webpack Chart](images/webpack-chart.png)

[Webpack Chart](https://alexkuz.github.io/webpack-chart/) 是另一个类似的可视化工具。

### webpack-unused

[webpack-unused](https://www.npmjs.com/package/webpack-unused) 打印出未使用的文件，可用于了解哪些资源不再使用，可以从项目中删除。

### Stellar Webpack

![Stellar Webpack](images/stellar-webpack.jpg)

[Stellar Webpack](https://alexkuz.github.io/stellar-webpack/) webpack 可视化 + 星空模拟器。

### webpack-bundle-tracker

[webpack-bundle-tracker](https://www.npmjs.com/package/webpack-bundle-tracker) 可以在webpack编译时捕获数据。它使用JSON来实现此目的。
[webpack-bundle-tracker](https://www.npmjs.com/package/webpack-bundle-tracker) can capture data while webpack is compiling. It uses JSON for this purpose.

### webpack-bundle-analyzer

![webpack-bundle-analyzer](images/webpack-bundle-analyzer.jpg)

[webpack-bundle-analyzer]（https://www.npmjs.com/package/webpack-bundle-analyzer）提供可缩放的树形图。
[webpack-bundle-analyzer](https://www.npmjs.com/package/webpack-bundle-analyzer) provides a zoomable treemap.

{pagebreak}

### webpack-bundle-size-analyzer

[webpack-bundle-size-analyzer](https://www.npmjs.com/package/webpack-bundle-size-analyzer) 提供了文字 bundle 大小分析。

```bash
$ webpack-bundle-size-analyzer stats.json
react: 93.99 KB (74.9%)
purecss: 15.56 KB (12.4%)
style-loader: 6.99 KB (5.57%)
fbjs: 5.02 KB (4.00%)
object-assign: 1.95 KB (1.55%)
css-loader: 1.47 KB (1.17%)
<self>: 572 B (0.445%)
```

### inspectpack

[inspectpack]（https://www.npmjs.com/package/inspectpack）可用于确定要改进的特定代码位置。以下示例执行重复分析：
[inspectpack](https://www.npmjs.com/package/inspectpack) can be used for figuring out specific places of code to improve. The example below performs duplication analysis:

```bash
$ inspectpack --action=duplicates --bundle=bundle.js
## Summary

* Bundle:
    * Path:                /PATH/TO/bundle.js
    * Bytes (min):         1678533
* Missed Duplicates:
    * Num Unique Files:    116
    * Num Extra Files:     131
    * Extra Bytes (min):   253955
    * Pct of Bundle Size:  15 %
```

{pagebreak}

### Jarvis

[Jarvis]（https://www.npmjs.com/package/webpack-jarvis）是一个用户界面，旨在显示与你的webpack构建相关的所有信息。例如，它显示项目中可跳过树的模块的数量以及资产对不同连接类型的执行情况。
[Jarvis](https://www.npmjs.com/package/webpack-jarvis) is a user interface that has been designed to show all information relevant to your webpack build. For example, it shows the amount of treeshakeable modules in the project and how well your assets perform against different connection types.

### webpack-runtime-analyzer
### webpack-runtime-analyzer

[webpack-runtime-analyzer]（https://www.npmjs.com/package/webpack-runtime-analyzer）提供了对webpack包的实时分析。你可以通过用户界面，包大小和模块详细信息查看多种格式的包组合。它将上述许多工具的功能组合到一个工具中。
[webpack-runtime-analyzer](https://www.npmjs.com/package/webpack-runtime-analyzer) gives real-time analysis over webpack bundles. You can see bundle composition in multiple formats through the user interface, bundle sizes, and module details. It combines features of many tools above into a single one.

### Webpack Monitor
### Webpack Monitor

[Webpack Monitor]（http://webpackmonitor.com/）是另一个类似的工具，强调清晰的用户界面。它能够提供有关改进构建的建议。
[Webpack Monitor](http://webpackmonitor.com/) is another similar tool with an emphasis on a clear user interface. It's able to provide recommendations on what to improve the build.

### webpack-deps-tree
### webpack-deps-tree

[webpack-deps-tree](https://restrry.github.io/webpack-deps-tree/static/) 显示webpack模块图。使用它，你可以了解捆绑包的模块是如何相互关联的。
[webpack-deps-tree](https://restrry.github.io/webpack-deps-tree/static/) displays webpack module graph. Using it you can understand how modules of your bundles are related to each other.

{pagebreak}

## 重复分析
## Duplication Analysis

除了inspectpack之外，还有其他工具可用于计算重复项：
In addition to inspectpack, there are other tools for figuring out duplicates:

* [bundle-duplicates-plugin]（https://www.npmjs.com/package/bundle-duplicates-plugin）在功能级别上运行。
* [bundle-duplicates-plugin](https://www.npmjs.com/package/bundle-duplicates-plugin) operates on a function level.
* [find-duplicate-dependencies]（https://www.npmjs.com/package/find-duplicate-dependencies）在npm包级别上实现相同。
* [find-duplicate-dependencies](https://www.npmjs.com/package/find-duplicate-dependencies) achieves the same on an npm package level.
* [depcheck]（https://www.npmjs.com/package/depcheck）进一步发出警告，如果项目中缺少冗余依赖项或依赖项。
* [depcheck](https://www.npmjs.com/package/depcheck) goes further and warns if there are redundant dependencies or dependencies missing from the project.
* [bundle-buddy]（https://www.npmjs.com/package/bundle-buddy）可以在包中找到重复项，同时提供用户界面来调整webpack代码拆分行为。 [bundle-buddy-webpack-plugin]（https://www.npmjs.com/package/bundle-buddy-webpack-plugin）使用起来更简单。
* [bundle-buddy](https://www.npmjs.com/package/bundle-buddy) can find duplicates across bundles while providing a user interface to tune webpack code splitting behavior. [bundle-buddy-webpack-plugin](https://www.npmjs.com/package/bundle-buddy-webpack-plugin) makes it simpler to use.

##独立工具
## Independent Tools

除了使用webpack输出的工具之外，还有一些与webpack无关，值得一提。
In addition to tools that work with webpack output, there are a couple that are webpack agnostic and worth a mention.

### source-map-explorer
### source-map-explorer

[source-map-explorer]（https://www.npmjs.com/package/source-map-explorer）是一个独立于webpack的工具。它允许你使用源映射深入了解你的构建。它提供了基于树图的可视化，显示了代码对结果的贡献。
[source-map-explorer](https://www.npmjs.com/package/source-map-explorer) is a tool independent of webpack. It allows you to get insight into your build by using source maps. It gives a treemap based visualization showing what code contributes to the result.

### madge
![madge](images/madge.png)

[madge]（https://www.npmjs.com/package/madge）是另一个可以根据模块输入输出图表的独立工具。图形输出允许你更详细地了解项目的依赖关系。
[madge](https://www.npmjs.com/package/madge) is another independent tool that can output a graph based on module input. The graph output allows you to understand the dependencies of your project in greater detail.

{pagebreak}

## 总结

当你优化捆绑输出的大小时，这些工具是非常宝贵的。官方工具具有最多的功能，但即使是基本的可视化也可以揭示问题点。你可以使用与旧项目相同的技术来了解它们的组成。
When you are optimizing the size of your bundle output, these tools are invaluable. The official tool has the most functionality, but even a rudimentary visualization can reveal problem spots. You can use the same technique with old projects to understand their composition.

回顾一下：

* Webpack允许你提取包含有关构建的信息的JSON文件。数据可以包括构建组成和时间。
* Webpack allows you to extract a JSON file containing information about the build. The data can include the build composition and timing.
*可以使用各种工具分析生成的数据，这些工具可以深入了解捆绑组成等方面。
* The generated data can be analyzed using various tools that give insight into aspects such as the bundle composition.
* **性能预算**允许你设置构建大小的限制。维持预算可以使开发人员更加意识到生成的捆绑包的大小。
* **Performance budget** allows you to set limits to the build size. Maintaining a budget can keep developers more conscious of the size of the generated bundles.
*了解捆绑包是了解如何优化整体规模，加载内容和时间的关键。它还可以揭示更重要的问题，例如冗余数据。
* Understanding the bundles is the key to insights on how to optimize the overall size, what to load and when. It can also reveal more significant issues, such as redundant data.
*你可以找到不依赖于webpack但仍然有价值的第三方工具进行分析。
* You can find third-party tools that don't depend on webpack but are still valuable for analysis.

你将学习如何在下一章中调整webpack性能。
You'll learn to tune webpack performance in the next chapter.

