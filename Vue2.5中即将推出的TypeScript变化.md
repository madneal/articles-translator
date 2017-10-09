## Vue 2.5中即将推出的TypeScript变化

## 输入提升

自Vue 2.0发布以来，我们一直在收到更好的 TypeScript 集成请求。 自从发布以来，我们已经为大多数核心库（`vue`, `vue-router`, `vuex`)包含了官方的 TypeScript 类型声明。 然而，当使用开箱即用的 Vue API 时，目前的集成有些缺乏。 例如，TypeScript 不能轻易地推断 Vue 使用的基于对象的默认 API 中 `this` 的类型。 为了使我们的 Vue 代码可以很好地使用 TypeScript，我们必须使用 `vue-class-component` 装饰器，这样我们可以使用基于类的语法来创建 Vue 组件。

对于喜欢基于类的API的用户来说，这可能已经足够好了，但对于仅仅为了类型推断的用户不得不使用不同的API是不幸的。 这也使得将现有 Vue 代码库迁移到 TypeScript 更具挑战性。

今年早些时候，TypeScript 引入了许多[新功能](https://github.com/Microsoft/TypeScript/pull/14141)，这样就可以改进Vue的类型声明从而使得 TypeScript 可以更好地理解基于对象字面量的 API。 来自 TypeScript 团队的 Daniel Rosenwasser 开始了一个雄心勃勃的PR（现在由核心团队成员 HerringtonDarkholme 在[这](https://github.com/vuejs/vue/pull/6391)维护），一旦合并，将提供：

* 使用默认的 Vue API 时，对于 `this` 可以使用适当的类型推断。 它也可以在单文件组件中工作！
* 基于组件的 `props` 选项，对于 `this` 中的 props 输入推断。
* 最重要的是，**这些改进也使得纯 JavaScript 用户受益匪浅！** 如果你使用 VSCode 与超级棒的的 [Vetur](https://github.com/vuejs/vetur) 扩展，你将获得大大改进的自动完成建议，甚至在Vue组件中使用纯 JavaScript 时也能获得输入提示！ 这是因为[vue-language-server](https://www.npmjs.com/package/vue-language-server)是负责分析 Vue 组件的内部包，可以利用 TypeScript 编译器来提取有关你的代码的更多信息。 此外，任何支持语言服务器协议的编辑器都可以利用 [`vue-language-server`](https://github.com/vuejs/vetur/tree/master/server)来提供类似的功能。

![VSCode + Vetur + New Type Declarations in Action](https://cdn-images-1.medium.com/max/3640/1*ftKUpzYGIzn1eS87JcBS8Q.gif)

对于那些好奇的人，你可以通过克隆这个 [playground 项目](https://github.com/octref/veturpack/tree/new-types)（确保获取 `new-types` 的分支）并使用 VSCode + Vetur 打开它来尝试一下！

## TypeScript用户可能需要的操作

输入升级将在 Vue 2.5 中发布，目前计划在10月初发布。 我们正在发布一个小版本，因为 JavaScript 公共 API 没有任何突破性的变化，但是升级可能需要现有的 TypeScript + Vue 用户采取一些操作。 这就是为什么我们现在宣布改变，以便你有足够的时间来计划升级。

* 新的输入至少需要 TypeScript 2.4 版本，建议升级到最新版本的 TypeScript 以及 Vue 2.5。
* 之前，我们已经推荐将 `tsconfig.json` 设为 `“allowSyntheticDefaultImports”: true` 从而在任何地方使用 ES 风格的导入(`import Vue from 'vue'`)。 新的输入将正式转换为ES风格的导入/导出语法，因此不再需要配置，并且用户在所有情况下都需要使用ES风格的导入。
* 为了配合导出语法的改变，以下依赖于 Vue 核心输入的核心库 `vuex`, `vue-router`, `vuex-router-sync`, `vue-class-component` 将会收到新的主要版本，并且应与 Vue 核心 2.5 一起升级。
* 当执行自定义模块扩充时，用户应该使用 `interface VueConstructor` 而不是 `namespace Vue`。(example diff)
* 如果使用 `ComponentOptions <Something>` 对组件选项进行注释，则此类型的 `computed`，`watch`，`render` 和生命周期钩子将需要手动类型注解。

我们尽力减少所需的升级工作，这些类型的改进与 `vue-class-component` 中使用的基于类的 API 兼容。 对于大多数用户来说，只需升级依赖并切换到ES风格的导入即可。 同时，我们还建议你将Vue 版本锁定到2.4.x，直到你准备升级为止。

## 未来规划：vue-cli中的TypeScript支持

2.5之后，我们计划在下一个版本的 vue-cli 中引入对TypeScript 的官方支持，以便使 TS + Vue 用户更轻松地启动新项目。 敬请关注！

## 对于非TypeScript用户

这些更改不会以任何负面的方式影响非 TypeScript Vue 用户; 根据公共JavaScript API，2.5 将完全向后兼容，并且TypeScript CLI集成将完全选择加入。 但是如上所述，如果你使用[vue-language-server](https://github.com/vuejs/vetur/tree/master/server)强大的编辑器扩展，则会注意到更好的自动完成建议。

—

感谢 [Daniel Rosenwasser](https://github.com/danielrosenwasser), [HerringtonDarkholme](https://github.com/HerringtonDarkholme), [Katashin](https://github.com/ktsn) 以及 [Pine Wu](https://github.com/octref) 对于这些特性的工作以及对这篇文章的审阅。
