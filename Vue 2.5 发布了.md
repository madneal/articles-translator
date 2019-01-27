## Vue 2.5 发布了

> 原文：[Vue 2.5 released](https://medium.com/the-vue-point/vue-2-5-released-14bd65bf030b)
>
> 译者：[neal1991](https://github.com/neal1991)
>
> welcome to star my [articles-translator](https://github.com/neal1991/articles-translator/), providing you advanced articles translation. Any suggestion, please issue or contact [me](mailto:bing@stu.ecnu.edu.cn)
>
> LICENSE: [MIT](https://opensource.org/licenses/MIT)

我们很高兴宣布 Vue 2.5 Level E 的发布！本次发布包括多个功能提升并且我们推荐你查看[发布说明](https://github.com/vuejs/vue/releases/tag/v2.5.0)来获取完整详细信息。 在这篇文章中，我们将重点介绍一些更重要的的变化：更好的 TypeScript 集成，更好的错误处理，更好地支持单文件组件中的函数式组件以及与环境无关的服务端渲染。

## 更好的 TypeScript 集成

![](https://cdn-images-1.medium.com/max/3200/1*vB-z-t961mJnd4a6re02Iw.png)

得益于 TypeScript 团队的帮助，2.5 提供了大大改进的类型声明，可以与 Vue 的开箱即用的 API 一起使用，而不需要组件类装饰器。 新的类型声明还可以让 [Vetur](https://marketplace.visualstudio.com/items?itemName=octref.vetur) 等编辑器扩展功能更强大，为纯 JavaScript 用户提供更好的Intellisense 支持。 更多详细信息，请查看[我们之前关于更改的信息](https://medium.com/the-vue-point/upcoming-typescript-changes-in-vue-2-5-e9bd7e2ecf08)。

*感谢来自 TypeScript 团队的 Daniel Rosenwasser 发起的 PR，以及核心团队成员 Herrington Darkholme和 Katashin 的改进和审查。*

>  注意：TypeScript 用户还应将以下包更新为最新版本从而兼容类型声明：`vue-router`，`vuex`，`vuex-router-sync` 和`vue-class-component`。

## 更好地错误处理

![](https://cdn-images-1.medium.com/max/2000/1*ZHamhzmnoQcQTxCJE3cmvA.jpeg)

在2.4及更早版本中，我们通常使用全局 `config.errorHandleroption` 来处理应用程序中的意外错误。 我们还有`renderError` 组件选项来处理渲染函数中的错误。 但是，我们缺少处理应用程序特定部分内的泛型错误的机制。

在2.5中，我们引入了新的 `errorCaptured` 钩子。 具有此钩子的组件捕获其子组件树（不包括其自身）中的所有错误（不包括在异步回调中调用的那些）。 如果你熟悉React，这与 React 16 中引入的[错误边界](https://reactjs.org/blog/2017/07/26/error-handling-in-react-16.html#introducing-error-boundaries)的概念相似。钩子接收与全局 `errorHandler` 相同的参数，你可以利用这个钩子来[优雅地处理和显示错误](https://gist.github.com/yyx990803/9bdff05e5468a60ced06c29c39114c6b#error-handling-with-errorcaptured-hook)。

## 更好地支持 `SFC` 中的函数式组件

![](https://cdn-images-1.medium.com/max/2828/1*jg9qGPkPadGBEa-KUPrMpA.png)

使用 `vue-loader> = 13.3.0` 和 `Vue 2.5`，在 `* .vue` 文件中定义为单个文件组件的函数式组件现在可以得到[正确的模板编译，Scoped CSS和热重新加载支持](https://vue-loader.vuejs.org/en/features/functional.html)。 这使得将叶子组件转换为函数式的更为容易，从而进行性能优化。

*感谢核心团队成员[Blake Newman](https://github.com/blake-newman) 对于这些功能做出的贡献。

## 与环境无关的服务端渲染

`vue-server-renderer` 的默认构建假定一个 Node.js 环境，这使得它在有的 JavaScript 运行时（如 [php-v8js](https://github.com/phpv8/v8js) 或Nashorn）中不可用。 在 2.5 中，我们已经发布了[一个与环境无关的 `vue-server-renderer` 版本](https://github.com/vuejs/vue/blob/dev/packages/vue-server-renderer/basic.j)，可以在浏览器或纯 JavaScript 引擎中使用。 这可以打开有趣的策略，例如[直接在 PHP 进程中使用 Vue 服务端渲染](https://gist.github.com/yyx990803/9bdff05e5468a60ced06c29c39114c6b#environment-agnostic-ssr)。

同样，我们建议你查看完整的[发布说明](https://github.com/vuejs/vue/releases/tag/v2.5.0)从而了解其他 API 的改进，包括 `v-on`，`v-model`，`scoped slot`，`provide/inject` 等。 你可能也对我们的[公共蓝图](https://github.com/vuejs/roadmap)感兴趣，详细说明了团队的工作。 干杯!
