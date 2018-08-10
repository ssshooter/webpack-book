＃分离CSS
# Separating CSS

即使现在有一个很好的构建设置，所有的CSS都去了哪里？根据配置，它已被内联到JavaScript！虽然这在开发过程中很方便，但听起来并不理想。
Even though there is a nice build set up now, where did all the CSS go? As per configuration, it has been inlined to JavaScript! Even though this can be convenient during development, it doesn't sound ideal.

当前的解决方案不允许缓存CSS。你还可以获得** Flash of Unstyled Content **（FOUC）。发生FOUC是因为浏览器需要一段时间才能加载JavaScript，并且只会应用样式。将CSS分离到自己的文件可以让浏览器单独管理它，从而避免了这个问题。
The current solution doesn't allow cache CSS. You can also get a **Flash of Unstyled Content** (FOUC). FOUC happens because the browser takes a while to load JavaScript and the styles would be applied only then. Separating CSS to a file of its own avoids the problem by letting the browser to manage it separately.

Webpack提供了一种使用[mini-css-extract-plugin]（https://www.npmjs.com/package/mini-css-extract-plugin）（MCEP）生成单独的CSS包的方法。它可以将多个CSS文件聚合为一个。因此，它配备了一个处理提取过程的装载机。然后，插件会获取加载器聚合的结果并发出单独的文件。
Webpack provides a means to generate a separate CSS bundles using [mini-css-extract-plugin](https://www.npmjs.com/package/mini-css-extract-plugin) (MCEP). It can aggregate multiple CSS files into one. For this reason, it comes with a loader that handles the extraction process. The plugin then picks up the result aggregated by the loader and emits a separate file.

由于这个过程，`MiniCssExtractPlugin`在编译阶段带来了开销。它不适用于热模块更换（HMR）。鉴于插件仅用于生产，这不是问题。
Due to this process, `MiniCssExtractPlugin` comes with overhead during the compilation phase. It doesn't work with Hot Module Replacement (HMR) yet. Given the plugin is used only for production, that is not a problem.

W>在生产中使用JavaScript中的内联样式可能有潜在危险，因为它代表攻击向量。 **关键路径渲染**包含了这个想法，并将关键CSS内联到初始HTML有效负载，从而提高了网站的感知性能。在有限的上下文中，内联少量的CSS可能是加速初始加载（减少请求）的可行选择。
W> It can be potentially dangerous to use inline styles within JavaScript in production as it represents an attack vector. **Critical path rendering** embraces the idea and inlines the critical CSS to the initial HTML payload improving the perceived performance of the site. In limited contexts inlining a small amount of CSS can be a viable option to speed up the initial load (fewer requests).

##设置`MiniCssExtractPlugin`
## Setting Up `MiniCssExtractPlugin`

首先安装插件：
Install the plugin first:

```bash
npm install mini-css-extract-plugin --save-dev
```

`MiniCssExtractPlugin`包含一个加载器，'MiniCssExtractPlugin.loader`，用于标记要提取的资产。然后，插件基于此注释执行其工作。
`MiniCssExtractPlugin` includes a loader, `MiniCssExtractPlugin.loader` that marks the assets to be extracted. Then a plugin performs its work based on this annotation.

将以下配置添加到配置的开头：
Add the configuration below to the beginning of your configuration:

** ** webpack.parts.js
**webpack.parts.js**

```javascript
const MiniCssExtractPlugin = require("mini-css-extract-plugin");

exports.extractCSS = ({ include, exclude, use = [] }) => {
  // Output extracted CSS to a file
  const plugin = new MiniCssExtractPlugin({
    filename: "[name].css",
  });

  return {
    module: {
      rules: [
        {
          test: /\.css$/,
          include,
          exclude,

          use: [
            MiniCssExtractPlugin.loader,
          ].concat(use),
        },
      ],
    },
    plugins: [plugin],
  };
};
```

`[name]`占位符使用引用CSS的条目的名称。占位符和散列将在*添加散列到文件名*章节中详细讨论。
That `[name]` placeholder uses the name of the entry where the CSS is referred. Placeholders and hashing are discussed in detail in the *Adding Hashes to Filenames* chapter.

T>如果要将结果文件输出到特定目录，可以通过传递路径来完成。示例：`filename：“styles / [name] .css”`。
T> If you wanted to output the resulting file to a specific directory, you could do it by passing a path. Example: `filename: "styles/[name].css"`.

{pagebreak}

###连接配置
### Connecting with Configuration

使用以下配置连接功能：
Connect the function with the configuration as below:

** ** webpack.config.js
**webpack.config.js**

```javascript
const commonConfig = merge([
  ...
leanpub-start-delete
  parts.loadCSS(),
leanpub-end-delete
]);

leanpub-start-delete
const productionConfig = merge([]);
leanpub-end-delete
leanpub-start-insert
const productionConfig = merge([
  parts.extractCSS({
    use: "css-loader",
  }),
]);
leanpub-end-insert

const developmentConfig = merge([
  ...
leanpub-start-insert
  parts.loadCSS(),
leanpub-end-insert
]);
```

使用此设置，你仍然可以在开发过程中受益于HMR。但是对于生产构建，可以生成单独的CSS。 `HtmlWebpackPlugin`自动选择它并将其注入`index.html`。
Using this setup, you can still benefit from the HMR during development. For a production build, it's possible to generate a separate CSS, though. `HtmlWebpackPlugin` picks it up automatically and injects it into `index.html`.

T>如果你正在使用* CSS Modules *，请记住相应地调整`use`，如* Loading Styles *章节中所述。你可以为标准CSS和CSS模块维护单独的设置，以便通过离散逻辑加载它们。
T> If you are using *CSS Modules*, remember to tweak `use` accordingly as discussed in the *Loading Styles* chapter. You can maintain separate setups for standard CSS and CSS Modules so that they get loaded through discrete logic.

{pagebreak}

运行`npm run build`后，你应该看到类似于以下内容的输出：
After running `npm run build`, you should see output similar to the following:

```bash
Hash: 45a5e26cc963eb12db02
Version: webpack 4.1.1
Time: 752ms
Built at: 3/16/2018 4:24:40 PM
     Asset       Size  Chunks             Chunk Names
   main.js  700 bytes       0  [emitted]  main
  main.css   33 bytes       0  [emitted]  main
index.html  220 bytes          [emitted]
Entrypoint main = main.js main.css
   [0] ./src/index.js + 1 modules 247 bytes {0} [built]
       | ./src/index.js 99 bytes [built]
       | ./src/component.js 143 bytes [built]
   [1] ./src/main.css 41 bytes {0} [built]
...
```

现在样式已被推送到单独的CSS文件。因此，JavaScript包变得略小。你也避免了FOUC问题。浏览器不必等待JavaScript加载以获取样式信息。相反，它可以单独处理CSS，避免闪存。
Now styling has been pushed to a separate CSS file. Thus, the JavaScript bundle has become slightly smaller. You also avoid the FOUC problem. The browser doesn't have to wait for JavaScript to load to get styling information. Instead, it can process the CSS separately, avoiding the flash.

T>如果你得到`模块构建失败：CssSyntaxError：`或`模块构建失败：未知单词`错误，请确保你的`common`配置没有设置与CSS相关的部分。
T> If you are getting `Module build failed: CssSyntaxError:` or `Module build failed: Unknown word` error, make sure your `common` configuration doesn't have a CSS-related section set up.

{pagebreak}

##在JavaScript之外管理样式
## Managing Styles Outside of JavaScript

即使通过JavaScript引用样式然后捆绑是推荐选项，也可以通过`entry`和[globbing]（https://www.npmjs.com/package/glob）通过CSS文件实现相同的结果一个条目：
Even though referring to styling through JavaScript and then bundling is the recommended option, it's possible to achieve the same result through an `entry` and [globbing](https://www.npmjs.com/package/glob) the CSS files through an entry:

```javascript
...
const glob = require("glob");

...

const commonConfig = merge([
  {
    entry: {
      ...
      style: glob.sync("./src/**/*.css"),
    },
    ...
  },
  ...
]);
```

在进行此类更改后，你不必从应用程序代码中引用样式。这也意味着CSS模块停止工作。你也必须小心CSS排序。
After this type of change, you would not have to refer to styling from your application code. It also means that CSS Modules stop working. You have to be careful with CSS ordering as well.

因此，你应该同时获得* style.css *和* style.js *。后一个文件包含的内容如`webpackJsonp（[1,3]，[function（n，c）{}]）;`它没有做任何事情，如[webpack issue 1967]（https：// github）所述.COM /的WebPack /的WebPack /问题/ 1967）。
As a result, you should get both *style.css* and *style.js*. The latter file contains content like `webpackJsonp([1,3],[function(n,c){}]);` and it doesn't do anything as discussed in the [webpack issue 1967](https://github.com/webpack/webpack/issues/1967).

如果要严格控制排序，可以设置一个CSS条目，然后使用`@ import`通过它将其余部分带到项目中。另一个选择是设置一个JavaScript条目并通过`import`来获得相同的效果。
If you want strict control over the ordering, you can set up a single CSS entry and then use `@import` to bring the rest to the project through it. Another option would be to set up a JavaScript entry and go through `import` to get the same effect.

T> [css-entry-webpack-plugin]（https://www.npmjs.com/package/css-entry-webpack-plugin）旨在帮助实现此使用模式。该插件可以从条目中提取CSS包而不使用MCEP。
T> [css-entry-webpack-plugin](https://www.npmjs.com/package/css-entry-webpack-plugin) has been designed to help with this usage pattern. The plugin can extract a CSS bundle from entry without MCEP.

##结论
## Conclusion

当前的设置将样式与JavaScript整齐地分开。即使该技术对CSS最有价值，它也可用于提取HTML模板或你使用的任何其他文件类型。关于`MiniCssExtractPlugin`的难点在于它的设置，但复杂性可以隐藏在抽象背后。
The current setup separates styling from JavaScript neatly. Even though the technique is most valuable with CSS, it can be used to extract HTML templates or any other files types you consume. The hard part about `MiniCssExtractPlugin` has to do with its setup, but the complexity can be hidden behind an abstraction.

回顾一下：
To recap:

*使用带有样式的“MiniCssExtractPlugin”解决了无格式内容Flash（FOUC）的问题。从JavaScript中分离CSS还可以改善缓存行为并删除潜在的攻击媒介。
* Using `MiniCssExtractPlugin` with styling solves the problem of Flash of Unstyled Content (FOUC). Separating CSS from JavaScript also improves caching behavior and removes a potential attack vector.
*如果你不希望通过JavaScript维护对样式的引用，则可以选择通过条目来处理它们。不过，在这种情况下你必须小心样式排序。
* If you don't prefer to maintain references to styling through JavaScript, an alternative is to handle them through an entry. You have to be careful with style ordering in this case, though.

在下一章中，你将学习如何从项目中消除未使用的CSS。
In the next chapter, you'll learn to eliminate unused CSS from the project.

