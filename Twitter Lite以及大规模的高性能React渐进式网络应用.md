# Twitter Lite以及大规模的高性能React渐进式网络应用

> 原文：[Twitter Lite and High Performance React Progressive Web Apps at Scale](https://medium.com/@paularmstrong/twitter-lite-and-high-performance-react-progressive-web-apps-at-scale-d28a00e780a3)
>
> 译者：[neal1991](https://github.com/neal1991)
>
> welcome to star my [articles-translator](https://github.com/neal1991/articles-translator/), providing you advanced articles translation. Any suggestion, please issue or contact [me](mailto:bing@stu.ecnu.edu.cn)
>
> LICENSE: [MIT](https://opensource.org/licenses/MIT)

> *让我们一起来了解世界最大的React.js PWA,  [Twitter Lite](https://mobile.twitter.com/)之中常见的和不太常见的性能瓶颈。*

创建一个快速的web应用包含很多方面，包括：时间花费在什么地方，理解其发生的原因并且应用潜在的解决方案。不幸的是，从来就没有一个快速的修复方法。性能是一个持续的问题，涉及到对需要提高的内容的持续观察和检测。在Twitter Lite中，我们在很多方面进行了一些小的提升：从初始加载时间搭配React组件的渲染（以及避免再次渲染）到图像的加载等等。大多数的变化往往是非常小的，当所有的变化叠加在一起让我们开发出了最大的以及最快的[渐进式web应用](https://developers.google.com/web/progressive-web-apps/)。

![timeline](http://i4.buimg.com/567571/b260f47416854328.png)

## 在继续阅读之前：

如果你才开始观测并且提升你的web应用，我强烈推荐你[学习如何阅读帧图](http://www.brendangregg.com/flamegraphs.html)，如果你还不知道如何去做的话。

下面的每个章节包括例子的 Chrome里面的开发者工具timeline记录的截图。为了让结果更清晰，我强调每一对例子坏的（左图）和好的（右图）进行对比（译者注：因为markdown图片显示的问题，因此原文的左右图在本文中是上图和下图）。

对于timeline和帧图特别的一点：因为我们针对的是很多种的手机设备，我们一般都会在一个模拟的环境中记录这些数据：比5x要慢的CPU以及3G的网络连接。这个不仅更现实，而且还会让问题更容易发现。

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

> 添加细粒度，基于路由的代码分割。为了加快初始化和主页timeline渲染，app的整体大小可能会更大，文件会在session期间内按需分块在40个代码块之中。--[Nicolas Gallagher](https://medium.com/@necolas)

![bad](https://cloud.githubusercontent.com/assets/12164075/25030441/8f688324-20f7-11e7-8ea0-28c8a664cd17.png)

![good](https://cloud.githubusercontent.com/assets/12164075/25030485/bbc4e6ce-20f7-11e7-94cd-750656dbaac8.png)

我们的原始设置（上面的图）加载我们主要的压缩包花费了超过5秒的时间，然而在通过路由和公共代码块进行代码分割之后（下面的图），就仅仅花费了3秒的时间（在模拟的3G网络中）。

我们在性能优化初期专注完成了这一点，但是这一点变化对于Google的[Lighthouse](https://developers.google.com/web/tools/lighthouse/)这一web应用审查工具的时候得到的结果却有了显著的变化：

![lighthouse](https://cloud.githubusercontent.com/assets/12164075/25030711/6bfb4b18-20f9-11e7-99d1-91e4e4d3648b.png)

## 避免函数导致的Jank

在我们[无限滚动的timeline](http://itsze.ro/blog/2017/04/09/infinite-list-and-react.html)的众多迭代中，我们使用不同的方式来计算你的滚动位置和方向，从而决定我们是否需要API来展示更多的Tweet。直到最近，我们使用了[react-waypoint](https://github.com/brigade/react-waypoint)，这在我们项目中工作的很好。然而，为了尽可能追求我们app的主要基础组件之一的最佳性能，它的速度还不足够快。

Wayponints通过计算很多元素不同的高度，宽度以及位置来决定你现在的滚动位置，以及你距离终点的距离，以及你滚动的方向。所有的这些信息都是有用的，但是因为它是在在每一次滚动事件发生的，因此这是有代价的：带来的计算会造成很多的jank。

但是首先，我们必须明白这意味着什么，如果开发者工具告诉我们这里有一个"jank"。

大多的设备屏幕每秒会刷新60次。如果有动画或者转换运行，或者用户正在滚动页面，浏览器需要匹配设备的刷新率，并为每个屏幕刷新添加一个新的图片或者帧。

这些帧中每一帧花费的时间都超过16ms（1秒/60=16.66ms)。然而，实际上浏览器还有其他的工作，所以所有的工作需要在10ms之内完成。当你不能满足这个要求的话，帧率就会下降，内容在屏幕上的显示就会断断续续。这通常成为jank，它对用户体验会造成负面的影响。-- [Paul Lewis 关于渲染性能](https://developers.google.com/web/fundamentals/performance/rendering/)

之后，我们开发了一个名为VirtualScroller的新的无限滚动的组件。有了这个新的组件，我们确切知道在任何给定时间，什么片段的Tweet被渲染到时间轴上，从而避免在视觉呈现上导致的昂贵的计算。

![jank](https://cloud.githubusercontent.com/assets/12164075/25031297/78997930-20fe-11e7-9c15-93ec722ef6b4.png)

![jank1](https://cloud.githubusercontent.com/assets/12164075/25031311/97df4c52-20fe-11e7-9300-9745f9ab096e.png)

通过避免函数调用带来的额外的jank，滚动Tweet的timeline看起来更加无缝，给我们一种更丰富，更原生的体验。虽然说这可能会更好，但是这种变化对于timeline的滚动的平滑性带来和显著的提升。这个是一个很好的提醒，对于我们研究性能的时候，每一小点都很重要。

## 使用更小的图片

我们开始推动在Twitter Lite上使用较少的带宽，通过和多个团队合作，我们能够在[CDN](https://en.wikipedia.org/wiki/Content_delivery_network)上获得新的更小尺寸的图片。事实证明，通过减小图片的大小，我们只会渲染我们需要展示的（在尺寸和质量方面），我们发现不仅可以减少带宽的使用，而且我们能够提升再浏览器中的性能，特别是在滚动带有大量图片Tweet的timeline的时候。

为了确定的更好的较小图片的性能，我们可以在Chrome开发者工具里面观察光栅timeline。在我们减小图片尺寸之前，对于一张图片的解码需要300ms甚至之上，如下图所示。这是图片下载之后的处理时间，但是是在它在页面展示之前。

当你滚动页面的时候，并且你的目标是60帧/秒的渲染标准时候，我们希望在16.777ms之内尽可能块地处理（1帧）。将一张图片渲染到试图就需要将近18帧，这也太多了。另外需要注意timeline的一点是：你可以看到这个主要的timeline一直都是阻塞的直到图片完成解码（如空白所示）。这意味着我们在这有一个相当大的性能瓶颈！

![image](https://cloud.githubusercontent.com/assets/12164075/25045860/e1feedbc-2160-11e7-833e-efd8c9705ad5.png)

![image1](https://cloud.githubusercontent.com/assets/12164075/25046025/b74caedc-2161-11e7-90dd-ea87f609bd05.png)

现在，在我们减少我们图片尺寸之后，我们观测到仅仅需要一帧就可以解码我们最大的图片。

## 优化React

### 使用shouldComponentUpdate方法

对于优化React应用性能一个最常见的建议就是使用[shouldCompoentUpdate方法](https://facebook.github.io/react/docs/optimizing-performance.html#shouldcomponentupdate-in-action)。我们尽可能在任何时候都做到这个一点，但是有时候有些东西总是会被遗漏。

![example](https://cloud.githubusercontent.com/assets/12164075/25047300/8306b65c-2168-11e7-91ac-8615e1f4bcd0.gif)

上图中的一个组件总是会进行更新：在主屏timeline的时候点击爱心图标去赞一篇Tweet的时候，任何一个在屏幕上的`Conversation`组件都会重新渲染。在这个动画例子中，你可以看到浏览器需要对被绿色盒子注明的地方进行重绘。因为我们针对的是Tweet下面的整个`Conversation`组件来进行更新action。

如下，你可以看到两个关于action的帧图。上面的没有使用`shouldComponentUpdate`，我们可以看到的它的整个树都被更新和重新渲染，只不过是为了改变屏幕上某个地方的爱心的颜色。在添加`shouldComponentUpdate`（下图）之后，我们阻止了整个树进行更新并且避免了浪费0.1秒来运行不需要的处理。

![shouldComponentUpdate](https://cloud.githubusercontent.com/assets/12164075/25060357/a363ba66-21cd-11e7-92cb-604cd3e7d7cf.png)

![shouldComponentUpdate1](https://cloud.githubusercontent.com/assets/12164075/25060361/dbf3a9a4-21cd-11e7-844e-5281bfb844f8.png)

### 避免不必要的工作直到componentDidMount

这种变化可能看起来是个人都会知道，但是在开发Twitter Lite这样的大型应用的时候，很容易忘记这种小事。

我们发现在我们代码中的很多地方，为了对`componentWillMount`[React周期方法](https://facebook.github.io/react/docs/react-component.html#componentwillmount)进行分析，我们花费大量的时间计算。每一次我们做这个的时候，都会或多或少地阻塞组件的渲染.20ms或者90ms，这些时间很快就累加到一起。最初，我们尝试在tweets被渲染之前（timeline如下）记录哪些是在`componentWillMount`组件中被渲染到我们的数据分析服务中。

![componentDidMount](https://cloud.githubusercontent.com/assets/12164075/25060519/ea2206ac-21d1-11e7-9623-6397464faa8d.png)

![componentDidMount1](https://cloud.githubusercontent.com/assets/12164075/25060565/f1c762de-21d2-11e7-9a86-52d9ae1ecc77.png)

通过将计算和网络调用移动到React组件的`componentDisMount`方法中，我们将主线程释放出来，并且减少了在渲染组件的时候不想要的jank。

### 避免dangerouslySetInnerHTML

在Twitter Lite中，我们使用SVG图标，因为这对于我们来说最便捷的并且缩放性最好的.不幸的是，在老的React版本中，大多SVG属性在利用组件创建元素的时候是不支持的。因此，当我们开始写这个应用的时候，我们不得不使用`dangerouslySetInnerHTML`在React组件中来使用SVG图标。

比如，我们最初的HeartIcon可能看起来是这个样子的：

```javascript
const HeartIcon = (props) => React.createElement('svg', {
  ...props,
  dangerouslySetInnerHTML: { __html: '<g><path d="M38.723 12c-7.187 0-11.16 7.306-11.723 8.131C26.437 19.306 22.504 12 15.277 12 8.791 12 3.533 18.163 3.533 24.647 3.533 39.964 21.891 55.907 27 56c5.109-.093 23.467-16.036 23.467-31.353C50.467 18.163 45.209 12 38.723 12z"></path></g>' },
  viewBox: '0 0 54 72'
});
```

不仅不鼓励使用`dangerouslySetInnerHTML`，而且事实证明这个正是导致安装和渲染慢的原因。

![svg](https://cloud.githubusercontent.com/assets/12164075/25060826/052d1ea8-21d9-11e7-9908-eaad30c3ecbc.png)

![svg1](https://cloud.githubusercontent.com/assets/12164075/25060705/940e4b0e-21d6-11e7-805f-a6333dd6eb1b.png)

在分析上面的帧图之后，我们最初的代码（上面的）显示需要20ms的时间在一个慢的设备上安装这个action，即Tweet底部的SVG图标。虽然这看起来差别不是很多，但是我们知道我们需要马上渲染，所有这些都在滚动无限的tweet的timeline，我们意识到这会非常的浪费时间。

自从React v15增加了对于大多数SVG属性的支持，我们想在前面来看一下如果我们避免使用`dangerouslySetInnnerHTML`会发生什么。看经过处理后的帧图（下面的），我们在每一次安装和渲染这些图标的时候可以节约60%。

现在，我们的SVG图标是简单的无状态的组件，不会使用“危险的”函数，并且安装速度提升了60%。它们看起来是这个样子的：

```javascript
const HeartIcon = (props = {}) => (
  <svg {...props} viewBox='0 0 ${width} ${height}'>
    <g><path d='M38.723 12c-7.187 0-11.16 7.306-11.723 8.131C26.437 19.306 22.504 12 15.277 12 8.791 12 3.533 18.163 3.533 24.647 3.533 39.964 21.891 55.907 27 56c5.109-.093 23.467-16.036 23.467-31.353C50.467 18.163 45.209 12 38.723 12z'></path></g>
  </svg>
);
```

### 延迟渲染 当安装以及卸载很多组件的时候

在慢的设备上，我们注意到需要很长的时间我们的主浏览条才可以点击，这经常会导致我们点击左慈，假设可能第一次点击并没有注册的话。

注意下图，可以看到主页的图标在点击之后几乎花了2秒的时间来进行更新：

![slow](https://cloud.githubusercontent.com/assets/12164075/25060886/ef8a172a-21da-11e7-8b48-080b5e90acc1.gif)

不，这并不是GIF在一个低的帧率下运行。事实上就是慢。但是，所有的主页屏幕的数据实际上已经加载了，为什么需要花费这么多的时间来显示呢？

事实证明安装和卸载大型组件树（比如Tweet的timeline）在React中是非常耗时的。

至少，我们希望去除掉这种点击导航栏之后没有响应的感觉。为此，我们创建了一个小型的高阶组件：

```javascript
import hoistStatics from 'hoist-non-react-statics';
import React from 'react';

/**
 * Allows two animation frames to complete to allow other components to update
 * and re-render before mounting and rendering an expensive `WrappedComponent`.
 */
export default function deferComponentRender(WrappedComponent) {
  class DeferredRenderWrapper extends React.Component {
    constructor(props, context) {
      super(props, context);
      this.state = { shouldRender: false };
    }

    componentDidMount() {
      window.requestAnimationFrame(() => {
        window.requestAnimationFrame(() => this.setState({ shouldRender: true }));
      });
    }

    render() {
      return this.state.shouldRender ? <WrappedComponent {...this.props} /> : null;
    }
  }

  return hoistStatics(DeferredRenderWrapper, WrappedComponent);
}
```

在应用到我们的主页timeline之后，我们可以看到导航栏一个比较快的响应速度，从而带来感官上一个大的提升。

```javascript
const DeferredTimeline = deferComponentRender(HomeTimeline);
render(<DeferredTimeline />);
```

![quick](https://cloud.githubusercontent.com/assets/12164075/25061012/c30c7ca2-21de-11e7-9b84-7bbcb0e02dbc.gif)

## 优化Redux

### 避免存储State太频繁

尽管[controlled components](https://facebook.github.io/react/docs/forms.html#controlled-components) 被推荐使用，但是这也意味这每一次按键之后都需要进行更新并且重新渲染。

尽管这对于一个3GHZ的台式机来说并不太难，但对于CPU数量有限的小型移动设备来说影响确实很大的，尤其是从input中删除多个字符的时候。

为了保持正在撰写Tweet的值的时候同时计算剩余字符的数量，我们使用controlled组件，并将输入的当前值传递到每个按键的Redux state。

如下（上面的），在一个典型的安卓5的设备上，每一次按键导致的更改可能会需要将近200ms的开销。这对于快速打字的人来说是很痛苦的，这也让我们最终陷入了一个非常糟糕的状态，用户经常投诉他们的字符插入会移动到各个地方，从而导致混乱的句子。

![redux](https://cloud.githubusercontent.com/assets/12164075/25061021/27c236a0-21df-11e7-9292-d44509b0301f.png)

![qucik](https://cloud.githubusercontent.com/assets/12164075/25061377/6e9fba9a-21e7-11e7-8244-ff223a701bbf.png)

通过移除每一次按键下更新主Redux state的Tweet草稿state并且在本地保留Redux组件的状态，可以将开销减少50%。

### 将批量Actions合并成一个Dispatch

在Twitter Lite中，我们使用[react-redux](http://redux.js.org/docs/basics/UsageWithReact.html) 配合[redux](http://redux.js.org/)来订阅我们组件的数据状态变化。我们通过使用 [Normalizr](https://github.com/paularmstrong/normalizr) 以及 [combineReducers](http://redux.js.org/docs/api/combineReducers.html)将大型的store来进行分割从而对我们的数据进一步优化。这些都工作的很好，避免了数据重复并且保持我们的store足够小。然而，每一次我们拿到新的数据的时候，我们必须分发多个action为了将它添加到合适的store中。

通过react-redux，这意味分派每个action都会导致我们连接的组件（成为容器）重新计算更改并且可能重新渲染。

尽管我们使用了一个自定义的middleware，还有其它的[批量middleware](https://www.npmjs.com/package/redux-batch-enhancer)。选择一个适合你的，或者你自己写一个。

说明使用批量action好处的最好方式是使用Chrome React Perf拓展。在初始加载之后，我们在后台pre-cache并且计算围堵的DM。当这种情况发生的时候，我们添加了很多不同的实体（会话，用户，消息条目等等）。如果没有批量action的时候（下面的），你可以看到我们渲染组件的时间相对于批量action时候的时间对比是~16ms对~8ms。

![action](https://cloud.githubusercontent.com/assets/12164075/25061599/c03abddc-21ec-11e7-92f8-bbceb774bc55.png)

![action1](https://cloud.githubusercontent.com/assets/12164075/25061913/2b8f4d5e-21f3-11e7-883a-25af266eb6e1.png)

## Service Workers

尽管Service Worker并没有在所有的浏览器得到支持，但是它还是Twitter Lite中非常重要的一个部分。当Service Worker被支持的时候，我们用它做推送，预先缓存应用资源以及其它。不幸的是，作为一个想当新的技术，还有很多性能提升方面的东西需要学习。

### 预先缓存资源

和大多数产品一样，Twitter Lite还远远没有完成。我们仍然在积极地开发它，添加新特性并且修复BUG以及让它运行得更快。这意味着我们经常需要部署新版本的JavaScript资源。

不幸的是，这对于返回应用的用户来说是一个负担，因为他们需要重新下载一大堆脚本文件仅仅是为了浏览一个Tweet。

在ServiceWorker被支持的浏览器中，worker能够在你返回之前自己在后台自动更新，下载并且缓存任何改变的文件，我们从中获益。

因此这对用户意味着什么？几乎是马上就可以加载应用，即使再在我们部署新版本之后！

![deploy](https://cloud.githubusercontent.com/assets/12164075/25061915/5305fe00-21f3-11e7-9368-00573e76f967.png)![1-r51dWgMIwA8-jFKno0_RcQ.png](https://ooo.0o0.ooo/2017/04/15/58f1d99c2f24f.png)

在上面展示的（上面的）是没有使用ServiceWorker预先缓存资源，当前view中每一个资源都会强制从网络中加载如果返回应用的时候。在3G的网络环境下差不多需要6秒的时间来完成加载。然而，当资源被ServiceWorker预先缓存之后（下面的），同样在3G网络环境下只需要1.5秒就可以完成页面的加载。75%的提升！

### 延迟ServiceWorker注册

在很多应用中，在页面加载的时候立即注册ServiceWorker是安全的：

```javascript
<script>
window.navigator.serviceWorker.register('/sw.js');
</script>
```

当我们发送尽可能多的数据到浏览器来渲染一个看起来比较完整的页面，但是在Twitter Lite情况不一定就是这样的。我们可能不会发送足够的数据，或者你打开的页面不会支持数据从服务器全部获取。因为这种或者那种的很多限制，我们需要在初始页面加载之后立刻发送一些API请求。

正常情况，这不会是一个问题。然而，如果浏览器还没有安装当前版本的ServiceWorker，我们需要让它安装，随之而来就是对于JS，CSS以及图片资源预缓存的50个请求。

当我们使用简单的方法来立即注册ServiceWorker的时候，我们可以看到网络连接发生在浏览器中，达到了并行请求的限制（上面的）。

![1-jLT8J20RRfKY_oxcsAiswQ.png](https://ooo.0o0.ooo/2017/04/15/58f1d9e6a3d09.png)

![1-poli81EjMdim8__g1HMioQ.png](https://ooo.0o0.ooo/2017/04/15/58f1da4a07318.png)

通过延迟ServiceWorker的注册直到我们完成了额外的API，CSS以及图片资源的请求，我们可以让页面完成渲染并且是响应式的，在下面的截图可以看到（下面的）。

总的来说，这是我们在[Twitter Lite](https://mobile.twitter.com/)的开发过程中的众多性能提升的列表。当然还会有更多的事情，我们希望能够继续分享我们发现的问题，以及我们克服问题所做的工作。关于事实以及耕作React以及PWA的更多状态，可以在Twitter上关注[我](https://mobile.twitter.com/paularmstrong)以及[我的团队](https://mobile.twitter.com/paularmstrong/lists/twitter-lite/members)。































