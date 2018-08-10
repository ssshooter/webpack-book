＃构建目标
# Build Targets

尽管webpack最常用于捆绑Web应用程序，但它可以做得更多。你可以使用它来定位节点或桌面环境，例如Electron。 Webpack还可以捆绑为库，同时编写适当的输出包装器，从而可以使用库。
Even though webpack is used most commonly for bundling web applications, it can do more. You can use it to target Node or desktop environments, such as Electron. Webpack can also bundle as a library while writing an appropriate output wrapper making it possible to consume the library.

Webpack的输出目标由`target`字段控制。你将了解下一个主要目标，然后深入了解库特定选项。
Webpack's output target is controlled by the `target` field. You'll learn about the primary targets next and dig into library specific options after that.

## Web Targets
## Web Targets

Webpack默认使用* web *目标。该目标非常适合你在本书中开发的Web应用程序。 Webpack引导应用程序并加载其模块。要加载的模块的初始列表在清单中维护，然后模块可以按照定义相互加载。
Webpack uses the *web* target by default. The target is ideal for a web application like the one you have developed in this book. Webpack bootstraps the application and loads its modules. The initial list of modules to load is maintained in a manifest, and then the modules can load each other as defined.

### Web Workers
### Web Workers

* webworker *目标将你的应用程序包装为[Web worker]（https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API）。如果要在应用程序的主线程之外执行计算而不降低用户界面的速度，则使用Web worker非常有用。你应该注意以下几个限制：
The *webworker* target wraps your application as a [web worker](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API). Using web workers is valuable if you want to execute computation outside of the main thread of the application without slowing down the user interface. There are a couple of limitations you should be aware of:

*使用* webworker *目标时，无法使用webpack的散列功能。
* You cannot use webpack's hashing features when the *webworker* target is used.
*你无法操纵Web工作者的DOM。如果你将图书项目包装为工人，则不会显示任何内容。
* You cannot manipulate the DOM from a web worker. If you wrapped the book project as a worker, it would not display anything.

T> Web Worker及其用法将在* Web Workers *章节中详细讨论。
T> Web workers and their usage are discussed in detail in the *Web Workers* chapter.

##节点目标
## Node Targets

Webpack提供了两个特定于节点的目标：`node`和`async-node`。除非使用异步模式，否则它使用标准节点`require`来加载块。在这种情况下，它包装模块，以便通过Node`fs`和`vm`模块异步加载它们。
Webpack provides two Node-specific targets: `node` and `async-node`. It uses standard Node `require` to load chunks unless async mode is used. In that case, it wraps modules so that they are loaded asynchronously through Node `fs` and `vm` modules.

使用Node目标的主要用例是* Server Side Rendering *（SSR）。这个想法在* Server Side Rendering *章节中讨论。
The main use case for using the Node target is *Server Side Rendering* (SSR). The idea is discussed in the *Server Side Rendering* chapter.

##桌面目标
## Desktop Targets

有桌面shell，例如[NW.js]（https://nwjs.io/）（以前是* node-webkit *）和[Electron]（http://electron.atom.io/）（以前是* Atom） *）。 Webpack可以如下定位：
There are desktop shells, such as [NW.js](https://nwjs.io/) (previously *node-webkit*) and [Electron](http://electron.atom.io/) (previously *Atom*). Webpack can target these as follows:

*`node-webkit`  - 在考虑实验的情况下瞄准NW.js。
* `node-webkit` - Targets NW.js while considered experimental.
*`atom`，`electron`，`electron-main`  - 目标[电子主过程]（https://github.com/electron/electron/blob/master/docs/tutorial/quick-start.md）。
* `atom`, `electron`, `electron-main` - Targets [Electron main process](https://github.com/electron/electron/blob/master/docs/tutorial/quick-start.md).
*`electron-renderer`  - 目标电子渲染器过程。
* `electron-renderer` - Targets Electron renderer process.

[electron-react-boilerplate]（https://github.com/chentsulin/electron-react-boilerplate）是一个很好的起点，如果你想为基于Electron和React的开发热装webpack设置。使用[Electron的官方快速入门]（https://github.com/electron/electron-quick-start）是一种方法。
[electron-react-boilerplate](https://github.com/chentsulin/electron-react-boilerplate) is a good starting point if you want hot loading webpack setup for Electron and React based development. Using [the official quick start for Electron](https://github.com/electron/electron-quick-start) is one way.

{pagebreak}

##结论
## Conclusion

Webpack支持Web以外的目标。基于此，你可以说名称“webpack”考虑其功能是轻描淡写。
Webpack supports targets beyond the web. Based on this you can say name "webpack" is an understatement considering its capabilities.

回顾一下：
To recap:

* Webpack的输出目标可以通过`target`字段控制。它默认为“web”，但也接受其他选项。
* Webpack's output target can be controlled through the `target` field. It defaults to `web` but accepts other options too.
*除了Web目标之外，Webpack还可以定位桌面，节点和Web工作者。
* Webpack can target the desktop, Node, and web workers in addition to its web target.
*如果特别是在服务器端渲染设置中，节点目标会派上用场。
* The Node targets come in handy if especially in Server Side Rendering setups.

你将在下一章学习如何处理多页设置。
You'll learn how to handle multi-page setups in the next chapter.

