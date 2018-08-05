#Code Splitting
# Code Splitting

随着功能的发展，Web应用程序往往会变大。应用程序加载所需的时间越长，对用户来说就越令人沮丧。在连接速度较慢的移动环境中，此问题会被放大。
Web applications tend to grow big as features are developed. The longer it takes for your application to load, the more frustrating it's to the user. This problem is amplified in a mobile environment where the connections can be slow.

即使拆分捆绑包可以帮助一个档次，它们也不是唯一的解决方案，您仍然可以最终下载大量数据。幸运的是，由于**代码拆分**，可以做得更好。它允许在您需要时懒洋洋地加载代码。
Even though splitting bundles can help a notch, they are not the only solution, and you can still end up having to download a lot of data. Fortunately, it's possible to do better thanks to **code splitting**. It allows loading code lazily as you need it.

您可以在用户输入应用程序的新视图时加载更多代码。您还可以将加载绑定到特定操作，例如滚动或单击按钮。您还可以尝试预测用户下一步要做什么，并根据您的猜测加载代码。这样，当用户尝试访问它时，功能就已存在。
You can load more code as the user enters a new view of the application. You can also tie loading to a specific action like scrolling or clicking a button. You could also try to predict what the user is trying to do next and load code based on your guess. This way the functionality would be already there as the user tries to access it.

T>顺便说一下，使用webpack的延迟加载可以实现Google的[PRPL模式]（https://developers.google.com/web/fundamentals/performance/prpl-pattern/）。 PRPL（推送，渲染，预缓存，延迟加载）的设计考虑了移动网络。
T> Incidentally, it's possible to implement Google's [PRPL pattern](https://developers.google.com/web/fundamentals/performance/prpl-pattern/) using webpack's lazy loading. PRPL (Push, Render, Pre-cache, Lazy-load) has been designed with mobile web in mind.

##代码拆分格式
## Code Splitting Formats

代码拆分可以在webpack中以两种主要方式完成：通过动态`import`或`require.ensure`语法。前者用于此项目。
Code splitting can be done in two primary ways in webpack: through a dynamic `import` or `require.ensure` syntax. The former is used in this project.

目标是最终得到一个按需加载的分裂点。拆分中可以有拆分，您可以根据拆分构建整个应用程序。这样做的好处是，您的应用程序的初始有效负载可能会比其他情况下更小。
The goal is to end up with a split point that gets loaded on demand. There can be splits inside splits, and you can structure an entire application based on splits. The advantage of doing this is that then the initial payload of your application can be smaller than it would be otherwise.

![Code splitting](images/code-splitting.png)

###动态`导入`
### Dynamic `import`

[dynamic`import`语法]（https://github.com/tc39/proposal-dynamic-import）尚未在官方语言规范中。由于这个原因，特别是在Babel设置中需要进行小的调整。
The [dynamic `import` syntax](https://github.com/tc39/proposal-dynamic-import) isn't in the official language specification yet. Minor tweaks are needed especially at the Babel setup for this reason.

动态导入定义为`Promise`s：
Dynamic imports are defined as `Promise`s:

```javascript
import(/* webpackChunkName: "optional-name" */ "./module").then(
  module => {...}
).catch(
  error => {...}
);
```

{pagebreak}

可选名称允许您将多个拆分点拉入单个包中。只要它们具有相同的名称，它们就会被分组。每个拆分点默认生成一个单独的包。
The optional name allows you to pull multiple split points into a single bundle. As long as they have the same name, they will be grouped. Each split point generates a separate bundle by default.

界面允许组合，您可以并行加载多个资源：
The interface allows composition, and you could load multiple resources in parallel:

```javascript
Promise.all([
  import("lunr"),
  import("../search_index.json"),
]).then(([lunr, search]) => {
  return {
    index: lunr.Index.load(search.index),
    lines: search.lines,
  };
});
```

上面的代码为请求创建单独的包。如果只需要一个，则必须使用命名或定义中间模块来“导入”。
The code above creates separate bundles to a request. If you wanted only one, you would have to use naming or define an intermediate module to `import`.

W>在以正确的方式配置之后，语法仅适用于JavaScript。如果您使用其他环境，则可能必须使用以下各节中介绍的替代方案。
W> The syntax works only with JavaScript after configuring it the right way. If you use another environment you may have to use alternatives covered in the following sections.

T>有一个较旧的语法，[require.ensure]（https://webpack.js.org/api/module-methods/#require-ensure）。实际上，新语法可以涵盖相同的功能。另见[require.include]（https://webpack.js.org/api/module-methods/#require-include）。
T> There's an older syntax, [require.ensure](https://webpack.js.org/api/module-methods/#require-ensure). In practice the new syntax can cover the same functionality. See also [require.include](https://webpack.js.org/api/module-methods/#require-include).

T> [webpack-pwa]（https://github.com/webpack/webpack-pwa）更大范围地说明了这个想法，并讨论了不同的基于shell的方法。您可以在* Multiple Pages *一章中回到这个主题。
T> [webpack-pwa](https://github.com/webpack/webpack-pwa) illustrates the idea on a larger scale and discusses different shell based approaches. You get back to this topic in the *Multiple Pages* chapter.

{pagebreak}

##设置代码拆分
## Setting Up Code Splitting

为了演示代码拆分的想法，您可以使用动态`import`。项目的Babel设置需要添加以使语法有效。
To demonstrate the idea of code splitting, you can use dynamic `import`. The Babel setup of the project needs additions to make the syntax work.

###配置Babel
### Configuring Babel

鉴于Babel不支持开箱即用的动态`import`语法，它需要[babel-plugin-syntax-dynamic-import]（https://www.npmjs.com/package/babel-plugin-syntax-dynamic - 进口）工作。
Given Babel doesn't support the dynamic `import` syntax out of the box, it needs [babel-plugin-syntax-dynamic-import](https://www.npmjs.com/package/babel-plugin-syntax-dynamic-import) to work.

首先安装它：
Install it first:

```bash
npm install babel-plugin-syntax-dynamic-import --save-dev
```

要将其与项目连接，请按如下方式调整配置：
To connect it with the project, adjust the configuration as follows:

**。** babelrc
**.babelrc**

```json
{
leanpub-start-insert
  "plugins": ["syntax-dynamic-import"],
leanpub-end-insert
  ...
}
```

W>如果你正在使用ESLint，你应该在ESLint配置中安装`babel-eslint`并设置`parser：“babel-eslint”`以及`parserOptions.allowImportExportEverywhere：true`。
W> If you are using ESLint, you should install `babel-eslint` and set `parser: "babel-eslint"` in addition to `parserOptions.allowImportExportEverywhere: true` at ESLint configuration.

{pagebreak}

###使用动态`import`定义分割点
### Defining a Split Point Using a Dynamic `import`

可以通过设置一个包含替换演示按钮文本的字符串的模块来演示这个想法：
The idea can be demonstrated by setting up a module that contains a string that replaces the text of the demo button:

** SRC / lazy.js **
**src/lazy.js**

```javascript
export default "Hello from lazy";
```

您还需要将应用程序指向此文件，以便应用程序知道通过将加载过程绑定到单击来加载它。只要用户碰巧点击按钮，就会触发加载过程并替换内容：
You also need to point the application to this file, so the application knows to load it by binding the loading process to click. Whenever the user happens to click the button, you trigger the loading process and replace the content:

** SRC / component.js **
**src/component.js**

```javascript
export default (text = "Hello world") => {
  const element = document.createElement("div");

  element.className = "pure-button";
  element.innerHTML = text;
leanpub-start-insert
  element.onclick = () =>
    import("./lazy")
      .then(lazy => {
        element.textContent = lazy.default;
      })
      .catch(err => {
        console.error(err);
      });
leanpub-end-insert

  return element;
};
```

如果打开应用程序（`npm start`）并单击按钮，您应该会在按钮中看到新文本。
If you open up the application (`npm start`) and click the button, you should see the new text in the button.

![Lazy loaded content](images/lazy.png)

如果你运行`npm run build`，你会看到一些东西：
If you run `npm run build`, you should see something:

```bash
Hash: 063e54c36163f79e8c90
Version: webpack 4.1.1
Time: 3185ms
Built at: 3/16/2018 5:04:04 PM
               Asset       Size  Chunks             Chunk Names
leanpub-start-insert
            0.js.map  198 bytes       0  [emitted]
                0.js  156 bytes       0  [emitted]
leanpub-end-insert
             main.js    2.2 KiB       2  [emitted]  main
            main.css   1.27 KiB       2  [emitted]  main
    vendors~main.css   2.27 KiB       1  [emitted]  vendors~main
...
```

* 0.js *是你的分裂点。检查文件显示webpack已将代码包装在`webpackJsonp`块中并处理了代码位。
That *0.js* is your split point. Examining the file reveals that webpack has wrapped the code in a `webpackJsonp` block and processed the code bit.

T>如果要调整块的名称，请设置`output.chunkFilename`。例如，将其设置为“chunk。[id] .js”`将为每个拆分块添加单词“chunk”。
T> If you want to adjust the name of the chunk, set `output.chunkFilename`. For example, setting it to `"chunk.[id].js"` would prefix each split chunk with the word "chunk".

T> [bundle-loader]（https://www.npmjs.com/package/bundle-loader）给出了类似的结果，但是通过加载器接口。它通过`name`选项支持bundle命名。
T> [bundle-loader](https://www.npmjs.com/package/bundle-loader) gives similar results, but through a loader interface. It supports bundle naming through its `name` option.

T> *动态加载*章节介绍了当您必须处理更复杂的拆分时派上用场的其他技术。
T> The *Dynamic Loading* chapter covers other techniques that come in handy when you have to deal with more complicated splits.

{pagebreak}

##中的##代码拆分
## Code Splitting in React

拆分模式可以包装到React组件中。 Airbnb使用以下解决方案[如Joe Lencioni所述]（https://gist.github.com/lencioni/643a78712337d255f5c031bfc81ca4cf）：
The splitting pattern can be wrapped into a React component. Airbnb uses the following solution [as described by Joe Lencioni](https://gist.github.com/lencioni/643a78712337d255f5c031bfc81ca4cf):

```javascript
import React from "react";

// Somewhere in code
<AsyncComponent loader={() => import("./SomeComponent")} />

class AsyncComponent extends React.Component {
  constructor(props) {
    super(props);

    this.state = { Component: null };
  }
  componentDidMount() {
    this.props.loader().then(
      Component => this.setState({ Component })
    );
  }
  render() {
    const { Component } = this.state;
    const { Placeholder, ...props } = this.props;

    return Component ? <Component {...props} /> : <Placeholder />;
  }
}
AsyncComponent.propTypes = {
  loader: PropTypes.func.isRequired,
  Placeholder: PropTypes.node.isRequired,
};
```

T> [react-async-component]（https://www.npmjs.com/package/react-async-component）将模式包装在`createAsyncComponent`调用中，并提供服务器端呈现特定功能。 [loadable-components]（https://www.npmjs.com/package/loadable-components）是另一种选择。
T> [react-async-component](https://www.npmjs.com/package/react-async-component) wraps the pattern in a `createAsyncComponent` call and provides server side rendering specific functionality. [loadable-components](https://www.npmjs.com/package/loadable-components) is another option.

##禁用代码拆分
## Disabling Code Splitting

尽管代码拆分是默认情况下的良好行为，但它总是不正确，尤其是在服务器端使用时。因此，可以将其禁用如下：
Although code splitting is good behavior to have by default, it's not correct always, especially on server-side usage. For this reason, it can be disabled as below:

```javascript
const webpack = require("webpack");

...

module.exports = {
  plugins: [
    new webpack.optimize.LimitChunkCountPlugin({
      maxChunks: 1,
    }),
  ],
};
```

T>参见[Glenn Reyes的详细解释]（https://medium.com/@glennreyes/how-to-disable-code-splitting-in-webpack-1c0b1754a3c5）。
T> See [Glenn Reyes' detailed explanation](https://medium.com/@glennreyes/how-to-disable-code-splitting-in-webpack-1c0b1754a3c5).

{pagebreak}

##结论
## Conclusion

代码拆分是一项功能，可让您进一步推动应用程序。您可以在需要时加载代码，以获得更快的初始加载时间和改善的用户体验，尤其是在带宽有限的移动环境中。
Code splitting is a feature that allows you to push your application a notch further. You can load code when you need it to gain faster initial load times and improved user experience especially in a mobile context where bandwidth is limited.

回顾一下：
To recap:

* **代码拆分**需要额外的努力，因为你必须决定分割什么和在哪里。通常，您会在路由器中找到良好的分裂点。或者您注意到仅在使用特定功能时才需要特定功能。制图就是一个很好的例子。
* **Code splitting** comes with extra effort as you have to decide what to split and where. Often, you find good split points within a router. Or you notice that specific functionality is required only when a particular feature is used. Charting is an excellent example of this.
*要使用动态`import`语法，Babel和ESLint都需要仔细调整。 Webpack支持开箱即用的语法。
* To use dynamic `import` syntax, both Babel and ESLint require careful tweaks. Webpack supports the syntax out of the box.
*使用命名将单独的分割点拉入相同的包中。
* Use naming to pull separate split points into the same bundles.
*这些技术可以在现代框架和像React这样的库中使用。您可以将相关逻辑包装到以用户友好的方式处理加载过程的特定组件。
* The techniques can be used within modern frameworks and libraries like React. You can wrap related logic to a specific component that handles the loading process in a user-friendly manner.
*要禁用代码拆分，请将`webpack.optimize.LimitChunkCountPlugin`与`maxChunks`设置为1。
* To disable code splitting, use `webpack.optimize.LimitChunkCountPlugin` with `maxChunks` set to one.

您将学习在下一章中整理构建。
You'll learn to tidy up the build in the next chapter.

T> *使用React *搜索附录包含代码拆分的完整示例。它显示了如何设置用户搜索信息时加载的静态站点索引。
T> The *Searching with React* appendix contains a complete example of code splitting. It shows how to set up a static site index that's loaded when the user searches information.

