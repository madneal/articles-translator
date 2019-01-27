## JavaScript 是如何工作的: Service Workers, 它们的生命周期和使用案例

>原文：[How JavaScript works: Service Workers, their lifecycle and use cases](https://blog.sessionstack.com/how-javascript-works-service-workers-their-life-cycle-and-use-cases-52b19ad98b58)
>
>译者：[neal1991](https://github.com/neal1991)
>
>welcome to star my [articles-translator](https://github.com/neal1991/articles-translator/), providing you advanced articles translation. Any suggestion, please issue or contact [me](mailto:bing@stu.ecnu.edu.cn)
>
>LICENSE: [MIT](https://opensource.org/licenses/MIT)

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

你可能已经知道，[渐进式 Web 应用](https://developers.google.com/web/progressive-web-apps/)只会越来越受欢迎，因为它们旨在使 Web 应用用户体验更加流畅，创建原生应用程序般的体验，而不是浏览器的外观和感觉。

构建渐进式 Web 应用程序的主要要求之一是使其在网络和加载方面非常可靠 - 它能够用于不确定或不存在的网络条件。

在这篇文章中，我们将深入探讨 Service Workers：它们如何运作以及你应该关心什么。 最后，我们还列出了你应该利用的 Service Workers 的一些特性，并在 [SessionStack](https://www.sessionstack.com/) 中分享我们团队的经验。

### 概述

如果你希望理解关于 Service Workers 的一切，你应该阅读我们关于 [Web Workers](https://blog.sessionstack.com/how-javascript-works-the-building-blocks-of-web-workers-5-cases-when-you-should-use-them-a547c0757f6a) 的博客。

基本上，Service Worker 是一种 Web Worker，更特定的来说，它就像是一个 [Shared Worker](https://developer.mozilla.org/en-US/docs/Web/API/SharedWorker):

* Serivice Worker 运行在它自己的全局脚本上下文中
* 它不会和特定的 web 页面绑定
* 它不能访问 DOM

Service Worker API 令人兴奋的主要原因之一是它可以让你的网络应用程序支持离线体验，从而使开发人员能够完全控制流程。

### Service Worker 的生命周期

Service worker 的生命周期完全独立于你的 web 页面。它由以下几步组成：

* 下载
* 安装
* 激活

### 下载

这就是浏览器下载包含 Service Worker 的 js 文件的时候。

### 安装

要为你的 Web 应用程序安装 Service Worker，你必须先注册它，你可以在 JavaScript 代码中进行注册。 当注册Service Worker 时，它会提示浏览器在后台启动 Service Worker 安装步骤。

通过注册服务 Service Worker，你可以告诉浏览器你的 Service Worker JavaScript 文件在哪里。 我们来看下面的代码：

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

该代码检查当前环境中是否支持 Service Worker API。如果是，则注册 /sw.js Service Worker。

你可以在每次加载页面时调用 register() 方法而不用担心 - 浏览器会判断 Service Worker 是否已经注册，并且会正确处理。

register() 方法的一个重要细节是 Service Worker 文件的位置。在这种情况下，你可以看到 Service Worker 文件位于域的根目录。这意味着 Service Worker 的范围将是整个来源。换句话说，这个 Service Worker 将会收到这个域的所有东西的 fetch 事件（我们将在后面讨论）。如果我们在 /example/sw.js 注册 Service Worker 文件，那么 Service Worker 将只能看到 URL 以 /example /（即 /example/page1/，/example/page2/）开头的页面的 fetch 事件。

在安装阶段，最好加载和缓存一些静态资源。资源成功缓存后，Service Worker 安装完成。如果没有（加载失败） - Service Worker 将重试。一旦安装成功，你将知道静态资源位于缓存中。

关于注册是否需要在加载事件之后发生的问题。这不是必须的，但它是绝对推荐的。

为什么这样？让我们考虑用户第一次访问你的网络应用程序。目前还没有 Service Worker ，浏览器无法事先知道是否会有最终安装的 Service Worker 。如果安装了 Service Worker，则浏览器需要为这个额外的线程花费额外的CPU 和内存，否则浏览器将花费在渲染网页上。

最重要的是，如果你只是在你的页面上安装一个 Service Worker ，你可能会延迟加载和渲染的风险 - 而不是尽快让你的用户可以使用这个页面。

请注意，这仅在第一次访问页面时很重要。后续页面访问不受 Service Worker 安装的影响。一旦在第一次访问页面时激活 Service Worker ，它可以处理加载/缓存事件，以便随后访问你的 Web 应用程序。这一切都是有道理的，因为它需要准备好处理有限的网络连接。

### 激活

安装 Service Worker 之后，下一步将是其激活。 这一步是管理之前缓存的好机会。

一旦激活， Service Worker 将开始控制所有属于其范围的页面。 一个有趣的事实是：首次注册 Service Worker 的页面将不会被控制，直到该页面再次被加载。 一旦 Service Worker 处于控制之下，它将处于以下状态之一：

* 它将处理从页面发出网络请求或消息时发生的 fetch 和消息事件
* 为了节省内存而被终止

生命周期看起来是这个样子的：

![](https://cdn-images-1.medium.com/max/2000/1*mVOrpKC9pFTMg4EXPozoog.png)

### 在 Service Worker 中处理安装

在页面加速注册过程之后，让我们看看在 Service Worker 脚本中发生了什么，它通过向 Service Worker 实例添加事件侦听器来处理安装事件。

这些是安装事件处理时需要采取的步骤：

* 打开一个缓存
* 缓存我们的文件
* 确认所有请求的资源是否被缓存

下面是 Service Worker 中一个简单的安装过程：

```javascript
var CACHE_NAME = 'my-web-app-cache';
var urlsToCache = [
  '/',
  '/styles/main.css',
  '/scripts/app.js',
  '/scripts/lib.js'
];

self.addEventListener('install', function(event) {
  // event.waitUntil takes a promise to know how
  // long the installation takes, and whether it 
  // succeeded or not.
  event.waitUntil(
    caches.open(CACHE_NAME)
      .then(function(cache) {
        console.log('Opened cache');
        return cache.addAll(urlsToCache);
      })
  );
});
```

如果所有的文件都被成功地缓存，那么 service worker 就安装成功。如果**任一**文件下载失败，那么安装步骤就会失败。因此留意你放在这的文件。

处理安装事件完全是可选的并且你可以避免它，这样你就不需要执行这里的任何步骤。

### 在运行时缓存请求

这部分是真正需要处理的部分。你将在这看到请求如何被拦截并且返回创建的缓存（或者新创建的请求）。

在 Service Worker 成功安装之后 ，用户浏览其他的页面或者刷新当前页面， Service Worker 都会收到 fetch 事件。下面的例子展示了如何返回缓存的资源或者执行一个新的请求再缓存结果：

```javascript
self.addEventListener('fetch', function(event) {
  event.respondWith(
    // This method looks at the request and
    // finds any cached results from any of the
    // caches that the Service Worker has created.
    caches.match(event.request)
      .then(function(response) {
        // If a cache is hit, we can return thre response.
        if (response) {
          return response;
        }

        // Clone the request. A request is a stream and
        // can only be consumed once. Since we are consuming this
        // once by cache and once by the browser for fetch, we need
        // to clone the request.
        var fetchRequest = event.request.clone();
        
        // A cache hasn't been hit so we need to perform a fetch,
        // which makes a network request and returns the data if
        // anything can be retrieved from the network.
        return fetch(fetchRequest).then(
          function(response) {
            // Check if we received a valid response
            if(!response || response.status !== 200 || response.type !== 'basic') {
              return response;
            }

            // Cloning the response since it's a stream as well.
            // Because we want the browser to consume the response
            // as well as the cache consuming the response, we need
            // to clone it so we have two streams.
            var responseToCache = response.clone();

            caches.open(CACHE_NAME)
              .then(function(cache) {
                // Add the request to the cache for future queries.
                cache.put(event.request, responseToCache);
              });

            return response;
          }
        );
      })
    );
});
```

在 nutshell 中会发生：

* event.respondWith() 将会决定如何响应 fetch 事件。我们将会从 caches.match() 中传递一个 promise 监听请求并且查看缓存中是否存在命中。
* 如果缓存存在，那么就发送响应。
* 否则就会执行 fetch
* 检查状态是否是 200。我们也会检查响应的类型是基本的，这也表示请求是同源请求。对于第三方资源的请求在这种情况下就不会被缓存。
* 响应被添加到缓存中。

请求和响应必须被克隆因为它们是[流](https://streams.spec.whatwg.org/)。流的主体只能够被消费一次。并且一旦我们想消费它们，我们就想克隆它们因为浏览器必须消费它们。

### 更新 Service Worker

当一个用户访问你的 web 应用，浏览器将会尝试重新下载包含你的 Service Worker 的 js 文件。这会在后台进行。

如果现在下载的 Service Worker 文件和现在的 Service Worker 文件有一个字节的差异，这个浏览器就会假设已经发生了变化并且启动新的 Service Worker。

这个新的 Service Worker 就会被启动并且安装事件就会被触发。然而那个时间点，旧的 Service Worker 依然会控制你的 web 应用，这意味着新的 Service Worker 依然会进入一个等待的状态。

一旦现在关闭你之前打开的的 web 应用的页面，旧的 Service Worker 就会被浏览器中止并且新安装的 Service Worker 就会全部接管。这时 active 事件就会被触发。

为什么所有的都需要？为了避免同时在不同的标签页中运行两种版本的 web 应用--这种事的确经常发生在 web 中并且会产生非常糟糕的 bug。（比如：你在浏览器本地存储了不同结构的数据）

### 从缓存中删除数据

在激活回调中最常见的步骤就是缓存管理。你现在就想做这件事因为你打算将安装步骤中的旧缓存删除掉，旧的  Service Worker 就会突然停止为缓存中的文件提供服务。

下面的例子就是你如何从不是白名单的缓存中删除文件（这种情况，page-1 以及 page-2就在他们的）

```javascript
self.addEventListener('activate', function(event) {

  var cacheWhitelist = ['page-1', 'page-2'];

  event.waitUntil(
    // Retrieving all the keys from the cache.
    caches.keys().then(function(cacheNames) {
      return Promise.all(
        // Looping through all the cached files.
        cacheNames.map(function(cacheName) {
          // If the file in the cache is not in the whitelist
          // it should be deleted.
          if (cacheWhitelist.indexOf(cacheName) === -1) {
            return caches.delete(cacheName);
          }
        })
      );
    })
  );
});
```

### HTTPS 需求

当你在构建你的 web 应用的时候，你能够在 localhost 使用 Service Worker ，但是你一旦将其部署到生产环境，那么你必须准备好 HTTPS （并且这是你使用 HTTPS 最后的原因）。

通过 Service Worker，你可以劫持连接并且制作响应。如果不使用 HTTPS，你的 web 应用容易导致[中间人攻击](https://en.wikipedia.org/wiki/Man-in-the-middle_attack)。

出于安全考虑，你需要在使用 HTTPS的服务上注册 Service Worker，这样才能知道浏览器接收到的 Service Worker 请求没有在网络传输过程中被修改。

### 浏览器支持

对于 Service Worker 的浏览器支持也越来越好：

![](http://ozfo4jjxb.bkt.clouddn.com/browser_support.png)

你可以在这参考所有浏览器的进程— [https://jakearchibald.github.io/isserviceworkerready/](https://jakearchibald.github.io/isserviceworkerready/)。

### Service Workers 拥抱更好的特性

Service Worker 提供了一些很特别的特性：

服务

* **推送通知**— 允许用户选择从网络应用程序及时更新。

* **后台同步**— 允许直到用户具有稳定的连接的时候才执行操作。这样你就可以确定无论用户希望发送什么都会被发送。

* **周期性同步** (未来) — API 提供后来周期性同步管理功能。

* **地理围栏** (未来) — 你可以定义参数，也可以认为是**地理围栏**即感兴趣的地区。这个 web 应用就会在设备经过电子围栏的时候发送推送通知，这样就可以允许你基于用户的地理信息提供有意义的体验。

在这个系列中的后续博客中我们继续讨论这些特性的细节。

目前我们一直让 SessionStack 的 UX 变得更佳顺滑，优化页面加载和响应时间。

如果你在 [SessionStack](https://www.sessionstack.com) 重现一个用户 session （或者实时观察它），SessionStack 的前台将会持续从服务器获取数据，这样可以无缝为你创建一个类似缓存的用户体验。为了给你提供一些背景知识-一旦你将 SessionStack 的库集成到你的 web 应用，它将会持续收集数据，比如 DOM 改变，用户交互，网络请求，未处理的异常以及调试消息。

当一个 session 被重现或者实时流式展示，SessionStack 将会为所有的数据提供问题，这样就允许你可看到你的用户在浏览器中的所有体验（包括视觉上和技术上）。这个过程必须足够快，因为我们不希望用户等待。

由于数据是由我们的前端提取的，因此这是一个很好的地方，可以利用 Service Worker 来处理重新加载我们的播放器和再次流式传输等情况。 处理较慢的网络连接也非常重要。

这是一个免费的计划，你可以[尝试 SessionStack](https://www.sessionstack.com/signup/).

![](http://ozfo4jjxb.bkt.clouddn.com/sessionstack.png)

### 资源

* [https://developers.google.com/web/fundamentals/primers/service-workers/](https://developers.google.com/web/fundamentals/primers/service-workers/)

* [https://github.com/w3c/ServiceWorker/blob/master/explainer.md](https://github.com/w3c/ServiceWorker/blob/master/explainer.md)
