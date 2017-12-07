## 🚀 宣布 Parcel：一个快速，零配置的 Web 应用打包工具 📦

https://hackernoon.com/announcing-parcel-a-blazing-fast-zero-configuration-web-application-bundler-feac43aac0f1

Today I’m super excited to announce [Parcel](https://parceljs.org), a blazing fast, zero configuration web application bundler that I’ve been working on for the past few months. Check it out [on Github](https://github.com/parcel-bundler/parcel)!

今天，我非常高兴地宣布 Parcel，一个快速，零配置的 Web 应用程序打包工具，我对于该工具的工作已经持续了几个月。 去 [Github](https://github.com/parcel-bundler/parcel)上看看吧！

![](https://cdn-images-1.medium.com/max/5994/1*Gjhk6qvPM5zAy1iPPS1ttg.png)

I started working on Parcel in order to solve two main problems that I have found with existing module bundlers like Browserify and Webpack: **performance**, and **configuration experience**.

为了解决我在 Browserify 和 Webpack 等现有模块打包工具中遇到的两个主要问题：性能和配置经验，我开始研究 Parcel。

### 特性

* 🚀 **Blazing fast** bundle times — multicore compilation, and a filesystem cache for fast rebuilds even after a restart.**非常快**的打包时间 - 多核编译，以及文件系统缓存，这样即使在重新启动后也能快速重建。
* 📦 [Out of the box support](https://parceljs.org/assets.html) for JS, CSS, HTML, images, file assets, and more — **no plugins to install**.对于 JS, CSS, HTML, 图片以及文件资源以及其它支持[开箱即用]((https://parceljs.org/assets.html)，**不需要安装插件**。
* 🐠 **Automatically [transforms](https://parceljs.org/transforms.html) modules** using Babel, PostCSS, and PostHTML when needed — even node_modules. 在需要时使用 Babel，PostCSS 和 PostHTML **自动[转换](https://parceljs.org/transforms.html)模块** - 甚至是node_modules。
* ✂️ **Zero configuration [code splitting](https://parceljs.org/code_splitting.html)** using dynamic import() statements.**零配置**[代码分割](https://parceljs.org/code_splitting.html)使用动态import() 语句。
* 🔥 Built in support for [**hot module replacement](https://parceljs.org/hmr.html)**内置支持[热替换](https://parceljs.org/hmr.htm)
* 🚨 Friendly error logging experience — syntax highlighted code frames help pinpoint the problem.友好的错误日志体验 - 语法高亮显示的代码帧有助于查明问题。

### 性能

The first reason I was motivated to build a new bundler was performance. I’ve worked on some pretty large apps with thousands of modules, and was always disappointed with the speed of existing bundlers. Large apps can take minutes to build, which is especially frustrating during development.

Many bundlers have focused on fast rebuild performance with delta builds, and this is great. However, initial build performance is also very important for both development and production/CI builds.

Parcel solves this problem by using worker processes to **compile your code in parallel**, utilizing modern multicore processors. This results in a huge speedup for initial builds. It also has a **file system cache**, which saves the compiled results per file for even faster subsequent startups.

我被激发建立一个新的打包工具的第一个原因是性能。 我已经在数千个模块上做了一些相当大的应用程序，并且总是对现有打包工具的速度感到失望。 大型应用程序可能需要几分钟才能完成，这在开发过程中尤其令人沮丧

许多打包工具专注于快速增量重新构建性能，这是很好的。 但是，最初的构建性能对于开发和生产/ CI 构建也非常重要。

Parcel 通过使用工作进程**并行编译代码**，利用现代多核处理器解决了这个问题。 这导致了初始构建的巨大加速。 它还有一个文件系统缓存，可以保存每个文件的编译结果，以便更快的后续启动。

![Based on a reasonably sized app, containing 1726 modules, 6.5M uncompressed. Built on a 2016 MacBook Pro with 4 physical CPUs.](https://cdn-images-1.medium.com/max/2000/1*t8afejIByMpoZKSs-URTIQ.png)

### 零配置体验

The second reason I built Parcel was to help with the pain of managing configuration. Most other bundlers are built around config files with lots of plugins, and it is not uncommon to see applications with upwards of 500 lines of configuration just to get things working.

This configuration is not just tedious and time consuming, but is also hard to get right and must be duplicated for each application. Oftentimes, this can lead to sub-optimized apps shipping to production.

Parcel is designed to need **zero configuration**: just point it at the entry point of your application, and it does the right thing. Parcel has out of the box support for JS, CSS, HTML, images, file assets, and more — no plugins needed.

Parcel’s zero configuration experience extends beyond file formats as well. **Transforms** like Babel, PostCSS, and PostHTML are also applied automatically when Parcel detects a .babelrc, .postcssrc, etc. This even works for third party code in node_modules for that module only, so application authors don’t need to know how to build each module they import, and the build doesn’t slow down running Babel over every file unnecessarily.

Finally, advanced bundling features like code splitting, and hot module reloading are supported out of the box as well. In production mode, Parcel automatically enables minification, with other optimizations like tree-shaking to come in the future.

我建立Parcel的第二个原因是帮助解决管理配置的痛苦。大多数其它打包工具都是围绕着配置文件以及大量的插件建立起来的，为了使事情顺利进行，看到 500 行以上的应用程序配置并不罕见。

这种配置不仅繁琐耗时，而且很难正确使用，并且必须针对每个应用程序进行复制。通常情况下，这可能导致次优化的应用程序转到到生产。

Parcel 被设计为**零配置**：只需将它指向你的应用程序的入口点，它就能正确工作。 Parcel 支持 JS，CSS，HTML，图片，文件资源等等 - 不需要任何插件。

Parcel 的零配置体验也涉及到文件格式。当 Parcel 检测到 .babelrc，.postcssrc 等时，也会自动应用像 Babel，PostCSS 和 PostHTML 这样的**转换**。这甚至适用于仅用于该模块的 node_modules 中的第三方代码，因此应用程序作者不需要知道如何构建他们导入的每个模块，并且构建不会减慢不必要地在每个文件上运行 Babel。

最后，还支持代码分割和热模块重新加载等高级打包功能。在生产模式下，Parcel 自动启用缩小，未来还会进行其他优化，如 tree-shaking。

### 未来架构

One advantage of starting a fresh project is that I was able to design a more modern architecture for Parcel that is more extendible, and more flexible, all while requiring zero configuration from users, and supporting advanced features like **code splitting** and **hot module reloading**.

Most bundlers are primarily focused on JavaScript, with support for other formats somewhat tacked on. For example, other file types are often inlined into JavaScript files by default, with additional plugins and hacks to extract them into separate files again.

In Parcel, any type of file can be a first-class citizen. It is easy to add new Asset types, which represent input files, and Packagers which combine assets of similar types together into output files.

For example, there is a CSS Asset type, which analyzes and produces CSS code, and a CSS Packager which combines CSS assets together into a final bundle. Similar types exist for JS, HTML, and more. In this way, Parcel is completely **file-type agnostic**.

You can read more about [how Parcel works](https://parceljs.org/how_it_works.html) on the website.

启动一个新项目的一个好处是，我能够为 Parcel 设计一个更加现代化的架构，这个架构更加可扩展，更灵活，同时无需用户配置，并支持**代码拆分**和**热加载**等高级功能。

大多数打包工具主要关注 JavaScript，并支持其他格式。例如，默认情况下，其他文件类型通常会内嵌到JavaScript 文件中，并使用额外的插件和 hack 将其再次提取到单独的文件中。

在 Parcel 中，任何类型的文件都可以成为一等公民。添加代表输入文件的新资源类型和将类似类型的资源组合到输出文件中的打包工具很容易。

例如，分析和生成 CSS 代码的 CSS 资源类型和将 CSS 资源组合成最终打包的 CSS Packager。 JS，HTML 等存在类似的类型。这样，Parcel 完全是**文件类型无关**的。

你可以阅读更多关于[Parcel 如何在网站上工作](https://parceljs.org/how_it_works.html)的信息。

### 试试把

Parcel is just getting started, but many applications already work out of the box with zero configuration! So try it out — delete your webpack/browserify configuration, uninstall those plugins, and try Parcel. 😎

Parcel  刚刚开始，但许多应用程序已经开箱即用并且零配置！ 所以试试看吧 - 删除你的webpack/browserify配置，卸载这些插件，然后尝试Parcel。😎

Let me know how it goes! You can find me [@devongovett](https://twitter.com/devongovett) on Twitter.

欢迎向我反馈！ 你可以在Twitter上找到我[@devongovett](https://twitter.com/devongovett)。

* [网站和文档](https://parceljs.org)

* [Github](https://github.com/parcel-bundler/parcel)