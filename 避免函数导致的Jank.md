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

















