#Server Side Rendering
# Server Side Rendering

**服务器端呈现**（SSR）是一种允许你使用HTML，JavaScript，CSS甚至应用程序状态提供初始有效负载的技术。即使没有启用JavaScript，你也可以提供完全呈现的HTML页面。除了提供潜在的性能优势外，这还有助于搜索引擎优化（SEO）。
**Server Side Rendering** (SSR) is a technique that allows you to serve an initial payload with HTML, JavaScript, CSS, and even application state. You serve a fully rendered HTML page that would make sense even without JavaScript enabled. In addition to providing potential performance benefits, this can help with Search Engine Optimization (SEO).

即使这个想法听起来不那么独特，但也存在技术成本。这种方法由React推广。从那以后框架封装了棘手的位，例如[Next.js]（https://www.npmjs.com/package/next）和[razzle]（https://www.npmjs.com/package/razzle），出现了。
Even though the idea does not sound that unique, there is a technical cost. The approach was popularized by React. Since then frameworks encapsulating the tricky bits, such as [Next.js](https://www.npmjs.com/package/next) and [razzle](https://www.npmjs.com/package/razzle), have appeared.

为了演示SSR，你可以使用webpack编译客户端构建，然后由服务器接收，该服务器遵循原则使用React呈现它。这样做足以了解它是如何工作的以及问题的开始。
To demonstrate SSR, you can use webpack to compile a client-side build that then gets picked up by a server that renders it using React following the principle. Doing this is enough to understand how it works and also where the problems begin.

T> SSR不是SEO问题的唯一解决方案。 ** Prerendering **是一种替代技术，如果它适合你的用例，更容易实现。这种方法不适用于高动态数据。 [prerender-spa-plugin]（https://www.npmjs.com/package/prerender-spa-plugin）允许你使用webpack实现它。
T> SSR isn't the only solution to the SEO problem. **Prerendering** is an alternate technique that is easier to implement if it fits your use case. The approach won't work well with highly dynamic data. [prerender-spa-plugin](https://www.npmjs.com/package/prerender-spa-plugin) allows you to implement it with webpack.

##使用React设置Babel
## Setting Up Babel with React

* Loading JavaScript *章节介绍了使用Babel和webpack的基本要点。不过，你应该执行React特有的设置。鉴于大多数React项目依赖于[JSX]（https://facebook.github.io/jsx/）格式，你必须通过Babel启用它。
The *Loading JavaScript* chapter covers the essentials of using Babel with webpack. There's setup that is particular to React you should perform, though. Given most of React projects rely on [JSX](https://facebook.github.io/jsx/) format, you have to enable it through Babel.

{pagebreak}

要让React，特别是JSX与Babel一起工作，首先安装预设：
To get React, and particularly JSX, work with Babel, install the preset first:

```bash
npm install babel-preset-react --save-dev
```

使用Babel配置连接预设，如下所示：
Connect the preset with Babel configuration as follows:

**。** babelrc
**.babelrc**

```json
{
  ...
  "presets": [
leanpub-start-insert
    "react",
leanpub-end-insert
    ...
  ]
}
```

##设置React演示
## Setting Up a React Demo

要确保项目具有相关性，请安装React和[react-dom]（https://www.npmjs.com/package/react-dom）。需要后一个包来将应用程序呈现给DOM。
To make sure the project has the dependencies in place, install React and [react-dom](https://www.npmjs.com/package/react-dom). The latter package is needed to render the application to the DOM.

```bash
npm install react react-dom --save
```

接下来，React代码需要一个小入口点。如果你在浏览器端，则应将“Hello world``div`挂载到文档中。要证明它有效，单击它应该给出一个带有“hello”消息的对话框。在服务器端，返回React组件，服务器可以接收它。
Next, the React code needs a small entry point. If you are on the browser side, you should mount `Hello world` `div` to the document. To prove it works, clicking it should give a dialog with a "hello" message. On server-side, the React component is returned and the server can pick it up.

{pagebreak}

调整如下：
Adjust as follows:

** SRC / ssr.js **
**src/ssr.js**

```javascript
const React = require("react");
const ReactDOM = require("react-dom");

const SSR = <div onClick={() => alert("hello")}>Hello world</div>;

// Render only in the browser, export otherwise
if (typeof document === "undefined") {
  module.exports = SSR;
} else {
  ReactDOM.hydrate(SSR, document.getElementById("app"));
}
```

你仍然缺少webpack配置来将此文件转换为服务器可以接收的内容。
You are still missing webpack configuration to turn this file into something the server can pick up.

W>鉴于ES2015样式导入和CommonJS导出不能混合，入口点是用CommonJS样式编写的。
W> Given ES2015 style imports and CommonJS exports cannot be mixed, the entry point was written in CommonJS style.

{pagebreak}

##配置Webpack
## Configuring Webpack

为了保持良好，我们将定义一个单独的配置文件。已经完成了很多工作。鉴于你必须使用来自多个环境的相同输出，使用UMD作为库目标是有意义的：
To keep things nice, we will define a separate configuration file. A lot of the work has been done already. Given you have to consume the same output from multiple environments, using UMD as the library target makes sense:

** ** webpack.ssr.js
**webpack.ssr.js**

```javascript
const path = require("path");
const merge = require("webpack-merge");

const parts = require("./webpack.parts");

const PATHS = {
  build: path.join(__dirname, "static"),
  ssrDemo: path.join(__dirname, "src", "ssr.js"),
};

module.exports = merge([
  {
    mode: "production",
    entry: {
      index: PATHS.ssrDemo,
    },
    output: {
      path: PATHS.build,
      filename: "[name].js",
      libraryTarget: "umd",
      globalObject: "this",
    },
  },
  parts.loadJavaScript({ include: PATHS.ssrDemo }),
]);
```

{pagebreak}

为了方便生成构建，请添加一个帮助脚本：
To make it convenient to generate a build, add a helper script:

** **的package.json
**package.json**

```json
"scripts": {
leanpub-start-insert
  "build:ssr": "webpack --config webpack.ssr.js",
leanpub-end-insert
  ...
},
```

如果你构建了SSR演示（`npm run build：ssr`），你应该在*。/ static / index.js *上看到一个新文件。下一步是设置服务器来呈现它。
If you build the SSR demo (`npm run build:ssr`), you should see a new file at *./static/index.js*. The next step is to set up a server to render it.

##设置服务器
## Setting Up a Server

为了清楚地理解，你可以设置一个独立的Express服务器来获取生成的捆绑包并按照SSR原则呈现它。首先安装Express：
To keep things clear to understand, you can set up a standalone Express server that picks up the generated bundle and renders it following the SSR principle. Install Express first:

```bash
npm install express --save-dev
```

然后，为了运行某些东西，按如下方式实现服务器：
Then, to get something running, implement a server as follows:

** ** server.js
**server.js**

```javascript
const express = require("express");
const { renderToString } = require("react-dom/server");

const SSR = require("./static");

server(process.env.PORT || 8080);

function server(port) {
  const app = express();

  app.use(express.static("static"));
  app.get("/", (req, res) =>
    res.status(200).send(renderMarkup(renderToString(SSR)))
  );

  app.listen(port);
}

function renderMarkup(html) {
  return `<!DOCTYPE html>
<html>
  <head>
    <title>Webpack SSR Demo</title>
    <meta charset="utf-8" />
  </head>
  <body>
    <div id="app">${html}</div>
    <script src="./index.js"></script>
  </body>
</html>`;
}
```

现在运行服务器（`node。/ server.js`）并在`http：// localhost：8080`下面，你会看到熟悉的东西：
Run the server now (`node ./server.js`) and go below `http://localhost:8080`, you should see something familiar:

![Hello world](images/hello_01.png)

{pagebreak}

即使现在运行React应用程序，也很难开发。如果你尝试修改代码，则不会发生任何事情。如本书前面所述，在多编译器模式下运行webpack可以解决该问题。另一种选择是针对当前配置以**监视模式**运行webpack，并为服务器设置监视器。你将接下来学习设置。
Even though there is a React application running now, it's difficult to develop. If you try to modify the code, nothing happens. The problem can be solved running webpack in a multi-compiler mode as earlier in this book. Another option is to run webpack in **watch mode** against the current configuration and set up a watcher for the server. You'll learn the setup next.

T>如果要调试服务器的输出，请设置`export DEBUG = express：application`。
T> If you want to debug output from the server, set `export DEBUG=express:application`.

T>如果你按照* Separating a Manifest *章节中的说明编写清单，则可以将webpack生成的资产的引用自动写入服务器端模板。
T> The references to the assets generated by webpack could be written automatically to the server side template if you wrote a manifest as discussed in the *Separating a Manifest* chapter.

##观看SSR更改并刷新浏览器
## Watching SSR Changes and Refreshing the Browser

问题的第一部分很快就能解决。在终端中运行`npm run build：ssr  -  --watch`。这迫使webpack以监视模式运行。为方便起见，可以将这个想法包装在npm脚本中，但这对于这个演示来说已经足够了。
The first portion of the problem is fast to solve. Run `npm run build:ssr -- --watch` in a terminal. That forces webpack to run in a watch mode. It would be possible to wrap this idea within an npm script for convenience, but this is enough for this demo.

剩下的部分比到目前为止所做的更难。如何让服务器知道更改以及如何将更改传达给浏览器？
The remaining part is harder than what was done so far. How to make the server aware of the changes and how to communicate the changes to the browser?

[browser-refresh]（https://www.npmjs.com/package/browser-refresh）可以派上用场，因为它解决了这两个问题。首先安装它：
[browser-refresh](https://www.npmjs.com/package/browser-refresh) can come in handy as it solves both of the problems. Install it first:

```bash
npm install browser-refresh --save-dev
```

{pagebreak}

客户端部分需要对服务器代码进行两处小的更改：
The client portion requires two small changes to the server code:

** ** server.js
**server.js**

```javascript
server(process.env.PORT || 8080);

function server(port) {
  ...

leanpub-start-delete
  app.listen(port);
leanpub-end-delete
leanpub-start-insert
  app.listen(port, () => process.send && process.send("online"));
leanpub-end-insert
}

function renderMarkup(html) {
  return `<!DOCTYPE html>
<html>
  ...
  <body>
    ...
leanpub-start-insert
    <script src="${process.env.BROWSER_REFRESH_URL}"></script>
leanpub-end-insert
  </body>
</html>`;
}
```

第一个更改告诉客户端应用程序已联机并准备就绪。后一种更改将客户端脚本附加到输出。 * browser-refresh *管理有问题的环境变量。
The first change tells the client that the application is online and ready to go. The latter change attaches the client script to the output. *browser-refresh* manages the environment variable in question.

在另一个终端中运行`node_modules / .bin / browser-refresh。/ server.js`并在之前打开浏览器`http：// localhost：8080`来测试设置。记得让webpack在另一个终端的监视模式下运行（`npm run build：ssr  -  --watch`）。如果一切正常，你对演示客户端脚本（* app / ssr.js *）所做的任何更改都应显示在浏览器中或导致服务器出现故障。
Run `node_modules/.bin/browser-refresh ./server.js` in another terminal and open the browser at `http://localhost:8080` as earlier to test the setup. Remember to have webpack running in the watch mode at another terminal (`npm run build:ssr -- --watch`). If everything went right, any change you make to the demo client script (*app/ssr.js*) should show up in the browser or cause a failure at the server.

如果服务器崩溃，则会丢失WebSocket连接。在这种情况下，你必须在浏览器中强制刷新。如果服务器也是通过webpack管理的，那么问题本来就可以避免。
If the server crashes, it loses the WebSocket connection. You have to force a refresh in the browser in this case. If the server was managed through webpack as well, the problem could have been avoided.

{pagebreak}

要证明SSR有效，请查看浏览器检查器。你应该看到熟悉的东西：
To prove that SSR works, check out the browser inspector. You should see something familiar there:

![SSR output](images/ssr.png)

而不是“div”在哪里安装应用程序，你可以在那里看到所有相关的HTML。在这种特殊情况下并不多，但它足以展示这种方法。
Instead of a `div` where to mount an application, you can see all related HTML there. It's not much in this particular case, but it's enough to showcase the approach.

T>可以通过为服务器实现生产模式来进一步细化实现，该模式将至少跳过注入浏览器刷新脚本。服务器可以将初始数据有效负载注入生成的HTML中。这样做可以避免客户端的查询。
T> The implementation could be refined further by implementing a production mode for the server that would skip injecting the browser refresh script at a minimum. The server could inject initial data payload into the generated HTML. Doing this would avoid queries on the client-side.

##开放式问题
## Open Questions

尽管该演示演示了SSR的基本概念，但它仍然存在悬而未决的问题：
Even though the demo illustrates the basic idea of SSR, it still leaves open questions:

*如何处理款式？ Node不了解CSS相关的导入。
* How to deal with styles? Node doesn't understand CSS related imports.
*如何处理除JavaScript之外的任何其他内容？如果服务器端是通过webpack处理的，那么这不是一个问题，因为你可以在webpack上修补它。
* How to deal with anything other than JavaScript? If the server side is processed through webpack, this is less of an issue as you can patch it at webpack.
*如何通过Node之外的其他东西运行服务器？一种选择是将Node实例包装在服务中，然后通过主机环境运行。理想情况下，结果将被缓存，你可以为每个平台找到更具体的解决方案。
* How to run the server through something else other than Node? One option would be to wrap the Node instance in a service you then run through your host environment. Ideally, the results would be cached, and you can find more specific solutions for this particular per platform.

像这样的问题是Next.js或razzle等解决方案存在的原因。它们旨在解决这些特定于SSR的问题。
Questions like these are the reason why solutions such as Next.js or razzle exist. They have been designed to solve SSR-specific problems like these.

T>路由是一个很大的问题，它自己解决了像Next.js这样的框架。 Patrick Hund [讨论如何使用React和React Router 4解决它]（https://ebaytech.berlin/universal-web-apps-with-react-router-4-15002bb30ccb）。
T> Routing is a big problem of its own solved by frameworks like Next.js. Patrick Hund [discusses how to solve it with React and React Router 4](https://ebaytech.berlin/universal-web-apps-with-react-router-4-15002bb30ccb).

##结论
## Conclusion

SSR带来了技术挑战，因此，围绕它出现了特定的解决方案。 Webpack非常适合SSR设置。
SSR comes with a technical challenge, and for this reason, specific solutions have appeared around it. Webpack is a good fit for SSR setups.

回顾一下：
To recap:

* **服务器端渲染**可以为浏览器提供更多初始渲染。你可以立即显示标记，而不是等待JavaScript加载。
* **Server Side Rendering** can provide more for the browser to render initially. Instead of waiting for the JavaScript to load, you can display markup instantly.
*服务器端呈现还允许你将初始数据有效负载传递到客户端，以避免对服务器进行不必要的查询。
* Server Side Rendering also allows you to pass initial payload of data to the client to avoid unnecessary queries to the server.
* Webpack可以管理问题的客户端部分。如果需要更集成的解决方案，它也可用于生成服务器。诸如Next.js之类的抽象隐藏了这些细节。
* Webpack can manage the client-side portion of the problem. It can be used to generate the server as well if a more integrated solution is required. Abstractions, such as Next.js, hide these details.
*服务器端渲染不是没有成本的，并且它会导致新的问题，因为你需要更好的方法来处理方面，例如样式或路由。服务器和客户端环境的基本方式不同，因此必须编写代码，以使其不依赖于特定于平台的功能。
* Server Side Rendering does not come without a cost, and it leads to new problems as you need better approaches for dealing with aspects, such as styling or routing. The server and the client environment differ in essential manners, so the code has to be written so that it does not rely on platform-specific features too much.

