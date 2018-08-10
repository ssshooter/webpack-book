＃词汇表
# Glossary

鉴于webpack带有特定的术语，下面根据讨论它们的书籍部分收集了主要术语及其解释。
Given webpack comes with specific terminology, the principal terms and their explanations have been gathered below based on the book part where they are discussed.

＃＃ 介绍
## Introduction

* **静态分析**  - 当工具执行静态分析时，它会检查代码而不运行它，这就是ESLint或webpack等工具的运行方式。静态可分析标准，如ES2015模块定义，可实现**树摇动等功能。
* **Static analysis** - When a tool performs static analysis, it examines the code without running it which is how tools like ESLint or webpack operate. Statically analyzable standards, like ES2015 module definition, enable features like **tree shaking**.
* **解析**是webpack遇到模块或加载器时发生的过程。当发生这种情况时，它会尝试根据给定的解析规则解决它。
* **Resolving** is the process that happens when webpack encounters a module or a loader. When that happens, it tries to resolve it based on the given resolution rules.

##开发
## Developing

* ** Entry **是指webpack使用的文件作为捆绑的起点。应用程序可以有多个条目，根据配置，每个条目可以生成多个包。条目在webpack的`entry`配置中定义。条目是依赖图开头的**模块**。
* **Entry** refers to a file used by webpack as a starting point for bundling. An application can have multiple entries and depending on configuration, each entry can result in multiple bundles. Entries are defined in webpack's `entry` configuration. Entries are **modules** at the beginning of the dependency graph.
* **模块**是描述一部分应用程序的通用术语。在webpack中，它可以引用JavaScript，样式表，图像或其他内容。 ** Loaders **允许webpack支持不同的文件类型，因此支持不同类型的模块。如果从代码库的多个位置指向同一个模块，webpack将在输出中生成单个模块，从而在模块级别启用单例模式。
* **Module** is a general term to describe a piece of the application. In webpack, it can refer to JavaScript, a style sheet, an image or something else. **Loaders** allows webpack to support different file types and therefore different types of module. If you point to the same module from multiple places of a code base, webpack will generate a single module in the output which enables the singleton pattern on module level.
* **插件**连接到webpack的事件系统，可以将功能注入其中。它们允许webpack扩展，并可与装载机组合以实现最大程度的控制。虽然加载器在单个文件上工作，但插件具有更广泛的访问权限，并且能够进行更全面的控制。
* **Plugins** connect to webpack's event system and can inject functionality into it. They allow webpack to be extended and can be combined with loaders for maximum control. Whereas a loader works on a single file, a plugin has much broader access and is capable of more global control.
* **热模块替换（HMR）**是指一种技术，可以在运行中修补在浏览器中运行的代码，而无需刷新整页。当应用程序包含复杂状态时，如果没有HMR或类似的解决方案，则恢复它可能很困难。
* **Hot Module Replacement (HMR)** refers to a technique where code running in the browser is patched on the fly without requiring a full page refresh. When an application contains complex state, restoring it can be difficult without HMR or a similar solution.
* ** Linting **涉及针对一系列用户定义的问题静态分析代码的过程。这些问题的范围可以从发现语法错误到强制执行代码样式。虽然linting在定义上受限于其功能，但linter对于帮助早期错误发现和强制执行代码一致性非常有用。
* **Linting** relates to the process in which code is statically analyzed for a series of user-defined issues. These issues can range from discovering syntax errors to enforcing code-style. While linting is by definition limited in its capabilities, a linter is invaluable for helping with early error discovery and enforcing code consistency.

##正在加载
## Loading

* ** Loader **执行转换，接受源并返回转换后的源。它也可以跳过处理并对输入执行检查。通过配置，加载器通常基于模块类型或位置来定位模块的子集。加载器一次只作用于一个模块，而插件可以作用于多个文件。
* **Loader** performs a transformation that accepts a source and returns transformed source. It can also skip processing and perform a check against the input instead. Through configuration, a loader targets a subset of modules, often based on the module type or location. A loader only acts on a single module at a time whereas a plugin can act on multiple files.
* **资产**是项目的媒体和源文件的通用术语，它是webpack构建捆绑包时使用的原材料。
* **Asset** is a general term for the media and source files of a project that are the raw material used by webpack in building a bundle.

＃＃ 建造
## Building

* **源映射**描述源代码和生成的代码之间的映射，允许浏览器提供更好的调试体验。例如，通过Babel运行ES2015代码会生成全新的ES5代码。如果没有源映射，开发人员将丢失生成的代码中发生的事情以及源代码中发生的事件的链接。样式表在通过预处理器或后处理器运行时也是如此。
* **Source maps** describe the mapping between the source code and the generated code, allowing browsers to provide a better debugging experience. For example, running ES2015 code through Babel generates completely new ES5 code. Without a source map, a developer would lose the link from where something happens in the generated code and where it happens in the source code. The same is true for style sheets when they run through a pre or post-processor.
* ** Bundle **是捆绑的结果。捆绑包括将应用程序的源材料处理成可以使用的最终捆绑包。捆绑器可以生成多个捆绑包。
* **Bundle** is the result of bundling. Bundling involves processing the source material of the application into a final bundle that is ready to use. A bundler can generate more than one bundle.
* **捆绑拆分**提供了一种优化构建的方法，允许webpack为单个应用程序生成多个捆绑包。因此，每个捆绑包都可以与影响其他捆绑的更改隔离开来，从而减少需要重新发布的代码量，从而减少客户端重新下载并利用浏览器缓存的优势。
* **Bundle splitting** offers one way of optimizing a build, allowing webpack to generate multiple bundles for a single application. As a result, each bundle can be isolated from changes affecting others, reducing the amount of code that needs to be republished and therefore re-downloaded by the client and taking advantage of browser caching.
* **代码拆分**产生的粒度束比束拆分更多。要使用它，开发人员必须通过源代码中的特定调用启用它。使用动态`import（）`是一种方法。
* **Code splitting** produces more granular bundles than bundle splitting. To use it, the developer has to enable it through specific calls in the source code. Using a dynamic `import()` is one way.
* ** Chunk **是一个特定于webpack的术语，用于内部管理捆绑过程。 Webpack从块中组合出束，并且有几种类型。
* **Chunk** is a webpack-specific term that is used internally to manage the bundling process. Webpack composes bundles out of chunks, and there are several types of those.

##优化
## Optimizing

* **缩小**或缩小是一种优化技术，其中代码以更紧凑的形式编写而不会失去意义。如果你不小心，特定的破坏性转换会破坏代码。
* **Minifying**, or minification, is an optimization technique in which code is written in a more compact form without losing meaning. Specific destructive transformations break code if you are not careful.
* **树抖动**是基于静态分析丢弃未使用代码的过程。 ES2015模块定义允许此过程，因为它可以以这种特定方式进行分析。
* **Tree shaking** is the process of dropping unused code based on static analysis. ES2015 module definition allows this process as it's possible to analyze in this particular manner.
* **散列**是指生成附加到资产/捆绑路径的散列以在客户端上使其无效的过程。散列包名称的示例：* app.f6f78b2fd2c38e8200d.js *。
* **Hashing** refers to the process of generating a hash that is attached to the asset/bundle path to invalidate it on the client. Example of a hashed bundle name: *app.f6f78b2fd2c38e8200d.js*.

##输出
## Output

* **输出**是指webpack发出的文件。更具体地说，webpack根据输出设置发出**包**和**资产**。
* **Output** refers to files emitted by webpack. More specifically, webpack emits **bundles** and **assets** based on the output settings.
* ** webpack的目标**选项允许你覆盖默认的Web目标。你可以使用webpack为特定的JavaScript平台开发代码。
* **Target** options of webpack allow you to override the default web target. You can use webpack to develop code for specific JavaScript platforms.

