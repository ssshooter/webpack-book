# 动态加载

尽管你可以通过 **Code Splitting** 章节中介绍的 webpack 代码拆分功能获得更多功能，但还有更多功能。 Webpack通过 `require.context` 提供了更多动态的方法来处理代码。
Even though you can get far with webpack's code splitting features covered in the *Code Splitting* chapter, there's more to it. Webpack provides more dynamic ways to deal with code through `require.context`.

## 使用 `require.context` 动态加载

[require.context](https://webpack.js.org/api/module-methods/#require-context) 提供了一种代码分割的一般形式。假设你正在webpack上编写静态站点生成器。你可以通过包含Markdown文件的`./ pages /`目录在目录结构中建模你的站点内容。
[require.context](https://webpack.js.org/api/module-methods/#require-context) provides a general form of code splitting. Let's say you are writing a static site generator on top of webpack. You could model your site contents within a directory structure by having a `./pages/` directory which would contain the Markdown files.

这些文件中的每一个都有一个用于元数据的YAML前端。可以基于文件名确定每个页面的URL并将其映射为站点。要使用`require.context`对这个想法进行建模，你可能会得到如下代码：
Each of these files would have a YAML frontmatter for their metadata. The url of each page could be determined based on the filename and mapped as a site. To model the idea using `require.context`, you could end up with the code as below:

```javascript
// Process pages through `yaml-frontmatter-loader` and `json-loader`.
// The first one extracts the front matter and the body and the latter
// converts it into a JSON structure to use later. Markdown
// hasn't been processed yet.
const req = require.context(
  "json-loader!yaml-frontmatter-loader!./pages",
  true, // Load files recursively. Pass false to skip recursion.
  /^\.\/.*\.md$/ // Match files ending with .md.
);
```

T> 可以将加载器定义推送到 webpack 配置。内联表单用于保持示例最小化。
T> The loader definition could be pushed to webpack configuration. The inline form is used to keep the example minimal.

`require.context`将函数返回给`require`。它还知道它的模块`id`，它提供了一个`keys（）`方法来计算上下文的内容。为了给你一个更好的示例，请考虑以下代码：
`require.context` returns a function to `require` against. It also knows its module `id` and it provides a `keys()` method for figuring out the contents of the context. To give you a better example, consider the code below:

```javascript
req.keys(); // ["./demo.md", "./another-demo.md"]
req.id; // 42

// {title: "Demo", body: "# Demo page\nDemo content\n\n"}
const demoPage = req("./demo.md");
```

该技术可用于其他目的，例如测试或添加webpack以供观看的文件。在这种情况下，你可以在文件中设置`require.context`，然后通过webpack`条目指向该文件。
The technique can be valuable for other purposes, such as testing or adding files for webpack to watch. In that case, you would set up a `require.context` within a file which you then point to through a webpack `entry`.

T> 这些信息足以生成[Antwar](https://github.com/antwarjs/antwar) 中展示的整个网站。
T> The information is enough for generating an entire site as showcased in [Antwar](https://github.com/antwarjs/antwar).

## 使用动态 `import` 的动态路径

同样的想法适用于动态 `import`。你可以传递部分路径，而不是传递完整路径。Webpack 在内部设置上下文。这是一个简短的例子：
The same idea works with dynamic `import`. Instead of passing a complete path, you can pass a partial one. Webpack sets up a context internally. Here's a brief example:

```javascript
// Set up a target or derive this somehow
const target = "fi";

// Elsewhere in code
import(`translations/${target}.json`).then(...).catch(...);
```

同样的想法适用于`require`，因为webpack可以执行静态分析。例如，`require(`assets/modals/${imageSrc}.js`);` 将生成一个上下文并根据传递给`require`的`imageSrc`来解析图片。
The same idea works with `require` as webpack can then perform static analysis. For example, `require(`assets/modals/${imageSrc}.js`);` would generate a context and resolve against an image based on the `imageSrc` that was passed to the `require`.

T> 使用动态导入时，请在路径中指定文件扩展名，以便通过保持较小的上下文来提高性能。
T> When using dynamic imports, specify file extension in the path as that helps with performance by keeping the context smaller.

## 组合多个 `require.context`
## Combining Multiple `require.context`s

多个单独的`require.context`可以通过将它们包装在函数后面来组合成一个：
Multiple separate `require.context`s can be combined into one by wrapping them behind a function:

```javascript
const { concat, uniq } = require("lodash");

const combineContexts = (...contexts) => {
  function webpackContext(req) {
    // Find the first match and execute
    const matches = contexts
      .map(context => context.keys().indexOf(req) >= 0 && context)
      .filter(a => a);

    return matches[0] && matches[0](req);
  }
  webpackContext.keys = () =>
    uniq(
      concat.apply(null, contexts.map(context => context.keys()))
    );

  return webpackContext;
};
```

{pagebreak}

##处理动态路径
## Dealing with Dynamic Paths

鉴于这里讨论的方法依赖于静态分析，webpack必须找到有问题的文件，它并不适用于所有可能的情况。如果你需要的文件在另一台服务器上或必须通过特定端点访问，那么webpack是不够的。
Given the approaches discussed here rely on static analysis and webpack has to find the files in question, it doesn't work for every possible case. If the files you need are on another server or have to be accessed through a particular end-point, then webpack isn't enough.

考虑使用浏览器端加载器，如[$ script.js]（https://www.npmjs.com/package/scriptjs）或[little-loader]（https://www.npmjs.com/package/little-loader ）在这种情况下，在webpack之上。
Consider using browser-side loaders like [$script.js](https://www.npmjs.com/package/scriptjs) or [little-loader](https://www.npmjs.com/package/little-loader) on top of webpack in this case.

## 总结


尽管`require.context`是一个小众特色，但要注意它是件好事。如果你必须对文件系统中可用的多个文件执行查找，这将变得很有价值。如果你的查找比这更复杂，则必须使用其他允许你执行加载运行时的替代方法。
Even though `require.context` is a niche feature, it's good to be aware of it. It becomes valuable if you have to perform lookups against multiple files available within the file system. If your lookup is more complicated than that, you have to resort to other alternatives that allow you to perform loading runtime.

回顾一下：


*`require.context`是一种经常隐藏在幕后的高级功能。如果必须对大量文件执行查找，请使用它。
* `require.context` is an advanced feature that's often hidden behind the scenes. Use it if you have to perform a lookup against a large number of files.
*如果以某种形式编写动态`import`，webpack会生成`require.context`调用。在这种情况下，代码读取稍微好一些。
* If you write a dynamic `import` in a certain form, webpack generates a `require.context` call. The code reads slightly better in this case.
*这些技术仅适用于文件系统。如果你必须对网址进行操作，则应该考虑客户端解决方案。
* The techniques work only against the file system. If you have to operate against urls, you should look into client-side solutions.

下一章将介绍如何将web worker与webpack一起使用。
The next chapter shows how to use web workers with webpack.

