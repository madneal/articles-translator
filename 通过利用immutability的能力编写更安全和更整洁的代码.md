# 通过利用immutability的能力编写更安全和更整洁的代码

> 原文：[Write safer and cleaner code by leveraging the power of “Immutability”](https://medium.freecodecamp.com/write-safer-and-cleaner-code-by-leveraging-the-power-of-immutability-7862df04b7b6)
>
> 译者：[neal1991](https://github.com/neal1991)
>
> welcome to star my [articles-translator ](https://github.com/neal1991), providing you advanced articles translation. Any suggestion, please issue or contact [me](mailto:bing@stu.ecnu.edu.cn)
>
> LICENSE: [MIT](https://opensource.org/licenses/MIT)

![](https://cloud.githubusercontent.com/assets/12164075/26274743/77a794ca-3d83-11e7-861f-6504b9c0e8c6.png)

Immutability is one of the building blocks of functional programming. It allows you to write safer and cleaner code. I’ll show you how you can achieve immutability through some JavaScript examples.

Immutability是函数式编程的重要基础之一。它允许你能编写更安全以及更整洁的代码。我将会通过一些JavaScript例子来向你展示如何来达到immutability。

**根据维基百科：**

>An immutable object (unchangeable object) is an object whose state cannot be modified after it is created. This is in contrast to a mutable object (changeable object), which can be modified after it is created. In some cases, an object is considered immutable even if some internally used attributes change but the object’s state appears to be unchanging from an external point of view.
>
>不可变对象是一个在创建之后不能修改其状态的对象。这正与可变对象相反，它能够在创建之后被修改。在某些情况下，对象被认为是不可变的，即使其内部的某些属性发生改变，但是从外部的角度来看这个对象的状态看起来还是没有发生变化的。

