## How JavaScript works: Service Workers, their lifecycle and use cases

>原文：[How JavaScript works: Service Workers, their lifecycle and use cases](https://blog.sessionstack.com/how-javascript-works-service-workers-their-life-cycle-and-use-cases-52b19ad98b58)
>
>译者：[neal1991](https://github.com/neal1991)
>
>welcome to star my [articles-translator ](https://github.com/neal1991), providing you advanced articles translation. Any suggestion, please issue or contact [me](mailto:bing@stu.ecnu.edu.cn)
>
>LICENSE: [MIT](https://opensource.org/licenses/MIT)

This is post # 8 of the series dedicated to exploring JavaScript and its building components. In the process of identifying and describing the core elements, we also share some best practice we use when building [SessionStack](https://www.sessionstack.com/?utm_source=medium&utm_medium=source&utm_content=javascript-series-web-workers-intro), a JavaScript application that has to be robust and highly-performant in order to show you real-time exactly how your users ran into a technical or UX issue in your web app.

If you missed the previous chapters, you can find them here:

这是专门探索 JavaScript 及其构建组件的系列 #11。 在识别和描述核心元素的过程中，我们也分享了我们在构建[SessionStack](https://www.sessionstack.com/?utm_source=medium&utm_medium=blog&utm_content=js-series-rendering-engine-intro) 时使用的一些经验法则，SessionStack 是一款 JavaScript 应用程序，利用强大且高性能的特性来帮助用户实时查看和重现其 Web 应用程序缺陷。

如果你错过了之前的章节，你可以从这找到他们：

* [JavaScript是如何工作的：引擎，运行时以及调用栈的概述](https://github.com/neal1991/articles-translator/blob/master/JavaScript%E6%98%AF%E5%A6%82%E4%BD%95%E5%B7%A5%E4%BD%9C%EF%BC%9A%E7%B3%BB%E5%88%97%E4%B8%80.md)（已翻译）

* [Inside Google’s V8 engine + 5 tips on how to write optimized code](https://blog.sessionstack.com/how-javascript-works-inside-the-v8-engine-5-tips-on-how-to-write-optimized-code-ac089e62b12e?source=collection_home---2------2----------------)

* [JavaScript是如何工作的：内存管理以及如何处理四种常见的内存泄漏](https://github.com/neal1991/articles-translator/blob/master/JavaScript%E6%98%AF%E5%A6%82%E4%BD%95%E5%B7%A5%E4%BD%9C%E7%9A%84%EF%BC%9A%E7%B3%BB%E5%88%97%E4%B8%89.md)（已翻译）

* [The event loop and the rise of Async programming + 5 ways to better coding with async/await](https://blog.sessionstack.com/how-javascript-works-event-loop-and-the-rise-of-async-programming-5-ways-to-better-coding-with-2f077c4438b5)

* [Deep dive into WebSockets and HTTP/2 with SSE + how to pick the right path](https://blog.sessionstack.com/how-javascript-works-deep-dive-into-websockets-and-http-2-with-sse-how-to-pick-the-right-path-584e6b8e3bf7?source=collection_home---4------0----------------)

* [A comparison with WebAssembly + why in certain cases it’s better to use it over JavaScript](https://blog.sessionstack.com/how-javascript-works-a-comparison-with-webassembly-why-in-certain-cases-its-better-to-use-it-d80945172d79)

* [The building blocks of Web Workers + 5 cases when you should use them](https://blog.sessionstack.com/how-javascript-works-the-building-blocks-of-web-workers-5-cases-when-you-should-use-them-a547c0757f6a)

![](https://cdn-images-1.medium.com/max/4872/1*oOcY2Gn-LVt1h-e9xOv5oA.jpeg)

You probably already know that [Progressive Web Apps](https://developers.google.com/web/progressive-web-apps/) will only be getting more popular as they aim at making web app user experience smoother, at creating a native app-like experiences rather than browser look and feel.

One of the main requirements to build a Progressive Web App is to make it very reliable in terms of network and loading — it should be usable in uncertain or non-existent network conditions.

In this post, we’ll be deep diving into Service Workers: how they function and what you should care about. At the end, we also list a few unique benefits of the Service Workers that you should take advantage of, and share our own team’s experience here at [SessionStack](https://www.sessionstack.com/).

你可能已经知道，[渐进式 Web 应用](https://developers.google.com/web/progressive-web-apps/)只会越来越受欢迎，因为它们旨在使 Web 应用用户体验更加流畅，创建原生应用程序般的体验，而不是浏览器的外观和感觉。

构建渐进式 Web 应用程序的主要要求之一是使其在网络和加载方面非常可靠 - 它应该可用于不确定或不存在的网络条件。

在这篇文章中，我们将深入探讨 Service Workers：他们如何运作以及你应该关心什么。 最后，我们还列出了你应该利用的 Service Workers 的一些独特优势，并在 [SessionStack](https://www.sessionstack.com/) 中分享我们自己团队的经验。

### 概述

If you want to understand everything about Service Workers, you should start by reading our blog post on [Web Workers](https://blog.sessionstack.com/how-javascript-works-the-building-blocks-of-web-workers-5-cases-when-you-should-use-them-a547c0757f6a).

如果你希望理解关于 Service Workers 的一起，你应该阅读我们关于 [Web Workers](https://blog.sessionstack.com/how-javascript-works-the-building-blocks-of-web-workers-5-cases-when-you-should-use-them-a547c0757f6a) 的博客。

Basically, the Service Worker is a type of Web Worker, and more specifically it’s like a [Shared Worker](https://developer.mozilla.org/en-US/docs/Web/API/SharedWorker):

基本上，Service Worker 是一种 Web Worker，更特定的来说，他就像是一个 [Shared Worker](https://developer.mozilla.org/en-US/docs/Web/API/SharedWorker):

* The Service Worker runs in its own global script context Serivice Worker 运行在它自己的全局脚本上下文中

* It isn’t tied to a specific web page它不会和特定的 web 页面绑定

* It cannot access the DOM 它不能访问 DOM

One of the main reasons why the Service Worker API is so exciting is that it allows your web apps to support offline experiences, giving developers complete control over the flow.

Service Worker API 令人兴奋的主要原因之一是它可以让你的网络应用程序支持离线体验，从而使开发人员能够完全控制流程。

### Service Worker 的生命周期

The lifecycle of a service worker is completely separated from your web page one. It consists of the following phases:

Service worker 的生命周期完全独立于你的 web 页面。它由以下几步组成：

* 下载

* 安装

* 激活

### 下载

This is when the browser downloads the .js file which contains the Service Worker.

这就是浏览器下载包含 Service Worker 的 js 文件的时候。

### 安装

To install a Service Worker for your web app, you have to register it first, which you can do in your JavaScript code. When a Service Worker is registered, it prompts the browser to start a Service Worker install step in the background.

By registering the Service Worker, you tell the browser where your Service Worker JavaScript file lives. Let’s look at the following code:

要为你的 Web 应用程序安装 Service Worker，你必须先注册它，你可以在 JavaScript 代码中进行注册。 当注册Service Worker 时，它会提示浏览器在后台启动 Service Worker 安装步骤。

通过注册服务工作者，你可以告诉浏览器你的服务工作者JavaScript文件在哪里。 我们来看下面的代码：

```javascript
if ('serviceWorker' in navigator) {
  window.addEventListener('load', function() {
    navigator.serviceWorker.register('/sw.js').then(function(registration) {
      // Registration was successful
      console.log('ServiceWorker registration successful');
    }, function(err) {
      // Registration failed
      console.log('ServiceWorker registration failed: ', err);
    });
  });
}
```

The code checks whether the Service Worker API is supported in the current environment. If it is, the /sw.js Service Worker is registered.

You can call the register() method every time a page loads with no concern — the browser will figure out if the service worker has already been registered, and will handle it properly.

An important detail of the register() method is the location of the service worker file. In this case you can see that the service worker file is at the root of the domain. This means that the service worker's scope will be the entire origin. In other words, this service worker will receive fetch events (which we’ll discuss later) for everything on this domain. If we register the service worker file at /example/sw.js, then the service worker would only see fetch events for pages which URLs start with /example/ (i.e. /example/page1/, /example/page2/).

During the installation phase, it’s best to load and cache some static assets. Once the assets are successfully cached, the Service Worker installation is complete. If not (the loading fails) — the Service Worker will do a retry. Once installed successfully, you’ll know that the static assets are in the cache.

This answers your question if registration need to happen after the load event. It’s not a must, but it’s definitely recommended.

Why so? Let’s consider a user’s first visit to your web app. There’s no service worker yet, and the browser has no way of knowing in advance whether there will be a service worker that will eventually be installed. If the Service Worker gets installed, the browser will need to spend extra CPU and memory for this additional thread which otherwise the browser will spend on rendering the web page instead.

The bottom line is that , if you just install a Service Worker on your page, you’re running the risk of delaying the loading and rendering — not making the page available to your users as quickly as possible.

Note that this is important only for the first page visit. Subsequent page visits don’t get impacted by the Service Worker installation. Once a Service Worker is activated on a first page visit, it can handle loading/caching events for subsequent visits to your web app. This all makes sense, because it needs to be ready to handle limited network connectivity.

该代码检查当前环境中是否支持 Service Worker API。如果是，则注册 /sw.js Service Worker。

你可以在每次加载页面时调用 register() 方法而不用担心 - 浏览器会判断 Service Worker 是否已经注册，并且会正确处理。

register() 方法的一个重要细节是 Service Worker 文件的位置。在这种情况下，你可以看到 Service Worker 文件位于域的根目录。这意味着 Service Worker 的范围将是整个来源。换句话说，这个 Service Worker 将会收到这个域的所有东西的 fetch 事件（我们将在后面讨论）。如果我们在 /example/sw.js 注册 Service Worker 文件，那么 Service Worker 将只能看到 URL 以 /example /（即 /example/page1/，/example/page2/）开头的页面的 fetch 事件。

在安装阶段，最好加载和缓存一些静态资源。资源成功缓存后，Service Worker 安装完成。如果没有（加载失败） - Service Worker 将重试。一旦安装成功，你将知道静态资源位于缓存中。

关于注册是否需要在加载事件之后发生的问题。这不是必须的，但它是绝对推荐的。

为什么这样？让我们考虑用户第一次访问你的网络应用程序。目前还没有 Service Worker ，浏览器无法事先知道是否会有最终安装的 Service Worker 。如果安装了 Service Worker，则浏览器需要为这个额外的线程花费额外的CPU 和内存，否则浏览器将花费在渲染网页上。

最重要的是，如果你只是在你的页面上安装一个 Service Worker ，你可能会延迟加载和渲染的风险 - 而不是尽快让你的用户可以使用这个页面。

请注意，这仅在第一次访问页面时很重要。后续页面访问不受 Service Worker 安装的影响。一旦在第一次访问页面时激活 Service Worker ，它可以处理加载/缓存事件，以便随后访问你的 Web 应用程序。这一切都是有道理的，因为它需要准备好处理有限的网络连接。

### 激活

After the Service Worker is installed, the next step will be its activation. This step is a great opportunity to manage previous caches.

Once activated, the Service Worker will start controlling all pages that fall under its scope. An interesting fact: the page that registered the Service Worker for the first time won’t be controlled until that page is loaded again. Once the Service Worker is in control, it will be in one of the following states:

安装 Service Worker 之后，下一步将是其激活。 这一步是管理之前缓存的好机会。

一旦激活， Service Worker 将开始控制所有属于其范围的页面。 一个有趣的事实是：首次注册 Service Worker 的页面将不会被控制，直到该页面再次被加载。 一旦 Service Worker 处于控制之下，它将处于以下状态之一：

* It will handle fetch and message events that occur when a network request or message is made from the page它将处理从页面发出网络请求或消息时发生的 fetch 和消息事件

* It will be terminated to save memory 为了节省内存而被终止

Here is how the lifecycle will look like:

生命周期看起来是这个样子的：

![](https://cdn-images-1.medium.com/max/2000/1*mVOrpKC9pFTMg4EXPozoog.png)

### 在 Service Worker 中处理安装

After a page spins up the registration process, let’s see what happens inside the Service Worker script, which handles the install event by adding an event listener to the Service Worker instance.

Those are the steps that need to be taken when the install event is handled:

在页面加速注册过程之后，让我们看看在 Service Worker 脚本中发生了什么，它通过向 Service Worker 实例添加事件侦听器来处理安装事件。

这些是安装事件处理时需要采取的步骤：

* 打开一个缓存

* 缓存我们的文件

* 确认所有请求的资源是否被缓存

Here is what a simple installation might look like inside a Service Worker:

下面是 Service Worker 中一个简单的安装过程：

 <iframe src="https://medium.com/media/8a4c5afca4a07c30b6c791be08bbdbc4" frameborder=0></iframe>

If all the files are successfully cached, then the service worker will be installed. If **any** of the files fail to download, then the install step will fail. So be careful what files you put there.

如果所有的文件都被成功地缓存，那么 service worker 就安装成功。如果**任一**文件下载失败，那么安装步骤就会失败。因此留意你放在这的文件。

Handling the install event is completely optional and you can avoid it, in which case you don’t need to perform any of the steps here.

处理安装事件完全是可选的并且你可以避免它，这样你就不需要执行这里的任何步骤。

### 在运行时缓存请求

This part is the real-deal. This is where you’ll see how to intercept requests and return the created caches (and create new ones).

这部分是真正需要处理的部分。你将在这看到请求如何被拦截并且返回创建的缓存（或者新创建的请求）。

After a Service Worker is installed and the user navigates to another page or refreshes the page he’s on, the Service Worker will receive fetch events. Here is an example that demonstrates how to return cached assets or perform a new request and then cache the result:

在 Service Worker 成功安装之后 ，用户浏览其他的页面或者刷新当前页面， Service Worker 都会收到 fetch 事件。下面的例子展示了如何返回缓存的资源或者执行一个新的请求再缓存结果：

 <iframe src="https://medium.com/media/636ebc25f8c60b32f15d19ddfafb8736" frameborder=0></iframe>

Here is what happens in a nutshell:

在 nutshell 中会发生：

* The event.respondWith() will determine how we’ll respond to the fetch event. We pass a promise from caches.match() which looks at the request and finds if there are any cached results from any of the caches that have been created. event.respondWith() 将会决定如何响应 fetch 事件。我们将会从 caches.match() 中传递一个 promise 监听请求并且查看缓存中是否存在命中。

* If there is a cache, the response is retrieved.如果缓存存在，那么就发送响应。

* Otherwise, a fetch will be performed.

* Check if the status is 200. We also check that the response type is **basic, **which indicates that it’s a request from our origin. Requests to third party assets won’t be cached in this case.

* The response is added to the cache.

Requests and responses have to be cloned because they’re [streams](https://streams.spec.whatwg.org/). The body of a stream can be consumed only once. And since we want to consume them, we want to clone them because the browser has to consume them as well.

### Updating a Service Worker

When a user visits your web app, the browser tries to re-download the .js file that contains your Service Worker code. This takes place in the background.

If there is even a single byte difference in the Service Worker’s file that was downloaded now compared to the current Service Worker’s file, the browser will assume that there is a change and a new Service Worker has to be started.

The new Service Worker will be started and the install event will be fired. At this point, however, the old Service Worker is still controlling the pages of your web app which means that the new Service Worker will enter a waiting state.

Once the currently opened pages of your web app are closed, the old Service Worker will be killed by the browser and the newly-installed Service Worker will take full control. This is when its activate event will be fired.

Why is all this needed? To avoid the problem of having two versions of a web app running simultaneously , in different tabs — something that is actually very common on the web and can create really bad bugs (e.g. cases in which you have different schema while storing data locally in the browser).

### 从缓存中删除数据

The most common step in the activate callback is cache management. You’d want to do this now because if you were to wipe out any old caches in the install step, old Service Workers will suddenly stop being able to serve files from that cache.

在激活回调中最常见的步骤就是缓存管理。你现在就想做这件事因为你打算将安装步骤中的旧缓存删除掉，旧的  Service Worker 就会突然停止为缓存中的文件提供服务。

Here is an example how you can delete some files from the cache that are not whitelisted (in this case, having page-1 or page-2 under their names):

 <iframe src="https://medium.com/media/05d9fbb176b3902e930496d2bcbd53e7" frameborder=0></iframe>

### HTTPS 需求

When you’re building your web app, you’ll be able to use Service Workers through localhost, but once you deploy it in production, you need to have HTTPS ready (and that’s the last reason for you to have HTTPS).

当你在构建你的 web 应用的时候，你能够在 localhost 使用 Service Worker ，但是你一旦将其部署到生产环境，那么你必须准备好 HTTPS （并且这是你使用 HTTPS 最后的原因）。

Using a Service Worker, you can hijack connections and fabricate responses. By not using HTTPs, your web app becomes prone to a [man-in-the-middle attacks](https://en.wikipedia.org/wiki/Man-in-the-middle_attack).

通过 Service Worker，你可以劫持连接并且制作响应。如果不使用 HTTPS，你的 web 应用容易导致[中间人攻击](https://en.wikipedia.org/wiki/Man-in-the-middle_attack)。

To make things safer, you’re required to register Service Workers on pages that are served over HTTPS so that you know that the Service Worker which the browser receives, hasn’t been modified while traveling through the network.

出于安全考虑，你需要在使用 HTTPS的服务上注册 Service Worker，这样才能知道浏览器接收到的 Service Worker 请求没有在网络传输过程中被修改。

### 浏览器支持

The browser support for Service Workers is getting better:

对于 Service Worker 的浏览器支持也越来越好：

![](https://cdn-images-1.medium.com/max/NaN/1*6o2TRDmrJlS97vh1wEjLYw.png)

You can follow the progress of all the browsers here — [https://jakearchibald.github.io/isserviceworkerready/](https://jakearchibald.github.io/isserviceworkerready/).

### Service Workers are opening the doors for great features

Some unique features that a Service Worker provides are:

* **Push notifications **— allow users to opt-in to timely updates from web apps.

* **Background sync **— allows you to defer actions until the user has stable connectivity. This way, you can make sure that whatever the user wants to send, is actually sent.

* **Periodic sync** (future) — API that provides functionality for managing periodic background synchronization.

* **Geofencing** (future) — you can define parameters, also referred to as **geofences **which surround the areas of interest. The web app gets a notification when the device crosses a geofence, which allows you to provide useful experience based on the geography of the user.

Each of these will be discussed in detail in future blog posts in this series.

We’re constantly working on making the UX of SessionStack as smooth as possible, optimizing page loading and response times.

When you replay a user session in [SessionStack](https://www.sessionstack.com) (or watch it real-time), the SessionStack front-end will be constantly pulling data from our servers in order to seamlessly create a buffering-like experience for you. To give you a bit of background — once you integrate SessionStack’s library in your web app, it will be continuously collecting data such as DOM changes, user interactions, network requests, unhandled exceptions and debug messages.

When a session is being replayed or streamed real-time, SessionStack serves all the data allowing you to see everything that the user experienced in his own browser (both visually and technically). This all needs to take place real quick as we don’t want to make users wait.

Since data is pulled by our front-end, this is a great place where Service Workers can be leveraged to handle situations like reloading our player and having to stream everything once again. Handling slow network connectivity is also very important.

There is a free plan if you’d like to [give SessionStack a try](https://www.sessionstack.com/?utm_source=medium&utm_medium=source&utm_content=javascript-series-web-workers-try-now).

![](https://cdn-images-1.medium.com/max/NaN/1*YKYHB1gwcVKDgZtAEnJjMg.png)

### 资源

* [https://developers.google.com/web/fundamentals/primers/service-workers/](https://developers.google.com/web/fundamentals/primers/service-workers/)

* [https://github.com/w3c/ServiceWorker/blob/master/explainer.md](https://github.com/w3c/ServiceWorker/blob/master/explainer.md)
