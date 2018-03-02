![](https://cdn-images-1.medium.com/max/9900/1*xmqGcZXL4t7mJoG1SBvErA.jpeg)

## 7 Hacks for ES6 Developers

Following the original [JavaScript hacks](https://hackernoon.com/javascript-hacks-for-hipsters-624d50c76e8e), here’s some new goodies.  *Coding JavaScript in 2018 is actually fun again！*

关注原来的 [JavaScript hacks](https://hackernoon.com/javascript-hacks-for-hipsters-624d50c76e8e)，上面有一些新的好东西。*2018 使用 JavaScript 写代码真的又变得有意思了！*

![](https://cdn-images-1.medium.com/max/2000/1*4877k4Hq9dPdtmvg9hnGFA.jpeg)

## Hack #1 — 交换变量

使用数组结构来交换值

    let a = 'world', b = 'hello'
    [a, b] = [b, a]
    console.log(a) // -> hello
    console.log(b) // -> world
    
    // 是的，很神奇

## Hack #2 — 使用解构的 Async/Await

Once again, Array Destructuring is great. Combined with async/await and promises to make a complex flow — simple.

再说一遍，数组结构真的很棒。通过和 async/await 以及 promise 结合可以让复杂的流程变得简单。

    const [user, account] = await Promise.all([
      fetch('/user'),
      fetch('/account')
    ])

## Hack #3 — 调试

For anyone who likes to debug using console.logs, here’s something awesome (and yes, I heard of console.table):

对于那些喜欢使用 console.logs 来调试的人来说，现在有一些特别酷的（并且我也听说过 console.table）：

    const a = 5, b = 6, c = 7
    console.log({ a, b, c })
    
    // 输出优雅的对象:
    // {
    //    a: 5,
    //    b: 6,
    //    c: 7
    // }

## Hack #4 — 一行搞定一切

Syntax can be so much more compact for array operations

对于数组操作，语法可以非常紧凑

    // 寻找最大值
    const max = (arr) => Math.max(...arr);
    max([123, 321, 32]) // outputs: 321
    
    // 对数组求和
    const sum = (arr) => arr.reduce((a, b) => (a + b), 0)
    sum([1, 2, 3, 4]) // output: 10

## Hack #5 — 数组拼接

The spread operator can be used instead of concat:

拓展操作符可以用来代替 concat：

    const one = ['a', 'b', 'c']
    const two = ['d', 'e', 'f']
    const three = ['g', 'h', 'i']
    
    // 老方法 #1
    const result = one.concat(two, three)
    
    // 老方法 #2
    const result = [].concat(one, two, three)
    
    // 新方法
    const result = [...one, ...two, ...three]

## Hack #6 — 克隆

Clone arrays and objects with ease:

轻松克隆数组和对象：

    const obj = { ...oldObj }
    const arr = [ ...oldArr ]

Note: This creates a shallow clone.

注意：这会产生一个浅克隆。

## Hack #7 — 命名参数

Making function and function calls more readable with destructuring:

通过结构让函数以及函数函数调用更具有可读性：

```javascript
const getStuffNotBad = (id, force, verbose) => {
  ...do stuff
}
const getStuffAwesome = ({ id, name, force, verbose }) => {
  ...do stuff
}

// 在代码的其它某个地方... 到底什么是 true, true?
getStuffNotBad(150, true, true)

// 在代码的其他某个地方.. I ❤ JS!!!
getStuffAwesome({ id: 150, force: true, verbose: true })
```

![](https://cdn-images-1.medium.com/max/2048/1*ZrJKJqBsksWd-8uKM9OvgA.png)

**已经全部知道了?**

You’re a true hacker, let’s continue the conversation on [**Twitter**](http://www.twitter.com/ketacode)You can also check my startup [Torii](https://toriihq.com) where we make “SaaS headache” go away.

你是一个真正的黑客，让我们继续在 Twitter上的谈话你还可以看看我的 [Torii](https://toriihq.com)  教学，我们让“SaaS头痛”消失。