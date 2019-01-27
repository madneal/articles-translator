## JavaScript 是如何工作的：渲染引擎以及优化建议

>原文：[How JavaScript works: the rendering engine and tips to optimize its performance](https://blog.sessionstack.com/how-javascript-works-the-rendering-engine-and-tips-to-optimize-its-performance-7b95553baeda)
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

* [Service Workers, their life-cycle, and use cases](https://blog.sessionstack.com/how-javascript-works-service-workers-their-life-cycle-and-use-cases-52b19ad98b58)

* [The mechanics of Web Push Notifications](ttps://blog.sessionstack.com/how-javascript-works-the-mechanics-of-web-push-notifications-290176c5c55d)

* [Tracking changes in the DOM using MutationObserver](https://blog.sessionstack.com/how-javascript-works-tracking-changes-in-the-dom-using-mutationobserver-86adc7446401)

到目前为止，在我们以前的“JavaScript 如何工作”系列博客文章中，我们一直关注 JavaScript 作为一种语言，它的特性，它在浏览器中的执行方式，如何优化它等。

但是，当你构建 Web 应用程序时，你不只是编写独立运行的独立 JavaScript 代码。 你编写的 JavaScript 与环境进行交互。 理解这种环境，它是如何工作的以及它的组成是什么，将使你能够构建更好的应用程序，并对应用程序发布后可能出现的潜在问题做好充分准备。

![](https://cdn-images-1.medium.com/max/2000/1*lMBu87MtEsVFqqbfMum-kA.png)

那么，让我们看看浏览器的主要组件是什么：

* **用户界面**:这包括地址栏，后退和前进按钮，书签菜单等。实质上，这是浏览器显示的每个部分，除了你看到网页本身的窗口。

* **浏览器引擎**: 它处理用户界面和渲染引擎之间的交互。

* **渲染引擎**: 它负责展示网页。 渲染引擎解析 HTML 和 CSS，并在屏幕上展示解析的内容。

* **网络**: 这些是诸如 XHR 请求之类的网络调用，通过对不同的平台使用不同的实现来实现，这些实现在平台无关的接口后面。 在本系列的[前一篇文章](https://blog.sessionstack.com/how-modern-web-browsers-accelerate-performance-the-networking-layer-f6efaf7bfcf4)中，我们更详细地讨论了网络层。

* **UI 后端**: 它用于绘制核心小部件，如复选框和窗口。 这个后端公开了一个不是平台特定的通用接口。 它使用底层的操作系统 UI 方法。

* **JavaScript 引擎**: 我们在该系列的[前一篇文章](https://blog.sessionstack.com/how-javascript-works-inside-the-v8-engine-5-tips-on-how-to-write-optimized-code-ac089e62b12e)中详细介绍了这一点。 基本上，这是 JavaScript 执行的地方。

* **数据持久化**:你的应用可能需要在本地存储所有数据。 支持的存储机制类型包括[localStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)，[indexDB](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_AP)，[WebSQL](https://en.wikipedia.org/wiki/Web_SQL_Database)和 [FileSystem](https://developer.mozilla.org/en-US/docs/Web/API/FileSystem)。

在这篇文章中，我们将关注渲染引擎，因为它处理 HTML 和 CSS 的解析和可视化，这是大多数 JavaScript 应用程序不断与之交互的东西。

### 渲染引擎的概述

渲染引擎的主要职责是在浏览器屏幕上显示请求的页面。

渲染引擎可以显示 HTML 和 XML 文档和图像。 如果你使用额外的插件，引擎还可以显示不同类型的文档，如PDF。

### 渲染引擎

与 JavaScript 引擎类似，不同的浏览器也使用不同的渲染引擎。 这些是一些主流的：

* **Gecko** — Firefox

* **WebKit** — Safari

* **Blink** — Chrome, Opera (from version 15 onwards)

### 渲染的过程

渲染引擎从网络层接收所请求文档的内容。

![下载.png](http://ozfo4jjxb.bkt.clouddn.com/下载.png)

### 构建 DOM 树

渲染引擎的第一步是解析 HTML 文档并将解析的元素转换为 **DOM 树**中的实际 [DOM](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model/Introduction) 节点。

想象一下你有以下的文字输入：

```html
<html>
  <head>
    <meta charset="UTF-8">
    <link rel="stylesheet" type="text/css" href="theme.css">
  </head>
  <body>
    <p> Hello, <span> friend! </span> </p>
    <div> 
      <img src="smiley.gif" alt="Smiley face" height="42" width="42">
    </div>
  </body>
</html>
```

DOM 树看起来应该是这个样子的：

![](https://cdn-images-1.medium.com/max/2000/1*ezFoXqgf91umls9FqO0HsQ.png)


基本上，每个元素都被表示为所有元素的父节点，它们直接包含在它的内部。 这是递归应用的。

### 构建 CSSOM 树

CSSOM 指的是 **CSS 对象模型**。 当浏览器构建页面的 DOM 时，它在 head 中遇到 link 标签的时候会引用外部theme.css CSS 样式表的。 预计它可能需要该资源来呈现页面，它立即发送请求。 假设 theme.css 文件包含以下内容：

```css

body { 
  font-size: 16px;
}

p { 
  font-weight: bold; 
}

span { 
  color: red; 
}

p span { 
  display: none; 
}

img { 
  float: right; 
}css
```



与 HTML 一样，引擎需要将 CSS 转换为浏览器可以使用的东西 - CSSOM。 以下是 CSSOM 树的外观：

![](https://cdn-images-1.medium.com/max/2014/1*5YU1su2mdzHEQ5iDisKUyw.png)


你想知道为什么 CSSOM 有一个树结构？当计算页面上任何对象的最后一组样式时，浏览器从适用于该节点的最通用规则开始（例如，如果它是 body 元素的子元素，则应用所有 body 样式），然后递归地细化通过应用更具体的规则来计算样式。

让我们来看看给出的具体例子。包含在 body 元素中的 span 标签中的任何文本的字体大小为16像素，并且字体颜色是红色。这些样式是从 body 元素继承而来的。如果 span 元素是 p 元素的子元素，则由于正在应用更具体的样式，因此不会显示其内容。

另外请注意，上面的树不是完整的 CSSOM 树，只显示了我们决定在样式表中重写的样式。每个浏览器都提供了一组默认的样式，也称为**“用户代理样式”** - 这是我们在没有明确提供任何样式时看到的。我们的样式简单地覆盖这些默认值。

### 构建渲染树

HTML 中的可视指令与 CSSOM 树中的样式数据结合在一起用于创建渲染树。

你可能会问什么是渲染树？ 这是按照它们在屏幕上显示的顺序构建的视觉元素树。 它是 HTML 和相应的 CSS 的可视化表示。 此树的目的是为了以正确的顺序绘制内容。

渲染树中的每个节点都被称为 Webkit 中的渲染器或渲染对象。

这就是上述 DOM 和 CSSOM 树的渲染器树的外观：

![](https://cdn-images-1.medium.com/max/2000/1*WHR_08AD8APDITQ-4CFDgg.png)



为了构建渲染树，浏览器基本会会做下面的这些事情：

* 从 DOM 树的根节点开始，遍历每个可见节点。 某些节点不可见（例如，脚本标记，元标记等），并且由于它们未反映在呈现的输出中而被忽略。 一些节点通过CSS隐藏，并且也从渲染树中省略。 例如，span 节点 - 在上面的例子中，它并不存在于渲染树中，因为我们有一个明确的规则来设置 display：none 属性

* 对于每一个可见节点，浏览器会找到合适的匹配 CSSDOM 规则并且加以应用。

* 它会给出带有内容及其计算样式的可见节点。

你可以在这里查看 RenderObject 的源代码（在WebKit中）：

[https://github.com/WebKit/webkit/blob/fde57e46b1f8d7dde4b2006aaf7ebe5a09a6984b/Source/WebCore/rendering/RenderObject.h](https://github.com/WebKit/webkit/blob/fde57e46b1f8d7dde4b2006aaf7ebe5a09a6984b/Source/WebCore/rendering/RenderObject.h)

让我们看一下这个类的核心代码：

```javascript
class RenderObject : public CachedImageClient {
  // Repaint the entire object.  Called when, e.g., the color of a border changes, or when a border
  // style changes.
  
  Node* node() const { ... }
  
  RenderStyle* style;  // the computed style
  const RenderStyle& style() const;
  
  ...
}
```



每个渲染器代表一个矩形区域，通常对应于一个节点的 CSS 盒子。 它包括几何信息，例如宽度，高度和位置。

### 渲染树的布局

当渲染器被创建并添加到树中时，它没有位置和大小。 计算这些值称为布局。

HTML 使用基于 flow 的布局模型，这意味着大部分时间内它可以一次性计算几何。 坐标系相对于根渲染器。 使用顶部和左侧坐标。

布局是一个递归过程 - 它从根渲染器开始，它对应于HTML文档的 <html> 元素。 布局通过部分或整个渲染器层次结构递归地继续递归，为需要它的每个渲染器计算几何信息。

根渲染器的位置是0,0，并且其尺寸具有浏览器窗口（也称为视口）的可见部分的尺寸。

开始布局过程意味着给每个节点确切的坐标，它应该出现在屏幕上。

### 绘制渲染树

在此阶段中，遍历渲染器树并调用渲染器的 paint() 方法以在屏幕上显示内容。

绘画可以是全局或增量式（与布局类似）：

* **全局的** — 整个树会被重新绘制。

* **增量的** — 只有一些渲染器以不影响整个树的方式进行更改。 渲染器使其矩形在屏幕上无效。 这会导致操作系统将其视为需要重绘和生成绘画事件的区域。 操作系统通过将多个区域合并为一个智能方式来实现。

一般来说，了解绘制是一个渐进的过程是很重要的。 为了更好的用户体验，渲染引擎会尝试尽快在屏幕上显示内容。 它不会等到所有的 HTML 被解析，才开始构建和布置渲染树。 内容的部分内容将被解析并显示，而该过程继续保持来自网络的其余内容项目。

### 处理脚本和样式表的顺序

当解析器遇到 <script> 标签时，脚本将被立即解析并执行。 文档解析暂停，直到脚本执行完毕。 这意味着该进程是**同步**。

如果脚本是外部的，那么它首先必须从网络获取（也是同步的）。 所有解析都会停止，直到抓取完成。

HTML5 添加了一个选项，将脚本标记为异步，以便它可以被其他线程解析和执行。

### 优化渲染性能


如果你想优化你的应用，那么你需要关注五个主要方面。 这些是你可以控制的区域：

 1. **JavaScript** — 在之前的文章中，我们介绍了优化代码的主题，这些代码不会阻止 UI 渲染，提高内存效率等等。当涉及渲染时，我们需要考虑 JavaScript 代码与页面上 DOM 元素交互的方式。 JavaScript 可以在 UI 中创建大量更改，尤其是在 SPA 中。

 2. **样式计算**—这是确定哪个 CSS 规则适用于基于匹配选择器的元素的过程。 一旦定义了规则，就会应用这些规则，并计算每个元素的最终样式。

 3. **布局** — 一旦浏览器知道哪些规则适用于元素，就可以开始计算后者占用的空间以及它在浏览器屏幕上的位置。 Web 的布局模型定义了一个元素可以影响其他元素。 例如，<body> 的宽度会影响其子元素的宽度等等。 这一切都意味着布局过程是计算密集型的。 该绘图是在多个层次完成的。

 4. **绘制** — 这是实际像素被填充的位置。 该过程包括绘制文本，颜色，图像，边框，阴影等 - 每个元素的每个视觉部分。

 5. **组装** — 由于页面部件被划分为多层，因此需要按照正确的顺序将其绘制到屏幕上，以便页面正确渲染。 这非常重要，特别是对于重叠元素。

### 优化你的 JavaScript

JavaScript 经常触发浏览器中的视觉变化。 当建立一个 SPA 时更是如此。

以下是关于 JavaScript 可以优化哪些部分以改善渲染的一些提示：

* 避免使用 setTimeout 或 setInterval 来进行视觉更新。 这些将在时间轴中的某个点调用回调，可能在最后阶段执行。 我们想要做的就是在画面开始时触发视觉变化，不要错过它。

* 正如我们[之前所讨论的](https://blog.sessionstack.com/how-javascript-works-the-building-blocks-of-web-workers-5-cases-when-you-should-use-them-a547c0757f6a?source=---------3----------------)，将长时间运行的 JavacScript 计算移动到 Web Workers。
* 使用微任务在多个 frame 中引入 DOM 更改。 这是为了防止任务需要访问 DOM，Web Worker 无法访问该 DOM。 这基本上意味着你将一个大任务分解成更小的任务，并根据任务的性质在 requestAnimationFrame，setTimeout，setInterval 中运行它们。

### 优化你的 CSS

通过添加和删除元素，更改属性等来修改DOM将使浏览器重新计算元素样式，并且在很多情况下还会整个页面的布局或至少部分布局。

要优化渲染，请考虑以下几点：

* 减少选择器的复杂性。 选择器的复杂性可能需要计算元素样式所需的时间的50％以上，而构造样式本身的其余工作则需要花费超过50％的时间。

* 减少样式计算必须改变的元素数量。 实质上，直接对几个元素进行样式更改，而不是使整个页面无效。

### 优化布局

浏览器的布局重新计算可能非常繁重。 考虑以下优化：

* 尽可能减少布局的数量。 当你更改样式时，浏览器会检查是否有任何更改要求重新计算布局。 对属性（如宽度，高度，左侧，顶部以及通常与几何相关的属性）的更改需要布局。 所以，尽量避免改变它们。

* 尽可能使用 flexbox 而不是之前的布局模型。因为它运行得更快并且能够为你的应用带来巨大的性能提升。

* 避免强制同步布局。 需要注意的是，在 JavaScript 运行时，前一帧中的所有旧布局值都是已知的，可供您查询。 如果你访问 box.offsetHeight 它不会是一个问题。 但是，如果您在访问该框之前更改了框的样式（例如，通过向该元素动态添加一些CSS类），浏览器必须先应用样式更改并运行布局。 这可能非常耗时且耗费资源，因此请尽可能避免。

**优化绘制**

这通常是所有任务中运行时间最长的，因此尽可能避免这种情况非常重要。 以下是我们可以做的事情：

* 避免更改 transforms 或者 opacity 这些触发绘制的属性。谨慎使用它。

* 如果你触发了一个布局，你将会触发一个绘制，因为改变几何形状会导致元素的视觉变化

* 通过图层提升和动画编排来减少绘制区域。

渲染是[SessionStack](https://www.sessionstack.com/?utm_source=medium&utm_medium=blog&utm_content=js-series-rendering-engine-outro)功能的重要方面。 SessionStack 必须重新创建视频中的所有内容，以便在浏览您的Web应用时遇到问题时发生。 为此，SessionStack 仅利用我们的库收集的数据：用户事件，DOM 更改，网络请求，异常，调试消息等。我们的播放器经过高度优化，能够按顺序正确呈现和使用所有收集的数据 从视觉和技术两方面为您的用户浏览器及其中发生的一切提供像素完美的模拟。

这是一个免费的计划，你可以[尝试 SessionStack](https://www.sessionstack.com/signup/).

![](http://ozfo4jjxb.bkt.clouddn.com/sessionstack.png)

### 资源

* [https://developers.google.com/web/fundamentals/performance/critical-rendering-path/constructing-the-object-model](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/constructing-the-object-model)

* [https://developers.google.com/web/fundamentals/performance/rendering/reduce-the-scope-and-complexity-of-style-calculations](https://developers.google.com/web/fundamentals/performance/rendering/reduce-the-scope-and-complexity-of-style-calculations)

* [https://www.html5rocks.com/en/tutorials/internals/howbrowserswork/#The_parsing_algorithm](https://www.html5rocks.com/en/tutorials/internals/howbrowserswork/#The_parsing_algorithm)
