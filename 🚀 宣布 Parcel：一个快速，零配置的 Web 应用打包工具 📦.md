## 🚀 宣布 Parcel：一个快速，零配置的 Web 应用打包工具 📦

>原文：[🚀 Announcing Parcel: A blazing fast, zero configuration web application bundler 📦](https://hackernoon.com/announcing-parcel-a-blazing-fast-zero-configuration-web-application-bundler-feac43aac0f1)
>
>译者：[neal1991](https://github.com/neal1991)
>
>welcome to star my [articles-translator](https://github.com/neal1991/articles-translator/), providing you advanced articles translation. Any suggestion, please issue or contact [me](mailto:bing@stu.ecnu.edu.cn)
>
>LICENSE: [MIT](https://opensource.org/licenses/MIT)

今天，我非常高兴地宣布 Parcel，一个快速，零配置的 Web 应用程序打包工具，我对于该工具的工作已经持续了几个月。 去 [Github](https://github.com/parcel-bundler/parcel)上看看吧！

![](https://cdn-images-1.medium.com/max/5994/1*Gjhk6qvPM5zAy1iPPS1ttg.png)

为了解决我在 Browserify 和 Webpack 等现有模块打包工具中遇到的两个主要问题：**性能**和**配置经验**，我开始研究 Parcel。

### 特性

* 🚀 **非常快**的打包时间 - 多核编译，以及文件系统缓存，这样即使在重新启动后也能快速重建。
* 📦对于 JS, CSS, HTML, 图片以及文件资源以及其它支持[开箱即用](https://parceljs.org/assets.html)，**不需要安装插件**。
* 🐠在需要时使用 Babel，PostCSS 和 PostHTML **自动**[**转换**](https://parceljs.org/transforms.html)**模块** - 甚至是node_modules。
* ✂️ **零配置**[代码分割](https://parceljs.org/code_splitting.html)使用动态import() 语句。
* 🔥内置支持[热加载](https://parceljs.org/hmr.htm)
* 🚨 友好的错误日志体验 - 语法高亮显示的代码帧有助于查明问题。

### 性能

我被激发建立一个新的打包工具的第一个原因是性能。 我已经在数千个模块上做了一些相当大的应用程序，并且总是对现有打包工具的速度感到失望。 大型应用程序可能需要几分钟才能完成，这在开发过程中尤其令人沮丧

许多打包工具专注于快速增量重新构建性能，这是很好的。 但是，最初的构建性能对于开发和生产/ CI 构建也非常重要。

Parcel 通过使用工作进程**并行编译代码**，利用现代多核处理器解决了这个问题。 这导致了初始构建的巨大加速。 它还有一个文件系统缓存，可以保存每个文件的编译结果，以便更快的后续启动。

![Based on a reasonably sized app, containing 1726 modules, 6.5M uncompressed. Built on a 2016 MacBook Pro with 4 physical CPUs.](https://cdn-images-1.medium.com/max/2000/1*t8afejIByMpoZKSs-URTIQ.png)

### 零配置体验

我建立Parcel的第二个原因是帮助解决管理配置的痛苦。大多数其它打包工具都是围绕着配置文件以及大量的插件建立起来的，为了使事情顺利进行，看到 500 行以上的应用程序配置并不罕见。

这种配置不仅繁琐耗时，而且很难正确使用，并且必须针对每个应用程序进行复制。通常情况下，这可能导致次优化的应用程序转到到生产。

Parcel 被设计为**零配置**：只需将它指向你的应用程序的入口点，它就能正确工作。 Parcel 支持 JS，CSS，HTML，图片，文件资源等等 - 不需要任何插件。

Parcel 的零配置体验也涉及到文件格式。当 Parcel 检测到 .babelrc，.postcssrc 等时，也会自动应用像 Babel，PostCSS 和 PostHTML 这样的**转换**。这甚至适用于仅用于该模块的 node_modules 中的第三方代码，因此应用程序作者不需要知道如何构建他们导入的每个模块，并且构建不会减慢不必要地在每个文件上运行 Babel。

最后，还支持代码分割和热模块重新加载等高级打包功能。在生产模式下，Parcel 自动启用缩小，未来还会进行其他优化，如 tree-shaking。

### 未来架构

启动一个新项目的一个好处是，我能够为 Parcel 设计一个更加现代化的架构，这个架构更加可扩展，更灵活，同时无需用户配置，并支持**代码拆分**和**热加载**等高级功能。

大多数打包工具主要关注 JavaScript，并支持其他格式。例如，默认情况下，其他文件类型通常会内嵌到JavaScript 文件中，并使用额外的插件和 hack 将其再次提取到单独的文件中。

在 Parcel 中，任何类型的文件都可以成为一等公民。添加代表输入文件的新资源类型和将类似类型的资源组合到输出文件中的打包工具很容易。

例如，分析和生成 CSS 代码的 CSS 资源类型和将 CSS 资源组合成最终打包的 CSS Packager。 JS，HTML 等存在类似的类型。这样，Parcel 完全是**文件类型无关**的。

你可以阅读更多关于[Parcel 如何在网站上工作](https://parceljs.org/how_it_works.html)的信息。

### 试试把

Parcel  刚刚开始，但许多应用程序已经开箱即用并且零配置！ 所以试试看吧 - 删除你的webpack/browserify配置，卸载这些插件，然后尝试Parcel。😎

欢迎向我反馈！ 你可以在Twitter上找到我[@devongovett](https://twitter.com/devongovett)。

* [网站和文档](https://parceljs.org)

* [Github](https://github.com/parcel-bundler/parcel)
