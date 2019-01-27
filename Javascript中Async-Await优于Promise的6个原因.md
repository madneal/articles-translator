# JavaScript中Async/Await优于Promise的6个原因 (教学)

> 原文：[6 Reasons Why JavaScript’s Async/Await Blows Promises Away (Tutorial)](https://hackernoon.com/6-reasons-why-javascripts-async-await-blows-promises-away-tutorial-c7ec10518dd9)
>
> 译者：[neal1991](https://github.com/neal1991)
>
> welcome to star my [articles-translator](https://github.com/neal1991/articles-translator/), providing you advanced articles translation. Any suggestion, please issue or contact [me](mailto:bing@stu.ecnu.edu.cn)
>
> LICENSE: [MIT](https://opensource.org/licenses/MIT)

你可能期待已久，Node 从7.6版本就是开始支持 async/await 了。如果你还到现在还没有尝试，下面是一些你应该立刻采用它的带有例子的理由，而且你永远都不想再回来了。

### Async/await 101

对于从来没有听过这个话题的人，下面将会是一个简短的介绍：

* async/await 是一种写异步代码的新方式。之前对于异步代码的选择包括回调和 promise。
* async/await 事实上是建立在promise之上的。它不能仅仅通过回调或者节点回调来使用。
* async/wait 和 promise类似，是非阻塞的。
* async/await 让异步代码看起来非常像同步代码。这就是它的能力所在。

### 语法

假设函数`getJSON`返回一个 promise，并且这个 promise 解析了一些 JSON 对象。我们只想调用它并且显示出 JSON，接着返回`"done"`。

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
// 这将不会在最高层次工作
// await makeRequest()

// 这将会工作
makeRequest().then((result) => {
  // 做一些事
})
```

3. `await getJSON()`意味着`console.log`只有等到`getJSON`promise解析完毕之后才会被调用然后打印值。

### 为什么它更好？

####  1.简洁干净

看看我们要少写多少代码！即使在上述设计的例子中，很明显我们节省了一定数量的代码。我们不需要再写`.then`，创建匿名函数来处理响应，或者将变量命名为`data`，我们并不需要使用它。我们也可以避免嵌套我们的代码。这些小优点很快就累加在一起，在下面的代码例子中将会更加明显。

#### 2.错误处理

Async/await能够在同样的构造函数下处理同步和异步错误，在`try.catch`也可以正常运行。在下面使用promise的例子中，`try/catch`将不会处理如果`JSON.parse`失败的话，因为这是在promise内部发生的。我们需要调用在promise上调用`.catch`并且再一次写一个错误处理代码，这将比你希望中在生产环境中使用`console.log`更复杂。

```javascript
const makeRequest = () => {
  try {
    getJSON()
    	.then(result => {
          // 这个解析可能会失败
      		const data = JSON.parse(result)
            console.log(data)
    	})
    	// 取消下面的注释来处理异步错误
    	// .catch((err) => {
        // 	console.log(err) 
       //})
  } catch (err) {
    console.log(err)
  }
}
```

现在我们在async/await中看一下相同的代码。`catch`作用域现在可以处理错误

```javascript
const makeRequest = async() => {
  try {
    // 这个解析可能会失败
    const data = JSON.parse(await getJSON())
    console.log(data)
  } catch (err) {
    console.log(err)
  }
}
```

#### 3.条件

考虑以下的代码，假如我们需要获取一些数据并且是否返回它或者基于这个数据获取更多的值。

```javascript
const makeRequest = () => {
  return getJSON()
  	.then(data => {
      if (data.needAnotherRequest) {
        return makeAnotherRequest(data)
        	.then(moreData => {
              console.log(moreData)
              return moreData
        	})
      } else {
        console.log(data)
        return data
      }
  	})
}
```

上述的代码肯定让你很头痛。很容易在层层包裹的代码中迷失自我（6层），大括号，返回语句只有在将最终结果传播到promise中才是需要的。

下面的例子通过使用async/await来进行重写，从而让代码更容易阅读。

```javascript
const makeRequest = async() => {
  const data = await getJSON()
  if (data.needAnotherRequest) {
    const moreData = await makeAnotherRequest(data)
    console.log(moreData)
    return moreData
  } else {
    console.log(data)
    return data
  }
}
```

#### 4.中间值

你可能会发现这样的情形，当你调用`promise1`之后，使用它的返回来继续调用`promise2`，接着使用这两个promise的返回结果来调用`promise3`。你的代码很有可能是这个样子的

```javascript
const makeRequest = () => {
  return promise1()
  	.then(value1 => {
      	// 做一些事
    	return promise2(value1)
        	.then(value2 => {
              	// 做一些事
          		return promise3(value1, value2)
        	})
  	})
}
```

如果`promise3`不需要`value1`的话，那么可以将promise的嵌套展平一点。如果你是那种没有办法忍受这种的人，你可以将value1以及value2包裹在`Promise.all`里面来避免更深层次的嵌套，如下

```javascript
const makeRequest = () => {
  return promise1()
  	.then(value1 => {
      // 做一些事
    	return Promise.all([value1, promise2(value1)])
  	})
  	.then([value1, value2]) => {
      // 做一些事
    	return promise3(value1, value2)
  	}
}
```

这段代码为了可读性牺牲了语义性。没有理由将value1和value2放在一个数组里面，除非是为了避免嵌套promise。

同样的逻辑在async/await中将会十分的简单直白。它可以在promise变得可怕之前让你知道你可能完成所有的事情。

```javascript
const makeRequest = async() => {
  const value1 = await promise1()
  const value2 = await promise2()
  return promise3(value1, value2)
}
```

#### 5.错误堆栈

想象以下一段代码链式调用多个promise，并且在链的的某个环节错误被抛出。

```javascript
const makeRequest = () => {
  return callAPromise()
  	.then(() => callApromise())
  	.then(() => callApromise())
  	.then(() => callApromise())
  	.then(() => callApromise())
  	.then(() => {
      throw new Error("oops")
  	})
}

makeRequest()
	.catch(err => {
      console.log(err)
      // 输出
      // Error: oops at callAPromise.then.then.then.then.then (index.js:8:13)
	})
```

这个错误对谈返回了一个promise链，对于错误发生的地方没有任何提示。更糟糕的是，它可能会误导；它包含的唯一的函数名称是`callAPromise`，但这并不是正确的错误（尽管这个文件和行数提示依然是有用的）。

然而，async/await中的错误堆栈指出了包含错误的函数

```javascript
const makeRequest = async() => {
  await callAPromise()
  await callAPromise()
  await callAPromise()
  await callAPromise()
  await callAPromise()
   throw new Error("oops")
}

makeRequest()
	.catch(err => {
      console.log(err)
      // 输出
      // Error: oops at makeRequest (index.js:7:9)
	})
```

 这可能在你本地的开发环境，并且你的文件是在编辑器中打开的，这个可能没有太大的区别。但是在理解生产环境中的服务器的错误日志却是非常有用的。在那样的情况下，知道错误发生在`makeRequest`要远远优于知道错误来自于`then`之后的`then`之后的·`then`...

#### 6.调试

最后很重要的一点是，使用async/await非常重要的一个优点是更加容易调试。调试promise经常是非常痛苦的，主要是如下2个原因

1. 你不可以在箭头函数的return表达式下设置断点

   ![debug](https://cdn-images-1.medium.com/max/1000/1*n_V4LaVdBOFgGCbmTR_VKA.png)

2. 如果你再一个`.then`块内设置断点，并且你在使用快捷键跳过的时候，debugger不会跳转到下一个`.then`中，因为它仅仅会在同步代码中跳过。

在async/await中，你不需要在使用那么多的箭头函数，并且你可以就像在普通的同步代码中跳过。

![debug1](https://cdn-images-1.medium.com/max/1000/1*GWYd4eLrs0U96MkNNVB56A.png))

### 总结

Async/await是近些年JavaScript中最具变革性的特点之一。它让你意识到promise语法的混乱，并且为你提供了一种更为直白的方式。

### 值得注意的

你可能对使用这个特性的一些有效的怀疑

* 它让异步代码看起来没有那么明显：我们的眼睛在看到回调或者`.then`的时候意识到这是异步的，当然过一段时间我们可能就适应了，但是在C#中这个特性已经存在多年，熟悉这一点的人应该这点微不足道的，短暂的不便利是值得的。
* Node 7不是一个LTS发布：是的，但是node 8将会在下一个月来临，因此迁移到你的代码到新的版本几乎不要什么额外的工作。
