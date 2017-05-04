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



