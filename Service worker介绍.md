# Service worker介绍

> 原文：[Service workers explained](https://github.com/w3c/ServiceWorker/blob/master/explainer.md)
>
> 译者：[neal1991](https://github.com/neal1991)
>
> welcome to star my [articles-translator](https://github.com/neal1991/articles-translator/), providing you advanced articles translation. Any suggestion, please issue or contact [me](mailto:bing@stu.ecnu.edu.cn)
>
> LICENSE: [MIT](https://opensource.org/licenses/MIT)

## 那么它是什么？

Service worker正是被开发用于解决web平台上经常出现的问题和疑虑，包括：

* 无法解释（[Extensible Web Manifesto](https://extensiblewebmanifesto.org/) 中）的HTTP缓存以及高级HTTP交互比如HTML5 AppCache。
* 难以自然地构建一个离线优先地web应用。
* 缺乏可以利用很多提出功能的上下文执行。


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

我们可以利用service workers：

* 利用网络拦截可以让让网站[更快以及/或者支持离线使用](https://www.youtube.com/watch?v=px-J9Ghvcx4)
* 作为其它'background'功能的基础比如[消息推送](http://updates.html5rocks.com/2015/03/push-notificatons-on-the-open-web)以及[后台同步](https://github.com/slightlyoff/BackgroundSync/blob/master/explainer.md)

## 开始

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

在这个例子中，`/my-app/sw.js`就是service worker脚本的位置，并且它控制那些页面的URL以`/my-app/`开头。

`.register`返回一个promise。如果你以前没接触过promise的话，可以看看[HTML5Rocks article](http://www.html5rocks.com/en/tutorials/es6/promises/)。

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

额，不完全是。当document浏览时，它会选择一个service worker作为它的控制器，因此你使用`.register`注册的document并不是被控制的，因为那并不是service worker首次加载的地方。

如果你刷新document，它将会是在service worker的控制之下。你可以通过`navigator.serviceWorker.controller`来看一下是哪个service worker在进行控制，如果没有的话结果就会是`null`。

注意：当你从一个service worker更新到另外一个的时候，可能会有一点点不一点。我们会进入“Updating"阶段。

如果使用shift来重载网页的话，加载就会有控制器了，这样做是为了测试CSS以及JS变化。

Document通常是和一个service worker存在于整个声明周期，或者根本就没有service worker。然而，service worker可以调用`self.skipWaiting()`([spec](https://w3c.github.io/ServiceWorker/#service-worker-global-scope-skipwaiting)) 来立刻接管范围内的所有页面。

## 网络截获

```js
self.addEventListener('fetch', function(event) {
  console.log(event.request);
});
```

你可以利用fetch事件：

* 在你的service worker作用域下浏览
* 任何被这些页面触发的请求，甚至是对其他源的请求

这意味着你可以监听所有对于这个页面的请求，CSS,JS，图片，XHR，图标等等所有。

* iframes & `<object>`--这些将根据它们的资源URL选择其控制器
* Service workers - 对于service worker的fetch/update请求不会通过service worker
* 请求是在service worker之内出发的 - 否则你会获得一个循环

`request`对象会给你关于这个request的信息，比如它的URL，方法以及头部。但是最有趣的是，他可以劫持请求并且给出不同的响应。

```js
self.addEventListener('fetch', function(event) {
  event.respondWith(new Response("Hello world!"));
});
```

[这是一个 demo](https://jakearchibald.github.io/isserviceworkerready/demos/manual-response/).

`.repondWith`使用一个`Reponse`对象或者一个解析后的promise。上面我们是在创建一个手工的response。这个`Reponse`对象来自于 [Fetch Spec](https://fetch.spec.whatwg.org/#response-class).。在这个规范里面同样也存在着`fetch()`方法，它会返回一个promise作为响应，这意味着你可以在任何地方获取你的响应。

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

在上面，我捕获了以`.jpg`结尾的请求并且将Google doodle作为响应。`fetch()`请求默认是 [CORS](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing)，但是通过设置`no-cors`我可用使用这个响应，即使他不能跨域访问headers（尽管我们不能利用JavaScript访问内容）。[这是demo](https://jakearchibald.github.io/isserviceworkerready/demos/img-rewrite/).

Promise能够让你从一个方法返回到另外一个方法：

```js
self.addEventListener('fetch', function(event) {
  event.respondWith(
    fetch(event.request).catch(function() {
      return new Response("Request failed!");
    })
  );
});
```

Service worker是带有一个cache API，使得以后可以方便的存储响应以便重用。不久之后，但是第一点

## 更新一个service worker

Service worker的生命周期是建立在Chrome的更新模型上的：在后台尽可能多地做，不要打扰用户，当当前版本关闭的时候完成更新。

无论何时你在你的service worker作用域内浏览页面，浏览器都会在后台检查更新。如果这个脚本是字节不同的，那么它就会被认为是一个新的版本，并且被安装（注意：只有这个脚本被检查，而不是外部的`importScripts`）。然而，老版本的会持续对页面的控制直到所有使用它的tab都被关闭了（除非在install的过程中调用`.replace()`）。接着这个老版本的就会被回收从而新的版本开始接管。

这样做是为了避免同时运行两个版本的service worker在不同的tab中。我们当前的策略是： [“cross fingers, hope it doesn’t happen”](https://twitter.com/jaffathecake/status/502779501936652289).

注意：更新遵顼header中worker脚本的新鲜度（比如`max-age`），除非`max-age`大于24个小时，否则最多只能保持24个小时。


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

[下面是实践中的实现](https://www.youtube.com/watch?v=VEshtDMHYyA)：

不幸的是，刷新一个tab不足够收集到旧的woker兵器让新的进行接管。浏览期在上传当前页面之前向下一个页面发送请求，所以不存在当前active worker被释放。

最简单的方法是关闭然后重新打开这个tab（cmd+w，然后cmd+shift+t Mac），或者shift+reload然后就是正常的重新加载了。

## 缓存

Service worker带有一个[caching API](https://w3c.github.io/ServiceWorker/#cache-objects)能够让你产生由请求作为键值的store。

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

在缓存之内匹配类似于浏览器的缓存。方法，URL以及`vary`header都被考虑在内，但是header的新鲜度被忽略了。缓存的东西只有在你手动移除的时候才生效。

你可以通过`cache.put(request, response)`向缓存中添加独立的条目，包括你自己产生的。你也可以控制匹配，[忽略其它的](https://w3c.github.io/ServiceWorker/#cache-query-options-dictionary)，比如查询字符串，方法以及vary header。

## 其它service worker相关的标准

由于service worker可以及时地调动事件，及时未打开页面，也可以在后台偶尔调用其它功能：

* [Push](http://w3c.github.io/push-api/)
* [Background sync](https://github.com/slightlyoff/BackgroundSync)
* [Geofencing](https://github.com/slightlyoff/Geofencing)

## 总结

这份文档只是简要地介绍了service worker的能力，并不是售空页面或者service worker实例的所有的可用的API。也不涉及创作，修改以及更新应用程序的service worker。通过这个，希望能够引导你理解service worker中的promise以及对于URL友好的以及可伸缩的默认支持离线使用的web应用的丰富的promise。

## Acknowledgments

Many thanks to [Web Personality of the Year nominee](http://www.ubelly.com/thecritters/) Jake (“B.J.”) Archibald, David Barrett-Kahn, Anne van Kesteren, Michael Nordman, Darin Fisher, Alec Flett, Andrew Betts, Chris Wilson, Aaron Boodman, Dave Herman, Jonas Sicking, Greg Billock, Karol Klepacki, Dan Dascalescu, and Christian Liebel for their comments and contributions to this document and to the discussions that have informed it.
