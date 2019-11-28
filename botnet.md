# 僵尸网络 Stantinko 犯罪活动新增加密货币挖矿

>原文：[Stantinko botnet adds cryptomining to its pool of criminal activities](https://www.welivesecurity.com/2019/11/26/stantinko-botnet-adds-cryptomining-criminal-activities/)
>
>译者：[neal1991](https://github.com/neal1991)
>
>welcome to star my [articles-translator](https://github.com/neal1991/articles-translator/), providing you advanced articles translation. Any suggestion, please issue or contact [me](mailto:bing@stu.ecnu.edu.cn)
>
>LICENSE: [MIT](https://opensource.org/licenses/MIT)

ESET 研究人员发现，Stantinko 僵尸网络背后的犯罪分子正在向他们控制的肉鸡分发加密货币挖矿模块。

[Stantinko 僵尸网络](https://www.welivesecurity.com/2017/07/20/stantinko-massive-adware-campaign-operating-covertly-since-2012/) 的操纵者已经通过一种新方法扩展了其工具集从受其控制的肉鸡中获利。多达 50 万的僵尸网络自 2012 年以来一直保持活跃，主要针对俄罗斯，乌克兰，白俄罗斯和哈萨克斯坦的用户，现在分发了一个加密货币挖矿模块。门罗币是一种加密货币，其汇率在 2019 年在 50 美元至 110 美元之间波动，自 2018 年 8 月以来，它一直是僵尸网络的获利手段。在此之前，僵尸网络进行了点击欺诈，广告注入，社交网络欺诈和密码窃取攻击。

在本文中，我们将介绍 Stantinko 的加密货币挖矿模块并对其功能进行分析。

该模块最显着的功能是它的混淆方式阻碍了分析并避免了检测。由于源代码级混淆以及随机性使用，而且 Stantinko 的操纵者会为每个新的受害者编译此模块，因此该模块的每个样本都是唯一的。

我们将在另一篇文章中为恶意软件分析人员介绍该模块的混淆技术，并提供一种处理其中某些问题的可行方法。

由于 Stantinko 一直在不断开发新的产品并改进其现有的自定义混淆器和模块，这些混淆器和模块被严重混淆，因此跟踪每个小的改进和修改非常困难。因此，我们决定仅提及和描述与早期样本相比比较重要的调整。最终，在本文中我们打算仅描述模块当前的状态。

## 修改后的开源加密货币挖矿软件

Stantinko 的加密货币挖矿模块通过挖掘加密货币来耗尽受感染机器的大部分资源，它是 [xmr-stak](https://github.com/fireice-uk/xmr-stak) 的大幅修改后的开源加密货币挖矿版本。为了逃避检测，删除了所有不必要的字符串甚至整个函数。其余的字符串和函数被严重混淆。ESET 安全产品将此恶意软件检测为 Win{32,64}/CoinMiner.Stantinko.。

## 挖矿代理的使用

CoinMiner.Stantinko 不会直接与其[矿池](https://en.wikipedia.org/wiki/Mining_pool)进行通信，而是通过 IP 地址为从 YouTube 视频的描述中获取的代理进行通信。使用了与银行恶意软件 [Casbaneiro](https://www.welivesecurity.com/2019/10/03/casbaneiro-trojan-dangerous-cooking/) 类似的技术在 YouTube 视频描述中隐藏数据。Casbaneiro 使用看起来更为合法的频道和描述，但目的大致相同：存储加密的 C&C。

此类视频的描述由以十六进制格式的挖矿的代理 IP 地址字符串组成。例如，图1中显示的 YouTube 视频的描述为 "03101f1712dec626"，它对应于两个十六进制格式的 IP 地址- 03101f17 对应于十进制点分四进制格式的 3.16.31[.]23，而 12dec626 对应 18.222.198[.]38。截至本文，格式已稍作调整。 IP地址当前用 “!!!!” 括起来，简化了解析过程，并防止了 YouTube 视频 HTML 结构的更改导致解析器无法正常工作。

![QS2kVI.png](https://s2.ax1x.com/2019/11/26/QS2kVI.png)
图1.示例 YouTube 视频，其描述为模块提供了与矿池通信的 IP 地址

在早期版本中，YouTube URL 在 CoinMiner.Stantinko 二进制文件中是硬编码编写的。当前，模块改为接收视频标识符作为命令行参数。然后，该参数用于以 https://www.youtube.com/watch?v=%PARAM% 的形式构造 YouTube URL。加密货币挖矿模块由 Stantinko 的 [BEDS](https://www.welivesecurity.com/wp-content/uploads/2017/07/Stantinko.pdf) 组件执行，或者由 rundll32.exe 通过我们未捕获到的的批处理文件执行，模块是从格式为 ％TEMP％\％RANDOM％\％RANDOM_GUID％.dll 本地文件系统位置加载。

我们已将这种滥用告知 YouTube；包含这些视频的所有频道均已关闭。

## 加密货币挖矿软件能力

我们将密码挖掘模块分为四个逻辑部分，分别代表不同的功能集。主要部分执行实际的加密货币挖矿；模块的其他部分负责附加功能：

* 暂停其他（即竞争性）加挖矿应用
* 检测安全软件
* 如果 PC 依靠电池供电或检测到任务管理器，则暂停加密采矿功能，以防止被用户发现

### 加密货币挖矿软件

加密货币挖矿的核心取决于哈希处理以及代理通信。上面描述了获取代理列表的方法；CoinMiner.Stantinko 与它发现的第一个存活的挖矿代理建立通信。

它的通信通过 TCP 进行，并由密钥为数字 pi 的前 26 个字符（包括小数点分隔符，硬编码为字符串"3,141592653589793238462643"）组成的 RC4 算法加密，然后由 base64 编码；我们看到的所有样本都使用相同的密钥。

在通信开始时，从挖矿代理下载哈希算法的代码，并将其加载到内存中或在较早版本中先从库 libcr64.dll 拖到磁盘中。

每次执行时下载哈希代码，可使 Stantinko 组在运行中更改代码。例如，此更改使得有可能适应现有货币中算法的调整，并切换到挖掘其他加密货币，以便在执行时挖掘利润最丰厚的加密货币。从远程服务器下载模块的核心部分并将其直接加载到内存中的主要好处是，这部分代码永远不会存储在磁盘上。此附加调整（较早版本中没有提供）让检测复杂化，因为这些算法中的模式对于检测安全产品而言是太微小。

我们已经分析了 Stantinko 加密货币挖矿模块的所有实例。我们从挖掘代理和哈希算法提供的作业中得出以上结论。例如，图2是由代理之一发送的作业。

```
{“error”:null,”result”:{“status”:”OK”}}
{“method”:”job”,”params”:”blob”:”0b0bbfdee1e50567042dcfdfe96018227f25672544521f8ee2564cf8b4c3139a6a88c5f0b32664000000a1c8ee5c185ed2661daab9d0c454fd40e9f53f0267fe391bdb4eb4690395deb36018″,”job_id”:”281980000000000a10″,”target”:”67d81500″,”height”:1815711}}
```
图2.从矿池代理接​​收的挖矿作业示例

我们分析了使用的哈希算法，发现它是 [CryptoNight R](https://github.com/SChernykh/CryptonightR)。由于有多种使用该算法的加密货币，仅凭这个算法还不足以识别；它只会缩短列表。在提供的作业中，可以看到当时 [blockchain 的高度](https://coinguides.org/block-height/) 为1815711，因此我们不得不使用 CryptoNight R 在专用[区块浏览器](https://marketbusinessnews.com/financial-glossary/block-explorer/)中查找汇率，我们推导为门罗币。解剖字符串0b0bbfdee1e50567042dcfdfe96018227f25672544521f8ee2564cf8b4c3139a6a88c5f0b32664000000a1c8ee5c185ed2661daab9d0c454fd40e9f53f0267fe391bdb4eb4690395deb36018 显示之前区块的哈希（67042dcfdfe96018227f25672544521f8ee2564cf8b4c3139a6a88c5f0b32664）和时间戳（1555590859）确实在 区块高度为 1815711 时和[门罗币区块链匹配](https://xmrchain.net/search?value=1815711)。通过在门罗币的源代码中检查[生成器函数](https://github.com/monero-project/monero/blob/a48ef0a65afd2d89b9a81479f587b5b516a31c9c/src/cryptonote_basic/cryptonote_format_utils.cpp#L1207)查找 blob 的结构。生成器函数公开了另一个称为 [block header](https://github.com/monero-project/monero/blob/a48ef0a65afd2d89b9a81479f587b5b516a31c9c/src/cryptonote_basic/cryptonote_basic.h#L446) 的结构，该结构同时包含前一个块的哈希和时间戳。

与 CoinMiner.Stantinko 的其余部分不同，哈希算法不会被混淆，因为混淆会显着影响哈希计算的速度，从而影响整体性能和盈利能力。但是，作者仍要确保不要留下任何有意义的字符串或组件。

### 中止其他加密货币挖矿软件

该恶意软件枚举运行进程来搜索其他加密货币挖矿软件。如果发现任何竞争对手，Stantinko 将中止其所有线程。

如果 CoinMiner.Stantinko 在进程命令行中发现包含一个特定的字符串或组合（因样本而异），则认为该过程为加密货币挖矿软件。 例如：

* minerd
* minergate
* xmr
* cpservice
* vidservice and stratum+tcp://
* stratum://
* -u and pool
* “-u and pool
* “-u and xmr
* -u and xmr
* -u and mining
* “-u and mining
* -encodedcommand and exe
* –donate-level
* windows and -c and cfgi
* regsvr32 and /n and /s and /q
* application data and exe
* appdata and exe

这些字符串引用以下合法的加密货币挖矿软件：https://github.com/pooler/cpuminer, https://minergate.com/, https://github.com/xmrig 甚至 https://github.com/fireice-uk/xmr-stak。有趣的是，这些正是 Stantinko 模块正是基于这些软件的。这些字符串还导致包含加密采矿功能的各种恶意软件样本。

有趣的是，Stantinko 操纵者[已经被人知道](https://www.welivesecurity.com/2017/07/20/stantinko-massive-adware-campaign-operating-covertly-since-2012/)试图消灭竞争代码。但是，他们依靠合法的反病毒工具套件提供的内置脚本语言编写的脚本来完成此任务。

### 检测预防

CoinMiner.Stantinko 如果检测到机器未连接任何电源，则将暂时中止挖矿。这项措施显然是针对笔记本电脑的，它可以防止电池快速耗尽……这可能会引起用户的怀疑。

此外，如果检测到任务管理器应用程序（procexp64.exe，procexp.exe 或 taskmgr.exe 的进程）正在运行，它会暂时中止挖矿。

该恶意软件还会扫描运行进程以查找安全软件，然后再次查找任务管理器。它计算出进程名称的 CRC-32，然后根据附录中硬编码的 CRC-32 检验和列表进行检查。通常，此技术可帮助逃避检测，因为这些安全产品的进程名称未包含在二进制文件中–通过不直接包含进程名称，增加了隐秘性。这也使分析者更难发现恶意软件作者目的所在，因为必须破解这些散列，从技术上讲，这与[密码破解](https://en.wikipedia.org/wiki/Password_cracking)是相同的问题。但是，使用已知进程名称的列表通常足以确定确切的名称。

如果找到 CRC-32 匹配项，则将 CRC 写入日志文件（api-ms-win-crt-io-l1-1-0.dll）。该日志文件可能稍后会是我们未发现的 Stantinko 组件释放出的，因为此模块中没有与其相关的其他功能。

## 混淆

除了加密货币挖矿功能外，CoinMiner.Stantinko 还以其避免检测和阻碍分析的混淆技术而著称。其中一些技术是独特的，我们将在后续文章中对其进行详细描述。

## 总结

我们的发现表明，Stantinko 背后的犯罪分子继续扩大利用其控制的僵尸网络的方式。他们之前的创新是在 Joomla 和 WordPress 网站上进行基于字典的分布式攻击来获取服务器凭据，目的可能是将其出售给其他罪犯。

这个远程配置的加密矿模块自 2018 年 8 月开始分发，在撰写本文时仍处于活动状态，表明该团队正在不断创新并扩展其赚钱能力。除了其标准的加密挖掘功能外，该模块还采用了一些有趣的混淆技术，我们将在下一篇文章中介绍该技术以及一些可能的对策。

## 威胁指示器 (IoCs)

### ESET 检测名称

Win32/CoinMiner.Stantinko
Win64/CoinMiner.Stantinko

### SHA-1


可以从[我们的 GitHub 仓库](https://github.com/eset/malware-ioc/tree/master/stantinko)获取 1500 多个哈希的完整列表。

00F0AED42011C9DB7807383868AF82EF5454FDD8
01504C2CE8180D3F136DC3C8D6DDDDBD2662A4BF
0177DDD5C60E9A808DB4626AB3161794E08DEF74
01A53BAC150E5727F12E96BE5AAB782CDEF36713
01BFAD430CFA034B039AC9ACC98098EB53A1A703
01FE45376349628ED402D8D74868E463F9047C30

### 文件名

api-ms-win-crt-io-l1-1-0.dll
libcr64.dll
C:\Windows\TEMP\%RANDOM%\%RANDOM_GUID%.dll

### 互斥体名以及 RC4 密钥

“3,141592653589793238462643”

### 带有挖矿代理配置数据的 YouTube 链接

* https://www.youtube[.]com/watch?v=kS1jXg99WiM
* https://www.youtube[.]com/watch?v=70g4kw2iRGo
* https://www.youtube[.]com/watch?v=cAW1xEpyr7Y
* https://www.youtube[.]com/watch?v=6SSKQdE5Vjo
* https://www.youtube[.]com/watch?v=fACKZewW22M
* https://www.youtube[.]com/watch?v=FDQOa5zCv3s
* https://www.youtube[.]com/watch?v=TpyOURRvFmE
* https://www.youtube[.]com/watch?v=2fpiR4NIpsU
* https://www.youtube[.]com/watch?v=TwnD0Kp_Ohc
* https://www.youtube[.]com/watch?v=wJsbj8zPPNs

### 挖矿代理的 IP 地址

* 3.16.150[.]123
* 3.16.152[.]201
* 3.16.152[.]64
* 3.16.167[.]92
* 3.16.30[.]155
* 3.16.31[.]23
* 3.17.167[.]43
* 3.17.23[.]144
* 3.17.25[.]11
* 3.17.59[.]6
* 3.17.61[.]161
* 3.18.108[.]152
* 3.18.223[.]195
* 13.58.182[.]92
* 13.58.22[.]81
* 13.58.77[.]225
* 13.59.31[.]61
* 18.188.122[.]218
* 18.188.126[.]190
* 18.188.249[.]210
* 18.188.47[.]132
* 18.188.93[.]252
* 18.191.104[.]117
* 18.191.173[.]48
* 18.191.216[.]242
* 18.191.230[.]253
* 18.191.241[.]159
* 18.191.47[.]76
* 18.216.127[.]143
* 18.216.37[.]78
* 18.216.55[.]205
* 18.216.71[.]102
* 18.217.146[.]44
* 18.217.177[.]214
* 18.218.20[.]166
* 18.220.29[.]72
* 18.221.25[.]98
* 18.221.46[.]136
* 18.222.10[.]104
* 18.222.187[.]174
* 18.222.198[.]38
* 18.222.213[.]203
* 18.222.253[.]209
* 18.222.56[.]98
* 18.223.111[.]224
* 18.223.112[.]155
* 18.223.131[.]52
* 18.223.136[.]87
* 18.225.31[.]210
* 18.225.32[.]44
* 18.225.7[.]128
* 18.225.8[.]249
* 52.14.103[.]72
* 52.14.221[.]47
* 52.15.184[.]25
* 52.15.222[.]174

### MITRE ATT&CK 技术

<table>
<tr><td>技术</td>	<td>ID</td><td>名称</td><td>描述</td></tr>

<tr><td rowspan="2">执行</td>	<td><a href="https://attack.mitre.org/techniques/T1085/">T1085</a></td>	<td>Rundll32</td>	<td>该模块由rundll32.exe 执行。</td>
<tr><td><a href="https://attack.mitre.org/techniques/T1035/">T1035</a></td>	<td>服务执行</td>	<td>这个恶意软件可以以服务形式执行。</td></tr>
<tr><td rowspan="3">防御规避</td><td><a href="https://attack.mitre.org/techniques/T1140">T1140</a></td><td>反混淆/解码文件或信息</td>	<td>在执行过程中，模块将对其代码中的字符串进行反混淆处理。</td></tr>
<tr><td><a href="https://attack.mitre.org/techniques/T1027/">T1027</a></td>	<td>混淆的文件或信息</td>	<td>该模块混淆了其代码和字符串，这显然使分析和检测变得困难。</td></tr>
<tr><td><a href="https://attack.mitre.org/techniques/T1102/">T1102</a></td>	<td>Web 服务</td>	<td>该恶意软件从 YouTube 视频的描述中获取配置数据。</td></tr>
<tr><td>发现</td>	<td><a href="https://attack.mitre.org/techniques/T1063/">T1063</a></td>	<td>安全软件发现</td>	<td>该恶意软件获取正在运行的安全产品列表。</td></tr>
<tr><td rowspan="7">命令与控制</td>	<td><a href="https://attack.mitre.org/techniques/T1090/">T1090</a></td>	<td>连接代理</td>	<td>该模块在其自身与矿池之间使用代理。</td></tr>
<tr><td><a href="https://attack.mitre.org/techniques/T1008/">T1008</a></td>	<td>备用频道</td>	<td>如果无法访问初始挖掘代理，则该模块将连接到另一个挖掘代理。</td>
<tr><td><a href="https://attack.mitre.org/techniques/T1095/">T1095</a></td>	<td>标准非应用层协议</td>	<td>该恶意软件使用 TCP 进行通信。</td></tr>
<tr><td><a href="https://attack.mitre.org/techniques/T1043/">T1043</a></td>	<td>常用端口</td>	<td>恶意软件通过端口 443 通信。</td></tr>
<tr><td><a href="https://attack.mitre.org/techniques/T1132/">T1132</a></td>	<td>数据编码</td>	<td>模块加密，然后 base64 编码一些网络流量。</td></tr>
<tr><td><a href="https://attack.mitre.org/techniques/T1032/">T1032</a></td>	<td>标准加密协议</td>	<td>该模块使用 RC4 加密流量。</td></tr>
<tr><td><a href="https://attack.mitre.org/techniques/T1071/">T1071</a></td>	<td>标准应用层协议</td>	<td>通过 HTTPS 从 YouTube 视频的描述中获取配置数据。</td></tr>
<tr><td>影响</td>	<td><a href="https://attack.mitre.org/techniques/T1496/">T1496</a></td>	<td>资源劫持</td>	<td>该模块挖掘加密货币。</td></tr>
</table>

## 附录


下面列出了 CoinMiner.Stantinko 检查的 CRC-32 校验和以及它们对应的文件名。


0xB18362C7	afwserv.exe
***
0x05838A63	ashdisp.exe
***
0x36C5019C	ashwebsv.exe
***
0xB3C17664	aswidsagent.exe
***
0x648E8307	avastsvc.exe
***
0x281AC78F	avastui.exe
***
0xAA0D8BF4	avgcsrva.exe
***
0x71B621D6	avgcsrvx.exe
***
0x7D6D668A	avgfws.exe
***
0x1EF12475	avgidsagent.exe
***
0x010B6C80	avgmfapx.exe
***
0x6E691216	avgnsa.exe
***
0xB5D2B834	avgnsx.exe
***
0x36602D00	avgnt.exe
***
0x222EBF57	avgrsa.exe
***
0xF9951575	avgrsx.exe
***
0x2377F90C	avgsvc.exe
***
0x37FAB74F	avgsvca.exe
***
0xEC411D6D	avgsvcx.exe
***
0x0BED9FA2	avgtray.exe
***
0x168022D0	avguard.exe
***
0x99BA6EAA	avgui.exe
***
0x7A77BA28	avguix.exe
***
0x022F74A	avgwdsvc.exe
***
0x98313E09	avira.servicehost.exe
***
0x507E7C15	avira.systray.exe
***
0xFF934F08	avp.exe
***
0x9AC5F806	avpui.exe
***
0xBD07F203	avshadow.exe
***
0x64FDC22A	avwebg7.exe
***
0x0BC69161	avwebgrd.exe
***
0xBACF2EAC	cureit.exe
***
0x8FDEA9A9	drwagntd.exe
***
0xE1856E76	drwagnui.exe
***
0xF9BF908E	drwcsd.exe
***
0xC84AB1DA	drwebcom.exe
***
0x183AA5AC	drwebupw.exe
***
0xAC255C5E	drwupsrv.exe
***
0x23B9BE14	dwantispam.exe
***
0xDAC9F2B7	dwarkdaemon.exe
***
0x7400E3CB	dwengine.exe
***
0x73982213	dwnetfilter.exe
***
0x1C6830BC	dwscanner.exe
***
0x86D81873	dwservice.exe
***
0xB1D6E120	dwwatcher.exe
***
0xD56C1E6F	egui.exe
***
0x69DD7DB4	ekrn.exe
***
0xFB1C0526	guardgui.exe
***
0x5BC1D859	ipmgui.exe
***
0x07711AAE	ksde.exe
***
0x479CB9C4	ksdeui.exe
***
0x6B026A91	nod32cc.exe 
***
0xCFFC2DBB	nod32krn.exe
***
0x59B8DF4D	nod32kui.exe
***
0x998B5896	procexp.exe
***
0xF3EEEFA8	procexp64.exe
***
0x81C16803	sched.exe
***
0x31F6B864	spideragent.exe
***
0x822C2BA2	taskmgr.exe
***
0x092E6ADA	updrgui.exe
***
0x09375DFF	wsctool.exe
***


