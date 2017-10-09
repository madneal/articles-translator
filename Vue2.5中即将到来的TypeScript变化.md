
## Upcoming TypeScript Changes in Vue 2.5

## Typing Improvements

We have been receiving requests for better TypeScript integration ever since the release of Vue 2.0. Since the release, we have included official TypeScript type declarations for most of the core libraries (`vue`, `vue-router`, `vuex`). However, the current integration is somewhat lacking when using the out-of-the-box Vue API. For example, TypeScript cannot easily infer the type of `this` inside the default object-based API that Vue uses. To make our Vue code play nicely with TypeScript, we have to use the`[vue-class-component`](https://github.com/vuejs/vue-class-component) decorator, which allows us to author Vue components using a class-based syntax.
For users that preferred a class-based API, this was probably good enough, but it was a bit unfortunate that users had to use a different API just for the sake of type inference. This also made migrating existing Vue codebases to TypeScript more challenging.

Earlier this year, TypeScript introduced a number of [new features](https://github.com/Microsoft/TypeScript/pull/14141) that makes it possible to improve Vue’s type declarations so that TypeScript can better understand the object literal based API. [Daniel Rosenwasser](https://github.com/DanielRosenwasser) from the TypeScript team started an [ambitious PR](https://github.com/vuejs/vue/pull/5887) (now being maintained [here](https://github.com/vuejs/vue/pull/6391) by core team member [HerringtonDarkholme](https://github.com/HerringtonDarkholme)) which, once merged, will provide:

* Proper type inference for `this` when using the default Vue API. It also works inside single-file components!

* Type inference for props on `this` based on the component’s `props`option.

* Most importantly, **these improvements also benefit plain JavaScript users!** If you are using VSCode with the awesome [Vetur](https://github.com/vuejs/vetur) extension, you will get greatly improved autocompletion suggestions and even type hints when using plain JavaScript in Vue components! This is because `[vue-language-server`](https://www.npmjs.com/package/vue-language-server), the internal package that is responsible for analyzing Vue components, can take advantage of the TypeScript compiler to extract more information about your code. Moreover, any editor that supports the language server protocol can leverage `[vue-language-server`](https://github.com/vuejs/vetur/tree/master/server) to provide similar features.

![VSCode + Vetur + New Type Declarations in Action](https://cdn-images-1.medium.com/max/3640/1*ftKUpzYGIzn1eS87JcBS8Q.gif)

For those who are curious, you can try it out today by cloning [this playground project](https://github.com/octref/veturpack/tree/new-types) (make sure to checkout the `new-types` branch) and opening it with VSCode + Vetur!

## Potential Required Actions for TypeScript Users

The typing upgrade will be shipped in Vue 2.5, currently planned to be released around early October. We are releasing it in a minor release because the JavaScript public API is not getting any breaking changes, however, the upgrade does potentially require some actions for existing TypeScript+Vue users. This is why we are announcing the changes now so that you can have enough time to plan for the upgrade.

* The new typings require a minimum of TypeScript 2.4, and it’s recommended to upgrade to the latest version of TypeScript along with Vue 2.5.

* Previously, we already recommend using ES-style imports (`import Vue from ‘vue’`) everywhere with “allowSyntheticDefaultImports”: true in `tsconfig.json`. The new typings will officially move to ES-style import/export syntax, so that config is no longer necessary, and users are required to use ES-style imports in all cases.

* To accompany the export syntax change, the following core libraries that have typings relying on Vue core typing will receive new major versions, and should be upgraded along with Vue core 2.5: `vuex`, `vue-router`, `vuex-router-sync`, `vue-class-component`.

* When performing custom module augmentations, the user should now use `interface VueConstructor` instead of `namespace Vue`. ([example diff](https://github.com/vuejs/vue/pull/6391/files#diff-1c3e3e4cf681d5fde88941717da1058aL11))

* ThisType of `computed`, `watch`, `render` and lifecycle hooks will need manual type annotations if you annotate your component options with as ComponentOptions<Something>

We have tried our best to minimize required upgrading efforts and these type improvements are compatible with the class-based API used in `vue-class-component`. For most users, simply upgrading the dependencies and switching to ES-style imports should be all it takes. In the meanwhile, we also recommend locking your Vue version to `2.4.x` until you are ready to upgrade.

## On the Roadmap: TypeScript Support in vue-cli

After 2.5, we are planning to introduce official support for TypeScript in the next version of vue-cli in order to make it easier for TS+Vue users to kick off new projects. Stay tuned!

## For non-TypeScript users

These changes do not affect non-TypeScript Vue users in any negative way; per semver, 2.5 will be fully backwards compatible in terms of public JavaScript API, and the TypeScript CLI integration will be completely opt-in. But as mentioned above, you will likely notice better auto-completion suggestions if you are using a `[vue-language-server`](https://github.com/vuejs/vetur/tree/master/server) powered editor extension.

—

Thanks to [Daniel Rosenwasser](https://github.com/danielrosenwasser), [HerringtonDarkholme](https://github.com/HerringtonDarkholme), [Katashin](https://github.com/ktsn) and [Pine Wu](https://github.com/octref) for working on these features and reviewing this post.
