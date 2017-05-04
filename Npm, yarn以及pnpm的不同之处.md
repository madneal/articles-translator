# npm, yarn以及pnpm的不同之处

> 原文：[Overview of differences between npm, yarn and pnpm](https://hackernoon.com/understanding-differences-between-npm-yarn-and-pnpm-31bb6b0c87b3)
>
> 译者：[neal1991](https://github.com/neal1991)
>
> welcome to star my [articles-translator ](https://github.com/neal1991), providing you advanced articles translation. Any suggestion, please issue or contact [me](mailto:bing@stu.ecnu.edu.cn)
>
> LICENSE: [MIT](https://opensource.org/licenses/MIT)

我并不是一个包管理器的专家。相反，直到最近我才意识到`npm`使用的是本地的缓存。在不知道这个时候，我写了这篇名为[我的天--NPM克隆终于有意义了](https://medium.com/@akras14/omg-npm-clone-that-finally-makes-sense-3478588879)并且给出了一些我的不正确的猜想。这些反馈迫使我回头并且重新绅士近来这些包管理器的区别。

我在过去的5年时间一直都是使用`npm`。我也折腾了下`yarn`当它第一次出来的时候，而且我是通过一个礼拜钱的文章“为什么我们应该使用pnpm”来学习`pnpm`的。

过去的一周，我一直花时间阅读`npm`，`yarn`以及`pnpm`相关的东西，想总结一下然后分享我的发现。我的目标读者是长期的`npm`用户，并且不愿意花费太多的时间了解有多少种`npm`的替代品，比如我自己。我只会关注这三种最长提及的（对于我）并且不会包括：`ied`，`npm-install`以及`npmd`，因为对于它们我是一无所知。

还有重要的一点需要指出，直到写这篇文章的时候，还没有一个具有竞争力的库的目标是替换NPM的**registry**（就是存储包的地方），它们的目的都是替换`npm`命令行客户端，提供另外一个可用的用户界面以及行为，并且其功能也是类似的。

## NPM

[npm](https://github.com/npm/npm)自从Node.js出现的那一天就存在了并且也是造成Node.js这个项目如此成功的原因之一。`npm`团队在让`npm`保持向后兼容以及在多种环境下持续工作都做了很多的工作。

`npm`的设计理念是根据 [Semantic Versioning (semver)](http://semver.org/)，这是一个相当直白的方法可以从他们官网的引用可以看出来。

给定一个版本号MAJOR.MINOR.PATCH，增量修改表示：

- MAJOR版本修改意味着你做出了不兼容的API变化。
- MINOR版本意味着你以向后兼容的方式增加了功能。
- PATCH版本意味着你做出了向后兼容的bug修复。

`npm`使用一个叫做`package.json`的文件，用户可以存储项目的所有依赖通过运行`npm install --save`。

例如，运行`npm install --save loadsh`将会将这一条加入到`package.json`之中。

```
"dependencies": {
  "loadsh": "^4.17.4"
}
```

注意`^`这个loadsh版本号之前的符号。这个符号告诉`npm`安装任何与MAJOR版本相同的包。因此如果一年后我运行`npm install`，`npm`会安装MAJOR版本号为4的最新版本的loadsh。例如，它可以是`loadsh@4.25.5`（`@`是一个npm惯例用来指定包名的版本）。你可以在这看到所有支持的符号： [https://docs.npmjs.com/misc/semver](https://docs.npmjs.com/misc/semver)。

这样做的原因是，因为MINOR版本的变动（理论上）应该只是包含向后兼容的滨化。因此安装最新版本的包可能会引入重要的bug或者安全问题，因为最初安装的版本是`4.17.4`。

另一方面，它可能会导致多个开发者的机器上安装了不同版本的包，即使他们是共享同一个`package.json`文件，这样会潜在地导致难以调试以及“在我的机器是好的啊”的情形。

大多数`npm`包都非常依赖其它的`npm`包。这会导致循环依赖以及增加了版本不匹配的可能。

可以通过`npm config set save-exact true`命令来关闭在包的版本前添加`^`的默认行为，但这个只会锁住高层次的依赖。因为i每一个引入的包都有它们自己的`package.json`文件，在这里面的依赖可能包含了`^`，没有办法通过`package.json`来保证嵌套的内容。

为了解决这个问题，npm提供了一个[shrinkwrap](https://docs.npmjs.com/cli/shrinkwrap)命令。这个命令能够生成一个`npm-shrinkwrap.json`文件，对于所有的包以及嵌套的依赖规定了明确的版本。

这也就是说，即使是通过`npm-shrinkwrap.json`这个文件，npm也只是锁住了包的版本而不是包的内容。即使[npm现在阻止用户多次发布相同版本的包](http://blog.npmjs.org/post/77758351673/no-more-npm-publish-f)，`npm`管理依然有权利强制更新某些包。

下面是 [shrinkwrap](https://docs.npmjs.com/cli/shrinkwrap)文档页面的引用：

>如果你希望锁定包内包含的指定字节，例如你有百分之百的信心能够重新发布或者构建，那么你应该将你的依赖检查到媛代码控制中，或者追求其它的某种能够验证内容而不是版本的机制。

`npm version 2`过去式对于每个包内所有引入的依赖全部都安装。如果已有一个项目，这个项目引入项目A，项目A引入项目B，项目B引入项目从，那么这个所有依赖的结构树看起来会是下面这样：

```
node_modules
- package-A
-- node_modules
--- package-B
----- node_modules
------ package-C
-------- some-really-really-really-long-file-name-in-package-c.js
```

这个结构可能会变得相当长。这可能仅仅是基于Unix系统上面的一个烦恼，在Windows上已经有很多破解程序能够解决文件路径超过260个字符的问题。

`npm version 3`通过展平依赖树来解决这个问题，因此这3个项目的结构看起来会是这个样子的：

```
node_modules
- package-A
- package-B
- package-C
-- some-file-name-in-package-c.js
```

这个变化的结果就是改变了一写长文件的文件路径从 `./node_modules/package-A/node_modules/package-B/node-modules/some-file-name-in-package-c.js` 变化为 `./node_modules/some-file-name-in-package-c.js`。

你可以从[这](https://docs.npmjs.com/how-npm-works/npm3)了解更多关于NPM 3依赖的解决方案。

这个方法的一个缺点是`npm`现在必须要遍历所有的项目依赖从而决定如何展平`node_modules`文件夹。`npm`被强制为所有使用过的模块建立依赖树，这样做的代价会很大。这也是[导致`npm install`安装速度变慢的原因之一](https://github.com/npm/npm/issues/8826)。（请看文末的更新）。

因为我没有仔细关注过`npm`的变化，我猜想NPM速度变慢的原因是我每次运行`npm install`的时候都需要从网上下载所有东西。

事实证明，我是错的，并且`npm`**确实**是具有本地缓存的，在其中保存了所有下载包的压缩文件。可以通过`npm cache ls`命令来查村本地缓存的内容。通过本地缓存可以加快安装速度。

总而言之，`npm`是一个成熟的，稳定的并且乐于使用的包管理器。

## yarn

[Yarn](https://github.com/yarnpkg/yarn) 是在2016年10月份发布的并且在Github上迅速获取了24K+star。作为对比，[npm](https://github.com/npm/npm) 仅仅只有12K+ star。这个项目具有高资质的开发者比如Sebastian McKenzie ([Babel.js](https://babeljs.io/)) 以及 Yehuda Katz ([Ember.js](https://www.emberjs.com/), [Rust](https://www.rust-lang.org/en-US/), [Bundler](http://bundler.io/) 等等)。

从我目前收集的来看，yarn的最初目的是