＃ 环境变量
# Environment Variables

有时，您的代码的一部分应该仅在开发期间执行。或者您可以在构建中具有尚未准备好进行生产的实验性功能。控制**环境变量**变得有价值，因为您可以使用它们切换功能。
Sometimes a part of your code should execute only during development. Or you could have experimental features in your build that are not ready for production yet. Controlling **environment variables** becomes valuable as you can toggle functionality using them.

由于JavaScript minifiers可以删除死代码（`if（false）`），你可以构建这个想法并编写转换成这个形式的代码。 Webpack的`DefinePlugin`允许替换**自由变量**，这样你就可以将`if（process.env.NODE_ENV ===“development”）`代码转换为`if（true）`或`if（false）`取决于环境。
Since JavaScript minifiers can remove dead code (`if (false)`), you can build on top of this idea and write code that gets transformed into this form. Webpack's `DefinePlugin` enables replacing **free variables** so that you can convert `if (process.env.NODE_ENV === "development")` kind of code to `if (true)` or `if (false)` depending on the environment.

您可以找到依赖此行为的包。 React可能是早期采用该技术的最着名的例子。使用`DefinePlugin`可以在某种程度上降低React生产构建的大小，并且您也可以看到与其他包类似的效果。
You can find packages that rely on this behavior. React is perhaps the most known example of an early adopter of the technique. Using `DefinePlugin` can bring down the size of your React production build somewhat as a result, and you can see a similar effect with other packages as well.

Webpack 4根据给定的模式设置`process.env.NODE_ENV`。不过，了解技术及其工作原理是件好事。
Webpack 4 sets `process.env.NODE_ENV` based on the given mode. It's good to know the technique and how it works, though.

{pagebreak}

## DefinePlugin`的基本思想
## The Basic Idea of `DefinePlugin`

要更好地理解“DefinePlugin”的概念，请考虑以下示例：
To understand the idea of `DefinePlugin` better, consider the example below:

```javascript
var foo;

// Not free due to "foo" above, not ok to replace
if (foo === "bar") {
  console.log("bar");
}

// Free since you don't refer to "bar", ok to replace
if (bar === "bar") {
  console.log("bar");
}
```

如果用“foobar”这样的字符串替换`bar`，那么你最终会得到如下代码：
If you replaced `bar` with a string like `"foobar"`, then you would end up with the code as below:

```javascript
var foo;

// Not free due to "foo" above, not ok to replace
if (foo === "bar") {
  console.log("bar");
}

// Free since you don't refer to "bar", ok to replace
if ("foobar" === "bar") {
  console.log("bar");
}
```

{pagebreak}

进一步的分析表明，“foobar”===“bar”`等于`false`所以minifier给出以下内容：
Further analysis shows that `"foobar" === "bar"` equals `false` so a minifier gives the following:

```javascript
var foo;

// Not free due to "foo" above, not ok to replace
if (foo === "bar") {
  console.log("bar");
}

// Free since you don't refer to "bar", ok to replace
if (false) {
  console.log("bar");
}
```

minifier消除了`if`语句，因为它已成为死代码：
A minifier eliminates the `if` statement as it has become dead code:

```javascript
var foo;

// Not free, not ok to replace
if (foo === "bar") {
  console.log("bar");
}

// if (false) means the block can be dropped entirely
```

消除是“DefinePlugin”的核心思想，它允许切换。缩小器执行分析并切换代码的​​整个部分。
Elimination is the core idea of `DefinePlugin` and it allows toggling. A minifier performs analysis and toggles entire portions of the code.

{pagebreak}

##设置`process.env.NODE_ENV`
## Setting `process.env.NODE_ENV`

和以前一样，将这个想法封装到一个函数中。由于webpack替换free变量的方式，你应该通过`JSON.stringify`推送它。你最终会得到一个类似`'“demo”'的字符串，然后webpack将它插入它找到的插槽中：
As before, encapsulate this idea to a function. Due to the way webpack replaces the free variable, you should push it through `JSON.stringify`. You end up with a string like `'"demo"'` and then webpack inserts that into the slots it finds:

** ** webpack.parts.js
**webpack.parts.js**

```javascript
const webpack = require("webpack");

exports.setFreeVariable = (key, value) => {
  const env = {};
  env[key] = JSON.stringify(value);

  return {
    plugins: [new webpack.DefinePlugin(env)],
  };
};
```

将其与配置连接：
Connect this with the configuration:

** ** webpack.config.js
**webpack.config.js**

```javascript
const commonConfig = merge([
  ...
leanpub-start-insert
  parts.setFreeVariable("HELLO", "hello from config"),
leanpub-end-insert
]);
```

{pagebreak}

最后，添加要替换的内容：
Finally, add something to replace:

** SRC / component.js **
**src/component.js**

```javascript
leanpub-start-delete
export default (text = "Hello world") => {
leanpub-end-delete
leanpub-start-insert
export default (text = HELLO) => {
leanpub-end-insert
  const element = document.createElement("div");

  ...
};
```

如果您运行该应用程序，您应该会在该按钮上看到一条新消息。
If you run the application, you should see a new message on the button.

T> [webpack-conditional-loader]（https://www.npmjs.com/package/webpack-conditional-loader）根据代码注释执行类似的操作。它可以用来消除整个代码块。
T> [webpack-conditional-loader](https://www.npmjs.com/package/webpack-conditional-loader) performs something similar based on code comments. It can be used to eliminate entire blocks of code.

T>`webpack.EnvironmentPlugin（[“NODE_ENV”]）`是一个允许您引用环境变量的快捷方式。它使用下面的`DefinePlugin`，你可以通过传递`process.env.NODE_ENV`来达到同样的效果。
T> `webpack.EnvironmentPlugin(["NODE_ENV"])` is a shortcut that allows you to refer to environment variables. It uses `DefinePlugin` underneath, and you can achieve the same effect by passing `process.env.NODE_ENV`.

##通过Babel替换自由变量
## Replacing Free Variables Through Babel

[babel-plugin-transform-inline-environment-variables]（https://www.npmjs.com/package/babel-plugin-transform-inline-environment-variables）可用于实现相同的效果。 [babel-plugin-transform-define]（https://www.npmjs.com/package/babel-plugin-transform-define）和[babel-plugin-minify-replace]（https://www.npmjs.com / package / babel-plugin-minify-replace）是Babel的其他替代品。
[babel-plugin-transform-inline-environment-variables](https://www.npmjs.com/package/babel-plugin-transform-inline-environment-variables) can be used to achieve the same effect. [babel-plugin-transform-define](https://www.npmjs.com/package/babel-plugin-transform-define) and [babel-plugin-minify-replace](https://www.npmjs.com/package/babel-plugin-minify-replace) are other alternatives for Babel.

{pagebreak}

##选择要使用的模块
## Choosing Which Module to Use

本章中讨论的技术可用于根据环境选择整个模块。如上所示，基于“DefinePlugin”的拆分允许您选择要使用的代码分支以及要丢弃的代码。这个想法可用于在模块级别实现分支。考虑下面的文件结构：
The techniques discussed in this chapter can be used to choose entire modules depending on the environment. As seen above, `DefinePlugin` based splitting allows you to choose which branch of code to use and which to discard. This idea can be used to implement branching on module level. Consider the file structure below:

```bash
.
└── store
    ├── index.js
    ├── store.dev.js
    └── store.prod.js
```

这个想法是你根据环境选择商店的'dev`或`prod`版本。这是* index.js *，它做了很多努力：
The idea is that you choose either `dev` or `prod` version of the store depending on the environment. It's that *index.js* which does the hard work:

```javascript
if (process.env.NODE_ENV === "production") {
  module.exports = require("./store.prod");
} else {
  module.exports = require("./store.dev");
}
```

Webpack可以根据`DefinePlugin`声明和这段代码选择正确的代码。你必须在这里使用CommonJS模块定义样式，因为ES2015`import`s不允许按设计进行动态行为。
Webpack can pick the right code based on the `DefinePlugin` declaration and this code. You have to use CommonJS module definition style here as ES2015 `import`s don't allow dynamic behavior by design.

T>相关技术，**别名**，在*消费包*章节中讨论。
T> A related technique, **aliasing**, is discussed in the *Consuming Packages* chapter.

{pagebreak}

##结论
## Conclusion

设置环境变量是一种允许您控制构建中包含源的哪些路径的技术。
Setting environment variables is a technique that allows you to control which paths of the source are included in the build.

回顾一下：
To recap:

* Webpack允许您通过`DefinePlugin`和`EnvironmentPlugin`设置**环境变量**。后者将系统级环境变量映射到源。
* Webpack allows you to set **environment variables** through `DefinePlugin` and `EnvironmentPlugin`. Latter maps the system level environment variables to the source.
*`DefinePlugin`基于**自由变量**运行，并在webpack分析源代码时替换它们。您可以使用Babel插件获得类似的结果。
* `DefinePlugin` operates based on **free variables** and it replaces them as webpack analyzes the source code. You can achieve similar results by using Babel plugins.
*鉴于minifiers消除了死代码，使用插件允许您从生成的构建中删除代码。
* Given minifiers eliminate dead code, using the plugins allows you to remove the code from the resulting build.
*插件启用模块级别模式。通过实现包装器，您可以选择webpack包含哪个文件到生成的构建中。
* The plugins enable module level patterns. By implementing a wrapper, you can choose which file webpack includes to the resulting build.
*除了这些插件之外，您还可以找到其他与优化相关的插件，它们允许您以多种方式控制构建结果。
* In addition to these plugins, you can find other optimization related plugins that allow you to control the build result in many ways.

为确保构建具有良好的缓存失效行为，您将学习在下一章中将哈希包含到生成的文件名中。这样，客户端会注意到资产是否已更改，并且可以获取更新的版本。
To ensure the build has good cache invalidation behavior, you'll learn to include hashes to the generated filenames in the next chapter. This way the client notices if assets have changed and can fetch the updated versions.

