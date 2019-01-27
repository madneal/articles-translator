## 采用Flow以及TypeScript 

 ### 两个过渡流程之间的比较

>原文：[Adopting Flow & TypeScript](http://thejameskyle.com/adopting-flow-and-typescript.html)
>
>译者：[neal1991](https://github.com/neal1991)
>
>welcome to star my [articles-translator](https://github.com/neal1991/articles-translator/), providing you advanced articles translation. Any suggestion, please issue or contact [me](mailto:bing@stu.ecnu.edu.cn)
>
>LICENSE: [MIT](https://opensource.org/licenses/MIT)

>  和这首歌[**“Cotton Heads” — Caravan Palace**](https://www.youtube.com/watch?v=QNkBLye7xfY)食用本文更佳。

让我们想象一个需要采用类型检查的场景。

之后我们注意到我们的app出现了很多NaN。我们寻找源头并且找到了下面的代码：


```javascript
// math.js
function square(n) {
  return n * n;
}
square("oops");
```
我们对自己叹了口气，并且决定或许是时候添加一个类型检查了。我们退后一步，看看我们的选项：[Flow](https://flow.org/)或者[TypeScript](http://www.typescriptlang.org/)。

这两个工具都有相当简单的文件逐个采用：

* **Flow:** 在你的文件顶部添加`// @flow` 注释
* **TypeScriptz:** 将.js后缀改成.ts后缀

但是让我们一起来比较。

 ### 采用TypeScript
为了采用TypeScript，我们将首先将math.js重命名成math.ts：

```typescript
// math.ts
function square(n) {
  return n * n;
}
square("oops");
```
现在我们运行typescript：

    (no errors)
没有错误，因为TypeScript需要添加注解到我们的函数在它如下检查类型之前：

```typescript
function square(n: number): number {
  return n * n;
}
square("oops");
```
但是没有这些类型，TypeScript将依赖你的配置来做以下两件事之一:

1. 隐含地将每个未知类型投射到任何一个。 这种任何类型将选择您退出所有类型检查。
2. 或者如果您使用--noImplicitAny选项，则会为任何未知类型引发错误，需要添加类型注解。

这意味着TypeScript覆盖的代码量与您所写的类型相关。 写入类型时，类型覆盖将线性上升。

 ### 类型覆盖
在我们进一步之前，我应该解释一下什么是类型覆盖。![Uncovered code shown in red](https://cdn-images-1.medium.com/max/2000/1*CgIv2yvDU_GTscCRLFp6DA.png) 

如果你观察代码中的值以及表达是并且询问类型检查“你知道这是什么类型吗？”。



如果类型检查知道这个类型，接着这个值或者表达式就被覆盖了。如果类型检查不知道这个类型，那么它就没有被覆盖。

类型检查器知道类型的代码的百分比是程序的“类型覆盖”。

你希望你的程序具有尽可能多的类型覆盖，因为那么他将能够告诉你在更多地方的错误。

如果没有类型覆盖，类型检查也就没什么意义了。

 ### 采用Flow
```javascript
// @flow
function square(n) {
  return n * n;
}
square("oops");
```
接着我们运行Flow会看到这个结果

```javascript
function square(n) {
  return n * n;
         ^   ^
         Error (x2)
}
square("oops");
Error (x2)
string. The operand of an arithmetic operation must be a number.
```
我们立刻就会得到类型错误，告诉我们某些地方有问题。

Flow只需要我们输入文件以及外部模块的导出。其他的一切都可以被*推导出*。这使得代码覆盖快得多。只需要几种类型，你就可以获取具有非常高代码覆盖的文件。

以我自己的经历，我能够在几分钟内令文件的代码覆盖达到大约70-90%。

下面是一个超级科学的差异图：

![](https://cdn-images-1.medium.com/max/3728/1*mhy7hyBK_lQaHTsu6YB3uA.png) 

这并不是我的个人观点，你可以自己去尝试一下，然后看下就利用几种类型会有什么不同。

为了在Flow中得到文件的代码覆盖，你可以运行：

    flow coverage path/to/file.js --color
你也可以利用[flow-coverage-report](https://github.com/rpl/flow-coverage-report) 来帮助你。

> **注意：** 对于TypeScript我不知道任何代码覆盖报告工具（如果你知道的话，请给我发个链接！）但是你可已测试一下报告中错误的地方是否被代码覆盖检查所覆盖到。

## 它是如何工作的？ 

两种工具采用的不同行为的原因归根到底是因为其架构的区别

 ### TypeScript架构: AST-directed
TypeScript将会遍历你的程序，并且构建已知类型的表。因为当它发现值以及表达式的化，会立刻给它分配一种类型。当TypeScript发现一个未知类型的时候，它必须立刻做出决定，这意味着要么将它分配到任何一种或者抛出一个错误。

 ### Flow架构: Graph-directed
Flow会基于你的值和表达式以及它们之间的关系来构建一个图。它接着就会开始给每个值以及表达式分配类型。如果它发现一个未知类型，它会将它标志位一个“开放的”类型并且会在后面再处理。

一旦Flow获得你程序的完整图，它将会开始连接所有的点，*流动*类型将会从一个值传递到其它的。开放的类型接受其它值流动到它们的类型--生成的类型被成为推断类型。

你能够看到发生了什么.让我们看一下在世Flow之前的类型错误：

```javascript
function square(n) {
  return n * n;
         ^   ^
         Error (x2)
}
square("oops");
Error (x2)
string. The operand of an arithmetic operation must be a number.
```
注意到错误是指向n * n而不是square("oops")。因为我们没有为n指定一个类型，接着"opps"字符串就会流入到其中那么Flow就开始检查n是不是一个字符串。

```javascript
function square(n: number) {
  return n * n;
}
square("oops");
       ^ Error
Error: string.
This type is incompatible with the expected param type of number.
```
这提出了重要的一点：因为Flow可以推导所有地方的类型，这并不意味着你不需要在你的代码中添加类型注解。

 ### 结论
TypeScript和Flow都具有很好的过渡过程。逐个文件是一个很好的经历。然而，如果你使用Flow，你将会有更高的代码覆盖并且更快。这样你也能够睡得更安心。

使用Flow，您将添加类型以使错误更好展示，而不是发现它们。
