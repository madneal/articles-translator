## Bootstrap 被认为是有害的

> 原文：[Bootstrap considered harmful](https://blog.sicara.com/a-progressive-web-application-with-vue-js-webpack-material-design-part-1-c243e2e6e402)
>
> 译者：[neal1991](https://github.com/neal1991)
>
> welcome to star my [articles-translator](https://github.com/neal1991/articles-translator/), providing you advanced articles translation. Any suggestion, please issue or contact [me](mailto:bing@stu.ecnu.edu.cn)
>
> LICENSE: [MIT](https://opensource.org/licenses/MIT)

这些年Bootstrap已经在前端项目中流行起来，它能够带来很多好处。然而，但是如果以你们的团队已经有了在职的前端开发人员，我觉得最好还是不要用Bootstrap，在某些地方，弊大于利。

### Bootstrap的好处是什么 
Bootstrap主要是栅格系统，但同时也带来了很多组件的样式表和脚本，包括表格，导航栏，进度条，页码，表单样式，模式和提示文本。在这篇文章，我所说的Bootstrap是包含它的所有功能的。
Bootstrap是一个很好的工具对于一个纸箱装饰他们的程序但是无须担心结果的样式问题的后端开发人员。如果因为某些原因，预算或者什么的，你的团队没有前端开发人员或者设计人员，Bootstrap是一个绝佳的弥补方法。
对于设计人员来说，Bootstrap也是有用处的：它可以快速地从设计软件切换到浏览器中，不需要过多担心前端的代码设计。
即使是对于那些基本只专注于数据但是很少关注UI和布局的前端开发人员来说，Bootstrap也是一个绝佳的工具。

### 什么时候你最好别用它

然而，如果你的团队已经拥有了前端开发人员，使用Bootstrap可能会潜在的浪费他们宝贵的时间，并让他们可能从解决实际问题上转移注意力。Bootstrap做的正是前端开发人员所擅长的事情，但是用的是一种很通用的方式。你的网站或者网络app是非常特别的，因此如果你使用一个通用的系统可能会不太合适。这意味着为了实现这种特殊性将会包含很多的异常发生。

###  当需要很多异常来复位Bootstrap

Bootstrap曾经是被Twitter 的开发人员用于系统化他们网络app的样式。如果你的网站app和他们的样式不一样，这意味着你需要解除他们中的某些样式。

很多网站和Twitter的样式并不相同。因此，如果他们装载了Bootstrap的时候，他们可能需要卸载很多地方。

在某些网站上，我看到有9/10的Bootstrap样式已经被网站自己的样式所替代。坦白说，这很荒谬。



### 当它让简单的事情变得复杂

CSS是给网站添加一套简单的样式规则，这有时候可能会被重写。当你在你的网站使用Bootstrap的样式的时候，几乎所有的元素都是用一个复杂的样式规则。任何异常都会在它之上表现。问题是大多数网站他们的样式异常都被表现在Bootstrap之上。

Bootstrap的样式是非常复杂的：你可以利用12列的栅格系统和任何元素相结合起来，对于需要特别处理的列则要区别对待。很多网站十分简单：它们在小屏幕设备上没有列或者只有一到两列在大一点的屏幕上。

### 当它产生技术债务的时候

前端依赖Bootstrap的时间越长，就会牵扯到更多的东西，更多的规则需要设置来覆盖Bootstrap的某些规则。这或多或少地让技术代码背负技术债务，尤其前端代码的部署需要手动的更新。随着依赖的增多，Bootstrap将变得更加难以移除。

### 当它命名一些不是你app的规定

命名是一件很困难的事情，为团队的应用中的规定命名需要花费相当多的时间。使用'btn'之类的缩写并不能很好的给组件命名。

### 结论

Bootstrap可能对于产生网站的多个流程都起到了很大的帮助。但是它并不能让所有的事情都变得简单：相反，很多问题可以由前端开发人员专注于UI就能够更好地解决。
