＃ 故障排除
# Troubleshooting

使用webpack可能会导致各种运行时警告或错误。通常，构建的特定部分由于某种原因而失败。可以使用基本过程来解决这些问题：
Using webpack can lead to a variety of runtime warnings or errors. Often a particular part of the build fails for a reason or another. A basic process can be used to figure out these problems:

1.将`--display-error-details`标志传递给webpack，以获得更准确的错误来学习。示例：`npm run build  -  --display-error-details`。
1. Pass `--display-error-details` flag to webpack to get a more accurate error to study. Example: `npm run build -- --display-error-details`.
2.仔细研究错误的起源。有时你可以推断出上下文有什么问题。如果webpack无法解析模块，则可能不会通过你期望的加载器传递它。
2. Study the origin of the error carefully. Sometimes you can infer what's wrong with context. If webpack fails to parse a module, it's likely not passing it through a loader you expect for example.
3.尝试了解错误的来源。它来自你的代码，依赖项或Webpack吗？
3. Try to understand where the error stems. Does it come from your code, a dependency, or webpack?
4.删除代码，直到错误消失并添加代码，直到它再次出现。尽可能简化以解决问题。
4. Remove code until the error goes away and add code back till it appears again. Simplify as much as possible to isolate the problem.
5.如果代码在另一个项目中有效，请找出不同之处。项目之间的依赖关系可能会有所不同，或者设置会有所不同。在最糟糕的情况下，你依赖的包装已经获得了回归。出于这个原因，使用* lockfile *是一个好主意。
5. If the code worked in another project, figure out what's different. It's possible the dependencies between the projects vary, or the setup differs somehow. At the worst case, a package you rely upon has gained a regression. Using a *lockfile* is a good idea for this reason.
6.仔细研究相关包装。有时查看包* package.json *可以产生洞察力。你使用的软件包可能无法按预期方式解析。
6. Study the related packages carefully. Sometimes looking into the package *package.json* can yield insight. It's possible the package you are using does not resolve the way you expect.
7.在线搜索错误。也许其他人遇到过它。 [Stack Overflow]（https://stackoverflow.com/questions/tagged/webpack）和[官方问题跟踪器]（https://github.com/webpack/webpack/issues）是很好的起点。
7. Search for the error online. Perhaps someone else has run into it. [Stack Overflow](https://stackoverflow.com/questions/tagged/webpack) and [the official issue tracker](https://github.com/webpack/webpack/issues) are good starting points.
8.启用`stats：'verbose“`以从webpack中获取更多信息。 [官方文档包含更多标志]（https://webpack.js.org/configuration/stats/）。
8. Enable `stats: "verbose"` to get more information out of webpack. The [official documentation covers more flags](https://webpack.js.org/configuration/stats/).
9.在错误附近添加临时`console.log`以更深入地了解问题。更重要的选择是[通过Chrome开发工具调试webpack]（https://medium.com/webpack/webpack-bits-learn-and-debug-webpack-with-chrome-dev-tools-da1c5b19554）。
9. Add a temporary `console.log` near the error to get more insight into the problem. A heavier option is to [debug webpack through Chrome Dev Tools](https://medium.com/webpack/webpack-bits-learn-and-debug-webpack-with-chrome-dev-tools-da1c5b19554).
10. [在Stack Overflow上提问]（https://stackoverflow.com/questions/tagged/webpack）或[使用官方Gitter频道]（https://gitter.im/webpack/webpack）。
10. [Ask a question at Stack Overflow](https://stackoverflow.com/questions/tagged/webpack) or [use the official Gitter channel](https://gitter.im/webpack/webpack).
11.如果一切都失败了，你确信你发现了一个错误，[在官方问题跟踪器上报告一个问题]（https://github.com/webpack/webpack/issues）或在其他适当的地方报道如果它是一个问题依赖。请仔细遵循问题模板，并提供最小的可运行示例，因为它有助于解决问题。
11. If everything fails and you are convinced you have found a bug, [report an issue at the official issue tracker](https://github.com/webpack/webpack/issues) or at other appropriate places if it's an issue in a dependency. Follow the issue template carefully, and provide a minimal runnable example as it helps to resolve the problem.

有时将错误丢弃到搜索引擎并以此方式获得答案是最快的。除此之外，这是一个很好的调试顺序。如果你的设置过去有效，你还可以考虑使用[git bisect]（https://git-scm.com/docs/git-bisect）之类的命令来确定已知工作状态和当前状态之间的变化破了一个。
Sometimes it's fastest to drop the error to a search engine and gain an answer that way. Other than that this is an excellent debugging order. If your setup worked in the past, you could also consider using commands like [git bisect](https://git-scm.com/docs/git-bisect) to figure out what has changed between the known working state and the current broken one.

你将了解下一个最常见的错误以及如何处理它们。
You'll learn about the most common errors next and how to deal with them.

找不到Entry模块中的## ERROR
## ERROR in Entry module not found

如果在不存在的位置创建入口路径点，则最终可能会出现此错误。该错误消息告诉你webpack无法找到的路径。
You can end up with this error if you make an entry path point at a place that does not exist. The error message tells you what path webpack fails to find.

## ERROR ...未找到模块
## ERROR ... Module not found

你可以通过两种方式获得错误。通过破坏加载程序定义以使其指向不存在的加载程序或通过破坏代码中的导入路径以使其导致不存在的模块。该消息指出要修复的内容。
You can get the error in two ways. Either by breaking a loader definition so that it points to a loader that does not exist or by breaking an import path within your code so that it leads to a module that doesn't exist. The message points out what to fix.

##模块解析失败
## Module parse failed

即使webpack可以很好地解析你的模块，它仍然无法构建它们。如果你使用的是加载程序无法理解的语法，则可能会发生这种情况。你可能会遗漏处理过程中的某些内容。
Even though webpack could resolve to your modules fine, it can still fail to build them. This case can happen if you are using syntax that your loaders don't understand. You could be missing something in your processing pass.

## Loader not Found
## Loader Not Found

还有另一个微妙的加载器相关错误。如果存在与未实现加载程序接口的加载程序名称匹配的程序包，则webpack与该程序包匹配并给出运行时错误，指出程序包不是加载程序。
There's another subtle loader related error. If a package matching to a loader name that does not implement the loader interface exists, webpack matches to that and gives a runtime error that says the package is not a loader.

这个错误可以通过编写`loader：“eslint”`而不是`loader：“eslint-loader”`来实现。如果加载器根本不存在，将引发“未找到模块”错误。
This mistake can be made by writing `loader: "eslint"` instead of `loader: "eslint-loader"`. If the loader doesn't exist at all, the `Module not found` error will be raised.

##模块构建失败：未知单词
## Module build failed: Unknown word

此错误适合相同的类别。解析文件成功，但有未知的语法。很可能问题是拼写错误，但是当Webpack遵循导入并遇到它不理解的语法时，也会发生此错误。这很可能意味着该特定文件类型缺少加载程序。
This error fits the same category. Parsing the file succeeded, but there was the unknown syntax. Most likely the problem is a typo, but this error can also occur when Webpack has followed an import and encountered syntax it doesn't understand. Most likely this means that a loader is missing for that particular file type.

## SyntaxError：意外的令牌
## SyntaxError: Unexpected token

`SyntaxError`是同一类别的另一个错误。如果你使用尚未与UglifyJS一起编译的ES2015语法，则可能出现此错误。当遇到它无法识别的语法构造时，它会引发错误。
`SyntaxError` is another error for the same category. This error is possible if you use ES2015 syntax that hasn't been transpiled alongside UglifyJS. As it encounters a syntax construct it does not recognize, it raises an error.

## DeprecationWarning
## DeprecationWarning

特别是在webpack更新为新的主要版本之后，Node可能会给出“DeprecationWarning”。你正在使用的插件或加载程序可能需要更新。通常所需的更改很少。要找出警告的来源，请通过Node运行webpack：`node --trace-deprecation node_modules / .bin / webpack --env production`。
Node may give a `DeprecationWarning` especially after webpack has been updated to a new major version. A plugin or a loader you are using may require updates. Often the changes required are minimal. To figure out where the warning is coming from, run webpack through Node: `node --trace-deprecation node_modules/.bin/webpack --env production`.

将`--trace-deprecation`标志传递给Node以查看警告的来源非常重要。使用`--trace-warnings`是另一种方式，它将捕获所有警告的跟踪信息，而不仅仅是弃用。
It's important to pass the `--trace-deprecation` flag to Node to see where the warning originates from. Using `--trace-warnings` is another way and it will capture the tracing information for all warnings, not only deprecations.

## 总结


这些只是错误的例子。 webpack方面发生了特定的错误，但其余的都来自它通过加载器和插件使用的软件包。简化你的项目是一个很好的步骤，因为这样可以更容易地理解错误发生的位置。
These are only examples of errors. Specific errors happen on the webpack side, but the rest comes from the packages it uses through loaders and plugins. Simplifying your project is a good step as that makes it easier to understand where the error happens.

在大多数情况下，如果你知道要查找的位置，错误很快就会解决，但在最坏的情况下，你会遇到修复工具的错误。在这种情况下，你应该向项目提供高质量的报告并帮助解决它。
In most cases, the errors are fast to solve if you know where to look, but in the worst case, you have come upon a bug to fix in the tooling. In that case, you should provide a high-quality report to the project and help to resolve it.

