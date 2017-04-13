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

