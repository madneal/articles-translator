## Vue 2.5 发布了

We are excited to announce the release of Vue 2.5 Level E! This release includes improvements of various features and we recommend checking out the [release note](https://github.com/vuejs/vue/releases/tag/v2.5.0) for full details. In this post we are going to highlight some of the more prominent changes: better TypeScript integration, better error handling, better support for functional components in single-file components, and environment-agnostic server-side rendering.

## Better TypeScript Integration

![](https://cdn-images-1.medium.com/max/3200/1*vB-z-t961mJnd4a6re02Iw.png)

With the help from the TypeScript team, 2.5 offers greatly improved type declarations that works with Vue’s out-of-the-box API without the need for a component class decorator. The new type declarations also power editor extensions like [Vetur](https://marketplace.visualstudio.com/items?itemName=octref.vetur), enabling better Intellisense support for plain JavaScript users as well. For more details, checkout our [earlier post regarding the changes](https://medium.com/the-vue-point/upcoming-typescript-changes-in-vue-2-5-e9bd7e2ecf08).

*Shoutout to [Daniel Rosenwasser](https://github.com/danielrosenwasser) from the TypeScript team for starting the PR, and to core team members [Herrington Darkholme](https://github.com/HerringtonDarkholme) and [Katashin](https://github.com/ktsn) for improving and reviewing the changes.*
>  Note: TypeScript users should also update the following packages to the latest version for type declaration compatibility: `vue-router`, `vuex`, `vuex-router-sync` and `vue-class-component`.

## Better Error Handling

![](https://cdn-images-1.medium.com/max/2000/1*ZHamhzmnoQcQTxCJE3cmvA.jpeg)

In 2.4 and earlier versions, we typical use the global `config.errorHandler `option for handling unexpected errors in our applications. We also have the `renderError` component option for handling errors in render functions. However, we lack a mechanism for handling generic errors inside a specific part of the application.

In 2.5 we have introduced the new `errorCaptured` hook. A component with this hook captures all errors (excluding those fired in async callbacks) from its child component tree (excluding itself). If you are familiar with React, this is similar to the concept of [Error Boundaries](https://reactjs.org/blog/2017/07/26/error-handling-in-react-16.html#introducing-error-boundaries) introduced in React 16. The hook receives the same arguments as the global `errorHandler`, and you can leverage this hook to [gracefully handle and display the error](https://gist.github.com/yyx990803/9bdff05e5468a60ced06c29c39114c6b#error-handling-with-errorcaptured-hook).

## Better Support for Functional Components in `SFCs`

![](https://cdn-images-1.medium.com/max/2828/1*jg9qGPkPadGBEa-KUPrMpA.png)

With `vue-loader >= 13.3.0` and Vue 2.5, functional components defined as a Single-File Component in a `*.vue` file now enjoys proper [template compilation, Scoped CSS and hot-reloading support](https://vue-loader.vuejs.org/en/features/functional.html). This makes it much easier to convert leaf-components into functional ones for performance optimizations.

*Shoutout to core team member [Blake Newman](https://github.com/blake-newman) for his contribution to these features.*

## Environment Agnostic SSR

The default build of `vue-server-renderer` assumes a Node.js environment, which makes it unusable in alternative JavaScript runtimes such as [php-v8js](https://github.com/phpv8/v8js) or Nashorn. In 2.5 we have shipped [a build of `vue-server-renderer` that is environment-agnostic](https://github.com/vuejs/vue/blob/dev/packages/vue-server-renderer/basic.js) which can be used in the browser or in pure JavaScript engines. This could open up interesting strategies such as [utilizing Vue server-rendering directly in your php process](https://gist.github.com/yyx990803/9bdff05e5468a60ced06c29c39114c6b#environment-agnostic-ssr).

Again, we recommend checking out the [full release note](https://github.com/vuejs/vue/releases/tag/v2.5.0) for improvements in other APIs, including `v-on`, `v-model`, scoped slots, provide/inject and more. You might also be interested in our [public roadmap](https://github.com/vuejs/roadmap) detailing what the team is working on. Cheers!