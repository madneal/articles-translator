## How JavaScript Works: Inside the Networking Layer + How to Optimize Its Performance and Security

## JavaScript 是如何工作的：在网络层如何优化性能和安全

>  原文：[How JavaScript Works: Inside the Networking Layer + How to Optimize Its Performance and Security](https://blog.sessionstack.com/how-javascript-works-inside-the-networking-layer-how-to-optimize-its-performance-and-security-f71b7414d34c)
>
>  译者：[neal1991](https://github.com/neal1991)
>
>  welcome to star my [articles-translator ](https://github.com/neal1991), providing you advanced articles translation. Any suggestion, please issue or contact [me](mailto:bing@stu.ecnu.edu.cn)
>
>  LICENSE: [MIT](https://opensource.org/licenses/MIT)

This is post # 12 of the series dedicated to exploring JavaScript and its building components. In the process of identifying and describing the core elements, we also share some rules of thumb we use when building [SessionStack](https://www.sessionstack.com/?utm_source=medium&utm_medium=blog&utm_content=js-series-networking-layer-intro), a JavaScript application that needs to be robust and highly-performant to help users see and reproduce their web app defects real-time.

这是专门探索 JavaScript 及其构建组件的系列 #12。 在识别和描述核心元素的过程中，我们还分享了构建SessionStack 时使用的一些经验法则，这是一个需要强大且高性能的 JavaScript 应用程序，可帮助用户实时查看和重现其Web应用程序缺陷。

If you missed the previous chapters, you can find them here:

如果你错过了之前的章节，你可以从这找到他们： 

* A[n overview of the engine, the runtime, and the call stack](https://blog.sessionstack.com/how-does-javascript-actually-work-part-1-b0bacc073cf?source=collection_home---2------1----------------)

* [Inside Google’s V8 engine + 5 tips on how to write optimized code](https://blog.sessionstack.com/how-javascript-works-inside-the-v8-engine-5-tips-on-how-to-write-optimized-code-ac089e62b12e?source=collection_home---2------2----------------)

* [Memory management + how to handle 4 common memory leaks](https://blog.sessionstack.com/how-javascript-works-memory-management-how-to-handle-4-common-memory-leaks-3f28b94cfbec?source=collection_home---2------0----------------)

* [The event loop and the rise of Async programming + 5 ways to better coding with async/await](https://blog.sessionstack.com/how-javascript-works-event-loop-and-the-rise-of-async-programming-5-ways-to-better-coding-with-2f077c4438b5)

* [Deep dive into WebSockets and HTTP/2 with SSE + how to pick the right path](https://blog.sessionstack.com/how-javascript-works-deep-dive-into-websockets-and-http-2-with-sse-how-to-pick-the-right-path-584e6b8e3bf7?source=collection_home---4------0----------------)

* [A comparison with WebAssembly + why in certain cases it’s better to use it over JavaScript](https://blog.sessionstack.com/how-javascript-works-a-comparison-with-webassembly-why-in-certain-cases-its-better-to-use-it-d80945172d79)

* [The building blocks of Web Workers + 5 cases when you should use them](https://blog.sessionstack.com/how-javascript-works-the-building-blocks-of-web-workers-5-cases-when-you-should-use-them-a547c0757f6a)

* [Service Workers, their life-cycle, and use cases](https://blog.sessionstack.com/how-javascript-works-service-workers-their-life-cycle-and-use-cases-52b19ad98b58)

* [The mechanics of Web Push Notifications](https://blog.sessionstack.com/how-javascript-works-the-mechanics-of-web-push-notifications-290176c5c55d)

* [Tracking changes in the DOM using MutationObserver](https://blog.sessionstack.com/how-javascript-works-tracking-changes-in-the-dom-using-mutationobserver-86adc7446401)

* [The rendering engine and tips to optimize its performance](https://blog.sessionstack.com/how-javascript-works-the-rendering-engine-and-tips-to-optimize-its-performance-7b95553baeda)

Just like we mentioned in our previous blog post about the [rendering engine](https://github.com/neal1991/articles-translator/blob/master/JavaScript%E6%98%AF%E5%A6%82%E4%BD%95%E5%B7%A5%E4%BD%9C%E7%9A%84%EF%BC%9A%E6%B8%B2%E6%9F%93%E5%BC%95%E6%93%8E%E4%BB%A5%E5%8F%8A%E4%BC%98%E5%8C%96%E5%BB%BA%E8%AE%AE.md), we believe that the difference between good and great JavaScript developers is that the latter not only understand the nuts and bolts of the language but also its internals and the surrounding environment.

就像我们之前在博客中提到的[渲染引擎](https://github.com/neal1991/articles-translator/blob/master/JavaScript%E6%98%AF%E5%A6%82%E4%BD%95%E5%B7%A5%E4%BD%9C%E7%9A%84%EF%BC%9A%E6%B8%B2%E6%9F%93%E5%BC%95%E6%93%8E%E4%BB%A5%E5%8F%8A%E4%BC%98%E5%8C%96%E5%BB%BA%E8%AE%AE.md)一样，我们认为好的和优秀的 JavaScript 开发人员之间的区别在于，后者不仅了解语言的具体细节，还要了解其内部和周围环境。

### 一小段历史

Forty-nine years ago, a thing called ARPAnet was created. It was [an early packet switching network](https://en.wikipedia.org/wiki/Packet_switching) and the first network [to implement the TCP/IP suite](https://en.wikipedia.org/wiki/Internet_protocol_suite). That network set up a link between University of California and Stanford Research Institute. 20 years later Tim Berners-Lee circulated a proposal for a “Mesh” which later became better known as the World Wide Web. In those 49, years the internet has come a long way, starting from just two computers exchanging packets of data, to reaching more than 75 million servers, 3.8B people using the internet and 1.3B websites.

四十九年前，创建了一个名为 ARPAnet 的东西。 这是一个[早期的分组交换网络](https://en.wikipedia.org/wiki/Packet_switching)，也是[实现 TCP/IP 套件](https://en.wikipedia.org/wiki/Internet_protocol_suite)的第一个网络。 该网络在加州大学和斯坦福研究院之间建立了连接。 20年后，Tim Berners-Lee 发布了一个名为“Mesh”的提案，该提案后来更为人称为万维网。 在那49年里，互联网已经走过了漫长的道路，从两台计算机交换数据包开始，到达超过7500万台服务器，38 亿人使用互联网和 1.3B 网站。

![increase_network1_x8P3OcgcgKrEEDpgT2IKkQ.jpeg](http://ozfo4jjxb.bkt.clouddn.com/increase_network1_x8P3OcgcgKrEEDpgT2IKkQ.jpeg)

In this post, we’ll try to analyze what techniques modern browsers employ to automatically boost performance (without you even knowing it), and we’ll specifically zoom in on the browser networking layer. At the end, we’ll provide some ideas on how to help browsers boost even more the performance of your web apps.

在这篇文章中，我们将尝试分析现代浏览器采用哪些技术来自动提升性能（甚至不知道它），并且我们将特别放大浏览器网络层。 最后，我们将提供一些关于如何帮助浏览器提升 Web 应用性能的想法。

## 概述

The modern web browser has been specifically designed for the fast, efficient and secure delivery of web apps/websites. With hundreds of components running on different layers, from process management and security sandboxing to GPU pipelines, audio and video, and many more, the web browser looks more like an operating system rather than just a software application.

The overall performance of the browser is determined by a number of large components: parsing, layout, style calculation, JavaScript and WebAssembly execution, rendering, and of course, **the networking stack**.

Engineers often think that the networking stack is a bottleneck. This is frequently the case since all resources need to be fetched from the internet before the rest of the steps are unblocked. For the networking layer to be efficient it needs to play the role of more than just a simple socket manager. It is presented to us as a very simple mechanism for resource fetching but it’s actually an entire platform with its own optimization criteria, APIs, and services.

现代网络浏览器专为快速，高效和安全地交付网络应用/网站而设计。数百个组件运行在不同的层上，从流程管理和安全沙箱到 GPU 流水线，音频和视频等等，Web 浏览器看起来更像是一个操作系统，而不仅仅是一个软件应用程序。

浏览器的整体性能取决于许多大型组件：解析，布局，样式计算，JavaScript 和 WebAssembly 执行，渲染，当然还有网络堆栈。

工程师经常认为网络堆栈是一个瓶颈。这通常是这种情况，因为在继续其他步骤之前，需要从互联网获取所有资源。为了提高网络层的效率，它不仅需要扮演简单的套接字管理员的角色。它作为一种非常简单的资源获取机制呈现给我们，但它实际上是一个拥有自己的优化标准，API 和服务的整个平台。

![web.jpeg](http://ozfo4jjxb.bkt.clouddn.com/web.jpeg)

As web developers, we don’t have to worry about the individual TCP or UDP packets, request formatting, caching and everything else that’s going on. The entire complexity is taken care of by the browser so we can focus on the application we’re developing. Knowing what’s happening under the hood, however, can help us create faster and more secure applications.

In essence, here’s what’s happening when the user starts interacting with the browser:

作为 Web 开发人员，我们不必担心单个 TCP 或 UDP 数据包，请求格式化，缓存和其他所有事情。 整个复杂性由浏览器处理，因此我们可以专注于我们正在开发的应用程序。 但是，了解发生了什么，可以帮助我们创建更快，更安全的应用程序。

实质上，用户开始与浏览器交互时发生了以下情况：

* The user enters a URL in the browser address bar用户在浏览器地址栏中输入 URL

* Given the URL of a resource on the web, the browser starts by checking its local and application caches and tries to use a local copy to fulfill the request.在给定 Web 上资源的 URL，浏览器首先检查本地和应用程序缓存，并尝试使用本地副本来完成请求。

* If the cache cannot be used, the browser takes the domain name from the URL and requests the IP address of the server from a [DNS](https://en.wikipedia.org/wiki/Domain_Name_System). If the domain is cached, no DNS query is needed.如果缓存无法使用，浏览器将从URL中获取域名，并从 [DNS](https://en.wikipedia.org/wiki/Domain_Name_System) 请求服务器的IP地址。 如果该域被缓存，则不需要 DNS 查询。

* The browser creates an HTTP packet saying that it requests a web page located on the remote server.浏览器会创建一个 HTTP 数据包，说明它请求位于远程服务器的网页。

* The packet is sent to the TCP layer which adds its own information on top of the HTTP packet. This information is required to maintain the started session.数据包被发送到 TCP 层，在 TCP 数据包的顶部添加它自己的信息。 此信息是维护启动会话所必需的。

* The packet is then handed to the IP layer which main job is to figure out a way to send the packet from the user to the remote server. This information is also stored on top of the packet.然后将数据包交给 IP层，其主要工作是将数据包从用户发送到远程服务器的方式。 这些信息也存储在数据包的顶部。

* The packet is sent to the remote server.数据包被发送到远程服务器。

* Once the packet is received, the response gets sent back in a similar manner.一旦收到数据包，就会以类似的方式发送响应。

The W3C [Navigation Timing specification](http://www.w3.org/TR/navigation-timing/) provides a browser API as well as visibility into the timing and performance data behind the life of every request in the browser. Let’s inspect the components, as each plays a critical role in delivering the optimal user experience:

[W3C Navigation Timing](http://www.w3.org/TR/navigation-timing/) 规范提供浏览器 API 以及浏览器中每个请求的生命周期背后的时间和性能数据。 让我们来看看这些组件，因为它们在提供最佳用户体验方面起着至关重要的作用：

![](https://cdn-images-1.medium.com/max/2234/1*rjBdCBwOx5Gp_A6b6FQgfw.png)

The whole networking process is very complex and there are many different layers which can become a bottleneck. This is why browsers strive to improve performance on their side by using various techniques so that the impact of the entire network communication is minimal.

整个联网过程非常复杂，有许多不同的层次可能会成为瓶颈。 这就是为什么浏览器努力通过使用各种技术来提高性能的原因，以便整个网络通信的影响最小。

## 套接字管理

Let’s start with some terminology:

让我们从一些术语开始：

* **Origin** — 应用协议，域名和端口号 (比如，https, [www.example.com](http://www.example.com), 443)

* **Socket pool** — a group of sockets belonging to the same origin (all major browsers limit the maximum pool size to 6 sockets)一组属于同源的套接字（所有主流浏览器都将最大池大小限制为6个套接字）

JavaScript and WebAssembly **do not** allow us to manage the lifecycle of individual network sockets, and that’s a good thing! This not only keeps our hands clean but it also allows the browser to automate a lot of performance optimizations some of which include socket reuse, request prioritization and late binding, protocol negotiation, enforcing connection limits, and many other.

Actually, modern browsers go the extra mile to separate the request management cycle from socket management. Sockets are organized in pools, which are grouped by origin, and each pool enforces its own connection limits and security constraints. Pending requests are queued, prioritized, and then bound to individual sockets in the pool. Unless the server intentionally closes the connection, the same socket can be automatically reused across multiple requests!

JavaScript 和 WebAssembly 不允许我们管理单个网络套接字的生命周期，这是一件好事！ 这不仅可以让我们更轻松，而且还可以让浏览器自动进行大量的性能优化，其中一些包括套接字重用，请求优先级和延迟绑定，协议协商，强制连接限制等等。

实际上，现代浏览器会花费更多的时间来将请求管理周期与套接字管理分开。 套接字按池组织，按源分组，每个池强制实施自己的连接限制和安全约束。 待处理的请求排队，优先，然后绑定到池中的单个套接字。 除非服务器有意关闭连接，否则可以在多个请求中自动重用相同的套接字！

![](https://cdn-images-1.medium.com/max/2000/1*0e8X3UTBpsiBSZKa3l1hXA.png)

Since the opening of a new TCP connection comes at an additional cost, the reuse of connections introduces great performance benefits on its own. By default, browsers use the so-called “keepalive” mechanism which saves time from opening a new connection to the server when a request is made. The average time for opening a new TCP connection is:

由于新的 TCP 连接的开通需要额外的成本，因此连接的重复使用对其本身具有很大的性能优势。 默认情况下，浏览器使用所谓的“keepalive”机制，这可以节省在发出请求时打开新连接到服务器的时间。 打开一个新的 TCP 连接的平均时间是：

* 局域请求 — 23ms

* 跨大陆请求— 120ms

* 洲际请求 — 225ms

This architecture opens the door to a number of other optimization opportunities. The requests can be executed in a different order depending on their priority. The browser can optimize the bandwidth allocation across all sockets or it can open sockets in anticipation of a request.

As I mentioned before, this is all managed by the browser and does not require any work on our side. But this doesn’t necessarily mean that we can’t do anything to help. Choosing the right network communication patterns, type, and frequency of transfers, choice of protocols and tuning/optimization of our server stack can play a great role in improving the overall performance of an application.

Some browsers even go one step further. For example, Chrome can self-teach itself to get faster as you use it. It learns based on the sites visited and the typical browsing patterns so it can anticipate likely user behavior and take action before the user does anything. The simplest example is pre-rendering a page when the user hovers on a link. If you’re interested in learning more about Chrome’s optimizations, you can check out this chapter [https://www.igvita.com/posa/high-performance-networking-in-google-chrome/](https://www.igvita.com/posa/high-performance-networking-in-google-chrome/) of the [High-Performance Browser Networking](https://hpbn.co) book.

这种架构创造了一些可能的优化机会。这些请求可以根据其优先级以不同的顺序执行。浏览器可以优化所有套接字上的带宽分配，或者可以在预期请求时打开套接字。

正如我之前提到的，这一切都是由浏览器管理的，并不需要我们的任何工作。但这并不一定意味着我们无能为力。选择正确的网络通信模式，传输类型和频率，选择协议以及调整/优化服务器堆栈可以在提高应用程序的整体性能方面发挥重要作用。

有些浏览器甚至更进一步。例如，Chrome 可以自我学习自己在使用它时变得更快。它根据访问的网站和典型的浏览模式进行学习，以便在用户做任何事情之前预测可能的用户行为并采取行动。最简单的例子是当用户在链接上悬停时预先呈现页面。如果您有兴趣了解更多关于Chrome优化的信息，请参阅本章的 https://www.igvita.com/posa/high-performance-networking-in-google-chrome/ [高性能浏览器](https://hpbn.co/)网络书。

## 网络安全和沙箱

Allowing the browser to manage the individual sockets has another very important purpose: this way the browser enables the enforcement of a consistent set of security and policy constraints on untrusted application resources. For example, the browser does not allow direct API access to raw network sockets as this would enable any malicious application to make arbitrary connections to any host. The browser also enforces connection limits which protect the server as well as the client from resource exhaustion.

The browser formats all outgoing requests to enforce consistent and well-formed protocol semantics to protect the server. Similarly, response decoding is done automatically to protect the user from malicious servers.

允许浏览器管理单个套接字具有另一个非常重要的目的：通过这种方式，浏览器可以对不可信的应用程序资源执行一致的安全和策略约束。 例如，浏览器不允许直接 API 访问原始网络套接字，因为这可以使任何恶意应用程序与任何主机进行任意连接。 浏览器还强制执行连接限制，以保护服务器以及客户端免受资源耗尽。

浏览器格式化所有传出请求，以强化一致且格式良好的协议语义来保护服务器。 同样，响应解码自动完成，以保护用户免受恶意服务器的侵害。

### TLS 协商

[Transport Layer Security (TLS)](https://en.wikipedia.org/wiki/Transport_Layer_Security) is a cryptographic protocol that provides communication security over a computer network. It finds widespread use in many applications, one of which is web browsing. Websites can use TLS to secure all communications between their servers and web browsers.

The entire TLS handshake consists of the following steps:

传输层安全性（TLS）是一种通过计算机网络提供通信安全性的加密协议。 它在许多应用程序中广泛使用，其中之一是网页浏览。 网站可以使用TLS来保护其服务器和Web浏览器之间的所有通信。

整个TLS握手由以下步骤组成：

 1. The client sends a “Client hello” message to the server, along with the client’s random value and supported cipher suites.客户端向服务器端发送“Client hello”消息，以及客户端的随机值和支持的加密套件。

 2. The server responds by sending a “Server hello” message to the client, along with the server’s random value.

 3. The server sends its certificate for authentication to the client and may request a similar certificate from the client. The server sends the “Server hello done” message.

 4. If the server has requested a certificate from the client, the client sends it.

 5. The client creates a random Pre-Master Secret and encrypts it with the public key from the server’s certificate, sending the encrypted Pre-Master Secret to the server.

 6. The server receives the Pre-Master Secret. The server and the client each generate the Master Secret and session keys based on the Pre-Master Secret.

 7. The client sends a “Change cipher spec” notification to the server to indicate that the client will start using the new session keys for hashing and encrypting messages. The client also sends a “Client finished” message.

 8. The server receives the “Change cipher spec” and switches its record layer security state to symmetric encryption using the session keys. The server sends a “Server finished” message to the client.

 9. Client and server can now exchange application data over the secured channel they have established. All messages sent from the client to the server and back are encrypted using the session key.

The user is warned in case any of the verifications fail — e.g., the server is using a self-signed certificate.

### Same-origin policy

Two pages have the same origin if the protocol, port (if one is specified), and host are the same for both pages.

Here are some examples of resources which may be embedded cross-origin:

* JavaScript with <script src=”…”></script>. Error messages for syntax errors are only available for same-origin scripts

* CSS with <link rel=”stylesheet” href=”…”>. Due to the relaxed syntax rules of CSS, cross-origin CSS requires a correct Content-Type header. Restrictions vary by browser

* Images with <img>

* Media files with <video> and <audio>

* Plug-ins with <object>, <embed> and <applet>

* Fonts with @font-face. Some browsers allow cross-origin fonts, others require same-origin fonts.

* Anything with <frame> and <iframe>. A site can use the X-Frame-Options header to prevent this form of cross-origin interaction.

The above list is far from complete; its goal is to highlight the principle of “least privilege” at work. The browser exposes only the APIs and resources that are necessary for the application code: the application supplies the data and the URL, and the browser formats the request and handles the full lifecycle of each connection.

It’s worth noting that there is no single concept of the “same-origin policy.” Instead, there is a set of related mechanisms that enforce restrictions on DOM access, cookie and session state management, networking, and other components of the browser.

## Resource and Client State Caching

The best and fastest request is a request not made. Prior to dispatching a request, the browser automatically checks its resource cache, performs the necessary validation checks, and returns a local copy of the resources if the specified conditions are met. If a local resource is not available in the cache, then a network request is made and the response is automatically placed in the cache for a subsequent access if such is permitted.

* The browser automatically evaluates caching directives on each resource

* The browser automatically revalidates expired resources when possible

* The browser automatically manages the size of the cache and resource eviction

Managing an efficient and optimized resource cache is hard. Thankfully, the browser takes care of the entire complexity on our behalf, and all we need to do is ensure that our servers are returning the appropriate cache directives; to learn more, see [Cache Resources on the Client](https://hpbn.co/optimizing-application-delivery/#cache-resources-on-the-client). You do provide Cache-Control, ETag, and Last-Modified response headers for all the resources on your pages, right?

Finally, an often-overlooked but critical function of the browser is its duty to provide authentication, session, and cookie management. The browser maintains separate “cookie jars” for each origin, provides necessary application and server APIs to read and write new cookie, session, and authentication data, and automatically appends and processes appropriate HTTP headers to automate the entire process on our behalf.

### An example:

A simple but illustrative example of the convenience of deferring session state management to the browser: an authenticated session can be shared across multiple tabs or browser windows, and vice versa; a sign-out action in a single tab will invalidate open sessions in all other open windows.

## Application APIs and Protocols

Walking up the ladder of provided network services we finally arrive at the application APIs and protocols. As we saw, the lower layers provide a wide array of critical services: socket and connection management, request and response processing, enforcement of various security policies, caching, and much more. Every time we initiate an HTTP or an XMLHttpRequest, a long-lived Server-Sent Event or WebSocket session, or open a WebRTC connection, we are interacting with some or all of these underlying services.

There is no single best protocol or API. Every non-trivial application will require a mix of different transports based on a variety of requirements: interaction with the browser cache, protocol overhead, message latency, reliability, type of data transfer, and more. Some protocols may offer low-latency delivery (e.g., Server-Sent Events, WebSocket), but may not meet other critical criteria, such as the ability to leverage the browser cache or support efficient binary transfers in all cases.

## A Few Things You Can Do To Improve Your Web App Performance and Security

* Always use the “Connection: Keep-Alive” header in your requests. Browsers do this by default. Make sure that the server uses the same mechanism.

* Use the proper Cache-Control, Etag and Last-Modified headers so you can save the browser some downloading time.

* Spend time tweaking and optimizing your web server. This is where real magic can happen! Bear in mind that the process if is very specific for each web app and the type of data you’re transmitting.

* Always use TLS! Especially if you have any type of authentication in your application.

* Research what security policies browsers provide and enforce them in your application.

Both performance and security are first-class citizens in [SessionStack](https://www.sessionstack.com/?utm_source=medium&utm_medium=blog&utm_content=js-series-networking-layer-outro). The reason why we can’t afford to compromise on either is that once SessionStack is integrated into your web app, it starts monitoring everything from DOM changes and user interaction to network requests, unhandled exceptions and debug messages. All the data is transmitted to our servers real-time which allows you to replay issues from your web apps as videos and see everything that happened to your users. And this all is taking place with minimum latency and no performance overhead to your app.

This is why we strive to employ all the above tips + a few more which we’ll discuss in a future post.

There is a free plan if you’d like to [give SessionStack a try](https://www.sessionstack.com/signup/).

![](https://cdn-images-1.medium.com/max/NaN/0*h2Z_BnDiWfVhgcEZ.)

### Resources

* [https://hpbn.co/](https://hpbn.co/)

* [https://www.amazon.com/Tangled-Web-Securing-Modern-Applications/dp/1593273886](https://www.amazon.com/Tangled-Web-Securing-Modern-Applications/dp/1593273886)

* [https://msdn.microsoft.com/en-us/library/windows/desktop/aa380513(v=vs.85).aspx](https://msdn.microsoft.com/en-us/library/windows/desktop/aa380513(v=vs.85).aspx)

* [http://www.internetlivestats.com/](http://www.internetlivestats.com/)

* [http://vanseodesign.com/web-design/browser-requests/](http://vanseodesign.com/web-design/browser-requests/)
