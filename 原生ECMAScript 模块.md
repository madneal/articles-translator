In the previous article
[Native ECMAScript modules: the new features and differences from Webpack modules](https://blog.hospodarets.com/native-ecmascript-modules-new-features)
we understood the differences between ES modules and their implementation in bundlers/compilers like Webpack/Babel.

在之前的文章[原生ECMAScript 模块：新特点以及与Webpack模块的区别](https://blog.hospodarets.com/native-ecmascript-modules-new-features)，我们弄明白了ES模块和它们在bundlers/compilers(比如Webpack/Babel)实现的区别。

So far we found couple gotchas and know how to use the `import``export` declarations and
到目前为止我们已经发现了几个问题，并且直到如何使用`import``export` 声明并且这些问题警告我们可能已经在JS中使用了这些。

But JavaScript went asynchronous many years ago, and it is a good practice is to use non-blocking Promise-based syntax for modern Web applications. ECMAScript modules are static by default: you have to define static import/exports on the top level of the module. It is very helpful to apply JS engine optimizations but doesn’t allow developers to apply the best practices of asynchronous module loading.

但是JavaScript在异步方面已经发展多年，这是一个是对现代Web应用程序使用基于Promise的非阻塞语法的很好的做法。 默认情况下，ECMAScript模块是静态的：您必须在模块的顶层定义静态导入/导出。 应用JS引擎优化非常有用，但其不允许开发人员应用异步模块加载的最佳实践。

Meet the dynamic `import()` operator, which adds the missed functionality and follows the best practices of Promise-based API.

为了满足动态import()操作符，增加了思念已久的功能，并遵循基于Promise的API的最佳实践。

# 建议和规范

As often, everything started from an idea.The idea of dynamic import was [introduced and processed](https://github.com/whatwg/loader/issues/149)
by [Domenic Denicola](https://twitter.com/domenic) and the module-loading community.

通常，一切事物都是从一个想法开始的。动态导入的想法是 [Domenic Denicola](https://twitter.com/domenic) 和module-loading社区介绍并且推进的。

Currently, we have a [spec draft](https://tc39.github.io/proposal-dynamic-import/) which is on Stage 3 of the [TC39 process](https://tc39.github.io/process-document/).

目前，我们有一个[规范草案](https://tc39.github.io/proposal-dynamic-import/)，在[TC39过程的第3阶段](https://tc39.github.io/process-document/)。

It means, that before it is finished (Stage 4), a couple of implementations are required plus additional gathering and addressing of the feedback from the implementations and users.

这意味着在阶段4完成之前，还需要收集一些实现以及来自于用户的反馈。

And it could be you, as the dynamic `import()` [is shipped](https://bugs.webkit.org/show_bug.cgi?id=165724) OOB in [Webkit Nightly](https://webkit.org/downloads/). You can download, start using and test it ([here is a simple demo](https://plnkr.co/edit/XZB9GaIBOatuNwFiTVCS?p=preview)).

您也可能就是用户之一，因为动态`import`可以在 [Webkit Nightly](https://webkit.org/downloads/)中运行[OOB](ttps://bugs.webkit.org/show_bug.cgi?id=165724)。你可以下载，开始使用并且测试（这是一个简单的[demo](https://plnkr.co/edit/XZB9GaIBOatuNwFiTVCS?p=preview)）。

Your feedback regarding the dynamic import can be appreciated and you can provide it either via
[the proposal issue tracker](https://github.com/tc39/proposal-dynamic-import/issues)or comment the [WHATWG proposal](https://github.com/whatwg/loader/issues/149).

我们将会感谢您对于动态导入的反馈，您可以通过[提案issue跟踪器](https://github.com/tc39/proposal-dynamic-import/issues)或者在[WHATWG提案](https://github.com/whatwg/loader/issues/149)上评论。

# 语法

The syntax is straightforward:

语法非常直接：

```
`import("./specifier.js"); // returns a Promise`

```

Here is the list of examples of switching from the static to the dynamic imports(you can try it [on the demo](https://plnkr.co/edit/jzQkC5GeB7ecmGaoCPDS?p=preview)):

下面是从静态导入向动态导入切换例子的列表（你可以在[demo](https://plnkr.co/edit/jzQkC5GeB7ecmGaoCPDS?p=preview)上进行尝试）：

```
// STATIC
import './a.js';

import b from './b.js';
b();

import {c} from './c.js';
c();

// DYNAMIC
import('./a.js').then(()=>{
  console.log('a.js is loaded dynamically');
});

import('./b.js').then((module)=>{
  const b = module.default;
  b('isDynamic');
});

import('./c.js').then(({c})=>{
  c('isDynamic');
});

```

`'isDynamic'` is passed to make the difference how the function in the module was called.
Here is the Dev Console screenshot:

`'isDynamic'` 被传递过来区别于在模块中调用函数。下面是DEV控制台截图：

![](http://p0.qhimg.com/t013f701dac1e956ac1.png)

Let’s analyze it. The first surprise- despite we imported `a.js` twice it was invoked only once.
As you may remember, it’s a [feature of ES modules](https://blog.hospodarets.com/native-ecmascript-modules-new-features#modules-are-singletons), as they are singletons and are invoked only once

让我们一起来分析以下。第一个惊喜-尽管我们两次导入了`a.js`，但是它仅仅被调用了一次。你可能记得，这是[ES模块的特性](https://blog.hospodarets.com/native-ecmascript-modules-new-features#modules-are-singletons)，因为它们是单例的，只会被调用一次。

Secondary, dynamic imports were invoked before the static.
and that is because I included the dynamic `import()` calls in a classic script in my HTML
(yes, you can use dynamic imports in classic scripts as well, not only in module ones!):

另外，动态导入在静态导入之前被调用。 这是因为我在我的HTML中包含了经典脚本中的动态`import()`调用（是的，你也可以在经典脚本中使用动态导入，不仅仅是在模块中！）：

```
`<script type="module" src="static.js">`</script>
`<script src="dynamic.js">`</script>

```

As we know, `type="module"` 
[scripts are deferred by default](https://blog.hospodarets.com/native-ecmascript-modules-the-first-overview#how-the-browser-loads-and-executes-the-modules)
and wait till the DOM
is parsed and then are invoked in the order.
That’s why the `dynamic` script was executed first.
The ability to use `import()` in classic scripts gives you the key to whole
the native ES modules from the classic JS ones- you can load and work with them from anywhere.

我们知道，`type =“module”`的[脚本默认延迟加载](https://blog.hospodarets.com/native-ecmascript-modules-the-first-overview#how-the-browser-loads-and-executes-the-modules)，直到DOM解析完毕，然后按顺序调用。 这就是为什么动态脚本首先被执行的原因。 在经典脚本中使用`import()`的能力为您提供了来自经典JS的整个本地ES模块的关键 - 您可以从任何地方加载和使用它们。

And the third difference:
the dynamically imported scripts are executed not in the order they appear in the code.
Though the static `import` guarantees you to execute the scripts in the order.
You have to know this feature, as each dynamic import lives by its own and is not connected to/doesn’t wait for
others to be finished.

第三个区别：动态导入的脚本不按它们在代码中出现的顺序执行。 虽然静态导入保证您按顺序执行脚本。 你必须知道这个特性，因为每个动态导入都是由它自己生成的，并且和其他的完成没有关系同时也不会等待其他的完成。

Let’s summarize the **takeaways**:

让我们总结以下：

*   **动态的 `import()` 提供一个基于Promise的API**

*   **`import()` 遵循ES模块规则：singleton，说明符，CORS等.**

*   **`import()` 可以在经典脚本和模块脚本中使用**

*   **在代码中使用的`import()`的顺序与它们被解析的顺序没有什么共同之处**

# 脚本执行以及上下文

As we already stated, you can call the `import()` from both classic and module scripts.
But how it’s executed as a module or just in the global context?

正如我们已经说明的，您可以从经典和模块脚本调用`import()`。 但是它如何作为一个模块或只是在全局上下文中执行？

As you may expect, the dynamic import executes a script as a module, providing its own context which is different from the global.

正如你所期望的，动态导入将脚本作为模块执行，提供与全局不同的上下文。

We can test it:

我们可以测试一下：

```
// imported.js
console.log(imported.js "this" reference is: ${this});
```

“this” reference is pointing to a global object if a script is executed in its context. 
So let’s just execute our example from a [classic script](https://plnkr.co/edit/pHoD7S9kXicUvvpsLoEz?p=preview)
and from a [module one](https://plnkr.co/edit/mHB6R5khaRcUHVbAgWWe?p=preview):

如果在其上下文中执行脚本，“this”引用指向一个全局对象。 所以让我们从一个[经典脚本](https://plnkr.co/edit/pHoD7S9kXicUvvpsLoEz?p=preview)和一个[模块](https://plnkr.co/edit/mHB6R5khaRcUHVbAgWWe?p=preview)执行我们的例子：

```html
<!--module.js-->
<script type="module" src="module.js"></script>

<!--classic.js-->
<script src="classic.js"></script>
```

```
// module/classic.js
import('./imported.js').then(()=>{
  console.log('imported.js is just imported from the module/classic.js');
});

```

Here is the console output, which shows that in both cases the `imported.js` is executed not in the global context:

下面是控制台输出，显示在这两种情况下，`imported.js`不是在全局上下文中执行：

![](http://p0.qhimg.com/t010f5ea82cc34c4f22.png)

which means, the `import()` executes the scripts as modules, which actually aligns with the syntax, where in the `then()` function
we can work with the exports from the modules (like `module.default` etc.).

这意味着，`import()`将脚本作为模块执行，这实际上与语法一致，在`then`函数中，我们可以使用模块的输出（如`module.default`等）。

# 额外特性

An additional feature you can get from the dynamic import operator is that
you finally can use it not only on the top level of the script. For instance:

您可以从动态导入操作符获得的一个附加功能是，您最终可以不仅在脚本的顶层使用它。 例如：

```
function loadUserPage(){
    import('user-page.js').then(doStuff);
}

loadUserPage();

```

Which gives you ability to use the lazy loading or import additional features on demand
(e.g. on user actions):

这使您能够使用延迟加载或按需导入其他功能（例如关于用户操作）：

```javascript
// 加载脚本并且相应用户操作
FBshareBtn.on('click', ()=>{
    import('/fb-sharing').then((FBshare)=>{
        FBshare.do();
    });
});
```

We already know that the `import()` script will be loaded only once,
which is just an additional advantage for this example.

我们已经知道`import()`脚本只会被加载一次，这只是这个例子的一个额外的优点。

Even better, the nonstatic nature of the dynamic import allows you to pass the template strings
and construct it depending on your needs, for example ([demo](https://plnkr.co/edit/R05qU5jhPRW6Hqt47xGQ?p=preview)):

更好的是，动态导入的非静态性质允许您传递模板字符串并根据您的需要构建它，例如（[demo](https://plnkr.co/edit/R05qU5jhPRW6Hqt47xGQ?p=preview)）：

```
const locale = 'en';
import(`./utils_${locale}.js`).then(
  (utils)=>{
    console.log('utils', utils);
    utils.default();
  }
);

```

As you already noticed, the default import is available under the `module.default` property.

And, of course, you can do the conditional loading:

正如您已经注意到的，默认导入在module.default属性下可用。

当然，你可以做条件加载：

```
if(user.loggedIn){
    import('user-widget.js');
}

```

**注意**

*   **您可以使用动态导入进行延迟或条件加载以及依赖用户的操作**

*   **动态的`import()` 可以在脚本的任何地方使用**

*   **`import()`接受字符串文字，你可以根据你的需要构造说明符**

## Promise API 优点

So the dynamic import uses JS Promise API.
What advantages does it give us?

First of all, we can load multiple scripts dynamically in parallel.
Let’s rework our [initial example](https://plnkr.co/edit/jzQkC5GeB7ecmGaoCPDS?p=preview)
to trigger and catch the loading of multiple scripts:

所以动态导入使用JS Promise API。 它给我们什么好处？

首先，我们可以并行地动态加载多个脚本。 让我们重做我们的[初始示例](https://plnkr.co/edit/jzQkC5GeB7ecmGaoCPDS?p=preview)来触发和捕获多个脚本的加载：

```
Promise.all([
        import('./a.js'),
        import('./b.js'),
        import('./c.js'),
    ])
    .then(([a, {default: b}, {c}]) => {
        console.log('a.js is loaded dynamically');

        b('isDynamic');

        c('isDynamic');
    });

```

I used [JavaScript destructuring](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)
to avoid something like `const _b = b.default;` in my script.

我使用JavaScript解构避免在我的脚本出现像`const _b = b.default` 。

There is also the [Promise.race](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Promise/race)
method, which checks which Promise is resolved or rejected first (faster).

还有`Promise.race`方法，它检查哪个Promise被首先（更快）resolved或reject。

In the case of `import()` we can use it e.g. to check [which CDN works faster](https://plnkr.co/edit/wmU3Se9JuMlhblaVu1Oo?p=preview):

我们可以使用`import()`来检查[哪个CDN速度更快](https://plnkr.co/edit/wmU3Se9JuMlhblaVu1Oo?p=preview)：

```
const CDNs = [
  {
    name: 'jQuery.com',
    url: 'https://code.jquery.com/jquery-3.1.1.min.js'
  },
  {
    name: 'googleapis.com',
    url: 'https://ajax.googleapis.com/ajax/libs/jquery/3.1.1/jquery.min.js'
  }
];

console.log(`------`);
console.log(`jQuery is: ${window.jQuery}`);

Promise.race([
  import(CDNs[0].url).then(()=>console.log(CDNs[0].name, 'loaded')),
  import(CDNs[1].url).then(()=>console.log(CDNs[1].name, 'loaded'))
]).then(()=> {
  console.log(`jQuery version: ${window.jQuery.fn.jquery}`);
});

```

And here is the console output after a couple of reloads, which shows that the method shows
which CDN-loaded the file faster (notice, `import()`s load and execute the both files, in this case, registering jQuery):

这里是几个重新加载后的控制台输出，这显示方法的结果。其中CDN加载文件更快（通知，`import()`加载和执行这两个文件，在这种情况下，注册jQuery）：

![](http://p0.qhimg.com/t01fc95cf3aea1b0a40.png)

Of course, it may be a bit strange method, just wanted to show you, that you can use all the power of Promises-based API.

当然，它可能有点奇怪的方法，只是想告诉你，你可以使用基于Promises的API的所有能力。

And finally, let’s get a bit of the syntax sugar.
[ECMAScript async/await feature is also Promise-based](http://www.2ality.com/2016/10/async-function-tips.html),
which means you can easily reuse dynamic imports with it.
So let’s try to have a syntax similar to static imports, but with all the power of the dynamic `import()`
([demo](https://plnkr.co/edit/cL07FqPLWqKWjEizTe2b?p=info)):

最后，让我们得到一点语法糖。 [ECMAScript async/await特性](http://www.2ality.com/2016/10/async-function-tips.html)也是基于Promise的，这意味着您可以轻松地使用动态导入。 因此，让我们尝试使用与静态导入类似的语法，但具有动态`import()`（[demo](https://plnkr.co/edit/cL07FqPLWqKWjEizTe2b?p=info)）的所有功能：

```javascript
// utils_en.js
const test = (isDynamic) => {
  let prefix;
  if (isDynamic) {
    prefix = 'Static import';
  } else {
    prefix = 'Dynamic import()';
  }

  const phrase = `${prefix}: ECMAScript dynamic module loader
                    "import()" works in this browser`;
  console.log(phrase);
  alert(phrase);
};

export {test};
```

```javascript
// STATIC
import {test} from './utils_en.js'; // no dynamic locale
test();

// DYNAMIC
(async () => {
  const locale = 'en';

  const {test} = await import(`./utils_${locale}.js`);
  test('isDynamic');
})();
```

**注意**

*   **使用`Promise.all`并行加载模块**

*   **所有的Promise API 对于 `import()` 操作符都是可用的**

*   **您可以通过async/await来动态导入**

## Promise API 警告

There is an additional caveat from Promises nature we always have to remember- the error handling.
If there is any error in static import with the specifier or in module graph or even during the execution-
the error is thrown automatically.

我们总是要记住Promise本质中一个额外的警告 - 错误处理。 如果使用说明符或在模块图中或者在执行期间的静态导入中存在任何错误，则会自动抛出错误。

In the case of the Promises, you either should provide a second function to `then()` method,
or catch errors in the `catch()` construction, otherwise your app never will know about it.

在使用Promises的时候，你应该为`then()`方法提供第二个函数，或者在`catch()`中捕获错误，否则你的应用程序永远不会知道错误。

Here is a [demo of importing a nonexisting script](https://plnkr.co/edit/18ZYVtBwCX9cacKqMuLq?p=preview):

下面是一个导入了不存在脚本的[demo](https://plnkr.co/edit/18ZYVtBwCX9cacKqMuLq?p=preview)：

```javascript
 import (`./non-existing.js`)
    .then(console.log)
   .catch((err) => {
     console.log(err.message); // "Importing a module script failed."
     // apply some logic, e.g. show a feedback for the user
   });
```

Since the recent time, the browsers/Node.js didn’t provide you any information if some of
your Promises was rejected and you didn’t handle that.
So the community introduced the ability to have a global handler,
without which you have errors in browser console, or in the case of Node.js application is terminated with a non-zero code.

从最近开始，如果你没有处理你的被reject的Promise，浏览器或者Node.js不会提供任何信息。 因此，社区引入了具有全局处理程序的能力，如果没有它，您在浏览器控制台中有错误，或者在Node.js应用程序都会以非零代码中止运行。

Here how you can add the global unhandled Promises listener:

下面是如何添加全局未处理Promises监听：

```javascript
window.addEventListener("unhandledrejection", (event)=> {
  console.warn(`WARNING: Unhandled promise rejection. Reason: ${event.reason}`);
  console.warn(event);
});
// process.on('unhandledRejection'... in case of Node.js
```

# 警告

Let’s discuss the relative paths in the `import()` specifier.
As you may expect, it is relative to the file, from which it is called.
It may lead to the caveats when you want to import a module from a different folder
and a method to do it is in some third location (e.g. a  `utils` folder or similar).

让我们讨论`import()`说明符中的相对路径。 正如你所期望的，它是相对于文件，从它被调用时。 当您要从不同的文件夹导入模块以及在第三个位置（例如`utils`文件夹或类似文件夹）执行此操作的方法时，可能会导致警告。

Let’s consider the following folder structure and the code:

让我们看一下下面的文件结构和代码：

![](http://p0.qhimg.com/t0124ba395654942d8f.png)

```
// utils.js - 用于加载依赖
export const loadDependency = (src) => {
    return import(src)
        .then((module) => {
            console.log('dependency is loaded');
            return module;
        })
};

// inner.js - 主文件将被用来测试传递的import()路径
import {loadDependency} from '../utils.js';

loadDependency('../dependency.js');
// 加载资源失败,因为import()是在 ../dependency.js中调用

loadDependency('./dependency.js');// 成功加载
```

[Demo](https://blog.hospodarets.com/demos/native-ecmascript-modules-dynamic-import/) 

As it is shown in the demo, the `import()` specifier is always relative to the file it’s called from,
so always keep in mind this fact to avoid unexpected bugs.

Regarding the debugging- the good of all this, that finally, you can play with ES modules in browsers DevTools console,
as `import()` is available from anywhere, but there is still [a bug](https://bugs.webkit.org/show_bug.cgi?id=165724#c19) and work in progress, till you can try this in Webkit

正如demo所演示的那样，`import()`说明符总是相对于调用它的文件，因此记住这点从而避免意想不到的bug。

关于调试-好处是你终于可以在浏览器开发工具控制台中使用ES模块，因为`import()`在任何地方都是可用的，但是在`Webkit`以外的地方，还是有一个bug。

**注意：:**

*   **`import()` 说明符总是相对于调用它的文件**

*   **你可以在浏览器开发工具控制台使用动态导入 (对于调试很有帮助)**

# 支持和polyfills

For now, `import()` has little browser support.
[Node.js is considering adding this feature](https://medium.com/@jasnell/an-update-on-es6-modules-in-node-js-42c958b890c#.e4eyz1aew)
which may look something like `require.import()`.

To detect if it’s supported in a particular browser or Node.js, run the following code
or try [the demo](https://plnkr.co/edit/pgsxdd6hE7uzQY9dHCwz?p=preview):

现在，`import()`几乎没有浏览器支持。 [Node.js正在考虑添加这个功能](https://medium.com/@jasnell/an-update-on-es6-modules-in-node-js-42c958b890c#.e4eyz1aew)，可能看起来像`require.import()`。

要检测它是否在特定浏览器或Node.js中受支持，请运行以下代码或尝试[demo](https://plnkr.co/edit/pgsxdd6hE7uzQY9dHCwz?p=preview)：

```javascript
let dynamicImportSupported = false;
try{
 Function('import("")');
 dynamicImportSupported = true;
}catch(err){};

console.log(dynamicImportSupported);
```

Regarding the polyfills, module-loading community prepared an
[`importModule` function solution](https://github.com/tc39/proposal-dynamic-import#using-host-specific-mechanisms),
which gives similar functionality to `import()`:

```
function importModule(url) {
  return new Promise((resolve, reject) => {
    const script = document.createElement("script");
    const tempGlobal = "__tempModuleLoadingVariable" +
        Math.random().toString(32).substring(2);
    script.type = "module";
    script.textContent = `import * as m from "${url}"; window.${tempGlobal} = m;`;

    script.onload = () => {
      resolve(window[tempGlobal]);
      delete window[tempGlobal];
      script.remove();
    };

    script.onerror = () => {
      reject(new Error("Failed to load module script with URL " + url));
      delete window[tempGlobal];
      script.remove();
    };

    document.documentElement.appendChild(script);
  });
}

```

But the solution has many problems, so you can check it just for a common knowledge.

Babel provides the [`dynamic-import-webpack` plugin](https://github.com/airbnb/babel-plugin-dynamic-import-webpack) for such a syntax,
which you can install and your `import()` operators will be processed by it.

[Webpack 2 supports code splitting using dynamic `import()`](https://webpack.js.org/guides/migrating/#code-splitting-with-es2015) 
out of the box,
where previously you used `require.ensure`

## importScripts(urls) alternative

In Worker / ServiceWorker scripts [importScripts(urls)](https://developer.mozilla.org/en-US/docs/Web/API/WorkerGlobalScope/importScripts)
interface is used to synchronously import one or more scripts into the worker’s scope.
It’s syntax is quite simple:

```
`importScripts('foo.js', 'bar.js' /*, ...*/);`

```

You may think about `import()` as an advanced, asynchronous and non-blocking version of `importScripts()`.

Everything noticeable for the community regarding that is, that when a Worker type is 
“module”,
[then try of the `importScripts` usage throws a TypeError exception](https://html.spec.whatwg.org/multipage/workers.html#importing-scripts-and-libraries).

As the dynamic import works everywhere, it’s a nice idea to start reworking any `importScripts()`
usages to the dynamic `import()` when it works in all the supported browsers.
Also double check the scope, in which module is executed to avoid problems.

# In the end

Dynamic `import()` brings us the additional power to use the ES modules in an asynchronous way.
To load them dynamically or conditionally depending on our needs,
which gives us the ability to create even more advantage apps faster and better.

Webpack 2 uses this API and it’s currently on the Stage 3 with the already existing implementation in the browser,
which means very soon the spec is supposed to become a standard.

Here are the additional links for you:

*   [ES proposal: import() – dynamically importing ES modules](http://www.2ality.com/2017/01/import-operator.html)

*   [import() spec draft](https://tc39.github.io/proposal-dynamic-import/)

*   [node-es-module-loader](https://www.npmjs.com/package/node-es-module-loader)
            and [systemjs](https://github.com/systemjs/systemjs) by [Guy Bedford](https://twitter.com/guybedford)
          ​              