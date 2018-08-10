＃ 介绍
# Introduction

[Webpack]（https://webpack.js.org/）通过解决基本问题简化了Web开发：捆绑。它接收各种资产，例如JavaScript，CSS和HTML，并将它们转换为便于通过浏览器使用的格式。这样做很好地消除了Web开发带来的巨大痛苦。
[Webpack](https://webpack.js.org/) simplifies web development by solving a fundamental problem: bundling. It takes in various assets, such as JavaScript, CSS, and HTML, and transforms them into a format that’s convenient to consume through a browser. Doing this well takes a significant amount of pain away from web development.

由于其配置驱动的方法，它不是最容易学习的工具，但它非常强大。本指南的目的是帮助你开始使用webpack，然后超越基础知识。
It's not the easiest tool to learn due to its configuration-driven approach, but it's incredibly powerful. The purpose of this guide is to help you get started with webpack and then go beyond the basics.

##什么是Webpack
## What Is Webpack

Web浏览器旨在使用HTML，CSS和JavaScript。随着项目的发展，跟踪和配置所有这些文件变得过于复杂，无法提供帮助。 Webpack旨在解决这些问题。管理复杂性是Web开发的基本问题之一，解决这个问题很有帮助。
Web browsers are designed to consume HTML, CSS, and JavaScript. As a project grows, tracking and configuring all of these files becomes too complicated to manage without help. Webpack was designed to address these problems. Managing complexity is one of the fundamental issues of web development, and solving this problem well helps significantly.

Webpack并不是唯一可用的捆绑器，并且已经出现了一系列不同的工具。任务运行者，如Grunt和Gulp，是高级工具的很好的例子。通常问题是你需要手动编写工作流程。将该问题推向捆绑商（例如webpack）是向前迈出的一步。
Webpack isn’t the only available bundler, and a collection of different tools have emerged. Task runners, such as Grunt and Gulp, are good examples of higher-level tools. Often the problem is that you need to write the workflows by hand. Pushing that issue to a bundler, such as webpack, is a step forward.

{pagebreak}

### Webpack如何改变这种情况
### How Webpack Changes The Situation

Webpack采取另一种方式。它允许你将项目视为依赖图。你可以在项目中使用* index.js *，通过标准的`require`或`import`语句引入项目所需的依赖项。如果需要，你可以以相同的方式引用样式文件和其他资源。
Webpack takes another route. It allows you to treat your project as a dependency graph. You could have an *index.js* in your project that pulls in the dependencies the project needs through the standard `require` or `import` statements. You can refer to your style files and other assets the same way if you want.

Webpack为你完成所有预处理，并为你提供通过配置和代码指定的软件包。这种声明式方法是通用的，但很难学习。
Webpack does all the preprocessing for you and gives you the bundles you specify through configuration and your code. This declarative approach is versatile, but it's difficult to learn.

在你开始了解Webpack的工作原理后，Webpack将成为不可或缺的工具。本书旨在通过最初的学习曲线，甚至更进一步。
Webpack becomes an indispensable tool after you begin to understand how it works. This book has been designed to get through that initial learning curve and even go further.

##你会学到什么？
## What Will You Learn

本书旨在补充[webpack的官方文档]（https://webpack.js.org/）。这本书可以被认为是它的伴侣。本书旨在通过最初的学习曲线，并进一步发展。
This book has been designed to complement [the official documentation of webpack](https://webpack.js.org/). This book can be considered a companion to it. This book has been designed to get through that initial learning curve and go even further.

本书教你开发用于开发和生产目的的可组合webpack配置。本书涵盖的高级技术使你可以充分利用webpack 4。
The book teaches you to develop a composable webpack configuration for both development and production purposes. Advanced techniques covered by the book allow you to get the most out of webpack 4.

{pagebreak}

##如何组织这本书
## How Is The Book Organized

本书首先解释了webpack是什么。之后，你将找到多个章节，从不同的角度讨论webpack。在阅读这些章节时，你将开发自己的webpack配置，同时学习基本技术。
The book starts by explaining what webpack is. After that, you will find multiple chapters that discuss webpack from a different viewpoint. As you go through these chapters, you will develop your own webpack configuration while at the same time learning essential techniques.

这本书分为以下几部分：
The book has been split into the following parts:

* **开发**可以让你运行webpack。此部分介绍了自动浏览器刷新等功能，并说明了如何组合配置以使其保持可维护状态。
* **Developing** gets you up and running with webpack. This part goes through features such as automatic browser refresh and explains how to compose your configuration so that it remains maintainable.
* **造型**非常注重造型相关主题。你将学习如何使用webpack加载样式以及如何在设置中引入自动修复等技术。
* **Styling** puts heavy emphasis on styling related topics. You will learn how to load styles with webpack and how to introduce techniques such as autoprefixing into your setup.
* ** Loading **详细解释了webpack的加载程序定义，并向你展示了如何加载图像，字体和JavaScript等资源。
* **Loading** explains webpack’s loader definitions in detail and shows you how to load assets such as images, fonts, and JavaScript.
* ** Building **介绍了源映射以及bundle和code splitting的思想。你将学会整理你的构建。
* **Building** introduces source maps and the ideas of bundle and code splitting. You will learn to tidy up your build.
* **优化**将你的构建推向生产质量水平，并引入许多较小的调整以使其更小。你将学习调整webpack的性能。
* **Optimizing** pushes your build to production quality level and introduces many smaller tweaks to make it smaller. You will learn to tune webpack for performance.
* **输出**讨论了webpack的输出相关技术。尽管它的名字，它不仅适用于网络。你将了解如何使用webpack管理多个页面设置，并了解服务器端呈现的基本概念。
* **Output** discusses webpack’s output related techniques. Despite its name, it’s not only for the web. You see how to manage multiple page setups with webpack and pick up the basic idea of Server Side Rendering.
* **技术**讨论了几个具体的想法，包括动态加载，Web工作者，国际化，部署应用程序，以及通过webpack使用npm包。
* **Techniques** discusses several specific ideas including dynamic loading, web workers, internationalization, deploying your applications, and consuming npm packages through webpack.
* **扩展**显示了如何使用加载器和插件扩展webpack。
* **Extending** shows how to extend webpack with loaders and plugins.

最后，有一个简短的结论章节回顾了本书的要点。它包含本书中的技术清单，允许你有条不紊地完成项目。
Finally, there is a short conclusion chapter that recaps the main points of the book. It contains checklists of techniques from this book that allow you to methodically go through your projects.

本书末尾的附录涵盖了第二个主题，有时还会深入探讨主要内容。你可以根据自己的兴趣以任何顺序接近它们。
The appendices at the end of the book cover secondary topics and sometimes dig deeper into the main ones. You can approach them in any order you want depending on your interest.

最后的*故障排除*附录介绍了当webpack给你一个错误时该怎么做。它涵盖了一个过程，因此你知道该怎么做以及如何调试该问题。如有疑问，请研究附录。如果你不确定术语及其含义，请参阅本书末尾的*词汇表*。
The *Troubleshooting* appendix at the end covers what to do when webpack gives you an error. It covers a process, so you know what to do and how to debug the problem. When in doubt, study the appendix. If you are unsure of a term and its meaning, see the *Glossary* at the end of the book.

##谁是这本书
## Who Is The Book For

你应该具备JavaScript，Node和npm的基本知识。如果你对webpack有所了解，那就太棒了。通过阅读本书，你可以加深对这些工具的理解。
You should have basic knowledge of JavaScript, Node, and npm. If you know something about webpack, that’s great. By reading this book, you deepen your understanding of these tools.

如果你对该主题知之甚少，请考虑仔细阅读早期部分。你可以扫描其余部分来挑选你觉得有价值的部分。如果你已经了解webpack，请浏览并选择你认为有价值的技术。
If you don’t know much about the topic, consider going carefully through the early parts. You can scan the rest to pick the bits you find worthwhile. If you know webpack already, skim and choose the techniques you find valuable.

如果你已经熟悉了webpack，那么本书中仍有一些内容供你使用。浏览它，看看你是否可以采用新技术。特别是阅读本章末尾和本书最后一章的摘要。
In case you know webpack well already, there is still something in the book for you. Skim through it and see if you can pick up new techniques. Especially read the summaries at the end of the chapters and the concluding chapter of the book.

## Book Versioning
## Book Versioning

鉴于本书因创新步伐而获得了相当多的维护和改进，因此有一个版本控制方案。每个新版本的发行说明都保存在[书籍博客]（https://survivejs.com/blog/）。你也可以使用GitHub * compare *工具来实现此目的。例：
Given this book receives a fair amount of maintenance and improvements due to the pace of innovation, there's a versioning scheme in place. Release notes for each new version are maintained at [the book blog](https://survivejs.com/blog/). You can also use GitHub *compare* tool for this purpose. Example:

```
https://github.com/survivejs/webpack-book/compare/v2.1.7...v2.4.1
```

该页面显示了在给定版本范围之间进入项目的各个提交。你还可以在书中看到已更改的行。
The page shows you the individual commits that went to the project between the given version range. You can also see the lines that have changed in the book.

该书的当前版本是** 2.4.1 **。
The current version of the book is **2.4.1**.

##获得支持
## Getting Support

如果你遇到麻烦或遇到与内容相关的问题，可以选择以下几种方法：
If you run into trouble or have questions related to the content, there are several options:

*通过[GitHub Issue Tracker]（https://github.com/survivejs/webpack-book/issues）与我联系。
* Contact me through [GitHub Issue Tracker](https://github.com/survivejs/webpack-book/issues).
*加入我的[Gitter Chat]（https://gitter.im/survivejs/webpack）。
* Join me at [Gitter Chat](https://gitter.im/survivejs/webpack).
*发送电子邮件至[info@survivejs.com]（mailto：info@survivejs.com）。
* Send me an email at [info@survivejs.com](mailto:info@survivejs.com).
*在[SurviveJS AmA]（https://github.com/survivejs/ama/issues）上向我询问有关webpack的任何信息。
* Ask me anything about webpack at [SurviveJS AmA](https://github.com/survivejs/ama/issues).

如果你向Stack Overflow发布问题，请使用** survivaljs **标记它们。你可以在Twitter上使用hashtag **＃survivaljs **获得相同的结果。
If you post questions to Stack Overflow, tag them using **survivejs**. You can use the hashtag **#survivejs** on Twitter for the same result.

##附加材料
## Additional Material

你可以从以下来源找到更多相关材料：
You can find more related material from the following sources:

*加入[邮件列表]（https://eepurl.com/bth1v5）进行不定期更新。
* Join the [mailing list](https://eepurl.com/bth1v5) for occasional updates.
*在Twitter上关注[@survivejs]（https://twitter.com/survivejs）。
* Follow [@survivejs](https://twitter.com/survivejs) on Twitter.
*订阅[博客RSS]（https://survivejs.com/atom.xml）以获取访问访谈等。
* Subscribe to the [blog RSS](https://survivejs.com/atom.xml) to get access interviews and more.
*订阅[Youtube频道]（https://www.youtube.com/channel/UCvUR-BJcbrhmRQZEEr4_bnw）。
* Subscribe to the [Youtube channel](https://www.youtube.com/channel/UCvUR-BJcbrhmRQZEEr4_bnw).
*查看[SurviveJS相关演示幻灯片]（https://presentations.survivejs.com/）。
* Check out [SurviveJS related presentation slides](https://presentations.survivejs.com/).

##致谢
## Acknowledgments

非常感谢[Christian Alfoni]（http://www.christianalfoni.com/）帮助我制作了本书的第一版。这就是整个SurviveJS努力的灵感来源。你现在看到的版本是完全重写的。
Big thanks to [Christian Alfoni](http://www.christianalfoni.com/) for helping me craft the first version of this book. This is what inspired the entire SurviveJS effort. The version you see now is a complete rewrite.

如果没有患者编辑和编辑的反馈，这本书不会好一半[JesúsRodríguez]（https://github.com/Foxandxss），[Artem Sapegin]（https://github.com/sapegin） ，和[Pedr Browne]（https://github.com/Undistraction）。谢谢。
This book wouldn’t be half as good as it's without patient editing and feedback by my editors [Jesús Rodríguez](https://github.com/Foxandxss), [Artem Sapegin](https://github.com/sapegin), and [Pedr Browne](https://github.com/Undistraction). Thank you.

没有最初的“SurviveJS  -  Webpack and React”努力，本书是不可能的。任何为此做出贡献的人都值得我的谢意。你可以查看该书以获得更准确的归因。
This book wouldn’t have been possible without the original "SurviveJS - Webpack and React" effort. Anyone who contributed to it deserves my thanks. You can check that book for more accurate attributions.

感谢Mike“Pomax”Kamermans，Cesar Andreu，Dan Palmer，ViktorJančík，Tom Byrer，Christian Hettlage，David A. Lee，Alexandar Castaneda，Marcel Olszewski，Steve Schwartz，Chris Sanders，Charles Ju，Aditya Bhardwaj，Rasheed Bustamam，José Menor，Ben Gale，Jake Goulding，Andrew Ferk，gabo，Giang Nguyen，@ Coaxial，@khronic，Henrik Raitasola，Gavin Orland，David Riccitelli，Stephen Wright，MajkyBašista，Gunnari Auvinen，JónLevy，Alexander Zaytsev，Richard Muller，Ava Mallory（Fiverr），Sun Zheng'an，Nancy（Fiverr），Aluan Haddad，Steve Mao，Craig McKenna，Tobias Koppers，Stefan Frede，Vladimir Grenaderov，Scott Thompson，Rafael De Leon，Gil Forcada Codinachs，Jason Aller，@ pikeshawn， Stephan Klinger，Daniel Carral，Nick Yianilos，Stephen Bolton，Felipe Reis，Rodolfo Rodriguez，Vicky Koblinski，Pyotr Ermishkin，Ken Gregory，Dmitry Kaminski，John Darryl Pelingo，Brian Cui，@ st-sloth，Nathan Klatt，Muhamadamin Ibragimov，Kema Akpala ，Roberto Fuentes，Eric Johnson，Luca Poldelmengo， Giovanni Iembo，Dmitry Anderson以及其他许多为本书提供直接反馈的人！
Thanks to Mike "Pomax" Kamermans, Cesar Andreu, Dan Palmer, Viktor Jančík, Tom Byrer, Christian Hettlage, David A. Lee, Alexandar Castaneda, Marcel Olszewski, Steve Schwartz, Chris Sanders, Charles Ju, Aditya Bhardwaj, Rasheed Bustamam, José Menor, Ben Gale, Jake Goulding, Andrew Ferk, gabo, Giang Nguyen, @Coaxial, @khronic, Henrik Raitasola, Gavin Orland, David Riccitelli, Stephen Wright, Majky Bašista, Gunnari Auvinen, Jón Levy, Alexander Zaytsev, Richard Muller, Ava Mallory (Fiverr), Sun Zheng’an, Nancy (Fiverr), Aluan Haddad, Steve Mao, Craig McKenna, Tobias Koppers, Stefan Frede, Vladimir Grenaderov, Scott Thompson, Rafael De Leon, Gil Forcada Codinachs, Jason Aller, @pikeshawn, Stephan Klinger, Daniel Carral, Nick Yianilos, Stephen Bolton, Felipe Reis, Rodolfo Rodriguez, Vicky Koblinski, Pyotr Ermishkin, Ken Gregory, Dmitry Kaminski, John Darryl Pelingo, Brian Cui, @st-sloth, Nathan Klatt, Muhamadamin Ibragimov, Kema Akpala, Roberto Fuentes, Eric Johnson, Luca Poldelmengo, Giovanni Iembo, Dmitry Anderson , and many others who have contributed direct feedback for this book!

