＃国际化
# Internationalization

**国际化**（i18n）本身就是一个很大的话题。最广泛的定义与将用户界面转换为其他语言有关。 **本地化**（l10n）是一个更具体的术语，它描述了如何使你的应用程序适应特定的区域或市场。不同的语言环境可以使用相同的语言，但它们仍然具有其习惯，例如日期格式或度量。
**Internationalization** (i18n) is a big topic by itself. The broadest definition has to do with translating your user interface to other languages. **Localization** (l10n) is a more specific term, and it describes how to adapt your application to a particular locale or market. Different locales can have the same language, but they still have their customs, like date formatting or measures.

这个问题可以通过推动端点后面的翻译并动态加载它们来解决webpack的问题来解决。这样做还可以让你在应用程序中实现翻译界面，使翻译人员甚至用户能够翻译应用程序。这种方法的缺点是，你有一个翻译后端来维护。
The problem could be solved by pushing the translations behind an endpoint and loading them dynamically to decouple the issue from webpack. Doing this would also allow you to implement a translation interface within your application to enable your translators, or even users, to translate the application. The downside of this approach is that then you have a translation backend to maintain.

另一种方法是让webpack生成每种语言的静态构建。问题是每次翻译更改时都必须更新应用程序。
Another approach is to let webpack generate static builds, each per language. The problem is that you have to update your application each time your translations change.

## i18n与Webpack
## i18n with Webpack

i18n与webpack的基本思想通常是一样的。你有一个转换定义，然后通过替换映射到应用程序。结果包含应用程序的翻译版本。你可以通过几种解决方案使用多种翻译格式：
The basic idea of i18n with webpack is often the same. You have a translation definition that is then mapped to the application through replacements. The result contains a translated version of the application. You can use multiple translation formats through a couple of solutions:

* [i18n-webpack-plugin]（https://www.npmjs.com/package/i18n-webpack-plugin）依赖于纯JSON定义，并通过`__（“Hello”）`占位符执行替换。
* [i18n-webpack-plugin](https://www.npmjs.com/package/i18n-webpack-plugin) relies on a pure JSON definition and performs the replacement through `__("Hello")` placeholders.
* [po-loader]（https://www.npmjs.com/package/po-loader）映射[GNU gettext PO文件]（https://www.gnu.org/software/gettext/manual/html_node/PO -Files.html）到多种格式，包括原始JSON和[Jed]（https://messageformat.github.io/Jed/）。
* [po-loader](https://www.npmjs.com/package/po-loader) maps [GNU gettext PO files](https://www.gnu.org/software/gettext/manual/html_node/PO-Files.html) to multiple formats including raw JSON and [Jed](https://messageformat.github.io/Jed/).
* [jed-webpack-plugin]（https://www.npmjs.com/package/jed-webpack-plugin）是一个基于插件的Jed解决方案。
* [jed-webpack-plugin](https://www.npmjs.com/package/jed-webpack-plugin) is a plugin-based solution for Jed.
* [globalize-webpack-plugin]（https://www.npmjs.com/package/globalize-webpack-plugin）补充[globalize]（https://www.npmjs.com/package/globalize）for i18n / l10n目的。
* [globalize-webpack-plugin](https://www.npmjs.com/package/globalize-webpack-plugin) complements [globalize](https://www.npmjs.com/package/globalize) for i18n/l10n purposes.

为了说明设置，* i18n-webpack-plugin *是一个很好的起点。
To illustrate the setup, *i18n-webpack-plugin* is a good starting point.

##设置项目
## Setting Up a Project

要证明翻译有效，请设置要替换的内容：
To prove that translation works, set up something to replace:

**应用程序/ i18n.js **
**app/i18n.js**

```javascript
console.log(__("Hello world"));
```

要将其翻译成芬兰语，请设置定义：
To translate that into Finnish, set up a definition:

**语言/ fi.json **
**languages/fi.json**

```json
{ "Hello world": "Terve maailma" }
```

下一步是使用webpack将文件粘合在一起。
The next step is to glue the files together using webpack.

T>要让ESLint知道全局`__`函数，你应该通过`globals .__：true`将它添加到你的linting规则中。
T> To make ESLint aware of the global `__` function, you should add it to your linting rules through `globals.__: true`.

{pagebreak}

##设置`I18n Webpack插件`
## Setting Up `I18nWebpackPlugin`

首先安装* i18n-webpack-plugin *和* glob * helper。后者是捕获翻译文件所必需的。
Install *i18n-webpack-plugin* and *glob* helper first. The latter is needed for capturing translation files.

```bash
npm install glob i18n-webpack-plugin --save-dev
```

在webpack方面，你应该遍历可用语言，然后为每个语言设置配置：
On the webpack side, you should iterate through the available languages, and then set up a configuration for each:

** ** webpack.i18n.js
**webpack.i18n.js**

```javascript
const path = require("path");
const glob = require("glob");
const I18nPlugin = require("i18n-webpack-plugin");

const PATHS = {
  build: path.join(__dirname, "i18n-build"),
  i18nDemo: path.join(__dirname, "app", "i18n.js"),
};

const TRANSLATIONS = [{ language: "en" }].concat(
  glob.sync("./languages/*.json").map(file => ({
    language: path.basename(file, path.extname(file)),
    translation: require(file),
  }))
);

module.exports = TRANSLATIONS.map(({ language, translation }) => ({
  entry: {
    index: PATHS.i18nDemo,
  },
  output: {
    path: PATHS.build,
    filename: `[name].${language}.js`,
  },
  plugins: [new I18nPlugin(translation)],
}));
```

为方便构建，请设置快捷方式：
To make it convenient to build, set a shortcut:

** **的package.json
**package.json**

```json
"scripts": {
  "build:i18n": "webpack --config webpack.i18n.js",
  ...
},
```

如果你现在构建（`npm run build：i18n`），你应该得到一个新目录，其中包含两个翻译文件和每个翻译过的代码。
If you build now (`npm run build:i18n`), you should end up with a new directory containing two translated files and translated code in each.

要进一步举例，请按照* Multiple Pages *章节中的说明为每个翻译生成一个页面，然后添加一个语言选择器。语言定义可以通过webpack的“DefinePlugin”来处理。用户界面小部件可以依赖于该小部件并基于页面或目录命名约定来加载语言。
To take the example further, generate a page for each translation as described in the *Multiple Pages* chapter and add a language selector. The language definition can be handled through webpack's `DefinePlugin`. A user interface widget could rely on that and load languages based on a page or directory naming convention.

* Code Splitting *章节中讨论的技术对i18n有效。你可以定义动态`import`s以按需加载翻译文件。这样做会推动在其他地方加载和维护翻译的问题。
The techniques discussed in the *Code Splitting* chapter are valid with i18n. You could define dynamic `import`s to load translation files on demand. Doing this would push the problem of loading and maintaining translations elsewhere.

{pagebreak}

##结论
## Conclusion

其他webpack方法遵循类似的想法，更灵活，但需要更多的工作。如果你使用基于加载器的解决方案，则可以设置拆分点以按需加载语言。
The other webpack approaches follow a similar idea and are more flexible but require more work. If you go with a loader based solution, then you can set up split points to load languages on demand.

回顾一下：
To recap:

* **国际化**（i18n）和**本地化**（l10n）是你在申请时针对多个市场的重要问题。
* **Internationalization** (i18n) and **localization** (l10n) are important problems if you target multiple markets with your application.
* Webpack支持多种i18n方法。作为起点，你可以替换特定注释，尽管可以使用更复杂的替代方案。
* Webpack supports multiple approaches to i18n. As a starting point you can replace specific annotations although more sophisticated alternatives are available.
*可以通过将其推送到服务器来解决问题。它还允许你通过相同的API处理实际应用程序的转换。
* The problem can be handled by pushing it to a server. It would also allow you to handle translating the actual application through the same API.

下一章将介绍与webpack一起使用的各种测试设置和工具。
The next chapter covers various testing setups and tools that work with webpack.

