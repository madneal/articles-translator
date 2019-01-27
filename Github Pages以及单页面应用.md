# Github Pages以及单页面应用

> 原文：[GitHub Pages and Single-Page Apps](https://dev.to/_evansalter/github-pages-and-single-page-apps)
>
> 译者：[neal1991](https://github.com/neal1991)
>
> welcome to star my [articles-translator](https://github.com/neal1991/articles-translator/), providing you advanced articles translation. Any suggestion, please issue or contact [me](mailto:bing@stu.ecnu.edu.cn)
>
> LICENSE: [MIT](https://opensource.org/licenses/MIT)

单页面应用（SPAs）现在特别火。它们使构建功能丰富，高性能的web应用变得特别容易。因此你可以自己去构建一个SPA，但是依然存在一个问题。你应该在什么地方托管呢？

这篇文章分为两个部分。第一部分，是列出了我用过的不同的SPA托管方法以及它们的优缺点。第二部分，是我使用Github Pages托管SPA的经验，包括我遇到的问题。我希望能够帮助读者在如何托管他们的app的时候做出一个知情决定，并且如果它们使用任何我谈论的方法，它们可以从我的错误中有所学习。

如果你希望专注于Github Pages并且跳过其它的方法，你可以直接跳到第四节：Github Pages。

## 1. Google App Engine

我没有深入web开发直到我开始我现在的工作。是的，我了解一些HTML以及CSS，并且我曾经上过大学的web开发课。然而，我对这些并不是很感兴趣直到我在工作领域使用这些技能。

在我的工作中，我们所有的应用都是使用App Engine来构建的。我们使用Python版本，Jinja2是我们的模板语言，Knockout.js是我们的前端代码。之后对于在App Engine中开发我觉得很舒适。我熟悉它，我已经到了可以相对快速，轻松地将新应用程序整合在一起的地步。

现在，回到SPAs。在2016的夏天，我开始从事于  [Roll Up Tracker](https://www.rolluptracker.com/)。Roll Up Tracker是一种Angular2的web-app，它能够让你追踪你在Tim Hortons举办的Rim To Win赛季中的胜利和损失。自然的，当我开始这个项目的时候我决定将它托管在App Engine中。这可能有一点奇怪，利用一个完整的App Engine项目来服务仅仅一些HTML以及JavaScript。然而，我是基于后台是Python，数据库使用的是Google Cloud Database这一事实来做出决定的。因此利用这个托管前端是有意义的。

实现方式非常直接。

1. 为前端文件添加一些规则，以及后端的规则全部添加到你的`app.yaml`中。
2. 创建一个通配符路由到一个处理程序之中，只为`index.html`提供服务。
3. 在你的通配符之上创建你的API路由从而能够首先匹配。

```
# app.yaml
- url: /assets/(.*)
  static_files: dist/assets/\1
  upload: dist/assets/.*

- url: /(.*\.(js|map))$
  static_files: dist/\1
  upload: dist/(.*\.(js|map))

- url: .*
  script: main.app
  secure: always
```

```python
class MainHandler(BaseHandler):

    def get(self, *args, **kwargs):
        context = {}
        self.response.write(template.render(os.path.join(TEMPLATE_DIR, 'index.html'), context))
```

这就是它了！现在开始看第二个

## Google App Engine Flex

哦，App Engine Flex！听起来酷炫吊炸天。它首先，的确是这样的。

我曾经利用Vue.js创建一个叫做[Scrobblin' With Friends](https://scrobblin-friends.appspot.com/)的非常小的SPA，从而能够将Last.fm带回到我的生活中。它是一个简单的app，允许你登录你的Last.fm的用户名，它展示了所有你关注的朋友。它是一个迷你的app，几乎是没有后台的。它所做的所有事情就是拿到一个用户名，大概每隔10秒调用一个API接着展示一些数据。他的确不需要采取一个App Engine项目。

然后我发现了Flex，我们发现它们具有一个Node.js环境。它非常容易创建：

1. 仅有2行的`app.yml`：`runtime: node.js`以及`env: flex`
2. 具有`start`脚本的`package.json`
3. 一个简单的express服务

```javascript
// server.js
var express = require('express');
var app = express();

const PORT = process.env.PORT || 8080;

app.get('/static/*', function(request, response){
     console.log('static file request : ' + request.params[0]);
     response.sendFile( __dirname + '/static/' + request.params[0]);
});

app.get('*', function(request, response) {
    response.sendFile(__dirname + '/index.html');
});

app.listen(PORT);
```

我很快就完成了配置并且完成了部署，并且运行的很好。直到我这个月收到了我的账单。

标准App Engine和Flex环境之间最重要的区别就是在Flex环境中你的项目永远不会消失。在标准环境中，如果没有人访问你的网站，你就不需要支付。即使你占用了一点点流量，它们也很慷慨的对此免费。我没有想到的是，在Flex中就没有免费的午餐了。在我注意到的几天之间，我已经积累了30美元的账单。

我认为如果你愿意花足够预算来进行托管的话，那么Flex将会是一个可行的方法。这并不是我所提及的项目之一。因此返回到我曾经使用的标准的App Engine。

## 3.now

如果你还没有听说过 [now](https://zeit.co/now)，那么**现在**你就应该去看一看。它真的很酷。在你的电脑安装之后，你只需要在你的命令行中输入`now`，接着你的应用就可以在web上访问。

你只需要提供带有`start`脚本的`package.json`文件。`now`将会帮你完成剩余的工作。

就和它所提供的酷的部署过程一样，也有一些我的确不喜欢的方面。

1. 免费的版本只能为你的项目提供一些杂乱的URL，例如coolproject-glkqdjsslm.now.sh。通常如果我没有为托管付费的话我介意使用一些奇怪的URL，但是问题每次你部署的时候，你获得一个新的URL，你基本不可能免费托管一个别人能够使用的应用。
2. 他们的最便宜的付费套餐，为你提供1000次部署每个月（而不是20次），私有的代码库，更多的是14.99美元每个月的。这远远超过了我愿意为那些简单项目提供的预算。

## 4. GitHub Pages

最终，你看完了我的SPA托管经验中的糟糕经历。我们最终来到了本文的重点，就是我在GitHub Pages上托管SPA的经验。

在这上面托管SPA是不是看起来很合理，对不对？你可以获得免费的托管。部署也非常容易，免费的SSL。再加上你的代码可能就要保存在GitHub上！

我最近决定是时候替换我的个人网站了，这是一个我几乎没怎么花精力的免费的Wordpress网站（并且最终我用密码保护它因为它令我很尴尬）。我的新网站[http://evansalter.com](http://evansalter.com/)，是利用Vue.js开发的并且没有后台，因此我能够选择任何一个托管服务。App Engine（包括标准的以及Flex）都对我来说太昂贵了，`now`也是不可行的，理由我已经在上文中阐述过了。因此在这，我们选择了GitHub Pages。

它并不是十全十美的。当然，这可能还有一些问题并且你需要一些方法来避免，但是我没有遇到任何阻拦，并且我确信你也会不会。这么说吧，如果GitHub Pages是一个付费的服务，那么我遇到的一些问题可能让我另投他家。因为它是免费的，所以我愿意花时间来解决这些问题。

我希望能够完整阐述整个运转过程，包括在这个过程中我遇到的各种问题。

### 第一部分：什么是GitHub Pages？

首先，可以看看[这](https://pages.github.com/)。这里面有GitHub Pages所有的基础概念，如果你希望得到更详细的文档你可以点击右上角的“Pages Help"。

有两种Pages。第一种是组织/个人page。它们存在于一个叫做`<YOUR_USERNAME>.github.io`的仓库中，并且具有一个匹配仓库名称的URL。这对于一个个人或者组织来说是一个通用的页面。这个网站的代码存在于这个项目之中。

另外一种，是一个项目page，大多数存在于`gh-pages`分支中，或者是在一个`master`分支中的`/docs`文件夹中。然后你就可以通过`<YOUR_USERNAME>.github.io/<PROJECT_NAME>`来访问页面。它允许你在一个地方保存所有的代码。我认为这很方便。（译者注：其实现在可以直接在master分支中直接托管你的page了，可参见我的另外一篇[文章](http://blog.csdn.net/neal1991/article/details/53535914)）

基于本文的目的，我将主要讨论个人的page。

### 第二部分：我的仓库应该是什么样的？

这是我遇到的第一个问题。一个个人主页是由`master`分支来提供服务的。那意味着我不能直接利用Vue CLI来创建一个项目，提交所有的代码到master，并且期望它能够运行。

在做过一些在线的研究之后，我开始了一个计划。我创建一个`sources`分支来存储源代码文件。接着，在我自己的电脑上，我构建这个项目，将`dist`文件假初始化成一个git仓库，并且让其追踪`master`分支，然后提交以及推送。它运行的很好但是大部分是手工操作的。加上如果我重新克隆这个项目或者使用另外一台电脑的话，我必须再次创建dist仓库。这好麻烦。

### 第三部分：自动部署

我不打算按照以上的流程来实施，因此我写了一个脚本来帮我做这些，接着通过创建Travis CI来运行它。在网上有很多脚本能够完成这样的功能。当我发现Travis事实上具有一个GitHub Pages部署服务的时候我就开始尝试这个并且将其为我的项目工作。在提交一些代码之后，我最终得到一个相当简单的`.travis.yml`来完成所有任务。

```yaml
language: node_js
node_js:
  - "node"
script:
  - yarn run lint
  - yarn run build
deploy:
  provider: pages
  skip_cleanup: true
  github_token: $GITHUB_TOKEN
  local_dir: dist
  target_branch: master
  on:
    branch: sources
```

它基本上会运行我的lint并进行构建，接着如果这个分支是`sources`的话，强制推送`dist`文件夹到`master`。关于此的更多信息，可以参考[Travis CI docs](https://docs.travis-ci.com/user/deployment/pages/)。

### 第四部分：路由

我们非常平滑的来到这一步骤。我在使用webpack dev server能够迅速地进行本地开发，通过hot module reloading。当我希望部署的时候，我只要推送我的变化然后不久之后这些变动就存在了。我再设置Vue路由就不会遇到任何问题。

事实证明，当我是用这个网站的时候我总是从这个网站的根目录开始地。我从来没有直接到`/projects`或者其它地任一page。那时候我遇到一个问题。当我尝试访问我app的一个路由的时候导致了一个404页面。

当我发现这一点的时候我一点都不震惊。这很正常，因为GitHub并不知道Vue app的路由。它所知道的所有就是在`index.html`文件中可以路由的路径。

我尝试使用两个不同的方案来解决这个问题。取决于你的需求，可能某一个更适合你。

**尝试一：Hacking这个404页面**

我发现网上另外一个人和我一样尝试解决这个问题。你可以参考他的[博客](http://www.backalleycoder.com/2016/05/13/sghpa-the-single-page-app-hack-for-github-pages/)。我承认这确实不是一个好的结局方案但是它也是非常容易实现。直到GitHub提出一个更好的解决方案，这是一个相当好的处理方法。

这个解决方案基于你可以使用一个在GH Page上自定义的404页面的事实。所有你需要的做的是在你的仓库中添加一个`404.html`页面。更具上面的链接，我创建了`404.html`并且在`<head>`中添加以下内容：

```html
<script>
  sessionStorage.redirect = location.href;
</script>
<meta http-equiv="refresh" content="0;URL='/'"></meta>
```

基本上这个所做的所有事情是在你的浏览器中保存请求的URL，并且加速你返回网站的根目录。很明显，这并不足够。如果某个人点击了这个博客的链接，我将不希望他们重新返回主页。下一步就很简单了。我就是向`index.html`中添加另外一个脚本来检查保存的URL，如果它存在的话就进行重定向操作。

```javascript
<script>
  (function(){
    var redirect = sessionStorage.redirect;
    delete sessionStorage.redirect;
    if (redirect && redirect != location.href) {
      history.replaceState(null, null, redirect);
    }
  })();
</script>
```

着意外地能够工作的很好。然而，一旦我我决定在我的博文中部署Open Graph以及Twitter meta 标签的时候，它没有成功由于这个scrapers没有遵守这个重定向规则。然而，如果你有一个简单的网站并且不需要meta标签的时候，那么这个可能就适合你。

**尝试二：预渲染一个静态网站**

上面的解决方案在服务meta标签中存在着两个问题：

1. 当Facebook, Slack, Twitter或者其他具有富媒体的网站会扫描我的链接，他们会和期望中的一样给出一个404页面。然而，他们不会根据我上面分享的方法来重新定向到合适的页面。这是因为GitHub Pages对于404页面返回一个404状态码，这会告诉scraper放弃。
2. 即使我不使用这个404页面hack，我的meta标签可以在我的Vue app中生成。因为这个scraper在没有执行任何javascript的时候来下载这个页面，它不会找到这些meta标签。

因此，另一个明显的解决方案是服务端渲染或者预渲染。我不想直接尝试服务端渲染，因为我不响利用一个服务器来托管这个网站。因此，我们采用了预渲染！

因为我使用的是Webpack，我认为通过添加一个webpack插件从我的网站中生成静态的HTML是有意义的。我发现了一些预先构建的插件来完成这个任务，这些都利用PhantomJS来渲染这个网站。因为某些原因，我不能利用它在我的网站中渲染任何内容。我尝试花费大量时间来令它工作，但最后也没有成功。

对此感到沮丧，我决定自己写一个插件。因此，[webpack-static-site-generator](https://github.com/esalter-va/webpack-static-site-generator)就诞生了（好名字，我知道）。通过利用这个插件，当我为生产环境进行构建的时候我能为我所有的页面获得静态的HTML文件。我依然能够在本地开发的时候使用Webpack dev server，并不需要构建，这将会是一个很大的优点。

因此，如果你在托管你的SPA上遇到任何的问题，你可能想要尝试预渲染。

## 第五部分：自定义域名

如果你已经走到这一步，那么你应该已经完成在GH Page上托管的种种细节。创建一个自定义域名非常的简单。有以下几个步骤：

1. 在你的仓库中创建一个`CNAME`文件，里面包含一行你希望使用的域名（比如`www.evansalter.com`）。确保在构建的时候这个文件已经提交到你的master分支。
2. 在你的域名注册上创建一个`CNAME`记录指向你的`github.io`地址（比如`esalter-va.github.io`）

如果你希望使用顶级域名（没有www）的时候，可能会有细微的不同，但应该不会涉及太多的工作。

我发现了一个小问题。就是当你在GH Page上使用自定义域名的时候就没有了SSL支持。我对此感到很烦恼，但是看到我的网站也没有使用HTTPS特别的理由，我也忽略了这一点。

# 最终的想法

当我第一次开始这个网站的时候，我很激动能够利用GitHub Page来托管单页面应用。然而，它并不是看起来的那样神奇。我在这个过程中遇到了很多问题。有一些解决方案比较简单，然而有一些就有一些复杂。然而，我依然乐于解决它并且坚持到了最后。Page的简易性让它很好，我将会建议每个人至少为下一个项目考虑下它。

