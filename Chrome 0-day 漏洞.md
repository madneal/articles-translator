# Operation WizardOpium 攻击使用的 Chrome 零日漏洞 CVE-2019-13720

>原文：[Chrome 0-day exploit CVE-2019-13720 used in Operation WizardOpium](https://securelist.com/chrome-0-day-exploit-cve-2019-13720-used-in-operation-wizardopium/94866/)
>
>译者：[neal1991](https://github.com/neal1991)
>
>welcome to star my [articles-translator](https://github.com/neal1991/articles-translator/), providing you advanced articles translation. Any suggestion, please issue or contact [me](mailto:bing@stu.ecnu.edu.cn)
>
>LICENSE: [MIT](https://opensource.org/licenses/MIT)

## 摘要

卡巴斯基安全防护是卡巴斯基产品的一部分，过去已成功检测到许多零日攻击。最近，为 Google的 Chrome 浏览器发现了一个未知的新漏洞。我们会立即将此情况报告给 Google Chrome 安全团队。在审核了我们提供的 PoC 之后，Google 确认存在零日漏洞并将其分配为 CVE-2019-13720。 Google 已针对 Windows，Mac 和 Linux 发布了 Chrome 版本78.0.3904.87，我们建议所有 Chrome 用户尽快将其更新为最新版本！你可以点击[此处](https://chromereleases.googleblog.com/2019/10/stable-channel-update-for-desktop_31.html)阅读 Google 公告。

卡巴斯基端点产品借助漏洞利用防御组件检测漏洞。该攻击的裁决是 Exploit.Win32.Generic。

我们称这些攻击为 Operation WizardOpium。到目前为止，我们还无法与任何已知的威胁者建立明确的联系。与蓝莲花攻击有某些非常弱的代码相似性，尽管这很可能是 false flag。目标网站的配置与最近部署了类似虚假标记攻击的早期 [DarkHotel](https://securelist.com/the-darkhotel-apt/66779/) 攻击更加一致。

卡巴斯基情报报告的客户可以获取有关 CVE-2019-13720 和最近的 DarkHotel 的 false flag 攻击的详细信息。有关更多信息，请联系：intelreports@kaspersky.com。

## 技术细节

攻击利用朝鲜语新闻门户上的水坑式注入。在主页中插入了恶意的 JavaScript 代码，恶意代码又从远程站点加载了分析脚本。

![KL1Qk8.png](https://s2.ax1x.com/2019/11/02/KL1Qk8.png)

**重定向到漏洞利用登录页面**

主页上托管了一个从 hxxp://code.jquery.cdn.behindcorona[.]com/ 中加载了远程脚本的微不足道的 JavaScript 标签。

然后，该脚本将加载另一个名为 .charlie.XXXXXXXX.js 的脚本。该 JavaScript 通过与浏览器的用户代理进行比较来检查受害者的系统是否能被感染，程序应在 64位 版本的 Windows 上运行，而不是 WOW64 进程；它还尝试获取浏览器的名称和版本。该漏洞试图利用 Google Chrome 浏览器中的 bug，脚本会检查该版本是否大于或等于65（当前的Chrome版本为78）：

![KLMwkD.png](https://s2.ax1x.com/2019/11/02/KLMwkD.png)

**分析脚本（.charlie.XXXXXXXX.js）中 Chrome 版本检测**

如果检测出浏览器版本，脚本将开始向攻击者的受控服务器 (behindcorona[.]com) 发送一些 AJAX 请求，其中路径名指向传递给脚本（xxxxxxx.php）的参数。首先需要获得一些将来有用的重要信息。该信息包括几个十六进制编码的字符串，这些字符串告诉脚本应从服务器下载多少个实际漏洞利用代码，以及图像文件的 URL，这个图片嵌入了最终载荷的密钥和RC4密钥从而对漏洞利用代码解密。

![KLM0te.png](https://s2.ax1x.com/2019/11/02/KLM0te.png)

**漏洞利用链– AJAX 请求 xxxxxxx.php**

下载完所有代码块后，RC4 脚本将所有部分解密并拼接在一起，这为攻击者提供了一个包含完整浏览器漏洞的新 JavaScript 代码。为了解密这些部分，使用了之前的 RC4 密钥。

![KLMgnP.png](https://s2.ax1x.com/2019/11/02/KLMgnP.png)

**另一次版本检测**

浏览器漏洞脚本被混淆；消除混淆后我们观察到一些奇怪的事情：

1. 对用户代理的字符串进行另一项检查–这次它检查浏览器版本是 76 还是77。这可能意味着漏洞利用作者仅使用这些版本（先前的漏洞利用阶段检查的版本号为65或更高）或过去使用曾在旧版 Chrome 中使用过其他漏洞利用。

![KLM4hQ.png](https://s2.ax1x.com/2019/11/02/KLM4hQ.png)

**混淆后的漏洞利用代码**

2. 操作浏览器的内置 BigInt 类，这个类在 JavaScript 代码中执行 64 位算术很有用，例如，在 64位 环境中使用原生指针。通常情况下，漏洞利用开发者通过与32位数字实现自己的功能。但是，在这种情况下，使用的是BigInt，它应该更快，因为它是在浏览器的代码中本地实现的。漏洞利用开发者此处并未使用全部 64 位，而是使用较小的数字范围。这就是为什么它们实现一些功能以与数字的较高/较低部分兼容原因。

![KLMqBV.png](https://s2.ax1x.com/2019/11/02/KLMqBV.png)

**使用 64 位数字的代码片段**

3. 在实际的代码中有许多未使用的函数和变量。这通常意味着它们用于调试代码，然后在将代码移至生产环境时被遗忘。

4. 大多数代码使用与浏览器的某些易受攻击组件相关的几个类。由于此 bug 仍未得到修复，因此我们此处不包括有关特定易受攻击组件的详细信息。

5. 有一些带有数字的大数组，这些数字代表一个 shellcode块 和一个嵌入式 PE 镜像。

由于存在漏洞披露原则，我们在此提供的分析特意简短。该漏洞利用了两个线程之间的竞争条件错误，原因是它们之间缺少适当的同步。它使攻击者处于释放后使用（UaF）的状态，这是非常危险的，因为它可能导致代码执行，这正是本例所发生的情况。

该漏洞利用程序首先尝试触发 UaF 对重要的64位地址（作为指针）执行信息泄漏。结果是：1）如果地址成功泄漏，则表明漏洞利用正常。2）泄漏的地址用于定位堆/栈的位置，这使地址空间布局随机化（ASLR）技术无效； 3）通过在该地址附近进行搜索，可以找到其他一些有用的指针，以供进一步利用。

之后，它尝试使用递归函数创建一堆大对象。这样做是为了确定一些重要的的堆布局，这对于成功利用漏洞很重要。同时，它尝试利用堆喷涂技术，该技术旨在重用先前在 UaF 部分释放的指针。尽管实际上它们位于相同的内存区域，但该技巧可能会引起混乱，并使攻击者能够对两个不同的对象进行操作（从 JavaScript 代码的角度来看）。

该漏洞尝试执行许多操作来分配/释放内存以及其他技术，这些技术最终为攻击者提供了任意的读/写能力。这用于制作可以与 WebAssembly 和 FileReader 一起使用的特殊对象来执行嵌入的 Shellcode 有效载荷。

![KLML7T.png](https://s2.ax1x.com/2019/11/02/KLML7T.png)

**第一阶段 shellcode**

## 有效载荷说明

最终的有效载荷将作为加密的二进制文件（worst.jpg）下载，并由shellcode解密。

![KLQPnx.png](https://s2.ax1x.com/2019/11/02/KLQPnx.png)

**加密的有效载荷– Worst.jpg**

解密后，恶意软件模块将作为 updata.exe 拖放到磁盘上并执行。 为了持久化，该恶意软件会在 Windows Task Scheduler 中安装任务。

有效载荷“安装程序”是 RAR SFX 归档文件，其中包含以下信息：

File size: 293,403
MD5: 8f3cd9299b2f241daf1f5057ba0b9054
SHA256: 35373d07c2e408838812ff210aa28d90e97e38f2d0132a86085b0d54256cc1cd

这个文档包含两个文件：
![KLQe9H.png](https://s2.ax1x.com/2019/11/02/KLQe9H.png)

文件名: iohelper.exe

MD5: 27e941683d09a7405a9e806cc7d156c9
SHA256: 8fb2558765cf648305493e1dfea7a2b26f4fc8f44ff72c95e9165a904a9a6a48

文件名: msdisp64.exe

MD5: f614909fbd57ece81d00b01958338ec2
SHA256: cafe8f704095b1f5e0a885f75b1b41a7395a1c62fd893ef44348f9702b3a0deb

这两个文件是同时编译的，我们确信的依据是时间戳 "Tue Oct 8 01:49:31 2019”。

主模块（msdisp64.exe）尝试从硬编码的 C2 服务器集中下载下一部分。下一部分位于 C2 服务器上具有受害计算机名称的文件夹中，因此威胁执行者可以了解有关哪些计算机被感染的信息，并将下一阶段模块放置在 C2 服务器上的特定文件夹中。

卡巴斯基情报报告的客户可以获取有关此攻击的更多详细信息。更多信息，请联系：intelreports@kaspersky.com。

## IoCs

* behindcorona[.]com
* code.jquery.cdn.behindcorona[.]com
* 8f3cd9299b2f241daf1f5057ba0b9054
* 35373d07c2e408838812ff210aa28d90e97e38f2d0132a86085b0d54256cc1cd
* 27e941683d09a7405a9e806cc7d156c9
* 8fb2558765cf648305493e1dfea7a2b26f4fc8f44ff72c95e9165a904a9a6a48
* f614909fbd57ece81d00b01958338ec2
* cafe8f704095b1f5e0a885f75b1b41a7395a1c62fd893ef44348f9702b3a0deb
* kennethosborne@protonmail.com


 

 