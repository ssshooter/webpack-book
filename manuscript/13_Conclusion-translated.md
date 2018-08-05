 - ＃结论
-# Conclusion

正如本书所展示的那样，webpack是一个多功能的工具。为了更容易回顾内容和技术，请查看下面的清单。
As this book has demonstrated, webpack is a versatile tool. To make it easier to recap the content and techniques, go through the checklists below.

##一般清单
## General Checklist

* **源映射**允许您在开发期间在浏览器中调试代码。如果捕获输出，它们还可以在生产使用期间提供更高质量的堆栈跟踪。 * Source Maps *章节深入探讨了该主题。
* **Source maps** allow you to debug your code in the browser during development. They can also give better quality stack traces during production usage if you capture the output. The *Source Maps* chapter delves into the topic.
*为了保持快速构建，请考虑优化。 * Performance *章节讨论了可用于实现此目的的各种策略。
* To keep your builds fast, consider optimizing. The *Performance* chapter discusses a variety of strategies you can use to achieve this.
*为了保持您的配置可维护性，请考虑编写它。由于webpack配置是JavaScript代码，因此可以通过多种方式进行安排。 * Composing Configuration *章节讨论了该主题。
* To keep your configuration maintainable, consider composing it. As webpack configuration is JavaScript code, it can be arranged in many ways. The *Composing Configuration* chapter discusses the topic.
* webpack消费包的方式可以定制。 *消费包*章节涵盖了与此相关的特定技术。
* The way webpack consumes packages can be customized. The *Consuming Packages* chapter covers specific techniques related to this.
*有时您必须扩展webpack。 *使用Loaders扩展*和*使用插件进行扩展*章节展示了如何实现这一目标。您还可以在webpack的配置定义之上工作，并根据您的目的实现自己的抽象。
* Sometimes you have to extend webpack. The *Extending with Loaders* and *Extending with Plugins* chapters show how to achieve this. You can also work on top of webpack’s configuration definition and implement an abstraction of your own for it to suit your purposes.

##开发清单
## Development Checklist

*要在开发期间充分利用webpack，请使用* webpack-dev-server *（WDS）。您还可以找到可在开发期间附加到节点服务器的中间件。 *自动浏览器刷新*章节更详细地介绍了WDS。
* To get most out of webpack during development, use *webpack-dev-server* (WDS). You can also find middlewares which you can attach to your Node server during development. The *Automatic Browser Refresh* chapter covers WDS in greater detail.
* Webpack实现**热模块替换**（HMR）。它允许您在应用程序运行时更换模块而不强制浏览器刷新。 *热模块更换*附录详细介绍了该主题。
* Webpack implements **Hot Module Replacement** (HMR). It allows you to replace modules without forcing a browser refresh while your application is running. The *Hot Module Replacement* appendix covers the topic in detail.

##生产清单
## Production Checklist

###造型
### Styling

* Webpack默认将样式定义内联到JavaScript。为避免这种情况，请使用“MiniCssExtractPlugin”或等效解决方案将CSS分离为自己的文件。 * Separating CSS *章节介绍了如何实现这一目标。
* Webpack inlines style definitions to JavaScript by default. To avoid this, separate CSS to a file of its own using `MiniCssExtractPlugin` or an equivalent solution. The *Separating CSS* chapter covers how to achieve this.
*要减少要写入的CSS规则的数量，请考虑** autoprefixing **您的规则。 * Autoprefixing *章节显示了如何执行此操作。
* To decrease the number of CSS rules to write, consider **autoprefixing** your rules. The *Autoprefixing* chapter shows how to do this.
*可以基于静态分析消除未使用的CSS规则。 *消除未使用的CSS *章节解释了这种技术的基本思想。
* Unused CSS rules can be eliminated based on static analysis. The *Eliminating Unused CSS* chapter explains the basic idea of this technique.

###资产
### Assets

*通过webpack加载图像时，优化它们，以便用户下载更少。 * Loading Images *章节显示了如何执行此操作。
* When loading images through webpack, optimize them, so the users have less to download. The *Loading Images* chapter shows how to do this.
*根据您必须支持的浏览器，仅加载您需要的字体。 * Loading Fonts *章节讨论了该主题。
* Load only the fonts you need based on the browsers you have to support. The *Loading Fonts* chapter discusses the topic.
*缩小源文件以确保浏览器减少客户端必须下载的有效负载。 * Minifying *章节展示了如何实现这一目标。
* Minify your source files to make sure the browser to decrease the payload the client has to download. The *Minifying* chapter shows how to achieve this.

###缓存
### Caching

*要从客户端缓存中受益，请将应用程序包拆分出应用程序。这样，客户端在理想情况下的下载量就会减少。 * Bundle Splitting *章节讨论了该主题。 *将哈希添加到文件名*章节显示了如何在此基础上实现缓存失效。
* To benefit from client caching, split a vendor bundle out of your application. This way the client has less to download in the ideal case. The *Bundle Splitting* chapter discusses the topic. The *Adding Hashes to Filenames* chapter shows how to achieve cache invalidation on top of that.
*使用webpack的**代码拆分**功能按需加载代码。如果您不需要同时使用所有代码，并且可以将其推送到逻辑触发器（例如单击用户界面元素）之后，该技术很方便。 * Code Splitting *章节详细介绍了该技术。 *动态加载*章节显示了如何处理更高级的方案。
* Use webpack’s **code splitting** functionality to load code on demand. The technique is handy if you don’t need all the code at once and instead can push it behind a logical trigger such as clicking a user interface element. The *Code Splitting* chapter covers the technique in detail. The *Dynamic Loading* chapter shows how to handle more advanced scenarios.
*将散列加到文件名中，如*添加散列到文件名*章节所述，以便从缓存中获益，并分离清单以进一步改进解决方案，如* Separating Manifest *章节中所述。
* Add hashes to filenames as covered in the *Adding Hashes to Filenames* chapter to benefit from caching and separate a manifest to improve the solution further as discussed in the *Separating Manifest* chapter.

###优化
### Optimization

*使用ES2015模块定义来利用**树摇动**。它允许webpack通过静态分析消除未使用的代码路径。请参阅* Tree Shaking *章节了解该想法。
* Use ES2015 module definition to leverage **tree shaking**. It allows webpack to eliminate unused code paths through static analysis. See the *Tree Shaking* chapter for the idea.
*设置特定于应用程序的环境变量以编译其生产模式。您可以通过这种方式实现功能标记。请参阅*环境变量*章节以概述该技术。
* Set application-specific environment variables to compile it production mode. You can implement feature flags this way. See the *Environment Variables* chapter to recap the technique.
*分析构建统计信息以了解要改进的内容。 * Build Analysis *章节显示了如何针对多个可用工具执行此操作。
* Analyze build statistics to learn what to improve. The *Build Analysis* chapter shows how to do this against multiple available tools.
*将部分计算推送给网络工作者。 * Web Workers *章节介绍了如何实现这一目标。
* Push a part of the computation to web workers. The *Web Workers* chapter covers how to achieve this.

###输出
### Output

*清理并将有关构建的信息附加到结果中。 * Tidying Up *章节介绍了如何执行此操作。
* Clean up and attach information about the build to the result. The *Tidying Up* chapter shows how to do this.

##结论
## Conclusion

Webpack允许您使用许多不同的技术来拼接您的构建。它支持多种输出格式，如本书* Output *部分所述。尽管它的名字，它不仅适用于网络。这是大多数人使用它的地方，但该工具远不止于此。
Webpack allows you to use a lot of different techniques to splice up your build. It supports multiple output formats as discussed in the *Output* part of the book. Despite its name, it’s not only for the web. That’s where most people use it, but the tool does far more than that.

