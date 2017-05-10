# Github Pages以及单页面应用

> 原文：[GitHub Pages and Single-Page Apps](https://dev.to/_evansalter/github-pages-and-single-page-apps)
>
> 译者：[neal1991](https://github.com/neal1991)
>
> welcome to star my [articles-translator ](https://github.com/neal1991), providing you advanced articles translation. Any suggestion, please issue or contact [me](mailto:bing@stu.ecnu.edu.cn)
>
> LICENSE: [MIT](https://opensource.org/licenses/MIT)

单页面应用（SPAs）现在特别火。它们使构建功能丰富，高性能的web应用变得特别容易。因此你可以自己去构建一个SPA，但是依然存在一个问题。你应该在什么地方托管呢？

这篇文章分为两个部分。第一部分，是列出了我用过的不同的SPA托管方法以及它们的优缺点。第二部分，是我使用Github Pages托管SPA的经验，包括我遇到的问题。我希望能够帮助读者在如何托管他们的app的时候做出一个知情决定，并且如果它们使用任何我谈论的方法，它们可以从我的错误中有所学习。

如果你希望专注于Github Pages并且跳过其它的方法，你可以直接跳到第四节：Github Pages。

## 1. Google App Engine

我没有深入web开发直到我开始我现在的工作。是的，我了解一些HTML以及CSS，并且我曾经上过大学的web开发课。然而，我对这些并不是很感兴趣知道我在工作领域使用这些技能。

在我的工作中，我们所有的应用都是使用App Engine来构建的。我们使用Python版本，Jinja2是我们的模板语言，Knockout.js是我们的前端代码。之后对于在App Engine中开发我觉得很舒适。我熟悉它，我已经到了可以相对快速，轻松地将新应用程序整合在一起的地步。