# Twitter Lite以及大规模的高性能React渐进式网络应用

> 原文：[Twitter Lite and High Performance React Progressive Web Apps at Scale](https://medium.com/@paularmstrong/twitter-lite-and-high-performance-react-progressive-web-apps-at-scale-d28a00e780a3)
>
> 译者：[neal1991](https://github.com/neal1991)

> *让我们一起来了解事件最大的React.js PWA,  [Twitter Lite](https://mobile.twitter.com/)之中常见的和不太常见的性能瓶颈。*

创建一个快速的web应用包含很多方面，包括：时间花费在什么地方，理解其发生的原因并且应用潜在的解决方案。不幸的是，从来就没有一个快速的修复方法。性能是一个持续的问题，涉及到需要对需要提高的内容的持续观察和检测。再Twitter Lite中，我们在很多方面进行了一些小的提升：从出事加载时间搭配React组件的渲染（以及避免再次渲染）到图像的加载等等。大多数的变化往往是非常小的，当时所有的变化叠加在一起让我们开发出了最大的以及最快的[渐进式web应用](https://developers.google.com/web/progressive-web-apps/)。

![timeline](https://cdn-images-1.medium.com/max/1375/1*6f1XFtCP9Ki04onTv-QEHw.png)

## 在继续阅读之前：

如果你才开始观测并且提升你的web应用，我强烈推荐你[学习如何阅读帧图](http://www.brendangregg.com/flamegraphs.html)，如果你还不知道如何去做的话。

下面的每个章节包括例子的 Chrome里面的开发者工具timeline记录的截图。为了让结果更清晰，我强调每一对例子坏的（左图）和好的（右图）进行对比。

对于timeline和帧图特别的笔记：因为我们针对的是很多种的手机设备，我们一般都会在一个模拟的环境中记录这些数据：比5x要慢的CPU以及3G的网络连接。这个不仅更现实，而且还会让问题更容易发现。

经过很多讨论，我们终于通过路由将公共区域分解成独立的块（例子如下）。当我们收件箱收到代码审查的通知的那一天终于来了：

```javascript
const plugins = [
  // 提取vendor和webpack模块的manifest
  new webpack.optimiza.CommonChunkPlugin({
    names: [ 'vendor', 'manifest'],
    minChunks: Infinity
  }),
  // 从所有的块中提取公共模块（不需要'name'属性）
  mew webpack.optimize.CommonChunkPlugin({
    async: true,
  	children: true,
  	minChunks: 4
  })
];
```

> 添加细粒度，基于路由的代码分割。为了加快初始化和主页timeline渲染，app的整体大小可能会更大，文件会在sesiion期间内按需分块在40个代码块之中。--[Nicolas Gallagher](https://medium.com/@necolas)

![bad](https://cloud.githubusercontent.com/assets/12164075/25030441/8f688324-20f7-11e7-8ea0-28c8a664cd17.png)

![good](https://cloud.githubusercontent.com/assets/12164075/25030485/bbc4e6ce-20f7-11e7-94cd-750656dbaac8.png)

我们的原始设置（上面的图）加载我们主要的压缩包花费了超过5秒的时间，然而在通过路由和公共代码块进行代码分割之后（下面的图），就仅仅花费了3秒的时间（在模拟的3G网络中）。

我们在性能优化初期专注完成了这一点，但是这一点变化对于Google的[Lighthouse](https://developers.google.com/web/tools/lighthouse/)这一web应用审查工具的时候得到的结果却有了显著的变化：

![lighthouse](https://cloud.githubusercontent.com/assets/12164075/25030711/6bfb4b18-20f9-11e7-99d1-91e4e4d3648b.png)

## 避免函数导致的Jank

在我们[无限滚动的timeline](http://itsze.ro/blog/2017/04/09/infinite-list-and-react.html)的众多迭代中，我们使用不同的方式来计算你的滚动位置和方向，从而决定我们是否需要API来展示更多的Tweet。直到最近，我们使用了[react-waypoint](https://github.com/brigade/react-waypoint)，这在我们项目中工作的很好。然而，为了尽可能追求我们app的主要基础组件之一的最佳性能，他的速度还不足够快。

Wayponints通过计算很多元素不同的高度，宽度以及位置来决定你现在的滚动位置，以及你距离终点的距离，以及你滚动的方向。所有的这些信息都是有用的，但是因为它是在在每一次滚动事件发生的，因此这是有代价的：带来的计算会造成很多的jank。

但是首先，我们必须明白这意味着什么，如果开发者工具告诉我们这里有一个"jank"。

*大多的设备屏幕每秒会刷新60次。如果有动画或者转换运行，或者用户正在滚动页面，浏览器需要匹配设备的刷新率，并为每个屏幕刷新添加一个新的图片或者帧。*

*这些帧中每一帧花费的时间都超过16ms（1秒/60=16.66ms)。然而，实际上浏览器还有其他的工作，所以所有的工作需要在10ms之内完成。当你不能满足这个要求的话，帧率就会下降，内容在屏幕上的显示就会断断续续。这通常成为jank，它对用户体验会造成负面的影响。--* [Paul Lewis 关于渲染性能](https://developers.google.com/web/fundamentals/performance/rendering/)

之后，我们开发了一个名为VirtualScroller的新的无限滚动的组件。有了这个新的组件，我们确切知道在任何给定时间，什么片段的Tweet被渲染到时间轴上，从而避免在视觉呈现上导致的昂贵的计算。

![jank](https://cloud.githubusercontent.com/assets/12164075/25031297/78997930-20fe-11e7-9c15-93ec722ef6b4.png)

![jank1](https://cloud.githubusercontent.com/assets/12164075/25031311/97df4c52-20fe-11e7-9300-9745f9ab096e.png)

通过避免函数调用带来的额外的jank，滚动Tweet的timeline看起来更加无缝，给我们一种更丰富，更原生的体验。虽然说这可能会更好，但是这种变化对于timeline的滚动的平滑性带来和显著的提升。这个是一个很好的提醒，对于我们研究性能的时候，每一小点都很重要。

## 使用更小的图片

我们开始推动在Twitter Lite上使用较少的带宽，通过和多个团队合作，我们能够在[CDN](https://en.wikipedia.org/wiki/Content_delivery_network)上获得新的更小尺寸的图片。事实证明，通过减小图片的大小，我们只会渲染我们需要展示的（在尺寸和质量方面），我们发现不仅可以减少带宽的使用，而且我们能够提升再浏览器中的性能，特别是在滚动带有大量图片Tweet的timeline的时候。

为了确定的更好的较小图片的性能，我们可以在Chrome开发者工具里面观察光栅timeline。在我们减小图片尺寸之前，对于一张图片的解码需要300ms甚至之上，如下图所示。这是图片下载之后的处理时间，但是是在它在页面展示之前。

当你滚动页面的时候，并且你的目标是60帧/秒的渲染标准时候，我们希望在16.777ms之内尽可能块地处理（1帧）。将一张图片渲染到试图就需要将近18帧，这也太多了。另外需要注意timeline的一点是：你可以看到这个主要的timeline一直都是阻塞的直到图片完成解码（如空白所示）。这意味着我们在这有一个相当大的性能瓶颈！

![image](https://cloud.githubusercontent.com/assets/12164075/25031311/97df4c52-20fe-11e7-9300-9745f9ab096e.png)

![image1](https://cloud.githubusercontent.com/assets/12164075/25046025/b74caedc-2161-11e7-90dd-ea87f609bd05.png)

现在，在我们减少我们图片尺寸之后，我们观测到仅仅需要一帧就可以解码我们最大的图片。

## 优化React

### 使用shouldComponentUpdate方法

对于优化React应用性能一个最常见的建议就是使用[shouldComponentUpdate方法](https://facebook.github.io/react/docs/optimizing-performance.html#shouldcomponentupdate-in-action)。我们尽可能在任何时候都做到这个一点，但是有时候有些东西总是会被遗漏。

![example](https://cloud.githubusercontent.com/assets/12164075/25047300/8306b65c-2168-11e7-91ac-8615e1f4bcd0.gif)

上图中的一个组件总是会进行更新：在主屏timeline的时候点击爱心图标去赞一篇Tweet的时候，任何一个在屏幕上的`Conversation`组件都会重新渲染。在这个动画例子中，你可以看到浏览器需要对被绿色盒子注明的地方进行重绘。因为我们针对的是Tweet下面的整个`Conversation`组件来进行更新action。

如下，你可以看到两个这个action的帧图。上面的没有使用`shouldComponentUpdate`，我们可以看到的它的整个树都被更新和重新渲染，只不过是为了改变屏幕上某个地方的爱心的颜色。在添加`shouldComponentUpdate`（下图）之后，我们阻止了整个树进行更新并且避免了浪费0.1秒来运行不需要的处理。

![shouldComponentUpdate](https://cloud.githubusercontent.com/assets/12164075/25060357/a363ba66-21cd-11e7-92cb-604cd3e7d7cf.png)

![shouldComponentUpdate1](https://cloud.githubusercontent.com/assets/12164075/25060361/dbf3a9a4-21cd-11e7-844e-5281bfb844f8.png)

### 避免不必要的工作直到componentDidMount

这种变化可能看起来是个人都会直到，但是在开发Twitter Lite这样的大型应用的时候，很容易忘记这种小事。

我们发现在我们代码中的很多地方，为了对`componentWillMount`[React周期方法](https://facebook.github.io/react/docs/react-component.html#componentwillmount)进行分析，我们花费大量的时间计算。每一次我们做这个的时候，都会或多或少地阻塞组件的渲染.20ms或者90ms，这些时间很快就累加到一起。最初，我们尝试在tweets被渲染之前（timeline如下）记录哪些是在`componentWillMount`组件中被渲染到我们的数据分析服务中。



























