# Vue in 2016

![vue trend](https://cdn-images-1.medium.com/max/800/1*ZJD3llCWveVH9-uUcCjMCw.png)

现在已经是2016的尾声了！在这过去的12个月里，Vue的持续增长速度已经超过了我的预期--这个项目已经从一个相对较小的小框架成长起来，现在已经被用来和这个领域最出名的框架相比较。让我们看看都发生了什么吧！

## 2016 统计总览

* NPM 下载量: 1,943,567 total, 1,531,217 YTD (up from 382,184 in 2015, +300%)
* 核心GitHub仓库star数: 37,664 total, ~26,000 YTD (up from ~7,600 in 2015, +240%)
* 核心仓库pull request合并数: 350 total, [258 YTD](https://github.com/vuejs/vue/pulls?utf8=%E2%9C%93&q=is%3Apr%20is%3Amerged%20merged%3A2016-01-01..2016-12-31%20) (up from [70](https://github.com/vuejs/vue/pulls?utf8=%E2%9C%93&q=is%3Apr%20is%3Amerged%20merged%3A2015-01-01..2015-12-31%20) in 2015, +268%)
* vuejs.org网页浏览数: 21,424,759 YTD (up from 3,761,728 in 2015, +470%)
* Chrome开发者工具周活跃用户: 84,098 (no prior year stats)

[Trivia: Vue是Github上面Javascript项目star数目前十，并且是2016年所有项目中的第三名](https://docs.google.com/spreadsheets/d/11bGpZq6ixlhrmQnzEUqbgbwTQwQVdtvILjp32vaOKBc/edit#gid=1735042899)* (是第一名在源代码项目中).*

## 这会仅仅就是个炒作吗？

我们当然希望不是！Vue现在已经开始差不多3年的时间了，并且在这过去的两年中增长曲线已经保持稳定。我们在我们将Vue用于产品的用户中收到了积极的反馈，今年在HN-front-page上已经有两名Vue用户在文章上表达了积极的意见。上面说，Vue不是也不打算成为下一个闪亮的东西你应该尽快向其发展--它的设计缪奥不是解决UI的开发范例；而是它是能够使更多的人可以来建设网站。Vue受到很多其他的完美的框架所激励，但是它的目标是结合并且以暴露这些想法，并且这个过程是循序渐进的，因此更多的开发者也能从中受益。因此不要因为它很流行就向其转换--但是你应该尝试一下来看一下它能否让你成为一个更加快乐的开发者！

### 变得独立

在三月份初我决定为Vue独立全职工作，并且启动了 [Patreon campaign](https://www.patreon.com/evanyou)，这个活动到目前为止都是很成功的，这一点其实我也很意外。这是我第一次不需要向经理报告而独自工作，这是一次自由自在的经历。好的方面是工作再也感觉不是工作了：我不需要强迫自己去工作，因为我需要做的正是我想做的，并且我能够完全控制如何并且何时开始工作。另外一个方面，它也是坏的方面：在生活和工作中平衡变得很困难，特别是作为一个新爸爸（今年我也有了一个新宝宝！）这也正是我学着处理的事情并且希望明年可以变得更好。但是总而言之，我觉得我现在更有动力和成就感，我想向所有的Patreon backers和赞助者表示感谢。

### 发布 2.0

The work on 2.0 was a big undertaking — I had been thinking about it for a long time, but only started the effort thanks to the full-time switch. The entire project was rewritten from the ground up to take advantage of completely different compilation/rendering strategies — but at the same time it had to preserve the same development experience and a largely similar API with the previous version. This also meant that we had to upgrade the surrounding ecosystem of vue-router, vuex and the build toolchain to work with the new core, plus updating the documentation for all of them. It turned out to be much more work than I anticipated: I started the first prototype of Vue 2.0 in early April, and the [official launch](https://medium.com/the-vue-point/vue-2-0-is-here-ef1f26acf4b8#.2ptwit6jz) was on Sep. 30th, after almost 6 months! The new release made Vue leaner, faster and more powerful with the new features and cross-environment capabilities. Looking back, I’m really proud of the work we’ve done.

2.0的工作是一个大的工程--为此我已经考虑了很长的时候，但是得益于全职工作的转换我才有时间开展这项工作。得益于编译/渲染的策略，这整个项目被从头到尾重写了--但是同时也要保持同样的开发经理并且API的调用也应该和之前版本差不多。这意味着我们必须对vue-router,vuex以及build工具链的相关的生态系统来对新的核心工作，并且更新了相应的所有文档。这其实涉及到更多我参与的工作：我在四月初开始了Vue 2.0的原型，[官方发布](https://medium.com/the-vue-point/vue-2-0-is-here-ef1f26acf4b8#.2ptwit6jz)实在九月三十号，差不多有六个月的时间！新的发布让Vue具有新特性和跨环境能力的同时，也使它的体积更小，更快都并且更强大，回顾过去，我对我们完成的工作感到很骄傲。

### 不再是一个个人项目

As the scope of Vue-related projects expanded, I was fortunate enough to be joined by many awesome contributors from the community. Today, many [core team](https://github.com/orgs/vuejs/people) members are actively contributing to all aspects of the project, from core features like server-side rendering to sub-projects like the documentation (vuejs.org), vue-router, vuex and official TypeScript integrations. I have been constantly humbled by their enthusiasm and dedication to open source work.

在Vue相关拓展的项目中，我有幸和社区中的很多杰出的贡献者一起合作。今天，很多[核心团队成员](https://github.com/orgs/vuejs/people)都对这个项目的哥哥方面做出了贡献，从核心功能比如服务端的渲染到子项目比如文档工作(vuejs.org)，vue-router,vuex以及官方Typescript的集成。他们对开源工作的激情和奉献让我自惭形秽。

Of course, the same appreciation goes to all other community contributors for participating in design discussions, proposing new features, triaging issues, and submitting bug fixes. We are getting more and more PRs (258 this year in the core repo alone!), with quite a few of them bringing substantial improvements to the codebase. It is the combination of everyone involved that is making the Vue ecosystem better day by day. This project wouldn’t be where it’s at now without your contributions!

当然，同样感谢所有其他参与讨论设计，提出新功能，issue分类并且提交BUG修复的社区贡献者。我们收到越来越多的PR（今年核心项目就有258个！）Vue生态系统通过综合大家共同的努力从而使这个生态系统变得越来越好。这个项目如果没有大家的努力也没有办法发展至今！

------

### 未来展望

There are still many aspects that can be improved about Vue. Here’s what’s on our list for 2017:

对于Vue还有很多方面可以进一步提升。现在让我们看看2017的清单有哪些：

#### 提高测试需求

Based on the feedback on Twitter, many users feel that there isn’t enough information on testing Vue components and applications. This is an area we will definitely focus on improving — both by providing more guidance in the docs, and also by providing official testing utilities that makes it easier to test Vue components.

根据推特上的反馈，很多用户反映Vue的组件和应用的测试信息还不够充分。这一领域毫无疑问我们会专注提升--不仅提供更多的文档知道，并且提供更多的官方测试单元从而使Vue组件测试变得更加容易。

#### 改善开发体验

We want to further polish the dev experience across the stack: better in-development tips and warnings, better server-side rendering (SSR) stack traces, performance profiling in DevTools, and better CLI templates that make it easier to use SSR or build PWAs.

我们想进一步提升开发体验：更好的开发建议和警告，更好的服务端渲染堆栈，开发者工具中更好的性能日志，以及更好的CLI模板从而更容易使用服务端渲染或者PWA。

#### 对于生态系统更好的可发现性

The Vue ecosystem has been growing rapidly, but as a result [awesome-vue](https://github.com/vuejs/awesome-vue) has become bloated and very difficult to navigate and evaluate. The team has been thinking hard about this and we are working on a project to help users identify high-quality community projects: it will be curated with a higher standard and will provide much more detailed information for each project included.

Vue的生态系统成长迅速，从而[awesome-vue](https://github.com/vuejs/awesome-vue)变得很难浏览和评价。团队已经在努力考虑这个问题，并且我们正在通过一个项目来帮助用户识别高质量的社区项目；对于每一个钣焊的项目都会提供更高标准和更多的信息。

#### 原生渲染

The collaboration with Alibaba’s [Weex project](http://weex-project.io/) has made some substantial progress. Our first major goal — making Vue 2 the actual JavaScript runtime framework on top of Weex’s native rendering engine — has been completed in the latest 0.9.4 release. In 2017 there will be revamped documentations, better onboarding experience and API improvements based on feedback from the community and large-scale production use at Alibaba.

和阿里巴巴合作的[Weex project](http://weex-project.io/)已经取得实质性的进展。我们的第一个主要目标--在最近的0.9.4发布的版本中通过Weex的原生渲染引擎使得Vue 2成为Javascript的实时框架。在2017年会对文档进行修补，通过社区的反馈和阿里巴巴的大规模使用来改进使用体验和API。

#### 拥抱Web平台特色

As new standards emerge and get implemented, we are keeping a close eye on ones that can potentially bring significant improvements to Vue. For example, it’s possible to leverage [ES2015 Proxy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy) to eliminate some of the current limitations of Vue’s reactivity system. We are also exploring the possibility of compiling and distributing Vue components as native Custom Elements. Right now, the biggest blocker is browser compatibility — to ensure consistent behavior in all supported browsers, it’s unlikely for us to prioritize these features until the support ratio reaches a significant majority. But rest assured that we are aware of these emerging standards and already experimenting. When the time is ready, Vue will swiftly evolve to leverage these new platform capabilities.

新的标准出现并且得以实现，我们密切关注能够潜在为Vue带来有意义进步的新的标准。比如，可能通过 [ES2015 Proxy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy)来消除Vue响应系统当前的限制。我们也在探索将Vue组件编译并且发布成原生的自定义元素。马上，最大的苦难是浏览器的兼容性--为了在所有支持的浏览器中保持一致的表现，我们只有在支持比例达到一定的程度才会发布这些新特色。但是其他出现的标准我们已经注意到并且在试验环节。如果实际成熟，Vue会快速地兼容这些新的标准。

#### 还有一点：我们在2017年有一个会议啦！

It’s still in early planning stages, but [visit the site](http://conf.vuejs.org/) and [take the survey](https://docs.google.com/forms/d/e/1FAIpQLSfiRF9JIpvAcWL7EsnpODIhf_JiNX3PETA_S3XnqmtuG2foQA/viewform) to help us make it a great event. Better yet, [submit a talk](https://docs.google.com/forms/d/e/1FAIpQLSdtbxBpV0j_zCnELXQuIkeGH8x6gaOWE0J8tTsAdpa0O5MYOw/viewform) to share your knowledge and experience with fellow Vue users!

当然实际尚早，但是浏览这个[网站](http://conf.vuejs.org/)并且参与[调研](https://docs.google.com/forms/d/e/1FAIpQLSfiRF9JIpvAcWL7EsnpODIhf_JiNX3PETA_S3XnqmtuG2foQA/viewform))将会使得这件事变得更加有意义。更好的是，如果你能提交一个[演讲](https://docs.google.com/forms/d/e/1FAIpQLSdtbxBpV0j_zCnELXQuIkeGH8x6gaOWE0J8tTsAdpa0O5MYOw/viewform)来分享你的知识和经历。

------

*If you are interested in Vue, checkout the *[*official site*](https://vuejs.org/)* to get started, and *[*follow us on Twitter*](https://twitter.com/vuejs)*!*

如果你对Vue感兴趣的话，那么你可以从[官方网址](https://vuejs.org/)开始，你也可以在[推特](https://twitter.com/vuejs)上关注我们！（然而我们有墙！！！）

