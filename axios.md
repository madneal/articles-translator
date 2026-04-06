>原文：[核心要点与行动指南：Axios NPM 供应链入侵紧急通报](https://www.sans.org/blog/what-we-learned-axios-npm-supply-chain-compromise-emergency-briefing)
>
>译者：[madneal](https://github.com/madneal)
>
>welcome to star my [articles-translator](https://github.com/madneal/articles-translator/), providing you advanced articles translation. Any suggestion, please issue or contact [me](mailto:bing@stu.ecnu.edu.cn)
>
>LICENSE: [MIT](https://opensource.org/licenses/MIT)

**作者**：SANS Institute
![SANS Institute](https://images.contentstack.io/v3/assets/bltabe50a4554f8e97f/blt12dfb7ac1329de96/69bd2121daccab04d60ba5e1/SANS_Profile_Image_2.jpg?width=768&quality=75&format=webp)

SANS 学院研究员 **Joshua Wright** 和认证讲师 **Rich Greene** 在 2026 年 SANS 奥兰多大会现场，针对仍在发酵的 Axios NPM 供应链入侵事件进行了紧急直播剖析。由于事发突然且形势严峻，两人甚至是在午休期间中断了课程录制来进行这场直播。

如果您还没有观看回放，以下是您需要了解的核心信息。

### 发生了什么

3月31日 UTC 时间刚过午夜，有威胁行为者发布了受损版本的 axios NPM 包（v1.14.1 和 v0.30.4）。axios 是全球使用最广泛的 JavaScript 库之一，估计每周下载量高达 8000 万至 1 亿次。这些恶意版本引入了一个名为 **plain-crypto-js** 的依赖项，该依赖项在 Windows、macOS 和 Linux 系统上部署了远程访问木马（RAT）。该恶意包在被下架前存活了大约三个小时。在此期间，预计可能发生了 60 万次安装。

该 RAT 立即开始抓取凭证，收集 GitHub 个人访问令牌、AWS 密钥、Azure 凭证以及其他身份验证材料。整个过程无需任何用户交互——只要拉取了该包，就会被攻陷。

### 为什么这不仅仅是一个包的问题

Josh 指出，供应链风险不在于你直接安装的软件，而在于“你安装的软件所依赖的软件”。他以 7-Zip 为例，该软件拥有 300 个外部依赖项。您的应用程序可能没有直接调用 axios，但您依赖的某个库可能会调用，而该库的底层依赖也可能会调用。

正如 Josh 所说：*“这就是层层嵌套的深渊（It’s turtles all the way down）。”*

这正是 Josh 上周在 RSAC 大会上所演讲的主题，当时他警告说，供应链入侵对攻击者来说是一个无法忽视的巨大诱惑。五天后，预言成真了。

### 幕后黑手可能是谁

**更新（4月2日）**：谷歌威胁情报组（GTIG）和微软威胁情报均已将此次攻击归因于朝鲜（DPRK）国家级黑客组织。谷歌将该组织追踪为 UNC1069；微软将其追踪为 Sapphire Sleet。该恶意软件已被正式认定为 WAVESHAPER.V2。这与 Trivy 和 LiteLLM 攻击背后的 TeamPCP 活动不同。Josh 在简报中提出“朝鲜可能牵涉其中”的评估现已得到证实。

*参考资料：[微软安全博客：缓解 Axios npm 供应链入侵](https://www.microsoft.com/en-us/security/blog/2026/04/01/mitigating-the-axios-npm-supply-chain-compromise/)*

虽然归因仍处于早期阶段，但 Josh 分享说，此次事件具有 **Team PCP**（与 Trivy 和 LiteLLM 供应链攻击有关的英语威胁组织）的特征。然而，axios 攻击在复杂性上表现出的一些差异表明可能有第二个威胁组织介入，极有可能是与朝鲜有关的组织。

Josh 的推测是：Team PCP 可能正在通过将早期受损的访问权限出售给其他威胁组织来变现。这是供应链攻击运作方式的一个重大进展。

### 多平台有效载荷释放了什么信号

受损的包交付了特定于平台的有效载荷：用于 macOS 的 AppleScript、用于 Windows 的 PowerShell 以及用于 Linux 的 Python。Josh 指出，这种多平台交付的速度和协调性可能是 AI 辅助自动化的一个指标。虽然尚未证实，但这种操作节奏表明，攻击者使用的工具已经超越了单个团队手动操作的范畴。

拉响此次事件最初警报并批准紧急响应的 SANS 总裁 Ed Skoudis 坦言：多平台覆盖不仅仅是一个技术细节，它更是攻击意图的信号。*“它代表着攻击者具有更彻底、更广泛的目标。这些人野心勃勃，企图攻陷视线范围内的一切。”*

### 现在的行动指南：三大受众，三大优先级

Josh 明确表示，修复工作需要跨部门协调，这绝不是单个团队能够解决的问题。

#### 1. 事件响应团队 (Incident Response Teams)
如果开发人员或 DevOps 确认受到了影响，请将其作为已确认的事件处理。Josh 梳理了 SANS 响应行动循环：范围界定、遏制、根除、恢复。
*   首先界定攻击者可能访问了哪些凭证。
*   如果受损系统包含 AWS 凭证，那么您的整个 AWS 环境现在都在风险范围内。
*   遏制并不一定意味着关闭所有系统，但确实意味着必须立即通过安全组、ACL 和凭证轮换来限制攻击者的访问权限。
*   警惕**凭证蔓延（credential sprawl）**——一组被盗的密钥会导致另一组泄露，然后层层递进。Josh 警告说，他在去年 RSAC 上介绍过的这种模式，直接适用于当前情况。

#### 2. DevOps / CI/CD 团队
如果您运行每晚构建、冒烟测试或任何解析 NPM 依赖项的自动化流水线，请检查在 3月31日 00:00 至 03:00 (UTC) 期间是否有任何构建运行。如果这些构建拉取了 axios，请假设已被攻陷。这一点尤为关键，因为 CI/CD 环境通常保存着高价值的机密信息：NPM 令牌、SSH 密钥、云凭证和 API 密钥。

#### 3. 开发人员
搜索所有源代码存储库中任何对 axios 的使用，无论是直接使用还是作为传递依赖项。特别检查 v1.14.1 或 v0.30.4 版本。如果在您的 `node_modules` 中发现任何地方存在 `plain-crypto-js` 包，这就确认了您已受到入侵。

### 为什么危机尚未结束

Ed Skoudis 强调，此次攻击获取的凭证才是真正的重头戏。axios 入侵本身只是开局之作。攻击者利用它窃取了凭证，从而获得了远远超出单个代码包的访问权限。*“如果攻击者足够聪明，他们现在会保持沉寂，然后在稍后的某个时间点让我们大吃一惊，展示他们通过这次攻击获得了对其他我们尚未察觉的代码包的访问权限。”*

换句话说：今天的事件响应是针对 axios 的；而明天的响应将是针对那些被盗凭证所解锁的所有系统。Ed 直言不讳：*“这次攻击通过今天收集到的凭证，可能会产生长期的影响，我们将在未来几个月甚至更长的时间里面对它的余波。”*

这意味着凭证轮换不是可选项，也不是一次性任务。如果您的环境暴露了，请假设攻击者已经掌握了您尚未发现的令牌、密钥和访问权限，并据此制定计划。

### 需要监控的关键妥协指标 (IOCs)

*   **恶意版本**：axios v1.14.1 和 v0.30.4
*   **恶意依赖项**：plain-crypto-js（任意版本）
*   **C2 域名**：sfrclak[.]com
*   **暴露窗口**：2026年3月31日 00:00 至 ~03:00 UTC
*   **有效载荷**：AppleScript (macOS)、PowerShell (Windows)、Python (Linux)
*   **恶意软件家族**：WAVESHAPER.V2（由谷歌 GTIG 追踪；归因于 UNC1069）
*   **目标**：GitHub PATs、AWS 密钥、Azure 凭证、SSH 密钥、云令牌

更多确定的 IOC 将发布在 SANS 博客上。

### 补充建议

Josh 在结束时强调了一点。事件响应是一场马拉松，而不是短跑。如果您的团队今天凌晨 2 点接到了电话，那么他们接下来将面临漫长的不眠之夜。请确保他们得到充足的资源、休息和支持。

正如 Josh 所说：*“无事可做的事件响应团队不是一个有效的事件响应团队。”*

照顾好你的团队，他们将非常需要。

*   [**观看完整回放**](https://www.sans.org/mlp/emergency-livestream-axios-npm-supply-chain-compromise)
*   [**阅读包含 IOCs 的技术博客**](https://www.sans.org/blog/axios-npm-supply-chain-compromise-malicious-packages-remote-access-trojan)

**关注 SANS 社交渠道，随时获取最新进展。**

随着更多细节的披露，我们将继续向社区通报。如果您的组织受到了影响，SANS 在此提供帮助。
