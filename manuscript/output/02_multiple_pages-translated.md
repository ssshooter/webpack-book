＃多页
# Multiple Pages

即使webpack经常用于捆绑单页面应用程序，也可以将它与多个单独的页面一起使用。这个想法类似于在* Targets *章节中生成多个输出文件的方式。但是，这一次，你必须生成单独的页面。这可以通过`HtmlWebpackPlugin`和一些配置来实现。
Even though webpack is often used for bundling single page applications, it's possible to use it with multiple separate pages as well. The idea is similar to the way you generated multiple output files in the *Targets* chapter. This time, however, you have to generate separate pages. That's achievable through `HtmlWebpackPlugin` and a bit of configuration.

##可能的方法
## Possible Approaches

使用webpack生成多个页面时，你有以下几种可能性：
When generating multiple pages with webpack, you have a couple of possibilities:

*通过*多编译器模式*并返回一系列配置。只要页面是分开的，并且跨越它们共享代码的最小需求，该方法就可以工作。这种方法的好处是你可以通过[parallel-webpack]（https://www.npmjs.com/package/parallel-webpack）处理它，以提高构建性能。
* Go through the *multi-compiler mode* and return an array of configurations. The approach would work as long as the pages are separate and there is a minimal need for sharing code across them. The benefit of this approach is that you can process it through [parallel-webpack](https://www.npmjs.com/package/parallel-webpack) to improve build performance.
*设置单个配置并提取共性。你执行此操作的方式可能会有所不同，具体取决于你的方式。
* Set up a single configuration and extract the commonalities. The way you do this can differ depending on how you chunk it up.
*如果你遵循[渐进式网络应用程序]（https://developers.google.com/web/progressive-web-apps/）（PWA）的想法，你最终可以使用** app shell **或a **页面shell **并在使用时加载应用程序的部分。
* If you follow the idea of [Progressive Web Applications](https://developers.google.com/web/progressive-web-apps/) (PWA), you can end up with either an **app shell** or a **page shell** and load portions of the application as it's used.

在实践中，你有更多的维度。例如，你必须为页面生成i18n变体。这些想法在基本方法之上发展。
In practice, you have more dimensions. For example, you have to generate i18n variants for pages. These ideas grow on top of the basic approaches.

##生成多个页面
## Generating Multiple Pages

要生成多个单独的页面，应以某种方式初始化它们。你还应该能够返回每个页面的配置，因此webpack会选择它们并通过多编译器模式处理它们。
To generate multiple separate pages, they should be initialized somehow. You should also be able to return a configuration for each page, so webpack picks them up and process them through the multi-compiler mode.

###抽象页面
### Abstracting Pages

要初始化页面，它至少应该接收页面标题，输出路径和可选模板。每个页面都应该接收可选的输出路径和用于自定义的模板。这个想法可以建模为配置部分：
To initialize a page, it should receive page title, output path, and an optional template at least. Each page should receive optional output path and a template for customization. The idea can be modeled as a configuration part:

** ** webpack.parts.js
**webpack.parts.js**

```javascript
...
const HtmlWebpackPlugin = require("html-webpack-plugin");

exports.page = ({
  path = "",
  template = require.resolve(
    "html-webpack-plugin/default_index.ejs"
  ),
  title,
} = {}) => ({
  plugins: [
    new HtmlWebpackPlugin({
      filename: `${path && path + "/"}index.html`,
      template,
      title,
    }),
  ],
});
```

{pagebreak}

###集成到配置
### Integrating to Configuration

要将这个想法融入到配置中，它的组合方式必须改变。此外，还需要页面定义。首先，让我们暂时为每个页面重用相同的JavaScript逻辑：
To incorporate the idea into the configuration, the way it's composed has to change. Also, a page definition is required. To get started, let's reuse the same JavaScript logic for each page for now:

** ** webpack.config.js
**webpack.config.js**

```javascript
...
leanpub-start-delete
const HtmlWebpackPlugin = require("html-webpack-plugin");
leanpub-end-delete
...


const commonConfig = merge([
leanpub-start-delete
  {
    plugins: [
      new HtmlWebpackPlugin({
        title: "Webpack demo",
      }),
    ],
  },
leanpub-end-delete
  ...
]);

...

module.exports = mode => {
leanpub-start-delete
  if (mode === "production") {
    return merge(commonConfig, productionConfig, { mode });
  }

  return merge(commonConfig, developmentConfig, { mode });
leanpub-end-delete
leanpub-start-insert
  const pages = [
    parts.page({ title: "Webpack demo" }),
    parts.page({ title: "Another demo", path: "another" }),
  ];
  const config =
    mode === "production" ? productionConfig : developmentConfig;

  return pages.map(page =>
    merge(commonConfig, config, page, { mode })
  );
leanpub-end-insert
};
```

在这个改变之后你应该在应用程序中有两个页面：`/`和`/ another`。应该可以在看到相同输出的同时导航到两者。
After this change you should have two pages in the application: `/` and `/another`. It should be possible to navigate to both while seeing the same output.

###每页注入不同的脚本
### Injecting Different Script per Page

问题是，如何为每个页面注入不同的脚本。在当前配置中，两者共享相同的`entry`。要解决此问题，你应该将`entry`配置移到较低级别并按页面进行管理。要使用脚本进行测试，请设置另一个入口点：
The question is, how to inject a different script per each page. In the current configuration, the same `entry` is shared by both. To solve the problem, you should move `entry` configuration to lower level and manage it per page. To have a script to test with, set up another entry point:

** SRC / another.js **
**src/another.js**

```javascript
import "./main.css";
import component from "./component";

const demoComponent = component("Another");

document.body.appendChild(demoComponent);
```

{pagebreak}

该文件可以转到自己的目录。这里重用现有代码以显示某些内容。 Webpack配置必须指向此文件：
The file could go to a directory of its own. Here the existing code is reused to get something to show up. Webpack configuration has to point to this file:

** ** webpack.config.js
**webpack.config.js**

```javascript
...

const commonConfig = merge([
leanpub-start-insert
  {
    output: {
      // Needed for code splitting to work in nested paths
      publicPath: "/",
    },
  },
leanpub-end-insert
  ...
]);

...

module.exports = mode => {
leanpub-start-delete
  const pages = [
    parts.page({ title: "Webpack demo" }),
    parts.page({ title: "Another demo", path: "another" }),
  ];
leanpub-end-delete
leanpub-start-insert
  const pages = [
    parts.page({
      title: "Webpack demo",
      entry: {
        app: PATHS.app,
      },
    }),
    parts.page({
      title: "Another demo",
      path: "another",
      entry: {
        another: path.join(PATHS.app, "another.js"),
      },
    }),
  ];
leanpub-end-insert
  const config =
    mode === "production" ? productionConfig : developmentConfig;

    return pages.map(page =>
      merge(commonConfig, config, page, { mode })
    );
};
```

调整还需要在相关部分进行更改，以便“entry”包含在配置中：
The tweak also requires a change at the related part so that `entry` gets included in the configuration:

** ** webpack.parts.js
**webpack.parts.js**

```javascript
exports.page = (
  {
    path = "",
    template = require.resolve(
      "html-webpack-plugin/default_index.ejs"
    ),
    title,
leanpub-start-insert
    entry,
leanpub-end-insert
  } = {}
) => ({
leanpub-start-insert
  entry,
leanpub-end-insert
  plugins: [
    new HtmlWebpackPlugin({
      filename: `${path && path + "/"}index.html`,
      title,
    }),
  ],
});
```

{pagebreak}

在这些变化之后``/ another`应该显示一些熟悉的东西：
After these changes `/another` should show something familiar:

![Another page shows up](images/another.png)

###优点和缺点
### Pros and Cons

如果你构建应用程序（`npm run build`），你应该找到* another / index.html *。根据生成的代码，你可以进行以下观察：
If you build the application (`npm run build`), you should find *another/index.html*. Based on the generated code, you can make the following observations:

*很清楚如何在设置中添加更多页面。
* It's clear how to add more pages to the setup.
*生成的资产直接位于构建根目录下方。页面是一个例外，因为它们由`HtmlWebpackPlugin`处理，但它们仍然指向根目录下的资产。可以以* webpack.page.js *的形式添加更多抽象，并通过公开接受页面配置的函数来管理路径。
* The generated assets are directly below the build root. The pages are an exception as those are handled by `HtmlWebpackPlugin`, but they still point to the assets below the root. It would be possible to add more abstraction in the form of *webpack.page.js* and manage the paths by exposing a function that accepts page configuration.
*记录应按照每页的文件单独写在自己的文件中。目前，写入最后一次胜利的配置。上述解决方案可以解决这个问题。
* Records should be written separately per each page in files of their own. Currently, the configuration that writes the last wins. The above solution would allow solving this.
*拉丝和清洁等过程现在运行两次。 * Targets *章节讨论了该问题的潜在解决方案。
* Processes like linting and cleaning run twice now. The *Targets* chapter discussed potential solutions to that problem.

通过删除多编译器模式，可以将方法推向另一个方向。尽管处理这种构建的速度较慢，但​​它可以实现代码共享和shell的实现。实现shell设置的第一步是重新配置配置，以便它获取页面之间共享的代码。
The approach can be pushed in another direction by dropping the multi-compiler mode. Even though it's slower to process this kind of build, it enables code sharing and the implementation of shells. The first step towards a shell setup is to rework the configuration so that it picks up the code shared between the pages.

##共享代码时生成多个页面
## Generating Multiple Pages While Sharing Code

由于使用模式，当前配置已经巧合地共享代码。只有一小部分代码不同，因此只有页面清单，而映射到其条目的包不同。
The current configuration shares code by coincidence already due to the usage patterns. Only a small part of the code differs, and as a result, only the page manifests, and the bundles mapping to their entries differ.

在更复杂的应用程序中，你应该跨页面应用* Bundle Splitting *章节中介绍的技术。因此，删除多编译器模式是值得的。
In a more complicated application, you should apply techniques covered in the *Bundle Splitting* chapter across the pages. Dropping the multi-compiler mode can be worthwhile then.

###调整配置
### Adjusting Configuration

需要进行调整以在页面之间共享代码。大多数代码可以保持不变。将它暴露给webpack的方式必须改变，以便它接收单个配置对象。由于`HtmlWebpackPlugin`默认选择所有块，你必须调整它以仅选取与每个页面相关的块：
Adjustment is needed to share code between the pages. Most of the code can remain the same. The way you expose it to webpack has to change so that it receives a single configuration object. As `HtmlWebpackPlugin` picks up all chunks by default, you have to adjust it to pick up only the chunks that are related to each page:

** ** webpack.config.js
**webpack.config.js**

```javascript
...

module.exports = mode => {
  const pages = [
    parts.page({
      title: "Webpack demo",
      entry: {
        app: PATHS.app,
      },
leanpub-start-insert
      chunks: ["app", "manifest", "vendor"],
leanpub-end-insert
    }),
    parts.page({
      title: "Another demo",
      path: "another",
      entry: {
        another: path.join(PATHS.app, "another.js"),
      },
leanpub-start-insert
      chunks: ["another", "manifest", "vendor"],
leanpub-end-insert
    }),
  ];
  const config =
    mode === "production" ? productionConfig : developmentConfig;

leanpub-start-delete
  return pages.map(page =>
    merge(commonConfig, config, page, { mode })
  );
leanpub-end-delete
leanpub-start-insert
  return merge([commonConfig, config, { mode }].concat(pages));
leanpub-end-insert
};
```

特定于页面的配置也需要一些小的调整：
The page-specific configuration requires a small tweak as well:

** ** webpack.parts.js
**webpack.parts.js**

```javascript
exports.page = (
  {
    path = "",
    template = require.resolve(
      "html-webpack-plugin/default_index.ejs"
    ),
    title,
    entry,
leanpub-start-insert
    chunks,
leanpub-end-insert
  } = {}
) => ({
  entry,
  plugins: [
    new HtmlWebpackPlugin({
leanpub-start-insert
      chunks,
leanpub-end-insert
      ...
    }),
  ],
});
```

如果你生成一个构建（`npm run build`），你应该注意到与你做的第一个多页构建相比有些不同。你只能找到一个，而不是两个清单文件。如果检查它，你会注意到它包含对生成的所有文件的引用。
If you generate a build (`npm run build`), you should notice that something is different compared to the first multiple page build you did. Instead of two manifest files, you can find only one. If you examine it, you notice it contains references to all files that were generated.

详细研究条目特定文件可以发现更多信息。你可以看到它们指向清单的不同部分。清单根据条目运行不同的代码。不需要多个单独的清单。
Studying the entry specific files in detail reveals more. You can see that they point to different parts of the manifest. The manifest runs different code depending on the entry. Multiple separate manifests are not needed.

###优点和缺点
### Pros and Cons

与之前的方法相比，获得了一些东西，但也丢失了：
Compared to the earlier approach, something was gained, but also lost:

*鉴于配置不再是多编译器形式，处理可能会更慢。
* Given the configuration isn't in the multi-compiler form anymore, processing can be slower.
*现在，如果没有额外的考虑，诸如`CleanWebpackPlugin`之类的插件就无法运行。
* Plugins such as `CleanWebpackPlugin` don't work without additional consideration now.
*而不是多个清单，只剩下一个。但结果不是问题，因为条目根据其设置使用它不同。
* Instead of multiple manifests, only one remains. The result is not a problem, though, as the entries use it differently based on their setup.

##渐进式Web应用程序
## Progressive Web Applications

如果通过将其与代码拆分和智能路由相结合来进一步推动这一想法，你将最终实现渐进式Web应用程序（PWA）的理念。 [webpack-pwa]（https://github.com/webpack/webpack-pwa）示例说明了如何通过app shell或页面shell使用webpack实现该方法。
If you push the idea further by combining it with code splitting and smart routing, you'll end up with the idea of Progressive Web Applications (PWA). [webpack-pwa](https://github.com/webpack/webpack-pwa) example illustrates how to implement the approach using webpack either through an app shell or a page shell.

App shell最初加载，它管理整个应用程序，包括其路由。页面外壳更精细，并且在使用应用程序时会加载更多内容。在这种情况下，应用程序的总大小更大。相反，你可以更快地加载初始内容。
App shell is loaded initially, and it manages the whole application including its routing. Page shells are more granular, and more are loaded as the application is used. The total size of the application is larger in this case. Conversely, you can load initial content faster.

PWA与[offline-plugin]（https://www.npmjs.com/package/offline-plugin）和[sw-precache-webpack-plugin]等插件完美结合（https://www.npmjs.com/package / SW-预缓存-的WebPack-插件）。使用[服务工作者]（https://developer.mozilla.org/en/docs/Web/API/Service_Worker_API）并改善离线体验。
PWA combines well with plugins like [offline-plugin](https://www.npmjs.com/package/offline-plugin) and [sw-precache-webpack-plugin](https://www.npmjs.com/package/sw-precache-webpack-plugin). Using [Service Workers](https://developer.mozilla.org/en/docs/Web/API/Service_Worker_API) and improves the offline experience.

T> [Twitter]（https://developers.google.com/web/showcase/2017/twitter）和[Tinder]（https://medium.com/@addyosmani/a-tinder-progressive-web-app- performance-case-study-78919d98ece0）案例研究说明了PWA方法如何改进平台。
T> [Twitter](https://developers.google.com/web/showcase/2017/twitter) and [Tinder](https://medium.com/@addyosmani/a-tinder-progressive-web-app-performance-case-study-78919d98ece0) case studies illustrate how the PWA approach can improve platforms.

## 总结


Webpack允许你管理多个页面设置。 PWA方法允许应用程序在使用时加载，webpack允许实现它。
Webpack allows you to manage multiple page setups. The PWA approach allows the application to be loaded as it's used and webpack allows implementing it.

回顾一下：


* Webpack可用于通过多编译器模式生成单独的页面，也可以将所有页面配置包含在一起。
* Webpack can be used to generate separate pages either through its multi-compiler mode or by including all the page configuration into one.
*多编译器配置可以使用外部解决方案并行运行，但是应用捆绑拆分等技术会更加困难。
* The multi-compiler configuration can run in parallel using external solutions, but it's harder to apply techniques such as bundle splitting against it.
*多页面设置可以导致**渐进式Web应用程序**。在这种情况下，你可以使用各种webpack技术来提供快速加载并根据需要获取功能的应用程序。这种技术的两种风格都有其自身的优点。
* A multi-page setup can lead to a **Progressive Web Application**. In this case, you use various webpack techniques to come up with an application that is fast to load and that fetches functionality as required. Both two flavors of this technique have their own merits.

你将学习如何在下一章中实现*服务器端渲染*。
You'll learn to implement *Server Side Rendering* in the next chapter.

