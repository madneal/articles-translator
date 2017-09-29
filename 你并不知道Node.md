# You don’t know Node

At this year’s Forward.js conference (a JavaScript one), I gave a talk titled “You don’t know Node”. In that talk, I challenged the audience with a set of questions about the Nodejs runtime and most of the *technical* audience could not answer most of the questions.

在今年 Forward.js （一个 JavaScript）会议中，我做了主题为“你并不知道 Node”的演讲。在这个演讲中，我向观众提出一些列关于 Node.js 运行时的具有挑战性的问题，大多数**技术相关观众**无法回答其中的大部分问题。

I didn’t really measure that but it certainly felt so in the room and a few brave people approached me after the talk and confessed to the fact.

我并没有进行实际的统计，但是在那个房间感觉是这个样子的。并且有一些勇敢的观众在演讲之后向我承认了这一事实。

Here is the problem that made me give that talk. I don’t think we teach Node the right way! Most educational content about Nodejs focuses on Node packages and not its runtime. Most of these packages wrap modules in the Node runtime itself (like *http*, or *stream*). When you run into problems, these problems might be inside the runtime itself and if you don’t know the Node runtime, you are in trouble.

这也就是我做此次演讲的原因。我不认为我们以正确的方式教学 Node！大多数的 Nodejs相关的教学概念集中于Node 包却不是它的运行时。大多数的包都将 Node 运行时包裹在模块中（比如 http 或者 stream）。当你遇到问题的时候，这些问题可能存在于运行时之中，如果你不知道Node 运行的话， 你就陷入麻烦了。

>  The problem: most educational content about Nodejs focuses on Node packages and not its runtime.
>
>  这个问题：多数的 Nodejs相关的教学概念集中于Node 包却不是它的运行时

I picked a few questions and answers to feature for this article. They are presented in headers below. Try to answer them in your head first!

我为这篇文章调取了一些问题并且做出回答。它们将以标题的形式在下面呈现。首先尝试在你自己心中做出回答！

*If you find a wrong or misleading answer here, please let me know.*

*如果你发现错误或者具有误导性的回答，请让我知道。*

## 问题 #1: 什么是调用栈并且它是V8的一部分么？

The Call Stack is definitely part of V8. It is the data structure that V8 uses to keep track of function invocations. Every time we invoke a function, V8 places a reference to that function on the call stack and it keeps doing so for each nested invocation of other functions. This also includes functions that call themselves recursively.

调用栈当然是 V8 的一部分。它是 V8 用来追踪函数调用的一种数据结构。每次我们调用一个函数的时候，V8都会将向函数调用栈中压入一个对于函数的引用，它对于其它函数的内嵌函数也会这样做。这也包括递归调用函数的函数。

![img](https://cdn-images-1.medium.com/max/800/1*9xKwtu4Gq-a7Pj_tWJ-tog.png)

When the nested invocations of functions reaches an end, V8 will pop one function at a time and use its returned value in its place.

当函数中的内嵌函数到达末端的时候，V8 就会每次弹出一个函数并且在它的位置使用返回值。

Why is this important to understand for Node? Because you only get ONE Call Stack per Node process. If you keep that Call Stack busy, your whole Node process is busy. Keep that in mind.

为什么理解 Node 是重要的？因为你在每个 Node 进程中只能获取一个调用栈。如果你让这个调用栈保持忙碌，那么你的整个 Node 进程也会是忙碌的。记住这一点。

#### 问题 #2: 什么是事件循环? 它是V8的一部分吗?

Where do you think the event loop is in this diagram?

你认为事件循环是在这张图的什么地方？

![img](https://cdn-images-1.medium.com/max/800/1*nLwOhFq_i4XbxRWUoXMlQQ.png)The event loop is provided by the *libuv* library. It is not part of V8.

时间循环是由 *libbuv* 库提供，它不是 V8 的一部分。

The Event Loop is the entity that handles external events and converts them into callback invocations. It is a loop that picks events from the event queues and pushes their callbacks into the Call Stack. It is also a multi-phase loop.

事件循环处理外部事件并且将它们转换成回调调用。它是一个从事件队列中跳去事件的循环并且将它们的回调压入到回调栈中。它也是一个多相回调。

If this is the first time you are hearing of the Event Loop, these definitions will not be that helpful. The Event Loop is part of a much bigger picture:

如果这是你第一次听说时间循环，这些概念将可能不会那么有用。这个时间循环是一张更大的图的一部分。

![img](https://cdn-images-1.medium.com/max/800/1*lj3_-x3yh-114QzWpFq8Ug.png)You need to understand that bigger picture to understand the Event Loop. You need to understand the role of V8, know about the Node APIs, and know how things get queued to be executed by V8.

你需要理解这张更大的图从而理解时间循环。你需要理解 V8 的角色，知道 Node 的 API，并且知道事件如何进入队列从而被 V8 所执行。

The Node APIs are functions like `setTimeout` or `fs.readFile`. These are not part of JavaScript itself. They are just functions provided by Node.

Node 的 API 就是一些函数，比如 `setTimeout` 或者 `fs.readFile`。这些并不是 JavaScript 中的一部分。它们是由 Node 提供的函数。

The Event Loop sits in the middle of this picture (a more complex version of it, really) and acts like an organizer. When V8 Call Stack is empty, the event loop can decide what to execute next.

事件循环位于这张图（真的是一个更复杂的版本）的中间位置，就好像是一个组织者。当 V8 调用栈为空的时候，事件循环可以决定下一步执行哪一个。

#### 问题 #3: 当调用栈以及事件循环都为空的时候，Node会做什么？

It will simply exit.

很简单，它会退出。

When you run a Node program, Node will automatically start the event loop and when that event loop is idle and has nothing else to do, the process will exit.

当你运行一个Node程序的时候，Node 将会自动开始事件循环并且当事件循环变为idle 的时候并且没有其它的事情需要做的时候，这个进程将会退出。

To keep a Node process running, you need to place something somewhere in event queues. For example, when you start a timer or an HTTP server you are basically telling the event loop to keep running and checking on these events.

为了保持 Node进行运行，你需要在事件队列的某个地方放一些东西。比如，当你启动一个计时器或者一个 HTTP 服务的时候，你基本上就是告诉事件循环保持运行并且检查这些事件。

#### 问题 #4: 除了V8以及Libuv，Node还有哪些外部依赖?

The following are all separate libraries that a Node process can use:

下面的是 Node 进行需要的所有的单独库：

- http-parser
- c-ares
- OpenSSL
- zlib

All of them are external to Node. They have their own source code. They have their own license. Node just uses them.

它们对于 Node 来说都是外部依赖。它们具有它们自己的源代码。它们具有它们自己的证书。Node 只是使用它们。

You want to remember that because you want to know where your program is running. If you are doing something with data compression, you might encounter trouble deep in the zlib library stack. You might be fighting a zlib bug. Do not blame everything on Node.

你希望记住因为你想知道你的程序在什么地方运行。如果你在处理数据压缩，你可能遇到一些 zlib 库使用的一些困难。你可能在解决一个 zlib 的 bug。不要把所有的事都归结为 Node。

#### 问题 #5: 可以不使用V8来运行 Node？

This might be a trick question. You do need a VM to run a Node process, but V8 is not the only VM that you can use. You can use *Chakra*.

这可能是一个棘手的问题。你需要一个 VM 来运行 Node 进程，但是 V8 并不是你唯一可以试用的 VM。你可以使用 *Chakra*。

Checkout this Github repo to track the progress of the node-chakra project:https://github.com/nodejs/node-**chakracore**

获取 Github仓库来跟追踪 node-chakra 的进程：https://github.com/nodejs/node-chakracore

#### 问题 #6: module.exports 和 exports 的区别是什么?

You can always use `module.exports` to export the API of your modules. You can also use `exports` except for one case:

你可以总是试用 `module.exports` 来导出你模块的 API。你也可以`exports` 除了一种情况：

```
module.exports.g = ...  // Ok
exports.g = ...         // Ok

module.exports = ...    // Ok
exports = ...           // Not Ok
```

*为什么？*

`exports` is just a reference or alias to `module.exports`. When you change `exports` you are changing that reference and no longer changing the official API (which is `module.exports`). You would just get a local variable in the module scope.

`exports` 只是一个对于 `module.exports` 的引用或者别名。当你改变 `exports` 的时候，你是在改变那个引用而不是改变官方的 API（`module.exports`）。你将只会获得一个模块作用域内的局部变量。

#### 问题 #7: 顶级变量怎么不是全局变量？

If you have `module1` that defines a top-level variable `g`:

如果你有一个 `module1` 定义了一个顶级变量 `g`：

```
// module1.js
var g = 42;
```

And you have `module2` that requires `module1` and try to access the variable `g`, you would get `g is not defined`.

并且你有一个 `module2` 引入了 `module1` 并且尝试访问变量 `g`，你将会发现 `g 是未定义的`。

*Why?* If you do the same in a browser you can access top-level defined variables in all scripts included after their definition.

*为什么？*在浏览器中你如果做同样的操作，你能够在所有的变量在定义之后被引入就可以访问顶级变量。

Every Node file gets its own *IIFE* (Immediately Invoked Function Expression) behind the scenes. All variables declared in a Node file are scoped to that IIFE.

每一个 Node 文件都会在背后获取它自己的立即执行函数表达式（IIFE）。所有在这个 Node 文件里面定义的变量都是在这个 IIFE 作用域内。

**Related Question:** What would be the output of running the following Node file that has just this single line:

**相关问题：**运行下面仅仅包含一行代码的 Node 文件的输出回事什么：

```
// script.js
console.log(arguments);
```

You are going to see some arguments!

你将会看到一些参数！

![img](https://cdn-images-1.medium.com/max/800/1*mLd8sj1_SFudZNisAeiOAQ.png)

*为什么?*

Because what Node executed is a function. Node wrapped your code with a function and that function explicitly defines the *five* arguments that you see above.

因为 Node 执行的是一个函数。 Node 将你的代码使用函数来包装并且这个函数准确定义了上面你看到的5个参数。

#### 问题 #8: 对象`exports`，`require`和`module`都是在每个文件中全局可用，但每个文件都不同。 这是如何实现的？

When you need to use the `require` object, you just use it directly as if it was a global variable. However, if you inspect `require` in two different files, you will see two different objects. How?

当你需要使用 `require` 对象的时候，你就可以直接使用它就好像是一个全局变量。然而，如果当你在不同的两个文件中检测 `require`，你将会看到两个不同的对象。这是如何实现的？

Because of that same magic IIFE:

都是因为相同的神奇的 IIFE：

![img](https://cdn-images-1.medium.com/max/800/1*W926fXZZIUf7vnvE2IOnZg.png)

As you can see, the magic IIFF passes to your code the following five arguments: `exports`, `require`, `module`, `__filename`, and `__dirname`.

正如你所见，神奇的 IIFE 将你的代码传递到5个参数之中：`exports`，`require`，`modue`，`__filename`以及`__dirname`。

These five variables appear to be global when you use them in Node, but they are actually just function arguments.

当你在 Node 试用这5个参数的时候，它们看起来是全局的，但是事实上它们仅仅是函数的参数。

#### 问题 #9: Node 中的循环模块依赖是什么?

If you have a `module1` that requires `module2` and `module2` that in-turn requires `module1`, what is going to happen? An error?

如果你有一个 `module1` 引入 `module2` 并且同样 `module2` 也引入了 `module1`，那么将会发生什么？一个错误？

```
// module1
require('./module2');
// module2
require('./module1');
```

You will not get an error. Node allows that.

你将不会得到一个错误。Node 允许这种情况。

So `module1` requires `module2`, but since `module2` needs `module1` and `module1` is not done yet, `module1` will just get a partial version of `module2`.

那么在 `module1` 引入 `module2` 的时候，但是因为 `module2` 需要 `module1` 并且 `module1` 尚未完成，`module1` 将会仅仅获取 `module2`的一个部分版本。

You have been warned.

你将被警告。

#### 问题 #10: 什么时候试用文件系统中的同步方法 (比如 readFileSync)是可以的？

Every `fs` method in Node has a synchronous version. Why would you use a sync method instead of the async one?

Node 中的每一个 `fs` 方法都有一个同步的版本。为什么你要使用一个同步方法而不是一个异步方法呢？

Sometimes using the sync method is fine. For example, it can be used in any initializing step while the server is still loading. It is often the case that everything you do after the initializing step depends on the data you obtain there. Instead of introducing a callback-level, using synchronous methods is acceptable, as long as what you use the synchronous methods for is a one-time thing.

有时候试用同步方法是不错的。比如，在初始化步骤中服务器依然在加载的情况下使用同步方法。大多数情况是初始化步骤之后的所有事取决与在初始化步骤中获取的数据。在不引入回调的层级，使用同步方法是可以接受的，只要你使用同步方法是一次性的事情。

However, if you are using synchronous methods inside a handler like an HTTP server on-request callback, that would simply be 100% wrong. Do not do that.

然而，如果你在一个处理程序中试用同步方法，比如 HTTP 服务器请求回调，那将会很明显 100% 报错。不要那样做。

------

I hope you were able to answer some or all of these challenge questions. I will leave you with some detailed articles I wrote about beyond-the-basics concepts in Node.js:

我希望你能够回答一些或者全部这些具有挑战性的问题。我将会给一些除了 Node.js 基本概念以外的文章。

https://medium.com/@samerbuna/you-dont-know-node-6515a658a1ed
