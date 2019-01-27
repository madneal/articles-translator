## How JavaScript Works: Inside the Networking Layer + How to Optimize Its Performance and Security

## JavaScript 是如何工作的：在网络层如何优化性能和安全

>  原文：[How JavaScript Works: Inside the Networking Layer + How to Optimize Its Performance and Security](https://blog.sessionstack.com/how-javascript-works-inside-the-networking-layer-how-to-optimize-its-performance-and-security-f71b7414d34c)
>
>  译者：[neal1991](https://github.com/neal1991)
>
>  welcome to star my [articles-translator](https://github.com/neal1991/articles-translator/), providing you advanced articles translation. Any suggestion, please issue or contact [me](mailto:bing@stu.ecnu.edu.cn)
>
>  LICENSE: [MIT](https://opensource.org/licenses/MIT)

这是专门探索 JavaScript 及其构建组件的系列 #12。 在识别和描述核心元素的过程中，我们还分享了构建SessionStack 时使用的一些经验法则，这是一个需要强大且高性能的 JavaScript 应用程序，可帮助用户实时查看和重现其 Web 应用程序缺陷。

如果你错过了之前的章节，你可以从这找到他们： 

* [JavaScript是如何工作的：引擎，运行时以及调用栈的概述](https://github.com/neal1991/articles-translator/blob/master/JavaScript%E6%98%AF%E5%A6%82%E4%BD%95%E5%B7%A5%E4%BD%9C%EF%BC%9A%E7%B3%BB%E5%88%97%E4%B8%80.md)（已翻译）
* [Inside Google’s V8 engine + 5 tips on how to write optimized code](https://blog.sessionstack.com/how-javascript-works-inside-the-v8-engine-5-tips-on-how-to-write-optimized-code-ac089e62b12e?source=collection_home---2------2----------------)
* [JavaScript是如何工作的：内存管理以及如何处理四种常见的内存泄漏](https://github.com/neal1991/articles-translator/blob/master/JavaScript%E6%98%AF%E5%A6%82%E4%BD%95%E5%B7%A5%E4%BD%9C%E7%9A%84%EF%BC%9A%E7%B3%BB%E5%88%97%E4%B8%89.md)（已翻译）
* [The event loop and the rise of Async programming + 5 ways to better coding with async/await](https://blog.sessionstack.com/how-javascript-works-event-loop-and-the-rise-of-async-programming-5-ways-to-better-coding-with-2f077c4438b5)
* [Deep dive into WebSockets and HTTP/2 with SSE + how to pick the right path](https://blog.sessionstack.com/how-javascript-works-deep-dive-into-websockets-and-http-2-with-sse-how-to-pick-the-right-path-584e6b8e3bf7?source=collection_home---4------0----------------)
* [A comparison with WebAssembly + why in certain cases it’s better to use it over JavaScript](https://blog.sessionstack.com/how-javascript-works-a-comparison-with-webassembly-why-in-certain-cases-its-better-to-use-it-d80945172d79)
* [The building blocks of Web Workers + 5 cases when you should use them](https://blog.sessionstack.com/how-javascript-works-the-building-blocks-of-web-workers-5-cases-when-you-should-use-them-a547c0757f6a)
* [JavaScript 是如何工作的: Service Workers, 它们的生命周期和使用案例](https://blog.sessionstack.com/how-javascript-works-service-workers-their-life-cycle-and-use-cases-52b19ad98b58)（已翻译）
* [The mechanics of Web Push Notifications](https://blog.sessionstack.com/how-javascript-works-the-mechanics-of-web-push-notifications-290176c5c55d)
* [Tracking changes in the DOM using MutationObserver](https://blog.sessionstack.com/how-javascript-works-tracking-changes-in-the-dom-using-mutationobserver-86adc7446401)
* [JavaScript 是如何工作的：渲染引擎以及优化建议](https://github.com/neal1991/articles-translator/blob/master/JavaScript%E6%98%AF%E5%A6%82%E4%BD%95%E5%B7%A5%E4%BD%9C%E7%9A%84%EF%BC%9A%E6%B8%B2%E6%9F%93%E5%BC%95%E6%93%8E%E4%BB%A5%E5%8F%8A%E4%BC%98%E5%8C%96%E5%BB%BA%E8%AE%AE.md)（已翻译）

就像我们之前在博客中提到的[渲染引擎](https://github.com/neal1991/articles-translator/blob/master/JavaScript%E6%98%AF%E5%A6%82%E4%BD%95%E5%B7%A5%E4%BD%9C%E7%9A%84%EF%BC%9A%E6%B8%B2%E6%9F%93%E5%BC%95%E6%93%8E%E4%BB%A5%E5%8F%8A%E4%BC%98%E5%8C%96%E5%BB%BA%E8%AE%AE.md)一样，我们认为好的和优秀的 JavaScript 开发人员之间的区别在于，后者不仅了解语言的具体细节，还要了解其内部和周围环境。

### 一小段历史

四十九年前，一个名为 ARPAnet 的东西被创建了。 这是一个[早期的分组交换网络](https://en.wikipedia.org/wiki/Packet_switching)，也是[实现 TCP/IP 套件](https://en.wikipedia.org/wiki/Internet_protocol_suite)的第一个网络。 该网络在加州大学和斯坦福研究院之间建立了连接。 20年后，Tim Berners-Lee 发布了一个名为“Mesh”的提案，该提案后来更为人称为万维网。 在那49年里，互联网已经走过了漫长的道路，从两台计算机交换数据包开始，到达超过7500万台服务器，38 亿人使用互联网和 1.3B 网站。

![increase_network1_x8P3OcgcgKrEEDpgT2IKkQ.jpeg](http://ozfo4jjxb.bkt.clouddn.com/increase_network1_x8P3OcgcgKrEEDpgT2IKkQ.jpeg)

在这篇文章中，我们将尝试分析现代浏览器采用哪些技术来自动提升性能（甚至不知道它），并且我们将特别研究浏览器网络层。 最后，我们将提供一些关于如何帮助浏览器提升 Web 应用性能的想法。

## 概述

现代网络浏览器专为快速，高效和安全地交付网络应用/网站而设计。数百个组件运行在不同的层上，从流程管理和安全沙箱到 GPU 流水线，音频和视频等等，Web 浏览器看起来更像是一个操作系统，而不仅仅是一个软件应用程序。

浏览器的整体性能取决于许多大型组件：解析，布局，样式计算，JavaScript 和 WebAssembly 执行，渲染，当然还有网络堆栈。

工程师经常认为网络堆栈是一个瓶颈。这通常是这种情况，因为在继续其他步骤之前，需要从互联网获取所有资源。为了提高网络层的效率，它不仅需要扮演简单的套接字管理员的角色。它作为一种非常简单的资源获取机制呈现给我们，但它实际上是一个拥有自己的优化标准，API 和服务的整个平台。

![web.jpeg](http://ozfo4jjxb.bkt.clouddn.com/web.jpeg)

作为 Web 开发人员，我们不必担心单个 TCP 或 UDP 数据包，请求格式化，缓存和其他所有事情。 整个复杂性由浏览器处理，因此我们可以专注于我们正在开发的应用程序。 但是，了解发生了什么，可以帮助我们创建更快，更安全的应用程序。

实质上，用户开始与浏览器交互时发生了以下情况：

* 用户在浏览器地址栏中输入 URL

* 在给定 Web 上资源的 URL，浏览器首先检查本地和应用程序缓存，并尝试使用本地副本来完成请求。

* 如果缓存无法使用，浏览器将从 URL 中获取域名，并从 [DNS](https://en.wikipedia.org/wiki/Domain_Name_System) 请求服务器的IP地址。 如果该域被缓存，则不需要 DNS 查询。

* 浏览器会创建一个 HTTP 数据包，说明它请求位于远程服务器的网页。

* 数据包被发送到 TCP 层，在 TCP 数据包的顶部添加它自己的信息。 此信息是维护启动会话所必需的。

* 然后将数据包交给 IP层，其主要工作是将数据包从用户发送到远程服务器的方式。 这些信息也存储在数据包的顶部。

* 数据包被发送到远程服务器。

* 一旦收到数据包，就会以类似的方式发送响应。

[W3C Navigation Timing](http://www.w3.org/TR/navigation-timing/) 规范提供浏览器 API 以及浏览器中每个请求的生命周期背后的时间和性能数据。 让我们来看看这些组件，因为它们在提供最佳用户体验方面起着至关重要的作用：

![](https://cdn-images-1.medium.com/max/2234/1*rjBdCBwOx5Gp_A6b6FQgfw.png)

整个联网过程非常复杂，有许多不同的层次可能会成为瓶颈。 这就是为什么浏览器努力通过使用各种技术来提高性能的原因，以便整个网络通信的影响最小。

## 套接字管理

让我们从一些术语开始：

* **Origin** — 应用协议，域名和端口号 (比如，https, [www.example.com](http://www.example.com), 443)

* **Socket pool** — 一组属于同源的套接字（所有主流浏览器都将最大池大小限制为6个套接字）

JavaScript 和 WebAssembly 不允许我们管理单个网络套接字的生命周期，这是一件好事！ 这不仅可以让我们更轻松，而且还可以让浏览器自动进行大量的性能优化，其中一些包括套接字重用，请求优先级和延迟绑定，协议协商，强制连接限制等等。

实际上，现代浏览器会花费更多的时间来将请求管理周期与套接字管理分开。 套接字按池组织，按源分组，每个池强制实施自己的连接限制和安全约束。 待处理的请求排队，优先，然后绑定到池中的单个套接字。 除非服务器有意关闭连接，否则可以在多个请求中自动重用相同的套接字！

![](https://cdn-images-1.medium.com/max/2000/1*0e8X3UTBpsiBSZKa3l1hXA.png)

由于新的 TCP 连接的开通需要额外的成本，因此连接的重复使用对其本身具有很大的性能优势。 默认情况下，浏览器使用所谓的“keepalive”机制，这可以节省在发出请求时打开新连接到服务器的时间。 打开一个新的 TCP 连接的平均时间是：

* 局域请求 — 23ms

* 跨大陆请求— 120ms

* 洲际请求 — 225ms

这种架构创造了一些可能的优化机会。这些请求可以根据其优先级以不同的顺序执行。浏览器可以优化所有套接字上的带宽分配，或者可以在预期请求时打开套接字。

正如我之前提到的，这一切都是由浏览器管理的，并不需要我们的任何工作。但这并不一定意味着我们无能为力。选择正确的网络通信模式，传输类型和频率，选择协议以及调整/优化服务器堆栈可以在提高应用程序的整体性能方面发挥重要作用。

有些浏览器甚至更进一步。例如，Chrome 可以自我学习自己在使用它时变得更快。它根据访问的网站和典型的浏览模式进行学习，以便在用户做任何事情之前预测可能的用户行为并采取行动。最简单的例子是当用户在链接上悬停时预先呈现页面。如果您有兴趣了解更多关于Chrome优化的信息，请参阅本章的 https://www.igvita.com/posa/high-performance-networking-in-google-chrome [高性能浏览器](https://hpbn.co/)网络书。

## 网络安全和沙箱

允许浏览器管理单个套接字具有另一个非常重要的目的：通过这种方式，浏览器可以对不可信的应用程序资源执行一致的安全和策略约束。 例如，浏览器不允许直接 API 访问原始网络套接字，因为这可以使任何恶意应用程序与任何主机进行任意连接。 浏览器还强制执行连接限制，以保护服务器以及客户端免受资源耗尽。

浏览器格式化所有传出请求，以强化一致且格式良好的协议语义来保护服务器。 同样，响应解码自动完成，以保护用户免受恶意服务器的侵害。

### TLS 协商

[传输层安全性（TLS）](https://en.wikipedia.org/wiki/Transport_Layer_Security)是一种通过计算机网络提供通信安全性的加密协议。 它在许多应用程序中广泛使用，其中之一是网页浏览。 网站可以使用 TLS 来保护其服务器和 Web 浏览器之间的所有通信。

整个TLS握手由以下步骤组成：

 1. 客户端向服务器端发送“Client hello”消息，以及客户端的随机值和支持的加密套件。

 2. 服务器端向客户端回复消息"Server hello"，以及服务器的随机值。

 3. 服务器向客户端发送证书从而验证自己的身份同时可能请求客户端发送一个类似的证书。服务器端发送"Server hello done" 消息。

 4. .如果服务器端向客户端请求过证书，那么客户端就会发送证书。

 5. 客户端创建一个随机的 Pre-Master 密钥并且使用服务器端证书的公钥进行加密，并且将加密过的 Pre-Master 密钥发送到服务器。

 6. 服务器接收到 Pre-Master 密钥。服务器端和客户端各自根据 Pre-Master 密钥生成主密钥和会话密钥。

 7. 客户端向服务器发送一个"Change cipher spec"的通知表示客户端将会开始使用新的会话密钥对消息进行哈希和加密。客户端也会发送一个"Client finished"的消息。

 8. 服务器端接收到"Change cipher spec"的消息并且将其记录层的安全状态使用会话密钥切换到对称加密。服务器端向客户端发送一个"Server finished"消息。

 9. 客户端和服务器端可以在它们已经建立过的安全通道交换应用数据。所有的消息从客户端发送到服务器端或者返回来的都会使用会话密钥进行加密。

在任何验证失败的情况下，用户都会受到警告--比如，服务器使用自己签发的证书。

### 同源策略

如果两个页面的协议，端口（如果指定了一个）和主机相同，则两个页面则是同源。

* 带有 <script src=”…”></script> 的 JavaScript，

* 带有 <link rel =“stylesheet”href =“...”> 的 CSS。 由于 CSS 的宽松语法规则，跨源 CSS 需要正确的 Content-Type 头。 浏览器的限制因人而异

* 带有 <img> 的 images

* 带有 <video> 和 <audio> 的 media

* 带有 <object>,<embed> 以及 <applet> 的  plug-ins

* 带有@ font-face 的字体。 有些浏览器允许使用跨源的字体，其他浏览器则需要同源的字体。

* 任何带有 <frame> 和 <iframe> 的东西。 网站可以使用 X-Frame-Options 头来阻止这种形式的跨源交互。

以上列表远非完整; 其目标是突出工作中的“最小特权”原则。 浏览器只公开应用程序代码所需的 API 和资源：应用程序提供数据和 URL，浏览器格式化请求并处理每个连接的完整生命周期。

值得注意的是，“同源策略”没有单一概念。相反，有一组相关机制强制限制 DOM 访问，Cookie 和会话状态管理，网络以及浏览器的其他组件。

## 资源和客户状态缓存

最好和最快的请求是未提出的请求。 在分派请求之前，浏览器会自动检查其资源缓存，执行必要的验证检查，并在满足指定条件时返回资源的本地副本。 如果本地资源在高速缓存中不可用，则会发出网络请求，并且响应会自动放入高速缓存中以供后续访问（如果允许）。

* 浏览器自动评估每个资源上的缓存指令

* 如果可能，浏览器会自动重新验证过期资源

* 浏览器会自动管理缓存的大小和清理

管理高效和优化的资源缓存很困难。 值得庆幸的是，浏览器为我们处理了整个复杂性，我们需要做的是确保我们的服务器返回适当的缓存指令; 要了解更多信息，请参阅[客户端上的缓存资源](https://hpbn.co/optimizing-application-delivery/#cache-resources-on-the-client)。你确实为网页上的所有资源提供了Cache-Control，ETag 和 Last-Modified 响应标头，对吗？

最后，浏览器经常被忽视但关键的功能是提供身份验证，会话和cookie管理。 浏览器为每个来源维护单独的“Cookie JAR”，提供必要的应用程序和服务器API来读取和写入新的Cookie，会话和身份验证数据，并自动附加和处理相应的HTTP头以代表我们自动执行整个过程。

### 一个例子：

用一个简单但却形象的例子说明将会话状态管理交给浏览器的便利性：多个 tab 或者浏览器窗口可以共享一个认证过的会话，反之亦然；在一个 tab 中的登出操作可以让其它打开的窗口的会话失效。

## 应用 API 以及协议

走上提供网络服务的阶梯，我们终于到达了应用程序 API 和协议。正如我们所看到的，底层提供了一系列关键服务：套接字和连接管理，请求和响应处理，各种安全策略的执行，缓存等等。每次我们启动一个 HTTP 或一个XMLHttpRequest，一个长期活跃服务器发送的事件或 WebSocket 会话，或打开一个 WebRTC 连接，我们都与这些底层服务的一部分或全部进行交互。

没有单一的最佳协议或 API。每个大型的应用程序都需要根据各种需求混合使用不同的传输：与浏览器缓存的交互，协议开销，消息延迟，可靠性，数据传输类型等等。某些协议可能提供低延迟传输（例如 Server-Sent Events，WebSocket），但可能不符合其他关键条件，例如在所有情况下利用浏览器缓存或支持高效二进制传输的能力。

## 你可以通过几件事来提高你的 Web 应用性能和安全性

* 在你的请求头中总是使用"Connection: Keep-Alive"。浏览器默认会这样设置。确定服务器使用相同的机制。

* 在请求头中使用合适的 Cache-Control, Etag, 以及 Last-Modified 可以帮助你的浏览器节省一些下载时间。

* 花时间调整和优化你的 Web 服务器。 这就是真正的魔法发生的地方！ 请记住，该流程是否针对每个 Web 应用程序以及你要传输的数据的类型都非常具体。

* 始终使用 TLS！ 特别是如果你在应用程序中有任何类型的身份验证。

* 研究浏览器在你的应用程序中提供并实施哪些安全策略。

性能和安全性都是 [SessionStack](https://www.sessionstack.com/?utm_source=medium&utm_medium=blog&utm_content=js-series-networking-layer-outro) 中的一等公民。 我们无法妥协的原因是，一旦 SessionStack 集成到你的 Web 应用程序中，它就会开始监视从 DOM 更改和用户交互到网络请求，未处理的异常和调试消息的所有内容。 所有数据都会实时传输到我们的服务器上，这样你就可以将视频中的问题作为视频重播，并查看用户发生的一切情况。 而这一切都是以最短的延迟进行的，并且不会对应用程序造成任何性能开销。

这就是为什么我们努力采用以上所有建议并且我们将在未来发布的内容中讨论的更多内容。

这是一个免费的计划，你可以[尝试 SessionStack](https://www.sessionstack.com/signup/).

![](http://ozfo4jjxb.bkt.clouddn.com/sessionstack.png)

### 资源

* [https://hpbn.co/](https://hpbn.co/)

* [https://www.amazon.com/Tangled-Web-Securing-Modern-Applications/dp/1593273886](https://www.amazon.com/Tangled-Web-Securing-Modern-Applications/dp/1593273886)

* [https://msdn.microsoft.com/en-us/library/windows/desktop/aa380513(v=vs.85).aspx](https://msdn.microsoft.com/en-us/library/windows/desktop/aa380513(v=vs.85).aspx)

* [http://www.internetlivestats.com/](http://www.internetlivestats.com/)

* [http://vanseodesign.com/web-design/browser-requests/](http://vanseodesign.com/web-design/browser-requests/)
