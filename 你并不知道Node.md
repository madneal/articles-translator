## 你并不知道Node

> 原文：[You don’t know Node](https://medium.com/@samerbuna/you-dont-know-node-6515a658a1ed)
>
> 译者：[neal1991](https://github.com/neal1991)
>
> welcome to star my [articles-translator](https://github.com/neal1991/articles-translator/), providing you advanced articles translation. Any suggestion, please issue or contact [me](mailto:bing@stu.ecnu.edu.cn)
>
> LICENSE: [MIT](https://opensource.org/licenses/MIT)

在今年 Forward.js （一个 JavaScript）会议中，我做了主题为“你并不知道 Node”的演讲。在这个演讲中，我向观众提出一些关于 Node.js 运行时的具有挑战性的问题，大多数**技术相关观众**无法回答其中的大部分问题。

我并没有进行实际的统计，但是在那个房间感觉是这个样子的。并且有一些勇敢的观众在演讲之后向我承认了这一事实。

这也就是我做此次演讲的原因。我不认为我们以正确的方式教学 Node！大多数的 Nodejs相关的教学概念集中于Node 包却不是它的运行时。大多数的包都将 Node 运行时包裹在模块中（比如 http 或者 stream）。当你遇到问题的时候，这些问题可能存在于运行时之中，如果你不知道Node 运行时的话， 你就陷入麻烦了。

>  这个问题：多数的 Nodejs相关的教学概念集中于Node 包却不是它的运行时

我为这篇文章选择了一些问题并且做出回答。它们将以标题的形式在下面呈现。首先尝试在你自己心中做出回答！

*如果你发现错误或者具有误导性的回答，请让我知道。*

### 问题 #1: 什么是调用栈并且它是V8的一部分么？

调用栈当然是 V8 的一部分。它是 V8 用来追踪函数调用的一种数据结构。每次我们调用一个函数的时候，V8都会将向函数调用栈中压入一个对于函数的引用，它对于其它函数的内嵌函数也会这样做。这也包括递归调用函数的函数。

![img](https://cdn-images-1.medium.com/max/800/1*9xKwtu4Gq-a7Pj_tWJ-tog.png)

当函数中的内嵌函数到达末端的时候，V8 就会每次弹出一个函数并且在它的位置使用返回值。

为什么理解 Node 是重要的？因为你在每个 Node 进程中只能获取一个调用栈。如果你让这个调用栈保持忙碌，那么你的整个 Node 进程也会是忙碌的。记住这一点。

#### 问题 #2: 什么是事件循环? 它是V8的一部分吗?

你认为事件循环是在这张图的什么地方？

![img](https://cdn-images-1.medium.com/max/800/1*nLwOhFq_i4XbxRWUoXMlQQ.png)The event loop is provided by the *libuv* library. It is not part of V8.

事件循环是由 *libbuv* 库提供，它不是 V8 的一部分。

事件循环处理外部事件并且将它们转换成回调调用。它是一个从事件队列中跳去事件的循环并且将它们的回调压入到调用栈中。它也是一个多相回调。

如果这是你第一次听说事件循环，这些概念将可能不会那么有用。这个事件循环是一张更大的图的一部分。

![img](https://cdn-images-1.medium.com/max/800/1*lj3_-x3yh-114QzWpFq8Ug.png)

你需要理解这张更大的图从而理解事件循环。你需要理解 V8 的角色，知道 Node 的 API，并且知道事件如何进入队列从而被 V8 所执行。


Node 的 API 就是一些函数，比如 `setTimeout` 或者 `fs.readFile`。这些并不是 JavaScript 中的一部分。它们是由 Node 提供的函数。


事件循环位于这张图（真的是一个更复杂的版本）的中间位置，就好像是一个组织者。当 V8 调用栈为空的时候，事件循环可以决定下一步执行哪一个。

#### 问题 #3: 当调用栈以及事件循环都为空的时候，Node会做什么？

很简单，它会退出。


当你运行一个Node程序的时候，Node 将会自动开始事件循环并且当事件循环变为idle 的时候并且没有其它的事情需要做的时候，这个进程将会退出。

为了保持 Node进行运行，你需要在事件队列的某个地方放一些东西。比如，当你启动一个计时器或者一个 HTTP 服务的时候，你基本上就是告诉事件循环保持运行并且检查这些事件。

### 问题 #4: 除了V8以及Libuv，Node还有哪些外部依赖?

下面的是 Node 进行需要的所有的单独库：

- http-parser
- c-ares
- OpenSSL
- zlib

它们对于 Node 来说都是外部依赖。它们具有它们自己的源代码。它们具有它们自己的证书。Node 只是使用它们。

你希望记住因为你想知道你的程序在什么地方运行。如果你在处理数据压缩，你可能遇到一些 zlib 库使用的一些困难。你可能在解决一个 zlib 的 bug。不要把所有的事都怪罪于 Node。

### 问题 #5: 可以不使用V8来运行 Node？

这可能是一个棘手的问题。你需要一个 VM 来运行 Node 进程，但是 V8 并不是你唯一可以试用的 VM。你可以使用 *Chakra*。

获取 Github仓库来跟追踪 node-chakra 的进程：https://github.com/nodejs/node-chakracore

### 问题 #6: module.exports 和 exports 的区别是什么?

你可以总是试用 `module.exports` 来导出你模块的 API。你也可以`exports` 除了一种情况：

```
module.exports.g = ...  // Ok
exports.g = ...         // Ok

module.exports = ...    // Ok
exports = ...           // Not Ok
```

*为什么？*

`exports` 只是一个对于 `module.exports` 的引用或者别名。当你改变 `exports` 的时候，你是在改变那个引用而不是改变官方的 API（`module.exports`）。你将只会获得一个模块作用域内的局部变量。

### 问题 #7: 顶级变量怎么不是全局变量？

如果你有一个 `module1` 定义了一个顶级变量 `g`：

```
// module1.js
var g = 42;
```

并且你有一个 `module2` 引入了 `module1` 并且尝试访问变量 `g`，你将会发现 `g 是未定义的`。

*为什么？*在浏览器中你如果做同样的操作，你能够在所有的变量在定义之后被引入就可以访问顶级变量。

每一个 Node 文件都会在背后获取它自己的立即执行函数表达式（IIFE）。所有在这个 Node 文件里面定义的变量都是在这个 IIFE 作用域内。

**相关问题**:运行下面仅仅包含一行代码的 Node 文件的输出回事什么：

```
// script.js
console.log(arguments);
```

你将会看到一些参数！

![img](https://cdn-images-1.medium.com/max/800/1*mLd8sj1_SFudZNisAeiOAQ.png)

*为什么?*


因为 Node 执行的是一个函数。 Node 将你的代码使用函数来包装并且这个函数准确定义了上面你看到的5个参数。

### 问题 #8: 对象`exports`，`require`和`module`都是在每个文件中全局可用，但每个文件都不同。 这是如何实现的？


当你需要使用 `require` 对象的时候，你就可以直接使用它就好像是一个全局变量。然而，如果当你在不同的两个文件中检测 `require`，你将会看到两个不同的对象。这是如何实现的？

Because of that same magic IIFE:

都是因为相同的神奇的 IIFE：

![img](https://cdn-images-1.medium.com/max/800/1*W926fXZZIUf7vnvE2IOnZg.png)

正如你所见，神奇的 IIFE 将你的代码传递到5个参数之中：`exports`，`require`，`modue`，`__filename` 以及 `__dirname`。

当你在 Node 试用这5个参数的时候，它们看起来是全局的，但是事实上它们仅仅是函数的参数。

### 问题 #9: Node 中的循环模块依赖是什么?

如果你有一个 `module1` 引入 `module2` 并且同样 `module2` 也引入了 `module1`，那么将会发生什么？一个错误？

```
// module1
require('./module2');
// module2
require('./module1');
```

你将不会得到一个错误。Node 允许这种情况。

那么在 `module1` 引入 `module2` 的时候，但是因为 `module2` 需要 `module1` 并且 `module1` 尚未完成，`module1` 将会仅仅获取 `module2`的一个部分版本。

你将被警告。

### 问题 #10: 什么时候试用文件系统中的同步方法 (比如 readFileSync)是可以的？

Node 中的每一个 `fs` 方法都有一个同步的版本。为什么你要使用一个同步方法而不是一个异步方法呢？

有时候试用同步方法是不错的。比如，在初始化步骤中服务器依然在加载的情况下使用同步方法。大多数情况是初始化步骤之后的所有事取决与在初始化步骤中获取的数据。在不引入回调的层级，使用同步方法是可以接受的，只要你使用同步方法是一次性的事情。

然而，如果你在一个处理程序中试用同步方法，比如 HTTP 服务器请求回调，那将会很明显 100% 报错。不要那样做。

------

我希望你能够回答一些或者全部这些具有挑战性的问题。我将会给一些除了 Node.js 基本概念以外的文章。
* https://medium.freecodecamp.org/before-you-bury-yourself-in-packages-learn-the-node-js-runtime-itself-f9031fbd8b69
* https://medium.freecodecamp.org/requiring-modules-in-node-js-everything-you-need-to-know-e7fbd119be8
* https://medium.freecodecamp.org/understanding-node-js-event-driven-architecture-223292fcbc2d
* https://medium.freecodecamp.org/node-js-streams-everything-you-need-to-know-c9141306be93
* https://medium.freecodecamp.org/node-js-child-processes-everything-you-need-to-know-e69498fe970a
* https://medium.freecodecamp.org/scaling-node-js-applications-8492bd8afadc
