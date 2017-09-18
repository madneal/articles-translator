https://blog.sessionstack.com/how-javascript-works-memory-management-how-to-handle-4-common-memory-leaks-3f28b94cfbec

## How JavaScript works: memory management + how to handle 4 common memory leaks
## JavaScript是如何工作的：内存管理以及如何处理四种常见的内存泄漏

A few weeks ago we started a series aimed at digging deeper into JavaScript and how it actually works: we thought that by knowing the building blocks of JavaScript and how they come to play together you’ll be able to write better code and apps.

几个礼拜之前我们开始一系列对于JavaScript以及其本质工作原理的深入挖掘：我们认为通过了解JavaScript的构建方式以及它们是如何共同合作的，你就能够写出更好的代码以及应用。

The first post of the series focused on providing [an overview of the engine, the runtime, and the call stack](https://blog.sessionstack.com/how-does-javascript-actually-work-part-1-b0bacc073cf). Thе [second post examined closely the internal parts of Google’s V8 JavaScript engine](https://blog.sessionstack.com/how-javascript-works-inside-the-v8-engine-5-tips-on-how-to-write-optimized-code-ac089e62b12e) and also provided a few tips on how to write better JavaScript code.

这个系列的第一篇博客专注于介绍[对于引擎，运行时以及调用栈的概述](https://blog.sessionstack.com/how-does-javascript-actually-work-part-1-b0bacc073cf)（译者注：[第一篇博客翻译版](https://github.com/neal1991/articles-translator/blob/master/JavaScript%E6%98%AF%E5%A6%82%E4%BD%95%E5%B7%A5%E4%BD%9C%EF%BC%9A%E7%B3%BB%E5%88%97%E4%B8%80.md)）。[第二篇博客近距离地检测了Google V8 引擎的内部](https://blog.sessionstack.com/how-javascript-works-inside-the-v8-engine-5-tips-on-how-to-write-optimized-code-ac089e62b12)并且提供了一些如何写出更好的JavaScript代码的建议。

In this third post, we’ll discuss another critical topic that’s getting ever more neglected by developers due to the increasing maturity and complexity of programming languages that are being used on a daily basis — memory management. We’ll also provide a few tips on how to handle memory leaks in JavaScript that we at [SessionStack](https://www.sessionstack.com/) follow as we need to make sure SessionStack causes no memory leaks or doesn’t increase the memory consumption of the web app in which we are integrated.

在第三篇博客中，我们将会讨论另外一个关键的话题。这个话题有趣编程语言的逐渐成熟和复杂化，越来越被开发者忽视，这个话题就是在日常中使用到的——内存管理。我们还将提供一些有关如何处理我们在[SessionStack](https://www.sessionstack.com/)中的JavaScript中的内存泄漏的建议，因为我们需要确保SessionStack不会导致内存泄漏或者增加我们集成的Web应用程序的内存消耗

### 概述

Languages, like C, have low-level memory management primitives such as malloc() and free(). These primitives are used by the developer to explicitly allocate and free memory from and to the operating system.

语言，比如C，具有低层次的内存管理方法，比如`malloc()`以及`free()`。开发者利用这些方法精确地为操作系统分配以及释放内存。

At the same time, JavaScript allocates memory when things (objects, strings, etc.) are created and “automatically” frees it up when they are not used anymore, a process called *garbage collection*. This seemingly “automatical” nature of freeing up resources is a source of confusion and gives JavaScript (and other high-level-language) developers the false impression they can choose not to care about memory management.** This is a big mistake.**

同时，JavaScript会在创建一些变量（对象，字符串等等）的时候分配内存，并且会在这些不被使用之后“自动地”释放这些内存，这个过程被称为*垃圾收集*。这个看起来“自动化的”特性其实就是产生误解的原因，并且给JavaScript（以及其他高层次语言）开发者一个假象，他们不需要关心内存管理。**大错特错。**

Even when working with high-level languages, developers should have an understanding of memory management (or at least the basics). Sometimes there are issues with the automatic memory management (such as bugs or implementation limitations in the garbage collectors, etc.) which developers have to understand in order to handle them properly (or to find a proper workaround, with a minimum trade off and code debt).

即使是使用高层次语言，开发者硬挨对于内存管理有一定的理解（或者最基本的理解）。有时候自动的内存管理会存在一些问题（比如一些bug或者垃圾收集器的一些限制等等），对于这些开发者必须能够理解从而能够合适地处理（或者使用最小的代价以及代码债务去绕过这个问题）。

### 内存生命周期

No matter what programming language you’re using, memory life cycle is pretty much always the same:

不管你在使用什么编程语言，内存的生命周期基本上都是一样的：

![](https://cdn-images-1.medium.com/max/2048/1*slxXgq_TO38TgtoKpWa_jQ.png) 


Here is an overview of what happens at each step of the cycle:

下面是对于周期中每一步所发生的情况的概述：

 * **Allocate memory **— memory is allocated by the operating system which allows your program to use it. In low-level languages (e.g. C) this is an explicit operation that you as a developer should handle. In high-level languages, however, this is taken care of for you.

 * **Use memory — **this is the time when your program actually makes use of the previously allocated memory. **Read** and **write** operations are taking place as you’re using the allocated variables in your code.

 * **Release memory** — now is the time to release the entire memory that you don’t need so that it can become free and available again. As with the **Allocate memory **operation, this one is explicit in low-level languages.

 * **分配内存**——操作系统为你的程序分配内存并且允许其使用。在低层次语言中（比如C），这正式开发者应该处理的操作。在高层次的语言，然而，就有语言帮你实现了。

 * **使用内存**——当你的程序确实在使用之前分配的内存的阶段。当你在使用你带吗里面分配的变量的时候会发生**读**以及**写**操作。

 * **释放内存**——这个阶段就是释放你不再需要的内存，从而这些内存被释放并且能够再次被使用。和**分配内存**操作一样，这在低层次的语言也是一个精确的操作。

For a quick overview of the concepts of the call stack and the memory heap, you can read our [first post on the topic](https://blog.sessionstack.com/how-does-javascript-actually-work-part-1-b0bacc073cf).

对于调用栈以及内存堆有一个快速的概念认识，你可以阅读我们[关于这个话题的第一篇博客](https://blog.sessionstack.com/how-does-javascript-actually-work-part-1-b0bacc073cf)。

### 什么是内存？

Before jumping straight to memory in JavaScript, we’ll briefly discuss what memory is in general and how it works in a nutshell.

在我们讲述JavaScript内存之前，我们将简要地讨论一下内存是什么以及它们是如何在nutshell中工作的。

On a hardware level, computer memory consists of a large number of
[flip flops](https://en.wikipedia.org/wiki/Flip-flop_%28electronics%29). Each flip flop contains a few transistors and is capable of storing one bit. Individual flip flops are addressable by a **unique identifier**, so we can read and overwrite them. Thus, conceptually, we can think of our entire computer memory as a just one giant array of bits that we can read and write.

在硬件层次上，计算机内存由大量的 [flip flops](https://en.wikipedia.org/wiki/Flip-flop_%28electronics%29) 组成。每一个 flip flop 都包含一些晶体管并且能够存储一比特。单独的 flip flop 可以通过**独特的标识符**去访问，因此我们能够读取以及重写它们。因此，从概念上来说，我们可以认为我们的整个计算机内存就是一个我们能够读写巨大的比特数组。

Since as humans, we are not that good at doing all of our thinking and arithmetic in bits, we organize them into larger groups, which together can be used to represent numbers. 8 bits are called 1 byte. Beyond bytes, there are words (which are sometimes 16, sometimes 32 bits).

因为作为人类，我们不擅长直接基于比特进行思考以及算术，我们将它们组织成大规模群组，它们在一起可以代表一个数字。8个比特称为一个字节。除了字节，还有词（有时候是16比特，有时候是32比特）。

A lot of things are stored in this memory:

内存中存储了很多东西：

    1. All variables and other data used by all programs.

    2. The programs’ code, including the operating system’s.

    3. 所有程序使用的变量和其他数据。

  2.程序的代码，包括操作系统的代码。

The compiler and the operating system work together to take care of most of the memory management for you, but we recommend that you take a look at what’s going on under the hood.

编译器和操作系统共同合作为你处理大部分的内存管理，但是我们建议你应该了解其内部的运行原理。

When you compile your code, the compiler can examine primitive data types and calculate ahead of time how much memory they will need. The required amount is then allocated to the program in the call** stack space**. The space in which these variables are allocated is called the stack space because as functions get called, their memory gets added on top of the existing memory. As they terminate, they are removed in a LIFO (last-in, first-out) order. For example, consider the following declarations:

当你编译你的代码的时候，编译器将会检查原始数据类型并且提前计算好它们需要多少内存。需要的内存被分配给程序，这被称为**栈空间**。这些被分配给变量的空间被称为栈空间，因为一旦函数被调用，它们的内存就会增加到现有内存的上面。当它们终止的时候，它们就会以后进先出(LIFO)的顺序移除。比如，考虑下面的声明。

```
    int n; // 4 bytes
    int x[4]; // array of 4 elements, each 4 bytes
    double m; // 8 bytes
```

The compiler can immediately see that the code requires 

编译器能够立即计算出代码需要

4 + 4 × 4 + 8 = 28 bytes.

>  That’s how it works with the current sizes for integers and doubles. About 20 years ago, integers were typically 2 bytes, and double 4 bytes. Your code should never have to depend on what is at this moment the size of the basic data types.

> 那就是它如何对于现有的整形以及双浮点型工作。大约20年前，整形典型都是2个字节，双浮点型是4个字节。你的代码不应该取决于当下基本数据类型的大小。

The compiler will insert code that will interact with the operating system to request the necessary number of bytes on the stack for your variables to be stored.

编译器将会插入能够与操作系统交互的代码，从而在栈上获取你需要存储变量需要的字节数。

In the example above, the compiler knows the exact memory address of each variable. In fact, whenever we write to the variable n, this gets translated into something like “memory address 4127963” internally.

在上述的例子中，编译器知道每一个变量的准确的内存地址。事实上，无论我们何时写变量 n ，这都会在内部转化为类似于“内存地址 4127963”的东西。

Notice that if we attempted to access x[4] here, we would have accessed the data associated with m . That’s because we’re accessing an element in the array that doesn’t exist — it’s 4 bytes further than the last actual allocated element in the array which is x[3], and may end up reading (or overwriting) some of m’s bits. This would almost certainly have very undesired consequences for the rest of the program.

注意如果我们希望在这访问 x[4] 我们将会需要访问和 m 一起的数据。这是因为我们在访问数组里面并不存在的元素——它比数组实际分配的最后一个元素x[3]要多4个字节，并且最后可能是阅读（或者重写）一些 m 的比特。这将很可能给程序的其他部分带来一些不良的后果。

![](https://cdn-images-1.medium.com/max/2048/1*5aBou4onl1B8xlgwoGTDOg.png) 

When functions call other functions, each gets its own chunk of the stack when it is called. It keeps all its local variables there, but also a program counter that remembers where in its execution it was. When the function finishes, its memory block is once again made available for other purposes.

当函数调用其它函数的时候，当它被调用的时候都会获取它自己的堆栈块。它在那保存了它所有的局部变量，但是还会有一个程序计数器记录它执行的位置。当这个函数执行完毕，它的内存块就可以再次用于其他目的。

### 动态分配

Unfortunately, things aren’t quite as easy when we don’t know at compile time how much memory a variable will need. Suppose we want to do something like the following:

不幸的是，当我们在编译的时候不知道变量需要多少内存的话事情不可能就这么简单。假设我们想做下面的事情：
```

    int n = readInput(); // reads input from the user

    ...

    // create an array with "n" elements
```

Here, at compile time, the compiler does not know how much memory the array will need because it is determined by the value provided by the user.

在此，在编译阶段中，编译器就没有办法知道数组需要多少内存，因为它取决于用户提供的值。

It, therefore, cannot allocate room for a variable on the stack. Instead, our program needs to explicitly ask the operating system for the right amount of space at run-time. This memory is assigned from the **heap space**. The difference between static and dynamic memory allocation is summarized in the following table:

因此，它就不能够为栈上的变量分配空间。相反，我们的程序需要明确地询问操作运行时需要的空间数量。这个内存是从**堆空间**中分配出来的。动态内存和静态内存分配的区别总结如下表格：

![Differences between statically and dynamically allocated memory](https://cdn-images-1.medium.com/max/2048/1*qY-yRQWGI-DLS3zRHYHm9A.png) 


To fully understand how dynamic memory allocation works, we need to spend more time on **pointers**, which might be a bit too much of a deviation from the topic of this post. If you’re interested in learning more, just let me know in the comments and we can go into more details about pointers in a future post.

为了深入地理解动态内存分配是如何工作的，我们需要花费更多的时间在**指针**，这个可能有点偏离这篇博客的话题。如果你感兴趣了解更多，在评论里面告诉，我将会在后续的博客中挖掘更多的细节。

### JavaScript中的分配

Now we’ll explain how the first step (allocate memory) works in JavaScript.

现在我们将解释JavaScript中的第一步（分配内存）。

JavaScript relieves developers from the responsibility to handle memory allocations — JavaScript does it by itself, alongside declaring values.

JavaScript 将开发者从内存分配的处理中解放出来——JavaScript自身可以利用生命变量来完成这些任务。

```
    var n = 374; // allocates memory for a number
    var s = 'sessionstack'; // allocates memory for a string 
    
    var o = {
      a: 1,
      b: null
    }; // allocates memory for an object and its contained values
    
    var a = [1, null, 'str'];  // (like object) allocates memory for the
                               // array and its contained values
    
    function f(a) {
      return a + 3;
    } // allocates a function (which is a callable object)
    
    // function expressions also allocate an object
    someElement.addEventListener('click', function() {
      someElement.style.backgroundColor = 'blue';
    }, false);
```

Some function calls result in object allocation as well:

一些函数调用也会导致一些对象的分配：

```

    var d = new Date(); // allocates a Date object

    var e = document.createElement('div'); // allocates a DOM element
```

Methods can allocate new values or objects:

能够分配新的值或者对象的方法：

```javascript
    var s1 = 'sessionstack';
    var s2 = s1.substr(0, 3); // s2 is a new string
    // Since strings are immutable, 
    // JavaScript may decide to not allocate memory, 
    // but just store the [0, 3] range.
    
    var a1 = ['str1', 'str2'];
    var a2 = ['str3', 'str4'];
    var a3 = a1.concat(a2); 
    // new array with 4 elements being
    // the concatenation of a1 and a2 elements
```

### 在JavaScript中使用内存

Using the allocated memory in JavaScript basically, means reading and writing in it.

基本上在JavaScript中分配内存，就意味着在其中读写。

This can be done by reading or writing the value of a variable or an object property or even passing an argument to a function.

这可以通过对一个变量或者一个对象的属性甚至是向函数传递一个参数来完成。

### 当内存不再需要的时候释放它

Most of the memory management issues come at this stage.

大多数的内存管理的问题就来自于这个阶段。

The hardest task here is to figure out when the allocated memory is not needed any longer. It often requires the developer to determine where in the program such piece of memory is not needed anymore and free it.

最困难的任务就是如何知道何时被分配的不再需要了。它经常需要开发者决定在程序的什么地方某段内存不再需要了并且对其进行释放。

High-level languages embed a piece of software called **garbage collector** which job is to track memory allocation and use in order to find when a piece of allocated memory is not needed any longer in which case, it will automatically free it.

高层次语言内嵌了一个被称为**垃圾收集器**的软件，他的任务就是跟踪内存分配并且用于需找不再需要的分配过的内存，并且自动地对其进行释放。

Unfortunately, this process is an approximation since the general problem of knowing whether some piece of memory is needed is [undecidable](http://en.wikipedia.org/wiki/Decidability_%28logic%29) (can’t be solved by an algorithm).

不幸的是，这个过程是一个近似，因为知道是否某块内存是需要的问题是[不可决定的](http://en.wikipedia.org/wiki/Decidability_%28logic%29)（无法通过算法解决）

Most garbage collectors work by collecting memory which can no longer be accessed, e.g. all variables pointing to it went out of scope. That’s, however, an under-approximation of the set of memory spaces that can be collected, because at any point a memory location may still have a variable pointing to it in scope, yet it will never be accessed again.

大多数的垃圾收集器通过收集再也无法访问的内存工作，比如：指向它的所有变量都超出了范围。然而，这依然是对于可以收集的内存空间的预估，因为在任何位置仍可能一些变量在范围内指向这个内存，然而它再也不能被访问了。

### 垃圾收集器

Due to the fact that finding whether some memory is “not needed anymore” is undecidable, garbage collections implement a restriction of a solution to the general problem. This section will explain the necessary notions to understand the main garbage collection algorithms and their limitations.

由于找到一些是“不再需要的”是不可决定的事实，垃圾收集实现了对一般问题的解决方案的限制。这一节将会解释理解主要的垃圾手机算法以及它们的限制的需要注意的事项。


### 内存引用

The main concept garbage collection algorithms rely on is the one of **reference**.

垃圾收集算法依赖的主要概念之一就是**引用**。

Within the context of memory management, an object is said to reference another object if the former has an access to the latter (can be implicit or explicit). For instance, a JavaScript object has a reference to its [prototype](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Inheritance_and_the_prototype_chain) (**implicit reference**) and to its properties’ values (**explicit reference**).

在内存管理的上下文中，一个对象被称为是对于另外一个对象的引用，如果前者可以访问后者（隐含或明确的）。例如，一个JavaScript对象都有一个指向其[原型](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Inheritance_and_the_prototype_chain)的引用（**隐含的引用**）

In this context, the idea of an “object” is extended to something broader than regular JavaScript objects and also contains function scopes (or the global **lexical scope**).

在这个上下文中，“对象”的概念扩展到比普通的JavaScript对象要广并且包括函数作用域（或者全局**词法作用域**）。

>  Lexical Scoping defines how variable names are resolved in nested functions: inner functions contain the scope of parent functions even if the parent function has returned.
>
>  词法作用域定义了变量名称是如何在嵌套函数中解析的：内部函数包含了父函数的作用域即使父函数已经返回了。
### 基于引用计数的垃圾收集器

This is the simplest garbage collection algorithm. An object is considered “garbage collectible” if there are **zero** references pointing to it.

这是最简单的垃圾收集器算法。如果没有引用指向这个对象的时候，这个对象就被认为是“可以作为垃圾收集”。

Take a look at the following code:

请看如下代码：

```javascript
var o1 = {
  o2: {
    x: 1
  }
};

// 2 objects are created. 
// 'o2' is referenced by 'o1' object as one of its properties.
// None can be garbage-collected
var o3 = o1; // the 'o3' variable is the second thing that 
            // has a reference to the object pointed by 'o1'. 
o1 = 1;      // now, the object that was originally in 'o1' has a         
            // single reference, embodied by the 'o3' variable

var o4 = o3.o2; // reference to 'o2' property of the object.
                // This object has now 2 references: one as
                // a property. 
                // The other as the 'o4' variable

o3 = '374'; // The object that was originally in 'o1' has now zero
            // references to it. 
            // It can be garbage-collected.
            // However, what was its 'o2' property is still
            // referenced by the 'o4' variable, so it cannot be
            // freed.

o4 = null; // what was the 'o2' property of the object originally in
           // 'o1' has zero references to it. 
           // It can be garbage collected.
```

### 循环在产生问题

There is a limitation when it comes to cycles. In the following example, two objects are created and reference one another, thus creating a cycle. They will go out of scope after the function call, so they are effectively useless and could be freed. However, the reference-counting algorithm considers that since each of the two objects is referenced at least once, neither can be garbage-collected.

当遇到循环的时候就会有一个限制。在下面的实例之中，创建两个对象，并且互相引用，因此就会产生一个循环。当函数调用结束之后它们会走出作用域之外，因此它们就没什么用并且可以被释放。但是，基于引用计数的算法认为这两个对象都会被至少引用一次，所以它俩都不会被垃圾收集器收集。

```javascript
function f() {
  var o1 = {};
  var o2 = {};
  o1.p = o2; // o1 references o2
  o2.p = o1; // o2 references o1. This creates a cycle.
}

f();
```
![](https://cdn-images-1.medium.com/max/2000/1*GF3p99CQPZkX3UkgyVKSHw.png) 

### 标记-清除算法

In order to decide whether an object is needed, this algorithm determines whether the object is reachable.

为了决定哪个对象是需要的，算法会决定是否这个对象是可访问的。

The algorithm consists of the following steps:

这个算法由以下步骤组成：

1. 这个垃圾收集器构建一个“roots”列表。Root是全局变量，被代码中的引用所保存。在JavaScript中，“window”就是这样的作为root的全局变量的例子。
2. 所有的root都会被监测并且被标志成活跃的（比如不是垃圾）。所有的子代也会递归地被监测。所有能够由root访问的一切都不会被认为是垃圾。
3. 所有不再被标志成活跃的内存块都被认为是垃圾。这个收集器现在就可以释放这些内存并将它们返还给操作系统。

    1. The garbage collector builds a list of “roots”. Roots usually are global variables to which a reference is kept in the code. In JavaScript, the “window” object is an example of a global variable that can act as a root.

    2. All roots are inspected and marked as active (i.e. not garbage). All children are inspected recursively as well. Everything that can be reached from a root is not considered garbage.

    3. All pieces of memory not marked as active can now be considered garbage. The collector can now free that memory and return it to the OS.
      ![A visualization of the mark and sweep algorithm in action](https://cdn-images-1.medium.com/max/2000/1*WVtok3BV0NgU95mpxk9CNg.gif) 


This algorithm is better than the previous one since “an object has zero reference” leads to this object being unreachable. The opposite is not true as we have seen with cycles.

这个算法要优于之前的因为“一个具有0引用的对象”可以让一个对象不能够再被访问。但是相反的却不一定成立，比如我们遇到循环的时候。

As of 2012, all modern browsers ship a mark-and-sweep garbage-collector. All improvements made in the field of JavaScript garbage collection (generational/incremental/concurrent/parallel garbage collection) over the last years are implementation improvements of this algorithm (mark-and-sweep), but not improvements over the garbage collection algorithm itself, nor its goal of deciding whether an object is reachable or not.

在2012年，所有的现代浏览器都使用标记-清除垃圾收集器。过去几年，JavaScript垃圾收集（代数/增量/并行/并行垃圾收集）领域的所有改进都是对该算法（标记和扫描）的实现进行了改进，但并没有对垃圾收集算法本身的改进， 其目标是确定一个对象是否可达。

[In this article](https://en.wikipedia.org/wiki/Tracing_garbage_collection), you can read in a greater detail about tracing garbage collection that also covers mark-and-sweep along with its optimizations.

[在这篇文章中](https://en.wikipedia.org/wiki/Tracing_garbage_collection)，你可以鱼都到更多关于垃圾收集跟踪并且也覆盖到了关于标记-清除算法的优化。

### 循环不再是一个问题

In the first example above, after the function call returns, the two objects are not referenced anymore by something reachable from the global object. Consequently, they will be found unreachable by the garbage collector.

在上述的第一个例子中，在函数调用返回之后，这两个对象不能够被全局对象所访问。因此，垃圾收集器就会发现它们不能够被访问了。

![](https://cdn-images-1.medium.com/max/2048/1*FbbOG9mcqWZtNajjDO6SaA.png) 

Even though there are references between the objects, they’re not reachable from the root.

即使在这两个对象之间存在着引用，它们再也不能从root访问了。

### 列举垃圾收集器的直观行为

Although Garbage Collectors are convenient they come with their own set of trade-offs. One of them is *non-determinism*. In other words, GCs are unpredictable. You can’t really tell when a collection will be performed. This means that in some cases programs use more memory that it’s actually required. In other cases, short-pauses may be noticeable in particularly sensitive applications. Although non-determinism means one cannot be certain when a collection will be performed, most GC implementations share the common pattern of doing collection passes during allocation. If no allocations are performed, most GCs stay idle. Consider the following scenario:

虽然垃圾收集器很方便，但它们自己也有自己的代价。 其中一个是非确定论。 换句话说，GC是不可预测的。 你不能真正地告诉你什么时候会收集。 这意味着在某些情况下，程序会使用实际需要的更多内存。 在其他情况下，特别敏感的应用程序可能会引起短暂暂停。 虽然非确定性意味着在执行集合时无法确定，但大多数GC实现共享在分配期间执行收集遍历的常见模式。 如果没有执行分配，大多数GC保持空闲状态。 考虑以下情况：

1. 执行相当大的一组分配。
2. 这些元素中的大多数（或全部）被标记为不可访问（假设我们将指向我们不再需要的缓存的引用置空）。
3. 不再执行分配。



    1. A sizable set of allocations is performed.

    2. Most of these elements (or all of them) are marked as unreachable (suppose we null a reference pointing to a cache we no longer need).

    3. No further allocations are performed.
In this scenario, most GCs will not run any further collection passes. In other words, even though there are unreachable references available for collection, these are not claimed by the collector. These are not strictly leaks but still, result in higher-than-usual memory usage.

在这种情况下，大多数GC不会再运行收集处理。换句话说，即使存在对于收集器来说不可访问的引用，它们也不会被收集器所认领。严格意义来说这并不是泄露，但是依然会导致比平常更多的内存使用。

### 什么是内存泄露？

In essence, memory leaks can be defined as memory that is not required by the application anymore but for some reason is not returned to the operating system or the pool of free memory.

实质上，内存泄漏可以被定义为应用程序不再需要的内存，但是由于某些原因不会返回到操作系统或可用内存池。

![](https://cdn-images-1.medium.com/max/2000/1*0B-dAUOH7NrcCDP6GhKHQw.jpeg) 

Programming languages favor different ways of managing memory. However, whether a certain piece of memory is used or not is actually an [undecidable problem](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Memory_Management#Release_when_the_memory_is_not_needed_anymore). In other words, only developers can make it clear whether a piece of memory can be returned to the operating system or not.

编程语言有支持管理内存的不同方法。 然而，某块内存是否被使用实际上是一个[不可判定的问题](ttps://developer.mozilla.org/en-US/docs/Web/JavaScript/Memory_Management#Release_when_the_memory_is_not_needed_anymore)。 换句话说，只有开发人员可以清楚一个内存是否可以返回到操作系统。

Certain programming languages provide features that help developers do this. Others expect developers to be completely explicit about when a piece of memory is unused. Wikipedia has good articles on [manual](https://en.wikipedia.org/wiki/Manual_memory_management) and [automatic](https://en.wikipedia.org/wiki/Manual_memory_management) memory management.

某些编程语言提供了帮助开发者执行此操作的功能。其他的则期望开发人员能够完全明确何时使用一块内存。 维基百科有关于[手动](https://en.wikipedia.org/wiki/Manual_memory_management)和[自动](https://en.wikipedia.org/wiki/Manual_memory_management)内存管理的好文章。

### 四种常见的JavaScript泄露
### 1: 全局变量

JavaScript handles undeclared variables in an interesting way: a reference to an undeclared variable creates a new variable inside the *global* object. In the case of browsers, the global object is window. In other words:

JavaScript使用一种有趣的方式处理未声明的变量：一个未声明变量的引用会在*全局*对象内部产生一个新的变量。在浏览器的情况，这个全局变量就会是window。换句话说：

    function foo(arg) {
        bar = "some text";
    }

等同于：

    function foo(arg) {
        window.bar = "some text";
    }

If bar was supposed to hold a reference to a variable only inside the scope of the foo function and you forget to use var to declare it, an unexpected global variable is created.

如果bar被期望仅仅在foo函数作用域内保持对变量的引用，并且你忘记使用var去声明它，一个意想不到的全局变量就产生了。

In this example, leaking a simple string won't do much harm, but it could certainly be worse.

在这个例子中，泄露就仅仅是一个字符串并不会带来太多危害，但是它可能会变得更糟。

Another way in which an accidental global variable can be created is through this:

另外一种可能产生意外的全局变量的方式是：

    function foo() {
        this.var1 = "potential accidental global";
    }
    
    // Foo called on its own, this points to the global object (window)
    // rather than being undefined.
    foo();
>  To prevent these mistakes from happening, add 'use strict'; at the beginning of your JavaScript files. This enables a stricter mode of parsing JavaScript that prevents accidental global variables. [Learn more](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode) about this mode of JavaScript execution.
>
>  为了阻止这些错误的发生，可以在js文件头部添加'use strict'。这将会使用严格模式来解析JavaScript从而阻止意外的全局变量。[了解更多](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode)关于JavaScript执行的模式。

Even though we talk about unsuspected globals, it’s still the case that much code is filled with explicit global variables. These are by definition non-collectible (unless assigned as null or reassigned). In particular, global variables that are used to temporarily store and process big amounts of information are of concern. If you must use a global variable to store lots of data, make sure to** assign it as null or reassign it** after you are done with it.

即使我们讨论了未预期的全局变量，但仍然有很多代码用显式的全局变量填充。 这些定义是不可收集的（除非分配为null或重新分配）。 特别是，用于临时存储和处理大量信息的全局变量值得关注。 如果您必须使用全局变量来存储大量数据，请确保在完成之后**将其分配为null或重新分配**。

### 2: 被遗忘的计时器和回调

The use of setInterval is quite common in JavaScript.

setInterval 在JavaScript中是经常被使用的。

Most libraries, that provide observers and other facilities that take callbacks, take care of making any references to the callback unreachable after their own instances become unreachable as well. In the case of setInterval, however, code like this is quite common:

    var serverData = loadData();
    setInterval(function() {
        var renderer = document.getElementById('renderer');
        if(renderer) {
            renderer.innerHTML = JSON.stringify(serverData);
        }
    }, 5000); //This will be executed every ~5 seconds.

This example illustrates what can happen with timers: timers that make reference to nodes or data that is no longer required.

The object represented by renderer may be removed in the future, making the whole block inside the interval handler unnecessary. However, the handler cannot be collected as the interval is still active, (the interval needs to be stopped for this to happen). If the interval handler cannot be collected, its dependencies cannot be collected either. This means that serverData, which presumably stores quite a big amount of data, cannot be collected either.

In the case of observers, it is important to make explicit calls to remove them once they are not needed anymore (or the associated object is about to be made unreachable).

In the past, this used to be particularly important as certain browsers (the good old IE 6) were not able to manage well cyclic references (see below for more info). Nowadays, most browsers can and will collect observer handlers once the observed object becomes unreachable, even if the listener is not explicitly removed. It remains good practice, however, to explicitly remove these observers before the object is disposed of. For instance:

    var element = document.getElementById('launch-button');
    var counter = 0;
    
    function onClick(event) {
       counter++;
       element.innerHtml = 'text ' + counter;
    }
    
    element.addEventListener('click', onClick);
    
    // Do stuff
    
    element.removeEventListener('click', onClick);
    element.parentNode.removeChild(element);
    
    // Now when element goes out of scope,
    // both element and onClick will be collected even in old browsers // that don't handle cycles well.

Nowadays, modern browsers (including Internet Explorer and Microsoft Edge) use modern garbage collection algorithms that can detect these cycles and deal with them correctly. In other words, it’s not strictly necessary to call removeEventListener before making a node unreachable.

Frameworks and libraries such as *jQuery* do remove listeners before disposing of a node (when using their specific APIs for that). This is handled internally by the libraries which also make sure that no leaks are produced, even when running under problematic browsers such as … yeah, IE 6.
### 3: Closures

A key aspect of JavaScript development are closures: an inner function that has access to the outer (enclosing) function’s variables. Due to the implementation details of the JavaScript runtime, it is possible to leak memory in the following way:

    var theThing = null;

    var replaceThing = function () {

      var originalThing = theThing;
      var unused = function () {
        if (originalThing) // a reference to 'originalThing'
          console.log("hi");
      };
    
      theThing = {
        longStr: new Array(1000000).join('*'),
        someMethod: function () {
          console.log("message");
        }
      };
    };
    
    setInterval(replaceThing, 1000);

This snippet does one thing: every time replaceThing is called, theThing gets a new object which contains a big array and a new closure (someMethod). At the same time, the variable unused holds a closure that has a reference to originalThing (theThing from the previous call to replaceThing). Already somewhat confusing, huh? The important thing is that **once a scope is created for closures that are in the same parent scope, that scope is shared**.

In this case, the scope created for the closure someMethod is shared with unused. unused has a reference to originalThing. Even though unused is never used, someMethod can be used through theThing outside of the scope of replaceThing (e.g. somewhere globally). And as someMethod shares the closure scope with unused, the reference unused has to originalThing forces it to stay active (the whole shared scope between the two closures). This prevents its collection.

When this snippet is run repeatedly a steady increase in memory usage can be observed. This does not get smaller when the GC runs. In essence, a linked list of closures is created (with its root in the form of the theThing variable), and each of these closures' scopes carries an indirect reference to the big array, resulting in a sizable leak.

This issue was found by the Meteor team and [they have a great article](https://blog.meteor.com/an-interesting-kind-of-javascript-memory-leak-8b47d2e7f156) that describes the issue in great detail.
### 4: Out of DOM references

Sometimes it may be useful to store DOM nodes inside data structures. Suppose you want to rapidly update the contents of several rows in a table. It may make sense to store a reference to each DOM row in a dictionary or an array. When this happens, two references to the same DOM element are kept: one in the DOM tree and the other in the dictionary. If at some point in the future you decide to remove these rows, you need to make both references unreachable.

    var elements = {
        button: document.getElementById('button'),
        image: document.getElementById('image')
    };
    
    function doStuff() {
        image.src = 'http://example.com/image_name.png';
    }
    
    function removeImage() {
        // The image is a direct child of the body element.
        document.body.removeChild(document.getElementById('image'));
    
        // At this point, we still have a reference to #button in the
        //global elements object. In other words, the button element is
        //still in memory and cannot be collected by the GC.
    }

There’s an additional consideration that has to be taken into account when it comes to references to inner or leaf nodes inside a DOM tree. Say you keep a reference to a specific cell of a table (a <td> tag) in your JavaScript code. One day you decide to remove the table from the DOM but keep the reference to that cell. Intuitively one may suppose the GC will collect everything but that cell. In reality, this won’t happen: the cell is a child node of that table and children keep references to their parents. That is, the reference to the table cell from JavaScript code causes **the whole table to stay in memory**. Consider this carefully when keeping references to DOM elements.



