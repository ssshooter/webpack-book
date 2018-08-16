---
id: 03
title: 介绍
---

> 原文链接：https://survivejs.com/webpack/introduction/

[Webpack](https://webpack.js.org/) 解决了 Web 开发的基本问题：打包，从而简化了 Web 开发。它接收各种资源，例如 JavaScript，CSS 和 HTML，并将它们转换为便于通过浏览器使用的格式。这样做很好地消除了 Web 开发带来的巨大痛苦。

因为其配置驱动（configuration-driven）的实现，要学习 Webpack 并不简单，但它非常强大。本指南的目的是从 0 教你如何使用 webpack，并深入理解 webpack。

## 什么是 Webpack

Web 浏览器用于浏览 HTML，CSS 和 JavaScript。随着项目的发展，跟踪和配置所有文件变得过于复杂，不借助工具实在难以维护。Webpack 旨在解决这个问题。管理的复杂性是 Web 开发面对的基本问题之一，解决这个问题是必须的。

打包工具不只有 Webpack，一系列不同的工具已经出现了。任务运行者，如 Grunt 和 Gulp 都是不错的上层（与底层相对）工具。但问题是你需要手写工作流程，把这一步交给打包工具（例如 webpack），是前端项目构建向前的一大步。


### Webpack 如何改变现状

Webpack 采取另一种方式。它允许你将项目视为一个依赖图。你可以在项目中使用 *index.js* 通过标准的 `require` 或 `import` 语句引入项目所需的依赖项。如果需要，你甚至可以以相同的方式引用样式文件和其他资源。

Webpack 为你完成所有预处理，并根据你提供的配置文件生成生产包，配置文件功能多样，但不容易学习。

在你开始了解 Webpack 的工作原理后，Webpack 将成为不可或缺的工具。希望你看完本书可以从入门到精通 webpack。

## 你能学到什么

本书旨在补充 [webpack的官方文档](https://webpack.js.org/)。你可以同时阅读文档和本书。

本书教你开发用于开发环境和生产环境的可组合 webpack 配置。本书涵盖的高级技术可以让你充分利用webpack 4。


## 这本书的结构

本书首先解释了 webpack 是什么。之后的章节会从不同的角度讨论 webpack。在阅读这些章节时，你可以开发自己的 webpack 配置，同时学习一些基本技巧。

这本书分为以下几部分：

* **开发（Developing）** 教你运行 webpack。此部分介绍了浏览器自动刷新等功能，并说明了如何组合配置以使其保持可维护状态。
* **样式（Styling）** 重点介绍样式相关问题。你将学习如何使用 webpack 加载样式以及如何在设置中使用 autoprefix 等技术。
* **加载（Loading）** 详细解释 webpack loader，告诉你如何加载图片、字体和 JavaScript 文件等资源。
* **构建（Building）** 介绍 source maps 和 bundle/代码拆分的思想，教你如何整理构建所得的文件。
* **优化（Optimizing）** 将你的构建提高到生产质量水平，通过调整压缩它的体积，并学习 webpack 性能优化。
* **输出（Output）** 讨论了 webpack 输出的相关技术。你将了解如何使用 webpack 管理多页面设置，并了解服务端渲染的基本概念。
* **技术（Techniques）** 讨论了几个话题：动态加载，Web worker，i18n，应用程序部署，以及通过 webpack 使用 npm 包。
* **扩展** 展示如何扩展 loader 和 plugin。

最后，有一个简短的总结章节回顾了本书的要点。它包含本书中的技术清单，让你条理清晰地完成你的项目。

本书末尾的附录涵盖了一些次要主题或是深入探讨前面提到的主要内容。你可以以自己喜欢的顺序阅读本书。

最后的 *Troubleshooting* 附录介绍了当 webpack 报错时该怎么做。阅读本书时如有疑问，可以看看附录。如果你不清楚一些术语及其含义，请参阅本书末尾的 *Glossary*。

## 谁适合看这本书

你应具备 JavaScript，Node 和 npm 的基本知识。如果你对 webpack 已经有初步了解，那就更好了。通过阅读本书，你可以加深对这些工具的理解。

如果你对 webpack 所知之甚少，请考虑仔细阅读本书前半部分。如果你已经了解 webpack，请选择你认为有价值的技术浏览。

如果你已经熟悉了 webpack，那么本书中仍有一些内容为你所写。看看你是否有什么新技术可以采用。特别是阅读本章末尾和本书最后一章的摘要。

## 本书的版本

因本书的创新，获得了相当多的维护和改进，所以必须有一个版本控制方案。每个新版本的发行说明都保存在[本书博客](https://survivejs.com/blog/)。你也可以使用 GitHub *compare* 工具对比版本间差异：

```
https://github.com/survivejs/webpack-book/compare/v2.1.7...v2.4.1
```

该页面显示了在给定版本范围之间的各个提交。你还可以在本书已修改的地方。

本书的当前版本是 **2.4.1**。

## 获得支持

如果你遇到麻烦或与内容相关的问题，可以选择以下几种方法：

* 在 [GitHub Issue](https://github.com/survivejs/webpack-book/issues) 联系我。
* 加入我的 [Gitter Chat](https://gitter.im/survivejs/webpack)。
* 发送电子邮件至[info@survivejs.com](mailto:info@survivejs.com)。
* 在[SurviveJS AmA](https://github.com/survivejs/ama/issues) 上向我询问有关 webpack 的任何问题。

如果你在 Stack Overflow 发布问题，请使用 **survivaljs** 标记它们。你可以在 Twitter 上使用 **#survivaljs**。

## 附加材料

你可以从以下来源找到更多相关材料：

* 加入[订阅](https://eepurl.com/bth1v5)获取不定期更新。
* 在 Twitter 关注 [@survivejs](https://twitter.com/survivejs)。
* 订阅[博客 RSS](https://survivejs.com/atom.xml) 以获取访问访谈等。
* 订阅[Youtube频道](https://www.youtube.com/channel/UCvUR-BJcbrhmRQZEEr4_bnw)。
* 查看 [SurviveJS 相关幻灯片]（https://presentations.survivejs.com/）。

## 致谢

Big thanks to [Christian Alfoni](http://www.christianalfoni.com/) for helping me craft the first version of this book. This is what inspired the entire SurviveJS effort. The version you see now is a complete rewrite.

This book wouldn’t be half as good as it's without patient editing and feedback by my editors [Jesús Rodríguez](https://github.com/Foxandxss), [Artem Sapegin](https://github.com/sapegin), and [Pedr Browne](https://github.com/Undistraction). Thank you.

This book wouldn’t have been possible without the original "SurviveJS - Webpack and React" effort. Anyone who contributed to it deserves my thanks. You can check that book for more accurate attributions.

Thanks to Mike "Pomax" Kamermans, Cesar Andreu, Dan Palmer, Viktor Jančík, Tom Byrer, Christian Hettlage, David A. Lee, Alexandar Castaneda, Marcel Olszewski, Steve Schwartz, Chris Sanders, Charles Ju, Aditya Bhardwaj, Rasheed Bustamam, José Menor, Ben Gale, Jake Goulding, Andrew Ferk, gabo, Giang Nguyen, @Coaxial, @khronic, Henrik Raitasola, Gavin Orland, David Riccitelli, Stephen Wright, Majky Bašista, Gunnari Auvinen, Jón Levy, Alexander Zaytsev, Richard Muller, Ava Mallory (Fiverr), Sun Zheng’an, Nancy (Fiverr), Aluan Haddad, Steve Mao, Craig McKenna, Tobias Koppers, Stefan Frede, Vladimir Grenaderov, Scott Thompson, Rafael De Leon, Gil Forcada Codinachs, Jason Aller, @pikeshawn, Stephan Klinger, Daniel Carral, Nick Yianilos, Stephen Bolton, Felipe Reis, Rodolfo Rodriguez, Vicky Koblinski, Pyotr Ermishkin, Ken Gregory, Dmitry Kaminski, John Darryl Pelingo, Brian Cui, @st-sloth, Nathan Klatt, Muhamadamin Ibragimov, Kema Akpala, Roberto Fuentes, Eric Johnson, Luca Poldelmengo, Giovanni Iembo, Dmitry Anderson , and many others who have contributed direct feedback for this book!

