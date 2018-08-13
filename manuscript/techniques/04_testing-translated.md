＃测试
# Testing

测试是开发的重要部分。尽管linting等技术可以帮助发现和解决问题，但它们还有其局限性。测试可以应用于许多不同级别的代码和应用程​​序。
Testing is a vital part of development. Even though techniques, such as linting, can help to spot and solve issues, they have their limitations. Testing can be applied to the code and an application on many different levels.

你可以**单元测试**一段特定的代码，或者你可以从用户的角度通过**验收测试来查看应用程序**。 **集成测试**适用于频谱的这两端，并关注单独的代码单元如何一起工作。
You can **unit test** a specific piece of code, or you can look at the application from the user's point of view through **acceptance testing**. **Integration testing** fits between these ends of the spectrum and is concerned about how separate units of code operate together.

你可以找到很多JavaScript的测试工具。配置正确后，最流行的选项适用于webpack。尽管测试运行器在没有webpack的情况下工作，但通过它运行它们可以让你处理测试运行者不了解的代码，同时控制模块的解析方式。你也可以使用webpack的监视模式，而不是依赖于测试运行者提供的模式。
You can find a lot of testing tools for JavaScript. The most popular options work with webpack after you configure it right. Even though test runners work without webpack, running them through it allows you to process code the test runners do not understand while having control over the way modules are resolved. You can also use webpack's watch mode instead of relying on one provided by a test runner.

## mocha
## Mocha

![Mocha](images/mocha.png)

[Mocha]（https://mochajs.org/）是Node的流行测试框架。虽然Mocha提供测试基础架构，但你必须将其断言。即使[Node`assert`]（https://nodejs.org/api/assert.html）足够，它也可以与其他断言库一起使用。
[Mocha](https://mochajs.org/) is a popular test framework for Node. While Mocha provides test infrastructure, you have to bring your asserts to it. Even though [Node `assert`](https://nodejs.org/api/assert.html) can be enough, it works with other assertion libraries as well.

[mocha-loader]（https://www.npmjs.com/package/mocha-loader）允许通过webpack运行Mocha测试。 [mocha-webpack]（https://www.npmjs.com/package/mocha-webpack）是另一个旨在提供更多功能的选项。
[mocha-loader](https://www.npmjs.com/package/mocha-loader) allows running Mocha tests through webpack. [mocha-webpack](https://www.npmjs.com/package/mocha-webpack) is another option that aims to provide more functionality.

###使用Webpack配置* mocha-loader *
### Configuring *mocha-loader* with Webpack

首先，将Mocha和* mocha-loader *添加到你的项目中：
To get started, include Mocha and *mocha-loader* to your project:

```bash
npm install mocha mocha-loader --save-dev
```

{pagebreak}

###设置要测试的代码
### Setting Up Code to Test

要测试一下，设置一个函数：
To have something to test, set up a function:

**测试/ add.js **
**tests/add.js**

```javascript
module.exports = (a, b) => a + b;
```

然后，为了测试它，设置一个小测试套件：
Then, to test that, set up a small test suite:

**测试/ add.test.js **
**tests/add.test.js**

```javascript
const assert = require("assert");
const add = require("./add");

describe("Demo", () => {
  it("should add correctly", () => {
    assert.equal(add(1, 1), 2);
  });
});
```

###配置Mocha
### Configuring Mocha

要针对测试运行Mocha，请添加脚本：
To run Mocha against the test, add a script:

** **的package.json
**package.json**

```json
"scripts": {
  "test:mocha": "mocha tests",
  ...
},
```

如果你现在执行`npm run test：mocha`，你应该看到以下输出：
If you execute `npm run test:mocha` now, you should see the following output:

```
Demo
  should add correctly


1 passing (5ms)
```

摩卡还提供了一种手表模式，你可以通过`npm run test：mocha  -  --watch`激活。它在你修改代码时运行测试套件。
Mocha also provides a watch mode which you can activate through `npm run test:mocha -- --watch`. It runs the test suite as you modify the code.

如果你只想关注一组特定的测试，可以使用T>`--grep <pattern>`来约束行为。
T> `--grep <pattern>` can be used for constraining the behavior if you want to focus only on a particular set of tests.

###配置Webpack
### Configuring Webpack

Webpack可以通过Web界面提供类似的功能。本书前面已经解决了问题的难点，剩下的就是通过配置将这些解决方案结合起来。
Webpack can provide similar functionality through a web interface. The hard parts of the problem have been solved earlier in this book, what remains is combining those solutions through configuration.

要告诉webpack要运行哪些测试，需要以某种方式导入它们。 *动态加载*章节讨论了`require.context`，它允许根据规则聚合文件。这里很理想。设置入口点如下：
To tell webpack which tests to run, they need to be imported somehow. The *Dynamic Loading* chapter discussed `require.context` that allows to aggregate files based on a rule. It's ideal here. Set up an entry point as follows:

**测试/ index.js **
**tests/index.js**

```javascript
// Skip execution in Node
if (module.hot) {
  const context = require.context(
    "mocha-loader!./", // Process through mocha-loader
    false, // Skip recursive processing
    /\.test.js$/ // Pick only files ending with .test.js
  );

  // Execute each test suite
  context.keys().forEach(context);
}
```

webpack方面需要进行一些小改动：
A small change is required on webpack side:

** ** webpack.mocha.js
**webpack.mocha.js**

```javascript
const path = require("path");
const merge = require("webpack-merge");

const parts = require("./webpack.parts");

module.exports = merge([
  parts.devServer(),
  parts.page({
    title: "Mocha demo",
    entry: {
      tests: path.join(__dirname, "tests"),
    },
  }),
]);
```

T>有关完整的`devServer`设置，请参阅* Composing Configuration *章节。页面设置在* Multiple Pages *章节中进行了说明。
T> See the *Composing Configuration* chapter for the full `devServer` setup. The page setup is explained in the *Multiple Pages* chapter.

添加帮助程序脚本以便于运行：
Add a helper script to make it convenient to run:

** **的package.json
**package.json**

```json
"scripts": {
  "test:mocha:watch":
    "webpack-dev-server --hot --config webpack.mocha.js",
  ...
},
```

T>如果你想了解`--hot`做得更好，请参阅*热模块更换*附录。
T> If you want to understand what `--hot` does better, see the *Hot Module Replacement* appendix.

如果你现在执行服务器并导航到`http：// localhost：8080 /`，你应该看到测试：
If you execute the server now and navigate to `http://localhost:8080/`, you should see the test:

![Mocha in browser](images/mocha-browser.png)

调整测试或代码应导致浏览器发生变化。你可以在查看测试状态的同时扩展规范或重构代码。
Adjusting either the test or the code should lead to a change in the browser. You can grow your specification or refactor the code while seeing the status of the tests.

与vanilla Mocha设置相比，通过webpack配置Mocha具有以下几个优点：
Compared to the vanilla Mocha setup, configuring Mocha through webpack comes with a couple of advantages:

*可以调整模块分辨率。 Webpack别名和其他技术现在可以工作，但这也会将代码绑定到webpack。
* It's possible to adjust module resolution. Webpack aliasing and other techniques work now, but this would also tie the code to webpack.
*你可以使用webpack的处理来编译你想要的代码。使用香草摩卡，意味着更多的设置。
* You can use webpack's processing to compile your code as you wish. With vanilla Mocha that would imply more setup outside of it.

在缺点方面，现在需要一个浏览器来检查测试。 * mocha-loader *作为开发助手处于最佳状态。通过无头浏览器运行测试可以解决这个问题。
On the downside, now you need a browser to examine the tests. *mocha-loader* is at its best as a development helper. The problem can be solved by running the tests through a headless browser.

## Karma和Mocha
## Karma and Mocha

![Karma](images/karma.png)

[Karma]（https://karma-runner.github.io/）是一个测试运行器，允许你在真实设备上运行测试和[PhantomJS]（http://phantomjs.org/），一个无头浏览器。 [karma-webpack]（https://www.npmjs.com/package/karma-webpack）是一个Karma预处理器，允许你将Karma与webpack连接。与之前相同的好处仍然适用。但是，这一次，对测试环境有了更多的控制。
[Karma](https://karma-runner.github.io/) is a test runner that allows you to run tests on real devices and [PhantomJS](http://phantomjs.org/), a headless browser. [karma-webpack](https://www.npmjs.com/package/karma-webpack) is a Karma preprocessor that allows you to connect Karma with webpack. The same benefits as before apply still. This time around, however, there is more control over the test environment.

要开始，请安装Karma，Mocha，* karma-mocha * reporter，以及* karma-webpack *：
To get started, install Karma, Mocha, *karma-mocha* reporter, and *karma-webpack*:

```bash
npm install karma mocha karma-mocha karma-webpack --save-dev
```

{pagebreak}

与webpack一样，Karma也依赖于配置约定。按如下方式设置文件以使其获取测试：
Like webpack, Karma relies on a configuration convention as well. Set up a file as follows to make it pick up the tests:

** ** karma.conf.js
**karma.conf.js**

```javascript
const parts = require("./webpack.parts");

module.exports = config => {
  const tests = "tests/*.test.js";

  config.set({
    frameworks: ["mocha"],
    files: [
      {
        pattern: tests,
      },
    ],
    preprocessors: {
      [tests]: ["webpack"],
    },
    webpack: parts.loadJavaScript(),
    singleRun: true,
  });
};
```

W>设置为每个测试生成一个包。如果你有大量的测试并希望提高性能，那么就像上面的Mocha一样设置`require.context`。有关详细信息，请参阅[karma-webpack issue 23]（https://github.com/webpack-contrib/karma-webpack/issues/23）。
W> The setup generates a bundle per each test. If you have a large number of tests and want to improve performance, set up `require.context` as for Mocha above. See [karma-webpack issue 23](https://github.com/webpack-contrib/karma-webpack/issues/23) for more details.

{pagebreak}

添加npm快捷方式：
Add an npm shortcut:

```json
...
"scripts": {
  "test:karma": "karma start",
  ...
},
...
```

如果你现在执行`npm run test：karma`，你应该看到终端输出：
If you execute `npm run test:karma` now, you should see the terminal output:

```
...
webpack: Compiled successfully.
...:INFO [karma]: Karma v1.7.1 server started at http://0.0.0.0:9876/
```

以上意味着Karma正在等待，你必须访问该URL才能运行测试。根据配置（`singleRun：true`），Karma在此之后终止执行：
The above means Karma is waiting and you have to visit that url to run the tests. As per configuration (`singleRun: true`), Karma terminates execution after that:

```
...
...:INFO [karma]: Karma v1.7.1 server started at http://0.0.0.0:9876/
...:INFO [Chrome 61...]: Connected on socket D...A with id manual-73
Chrome 61...): Executed 1 of 1 SUCCESS (0.003 secs / 0 secs)
```

鉴于运行测试这种方式会变得很烦人，配置替代方法是个好主意。使用PhantomJS是一种选择。
Given running tests this way can become annoying, it's a good idea to configure alternative ways. Using PhantomJS is one option.

T>你可以通过`browsers`字段将Karma指向特定的浏览器。示例：`browsers：['Chrome']`。
T> You can point Karma to specific browsers through the `browsers` field. Example: `browsers: ['Chrome']`.

{pagebreak}

###通过PhantomJS运行测试
### Running Tests Through PhantomJS

通过PhantomJS运行测试需要几个依赖项：
Running tests through PhantomJS requires a couple of dependencies:

```bash
npm install karma-phantomjs-launcher phantomjs-prebuilt --save-dev
```

要通过Phantom进行Karma运行测试，请按如下方式调整其配置：
To make Karma run tests through Phantom, adjust its configuration as follows:

** ** karma.conf.js
**karma.conf.js**

```javascript
module.exports = config => {
  ...

  config.set({
    ...
leanpub-start-insert
    browsers: ["PhantomJS"],
leanpub-end-insert
  });
};
```

如果你再次执行测试（`npm run test：karma`），你应该得到输出而不必访问url：
If you execute the tests again (`npm run test:karma`), you should get output without having to visit an url:

```
...
webpack: Compiled successfully.
...:INFO [karma]: Karma v1.7.1 server started at http://0.0.0.0:9876/
...:INFO [launcher]: Launching browser PhantomJS with unlimited concurrency
...:INFO [launcher]: Starting browser PhantomJS
...:INFO [PhantomJS ...]: Connected on socket 7...A with id 123
PhantomJS ...: Executed 1 of 1 SUCCESS (0.005 secs / 0.001 secs)
```

经过一段时间后变化可能会变得无聊的运行测试，Karma提供了一种观察模式。
Given running tests after the change can get boring after a while, Karma provides a watch mode.

W> PhantomJS还不支持ES2015功能，因此你必须使用它们预处理测试代码。计划为PhantomJS 2.5提供ES2015支持。
W> PhantomJS does not support ES2015 features yet, so you have to preprocess the code for tests using them. ES2015 support is planned for PhantomJS 2.5.

###使用Karma观看模式
### Watch Mode with Karma

可以按如下方式访问Karma的监视模式：
Accessing Karma's watch mode is possible as follows:

** **的package.json
**package.json**

```json
"scripts": {
leanpub-start-insert
  "test:karma:watch": "karma start --auto-watch --no-single-run",
leanpub-end-insert
  ...
},
```

如果你现在执行`npm run test：karma：watch`，你应该看到监视行为。
If you execute `npm run test:karma:watch` now, you should see watch behavior.

###生成覆盖率报告
### Generating Coverage Reports

要了解测试涵盖的代码量，最好生成覆盖率报告。这样做需要代码级检测。此外，必须报告添加的信息，并且可以通过HTML和LCOV报告完成。
To know how much of the code the tests cover, it can be a good idea to generate coverage reports. Doing this requires code-level instrumentation. Also, the added information has to be reported and can be done through HTML and LCOV reports.

T> LCOV与可视化服务完美集成。你可以通过持续集成环境将覆盖信息发送到外部服务，并在一个位置跟踪状态。
T> LCOV integrates well with visualization services. You can send coverage information to an external service through a continuous integration environment and track the status in one place.

[isparta]（https://www.npmjs.com/package/isparta）是一款流行的ES2015兼容代码覆盖工具。将其与Karma连接需要配置。最重要的是，代码必须通过[babel-plugin-istanbul]（https://www.npmjs.com/package/babel-plugin-istanbul）进行检测。这样做需要少量的webpack配置，因为设置。 [karma-coverage]（https://www.npmjs.com/package/karma-coverage）是问题报告部分所必需的。
[isparta](https://www.npmjs.com/package/isparta) is a popular, ES2015 compatible code coverage tool. Connecting it with Karma requires configuration. Most importantly the code has to be instrumented through [babel-plugin-istanbul](https://www.npmjs.com/package/babel-plugin-istanbul). Doing this requires a small amount of webpack configuration as well due to the setup. [karma-coverage](https://www.npmjs.com/package/karma-coverage) is required for the reporting portion of the problem.

{pagebreak}

首先安装依赖项：
Install the dependencies first:

```
npm install babel-plugin-istanbul karma-coverage --save-dev
```

连接Babel插件，以便在运行Karma时发生检测：
Connect the Babel plugin so that the instrumentation happens when Karma is run:

**。** babelrc
**.babelrc**

```json
...
leanpub-start-insert
"env": {
  "karma": {
    "plugins": [
      [
        "istanbul",
        { "exclude": ["tests/*.test.js"] }
      ]
    ]
  }
}
leanpub-end-insert
```

确保设置Babel环境，以便它获取插件：
Make sure to set Babel environment, so it picks up the plugin:

** ** karma.conf.js
**karma.conf.js**

```javascript
module.exports = config => {
  ...

leanpub-start-insert
  process.env.BABEL_ENV = "karma";
leanpub-end-insert

  config.set({
    ...
  });
};
```

T>如果你想了解`env`的想法，请参阅* Loading JavaScript *章节。
T> If you want to understand the `env` idea, see the *Loading JavaScript* chapter.

在Karma方面，必须设置报告，并且必须将Karma配置与webpack连接。 * karma-webpack *为此提供了两个字段：`webpack`和`webpackMiddleware`。在这种情况下，你应该使用前者来确保通过Babel处理代码。
On Karma side, reporting has to be set up, and Karma configuration has to be connected with webpack. *karma-webpack* provides two fields for this purpose: `webpack` and `webpackMiddleware`. You should use the former in this case to make sure the code gets processed through Babel.

** ** karma.conf.js
**karma.conf.js**

```javascript
leanpub-start-insert
const path = require("path");
leanpub-end-insert

...

module.exports = config => {
  ...

  config.set({
    ...
leanpub-start-insert
    reporters: ["coverage"],
    coverageReporter: {
      dir: "build",
      reporters: [{ type: "html" }, { type: "lcov" }],
    },
leanpub-end-insert
  });
};
```

T>如果要将报告发送到`dir`下面的特定目录，请为每个报告设置`subdir`。
T> If you want to emit the reports to specific directories below `dir`, set `subdir` per each report.

如果你现在执行karma（`npm run test：karma`），你应该在* build *下面看到一个包含覆盖率报告的新目录。可以通过浏览器检查HTML报告。
If you execute karma now (`npm run test:karma`), you should see a new directory below *build* containing coverage reports. The HTML report can be examined through the browser.

![Coverage in browser](images/coverage.png)

LCOV需要特定的工具才能工作。你可以找到Atom的[lcov-info]（https://atom.io/packages/lcov-info）等编辑器插件。在使用监视模式进行开发时，正确配置的插件可以为你提供覆盖信息。
LCOV requires specific tooling to work. You can find editor plugins such as [lcov-info](https://atom.io/packages/lcov-info) for Atom. A correctly configured plugin can give you coverage information while you are developing using the watch mode.

##是
## Jest

![Jest](images/jest.png)

Facebook的[Jest]（https://facebook.github.io/jest/）是一种自以为是的选择，它以最小的设置封装了功能，包括覆盖和模拟。它可以捕获数据的快照，使其对你有记录和保留行为的项目很有价值。
Facebook's [Jest](https://facebook.github.io/jest/) is an opinionated alternative that encapsulates functionality, including coverage and mocking, with minimal setup. It can capture snapshots of data making it valuable for projects where you have the behavior you would like to record and retain.

Jest测试遵循[Jasmine]（https://www.npmjs.com/package/jasmine）测试框架语义，它支持开箱即用的Jasmine风格的断言。特别是套件定义与Mocha足够接近，因此当前测试应该可以在不对测试代码本身进行任何调整的情况下工作。 Jest提供了[jest-codemods]（https://www.npmjs.com/package/jest-codemods），用于将更复杂的项目迁移到Jest语义。
Jest tests follow [Jasmine](https://www.npmjs.com/package/jasmine) test framework semantics, and it supports Jasmine-style assertions out of the box. Especially the suite definition is close enough to Mocha so that the current test should work without any adjustments to the test code itself. Jest provides [jest-codemods](https://www.npmjs.com/package/jest-codemods) for migrating more complicated projects to Jest semantics.

安装是第一个：
Install Jest first:

```
npm install jest --save-dev
```

Jest通过* package.json * [configuration]（https://facebook.github.io/jest/docs/en/configuration.html）捕获测试。它检测* __ tests __ *目录中的测试，它也恰好捕获项目默认使用的命名模式：
Jest captures tests through *package.json* [configuration](https://facebook.github.io/jest/docs/en/configuration.html). It detects tests within a *__tests__* directory it also happens to capture the naming pattern the project is using by default:

** **的package.json
**package.json**

```json
"scripts": {
leanpub-start-insert
  "test:jest:watch": "jest --watch",
  "test:jest": "jest",
leanpub-end-insert
  ...
},
```

现在你有两个新命令：一个用于运行测试一次，另一个用于在监视模式下运行它们。要捕获覆盖率信息，你必须在* package.json *中的``jest'`设置中设置`“collectCoverage”：true`或将`--coverage`标志传递给Jest。默认情况下，它会在* coverage *目录下发布覆盖率报告。
Now you have two new commands: one to run tests once and other to run them in a watch mode. To capture coverage information, you have to set `"collectCoverage": true` at `"jest"` settings in *package.json* or pass `--coverage` flag to Jest. It emits the coverage reports below *coverage* directory by default.

鉴于生成覆盖率报告带来性能开销，通过标志启用行为可能是一个好主意。这样你就可以准确控制何时捕获信息。
Given generating coverage reports comes with a performance overhead, enabling the behavior through the flag can be a good idea. This way you can control exactly when to capture the information.

将Webpack设置移植到Jest需要更多的努力，特别是如果你依赖webpack特定功能。 [官方指南]（https://facebook.github.io/jest/docs/en/webpack.html）涵盖了一些常见问题。你也可以配置Jest通过[babel-jest]（https://www.npmjs.com/package/babel-jest）使用Babel，因为它允许你使用像[babel-plugin-module-resolver]这样的Babel插件（ https://www.npmjs.com/package/babel-plugin-module-resolver）以匹配webpack的功能。
Porting a webpack setup to Jest requires more effort especially if you rely on webpack specific features. [The official guide](https://facebook.github.io/jest/docs/en/webpack.html) covers quite a few of the common problems. You can also configure Jest to use Babel through [babel-jest](https://www.npmjs.com/package/babel-jest) as it allows you to use Babel plugins like [babel-plugin-module-resolver](https://www.npmjs.com/package/babel-plugin-module-resolver) to match webpack's functionality.

T> [jest-webpack]（https://www.npmjs.com/package/jest-webpack）提供了webpack和Jest之间的集成。
T> [jest-webpack](https://www.npmjs.com/package/jest-webpack) provides an integration between webpack and Jest.

## AVA
## AVA

![AVA](images/ava.png)

[AVA]（https://www.npmjs.com/package/ava）是一个旨在利用并行执行的测试运行器。它附带了自己的测试套件定义。 [webpack-ava-recipe]（https://github.com/greyepoxy/webpack-ava-recipe）介绍了如何将其与webpack连接。
[AVA](https://www.npmjs.com/package/ava) is a test runner that has been designed to take advantage of parallel execution. It comes with a test suite definition of its own. [webpack-ava-recipe](https://github.com/greyepoxy/webpack-ava-recipe) covers how to connect it with webpack.

主要思想是在监视模式下运行webpack和AVA，以将处理代码的问题推送到webpack，同时允许AVA使用已处理的代码。与Mocha讨论的`require.context`想法在这里派上用场，因为你必须捕获webpack的测试以便以某种方式处理。
The main idea is to run both webpack and AVA in watch mode to push the problem of processing code to webpack while allowing AVA to consume the processed code. The `require.context` idea discussed with Mocha comes in handy here as you have to capture tests for webpack to handle somehow.

##嘲笑
## Mocking

模拟是一种允许你替换测试对象的技术。考虑以下解决方案：
Mocking is a technique that allows you to replace test objects. Consider the solutions below:

* [Sinon]（https://www.npmjs.com/package/sinon）提供模拟，存根和间谍。从2.0版开始，它适用于webpack。
* [Sinon](https://www.npmjs.com/package/sinon) provides mocks, stubs, and spies. It works well with webpack since version 2.0.
* [inject-loader]（https://www.npmjs.com/package/inject-loader）允许你通过其依赖项将代码注入模块，使其对模拟有价值。
* [inject-loader](https://www.npmjs.com/package/inject-loader) allows you to inject code into modules through their dependencies making it valuable for mocking.
* [rewire-webpack]（https://www.npmjs.com/package/rewire-webpack）允许模拟和覆盖模块全局变量。 [babel-plugin-rewire]（https://www.npmjs.com/package/babel-plugin-rewire）为Babel实现[rewire]（https://www.npmjs.com/package/rewire）。
* [rewire-webpack](https://www.npmjs.com/package/rewire-webpack) allows mocking and overriding module globals. [babel-plugin-rewire](https://www.npmjs.com/package/babel-plugin-rewire) implements [rewire](https://www.npmjs.com/package/rewire) for Babel.

##从测试中删除文件
## Removing Files From Tests

如果你通过webpack执行测试，你可能希望改变它处理像图像这样的资产的方式。你可以匹配它们，然后使用`noop`函数替换模块，如下所示：
If you execute tests through webpack, you may want to alter the way it treats assets like images. You can match them and then use a `noop` function to replace the modules as follows:

```javascript
plugins: [
  new webpack.NormalModuleReplacementPlugin(
    /\.(gif|png|scss|css)$/, "lodash/noop"
  ),
]
```

{pagebreak}

##总结
## Conclusion

Webpack可以配置为使用各种各样的测试工具。每个工具都有它的甜点，但它们也有相当多的共同点。
Webpack can be configured to work with a large variety of testing tools. Each tool has its sweet spots, but they also have quite a bit of common ground.

回顾一下：
To recap:

*运行测试工具可以让你从webpack的模块解析机制中受益。
* Running testing tools allows you to benefit from webpack's module resolution mechanism.
*有时候测试设置可能非常复杂。像Jest这样的工具可以删除大部分样板，并允许你以最少的配置开发测试。
* Sometimes the test setup can be quite involved. Tools like Jest remove most of the boilerplate and allow you to develop tests with minimal configuration.
*你可以找到多个webpack的模拟工具。它们允许你塑造测试环境。但有时你可以避免通过设计进行嘲弄。
* You can find multiple mocking tools for webpack. They allow you to shape test environment. Sometimes you can avoid mocking through design, though.

你将学习在下一章中使用webpack部署应用程序。
You'll learn to deploy applications using webpack in the next chapter.

