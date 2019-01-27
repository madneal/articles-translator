# 基于Vue JS, Webpack 以及Material Design的渐进式web应用 [Part 1]

> 原文：[基于Vue JS, Webpack 以及Material Design的渐进式web应用 [Part 1]](https://blog.sicara.com/a-progressive-web-application-with-vue-js-webpack-material-design-part-1-c243e2e6e402)
>
> 译者：[neal1991](https://github.com/neal1991)
>
> welcome to star my [articles-translator](https://github.com/neal1991/articles-translator/), providing you advanced articles translation. Any suggestion, please issue or contact [me](mailto:bing@stu.ecnu.edu.cn)
>
> LICENSE: [MIT](https://opensource.org/licenses/MIT)



渐进式web应用是大势所趋。越来越多的大公司开始使用这些技术（比如推特：https://mobile.twitter.com/）。

想象你可以在地铁中浏览一个web应用，这个应用能够向用户推送通知并且提供实时的数据，以及类似于app的浏览，这些就是PWA的大致能力。

渐进式web应用（PWA）是一个web应用能够，提供给用户一种类似于app的体验。PWA得益于现代web科技创新（Service Workers, Native APIS, JS frameworks），并且提升的web应用质量标准。

![](https://cloud.githubusercontent.com/assets/12164075/26136809/b7478288-3af0-11e7-9f2b-cd7b4a1595a0.png)

如果你想了解更多关于PWA，请访问这个很棒的[Google developer page](https://developers.google.com/web/progressive-web-apps/)。

看一下下面的PWA！看起来很像原生的app，是不是？

![](https://cloud.githubusercontent.com/assets/12164075/26137264/b3dbd31c-3af3-11e7-93bb-12822c370704.png)

推特渐进式web应用

从开发者的角度来看，PWA相对于原生应用具有巨大的优点。它基本上就是一个网站，因此：

*   你可以选择任何你喜欢的框架来进行开发；

*   一段代码搞定一切：它是跨平台的以及跨设备的（代码是通过用户的浏览器执行的）；

*   很容易获得：不需要通过应用商店来下载。

然而，在2017年早期，PWA仍然面临一些限制条件：

*   Safari不支持一些基本的PWA特性，比如 [Service workers](https://developer.mozilla.org/fr/docs/Web/API/Service_Worker_API)，但是苹果公司似乎已经准备开始着手了；

*   一些原生的函数依然没有得到支持：对于更多信息，浏览这个页面[What web can do](https://whatwebcando.today/)。

### 教程目标

本教程的目标是利用VueJS以及Webpack从头创建一个基本的但是完整的渐进式web应用。我们的应用将会满足介绍里面的所有需求：渐进式的，响应式的，连接独立的等等。我想给你一个能够在PWA内完成的目标的总览：流畅的原生式的应用，离线行为，原生特性结构，推送通知。

为了让事情保持挑战性，我们打算构建一个猫图信息app：CropChat！CropChat用户能够阅读主流的猫的图片，并且能够打开他们了解更多细节以及上传新的猫的图片（首先从互联网，接着是从设备驱动或者照相机）。

这个教程将会分为几个部分，它们将会连续地进行发布

*   [Part 1]  Lite基于Vue JS, Webpack 以及Material Design的渐进式web应用

*   [Part 2] 基于Vue-Resource以及VueFire将App和远程的API进行连接

*   [Part 3] 基于Service Worker来实现离线模式

*   [Part 4] 访问设备照相机来拍照

*   [Part 5] 访问设备驱动来上传图片

*   [Part 6] 实现推送通知

*   [Part 7] 访问设备地址

### 我们的PWA的基本组件

我们的渐进式web应用是基于你喜欢的现在组件！

*   [VueJS 2](https://vuejs.org/)视图层： 通过利用[Material Design Lite](https://getmdl.io/)来渲染视图

*   [Vue-Router](https://github.com/vuejs/vue-router)：处理SPA路由

*   [Vue-Resource](https://github.com/pagekit/vue-resource) & [Vuefire](https://github.com/vuejs/vuefire)：  处理和[Firebase](https://firebase.google.com/)数据库的通信

*   [Service Worker](https://developers.google.com/web/fundamentals/getting-started/primers/service-workers)：处理离线模式并且保持数据更新

*   [Webpack](https://webpack.github.io/) & [Vue-loader](https://github.com/vuejs/vue-loader)：构建我们的应用，提供热加载， ES2016 以及 pre-processors.

让我们开始part 1！

### [PART 1] Lite基于VueJS, Webpack 以及 Material Design Lite创建一个单页面应用

如果你不熟悉VueJS 2,我强烈建议你阅读[官方教程](https://vuejs.org/v2/guide/)。

### 构建VueJS APP基础

我们打算利用[Vue-cli](https://github.com/vuejs/vue-cli)来创建我们的应用：

```
npm install -g vue-cli
```

Vue-cli自带一些模板。我们将会选择webpack模板。[Webpack](https://webpack.github.io/)是一个对于Javascript应用的现代模块打包工具，它能够处理并且构建我们的资源。Vue-cli将使用Webpack，vue-loader（热加载！），JS linter以及测试套件来创建一个虚拟的VueJS应用。

```
vue init webpack cropchat
```

你可能会问一些问题。下面是我使用过的配置：

```
This will install Vue 2.x version of the template.
```

```
 `For Vue 1.x use: vue init webpack#1.0 cropchat`

```

```
? Project name cropchat
? Project description Image messenging application
? Author Charles BOCHET <charlesb@theodo.fr>
? Vue build standalone
? Install vue-router? Yes
? Use ESLint to lint your code? Yes
? Pick an ESLint preset Standard
? Setup unit tests with Karma + Mocha? Yes
? Setup e2e tests with Nightwatch? No

```

```
 vue-cli · Generated "cropchat".
```

这个过程会创建一个包含以下子文件夹的项目文件夹：

*   build: 包含webpack以及vue-loader配置文件

*   config: 包含我们的app配置（环境，参数等等）

*   src: 我们应用的源代码

*   static: 图片，css以及其它的公共资源

*   test: Karma & Mocha创建的单元测试文件

然后运行：

```
cd cropchat
npm install
npm run dev

```

这将会在你的浏览器打开`localhost:8080`：

![](http://p0.qhimg.com/t015505db9d169c619f.png)

### 通过一个合适的Manifest让它可以进行安装：

PWA的最大优点之一就是容易安装并且分享。让我们别再等待了！

为了这样做，我们需要添加一个manifest.json文件，并且在index.html文件中进行声明。

`pwa-manifest-webpack-plugin`能够让我们在应用构建的时候生成文件：

`npm i pwa-manifest-webpack-plugin --save`

我们接着能够通过编辑`build/webpack.dev.conf.js` 以及`build/webpack.prod.conf.js`来更新构建过程。

在顶部引入`pwa-manifest-webpack-plugin` ：

```
var path = require('path')
var manifestPlugin = require('pwa-manifest-webpack-plugin')

```

并且将它添加到插件：

```
plugins: [
  new manifestPlugin({
    name: 'CropChat',
    description: 'CropChat - Image Messenger Application',
    display: 'fullscreen',
    icon: {
      src: path.resolve('src/assets/logo.png'),
      sizes: [200]
    }
  }),

```

最后，在 `index.html`中声明使用`manifest.json`：

```
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<link rel="manifest" href="./manifest.json">

```

你可能需要重启你的应用：来这样做，杀掉之间的进程并且再次运行：

```
npm run dev
```

就是它了！让我们在手机设备上安装CropChat。有多种方式可以从不同的手机设备上访问`localhost:8080`。我最喜欢就是使用[ngrok](https://ngrok.com/)。

Ngrok是一种服务，可以远程登录您的本地环境，免费！

安装它：

```
npm install -G ngrok
```

接着，运行：

```
ngrok http 8080
```

那应该会给你以下的输出：

```
ngrok by @inconshreveable                                                                   (Ctrl+C to quit)

Session Status                online                                                                        
Version                       2.1.18                                                                        
Region                        United States (us)                                                            
Web Interface                 http://127.0.0.1:4040                                                         
Forwarding                    http://5ef29506.ngrok.io -> localhost:8080                                    
Forwarding                    https://5ef29506.ngrok.io -> localhost:8080                                   

Connections                   ttl     opn     rt1     rt5     p50     p90                                   
                              39      3       0.01    0.01    120.01  881.89

```

通过你的手机浏览ngrok的url `http://5ef29506.ngrok.io` 。你可以在你的设备桌面添加！

![](http://p0.qhimg.com/t013f3749db09517261.png)

![](http://p0.qhimg.com/t01d4ca0a2d61fc0a70.png)![](http://p0.qhimg.com/t01e9913ace27e7b1ad.png)![](http://p0.qhimg.com/t0134bba83d0d53d338.png)

Cropchat的源代码能够在GitHub [here](https://github.com/charlesBochet/vueJSPwa/tree/master/cropchat)上能够访问。Git历史符合教程的步骤：你可以在下面的commit [5ff77fd3cd71a988fad9c187d57e87ea80d670f0](https://github.com/charlesBochet/vueJSPwa/commit/5ff77fd3cd71a988fad9c187d57e87ea80d670f0)种发现这个步骤的变化

想了解更多关于ngrok，你可以阅读Matthieu Auger的文章：[Expose your local environment to the world with ngrok](http://www.theodo.fr/blog/2016/06/expose-your-local-environment-to-the-world-with-ngrok/)

### 创建视图框架和句柄路由

既然我们已经具有合适的基础，那么我们打算开始构建CropChat的特性。CropChat具有三个视图：

*   Home View: 展示一个猫的图片列表

*   Detail View: 展示特定猫的图片的细节（在Home View种点击访问）

*   Post View: 能够让用户上传一个新的图片

创建一个具有以下框架的`src/component/HomeView.vue` 视图：

```
<template>
  <ul class="list">
  </ul>
</template>

```

```
<script>
export default {
}
</script>

```

```
<style scoped>
  .list {
    width: 100%;
    padding: 0;
  }
</style>

```

对于`src/component/DetailView.vue`视图也是一样的：

```
<template>
  <div class="card-image">
  </div>
</template>

```

```
<script>
export default {
}
</script>

```

```
<style scoped>
</style>

```

对于 `src/component/PostView.vue`也是一样的：

```
<template>
  <div class="waiting">
    Not yet available
  </div>
</template>

```

```
<script>
export default {
}
</script>

```

```
<style scoped>
  .waiting {
    padding: 10px;
    color: #555;
  }
</style>

```

最终，更新路由文件 `src/router/index.js`：

```
import Vue from 'vue'
import Router from 'vue-router'
import HomeView from 'components/HomeView'
import DetailView from 'components/DetailView'
import PostView from 'components/PostView'

```

```
`Vue.use(Router)`

```

```
export default new Router({
  routes: [
    {
      path: '/',
      name: 'home',
      component: HomeView
    },
    {
      path: '/detail/:id',
      name: 'detail',
      component: DetailView
    },
    {
      path: '/post',
      name: 'post',
      component: PostView
    }
  ]
})

```

也移除没有使用的Hello.vue视图。你应该你能够直接看到影响你手机app的变化（热加载很棒，是不是？）

Git commit: [22ab9a2058dae8f7689b8635ff52d89652675aa6](https://github.com/charlesBochet/vueJSPwa/commit/22ab9a2058dae8f7689b8635ff52d89652675aa6)

### 安装 Material Design Lite

不知道Material Design Lite？它是一个轻量级的并且容易在你的web应用上实现[Material Design](https://material.io/guidelines/) 的框架。

你可以在这看到更多的文档：[Get MDL.io](https://getmdl.io/)

更新依赖：

```
`npm install material-design-lite --save`

```

更新`src/App.vue`来导入MDL样式并且加载MDL模块：

```
<script>
require('material-design-lite')
...
</script>

```

```
<style>
  @import url('https://fonts.googleapis.com/icon?family=Material+Icons');
  @import url('https://code.getmdl.io/1.2.1/material.blue-red.min.css');
</style>

```

Git commit: [b726b40488132c400dd861bd397f61b15e81631e](https://github.com/charlesBochet/vueJSPwa/commit/b726b40488132c400dd861bd397f61b15e81631e)

### 为你的单页面应用提供一个导航栏：

更新主要组件`src/App.vue`种的模块部分：

```
<template>
  <div class="mdl-layout mdl-js-layout mdl-layout--fixed-header">
    <header class="mdl-layout__header">
      <div class="mdl-layout__header-row">
        <span class="mdl-layout-title">CropChat</span>
      </div>
    </header>
    <div class="mdl-layout__drawer">
      <span class="mdl-layout-title">CropChat</span>
      <nav class="mdl-navigation">
        <a class="mdl-navigation__link" href="/#/" @click="hideMenu">Home</a>
        <a class="mdl-navigation__link" href="/#/post" @click="hideMenu">Post a picture</a>
      </nav>
    </div>
    <main class="mdl-layout__content">
      <div class="page-content">
        <router-view></router-view>
      </div>
    </main>
  </div>
</template>

```

因为Material Design Lite不是特别为单页面应用构建的，因此在用户点击菜单链接的时候我们需要隐藏burger菜单：

```
<script>
...
export default {
  name: 'app',
  methods: {
    hideMenu: function () {
      document.getElementsByClassName('mdl-layout__drawer')[0].classList.remove('is-visible')
      document.getElementsByClassName('mdl-layout__obfuscator')[0].classList.remove('is-visible')
    }
  }
}
</script>

```

![](http://p0.qhimg.com/t01ad688da96c8d8880.png)

![](http://p0.qhimg.com/t014ca5be7fe44493bf.png)

Git commit: [829d0af767a9f7cba13355296d9da79384d80099](https://github.com/charlesBochet/vueJSPwa/commit/829d0af767a9f7cba13355296d9da79384d80099)

### 传播视图并将你的应用带到生活

我们还没有连接一个后台的服务。我们现在打算暂时使用假数据。

创建一个`src/data.js`文件：

```
export default {
  pictures: [
    {
      'id': 0,
      'url': 'http://25.media.tumblr.com/tumblr_m40h4ksiUa1qbyxr0o1_400.gif',
      'comment': 'A cat game',
      'info': 'Posted by Kevin on Friday'
    },
    {
      'id': 1,
      'url': 'http://25.media.tumblr.com/tumblr_lhd7n9Qec01qgnva2o1_500.jpg',
      'comment': 'Tatoo & cat',
      'info': 'Posted by Charles on Tuesday'
    },
    {
      'id': 2,
      'url': 'http://24.media.tumblr.com/tumblr_m4j2atctRm1qejbiro1_1280.jpg',
      'comment': 'Santa cat',
      'info': 'Posted by Richard on Monday'
    },
    {
      'id': 3,
      'url': 'http://25.media.tumblr.com/tumblr_m3rmbwhVB51qhwmnpo1_1280.jpg',
      'comment': 'Mexico cat',
      'info': 'Posted by Richard on Monday'
    },
    {
      'id': 4,
      'url': 'http://24.media.tumblr.com/tumblr_mceknxs4Lo1qd477zo1_500.jpg',
      'comment': 'Curious cat',
      'info': 'Posted by Richard on Monday'
    }
  ]
}

```

在`HomeView.vue` script部分中导入数据并且将图片链接到对应的细节视图：

```
<script>
import data from '../data'
export default {
  methods: {
    displayDetails (id) {
      this.$router.push({ name: 'detail', params: { id: id }})
 }
  },
  data () {
    return {
      'pictures': data.pictures
    }
  }
}
</script>

```

更新`HomeView.vue` 模板和样式：

```
<template>
  <div>
    <div class="mdl-grid">
      <div class="mdl-cell mdl-cell--3-col mdl-cell mdl-cell--1-col-tablet mdl-cell--hide-phone"></div>
      <div class="mdl-cell mdl-cell--6-col mdl-cell--4-col-phone">
        <div v-for="picture in this.$data.pictures" class="image-card" @click="displayDetails(picture.id)">
          <div class="image-card__picture">
            <img :src="picture.url" />
          </div>
          <div class="image-card__comment mdl-card__actions">
            <span>{{ picture.comment }}</span>
          </div>
        </div>
      </div>
    </div>
    <a class="add-picture-button mdl-button mdl-js-button mdl-button--fab mdl-button--colored" href="/#/post">
      <i class="material-icons">add</i>
    </a>
  </div>
</template>
...
<style scoped>
  .add-picture-button {
    position: fixed;
    right: 24px;
    bottom: 24px;
    z-index: 998;
  }
  .image-card {
    position: relative;
    margin-bottom: 8px;
  }
  .image-card__picture > img {
    width:100%;
  }
  .image-card__comment {
    position: absolute;
    bottom: 0;
    height: 52px;
    padding: 16px;
    text-align: right;
    background: rgba(0, 0, 0, 0.5);
  }
  .image-card__comment > span {
    color: #fff;
    font-size: 14px;
    font-weight: bold;
  }
</style>

```

对`DetailView.vue`进行同样的操作：

```
<template>
  <div class="mdl-grid">
    <div class="mdl-cell mdl-cell--8-col">
      <div class="picture">
        <img :src="this.$data.pictures[$route.params.id].url" />
      </div>
      <div class="info">
        <span>{{ this.$data.pictures[$route.params.id].info }}</span>
      </div>
    </div>
    <div class="mdl-cell mdl-cell--4-col mdl-cell--8-col-tablet">
      <div class="comment">
        <span>{{ this.$data.pictures[$route.params.id].comment }}</span>
      </div>
      <div class="actions">
        <a class="mdl-button mdl-js-button mdl-button--raised mdl-button--colored" href="/#/post">
          ANSWER
        </a>
      </div>
    </div>
  </div>
</template>

```

```
<script>
import data from '../data'
export default {
  data () {
    return {
      'pictures': data.pictures
    }
  }
}
</script>

```

```
<style scoped>
  .picture > img {
    color: #fff;
    width:100%;
  }
  .info {
    text-align: right;
    padding: 5px;
    color: #555;
    font-size: 10px;
  }
  .comment {
    padding: 10px;
    color: #555;
  }
  .actions {
    text-align: center;
  }
</style>

```

Git commit: [39360f251da153c780cd148dc3cf234348bb1e87](https://github.com/charlesBochet/vueJSPwa/commit/39360f251da153c780cd148dc3cf234348bb1e87)

关于'href'链接的使用：我推荐使用vuejs的组件但是在这我想使代码尽可能的简单。

### 最后的结果

我们完成了，CropChat完成啦！

![](http://p0.qhimg.com/t01426ec795f7e96349.png)

![](http://p0.qhimg.com/t01be1c11b6c06f9969.png)

**源代码能够在GitHub repository访问：** [https://github.com/charlesBochet/vueJSPwa](https://github.com/charlesBochet/vueJSPwa)

### Conclusions

我希望我已经令你确信你能够利用VueJS和Webpack就可以简单地创建web应用。总结来说：

*  Vue-cli可以在命令行种创建一个虚拟的VueJS + Webpack应用

*  通过添加Manifest.json文件让你的web应用能够安装

*  使用Vue-Router以及Material Design来创建一个类似于app用户体验的应用

然而，CropChat还依然不是一个渐进式web应用：让我们看一下PWA的需求清单：

一半的需求还没有满足。在下一个部分将会有其它的目标。未完待续！

![](http://p0.qhimg.com/t015f3ce37f416cf068.png)
> 译者注：
> 安利几个我之前翻译过的其它的关于PWA的文章以及我自己的PWA小应用：
*   [Twitter Lite以及大规模的高性能React渐进式网络应用](https://github.com/neal1991/articles-translator/blob/master/Twitter%20Lite%E4%BB%A5%E5%8F%8A%E5%A4%A7%E8%A7%84%E6%A8%A1%E7%9A%84%E9%AB%98%E6%80%A7%E8%83%BDReact%E6%B8%90%E8%BF%9B%E5%BC%8F%E7%BD%91%E7%BB%9C%E5%BA%94%E7%94%A8.md)
*   [Service worker介绍](https://github.com/neal1991/articles-translator/blob/master/Service%20worker%E4%BB%8B%E7%BB%8D.md)
*   [weather app](https://neal1991.github.io/pwa/)
*   [subway-shanghai](https://neal1991.github.io/subway-shanghai/)
    ​              