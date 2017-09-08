# How JavaScript works: an overview of the engine, the runtime, and the call stack 
# JavaScript是如何工作的：引擎，运行时间以及回调的概述

> 原文：[How JavaScript works: an overview of the engine, the runtime, and the call stack](https://blog.sessionstack.com/how-does-javascript-actually-work-part-1-b0bacc073cf)
>
> 译者：[neal1991](https://github.com/neal1991)
>
> welcome to star my [articles-translator ](https://github.com/neal1991), providing you advanced articles translation. Any suggestion, please issue or contact [me](mailto:bing@stu.ecnu.edu.cn)
>
> LICENSE: [MIT](https://opensource.org/licenses/MIT)

As JavaScript is getting more and more popular, teams are leveraging its support on many levels in their stack - front-end, back-end, hybrid apps, embedded devices and much more.

随着JavaScript变得越来越流行，团队在多个层级都对它进行利用－前端，后端，混合应用，嵌入式设备以及更多。

As shown in the [GitHut stats](http://githut.info/), JavaScript is at the top in terms of Active Repositories and Total Pushes in GitHub. It doesn’t lag behind much in the other categories either.

正如[GitHut stats](http://githut.info/)所展示的那样，JavaScript是Github上面最活跃以及总Push次数最多的语言。
![](https://cdn-images-1.medium.com/max/3036/1*Zf4reZZJ9DCKsXf5CSXghg.png) 


([Check out up-to-date GitHub language stats](https://madnight.github.io/githut/)).

If projects are getting so much dependent on JavaScript, this means that developers have to be utilizing everything that the language and the ecosystem provide with deeper and deeper understanding of the internals, in order to build amazing software.

As it turns out, there are a lot of developers that are using JavaScript on a daily basis but don’t have the knowledge of what happens under the hood.
## Overview 


Almost everyone has already heard of the V8 Engine as a concept, and most people know that JavaScript is single-threaded or that it is using a callback queue.

In this post, we’ll go through all these concepts in detail and explain how JavaScript actually runs. By knowing these details, you’ll be able to write better, non-blocking apps that are properly leveraging the provided APIs.

If you’re relatively new to JavaScript, this blog post will help you understand why JavaScript is so “weird” compared to other languages.

And if you’re an experienced JavaScript developer, hopefully, it will give you some fresh insights on how the JavaScript Runtime you’re using every day actually works.
## **The JavaScript Engine** 


A popular example of a JavaScript Engine is Google’s V8 engine. The V8 engine is used inside Chrome and Node.js for example. Here is a very simplified view of what it looks like:
![](https://cdn-images-1.medium.com/max/2048/1*OnH_DlbNAPvB9KLxUCyMsA.png) 


The Engine consists of two main components:
* Memory Heap — this is where the memory allocation happens
* Call Stack — this is where your stack frames are as your code executes
## **The Runtime** 


There are APIs in the browser that have been used by almost any JavaScript developer out there (e.g. “setTimeout”). Those APIs, however, are not provided by the Engine.

So, where are they coming from?

It turns out that the reality is a bit more complicated.
![](https://cdn-images-1.medium.com/max/2048/1*4lHHyfEhVB0LnQ3HlhSs8g.png) 


So, we have the Engine but there is actually a lot more. We have those things called Web APIs which are provided by browsers, like the DOM, AJAX, setTimeout and much more.

And then, we have the so popular **event loop** and the **callback queue**.
## The Call Stack 


JavaScript is a single-threaded programming language, which means it has a single Call Stack. Therefore it can do one thing at a time.

The Call Stack is a data structure which records basically where in the program we are. If we step into a function, we put it on the top of the stack. If we return from a function, we pop off the top of the stack. That’s all the stack can do.

Let’s see an example. Take a look at the following code:

    function multiply(x, y) {
        return x * y;
    }
    
    function printSquare(x) {
        var s = multiply(x, x);
        console.log(s);
    }
    
    printSquare(5);

When the engine starts executing this code, the Call Stack will be empty. Afterwards, the steps will be the following:
![](https://cdn-images-1.medium.com/max/2048/1*Yp1KOt_UJ47HChmS9y7KXw.png) 


Each entry in the Call Stack is called a **Stack Frame**.

And this is exactly how stack traces are being constructed when an exception is being thrown — it is basically the state of the Call Stack when the exception happened. Take a look at the following code:

    function foo() {
        throw new Error('SessionStack will help you resolve crashes :)');
    }
    
    function bar() {
        foo();
    }
    
    function start() {
        bar();
    }
    
    start();

If this is executed in Chrome (assuming that this code is in a file called foo.js), the following stack trace will be produced:
![](https://cdn-images-1.medium.com/max/2000/1*T-W_ihvl-9rG4dn18kP3Qw.png) 


“**Blowing the stack**” — this happens when you reach the maximum Call Stack size. And that could happen quite easily, especially if you’re using recursion without testing your code very extensively. Take a look at this sample code:

    function foo() {
        foo();
    }
    
    foo();

When the engine starts executing this code, it starts with calling the function “foo”. This function, however, is recursive and starts calling itself without any termination conditions. So at every step of the execution, the same function gets added to the Call Stack over and over again. It looks something like this:
![](https://cdn-images-1.medium.com/max/2048/1*AycFMDy9tlDmNoc5LXd9-g.png) 


At some point, however, the number of function calls in the Call Stack exceeds the actual size of the Call Stack, and the browser decides to take action, by throwing an error, which can look something like this:
![](https://cdn-images-1.medium.com/max/2000/1*e0nEd59RPKz9coyY8FX-uw.png) 


Running code on a single thread can be quite easy since you don’t have to deal with complicated scenarios that are arising in multi-threaded environments — for example, deadlocks.

But running on a single thread is quite limiting as well. Since JavaScript has a single Call Stack, **what happens when things are slow?**
## **Concurrency & the Event Loop** 


What happens when you have function calls in the Call Stack that take a huge amount of time in order to be processed? For example, imagine that you want to do some complex image transformation with JavaScript in the browser.

You may ask — why is this even a problem? The problem is that while the Call Stack has functions to execute, the browser can’t actually do anything else — it’s getting blocked. This means that the browser can’t render, it can’t run any other code, it’s just stuck. And this creates problems if you want nice fluid UIs in your app.

And that’s not the only problem. Once your browser starts processing so many tasks in the Call Stack, it may stop being responsive for quite a long time. And most browsers take action by raising an error, asking you whether you want to terminate the web page.
![](https://cdn-images-1.medium.com/max/2000/1*WlMXK3rs_scqKTRV41au7g.jpeg) 


Now, that’s not the best user experience out there, is it?

So, how can we execute heavy code without blocking the UI and making the browser unresponsive? Well, the solution is **asynchronous callbacks**.

This will be explained in great detail in **Part 2** of the “How JavaScript actually work” tutorial. Stay tuned :)

In the meantime, if you’re having a hard time reproducing and understanding issues in your JavaScript apps, take a look at [SessionStack](https://www.sessionstack.com). SessionStack records everything in your web apps: all DOM changes, user interactions, JavaScript exceptions, stack traces, failed network requests, and debug messages. 
With SessionStack, you can replay issues in your web apps as videos and see everything that happened to your user.
There is a free plan that allows you to [get started for free](https://www.sessionstack.com/signup/).
![](https://cdn-images-1.medium.com/max/2062/1*kEQmoMuNBDfZKNSBh0tvRA.png) 
