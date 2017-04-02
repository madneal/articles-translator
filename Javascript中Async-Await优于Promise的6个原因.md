# JavaScript中Async/Await优于Promise的6个原因 (教学)

> 原文：[6 Reasons Why JavaScript’s Async/Await Blows Promises Away (Tutorial)](https://hackernoon.com/6-reasons-why-javascripts-async-await-blows-promises-away-tutorial-c7ec10518dd9)
>
> 译者：[neal1991](https://github.com/neal1991/)

你可能期待已久，Node从7.6版本就是开始支持async/await了。如果你还到现在还没有尝试，下面是一些你应该立刻采用它的带有例子的理由，而且你永远都不想再回来了。

### Async/await 101

对于从来没有听过这个话题的人，下面将会是一个简短的介绍：

* async/await是一种写异步代码的新方式。之前对于异步代码的选择包括回调和promise。
* async/await事实上是建立在promise之上的。他不能仅仅通过回调或者节点回调来使用。
* async/wait和promise类似，是非阻塞的。
* async/await让异步代码看起来非常像同步代码。这就是它的能力所在。

### 语法

假设函数`getJSON`返回一个promise，并且这个promise解析了一些JSON对象。我们只想调用它并且显示出JSON，接着返回`"done"`。

下面是使用promise来实现

```javascript
const makeRequest = () => 
	getJSON()
		.then(data => {
          console.log(data);
          return "done"
		})
makeRequest()
```

接着看看使用async/awiat应该是什么样子的

```javascript
const makeRequest = async() => {
  console.log(await getJSON())
  return "done"
}
makeRequest()
```

在这主要有几点不同

1. 我们的函数名之前有一个关键字`async`。`await`这个关键字只能在使用`async`定义的函数里面使用。任何`async`函数都会默认返回promise，并且这个promise解析的值都将会是这个函数的返回值（在我们这个例子里面是字符串`"done"`）
2. 上面也指出了我们不能在我们代码的最高层级使用await，因为它不是在`async`函数之内的。

```javascript
// 浙江不会再最高层次工作
// await makeRequest()

// 这将会工作
makeRequest().then((result) => {
  // 做什么
})
```

3. `await getJSON()`意味着`console.log`只有等到`getJSON`promise解析完毕之后才会被调用然后打印值。

### 为什么它更好？

####  简洁干净

看看我们要少写多少代码！



