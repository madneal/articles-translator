
## How JavaScript works: Service Workers, their lifecycle and use cases

This is post # 8 of the series dedicated to exploring JavaScript and its building components. In the process of identifying and describing the core elements, we also share some best practice we use when building [SessionStack](https://www.sessionstack.com/?utm_source=medium&utm_medium=source&utm_content=javascript-series-web-workers-intro), a JavaScript application that has to be robust and highly-performant in order to show you real-time exactly how your users ran into a technical or UX issue in your web app.

If you missed the previous chapters, you can find them here:

* A[n overview of the engine, the runtime, and the call stack](https://blog.sessionstack.com/how-does-javascript-actually-work-part-1-b0bacc073cf?source=collection_home---2------1----------------)

* [Inside Google’s V8 engine + 5 tips on how to write optimized code](https://blog.sessionstack.com/how-javascript-works-inside-the-v8-engine-5-tips-on-how-to-write-optimized-code-ac089e62b12e?source=collection_home---2------2----------------)

* [Memory management + how to handle 4 common memory leaks](https://blog.sessionstack.com/how-javascript-works-memory-management-how-to-handle-4-common-memory-leaks-3f28b94cfbec?source=collection_home---2------0----------------)

* [The event loop and the rise of Async programming + 5 ways to better coding with async/await](https://blog.sessionstack.com/how-javascript-works-event-loop-and-the-rise-of-async-programming-5-ways-to-better-coding-with-2f077c4438b5)

* [Deep dive into WebSockets and HTTP/2 with SSE + how to pick the right path](https://blog.sessionstack.com/how-javascript-works-deep-dive-into-websockets-and-http-2-with-sse-how-to-pick-the-right-path-584e6b8e3bf7?source=collection_home---4------0----------------)

* [A comparison with WebAssembly + why in certain cases it’s better to use it over JavaScript](https://blog.sessionstack.com/how-javascript-works-a-comparison-with-webassembly-why-in-certain-cases-its-better-to-use-it-d80945172d79)

* [The building blocks of Web Workers + 5 cases when you should use them](https://blog.sessionstack.com/how-javascript-works-the-building-blocks-of-web-workers-5-cases-when-you-should-use-them-a547c0757f6a)

![](https://cdn-images-1.medium.com/max/4872/1*oOcY2Gn-LVt1h-e9xOv5oA.jpeg)

You probably already know that [Progressive Web Apps](https://developers.google.com/web/progressive-web-apps/) will only be getting more popular as they aim at making web app user experience smoother, at creating a native app-like experiences rather than browser look and feel.

One of the main requirements to build a Progressive Web App is to make it very reliable in terms of network and loading — it should be usable in uncertain or non-existent network conditions.

In this post, we’ll be deep diving into Service Workers: how they function and what you should care about. At the end, we also list a few unique benefits of the Service Workers that you should take advantage of, and share our own team’s experience here at [SessionStack](https://www.sessionstack.com/).

### Overview

If you want to understand everything about Service Workers, you should start by reading our blog post on [Web Workers](https://blog.sessionstack.com/how-javascript-works-the-building-blocks-of-web-workers-5-cases-when-you-should-use-them-a547c0757f6a).

Basically, the Service Worker is a type of Web Worker, and more specifically it’s like a [Shared Worker](https://developer.mozilla.org/en-US/docs/Web/API/SharedWorker):

* The Service Worker runs in its own global script context

* It isn’t tied to a specific web page

* It cannot access the DOM

One of the main reasons why the Service Worker API is so exciting is that it allows your web apps to support offline experiences, giving developers complete control over the flow.

### Lifecycle of a Service Worker

The lifecycle of a service worker is completely separated from your web page one. It consists of the following phases:

* Download

* Installation

* Activation

### Download

This is when the browser downloads the .js file which contains the Service Worker.

### Installation

To install a Service Worker for your web app, you have to register it first, which you can do in your JavaScript code. When a Service Worker is registered, it prompts the browser to start a Service Worker install step in the background.

By registering the Service Worker, you tell the browser where your Service Worker JavaScript file lives. Let’s look at the following code:

 <iframe src="https://medium.com/media/8cb6eb7ff8df52d6c0a53db5b313a6fd" frameborder=0></iframe>

The code checks whether the Service Worker API is supported in the current environment. If it is, the /sw.js Service Worker is registered.

You can call the register() method every time a page loads with no concern — the browser will figure out if the service worker has already been registered, and will handle it properly.

An important detail of the register() method is the location of the service worker file. In this case you can see that the service worker file is at the root of the domain. This means that the service worker's scope will be the entire origin. In other words, this service worker will receive fetch events (which we’ll discuss later) for everything on this domain. If we register the service worker file at /example/sw.js, then the service worker would only see fetch events for pages which URLs start with /example/ (i.e. /example/page1/, /example/page2/).

During the installation phase, it’s best to load and cache some static assets. Once the assets are successfully cached, the Service Worker installation is complete. If not (the loading fails) — the Service Worker will do a retry. Once installed successfully, you’ll know that the static assets are in the cache.

This answers your question if registration need to happen after the load event. It’s not a must, but it’s definitely recommended.

Why so? Let’s consider a user’s first visit to your web app. There’s no service worker yet, and the browser has no way of knowing in advance whether there will be a service worker that will eventually be installed. If the Service Worker gets installed, the browser will need to spend extra CPU and memory for this additional thread which otherwise the browser will spend on rendering the web page instead.

The bottom line is that , if you just install a Service Worker on your page, you’re running the risk of delaying the loading and rendering — not making the page available to your users as quickly as possible.

Note that this is important only for the first page visit. Subsequent page visits don’t get impacted by the Service Worker installation. Once a Service Worker is activated on a first page visit, it can handle loading/caching events for subsequent visits to your web app. This all makes sense, because it needs to be ready to handle limited network connectivity.

### Activation

After the Service Worker is installed, the next step will be its activation. This step is a great opportunity to manage previous caches.

Once activated, the Service Worker will start controlling all pages that fall under its scope. An interesting fact: the page that registered the Service Worker for the first time won’t be controlled until that page is loaded again. Once the Service Worker is in control, it will be in one of the following states:

* It will handle fetch and message events that occur when a network request or message is made from the page

* It will be terminated to save memory

Here is how the lifecycle will look like:

![](https://cdn-images-1.medium.com/max/2000/1*mVOrpKC9pFTMg4EXPozoog.png)

### Handling the installation inside the Service Worker

After a page spins up the registration process, let’s see what happens inside the Service Worker script, which handles the install event by adding an event listener to the Service Worker instance.

Those are the steps that need to be taken when the install event is handled:

* Open a cache

* Cache our files

* Confirm whether all of the required assets are cached

Here is what a simple installation might look like inside a Service Worker:

 <iframe src="https://medium.com/media/8a4c5afca4a07c30b6c791be08bbdbc4" frameborder=0></iframe>

If all the files are successfully cached, then the service worker will be installed. If **any** of the files fail to download, then the install step will fail. So be careful what files you put there.

Handling the install event is completely optional and you can avoid it, in which case you don’t need to perform any of the steps here.

### Caching requests during runtime

This part is the real-deal. This is where you’ll see how to intercept requests and return the created caches (and create new ones).

After a Service Worker is installed and the user navigates to another page or refreshes the page he’s on, the Service Worker will receive fetch events. Here is an example that demonstrates how to return cached assets or perform a new request and then cache the result:

 <iframe src="https://medium.com/media/636ebc25f8c60b32f15d19ddfafb8736" frameborder=0></iframe>

Here is what happens in a nutshell:

* The event.respondWith() will determine how we’ll respond to the fetch event. We pass a promise from caches.match() which looks at the request and finds if there are any cached results from any of the caches that have been created.

* If there is a cache, the response is retrieved.

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

### Deleting data from the cache

The most common step in the activate callback is cache management. You’d want to do this now because if you were to wipe out any old caches in the install step, old Service Workers will suddenly stop being able to serve files from that cache.

Here is an example how you can delete some files from the cache that are not whitelisted (in this case, having page-1 or page-2 under their names):

 <iframe src="https://medium.com/media/05d9fbb176b3902e930496d2bcbd53e7" frameborder=0></iframe>

### HTTPS requirement

When you’re building your web app, you’ll be able to use Service Workers through localhost, but once you deploy it in production, you need to have HTTPS ready (and that’s the last reason for you to have HTTPS).

Using a Service Worker, you can hijack connections and fabricate responses. By not using HTTPs, your web app becomes prone to a [man-in-the-middle attacks](https://en.wikipedia.org/wiki/Man-in-the-middle_attack).

To make things safer, you’re required to register Service Workers on pages that are served over HTTPS so that you know that the Service Worker which the browser receives, hasn’t been modified while traveling through the network.

### Browser support

The browser support for Service Workers is getting better:

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

### Resources

* [https://developers.google.com/web/fundamentals/primers/service-workers/](https://developers.google.com/web/fundamentals/primers/service-workers/)

* [https://github.com/w3c/ServiceWorker/blob/master/explainer.md](https://github.com/w3c/ServiceWorker/blob/master/explainer.md)
