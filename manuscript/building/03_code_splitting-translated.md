# Code Splitting

随着功能的发展，Web 应用程序往往会变得越来越大。应用程序加载所需的时间越长，用户体验越差。在连接速度较慢的移动环境中，此问题更会被放大。

即使 Bundle 拆分可以在一定程度上缓解这个问题，但这不是唯一的解决方案，你始终要把大量的数据都下载完。幸运的是，有**代码拆分**，它允许你在需要时懒加载代码。

你可以在用户进入新页面时加载更多代码。还可以将加载绑定到特定操作，例如滚动或单击按钮。你还可以尝试预测用户下一步要做什么，预先加载相关代码。这样，当用户尝试访问它时，功能就已存在。

T> 顺便说一下，使用 webpack 的延迟加载可以实现 Google 的 [PRPL模式](https://developers.google.com/web/fundamentals/performance/prpl-pattern/)。PRPL (Push, Render, Pre-cache, Lazy-load)。 PRPL（推送，渲染，预缓存，延迟加载）为移动网络设计。

## 代码拆分格式

Webpack 代码拆分主要以两种主要方式完成：通过动态 `import` 或 `require.ensure` 语法。前者用于此项目。

目标是最终得到一个按需加载的拆分点。拆分中也可以有拆分，甚至可以根据拆分构建整个应用程序。这样做的好处是，你的应用程序的初始加载会更小。

![Code splitting](images/code-splitting.png)

### 动态 `import`

[动态 `import` 语法](https://github.com/tc39/proposal-dynamic-import) 尚未在官方语言规范中。由于这个原因，特别是在 Babel 设置中需要进行一点调整。

动态导入定义为 `Promise`：

```javascript
import(/* webpackChunkName: "optional-name" */ "./module").then(
  module => {...}
).catch(
  error => {...}
);
```

{pagebreak}

可选名称允许你将多个拆分点拉入单个 bundle 中。只要它们具有相同的名称，它们就会被分组。每个拆分点默认生成一个单独的包。

接口允许组合，你可以并行加载多个资源：

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

上面的代码为请求创建单独的 bundle。如果只需要一个，则必须使用命名或定义中间模块来 `import`。

W> 在以正确的方式配置之后，语法仅适用于 JavaScript。如果你使用其他环境，则可能要使用以下各节中介绍的替代方案。

T> 有一个较旧的语法，[require.ensure](https://webpack.js.org/api/module-methods/#require-ensure)。实际上，新语法可以涵盖相同的功能。另见[require.include](https://webpack.js.org/api/module-methods/#require-include)。

T> [webpack-pwa](https://github.com/webpack/webpack-pwa) 更详尽地说明了这个想法，并讨论了基于不同 shell 的方法。你可以在 *Multiple Pages* 一章中再次探讨这个主题。

{pagebreak}

## 设置代码拆分

为了演示代码拆分的思路，你可以使用动态 `import`。注意，需要在项目中添加 Babel 设置以使语法有效。

### 配置 Babel

鉴于 Babel 不支持开箱即用的动态 `import` 语法，需要借助 [babel-plugin-syntax-dynamic-import](https://www.npmjs.com/package/babel-plugin-syntax-dynamic-import)。

首先安装插件：

```bash
npm install babel-plugin-syntax-dynamic-import --save-dev
```

要将其与项目连接，请按如下方式调整配置：

**.babelrc**

```json
{
leanpub-start-insert
  "plugins": ["syntax-dynamic-import"],
leanpub-end-insert
  ...
}
```

W> 如果你正在使用ESLint，你应该在ESLint配置中安装`babel-eslint`并设置`parser：“babel-eslint”`以及`parserOptions.allowImportExportEverywhere：true`。
W> If you are using ESLint, you should install `babel-eslint` and set `parser: "babel-eslint"` in addition to `parserOptions.allowImportExportEverywhere: true` at ESLint configuration.

{pagebreak}

### 使用动态 `import` 定义分割点

可以通过设置一个包含替换演示按钮文本的字符串的模块来演示这个想法：
The idea can be demonstrated by setting up a module that contains a string that replaces the text of the demo button:

**src/lazy.js**

```javascript
export default "Hello from lazy";
```

你还需要将应用程序指向此文件，以便应用程序知道通过绑定单击来加载它。只要用户点击按钮，就会触发加载过程并替换内容：

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

如果启动应用程序（`npm start`）并单击按钮，你应该会在按钮中看到更改后的文本。

![Lazy loaded content](images/lazy.png)

如果你运行 `npm run build`，你会看到这些东西：

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

*0.js* 是你的拆分点。打开该文件可以看到，webpack 已将代码包装在 `webpackJsonp` 块中并处理了代码位（code bit）。

T> 如果要调整块的名称，请设置`output.chunkFilename`。例如，将其设置为“chunk。[id] .js”`将为每个拆分块添加单词“chunk”。
T> If you want to adjust the name of the chunk, set `output.chunkFilename`. For example, setting it to `"chunk.[id].js"` would prefix each split chunk with the word "chunk".

T> [bundle-loader]（https://www.npmjs.com/package/bundle-loader）给出了类似的结果，但是通过加载器接口。它通过`name`选项支持bundle命名。
T> [bundle-loader](https://www.npmjs.com/package/bundle-loader) gives similar results, but through a loader interface. It supports bundle naming through its `name` option.

T> **动态加载**章节介绍了当你必须处理更复杂的拆分时可以派上用场的其他技术。

{pagebreak}

## React 中的代码拆分

拆分模式可以包装到 React 组件中。 Airbnb使用 [Joe Lencioni 提出](https://gist.github.com/lencioni/643a78712337d255f5c031bfc81ca4cf)的以下解决方案：

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

## 禁用代码拆分

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

## 结论

代码拆分是一项功能，可让你进一步推动应用程序。你可以在需要时加载代码，以获得更快的初始加载时间和改善的用户体验，尤其是在带宽有限的移动环境中。
Code splitting is a feature that allows you to push your application a notch further. You can load code when you need it to gain faster initial load times and improved user experience especially in a mobile context where bandwidth is limited.

回顾一下：

* **代码拆分**需要花费一番功夫，你必须决定拆分什么和在哪里拆分。通常，你会在路由器中找到良好的拆分点。或者你注意到仅在使用特定情况才需要特定功能。绘制图表就是一个很好的例子。
* 要使用动态 `import` 语法，需要先配置 Babel 和 ESLint都。Webpack 支持开箱即用的语法。
* 使用命名将单独的拆分点加入相同的 bundle 中。
* 这些技术可以在现代框架 React 这样的库中使用。你可以将相关逻辑包装到的特定组件，以用户友好的方式进行加载。
* 要禁用代码拆分，请将 `webpack.optimize.LimitChunkCountPlugin` 的 `maxChunks` 设置为1。

你将在下一章中学习如何整理构建。

T> 在 *Searching with React* 附录中包含代码拆分的完整示例。它展示了如何写一个用户搜索信息时加载的静态站点。

