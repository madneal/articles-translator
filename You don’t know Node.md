# You don’t know Node

At this year’s Forward.js conference (a JavaScript one), I gave a talk titled “You don’t know Node”. In that talk, I challenged the audience with a set of questions about the Nodejs runtime and most of the *technical* audience could not answer most of the questions.

I didn’t really measure that but it certainly felt so in the room and a few brave people approached me after the talk and confessed to the fact.

Here is the problem that made me give that talk. I don’t think we teach Node the right way! Most educational content about Nodejs focuses on Node packages and not its runtime. Most of these packages wrap modules in the Node runtime itself (like *http*, or *stream*). When you run into problems, these problems might be inside the runtime itself and if you don’t know the Node runtime, you are in trouble.

>  The problem: most educational content about Nodejs focuses on Node packages and not its runtime.

I picked a few questions and answers to feature for this article. They are presented in headers below. Try to answer them in your head first!

*If you find a wrong or misleading answer here, please let me know.*

## Question #1: What is the Call Stack and is it part of V8?

The Call Stack is definitely part of V8. It is the data structure that V8 uses to keep track of function invocations. Every time we invoke a function, V8 places a reference to that function on the call stack and it keeps doing so for each nested invocation of other functions. This also includes functions that call themselves recursively.

![img](https://cdn-images-1.medium.com/max/800/1*9xKwtu4Gq-a7Pj_tWJ-tog.png)

When the nested invocations of functions reaches an end, V8 will pop one function at a time and use its returned value in its place.

Why is this important to understand for Node? Because you only get ONE Call Stack per Node process. If you keep that Call Stack busy, your whole Node process is busy. Keep that in mind.

#### Question #2: What is the Event Loop? Is it part of V8?

Where do you think the event loop is in this diagram?

![img](https://cdn-images-1.medium.com/max/800/1*nLwOhFq_i4XbxRWUoXMlQQ.png)The event loop is provided by the *libuv* library. It is not part of V8.

The Event Loop is the entity that handles external events and converts them into callback invocations. It is a loop that picks events from the event queues and pushes their callbacks into the Call Stack. It is also a multi-phase loop.

If this is the first time you are hearing of the Event Loop, these definitions will not be that helpful. The Event Loop is part of a much bigger picture:

![img](https://cdn-images-1.medium.com/max/800/1*lj3_-x3yh-114QzWpFq8Ug.png)You need to understand that bigger picture to understand the Event Loop. You need to understand the role of V8, know about the Node APIs, and know how things get queued to be executed by V8.

The Node APIs are functions like `setTimeout` or `fs.readFile`. These are not part of JavaScript itself. They are just functions provided by Node.

The Event Loop sits in the middle of this picture (a more complex version of it, really) and acts like an organizer. When V8 Call Stack is empty, the event loop can decide what to execute next.

#### Question #3: What will Node do when the Call Stack and the event loop queues are all empty?

It will simply exit.

When you run a Node program, Node will automatically start the event loop and when that event loop is idle and has nothing else to do, the process will exit.

To keep a Node process running, you need to place something somewhere in event queues. For example, when you start a timer or an HTTP server you are basically telling the event loop to keep running and checking on these events.

#### Question #4: Besides V8 and Libuv, what other external dependencies does Node have?

The following are all separate libraries that a Node process can use:

- http-parser
- c-ares
- OpenSSL
- zlib

All of them are external to Node. They have their own source code. They have their own license. Node just uses them.

You want to remember that because you want to know where your program is running. If you are doing something with data compression, you might encounter trouble deep in the zlib library stack. You might be fighting a zlib bug. Do not blame everything on Node.

#### Question #5: Would it be possible to run a Node process without V8?

This might be a trick question. You do need a VM to run a Node process, but V8 is not the only VM that you can use. You can use *Chakra*.

Checkout this Github repo to track the progress of the node-chakra project:https://github.com/nodejs/node-**chakracore**

#### Question #6: What is the difference between module.exports and exports?

You can always use `module.exports` to export the API of your modules. You can also use `exports` except for one case:

```
module.exports.g = ...  // Ok
```

```
exports.g = ...         // Ok
```

```
module.exports = ...    // Ok
exports = ...           // Not Ok
```

*Why?*

`exports` is just a reference or alias to `module.exports`. When you change `exports` you are changing that reference and no longer changing the official API (which is `module.exports`). You would just get a local variable in the module scope.

#### Question #7: How come top-level variables are not global?

If you have `module1` that defines a top-level variable `g`:

```
// module1.js
```

```
var g = 42;
```

And you have `module2` that requires `module1` and try to access the variable `g`, you would get `g is not defined`.

*Why?* If you do the same in a browser you can access top-level defined variables in all scripts included after their definition.

Every Node file gets its own *IIFE* (Immediately Invoked Function Expression) behind the scenes. All variables declared in a Node file are scoped to that IIFE.

**Related Question:** What would be the output of running the following Node file that has just this single line:

```
// script.js
console.log(arguments);
```

You are going to see some arguments!



![img](https://cdn-images-1.medium.com/max/800/1*mLd8sj1_SFudZNisAeiOAQ.png)

*Why?*

Because what Node executed is a function. Node wrapped your code with a function and that function explicitly defines the *five* arguments that you see above.

#### Question #8: The objects `exports`, `require`, and `module` are all globally available in every file but they are different in every file. How?

When you need to use the `require` object, you just use it directly as if it was a global variable. However, if you inspect `require` in two different files, you will see two different objects. How?

Because of that same magic IIFE:



![img](https://cdn-images-1.medium.com/max/800/1*W926fXZZIUf7vnvE2IOnZg.png)

As you can see, the magic IIFF passes to your code the following five arguments: `exports`, `require`, `module`, `__filename`, and `__dirname`.

These five variables appear to be global when you use them in Node, but they are actually just function arguments.

#### Question #9: What are circular module dependencies in Node?

If you have a `module1` that requires `module2` and `module2` that in-turn requires `module1`, what is going to happen? An error?

```
// module1
```

```
require('./module2');
```

```
// module2
```

```
require('./module1');
```

You will not get an error. Node allows that.

So `module1` requires `module2`, but since `module2` needs `module1` and `module1` is not done yet, `module1` will just get a partial version of `module2`.

You have been warned.

#### Question #10: When is it OK to use the file system *Sync methods (like readFileSync)

Every `fs` method in Node has a synchronous version. Why would you use a sync method instead of the async one?

Sometimes using the sync method is fine. For example, it can be used in any initializing step while the server is still loading. It is often the case that everything you do after the initializing step depends on the data you obtain there. Instead of introducing a callback-level, using synchronous methods is acceptable, as long as what you use the synchronous methods for is a one-time thing.

However, if you are using synchronous methods inside a handler like an HTTP server on-request callback, that would simply be 100% wrong. Do not do that.

------

I hope you were able to answer some or all of these challenge questions. I will leave you with some detailed articles I wrote about beyond-the-basics concepts in Node.js:

https://medium.com/@samerbuna/you-dont-know-node-6515a658a1ed