#Autoprefixing
# Autoprefixing

记住哪些供应商前缀必须用于特定的CSS规则以支持各种各样的用户，这可能很有挑战性。 ** Autoprefixing **解决了这个问题。它可以通过PostCSS和[autoprefixer]（https://www.npmjs.com/package/autoprefixer）插件启用。 * autoprefixer *使用[我可以使用]（http://caniuse.com/）服务来确定哪些规则应该加上前缀，并且可以进一步调整其行为。
It can be challenging to remember which vendor prefixes you have to use for specific CSS rules to support a large variety of users. **Autoprefixing** solves this problem. It can be enabled through PostCSS and the [autoprefixer](https://www.npmjs.com/package/autoprefixer) plugin. *autoprefixer* uses [Can I Use](http://caniuse.com/) service to figure out which rules should be prefixed and its behavior can be tuned further.

##设置自动修复
## Setting Up Autoprefixing

实现自动修复需要对当前设置进行少量添加。首先安装* postcss-loader *和* autoprefixer *：
Achieving autoprefixing takes a small addition to the current setup. Install *postcss-loader* and *autoprefixer* first:

```bash
npm install postcss-loader autoprefixer --save-dev
```

添加启用自动修复的片段：
Add a fragment enabling autoprefixing:

** ** webpack.parts.js
**webpack.parts.js**

```javascript
exports.autoprefix = () => ({
  loader: "postcss-loader",
  options: {
    plugins: () => [require("autoprefixer")()],
  },
});
```

{pagebreak}

要使用CSS提取连接加载器，请按如下方式将其挂钩：
To connect the loader with CSS extraction, hook it up as follows:

** ** webpack.config.js
**webpack.config.js**

```javascript
const productionConfig = merge([
  parts.extractCSS({
leanpub-start-delete
    use: "css-loader",
leanpub-end-delete
leanpub-start-insert
    use: ["css-loader", parts.autoprefix()],
leanpub-end-insert
  }),
  ...
]);
```

要确认设置是否有效，我们必须向autoprefix添加一些内容。调整CSS：
To confirm that the setup works, we have to add something to autoprefix. Adjust the CSS:

**应用程序/ **的main.css
**app/main.css**

```css
...

leanpub-start-insert
.pure-button {
  -webkit-border-radius: 1em;
  border-radius: 1em;
}
leanpub-end-insert
```

如果你知道你希望支持哪些浏览器，则可以设置[.browserslistrc]（https://www.npmjs.com/package/browserslist）文件。不同的工具选择此定义，包括* autoprefixer *。
If you know what browsers you prefer to support, it's possible to set up a [.browserslistrc](https://www.npmjs.com/package/browserslist) file. Different tools pick up this definition, *autoprefixer* included.

T>你可以通过[Stylelint]（http://stylelint.io/）来lint CSS。它可以通过* postcss-loader *以相同的方式设置为上面的autoprefixing。
T> You can lint CSS through [Stylelint](http://stylelint.io/). It can be set up the same way through *postcss-loader* as autoprefixing above.

{pagebreak}

设置文件如下：
Set up a file as follows:

**。** browserslistrc
**.browserslistrc**

```
> 1% # Browser usage over 1%
Last 2 versions # Or last two versions
IE 8 # Or IE 8
```

如果你现在构建应用程序（`npm run build`）并检查构建的CSS，你应该能够在没有webkit部分的情况下找到一个声明：
If you build the application now (`npm run build`) and examine the built CSS, you should be able to find a declaration there without the webkit portion:

```css
...

leanpub-start-insert
.pure-button {
  border-radius: 1em;
}
leanpub-end-insert
```

* autoprefixer *能够**删除**不必要的规则，并根据浏览器定义添加所需的规则。
*autoprefixer* is able to **remove** unnecessary rules and also add rules which are required based on the browser definition.

##总结
## Conclusion

自动修复是一种方便的技术，因为它减少了制作CSS时所需的工作量。你可以在* .browserslistrc *文件中维护最低浏览器要求。然后，工具可以使用该信息来生成最佳输出。
Autoprefixing is a convenient technique as it decreases the amount of work needed while crafting CSS. You can maintain minimum browser requirements within a *.browserslistrc* file. The tooling can then use that information to generate optimal output.

回顾一下：
To recap:

*可以通过* autoprefixer * PostCSS插件启用自动修复。
* Autoprefixing can be enabled through the *autoprefixer* PostCSS plugin.
* Autoprefixing根据你的最低浏览器定义编写缺少的CSS定义。
* Autoprefixing writes missing CSS definitions based on your minimum browser definition.
* * .browserslistrc *是一个标准文件，适用于* autoprefixer之外的工具*
* *.browserslistrc* is a standard file that works with tooling beyond *autoprefixer*

