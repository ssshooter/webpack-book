# 扩展 Loader

正如你所看到的那样，加载器是webpack的构建块之一。如果要加载资产，则很可能需要设置匹配的加载程序定义。虽然有很多[可用的装载机]（https://webpack.js.org/loaders/），但是你可能错过了一个符合你目的的装载机。
As you have seen so far, loaders are one of the building blocks of webpack. If you want to load an asset, you most likely need to set up a matching loader definition. Even though there are a lot of [available loaders](https://webpack.js.org/loaders/), it's possible you are missing one fitting your purposes.

接下来你将学会开发几个小型装载机。但在此之前，了解如何单独调试它们是很好的。
You'll learn to develop a couple of small loaders next. But before that, it's good to understand how to debug them in isolation.

T>如果你想要一个独立的加载器或插件项目的良好起点，请考虑使用[webpack-defaults]（https://github.com/webpack-contrib/webpack-defaults）。它提供了一个固有的起点，包括linting，测试和其他好东西。
T> If you want a good starting point for a standalone loader or plugin project, consider using [webpack-defaults](https://github.com/webpack-contrib/webpack-defaults). It provides an opinionated starting point that comes with linting, testing, and other goodies.

## 使用 *loader-runner* 调试 Loader

[loader-runner](https://www.npmjs.com/package/loader-runner) 可以让你在没有 webpack 的情况下运行 loader，有了它你可以更方便地进行 loader 开发。首先安装它：

```bash
npm install loader-runner --save-dev
```

{pagebreak}

要测试一些东西，设置一个返回传递给它的两倍的加载器：
To have something to test with, set up a loader that returns twice what's passed to it:

**loaders/demo-loader.js**

```javascript
module.exports = input => input + input;
```

准备待处理文件：

**demo.txt**

```
foobar
```

代码中暂时没有任何 webpack 规范。下一步是通过 *loader-runner* 运行 loader：

**run-loader.js**

```javascript
const fs = require("fs");
const path = require("path");
const { runLoaders } = require("loader-runner");

runLoaders(
  {
    resource: "./demo.txt",
    loaders: [path.resolve(__dirname, "./loaders/demo-loader")],
    readResource: fs.readFile.bind(fs),
  },
  (err, result) => (err ? console.error(err) : console.log(result))
);
```

{pagebreak}

如果你现在运行脚本（`node run-loader.js`），你应该可以看到输出：

```javascript
{ result: [ 'foobar\nfoobar\n' ],
  resourceBuffer: <Buffer 66 6f 6f 62 61 72 0a>,
  cacheable: true,
  fileDependencies: [ './demo.txt' ],
  contextDependencies: [] }
```

输出告诉处理的“结果”，作为缓冲区处理的资源以及其他元信息。数据足以开发更复杂的加载器。
The output tells the `result` of the processing, the resource that was processed as a buffer, and other meta information. The data is enough to develop more complicated loaders.

T>如果要将输出捕获到文件，请使用`fs.writeFileSync（“./ output.txt”，result.result）`或其异步版本，如[节点文档]中所述（https：// nodejs .ORG / API / fs.html）。
T> If you want to capture the output to a file, use either `fs.writeFileSync("./output.txt", result.result)` or its asynchronous version as discussed in [Node documentation](https://nodejs.org/api/fs.html).

T>可以通过名称引用安装到本地项目的加载器，而不是解析它们的完整路径。示例：`loaders：[“raw-loader”]`。
T> It's possible to refer to loaders installed to the local project by name instead of resolving a full path to them. Example: `loaders: ["raw-loader"]`.

## 实现异步 Loader

即使你可以使用同步接口实现许多加载器，但有时需要进行异步计算。将第三方软件包包装为加载程序可以强制你执行此操作。
Even though you can implement a lot of loaders using the synchronous interface, there are times when an asynchronous calculation is required. Wrapping a third party package as a loader can force you to this.

上面的例子可以通过`this.async（）`使用webpack特定的API来适应异步形式。 Webpack设置了这个，并且该函数返回遵循Node约定的回调（错误优先，结果秒）。
The example above can be adapted to asynchronous form by using webpack specific API through `this.async()`. Webpack sets this, and the function returns a callback following Node conventions (error first, result second).

{pagebreak}

调整如下：
Tweak as follows:

**loaders/demo-loader.js**

```javascript
module.exports = function(input) {
  const callback = this.async();

  // No callback -> return synchronous results
  // if (callback) { ... }

  callback(null, input + input);
};
```

W>鉴于webpack通过`this`注入其API，这里不能使用较短的函数形式（`（）=> ...`）。
W> Given webpack injects its API through `this`, the shorter function form (`() => ...`) cannot be used here.

T>如果要将源映射传递给webpack，请将其作为回调的第三个参数。
T> If you want to pass a source map to webpack, give it as the third parameter of the callback.

再次运行演示脚本（`node run-loader.js`）应该得到与以前相同的结果。要在执行期间引发错误，请尝试以下操作：
Running the demo script (`node run-loader.js`) again should give the same result as before. To raise an error during execution, try the following:

**装载机/演示loader.js **
**loaders/demo-loader.js**

```javascript
module.exports = function(input) {
  const callback = this.async();

  callback(new Error("Demo error"));
};
```

结果应包含“错误：演示错误”，其中堆栈跟踪显示错误发生的位置。
The result should contain `Error: Demo error` with a stack trace showing where the error originates.

{pagebreak}

##仅返回输出
## Returning Only Output

装载程序可用于单独输出代码。你可以拥有如下实现：
Loaders can be used to output code alone. You could have an implementation as below:

**装载机/演示loader.js **
**loaders/demo-loader.js**

```javascript
module.exports = function() {
  return "foobar";
};
```

但重点是什么？你可以通过webpack条目传递给加载器。你可以给动态生成代码的加载器，而不是像在大多数情况下那样指向预先存在的文件。
But what's the point? You can pass to loaders through webpack entries. Instead of pointing to pre-existing files as you would in a majority of the cases, you could give to a loader that generates code dynamically.

T>如果要返回`Buffer`输出，请设置`module.exports.raw = true`。该标志将覆盖期望返回字符串的默认行为。
T> If you want to return `Buffer` output, set `module.exports.raw = true`. The flag overrides the default behavior which expects a string is returned.

{pagebreak}

##编写文件
## Writing Files

像* file-loader *这样的加载器会发出文件。为此，Webpack提供了一个单独的方法`this.emitFile`。鉴于* loader-runner *没有实现它，你必须嘲笑它​​：
Loaders, like *file-loader*, emit files. Webpack provides a single method, `this.emitFile`, for this. Given *loader-runner* does not implement it, you have to mock it:

**运行loader.js **
**run-loader.js**

```javascript
...

runLoaders(
  {
    resource: "./demo.txt",
    loaders: [path.resolve(__dirname, "./loaders/demo-loader")],
leanpub-start-insert
    context: {
      emitFile: () => {},
    },
leanpub-end-insert
    readResource: fs.readFile.bind(fs),
  },
  (err, result) => (err ? console.error(err) : console.log(result))
);
```

要实现* file-loader *的基本思想，你必须做两件事：发出文件并返回它的路径。你可以按如下方式申请：
To implement the essential idea of *file-loader*, you have to do two things: emit the file and return path to it. You could apply it as below:

**装载机/演示loader.js **
**loaders/demo-loader.js**

```javascript
const loaderUtils = require("loader-utils");

module.exports = function(content) {
  const url = loaderUtils.interpolateName(this, "[hash].[ext]", {
    content,
  });

  this.emitFile(url, content);

  const path = `__webpack_public_path__ + ${JSON.stringify(url)};`;

  return `export default ${path}`;
};
```

Webpack提供了两个额外的`emit`方法：
Webpack provides two additional `emit` methods:

*`this.emitWarning（<string>）`
* `this.emitWarning(<string>)`
*`this.emitError（<string>）`
* `this.emitError(<string>)`

这些调用应该用于基于“console”的替代方案。与`this.emitFile`一样，你必须为* loader-runner *模拟它们才能工作。
These calls should be used over `console` based alternatives. As with `this.emitFile`, you have to mock them for *loader-runner* to work.

接下来的问题是，如何将文件名传递给加载器。
The next question is, how to pass a file name to the loader.

##将选项传递给装载程序
## Passing Options to Loaders

为了演示传球选项，跑步者需要进行一些小调整：
To demonstrate passing options, the runner needs a small tweak:

**运行loader.js **
**run-loader.js**

```javascript
const fs = require("fs");
const path = require("path");
const { runLoaders } = require("loader-runner");

runLoaders(
  {
    resource: "./demo.txt",
leanpub-start-delete
    loaders: [path.resolve(__dirname, "./loaders/demo-loader")],
leanpub-end-delete
leanpub-start-insert
    loaders: [
      {
        loader: path.resolve(__dirname, "./loaders/demo-loader"),
        options: {
          name: "demo.[ext]",
        },
      },
    ],
leanpub-end-insert
    context: {
      emitFile: () => {},
    },
    readResource: fs.readFile.bind(fs),
  },
  (err, result) => (err ? console.error(err) : console.log(result))
);
```

要捕获该选项，你需要使用[loader-utils]（https://www.npmjs.com/package/loader-utils）。它旨在解析加载程序选项和查询。安装它：
To capture the option, you need to use [loader-utils](https://www.npmjs.com/package/loader-utils). It has been designed to parse loader options and queries. Install it:

```bash
npm install loader-utils --save-dev
```

要将它连接到加载程序，将其设置为捕获`name`并将其传递给webpack的插补器：
To connect it to the loader, set it to capture `name` and pass it through webpack's interpolator:

**装载机/演示loader.js **
**loaders/demo-loader.js**

```javascript
const loaderUtils = require("loader-utils");

module.exports = function(content) {
leanpub-start-insert
  const { name } = loaderUtils.getOptions(this);
leanpub-end-insert
leanpub-start-delete
  const url = loaderUtils.interpolateName(this, "[hash].[ext]", {
    content,
  });
leanpub-end-delete
leanpub-start-insert
  const url = loaderUtils.interpolateName(this, name, { content });
leanpub-end-insert
  );

  ...
};
```

{pagebreak}

运行后（`node。/ run-loader.js`），你应该看到一些东西：
After running (`node ./run-loader.js`), you should see something:

```javascript
{ result: [ 'export default __webpack_public_path__+"demo.txt";' ],
  resourceBuffer: <Buffer 66 6f 6f 62 61 72 0a>,
  cacheable: true,
  fileDependencies: [ './demo.txt' ],
  contextDependencies: [] }
```

你可以看到结果与加载程序应返回的内容相匹配。你可以尝试将更多选项传递给加载器或使用查询参数来查看不同组合会发生什么。
You can see that the result matches what the loader should have returned. You can try to pass more options to the loader or use query parameters to see what happens with different combinations.

T>如果选项不符合你的预期，那么验证选项是一个好主意，而不是默默地失败。 [schema-utils]（https://www.npmjs.com/package/schema-utils）专为此目的而设计。
T> It's a good idea to validate options and rather fail hard than silently if the options aren't what you expect. [schema-utils](https://www.npmjs.com/package/schema-utils) has been designed for this purpose.

##使用Webpack连接自定义加载器
## Connecting Custom Loaders with Webpack

要充分利用加载器，必须将它们与webpack连接起来。要实现这一目标，你可以通过导入：
To get most out of loaders, you have to connect them with webpack. To achieve this, you can go through imports:

** SRC / component.js **
**src/component.js**

```javascript
leanpub-start-insert
import "!../loaders/demo-loader?name=foo!./main.css";
leanpub-end-insert
```

{pagebreak}

鉴于定义很冗长，加载器可以如下别名：
Given the definition is verbose, the loader can be aliased as below:

** ** webpack.config.js
**webpack.config.js**

```javascript
const commonConfig = merge([
  {
  ...
leanpub-start-insert
    resolveLoader: {
      alias: {
        "demo-loader": path.resolve(
          __dirname,
          "loaders/demo-loader.js"
        ),
      },
    },
leanpub-end-insert
  },
  ...
]);
```

通过此更改，可以简化导入：
With this change the import can be simplified:

```javascript
leanpub-start-delete
import "!../loaders/demo-loader?name=foo!./main.css";
leanpub-end-delete
leanpub-start-insert
import "!demo-loader?name=foo!./main.css";
leanpub-end-insert
```

你还可以通过`rules`处理加载器定义。一旦加载器足够稳定，建立一个基于* webpack-defaults *的项目，在那里推送逻辑，并开始将加载器作为包使用。
You could also handle the loader definition through `rules`. Once the loader is stable enough, set up a project based on *webpack-defaults*, push the logic there, and begin to consume the loader as a package.

W>尽管使用* loader-runner *可以方便地开发和测试加载器，但实现针对webpack运行的集成测试。环境之间的微妙差异使这一点至关重要。
W> Although using *loader-runner* can be convenient for developing and testing loaders, implement integration tests that run against webpack. Subtle differences between environments make this essential.

## Pitch Loaders
## Pitch Loaders

![Webpack loader processing](images/loader-processing.png)

Webpack分两个阶段评估加载器：投球和评估。如果你习惯于Web事件语义，则这些映射到捕获和冒泡。我们的想法是webpack允许你在投球（捕捉）阶段拦截执行。它首先从左到右穿过装载机，然后从右到左执行。
Webpack evaluates loaders in two phases: pitching and evaluating. If you are used to web event semantics, these map to capturing and bubbling. The idea is that webpack allows you to intercept execution during the pitching (capturing) phase. It goes through the loaders left to right first and executes them from right to left after that.

{pagebreak}

音调加载器允许你调整请求甚至终止请求。设置它：
A pitch loader allows you shape the request and even terminate it. Set it up:

**装载机/俯仰loader.js **
**loaders/pitch-loader.js**

```javascript
const loaderUtils = require("loader-utils");

module.exports = function(input) {
  const { text } = loaderUtils.getOptions(this);

  return input + text;
};
module.exports.pitch = function(remainingReq, precedingReq, input) {
  console.log(`
Remaining request: ${remainingReq}
Preceding request: ${precedingReq}
Input: ${JSON.stringify(input, null, 2)}
  `);

  return "pitched";
};
```

要将其连接到运行器，请将其添加到加载程序定义：
To connect it to the runner, add it to the loader definition:

**运行loader.js **
**run-loader.js**

```javascript
runLoaders(
  {
    resource: "./demo.txt",
    loaders: [
      ...
leanpub-start-insert
      path.resolve(__dirname, "./loaders/pitch-loader"),
leanpub-end-insert
    ],
    ...
  },
  (err, result) => (err ? console.error(err) : console.log(result))
);
```

{pagebreak}

如果你现在运行（`node。/ run-loader.js`），音调加载器应该记录中间数据并拦截执行：
If you run (`node ./run-loader.js`) now, the pitch loader should log intermediate data and intercept the execution:

```javascript
Remaining request: ./demo.txt
Preceding request: .../webpack-demo/loaders/demo-loader?{"name":"demo.[ext]"}
Input: {}

{ result: [ 'export default __webpack_public_path__ + "demo.txt";' ],
  resourceBuffer: null,
  cacheable: true,
  fileDependencies: [],
  contextDependencies: [] }
```

##使用Loaders进行缓存
## Caching with Loaders

虽然webpack默认缓存加载器，除非它们设置了`this.cacheable（false）`，编写缓存加载器可能是一个很好的练习，因为它可以帮助你理解加载器阶段如何协同工作。以下示例显示了如何实现此目标（由Vladimir Grenaderov提供）：
Although webpack caches loaders by default unless they set `this.cacheable(false)`, writing a caching loader can be a good exercise as it helps you to understand how loader stages can work together. The example below shows how to achieve this (courtesy of Vladimir Grenaderov):

```javascript
const cache = new Map();

module.exports = function(content) {
  // Calls only once for given resourcePath
  const callbacks = cache.get(this.resourcePath);
  callbacks.forEach(callback => callback(null, content));

  cache.set(this.resourcePath, content);

  return content;
};
module.exports.pitch = function() {
  if (cache.has(this.resourcePath)) {
    const item = cache.get(this.resourcePath);

    if (item instanceof Array) {
      // Load to cache
      item.push(this.async());
    } else {
      // Hit cache
      return item;
    }
  } else {
    // Missed cache
    cache.set(this.resourcePath, []);
  }
};
```

可以使用音调加载器将元数据附加到输入以供稍后使用。在该示例中，在投球阶段期间构建了高速缓存，并且在正常执行期间访问了高速缓存。
A pitch loader can be used to attach metadata to the input to use later. In this example, a cache was constructed during the pitching stage, and it was accessed during normal execution.

T> [官方文档]（https://webpack.js.org/api/loaders/）详细介绍了加载器API。你可以通过`this`看到所有可用的字段。
T> The [official documentation](https://webpack.js.org/api/loaders/) covers the loader API in detail. You can see all fields available through `this` there.

{pagebreak}

##结论
## Conclusion

编写加载器很有趣，因为它们描述了从格式到另一种格式的转换。通常，你可以通过研究API文档或现有的加载器来弄清楚如何实现某些特定的东西。
Writing loaders is fun in the sense that they describe transformations from a format to another. Often you can figure out how to achieve something specific by either studying either the API documentation or the existing loaders.

回顾一下：
To recap:

* * loader-runner *是了解加载器如何工作的宝贵工具。用它来调试加载器的工作方式。
* *loader-runner* is a valuable tool for understanding how loaders work. Use it for debugging how loaders work.
* Webpack **加载器**接受输入并基于它生成输出。
* Webpack **loaders** accept input and produce output based on it.
*加载器可以是同步的也可以是异步的。在后一种情况下，你应该使用`this.async（）`webpack API来捕获webpack公开的回调。
* Loaders can be either synchronous or asynchronous. In the latter case, you should use `this.async()` webpack API to capture the callback exposed by webpack.
*如果你想为webpack条目动态生成代码，那么加载器可以派上用场。加载器不必接受输入。在这种情况下，它只返回输出是可以接受的。
* If you want to generate code dynamically for webpack entries, that's where loaders can come in handy. A loader does not have to accept input. It's acceptable that it returns only output in this case.
*使用** loader-utils **来解析传递给加载器的可能选项，并考虑使用** schema-utils **验证它们。
* Use **loader-utils** to parse possible options passed to a loader and consider validating them using **schema-utils**.
*在本地开发加载器时，请考虑设置`resolveLoader.alias`来清理引用。
* When developing loaders locally, consider setting up a `resolveLoader.alias` to clean up references.
*投球阶段补充了默认行为，允许你拦截和附加元数据。
* Pitching stage complements the default behavior allowing you to intercept and to attach metadata.

你将学习在下一章编写插件。插件允许你拦截webpack的执行过程，并且它们可以与加载器组合以开发更高级的功能。
You'll learn to write plugins in the next chapter. Plugins allow you to intercept webpack's execution process and they can be combined with loaders to develop more advanced functionality.

