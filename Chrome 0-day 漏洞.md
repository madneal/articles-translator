https://securelist.com/chrome-0-day-exploit-cve-2019-13720-used-in-operation-wizardopium/94866/

# Operation WizardOpium 攻击使用的 Chrome 零日漏洞 CVE-2019-13720

## 摘要

Kaspersky Exploit Prevention is a component part of Kaspersky products that has successfully detected a number of zero-day attacks in the past. Recently, it caught a new unknown exploit for Google’s Chrome browser. We promptly reported this to the Google Chrome security team. After reviewing of the PoC we provided, Google confirmed there was a zero-day vulnerability and assigned it CVE-2019-13720. Google has released Chrome version 78.0.3904.87 for Windows, Mac, and Linux and we recommend all Chrome users to update to this latest version as soon as possible! You can read Google’s bulletin by clicking here.

Kaspersky endpoint products detect the exploit with the help of the exploit prevention component. The verdict for this attack is Exploit.Win32.Generic.

We are calling these attacks Operation WizardOpium. So far, we have been unable to establish a definitive link with any known threat actors. There are certain very weak code similarities with Lazarus attacks, although these could very well be a false flag. The profile of the targeted website is more in line with earlier DarkHotel attacks that have recently deployed similar false flag attacks.

卡巴斯基安全防护是卡巴斯基产品的一部分，过去已成功检测到许多零日攻击。最近，为 Google的 Chrome 浏览器发现了一个未知的新漏洞。我们会立即将此情况报告给 Google Chrome 安全团队。在审核了我们提供的 PoC 之后，Google 确认存在零日漏洞并将其分配为 CVE-2019-13720。 Google 已针对 Windows，Mac 和 Linux 发布了 Chrome 版本78.0.3904.87，我们建议所有 Chrome 用户尽快将其更新为最新版本！你可以点击[此处](https://chromereleases.googleblog.com/2019/10/stable-channel-update-for-desktop_31.html)阅读 Google 公告。

卡巴斯基端点产品借助漏洞利用防御组件检测漏洞。该攻击的裁决是 Exploit.Win32.Generic。

我们称这些攻击为 Operation WizardOpium。到目前为止，我们还无法与任何已知的威胁者建立明确的联系。与蓝莲花攻击有某些非常弱的代码相似性，尽管这很可能是 false flag。目标网站的配置与最近部署了类似虚假标记攻击的早期 DarkHotel 攻击更加一致。

![KLMJ61.png](https://s2.ax1x.com/2019/11/02/KLMJ61.png)

More details about CVE-2019-13720 and recent DarkHotel false flag attacks are available to customers of Kaspersky Intelligence Reporting. For more information, contact: intelreports@kaspersky.com.

卡巴斯基情报报告的客户可以获取有关 CVE-2019-13720 和最近的 DarkHotel 的 false flag 攻击的详细信息。有关更多信息，请联系：intelreports@kaspersky.com。

## 技术细节

The attack leverages a waterhole-style injection on a Korean-language news portal. A malicious JavaScript code was inserted in the main page, which in turn, loads a profiling script from a remote site.

攻击利用朝鲜语新闻门户上的水坑式注入。在主页中插入了恶意的 JavaScript 代码，恶意代码又从远程站点加载了分析脚本。

![KL1Qk8.png](https://s2.ax1x.com/2019/11/02/KL1Qk8.png)

Redirect to the exploit landing page

The main index page hosted a small JavaScript tag that loaded a remote script from hxxp://code.jquery.cdn.behindcorona[.]com/.

The script then loads another script named .charlie.XXXXXXXX.js. This JavaScript checks if the victim’s system can be infected by performing a comparison with the browser’s user agent, which should run on a 64-bit version of Windows and not be a WOW64 process; it also tries to get the browser’s name and version. The vulnerability tries to exploit the bug in Google Chrome browser and the script checks if the version is greater or equal to 65 (current Chrome version is 78):

重定向到漏洞利用登录页面

主页上托管了一个从 hxxp://code.jquery.cdn.behindcorona[.]com/ 中加载了远程脚本的微不足道的 JavaScript 标签。

然后，该脚本将加载另一个名为 .charlie.XXXXXXXX.js 的脚本。该 JavaScript 通过与浏览器的用户代理进行比较来检查受害者的系统是否能被感染，程序应在 64位 版本的 Windows 上运行，而不是 WOW64 进程；它还尝试获取浏览器的名称和版本。该漏洞试图利用 Google Chrome 浏览器中的 bug，脚本会检查该版本是否大于或等于65（当前的Chrome版本为78）：

![KLMwkD.png](https://s2.ax1x.com/2019/11/02/KLMwkD.png)

Chrome version checks in the profiling script (.charlie.XXXXXXXX.js)

If the browser version checks out, the script starts performing a number of AJAX requests to the attacker’s controlled server (behindcorona[.]com) where a path name points to the argument that is passed to the script (xxxxxxx.php). The first request is necessary to obtain some important information for further use. This information includes several hex-encoded strings that tell the script how many chunks of the actual exploit code should be downloaded from the server, as well as a URL to the image file that embeds a key for the final payload and RC4 key to decrypt these chunks of the exploit’s code.

分析脚本（.charlie.XXXXXXXX.js）中 Chrome 版本检测

如果检测出浏览器版本，脚本将开始向攻击者的受控服务器 (behindcorona[.]com) 发送一些 AJAX 请求，其中路径名指向传递给脚本（xxxxxxx.php）的参数。首先需要获得一些将来有用的重要信息。该信息包括几个十六进制编码的字符串，这些字符串告诉脚本应从服务器下载多少个实际漏洞利用代码，以及图像文件的 URL，这个图片嵌入了最终载荷的密钥和RC4密钥从而对漏洞利用代码解密。

![KLM0te.png](https://s2.ax1x.com/2019/11/02/KLM0te.png)

Exploitation chain – AJAX requests to xxxxxxx.php

After downloading all the chunks, the RC4 script decrypts and concatenates all the parts together, which gives the attacker a new JavaScript code containing the full browser exploit. To decrypt the parts, the previously retrieved RC4 key is used.

利用链– AJAX 请求 xxxxxxx.php

下载完所有代码块后，RC4 脚本将所有部分解密并拼接在一起，这为攻击者提供了一个包含完整浏览器漏洞的新 JavaScript 代码。为了解密这些部分，使用了之前的 RC4 密钥。

![KLMgnP.png](https://s2.ax1x.com/2019/11/02/KLMgnP.png)

One more version check

The browser exploit script is obfuscated; after de-obfuscation we observed a few peculiar things:

Another check is made against the user agent’s string – this time it checks that the browser version is 76 or 77. It could mean that the exploit authors have only worked on these versions (a previous exploitation stage checked for version number 65 or newer) or that other exploits have been used in the past for older Chrome versions.

另一次版本检测

浏览器漏洞脚本被混淆；消除混淆后我们观察到一些奇怪的事情：

对用户代理的字符串进行另一项检查–这次它检查浏览器版本是 76 还是77。这可能意味着漏洞利用作者仅使用这些版本（先前的漏洞利用阶段检查的版本号为65或更高）或过去曾在旧版 Chrome 中使用过其他漏洞利用。

![KLM4hQ.png](https://s2.ax1x.com/2019/11/02/KLM4hQ.png)
Obfuscated exploit code

There are a few functions that operate on the browser’s built-in BigInt class, which is useful for doing 64-bit arithmetic inside JavaScript code, for example, to work with native pointers in a 64-bit environment. Usually, exploit developers implements their own functions for doing this by working with 32-bit numbers. However, in this case, BigInt is used, which should be faster because it’s implemented natively in the browser’s code. The exploit developers don’t use all 64 bits here, but instead operate on a smaller range of numbers. This is why they implement a few functions to work with higher/lower parts of the number.

混淆后的漏洞利用代码

操作浏览器的内置 BigInt 类，这个类在 JavaScript 代码中执行 64 位算术很有用，例如，在 64位 环境中使用原生指针。通常情况下，漏洞利用开发者通过与32位数字实现自己的功能。但是，在这种情况下，使用的是BigInt，它应该更快，因为它是在浏览器的代码中本地实现的。漏洞利用开发者此处并未使用全部 64 位，而是使用较小的数字范围。这就是为什么它们实现一些功能以与数字的较高/较低部分兼容原因。

![KLMqBV.png](https://s2.ax1x.com/2019/11/02/KLMqBV.png)
Snippet of code to work with 64-bit numbers

There are many functions and variables that are not used in the actual code. This usually means that they were used for debugging code and were then left behind when the code was moved to production.
The majority of the code uses several classes related to a certain vulnerable component of the browser. As this bug has still not been fixed, we are not including details about the specific vulnerable component here.
There are a few big arrays with numbers that represent a shellcode block and an embedded PE image.
The analysis we have provided here is deliberately brief due to vulnerability disclosure principles. The exploit used a race condition bug between two threads due to missing proper synchronization between them. It gives an attacker an a Use-After-Free (UaF) condition that is very dangerous because it can lead to code execution scenarios, which is exactly what happens in our case.

The exploit first tries to trigger UaF to perform an information leak about important 64-bit addresses (as a pointer). This results in a few things: 1) if an address is leaked successfully, it means the exploit is working correctly; 2) a leaked address is used to know where the heap/stack is located and that defeats the address space layout randomization (ASLR) technique; 3) a few other useful pointers for further exploitation could be located by searching near this address.

After that it tries to create a bunch of large objects using a recursive function. This is done to make some deterministic heap layout, which is important for a successful exploitation. At the same time, it attempts to utilize a heap spraying technique that aims to reuse the same pointer that was freed earlier in the UaF part. This trick could be used to cause confusion and give the attacker the ability to operate on two different objects (from a JavaScript code perspective), though in reality they are located in the same memory region.

The exploit attempts to perform numerous operations to allocate/free memory along with other techniques that eventually give the attackers an arbitrary read/write primitive. This is used to craft a special object that can be used with WebAssembly and FileReader together to perform code execution for the embedded shellcode payload.

使用 64 位数字的代码片段

在实际的代码中有许多未使用的函数和变量。这通常意味着它们用于调试代码，然后在将代码移至生产环境时被遗忘。

大多数代码使用与浏览器的某些易受攻击组件相关的几个类。由于此 bug 仍未得到修复，因此我们此处不包括有关特定易受攻击组件的详细信息。

有一些带有数字的大数组，这些数字代表一个 shellcode块 和一个嵌入式 PE 镜像。

由于存在漏洞披露原则，我们在此提供的分析特意简短。该漏洞利用了两个线程之间的竞争条件错误，原因是它们之间缺少适当的同步。它使攻击者处于释放后使用（UaF）的状态，这是非常危险的，因为它可能导致代码执行，这正是本例所发生的情况。

该漏洞利用程序首先尝试触发 UaF 对重要的64位地址（作为指针）执行信息泄漏。结果是：1）如果地址成功泄漏，则表明漏洞利用正常。2）泄漏的地址用于定位堆/栈的位置，这使地址空间布局随机化（ASLR）技术无效； 3）通过在该地址附近进行搜索，可以找到其他一些有用的指针，以供进一步利用。

之后，它尝试使用递归函数创建一堆大对象。这样做是为了确定一些重要的的堆布局，这对于成功利用漏洞很重要。同时，它尝试利用堆喷涂技术，该技术旨在重用先前在 UaF 部分释放的指针。尽管实际上它们位于相同的内存区域，但该技巧可能会引起混乱，并使攻击者能够对两个不同的对象进行操作（从 JavaScript 代码的角度来看）。

该漏洞尝试执行许多操作来分配/释放内存以及其他技术，这些技术最终为攻击者提供了任意的读/写能力。这用于制作可以与 WebAssembly 和 FileReader 一起使用的特殊对象来执行嵌入的 Shellcode 有效载荷。

![KLML7T.png](https://s2.ax1x.com/2019/11/02/KLML7T.png)
First stage shellcode

Payload description
The final payload is downloaded as an encrypted binary (worst.jpg) that is decrypted by the shellcode.

![KLQPnx.png](https://s2.ax1x.com/2019/11/02/KLQPnx.png)
Encrypted payload – worst.jpg

After decryption, the malware module is dropped as updata.exe to disk and executed. For persistence the malware installs tasks in Windows Task Scheduler.

The payload ‘installer’ is a RAR SFX archive, with the following information:

第一阶段 shellcode

有效载荷说明

最终的有效载荷将作为加密的二进制文件（worst.jpg）下载，并由shellcode解密。

！[KLQPnx.png]（https://s2.ax1x.com/2019/11/02/KLQPnx.png）

加密的有效载荷– Worst.jpg

解密后，恶意软件模块将作为 updata.exe 拖放到磁盘上并执行。 为了持久化，该恶意软件会在 Windows Task Scheduler 中安装任务。

有效载荷“安装程序”是 RAR SFX 归档文件，其中包含以下信息：

File size: 293,403
MD5: 8f3cd9299b2f241daf1f5057ba0b9054
SHA256: 35373d07c2e408838812ff210aa28d90e97e38f2d0132a86085b0d54256cc1cd

The archive contains two files:
这个文档包含两个文件：
![KLQe9H.png](https://s2.ax1x.com/2019/11/02/KLQe9H.png)

File name: iohelper.exe
MD5: 27e941683d09a7405a9e806cc7d156c9
SHA256: 8fb2558765cf648305493e1dfea7a2b26f4fc8f44ff72c95e9165a904a9a6a48

File name: msdisp64.exe
MD5: f614909fbd57ece81d00b01958338ec2
SHA256: cafe8f704095b1f5e0a885f75b1b41a7395a1c62fd893ef44348f9702b3a0deb

Both files were compiled at the same time, which if we are to believe the timestamp, was “Tue Oct 8 01:49:31 2019”.
The main module (msdisp64.exe) tries to download the next stage from a hardcoded C2 server set. The next stages are located on the C2 server in folders with the victim computer names, so the threat actors have information about which machines were infected and place the next stage modules in specific folders on the C2 server.

More details about this attack are available to customers of Kaspersky Intelligence Reporting. For more information, contact: intelreports@kaspersky.com.

这两个文件是同时编译的，我们确信的依据是时间戳 "Tue Oct 8 01:49:31 2019”。

主模块（msdisp64.exe）尝试从硬编码的 C2 服务器集中下载下一部分。下一部分位于 C2 服务器上具有受害计算机名称的文件夹中，因此威胁执行者可以了解有关哪些计算机被感染的信息，并将下一阶段模块放置在 C2 服务器上的特定文件夹中。

卡巴斯基情报报告的客户可以获取有关此攻击的更多详细信息。 有关更多信息，请联系：intelreports@kaspersky.com。

IoCs
behindcorona[.]com
code.jquery.cdn.behindcorona[.]com
8f3cd9299b2f241daf1f5057ba0b9054
35373d07c2e408838812ff210aa28d90e97e38f2d0132a86085b0d54256cc1cd
27e941683d09a7405a9e806cc7d156c9
8fb2558765cf648305493e1dfea7a2b26f4fc8f44ff72c95e9165a904a9a6a48
f614909fbd57ece81d00b01958338ec2
cafe8f704095b1f5e0a885f75b1b41a7395a1c62fd893ef44348f9702b3a0deb
kennethosborne@protonmail.com
GOOGLE CHROME JAVASCRIPT PROOF-OF-CONCEPT TARGETED ATTACKS VULNERABILITIES AND EXPLOITS WATERING HOLE ATTACKS WEBSITE HACKS ZERO-DAY VULNERABILITIES
Share post on:

 

 