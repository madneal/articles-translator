# 通过利用immutability的能力编写更安全和更整洁的代码

> 原文：[Write safer and cleaner code by leveraging the power of “Immutability”](https://medium.freecodecamp.com/write-safer-and-cleaner-code-by-leveraging-the-power-of-immutability-7862df04b7b6)
>
> 译者：[neal1991](https://github.com/neal1991)
>
> welcome to star my [articles-translator](https://github.com/neal1991/articles-translator/), providing you advanced articles translation. Any suggestion, please issue or contact [me](mailto:bing@stu.ecnu.edu.cn)
>
> LICENSE: [MIT](https://opensource.org/licenses/MIT)

![](https://cloud.githubusercontent.com/assets/12164075/26274743/77a794ca-3d83-11e7-861f-6504b9c0e8c6.png)

Immutability是函数式编程的重要基础之一。它允许你能编写更安全以及更整洁的代码。我将会通过一些JavaScript例子来向你展示如何来达到immutability。

**根据维基百科：**

>不可变对象是一个在创建之后不能修改其状态的对象。这正与可变对象相反，它能够在创建之后被修改。在某些情况下，对象被认为是不可变的，即使其内部的某些属性发生改变，但是从外部的角度来看这个对象的状态看起来还是没有发生变化的。

## Immutable数组

数组是理解immutability如何工作的很好的起点。让我们一起来看一看。

```javascript
const arrayA = [1, 2, 3];
arrayA.push(4);
 
const arrayB = arrayA;
arrayB.push(5);
 
console.log(arrayA); // [1, 2, 3, 4, 5]
console.log(arrayB); // [1, 2, 3, 4, 5]
```

这个例子将**arrayA**的引用分配给**arrayB**，因此这个push方法在这两个变量中都会添加5这个值。我们的代码间接地修改其它的值，这并不是我们想要的。这也违反了immutability的原则。

我们可以通过使用 [slice](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/slice)函数将我们的例子提升为immutable，并且这个代码的行为也产生了变化。

```javascript
const arrayA = [1, 2, 3];
arrayA.push(4);
 
const arrayB = arrayA.slice(0);
arrayB.push(5);
 
console.log(arrayA); // [1, 2, 3, 4]
console.log(arrayB); // [1, 2, 3, 4, 5]
```

这正是我们想要的。代码没有改变其它值。

提醒：当你使用 [push](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/push) 向数组添加一个值的时候，你是在**改变**这个数组。你想要避免修改变量，因为这可能会给你的代码带来负面影响。 [slice](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/slice)函数能够返回数组的拷贝。

## 函数

现在你知道如何避免修改其它值。那么你知道如何写“纯”函数了嘛？纯函数是对于一个不会又任何副作用以及不会改变状态的函数另一个称呼。

我们来看一个利用数组示例中相同原理的函数。 首先，我们创建一个改变其它值的函数，然后我们将该函数改进为“纯”。

```javascript
const add = (arrayInput, value) => {
  arrayInput.push(value);
 
  return arrayInput;
};
const array = [1, 2, 3];
 
console.log(add(array, 4)); // [1, 2, 3, 4]
console.log(add(array, 5)); // [1, 2, 3, 4, 5]
```

因此再一次，我们**修改**我们的输入，这会产生一个无法预测的函数。在函数式编程的世界中，对于函数有一个黄金法则：**使用相同输入的函数应该返回相同的结果。**

上述的函数违反了这一黄金法则。每一次我们的**add**函数被调用的时候，它就会修改这个**array**变量，结果也就不同了。

让我们一起看看看如何改变我们**add**函数的事先，因此让其成为immutable。

```javascript
const add = (arrayInput, value) => {
  const copiedArray = arrayInput.slice(0);
  copiedArray.push(value);
 
  return copiedArray;
};
 
const array = [1, 2, 3];
const resultA = add(array, 4);
console.log(resultA); // [1, 2, 3, 4]
const resultB = add(array, 5);
console.log(resultB); // [1, 2, 3, 5]
```

现在我们可以调用我们的函数多次，并且可以预期在相同输入的时候，输出都是相同的。这是因为我们不再会修改这个**array**变量。我们能够将这个函数成为“纯函数”。

> **注意**：你也能使用**concat**，而不是**slice**以及**push**。
>
> 因此：arrayInput.concat(value)

我们能够利用ES6中的[扩展语法]((https://developer.mozilla.org/nl/docs/Web/JavaScript/Reference/Operators/Spread_operator)来缩短函数。

```
const add = (arrayInput, value) => […arrayInput, value];
```

### 并发

NodeJS应用使用了一个叫做并发的概念。一个并发操作意味着两个计算能够能够同时进行并且不用考虑另外一个。如果这里有两个线程的话，第二个计算不需要等到第一个计算完成才能只需执行。

![](https://cloud.githubusercontent.com/assets/12164075/26275010/948ad254-3d89-11e7-8a1a-300144626274.png)

NodeJS通过event-loop让并发变得可能。event-loop会重复获取一个事件，并且每次会激活任一一个事件处理器来监听事件。这个模型允许NodeJS应用处理大量的请求。如果你想了解更多，阅读[这篇关于event-loop的文章](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick)。

Immutability和并发之间有什么关系呢？因为做个操作能够在函数的作用域外以并发的方式改变值，这会产生一个不可靠的输出以及无法预期的结果。意识到函数可能会在作用域外修改变量，因此这可能会非常危险的。

### 下一步

Immutability对于你理解和学习函数式编程是非常重要的一个概念。你可能希望了解mmutableJS](https://facebook.github.io/immutable-js)，其由Facebook开发者编写。这个library提供了某些不可变的数据机构，比如 **Map**, **Set**以及**List**.

[Immutable.js, persistent data structures and structural sharing](https://medium.com/@dtinth/immutable-js-persistent-data-structures-and-structural-sharing-6d163fbd73d2)（译者注：墙外地址）