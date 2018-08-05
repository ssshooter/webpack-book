#CSS模块
# CSS Modules

也许CSS最重要的挑战是所有规则都存在于**全局范围**中，这意味着两个具有相同名称的类将发生冲突。限制是CSS规范固有的，但项目有解决问题的方法。 [CSS模块]（https://github.com/css-modules/css-modules）为每个模块引入了**本地范围**，方法是通过在名称中包含一个全局唯一的哈希值来声明每个类都是唯一的模块。
Perhaps the most significant challenge of CSS is that all rules exist within **global scope**, meaning that two classes with the same name will collide. The limitation is inherent to the CSS specification, but projects have workarounds for the issue. [CSS Modules](https://github.com/css-modules/css-modules) introduces **local scope** for every module by making every class declared within unique by including a hash in their name that is globally unique to the module.

## CSS模块通过* css-loader *
## CSS Modules Through *css-loader*

Webpack的* css-loader *支持CSS模块。您可以通过上面的加载程序定义启用它，同时启用支持：
Webpack's *css-loader* supports CSS Modules. You can enable it through a loader definition as above while enabling the support:

```javascript
{
  use: {
    loader: "css-loader",
    options: {
      modules: true,
    },
  },
},
```

完成此更改后，您的类定义将保留在文件的本地。如果你想要全局类定义，你需要将它们包装在`：global（.redButton）{...}种类的声明中。
After this change, your class definitions remain local to the files. In case you want global class definitions, you need to wrap them within `:global(.redButton) { ... }` kind of declarations.

{pagebreak}

在这种情况下，`import`语句为您提供了可以绑定到元素的本地类。假设你有CSS如下：
In this case, the `import` statement gives you the local classes you can then bind to elements. Assume you had CSS as below:

**应用程序/ **的main.css
**app/main.css**

```css
body {
  background: cornsilk;
}

.redButton {
  background: red;
}
```

然后，您可以将结果类绑定到组件：
You could then bind the resulting class to a component:

**应用程序/ component.js **
**app/component.js**

```javascript
import styles from "./main.css";

...

// Attach the generated class name
element.className = styles.redButton;
```

`body`仍然是一个全球宣言。正是那种“redButton”才有所作为。您可以构建特定于组件的样式，这些样式不会以其他方式泄漏。
`body` remains as a global declaration still. It's that `redButton` that makes the difference. You can build component-specific styles that don't leak elsewhere this way.

CSS模块提供了诸如合成之类的附加功能，以便更轻松地使用您的样式。只要在* css-loader *之前应用它们，您也可以将它与其他加载器结合使用。
CSS Modules provides additional features like composition to make it easier to work with your styles. You can also combine it with other loaders as long as you apply them before *css-loader*.

T> CSS模块行为可以修改[正如官方文档中所讨论的]（https://www.npmjs.com/package/css-loader#local-scope）。例如，您可以控制它生成的名称。
T> CSS Modules behavior can be modified [as discussed in the official documentation](https://www.npmjs.com/package/css-loader#local-scope). You have control over the names it generates for instance.

T> [eslint-plugin-css-modules]（https://www.npmjs.com/package/eslint-plugin-css-modules）可以方便地跟踪CSS模块相关的问题。
T> [eslint-plugin-css-modules](https://www.npmjs.com/package/eslint-plugin-css-modules) is handy for tracking CSS Modules related problems.

##将CSS模块与第三方库和CSS一起使用
## Using CSS Modules with Third Party Libraries and CSS

如果您在项目中使用CSS模块，则应该通过单独的加载器定义处理标准CSS，而不启用* css-loader *的`modules`选项。否则，所有类都将作用于其模块。对于第三方库，这几乎肯定不是您想要的。
If you are using CSS Modules in your project, you should process standard CSS through a separate loader definition without the `modules` option of *css-loader* enabled. Otherwise, all classes will be scoped to their module. In the case of third-party libraries, this is almost certainly not what you want.

您可以通过针对* node_modules *的`include`定义以不同方式处理第三方CSS来解决问题。或者，您可以使用文件扩展名（`.mcss`）来使用CSS模块分析文件，然后在装载程序`test`中管理这种情况。
You can solve the problem by processing third-party CSS differently through an `include` definition against *node_modules*. Alternately, you could use a file extension (`.mcss`) to tell files using CSS Modules apart from the rest and then manage this situation in a loader `test`.

##结论
## Conclusion

CSS模块通过默认为每个文件的本地范围来解决CSS的范围问题。您仍然可以拥有全局样式，但需要额外的努力。如上所示，可以将Webpack设置为轻松支持CSS模块。
CSS Modules solve the scoping problem of CSS by defaulting to local scope per file. You can still have global styling, but it requires additional effort. Webpack can be set up to support CSS Modules easily as seen above.

