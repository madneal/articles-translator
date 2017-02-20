In the previous article
[Native ECMAScript modules: the new features and differences from Webpack modules](https://blog.hospodarets.com/native-ecmascript-modules-new-features)
we understood the differences between ES modules and their implementation in bundlers/compilers like Webpack/Babel.
So far we found couple gotchas and know how to use the `import``export` declarations and
which caveats we may have using them in JS.

But JavaScript went asynchronous many years ago, and it is a good practice is to use non-blocking Promise-based syntax
for modern Web applications.
ECMAScript modules are static by default: you have to define static import/exports on the top level
of the module. It is very helpful to apply JS engine optimizations
but doesn’t allow developers to apply the best practices of asynchronous module loading.

Meet the dynamic `import()` operator, which adds the missed functionality and follows the best practices of
Promise-based API.

# The proposal and the spec

As often, everything started from an idea.
The idea of dynamic import was [introduced and processed](https://github.com/whatwg/loader/issues/149)
by [Domenic Denicola](https://twitter.com/domenic) and the module-loading community.

Currently, we have a [spec draft](https://tc39.github.io/proposal-dynamic-import/)
which is on Stage 3 of the [TC39 process](https://tc39.github.io/process-document/).

It means, that before it is finished (Stage 4), a couple of implementations are required
plus additional gathering and addressing of the feedback from the implementations and users.

And it could be you, as the dynamic `import()` [is shipped](https://bugs.webkit.org/show_bug.cgi?id=165724)
OOB in [Webkit Nightly](https://webkit.org/downloads/).
You can download, start using and test it ([here is a simple demo](https://plnkr.co/edit/XZB9GaIBOatuNwFiTVCS?p=preview)).

Your feedback regarding the dynamic import can be appreciated and you can provide it either via
[the proposal issue tracker](https://github.com/tc39/proposal-dynamic-import/issues)
or comment the [WHATWG proposal](https://github.com/whatwg/loader/issues/149).

# Syntax

The syntax is straightforward:

```
`import("./specifier.js"); // returns a Promise`

```

Here is the list of examples of switching from the static to the dynamic imports
(you can try it [on the demo](https://plnkr.co/edit/jzQkC5GeB7ecmGaoCPDS?p=preview)):

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

![](http://p0.qhimg.com/t013f701dac1e956ac1.png)

Let’s analyze it.
The first surprise- despite we imported `a.js` twice it was invoked only once.
As you may remember, it’s a
[feature of ES modules](https://blog.hospodarets.com/native-ecmascript-modules-new-features#modules-are-singletons)
, as they are singletons and are invoked only once

Secondary, dynamic imports were invoked before the static.
and that is because I included the dynamic `import()` calls in a classic script in my HTML
(yes, you can use dynamic imports in classic scripts as well, not only in module ones!):

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

And the third difference:
the dynamically imported scripts are executed not in the order they appear in the code.
Though the static `import` guarantees you to execute the scripts in the order.
You have to know this feature, as each dynamic import lives by its own and is not connected to/doesn’t wait for
others to be finished.

Let’s summarize the **takeaways**:

*   **dynamic `import()` provides Promise-based API**

*   **`import()` follows the ES modules rules: singleton, specifiers, CORS etc.**

*   **`import()` can be used in both classic and module scripts**

*   **the order of used `import()` in the code doesn’t have anything in common with the order they are resolved**

# Script execution and the context

As we already stated, you can call the `import()` from both classic and module scripts.
But how it’s executed as a module or just in the global context?

As you may expect, the dynamic import executes a script as a module, providing its own context which is different from the global.

We can test it:

```
// imported.js
console.log(`imported.js "this" reference is: ${this}`);

```

“this” reference is pointing to a global object if a script is executed in its context. 
So let’s just execute our example from a [classic script](https://plnkr.co/edit/pHoD7S9kXicUvvpsLoEz?p=preview)
and from a [module one](https://plnkr.co/edit/mHB6R5khaRcUHVbAgWWe?p=preview):

```
<!--module.js-->
`<script type="module" src="module.js">`</script>

<!--classic.js-->
`<script src="classic.js">`</script>

```

```
// module/classic.js
import('./imported.js').then(()=>{
  console.log('imported.js is just imported from the module/classic.js');
});

```

Here is the console output, which shows that in both cases the `imported.js` is executed not in the global context:

![](http://p0.qhimg.com/t010f5ea82cc34c4f22.png)

which means, the `import()` executes the scripts as modules, which actually aligns with the syntax, where in the `then()` function
we can work with the exports from the modules (like `module.default` etc.).

# Additional features

An additional feature you can get from the dynamic import operator is that
you finally can use it not only on the top level of the script. For instance:

```
function loadUserPage(){
    import('user-page.js').then(doStuff);
}

loadUserPage();

```

Which gives you ability to use the lazy loading or import additional features on demand
(e.g. on user actions):

```
// load a script and use it on user actions
FBshareBtn.on('click', ()=>{
    import('/fb-sharing').then((FBshare)=>{
        FBshare.do();
    });
});

```

We already know that the `import()` script will be loaded only once,
which is just an additional advantage for this example.

Even better, the nonstatic nature of the dynamic import allows you to pass the template strings
and construct it depending on your needs, for example ([demo](https://plnkr.co/edit/R05qU5jhPRW6Hqt47xGQ?p=preview)):

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

```
if(user.loggedIn){
    import('user-widget.js');
}

```

**Takeaways:**

*   **you can use dynamic import for lazy or conditional loading and in user-depending action**

*   **dynamic `import()` can be used anywhere in your scripts**

*   **`import()` takes string literals and you can construct the specifier depending on your needs**

## Promise API advantages

So the dynamic import uses JS Promise API.
What advantages does it give us?

First of all, we can load multiple scripts dynamically in parallel.
Let’s rework our [initial example](https://plnkr.co/edit/jzQkC5GeB7ecmGaoCPDS?p=preview)
to trigger and catch the loading of multiple scripts:

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

There is also the [Promise.race](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Promise/race)
method, which checks which Promise is resolved or rejected first (faster).

In the case of `import()` we can use it e.g. to check [which CDN works faster](https://plnkr.co/edit/wmU3Se9JuMlhblaVu1Oo?p=preview):

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

![](http://p0.qhimg.com/t01fc95cf3aea1b0a40.png)

Of course, it may be a bit strange method, just wanted to show you, that you can use all the power of Promises-based API.

And finally, let’s get a bit of the syntax sugar.
[ECMAScript async/await feature is also Promise-based](http://www.2ality.com/2016/10/async-function-tips.html),
which means you can easily reuse dynamic imports with it.
So let’s try to have a syntax similar to static imports, but with all the power of the dynamic `import()`
([demo](https://plnkr.co/edit/cL07FqPLWqKWjEizTe2b?p=info)):

```
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

```
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

**Takeaways:**

*   **use `Promise.all` to load modules in parallel**

*   **all Promise API power is available for the `import()` operator usages**

*   **you can use the dynamic import with async/await**

## Promise API caveats

There is an additional caveat from Promises nature we always have to remember- the error handling.
If there is any error in static import with the specifier or in module graph or even during the execution-
the error is thrown automatically.

In the case of the Promises, you either should provide a second function to `then()` method,
or catch errors in the `catch()` construction, otherwise your app never will know about it.

Here is a [demo of importing a nonexisting script](https://plnkr.co/edit/18ZYVtBwCX9cacKqMuLq?p=preview):

```
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

Here how you can add the global unhandled Promises listener:

```
window.addEventListener("unhandledrejection", (event)=> {
  console.warn(`WARNING: Unhandled promise rejection. Reason: ${event.reason}`);
  console.warn(event);
});
// process.on('unhandledRejection'... in case of Node.js

```

# Caveats

Let’s discuss the relative paths in the `import()` specifier.
As you may expect, it is relative to the file, from which it is called.
It may lead to the caveats when you want to import a module from a different folder
and a method to do it is in some third location (e.g. a  `utils` folder or similar).

Let’s consider the following folder structure and the code:

![](http://p0.qhimg.com/t0124ba395654942d8f.png)

```
// utils.js - is used to load a dependency
export const loadDependency = (src) => {
    return import(src)
        .then((module) => {
            console.log('dependency is loaded');
            return module;
        })
};

// inner.js - the main file we will use to test the passed import() path
import {loadDependency} from '../utils.js';

loadDependency('../dependency.js');
// Failed to load resource, as import() is called in ../dependency.js

loadDependency('./dependency.js');// Successfully loaded

```

[Demo](https://blog.hospodarets.com/demos/native-ecmascript-modules-dynamic-import/) 

As it is shown in the demo, the `import()` specifier is always relative to the file it’s called from,
so always keep in mind this fact to avoid unexpected bugs.

Regarding the debugging- the good of all this, that finally, you can play with ES modules in browsers DevTools console,
as `import()` is available from anywhere, but there is still [a bug](https://bugs.webkit.org/show_bug.cgi?id=165724#c19) and work in progress, till you can try this in Webkit

**Takeaways:**

*   **`import()` specifier is always related to the file its called**

*   **you can use dynamic imports in DevTools console (useful for debugging)**

# Support and polyfills

For now, `import()` has little browser support.
[Node.js is considering adding this feature](https://medium.com/@jasnell/an-update-on-es6-modules-in-node-js-42c958b890c#.e4eyz1aew)
which may look something like `require.import()`.

To detect if it’s supported in a particular browser or Node.js, run the following code
or try [the demo](https://plnkr.co/edit/pgsxdd6hE7uzQY9dHCwz?p=preview):

```
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