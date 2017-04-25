# Service worker介绍

> 原文：[Twitter Lite and High Performance React Progressive Web Apps at Scale](https://medium.com/@paularmstrong/twitter-lite-and-high-performance-react-progressive-web-apps-at-scale-d28a00e780a3)
>
> 译者：[neal1991](https://github.com/neal1991)
>
> welcome to star my [articles-translator ](https://github.com/neal1991), providing you advanced articles translation. Any suggestion, please issue or contact [me](mailto:bing@stu.ecnu.edu.cn)
>
> LICENSE: [MIT](https://opensource.org/licenses/MIT)

## 那么它是什么？

Service worker正是被开发用于解决web平台上经常出现的问题和疑虑，包括：

* 无法解释（[Extensible Web Manifesto](https://extensiblewebmanifesto.org/) 中）的HTTP缓存以及高级HTTP交互比如HTML5 AppCache。
* 难以自然地构建一个离线优先地web应用。
* 缺乏可以利用很多提出功能的上下文执行。


We also note that the long lineage of declarative-only solutions ([Google Gears](https://gears.google.com), [Dojo Offline](http://www.sitepen.com/blog/category/dojo-offline/), and [HTML5 AppCache](http://alistapart.com/article/application-cache-is-a-douchebag)) have failed to deliver on their promise. Each successive declarative-only approach failed in many of the same ways, so the service worker effort has taken a different design approach: a largely-imperative system that puts developers firmly in control.

我们也注意到了声明解决方案([Google Gears](https://gears.google.com), [Dojo Offline](http://www.sitepen.com/blog/category/dojo-offline/)以及[HTML5 AppCache](http://alistapart.com/article/application-cache-is-a-douchebag)都没能实现他们的承诺。每个连续的仅有声明的方法都以相同的方式失败了，所以service worker采取了一个不同的设计方法：一个可以用开发者牢牢把控的重要系统：

Service worker就好像它的内部有一个有一个[shared worker](https://html.spec.whatwg.org/multipage/workers.html#sharedworker) ：

* 在它自己的全局脚本上下文中运行（通常是在它自己的线程中）
* 不会和特定的页面绑定
* 不能够访问DOM

不像shared worker，它：

* 即使没有页面也能够运行
* 如果不使用的话可以终止，还可以再次运行当需要的时候（比如，他不是事件驱动的）
* 拥有一个定义的升级模式
* 只允许HTTPS（更多的是在这一点上）

We can use service workers:

* To make sites work [faster and/or offline](https://www.youtube.com/watch?v=px-J9Ghvcx4) using network intercepting
* As a basis for other ‘background’ features such as [push messaging](http://updates.html5rocks.com/2015/03/push-notificatons-on-the-open-web) and [background synchronization](https://github.com/slightlyoff/BackgroundSync/blob/master/explainer.md)

我们可以利用service workers：

* 利用网络拦截可以让让网站[更快以及/或者支持离线使用](https://www.youtube.com/watch?v=px-J9Ghvcx4)
* 作为其它'background'功能的基础比如[消息推送](http://updates.html5rocks.com/2015/03/push-notificatons-on-the-open-web)以及[后台同步](https://github.com/slightlyoff/BackgroundSync/blob/master/explainer.md)

## 开始

First you need to register for a service worker:

首先你需要注册一个service worker:

```js
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/my-app/sw.js').then(function(reg) {
    console.log('Yey!', reg);
  }).catch(function(err) {
    console.log('Boo!', err);
  });
}
```

In this example, `/my-app/sw.js` is the location of the service worker script, and it controls pages whose URL begins with `/my-app/`.

在这个例子中，`/my-app/sw.js`就是service worker脚本的位置，并且它控制那些页面的URL以`/my-app/`开头。

`.register`返回一个promise。如果你以前没接触过promise的话，可以看看[HTML5Rocks article](http://www.html5rocks.com/en/tutorials/es6/promises/)。

`.register` returns a promise. If you’re new to promises, check out the [HTML5Rocks article](http://www.html5rocks.com/en/tutorials/es6/promises/).

Some restrictions:

一些限制：

* 注册页面必须安全地提供（没有证书错误的HTTPS）
* service worker和页面必须同源，尽管你可使用 [`importScripts`](https://html.spec.whatwg.org/multipage/workers.html#apis-available-to-workers:dom-workerglobalscope-importscripts)去导入其它地方的脚本


* 作为必须的范围

### 只有你说HTTPS？

使用service worker，你可以劫持请求，进行不同的响应，并且过滤响应。这些功能都很强大。尽管你可以将这些能力用在好的地方，但是中间人可能不会。为了避免这一点，你只能在HTTPS上提供的页面上注册service worker，所以我们知道浏览器接收的service worker没有在网络种没有被篡改。

Github Pages是由HTTPS提供服务的，所以是一个绝佳的展示demo的地方。

## 初始生命周期

当你调用`.register`之后，你的service worker会经历三个阶段

1. Download
2. Install
3. Activate

你可以使用事件和`install`以及`activate`进行交互：

```js
self.addEventListener('install', function(event) {
  event.waitUntil(
    fetchStuffAndInitDatabases()
  );
});

self.addEventListener('activate', function(event) {
  // You're good to go!
});
```

你可以向`event.waitUntill`传递一个promise从而来继承这个过程。一旦`activate`事件被触发了，你的service worker就可以控制页面了！

## 那么我现在可以控制页面了？

额，不完全是。当documen浏览时，它会选择一个service worker作为它的控制器，因此你使用`.register`注册的document并不是被控制的，因为那并不是service worker首次加载的地方。

If you refresh the document, it’ll be under the service worker’s control. You can check `navigator.serviceWorker.controller` to see which service worker is in control, or `null` if there isn’t one. Note: when you’re updating from one service worker to another, things work a little differently. We’ll get into that in the “Updating” section.

如果你刷新document，它将会是在service worker的控制之下。你可以通过`navigator.serviceWorker.controller`来看一下是哪个service worker在进行控制，如果没有的话结果就会是`null`。

注意：当你从一个service worker更新到另外一个的时候，可能会有一点点不一点。我们会进入“Updating"阶段。

If you shift+reload a document, it’ll always load without a controller, which is handy for testing quick CSS & JS changes.

如果使用shift来重载网页的话，加载就会有控制器了，这样做是为了测试CSS以及JS变化。

Document通常是和一个service worker存在于整个声明周期，或者根本就没有service worker。然而，service worker可以调用`self.skipWaiting()`([spec](https://w3c.github.io/ServiceWorker/#service-worker-global-scope-skipwaiting)) 来立刻接管范围内的所有页面。

## 网络截获

```js
self.addEventListener('fetch', function(event) {
  console.log(event.request);
});
```

You get fetch events for:

你可以利用fetch事件：

* 在你的service worker作用域下浏览
* 任何被这些页面触发的请求，甚至是对其他源的请求

这意味着你可以监听所有对于这个页面的请求，CSS,JS，图片，XHR，图标等等所有。

* iframes & `<object>`s – these will pick their own controller based on their resource URL
* Service workers – requests to fetch/update a service worker don’t go through the service worker
* Requests triggered within a service worker – you’d get a loop otherwise

The `request` object gives you information about the request such as its URL, method & headers. But the really fun bit, is you can hijack it and respond differently:

```js
self.addEventListener('fetch', function(event) {
  event.respondWith(new Response("Hello world!"));
});
```

[Here’s a live demo](https://jakearchibald.github.io/isserviceworkerready/demos/manual-response/).

`.respondWith` takes a `Response` object or a promise that resolves to one. We’re creating a manual response above. The `Response` object comes from the [Fetch Spec](https://fetch.spec.whatwg.org/#response-class). Also in the spec is the `fetch()` method, which returns a promise for a response, meaning you can get your response from elsewhere:

```js
self.addEventListener('fetch', function(event) {
  if (/\.jpg$/.test(event.request.url)) {
    event.respondWith(
      fetch('//www.google.co.uk/logos/…3-hp.gif', {
        mode: 'no-cors'
      })
    );
  }
});
```

In the above, I’m capturing requests that end in `.jpg` and instead responding with a Google doodle. `fetch()` requests are [CORS](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing) by default, but by setting `no-cors` I can use the response even if it doesn’t have CORS access headers (although I can’t access the content with JavaScript). [Here’s a demo of that](https://jakearchibald.github.io/isserviceworkerready/demos/img-rewrite/).

Promises let you fall back from one method to another:

```js
self.addEventListener('fetch', function(event) {
  event.respondWith(
    fetch(event.request).catch(function() {
      return new Response("Request failed!");
    })
  );
});
```

The service worker comes with a cache API, making it easy to store responses for reuse later. More on that shortly, but first…

## Updating a service worker

The lifecycle of a service worker is based on Chrome’s update model: do as much as possible in the background, don’t disrupt the user, complete the update when the current version closes.

Whenever you navigate to a page within scope of your service worker, the browser checks for updates in the background. If the script is byte-different, it’s considered to be a new version, and installed (note: only the script is checked, not external `importScripts`). However, the old version remains in control over pages until all tabs using it are gone (unless `.replace()` is called during install). Then the old version is garbage collected and the new version takes over.

This avoids the problem of two versions of a site running at the same time, in different tabs. Our current strategy for this is [“cross fingers, hope it doesn’t happen”](https://twitter.com/jaffathecake/status/502779501936652289).

Note: Updates obey the freshness headers of the worker script (such as `max-age`), unless the `max-age` is greater than 24 hours, in which case it is capped to 24 hours.


```js
self.addEventListener('install', function(event) {
  // this happens while the old version is still in control
  event.waitUntil(
    fetchStuffAndInitDatabases()
  );
});

self.addEventListener('activate', function(event) {
  // the old version is gone now, do what you couldn't
  // do while it was still around
  event.waitUntil(
    schemaMigrationAndCleanup()
  )
});
```

Here’s [how that looks in practice](https://www.youtube.com/watch?v=VEshtDMHYyA).

Unfortunately refreshing a single tab isn’t enough to allow an old worker to be collected and a new one take over. Browsers make the next page request before unloading the current page, so there isn’t a moment when current active worker can be released.

The easiest way at the moment is to close & reopen the tab (cmd+w, then cmd+shift+t on Mac), or shift+reload then normal reload.

## The cache

Service worker comes with a [caching API](https://w3c.github.io/ServiceWorker/#cache-objects), letting you create stores of responses keyed by request.

```js
self.addEventListener('install', function(event) {
  // pre cache a load of stuff:
  event.waitUntil(
    caches.open('myapp-static-v1').then(function(cache) {
      return cache.addAll([
        '/',
        '/styles/all.css',
        '/styles/imgs/bg.png',
        '/scripts/all.js'
      ]);
    })
  )
});

self.addEventListener('fetch', function(event) {
  event.respondWith(
    caches.match(event.request).then(function(cachedResponse) {
      return cachedResponse || fetch(event.request);
    })
  );
});
```

Matching within the cache is similar to the browser cache. Method, URL and `vary` headers are taken into account, but freshness headers are ignored. Things are only removed from caches when you remove them.

You can add individual items to the cache with `cache.put(request, response)`, including ones you’ve created yourself. You can also control matching, [discounting things](https://w3c.github.io/ServiceWorker/#cache-query-options-dictionary) such as query string, methods, and vary headers.

## Other service worker–related specifications

Since service workers can spin up in time for events, they’ve opened up the possibility for other features that happen occasionally in the background, even when the page isn’t open. Such as:

* [Push](http://w3c.github.io/push-api/)
* [Background sync](https://github.com/slightlyoff/BackgroundSync)
* [Geofencing](https://github.com/slightlyoff/Geofencing)

## Conclusions

This document only scratches the surface of what service workers enable, and isn’t an exhaustive list of all of the available APIs available to controlled pages or service worker instances. Nor does it cover emergent practices for authoring, composing, and upgrading applications architected to use service workers. It is, hopefully, a guide to understanding the promise of service workers and the rich promise of offline-by-default web applications that are URL friendly and scalable.

## Acknowledgments

Many thanks to [Web Personality of the Year nominee](http://www.ubelly.com/thecritters/) Jake (“B.J.”) Archibald, David Barrett-Kahn, Anne van Kesteren, Michael Nordman, Darin Fisher, Alec Flett, Andrew Betts, Chris Wilson, Aaron Boodman, Dave Herman, Jonas Sicking, Greg Billock, Karol Klepacki, Dan Dascalescu, and Christian Liebel for their comments and contributions to this document and to the discussions that have informed it.
