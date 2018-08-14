＃热模块更换
# Hot Module Replacement

**热模块更换**（HMR）建立在WDS之上。它启用了一个可以实时交换模块的接口。例如，* style-loader *可以在不强制刷新的情况下更新CSS。为样式实现HMR是理想的，因为CSS在设计上是无状态的。
**Hot Module Replacement** (HMR) builds on top of the WDS. It enables an interface that makes it possible to swap modules live. For example, *style-loader* can update your CSS without forcing a refresh. Implementing HMR for styles is ideal because CSS is stateless by design.

HMR也可以使用JavaScript，但由于应用程序状态，它更难。 [react-hot-loader]（https://github.com/gaearon/react-hot-loader/）和[vue-hot-reload-api]（https://www.npmjs.com/package/vue- hot-reload-api）就是很好的例子。
HMR is possible with JavaScript too, but due to application state, it's harder. [react-hot-loader](https://github.com/gaearon/react-hot-loader/) and [vue-hot-reload-api](https://www.npmjs.com/package/vue-hot-reload-api) are good examples.

T>鉴于HMR实现起来可能很复杂，一个很好的折衷方案是将应用程序状态存储到`localStorage`，然后在刷新后基于该程序对应用程序进行水合。这样做会将问题推向应用程序端。
T> Given HMR can be complex to implement, a good compromise is to store application state to `localStorage` and then hydrate the application based on that after a refresh. Doing this pushes the problem to the application side.

##启用HMR
## Enabling HMR

需要启用以下步骤才能使HMR正常工作：
The following steps need to be enabled for HMR to work:

1. WDS必须在热模式下运行才能将热模块替换接口暴露给客户端。
1. WDS has to run in the hot mode to expose the hot module replacement interface to the client.
2. Webpack必须为服务器提供热更新，并且可以使用`webpack.HotModuleReplacementPlugin`来实现。
2. Webpack has to provide hot updates to the server and can be achieved using `webpack.HotModuleReplacementPlugin`.
3.客户端必须运行WDS提供的特定脚本。它们将自动注入，但可以通过输入配置显式启用。
3. The client has to run specific scripts provided by the WDS. They will be injected automatically but can be enabled explicitly through entry configuration.
4.客户端必须通过`module.hot.accept`实现HMR接口。
4. The client has to implement the HMR interface through `module.hot.accept`.

使用`webpack-dev-server --hot`解决了前两个问题。在这种情况下，如果要修补JavaScript应用程序代码，则必须自己处理最后一个。跳过`--hot`标志并通过webpack配置可以提供更大的灵活性。
Using `webpack-dev-server --hot` solves the first two problems. In this case, you have to handle only the last one yourself if you want to patch JavaScript application code. Skipping the `--hot` flag and going through webpack configuration gives more flexibility.

以下清单包含与此方法相关的基本部分。你必须从此处进行调整以匹配你的配置样式：
The following listing contains the essential parts related to this approach. You will have to adapt from here to match your configuration style:

```javascript
{
  devServer: {
    // Don't refresh if hot loading fails. Good while
    // implementing the client interface.
    hotOnly: true,

    // If you want to refresh on errors too, set
    // hot: true,
  },
  plugins: [
    // Enable the plugin to let webpack communicate changes
    // to WDS. --hot sets this automatically!
    new webpack.HotModuleReplacementPlugin(),
  ],
}
```

{pagebreak}

如果在不实现客户端界面的情况下实现上述配置，则很可能最终会出现错误：
If you implement configuration like above without implementing the client interface, you will most likely end up with an error:

![No refresh](images/no-refresh2.png)

该消息告诉我即使HMR接口通知客户端部分热更新的代码，也没有做任何事情，这是接下来要解决的问题。
The message tells that even though the HMR interface notified the client portion of the code of a hot update, nothing was done about it and this is something to fix next.

T>设置假设你已启用`webpack.NamedModulesPlugin（）`。如果你在`development`模式下运行webpack，默认情况下它将处于打开状态。
T> The setup assumes you have enabled `webpack.NamedModulesPlugin()`. If you run webpack in `development` mode, it will be on by default.

W> * webpack-dev-server *可以挑剔路径。 Webpack [问题＃675]（https://github.com/webpack/webpack/issues/675）更详细地讨论了这个问题。
W> *webpack-dev-server* can be picky about paths. Webpack [issue #675](https://github.com/webpack/webpack/issues/675) discusses the problem in more detail.

W>你应该**不要**为你的生产配置启用HMR。它可能有效，但它使你的捆绑比它们应该更重要。
W> You should **not** enable HMR for your production configuration. It likely works, but it makes your bundles more significant than they should be.

W>如果你使用的是Babel，请将其配置为允许webpack控制模块生成，否则HMR逻辑将无法正常工作！
W> If you are using Babel, configure it so that it lets webpack control module generation as otherwise, HMR logic won't work!

##实现HMR接口
## Implementing the HMR Interface

Webpack通过全局变量公开HMR接口：`module.hot`。它通过`module.hot.accept（<path to watch>，<handler>）`函数提供更新，你需要在那里修补应用程序。
Webpack exposes the HMR interface through a global variable: `module.hot`. It provides updates through `module.hot.accept(<path to watch>, <handler>)` function and you need to patch the application there.

以下实现说明了针对教程应用程序的想法：
The following implementation illustrates the idea against the tutorial application:

** SRC / index.js **
**src/index.js**

```javascript
import component from "./component";

let demoComponent = component();

document.body.appendChild(demoComponent);

// HMR interface
if (module.hot) {
  // Capture hot update
  module.hot.accept("./component", () => {
    const nextComponent = component();

    // Replace old content with the hot loaded one
    document.body.replaceChild(nextComponent, demoComponent);

    demoComponent = nextComponent;
  });
}
```

如果刷新浏览器，尝试在此更改后修改* src / component.js *，并将文本更改为其他内容，你应该注意到浏览器根本不刷新。相反，它应该替换DOM节点，同时保留应用程序的其余部分。
If you refresh the browser, try to modify *src/component.js* after this change, and alter the text to something else, you should notice that the browser does not refresh at all. Instead, it should replace the DOM node while retaining the rest of the application as is.

{pagebreak}

下图显示了可能的输出：
The image below shows possible output:

![Patched a module successfully through HMR](images/hmr.png)

这个想法与造型，React，Redux和其他技术相同。有时你不必自己实现界面，即使可用的工具为你处理。
The idea is the same with styling, React, Redux, and other technologies. Sometimes you don't have to implement the interface yourself even as available tooling takes care of that for you.

T>要证明HMR保留应用程序状态，请在原始文件旁边设置[复选框]（https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/checkbox）组件。 `module.hot.accept`代码必须发展以捕获对它的更改。
T> To prove that HMR retains application state, set up [a checkbox](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/checkbox) based component next to the original. The `module.hot.accept` code has to evolve to capture changes to it as well.

T>`if（module.hot）`块完全从生产版本中删除，因为minifier选择了它。 * Minifying *章节深入探讨了这个主题。
T> The `if(module.hot)` block is eliminated entirely from the production build as minifier picks it up. The *Minifying* chapter delves deeper into this topic.

{pagebreak}

##手动设置WDS入口点
## Setting WDS Entry Points Manually

在上面的设置中，自动注入与WDS相关的条目。假设你正在通过Node使用WDS，你必须自己设置它们，因为Node API不支持注入。以下示例说明了如何实现此目的：
In the setup above, the WDS-related entries were injected automatically. Assuming you are using WDS through Node, you would have to set them yourself as the Node API doesn't support injecting. The example below illustrates how to achieve this:

```javascript
entry: {
  hmr: [
    // Include the client code. Note host/post.
    "webpack-dev-server/client?http://localhost:8080",

    // Hot reload only when compiled successfully
    "webpack/hot/only-dev-server",

    // Alternative with refresh on failure
    // "webpack/hot/dev-server",
  ],
  ...
},
```

## HMR和动态加载
## HMR and Dynamic Loading

*动态加载*通过`require.context`和HMR需要额外的努力：
*Dynamic Loading* through `require.context` and HMR requires extra effort:

```javascript
const req = require.context("./pages", true, /^(.*\.(jsx$))[^.]*$/g);

module.hot.accept(req.id, ...); // Replace modules here as above
```

## 总结


HMR是webpack的一个方面，使其对开发人员具有吸引力，而webpack已经将其实现了很多。要工作，HMR需要客户端和服务器端支持。为此，webpack-dev-server提供了这两者。通常你必须实现客户端接口，尽管像* style-loader *这样的加载器会为你实现它。
HMR is one of those aspects of webpack that makes it attractive for developers and webpack has taken its implementation far. To work, HMR requires both client and server side support. For this purpose, webpack-dev-server provides both. Often you have to implement the client side interface although loaders like *style-loader* implement it for you.

