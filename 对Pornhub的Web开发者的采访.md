# Pornhub Web 开发者访谈

>原文：[Interview with a Pornhub Web Developer](https://davidwalsh.name/pornhub-interview)
>
>译者：[neal1991](https://github.com/neal1991)
>
>welcome to star my [articles-translator](https://github.com/neal1991/articles-translator/), providing you advanced articles translation. Any suggestion, please issue or contact [me](mailto:bing@stu.ecnu.edu.cn)
>
>LICENSE: [MIT](https://opensource.org/licenses/MIT)

Regardless of your stance on pornography, it would be impossible to deny the massive impact the adult website industry has had on pushing the web forward. From pushing the browser's video limits to [pushing ads through WebSocket](https://medium.com/thebugreport/pornhub-bypasses-ad-blockers-with-websockets-cedab35a8323) so ad blockers don't detect them, you have to be clever to innovate at the bleeding edge of the web.

无论你对色情内容采取何种立场，都无法否认成人网站行业对推动互联网发展具有巨大影响。从将浏览器的视频限制推送到[通过WebSocket推送广告](https://medium.com/thebugreport/pornhub-bypasses-ad-blockers-with-websockets-cedab35a8323)，以便广告拦截器无法检测到它们，你必须足够聪明才能在互联网的前沿进行创新。

I was recently lucky enough to interview a Web Developer at the web's largest adult website: Pornhub. I wanted to learn about the tech, how web APIs can improve, and what it's like working on adult websites. Enjoy!

最近，我很有幸采访互联网最大的成人网站 Pornhub 的一名 Web 开发者。我想了解技术，Web API 如何改进以及在成人网站上工作的感受。请享用！

*Note: The adult industry is very competitive so there were a few questions they could not answer.  I respect their need to keep their tricks close to the vest.*

*注意：成人产业竞争激烈，因此有一些他们无法回答的问题。我尊重他们保守商业机密的需要。*

**Adult sites obviously display lots of graphic content.  During the development process, are you using lots of placeholder images and videos?  How far is the development content and experience from the end product?**  

**成人网站显然会显示许多图形内容。在开发过程中，你是否使用了大量的占位符图像和视频？最终产品和开发时的内容和经验有什么区别？**

We actually don’t use placeholders when we are developing the sites! In the end, what matters is the code and functionality, the interface is something we are very used to at this point. There’s definitely a little bit of a learning curve at first, but we all got used to it pretty quickly. 

实际上，我们在开发网站时不使用占位符！最后，重要的是代码和功能，接口是我们现在非常习惯的东西。一开始肯定会有一些学习曲线，但是我们大家很快就习惯了。

**When it comes to cam streams and third party ad scripts, how do you mock such important, dynamic resources during site and feature development?**

**对于网络流和第三方广告脚本，你如何在网站和功能开发过程中模拟这些重要的动态资源？**

For development, the player is broken into two components.  The basic player implements the core functionality and fires events.  Development is done in a clean room. For integration on the sites, we want those third-party scripts and ads running so we can find problems as early in the process as possible.  For special circumstances we’ll work with advertisers to allow us to manually trigger events that might normally be random.

为了进行开发，播放器分为两个部分。基本播放器实现核心功能并触发事件。开发不会受其他因素干扰。为了在网站上进行集成，我们希望运行那些第三方脚本和广告，以便我们尽早发现问题。在特殊情况下，我们将与广告客户合作，允许我们手动触发通常可能是随机的事件。

**An average page probably has at least one video, GIF advertisements, a few cam performer previews, and thumbnails of other videos.  How do you measure page performance and how do you keep the page as performant as possible? Any tricks you can share?**

**平均每个页面可能至少包含一个视频，GIF 广告，一些 cam 表演者预览以及其他视频的缩略图。你如何测量页面性能以及如何使页面保持最佳性能？有什么你可已分享的技巧吗？**

We use a few measurement systems. 

* Our player reports metrics back to us about video playback performance and general usage
* A third-party RUM system for general site performance.
* WebpageTest private instances to script tests in the available AWS data centers.  We use this mostly for seeing what might have been going on at a given time. It also allows us to view the “waterfall” from different locations and providers.

我们使用一些测量系统。

* 我们的播放器会向我们报告有关视频播放性能和一般用法的指标
* 用于一般站点性能的第三方 RUM 系统。
* WebpageTest 私有实例，用于在可用的 AWS 数据中心中编写测试脚本。我们主要将其用于查看给定时间可能发生的情况。它还使我们能够查看来自不同位置和提供者的“瀑布”。

**I have to assume the most important and complex feature on the front-end is the video player.  From incorporating ads before videos, marking highlight moments of the video, changing video speed, and other features, how do you maintain the performance, functionality, and stability of this asset?**

**我必须假设前端最重要，最复杂的功能是视频播放器。从在视频之前加入广告，标记视频的精彩时刻，更改视频速度和其他功能，你如何维护该资产的性能，功能和稳定性？**

We have a dedicated team working strictly on the video player, their first priority is to constantly monitor for performance and efficiency. To do so we use pretty much everything that is available to us; browsers performance tools, web page tests, metrics  etc. The stability and quality is assured by a solid QA round for any updates we do. 

我们有一支专门致力于视频播放器的团队，他们的首要任务是持续监控性能和效率。我们为此几乎使用了所有可用的东西；浏览器性能工具，网页测试，指标等。我们进行的所有更新均通过可靠的质量检查来确保稳定性和质量。

**How many people are on the dedicated video team?  How many front-end developers are on the team?**

**专门的视频团队有多少人？团队中有多少前端开发人员？**

I’d say given the size of the product the team size is lean to average.

我要说的是，团队规模倾向于基于产品规模的平均水平。

**During your time working on adult websites, how have you seen the front-end landscape change?  What new Web APIs have made your life easier?**

**在成人网站上工作期间，你如何看待前端未来的变化？哪些新的 Web API 使你的生活更轻松？**

I’ve definitely seen a lot of improvements on every single aspect of the frontend world;

* From plain CSS to finally using LESS and Mixins, to a flexible Grid system with media queries and picture tags to accommodate different resolutions and screen sizes
* jQuery and jQueryUI are slowly moving away, so we are going back to more efficient object oriented programming in vanilla JS. The frameworks are also very interesting in some cases
* We love the new IntersectionObserver API, very useful for a more efficient way to load images
* We started playing with the Picture-in-Picture API  as well, to have that floating video on some of our pages, mainly to get user feedback about the idea.

我肯定在前端世界的每个方面都看到了很多改进；

* 从纯 CSS 到最终使用 LESS 和 Mixins，再到使用具有媒体查询和图片标签的灵活 Grid 系统，以适应不同的分辨率和屏幕尺寸
* jQuery 和 jQueryUI 慢慢地被淘汰，因此我们将回到 vanilla JS 中更高效的面向对象编程。在某些情况下，框架也非常有趣
* 我们喜欢新的 IntersectionObserver API，对于以更有效的方式加载图像非常有用
* 我们也开始使用画中画 API，以便在我们的某些页面上播放该浮动视频，主要是为了获得用户对该想法的反馈。

**Looking forward, are there any Web APIs that you’d love changed, improved, or even created?**

**展望未来，有没有你想要更改，改进甚至创建的 Web API？**

Some of them that we would like changed or improved; Beacon, WebRTC, Service Workers and Fetch:

* Beacon: some IOS issues where it doesn’t quite work with pageHide events
* Fetch:  No download progress and doesn’t provide a way to intercept requests
* WebRTC:  Simulcast layers are limited even for screenshare, if the resolution is not big enough
* Service Workers: Making calls to navigator.serviceWorker.register isn't intercepted by any service worker's Fetch event handlers

其中有一些是我们希望改变或改进的；Beacon，WebRTC, Service Workers 以及 Fetch：

* Beacon：在 IOS 上存在 pageHide 事件无正常工作的问题
* Fetch：没有下载进度，也没有提供拦截请求的方法
* WebRTC：如果分辨率不够大，则即使进行屏幕共享，Simulcast 层也会受到限制
* Service Workers：调用 navigator.serviceWorker.register 不会被任何 Service Workers 的 Fetch 事件处理程序拦截

**WebVR is has been improving in the past few years -- how useful is WebVR in its current state and how much of an effort are adult sites putting into support for VR content?  Do haptics have a role in WebVR on your sites?**

**WebVR 在过去几年中一直在进步--WebVR 在当前状态下作用有多大，成人网站为支持 VR 内容付出了多少努力？触觉在你们网站上的 WebVR 中有起到作用吗？**

We’re investigating webXR and how to best adapt to emerging spatial computing use cases, and as the largest distribution platform we need to support creators and users however they want to experience our content. But we’re still exploring what content and platforms should be like in these new mediums.

We were the first major platform to support VR, computer vision, and virtual performers, and will continue to push new technology and the open web. 

我们正在研究 webXR 以及如何最好地适应新兴的空间计算用例，作为最大的发布平台，我们需要支持创作者和用户，无论他们想体验我们的内容如何。但是，我们仍在探索这些新媒体应具有什么样的内容和平台。

我们是第一个支持 VR，计算机视觉和虚拟表演者的主要平台，并将继续推动新技术和开放互联网的发展。

**With so many different types of media and content on each page, what are the biggest considerations when it comes to desktop vs. mobile, if any?** 

**每个页面上的媒体和内容种类繁多，那么桌面设备与移动设备之间最大的考虑是什么？**

Functionality restricted by operating system and browsers type mainly. iOS vs Android is the perfect example when it comes to a completely different set of access and features. 

For example, some iOS Mobile devices don’t allow us to have a custom video player while in Fullscreen, they force the native QuickTime player. That has to be considered when we develop new ideas. Android on the other hand gives us complete control and we can push our features to the Fullscreen mode.

Adaptive streaming in HLS is also another example, IE and Edge are picky when it comes to HLS streaming quality, in that we need to prevent certain of the higher qualities, otherwise the video would constantly stutter and have artifacts.

功能主要受操作系统和浏览器类型的限制。当涉及一组套完全不同的访问和功能时，iOS 对比 Android 是一个完美的例子。

例如，某些 iOS 移动设备不允许我们在全屏模式下使用自定义视频播放器，而是强制使用本机 QuickTime 播放器。我们提出新想法时必须考虑这一点。另一方面，Android 为我们提供了完全的控制权，我们可以将功能在全屏模式实现。

HLS 中的自适应流式传输也是另一个示例，当涉及到 HLS 流式传输质量时，IE 和 Edge 需要有所顾虑，因为我们需要防止某些更高质量的内容，否则视频将不断卡顿并出现伪像。

**What is the current minimum browser support for the adult sites you work on?  Is Internet Explorer phased out?**

**当前针对你工作的成人网站的最低浏览器版本支持是什么？ IE 浏览器上是否已淘汰？**

We supported IE for a very long time but recently dropped support for anything older than IE11. With it we also stopped working with Flash for the video player. We are focusing on Chrome, Firefox and Safari mainly. 
 
我们对 IE 的支持时间很长，但是最近放弃了对 IE11 之前的任何版本的支持。有了它，我们也停止了为视频播放器使用 Flash。我们主要关注 Chrome，Firefox 和 Safari。

**More broadly, can you share a little about the typical adult site’s stack?  Server and/or front-end? Which libraries are you using?**

**更广泛地说，你能否分享一些有关典型成人网站技术栈的信息？服务器和/或前端？你正在使用哪些库？**

Most of our sites use the following as a base:

* Nginx
* PHP
* MySQL
Memcached and/or Redis

Other technologies like Varnish, ElasticSearch, NodeJS, Go, Vertica are used where appropriate.

For frontend, we run mostly vanilla Javascript, we’re slowly getting rid of jQuery and we are just beginning to play with frameworks, mostly Vue.js

我们的大多数网站都以以下内容为基础：

* Nginx
* PHP
* MySQL
* Memcached 和/或 Redis

在适当的地方使用其他技术，例如 Varnish，ElasticSearch，NodeJS，Go，Vertica。

对于前端，我们主要运行原生 Javascript，我们逐渐摆脱了 jQuery，我们才刚刚开始使用框架，主要是Vue.js

**From an outsider’s perspective, adult sites generally seem to be very much alike:  lots of video thumbnails, aggregated video content, cam performers, adverts. As someone who works on them, what are the differentiating features that make adult sites unique?**

**从局外人的角度来看，成人网站通常看起来很相似：很多视频缩略图，聚合的视频内容，摄像头表演，广告。作为从事这些工作的人，使成人网站与众不同的特征是什么？**

We work very hard to give each brand some uniqueness at different levels; content library, UX and features sets, and across a lot of different algorithms. 

**Before applying and interviewing for your current employer, what were your thoughts on potentially working on adult sites?  Did you have any hesitations? If so, how were your fears to put rest?**

It never really bothered me, in the end the challenge was so appealing. The idea of millions of people potentially interacting with features I worked on was really motivating. That proved to be true very quickly, the first time something I worked on went live, I was super proud, and I indeed told all my friends to go check it out! The fact that porn will never die is reassuring for job stability as well!

**In as far as end product, sharing that you work on adult sites may not be the same as working at a local web agency.  Is there a stigma attached to telling friends, family, and acquaintances you work on adult sites? Is there any hesitance in telling people you work on adult sites?**

I’m very proud to work on these products, those close to me are aware and fascinated by it. It’s always an amazing source of conversation, jokes and is genuinely interesting. 

**Having worked at agencies outside the adult industry, is there a difference in atmosphere when working on adult sites?**

The atmosphere here is very relaxed and friendly. I don’t notice any major differences with respect to work culture at other agencies, other than the fact that it’s much bigger here than anywhere I have worked previously. 

Being a front-end developer, which teams do you work most closely with?  What are the most common daily communication methods?

We work equally with backend developers, QA testers and product managers - most of the time we simply go up to each other’s desk and talk. If not, chat (MS Teams) is very common. Then come emails.

**Lastly, is there anything you’d like to share as a front-end developer working on adult sites?**

It’s really exciting being a part of creating how users experience such a widely used product. We are generally at the forefront of trends and big changes in tech as they roll out, which keeps it fun and challenging.

*Interview end*

I found our interview really enlightening. I was a bit surprised they didn't use images while developing features and designs. It's exciting to see that Pornhub continues to push the bleeding edge of the web with WebXR, WebRTC, and Intersection Observer. I was also happy to see that they consider the current set of web APIs sufficient to start dropping jQuery.

I really wish I'd have been able to get more specific tech tips out of them; performance and clever hacks especially. I'm sure there's a wealth of knowledge to be learned behind their source code! What questions would you have asked?

