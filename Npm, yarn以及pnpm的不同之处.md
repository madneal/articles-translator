# npm, yarn以及pnpm的不同之处

> 原文：[Overview of differences between npm, yarn and pnpm](https://hackernoon.com/understanding-differences-between-npm-yarn-and-pnpm-31bb6b0c87b3)
>
> 译者：[neal1991](https://github.com/neal1991)
>
> welcome to star my [articles-translator](https://github.com/neal1991/articles-translator/), providing you advanced articles translation. Any suggestion, please issue or contact [me](mailto:bing@stu.ecnu.edu.cn)
>
> LICENSE: [MIT](https://opensource.org/licenses/MIT)

我并不是一个包管理器的专家。相反，直到最近我才意识到`npm`使用的是本地的缓存。在不知道这个时候，我写了这篇名为[我的天--NPM克隆终于有意义了](https://medium.com/@akras14/omg-npm-clone-that-finally-makes-sense-3478588879)并且给出了一些我的不正确的猜想。这些反馈迫使我回头并且重新审视近来这些包管理器的区别。

我在过去的5年时间一直都是使用`npm`。我也折腾了下`yarn`当它第一次出来的时候，而且我是通过一个礼拜前的文章“为什么我们应该使用pnpm”来学习`pnpm`的。

过去的一周，我一直花时间阅读`npm`，`yarn`以及`pnpm`相关的东西，想总结一下然后分享我的发现。我的目标读者是长期的`npm`用户，并且不愿意花费太多的时间了解有多少种`npm`的替代品，比如我自己。我只会关注这三种最常提及的（对于我）并且不会包括：`ied`，`npm-install`以及`npmd`，因为对于它们我是一无所知。

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

可以通过`npm config set save-exact true`命令来关闭在包的版本前添加`^`的默认行为，但这个只会锁住高层次的依赖。因为每一个引入的包都有它们自己的`package.json`文件，在这里面的依赖可能包含了`^`，没有办法通过`package.json`来保证嵌套的内容。

为了解决这个问题，npm提供了一个[shrinkwrap](https://docs.npmjs.com/cli/shrinkwrap)命令。这个命令能够生成一个`npm-shrinkwrap.json`文件，对于所有的包以及嵌套的依赖规定了明确的版本。

这也就是说，即使是通过`npm-shrinkwrap.json`这个文件，npm也只是锁住了包的版本而不是包的内容。即使[npm现在阻止用户多次发布相同版本的包](http://blog.npmjs.org/post/77758351673/no-more-npm-publish-f)，`npm`管理依然有权利强制更新某些包。

下面是 [shrinkwrap](https://docs.npmjs.com/cli/shrinkwrap)文档页面的引用：

>如果你希望锁定包内包含的指定字节，例如你有百分之百的信心能够重新发布或者构建，那么你应该将你的依赖检查到源代码控制中，或者追求其它的某种能够验证内容而不是版本的机制。

`npm version 2`过去式对于每个包内所有引入的依赖全部都安装。如果已有一个项目，这个项目引入项目A，项目A引入项目B，项目B引入项目C，那么这个所有依赖的结构树看起来会是下面这样：

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

事实证明，我是错的，并且`npm`**确实**是具有本地缓存的，在其中保存了所有下载包的压缩文件。可以通过`npm cache ls`命令来查看本地缓存的内容。通过本地缓存可以加快安装速度。

总而言之，`npm`是一个成熟的，稳定的并且乐于使用的包管理器。

## yarn

[Yarn](https://github.com/yarnpkg/yarn) 是在2016年10月份发布的并且在Github上迅速获取了24K+star。作为对比，[npm](https://github.com/npm/npm) 仅仅只有12K+ star。这个项目具有高资质的开发者比如Sebastian McKenzie ([Babel.js](https://babeljs.io/)) 以及 Yehuda Katz ([Ember.js](https://www.emberjs.com/), [Rust](https://www.rust-lang.org/en-US/), [Bundler](http://bundler.io/) 等等)。

从我目前收集的来看，yarn的最初的主要目的是针对`npm`由于之前章节提及的semver相关行为导致的安装的不确定性。然而可预测的依赖树（如果需要的话）能够通过`npm shrinkwarp`来完成，这不是默认的行为并且依赖于开发者了解这一选项并且来进行相应的操作。

Yarn采取了一个不同的方法。每次`yarn`安装都会生成一个和`npm-shrinkwrap.json`类似的`yarn.lock`文件，但是它是默认产生的。除了常规信息，`yarn.lock`文件还包含了安装内容的检查从而确保使用相同版本的包。

因为yarn是一个才重写的`npm`客户端，开发者能够适宜地并行所有需要的操作并且增加一些改进，这同时也显著提升了整体地安装时间。我认为速度地加快是`yarn`流行的主要原因。

像`npm`一样，`yarn`也使用了本地缓存。但是不像`npm`，`yarn`在安装已经缓存的依赖的时候并不需要网络连接，提供了一种`offline`模式。这个特性[在npm上自从2012年就收到了请求](https://github.com/npm/npm/issues/2568)，但是一直没有得到解决。

Yarn提供一些其它的好处。比如，它允许聚合项目中使用的所有的licence，并且很容易看到。

有意思的一点事，`yarn`文档对于`npm`态度的转变自从其变得流行之后。

最初的[yarn发布的时候](https://code.facebook.com/posts/1840075619545360)说的安装`yarn`的步骤是：

> 最容易开始运行的方式是：
>
> ```
> npm install -g yarn
> yarn
> ```

现在yarn[关于安装yarn的方式是](https://yarnpkg.com/en/docs/install#alternatives-tab)：

> 注意：不建议使用npm来进行安装。npm是非确定性的，包是没有签名的，npm仅仅是做了基本的SHA1 哈希并没有做任何整体性检查，这对于安装系统级别的应用是有风险的。
>
> 鉴于以上原因，强烈建议你通过适合你操作系统的安装方法来安装yarn。

以这种速度，即使`yarn`宣布它们自己的registry从而让开发者缓慢淘汰`npm`我都不觉得惊讶。

同样也是由于`yarn`，`npm`终于一是到它们需要密切关注那些强烈请求的issue。[NPM最初对于yarn发布的回应](http://blog.npmjs.org/post/151660845210/hello-yarn)在我看来是“它是可爱的”。现在，当我重新看那个被强烈要求的我之前提到过的“离线”特性已经有在被积极地解决，[在我们讨论这一点的时候](https://github.com/npm/npm/commit/8cf4c662903ce84a24138acf3cc37083aecac08b)。

## pnpm

正如我之前所提及的那样，我只是才知道[pnpm](https://github.com/pnpm/pnpm) 不久通过阅读Zoltan Kochan的[“为什么我们应该使用pnpm？”](https://www.kochan.io/nodejs/why-should-we-use-pnpm.html)，他就是npm的作者。

我不想设计到过多的细节（因为这篇文章已经很长了），但是你可以从我[最初的博文](https://medium.com/@akras14/omg-npm-clone-that-finally-makes-sense-3478588879)中可以了解一些以及[推特上的讨论](https://twitter.com/akras14/status/855474658832900096)。

**但是**

我想指出的是`pnpm`比[npm以及yarn都要快](https://github.com/pnpm/node-package-manager-benchmark)。

为什么它这么快的原因？因为它采用了巧妙的方式，利用硬链接和符号链接，以避免复制所有本地缓存的源文件，这是打败yarn在性能上最主要的一方面。

使用链接并不容易，需要考虑一系列的问题。

正如Sebastian[在推特上](https://twitter.com/sebmck/status/855553631680069637)所指出的，他最初是想在`yarn`里面使用符号链接，但最后因为[很多原因](https://github.com/yarnpkg/yarn/issues/1761#issuecomment-259706202)来对抗yarn。

同时，这个项目在Github上也具有2K+star，`pnpm`能够让链接为很多人工作。

另外，自从[2017年3月](https://www.kochan.io/nodejs/why-should-we-use-pnpm.html)它提供了所有`yarn`提供的优点，包括离线模式以及确定性安装。

## 结论

我认为`yarn`和`pnpm`的开发者都做了很好的工作。我个人的偏好是确定性的安装，因为我喜欢自己掌控并且我不喜欢惊喜。

无论最后竞争的结果是什么（这也提醒了我[io.js](https://iojs.org/en/) fork），我感谢`yarn`给`npm`带来的这些麻烦因此尘埃落定之前还有很多选择余地。

我也认为`yarn`可能很早之前就考虑过硬链接以及软链接。我想知道的是`yarn`团队针对这个想法会做些什么，取决于`pnpm`造成的杀伤以及用户对于安装速度重视的程度。

我确实认为总的来说`yarn`是一个安全的选择，但是`pnpm`在某些案例上可能是一个更好的选择。比如，对于一个需要运行很多集成测试并且希望尽可能提高安装依赖速度的中小型团队。

最后值得说得一点是，我认为`npm`依然提供了对于大多数用户案例非常有用的解决方案。大多数开发者可以继续只使用`npm`客户端。

无论是任何情况，我都感谢所有的努力保持生态健康的竞争者。当公司竞争的时候，获利的是用户。

来自于一下[@ReBeccaOrg](https://twitter.com/ReBeccaOrg)推特的更新。

> - FYI, 展平不是“遍历整棵树”的来源。遍历整棵树是关于自愈。
> - 和npm@2相比，它减缓了速度。
> - 如果你通过`rm -fr`来删除嵌套的依赖并且通过`npm install`来安装，你会注意到这可能帮你解决问题。
> - 现在事实证明，npm@1-@npm@4缓存是缓慢的，非常慢。慢的超乎想象。经常往往是比在一个快速网络中下载快一点点。因此随着npm@5的重写（自从npm1.5就开始计划）它突然变得快多了。
> - yarn通过沙箱可以伤你免于http://registry.npmjs.org 敌对的限制，但是并不抱进一步的保证。