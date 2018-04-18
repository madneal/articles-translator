## JavaScript 是如何工作的以及优化建议

>原文：[How JavaScript works: the rendering engine and tips to optimize its performance](https://blog.sessionstack.com/how-javascript-works-the-rendering-engine-and-tips-to-optimize-its-performance-7b95553baeda)
>
>译者：[neal1991](https://github.com/neal1991)
>
>welcome to star my [articles-translator ](https://github.com/neal1991), providing you advanced articles translation. Any suggestion, please issue or contact [me](mailto:bing@stu.ecnu.edu.cn)
>
>LICENSE: [MIT](https://opensource.org/licenses/MIT)

This is post # 11 of the series dedicated to exploring JavaScript and its building components. In the process of identifying and describing the core elements, we also share some rules of thumb we use when building [SessionStack](https://www.sessionstack.com/?utm_source=medium&utm_medium=blog&utm_content=js-series-rendering-engine-intro), a JavaScript application that needs to be robust and highly-performant to help users see and reproduce their web app defects real-time.

这是专门探索 JavaScript 及其构建组件的系列 #11。 在识别和描述核心元素的过程中，我们也分享了我们在构建[SessionStack](https://www.sessionstack.com/?utm_source=medium&utm_medium=blog&utm_content=js-series-rendering-engine -intro)时使用的一些经验法则，SessionStack 是一款 JavaScript 应用程序，需要强大且高性能的功能来帮助用户实时查看和重现其 Web 应用程序缺陷。

If you missed the previous chapters, you can find them here:

如果你错过了之前的章节，你可以从这找到他们：

* [An overview of the engine, the runtime, and the call stack](https://blog.sessionstack.com/how-does-javascript-actually-work-part-1-b0bacc073cf?source=collection_home---2------1----------------)

* [Inside Google’s V8 engine + 5 tips on how to write optimized code](https://blog.sessionstack.com/how-javascript-works-inside-the-v8-engine-5-tips-on-how-to-write-optimized-code-ac089e62b12e?source=collection_home---2------2----------------)

* [Memory management + how to handle 4 common memory leaks](https://blog.sessionstack.com/how-javascript-works-memory-management-how-to-handle-4-common-memory-leaks-3f28b94cfbec?source=collection_home---2------0----------------)

* [The event loop and the rise of Async programming + 5 ways to better coding with async/await](https://blog.sessionstack.com/how-javascript-works-event-loop-and-the-rise-of-async-programming-5-ways-to-better-coding-with-2f077c4438b5)

* [Deep dive into WebSockets and HTTP/2 with SSE + how to pick the right path](https://blog.sessionstack.com/how-javascript-works-deep-dive-into-websockets-and-http-2-with-sse-how-to-pick-the-right-path-584e6b8e3bf7?source=collection_home---4------0----------------)

* [A comparison with WebAssembly + why in certain cases it’s better to use it over JavaScript](https://blog.sessionstack.com/how-javascript-works-a-comparison-with-webassembly-why-in-certain-cases-its-better-to-use-it-d80945172d79)

* [The building blocks of Web Workers + 5 cases when you should use them](https://blog.sessionstack.com/how-javascript-works-the-building-blocks-of-web-workers-5-cases-when-you-should-use-them-a547c0757f6a)

* [Service Workers, their life-cycle, and use cases](https://blog.sessionstack.com/how-javascript-works-service-workers-their-life-cycle-and-use-cases-52b19ad98b58)

* [The mechanics of Web Push Notifications](ttps://blog.sessionstack.com/how-javascript-works-the-mechanics-of-web-push-notifications-290176c5c55d)

* [Tracking changes in the DOM using MutationObserver](https://blog.sessionstack.com/how-javascript-works-tracking-changes-in-the-dom-using-mutationobserver-86adc7446401)

So far, in our previous blog posts of the “How JavaScript works” series we’ve been focusing on JavaScript as a language, its features, how it gets executed in the browser, how to optimize it, etc.

When you’re building web apps, however, you don’t just write isolated JavaScript code that runs on its own. The JavaScript you write is interacting with the environment. Understanding this environment, how it works and what it is composed of will allow you to build better apps and be well-prepared for potential issues that might arise once your apps are released into the wild.

到目前为止，在我们以前的“JavaScript 如何工作”系列博客文章中，我们一直关注 JavaScript 作为一种语言，它的特性，它在浏览器中的执行方式，如何优化它等。

但是，当你构建 Web 应用程序时，你不只是编写独立运行的独立 JavaScript 代码。 你编写的 JavaScript 与环境进行交互。 理解这种环境，它是如何工作的以及它的组成是什么，将使你能够构建更好的应用程序，并对应用程序发布后可能出现的潜在问题做好充分准备。

![](https://cdn-images-1.medium.com/max/2000/1*lMBu87MtEsVFqqbfMum-kA.png)

So, let’s see what the browser main components are:

那么，让我们看看浏览器的主要组件是什么：

* **用户界面**: this includes the address bar, the back and forward buttons, bookmarking menu, etc. In essence, this is every part of the browser display except for the window where you see the web page itself.这包括地址栏，后退和前进按钮，书签菜单等。实质上，这是浏览器显示的每个部分，除了你看到网页本身的窗口。

* **浏览器引擎**: it handles the interactions between the user interface and the rendering engine它处理用户界面和渲染引擎之间的交互

* **渲染引擎**: it’s responsible for displaying the web page. The rendering engine parses the HTML and the CSS and displays the parsed content on the screen.它负责展示网页。 渲染引擎解析HTML和CSS，并在屏幕上展示解析的内容。

* **网络**: these are network calls such as XHR requests, made by using different implementations for the different platforms, which are behind a platform-independent interface. We talked about the networking layer in more detail in a [previous post](https://blog.sessionstack.com/how-modern-web-browsers-accelerate-performance-the-networking-layer-f6efaf7bfcf4) of this series.这些是诸如 XHR 请求之类的网络调用，通过对不同的平台使用不同的实现来实现，这些实现在平台无关的接口后面。 在本系列的[前一篇文章](https://blog.sessionstack.com/how-modern-web-browsers-accelerate-performance-the-networking-layer-f6efaf7bfcf4)中，我们更详细地讨论了网络层。

* **UI 后端**: it’s used for drawing the core widgets such as checkboxes and windows. This backend exposes a generic interface that is not platform-specific. It uses operating system UI methods underneath.它用于绘制核心小部件，如复选框和窗口。 这个后端公开了一个不是平台特定的通用接口。 它使用底层的操作系统 UI 方法。

* **JavaScript 引擎**: We’ve covered this in great detail in a [previous post](https://blog.sessionstack.com/how-javascript-works-inside-the-v8-engine-5-tips-on-how-to-write-optimized-code-ac089e62b12e) from the series. Basically, this is where the JavaScript gets executed.我们在该系列的[前一篇文章]((https://blog.sessionstack.com/how-javascript-works-inside-the-v8-engine-5-tips-on-how-to-write-optimized-code-ac089e62b12e)中详细介绍了这一点。 基本上，这是 JavaScript 执行的地方。

* **数据持久化**: your app might need to store all data locally. The supported types of storage mechanisms include [localStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage), [indexDB](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API), [WebSQL](https://en.wikipedia.org/wiki/Web_SQL_Database) and [FileSystem](https://developer.mozilla.org/en-US/docs/Web/API/FileSystem).你的应用可能需要在本地存储所有数据。 支持的存储机制类型包括[localStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)，[indexDB](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_AP)，[WebSQL](https://en.wikipedia.org/wiki/Web_SQL_Database)和 [FileSystem](https://developer.mozilla.org/en-US/docs/Web/API/FileSystem)。

In this post, we’re going to focus on the rendering engine, since it’s handling the parsing and the visualization of the HTML and the CSS, which is something that most JavaScript apps are constantly interacting with.

在这篇文章中，我们将关注渲染引擎，因为它处理 HTML 和 CSS 的解析和可视化，这是大多数 JavaScript 应用程序不断与之交互的东西。

### 渲染引擎的概述

The main responsibility of the rendering engine is to display the requested page on the browser screen.

Rendering engines can display HTML and XML documents and images. If you’re using additional plugins, the engines can also display different types of documents such as PDF.

渲染引擎的主要职责是在浏览器屏幕上显示请求的页面。

渲染引擎可以显示 HTML 和 XML 文档和图像。 如果你使用额外的插件，引擎还可以显示不同类型的文档，如PDF。

### 渲染引擎

Similar to the JavaScript engines, different browsers use different rendering engines as well. These are some of the popular ones:

与 JavaScript 引擎类似，不同的浏览器也使用不同的渲染引擎。 这些是一些流行的：

* **Gecko** — Firefox

* **WebKit** — Safari

* **Blink** — Chrome, Opera (from version 15 onwards)

### 渲染的过程

The rendering engine receives the contents of the requested document from the networking layer.

渲染引擎从网络层接收所请求文档的内容。

![下载.png](http://ozfo4jjxb.bkt.clouddn.com/下载.png)

![](https://cdn-images-1.medium.com/max/2002/1*9b1uEMcZLWuGPuYcIn7ZXQ.png)

### 构建 DOM 树

The first step of the rendering engine is parsing the HTML document and converting the parsed elements to actual [DOM](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model/Introduction) nodes in a **DOM tree**.

Imagine you have the following textual input:

 <iframe src="https://medium.com/media/c8c8dce0c5ef0f9485948af1f23c1726" frameborder=0></iframe>

The DOM tree for this HTML will look like this:

![](https://cdn-images-1.medium.com/max/2000/1*ezFoXqgf91umls9FqO0HsQ.png)

Basically, each element is represented as the parent node to all of the elements, which are directly contained inside of it. And this is applied recursively.

### Constructing the CSSOM tree

CSSOM refers to the **CSS Object Model**. While the browser was constructing the DOM of the page, it encountered a link tag in the head section which was referencing the external theme.css CSS style sheet. Anticipating that it might need that resource to render the page, it immediately dispatched a request for it. Let’s imagine that the theme.css file has the following contents:

 <iframe src="https://medium.com/media/24b0764e3c928d5156b60ddbdd5763af" frameborder=0></iframe>

As with the HTML, the engine needs to convert the CSS into something that the browser can work with — the CSSOM. Here is how the CSSOM tree will look like:

![](https://cdn-images-1.medium.com/max/2014/1*5YU1su2mdzHEQ5iDisKUyw.png)

Do you wonder why does the CSSOM have a tree structure? When computing the final set of styles for any object on the page, the browser starts with the most general rule applicable to that node (for example, if it is a child of a body element, then all body styles apply) and then recursively refines the computed styles by applying more specific rules.

Let’s work with the specific example that we gave. Any text contained within a span tag that is placed within the body element, has a font size of 16 pixels and has a red color. Those styles are inherited from the body element. If a span element is a child of a p element, then its contents are not displayed due to the more specific styles that are being applied to it.

Also, note that the above tree is not the complete CSSOM tree and only shows the styles we decided to override in our style sheet. Every browser provides a default set of styles also known as **“user agent styles”** — that’s what we see when we don’t explicitly provide any. Our styles simply override these defaults.

### Constructing the render tree

The visual instructions in the HTML, combined with the styling data from the CSSOM tree, are being used to create a **render tree**.

What is a render tree you may ask? This is a tree of the visual elements constructed in the order in which they will be displayed on the screen. It is the visual representation of the HTML along with the corresponding CSS. The purpose of this tree is to enable painting the contents in their correct order.

Each node in the render tree is known as a renderer or a render object in Webkit.

This is how the renderer tree of the above DOM and CSSOM trees will look like:

![](https://cdn-images-1.medium.com/max/2000/1*WHR_08AD8APDITQ-4CFDgg.png)

To construct the render tree, the browser does roughly the following:

* Starting at the root of the DOM tree, it traverses each visible node. Some nodes are not visible (for example, script tags, meta tags, and so on), and are omitted since they are not reflected in the rendered output. Some nodes are hidden via CSS and are also omitted from the render tree. For example, the span node — in the example above it’s not present in the render tree because we have an explicit rule that sets the display: none property on it.

* For each visible node, the browser finds the appropriate matching CSSOM rules and applies them.

* It emits visible nodes with content and their computed styles

You can take a look at the RenderObject’s source code (in WebKit) here: [https://github.com/WebKit/webkit/blob/fde57e46b1f8d7dde4b2006aaf7ebe5a09a6984b/Source/WebCore/rendering/RenderObject.h](https://github.com/WebKit/webkit/blob/fde57e46b1f8d7dde4b2006aaf7ebe5a09a6984b/Source/WebCore/rendering/RenderObject.h)

Let’s just look at some of the core things for this class:

 <iframe src="https://medium.com/media/2b55c74b1624ef2333014926f78c88dd" frameborder=0></iframe>

Each renderer represents a rectangular area usually corresponding to a node’s CSS box. It includes geometric info such as width, height, and position.

### Layout of the render tree

When the renderer is created and added to the tree, it does not have a position and size. Calculating these values is called layout.

HTML uses a flow-based layout model, meaning that most of the time it can compute the geometry in a single pass. The coordinate system is relative to the root renderer. Top and left coordinates are used.

Layout is a recursive process — it begins at the root renderer, which corresponds to the <html> element of the HTML document. Layout continues recursively through a part or the entire renderer hierarchy, computing geometric info for each renderer that requires it.

The position of the root renderer is 0,0 and its dimensions have the size of the visible part of the browser window (a.k.a. the viewport).

Starting the layout process means giving each node the exact coordinates where it should appear on the screen.

### Painting the render tree

In this stage, the renderer tree is traversed and the renderer’s paint() method is called to display the content on the screen.

Painting can be global or incremental (similar to layout):

* **Global** — the entire tree gets repainted.

* **Incremental** — only some of the renderers change in a way that does not affect the entire tree. The renderer invalidates its rectangle on the screen. This causes the OS to see it as a region that needs repainting and to generate a paint event. The OS does it in a smart way by merging several regions into one.

In general, it’s important to understand that painting is a gradual process. For better UX, the rendering engine will try to display the contents on the screen as soon as possible. It will not wait until all the HTML is parsed to start building and laying out the render tree. Parts of the content will be parsed and displayed, while the process continues with the rest of the content items that keep coming from the network.

### 处理脚本和样式表的顺序

Scripts are parsed and executed immediately when the parser reaches a <script> tag. The parsing of the document halts until the script has been executed. This means that the process is **synchronous**.

If the script is external then it first has to be fetched from the network (also synchronously). All the parsing stops until the fetch completes.

HTML5 adds an option to mark the script as asynchronous so that it gets parsed and executed by a different thread.

当解析器遇到 <script> 标签时，脚本将被立即解析并执行。 文档解析暂停，直到脚本执行完毕。 这意味着该进程是**同步**。

如果脚本是外部的，那么它首先必须从网络获取（也是同步的）。 所有解析都会停止，直到抓取完成。

HTML5 添加了一个选项，将脚本标记为异步，以便它可以被其他线程解析和执行。

### 优化渲染性能

If you’d like to optimize your app, there are five major areas that you need to focus on. These are the areas over which you have control:
如果你想优化你的应用，那么你需要关注五个主要方面。 这些是你可以控制的区域：

 1. **JavaScript** — in previous posts we covered the topic of writing optimized code that doesn’t block the UI, is memory efficient, etc. When it comes to rendering, we need to think about the way your JavaScript code will interact with the DOM elements on the page. JavaScript can create lots of changes in the UI, especially in SPAs.在之前的文章中，我们介绍了优化代码的主题，这些代码不会阻止 UI 渲染，提高内存效率等等。当涉及渲染时，我们需要考虑 JavaScript 代码与页面上 DOM 元素交互的方式。 JavaScript 可以在 UI 中创建大量更改，尤其是在 SPA 中。

 2. **样式计算**— this is the process of determining which CSS rule applies to which element based on matching selectors. Once the rules are defined, they are applied and the final styles for each element are calculated.这是确定哪个 CSS 规则适用于基于匹配选择器的元素的过程。 一旦定义了规则，就会应用这些规则，并计算每个元素的最终样式。

 3. **布局** — once the browser knows which rules apply to an element, it can begin to calculate how much space the latter takes up and where it is located on the browser screen. The web’s layout model defines that one element can affect others. For example, the width of the <body> can affect the width of its children and so on. This all means that the layout process is computationally intensive. The drawing is done in multiple layers.一旦浏览器知道哪些规则适用于元素，就可以开始计算后者占用的空间以及它在浏览器屏幕上的位置。 Web 的布局模型定义了一个元素可以影响其他元素。 例如，<body> 的宽度会影响其子元素的宽度等等。 这一切都意味着布局过程是计算密集型的。 该绘图是在多个层次完成的。

 4. **绘制** — this is where the actual pixels are being filled. The process includes drawing out text, colors, images, borders, shadows, etc. — every visual part of each element.这是实际像素被填充的位置。 该过程包括绘制文本，颜色，图像，边框，阴影等 - 每个元素的每个视觉部分。

 5. **组装** — since the page parts were drawn into potentially multiple layers they need to be drawn onto the screen in the correct order so that the page renders properly. This is very important, especially for overlapping elements.由于页面部件被划分为多层，因此需要按照正确的顺序将其绘制到屏幕上，以便页面正确渲染。 这非常重要，特别是对于重叠元素。

### 优化你的 JavaScript

JavaScript often triggers visual changes in the browser. All the more so when building an SPA.

Here are a few tips on which parts of your JavaScript you can optimize to improve rendering:

JavaScript 经常触发浏览器中的视觉变化。 当建立一个 SPA 时更是如此。

以下是关于 JavaScript 可以优化哪些部分以改善渲染的一些提示：

* Avoid setTimeout or setInterval for visual updates. These will invoke the callback at some point in the frame, possible right at the end. What we want to do is trigger the visual change right at the start of the frame not to miss it.避免使用 setTimeout 或 setInterval 来进行视觉更新。 这些将在时间轴中的某个点调用回调，可能在最后阶段执行。 我们想要做的就是在画面开始时触发视觉变化，不要错过它。

* Move long-running JavaScript computations to Web Workers as we have [previously discussed](https://blog.sessionstack.com/how-javascript-works-the-building-blocks-of-web-workers-5-cases-when-you-should-use-them-a547c0757f6a?source=---------3----------------).正如我们[之前所讨论的](https://blog.sessionstack.com/how-javascript-works-the-building-blocks-of-web-workers-5-cases-when-you-should-use-them-a547c0757f6a?source=---------3----------------)，将长时间运行的 JavacScript 计算移动到 Web Workers。
* Use micro-tasks to introduce DOM changes over several frames. This is in case the tasks need access to the DOM, which is not accessible by Web Workers. This basically means that you’d break up a big task into smaller ones and run them inside requestAnimationFrame , setTimeout, setInterval depending on the nature of the task.使用微任务在多个 frame 中引入 DOM 更改。 这是为了防止任务需要访问 DOM，Web Worker 无法访问该 DOM。 这基本上意味着你将一个大任务分解成更小的任务，并根据任务的性质在 requestAnimationFrame，setTimeout，setInterval 中运行它们。

### 优化你的 CSS

Modifying the DOM through adding and removing elements, changing attributes, etc. will make the browser recalculate element styles and, in many cases, the layout of the entire page or at least parts of it.

To optimize the rendering, consider the following:

通过添加和删除元素，更改属性等来修改DOM将使浏览器重新计算元素样式，并且在很多情况下还会整个页面的布局或至少部分布局。

要优化渲染，请考虑以下几点：

* Reduce the complexity of your selectors. Selector complexity can take more than 50% of the time needed to calculate the styles for an element, compared to the rest of the work which is constructing the style itself.减少选择器的复杂性。 选择器的复杂性可能需要计算元素样式所需的时间的50％以上，而构造样式本身的其余工作则需要花费超过50％的时间。

* Reduce the number of elements on which style calculation must happen. In essence, make style changes to a few elements directly rather than invalidating the page as a whole.减少样式计算必须改变的元素数量。 实质上，直接对几个元素进行样式更改，而不是使整个页面无效。

### 优化布局

Layout re-calculations can be very heavy for the browser. Consider the following optimizations:

浏览器的布局重新计算可能非常繁重。 考虑以下优化：

* Reduce the number of layouts whenever possible. When you change styles the browser checks to see if any of the changes require the layout to be re-calculated. Changes to properties such as width, height, left, top, and in general, properties related to geometry, require layout. So, avoid changing them as much as possible.尽可能减少布局的数量。 当您更改样式时，浏览器会检查是否有任何更改要求重新计算布局。 对属性（如宽度，高度，左侧，顶部以及通常与几何相关的属性）的更改需要布局。 所以，尽量避免改变它们。

* Use flexbox over older layout models whenever possible. It works faster and can create a huge performance advantage for your app.尽可能使用 flexbox 而不是之前的布局模型。因为它运行得更快并且能够为你的应用带来巨大的性能提升。

* Avoid forced synchronous layouts. The thing to keep in mind is that while JavaScript runs, all the old layout values from the previous frame are known and available for you to query. If you access box.offsetHeight it won’t be an issue. If you, however, change the styles of the box before it’s accessed (e.g. by dynamically adding some CSS class to the element), the browser will have to first apply the style change and then run the layout. This can be very time-consuming and resource-intensive, so avoid it whenever possible.避免强制同步布局。 需要注意的是，在 JavaScript 运行时，前一帧中的所有旧布局值都是已知的，可供您查询。 如果你访问 box.offsetHeight 它不会是一个问题。 但是，如果您在访问该框之前更改了框的样式（例如，通过向该元素动态添加一些CSS类），浏览器必须先应用样式更改并运行布局。 这可能非常耗时且耗费资源，因此请尽可能避免。

**优化绘制**

This often is the longest-running of all the tasks so it’s important to avoid it as much as possible. Here is what we can do:

这通常是所有任务中运行时间最长的，因此尽可能避免这种情况非常重要。 以下是我们可以做的事情：

* Changing any property other than transforms or opacity triggers a paint. Use it sparingly.避免更改 transforms 或者 opacity 这些触发绘制的属性。谨慎使用它。

* If you trigger a layout, you will also trigger a paint, since changing the geometry results in a visual change of the element.如果你触发了一个布局，你将会触发一个绘制，因为改变几何形状会导致元素的视觉变化

* Reduce paint areas through layer promotion and orchestration of animations.通过图层提升和动画编排来减少绘制区域。

Rendering is a vital aspect of how [SessionStack ](https://www.sessionstack.com/?utm_source=medium&utm_medium=blog&utm_content=js-series-rendering-engine-outro)functions. SessionStack has to recreate as a video everything that happened to your users at the time they experienced an issue while browsing your web app. To do this, SessionStack leverages only the data that was collected by our library: user events, DOM changes, network requests, exceptions, debug messages, etc. Our player is highly optimized to properly render and make use of all the collected data in order to offer a pixel-perfect simulation of your users’ browser and everything that happened in it, both visually and technically.

渲染是[SessionStack](https://www.sessionstack.com/?utm_source=medium&utm_medium=blog&utm_content=js-series-rendering-engine-outro)功能的重要方面。 SessionStack 必须重新创建视频中的所有内容，以便在浏览您的Web应用时遇到问题时发生。 为此，SessionStack 仅利用我们的库收集的数据：用户事件，DOM 更改，网络请求，异常，调试消息等。我们的播放器经过高度优化，能够按顺序正确呈现和使用所有收集的数据 从视觉和技术两方面为您的用户浏览器及其中发生的一切提供像素完美的模拟。

这是一个免费的计划，你可以[尝试 SessionStack](https://www.sessionstack.com/signup/).

![](https://cdn-images-1.medium.com/max/NaN/0*h2Z_BnDiWfVhgcEZ.)

### 资源

* [https://developers.google.com/web/fundamentals/performance/critical-rendering-path/constructing-the-object-model](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/constructing-the-object-model)

* [https://developers.google.com/web/fundamentals/performance/rendering/reduce-the-scope-and-complexity-of-style-calculations](https://developers.google.com/web/fundamentals/performance/rendering/reduce-the-scope-and-complexity-of-style-calculations)

* [https://www.html5rocks.com/en/tutorials/internals/howbrowserswork/#The_parsing_algorithm](https://www.html5rocks.com/en/tutorials/internals/howbrowserswork/#The_parsing_algorithm)
