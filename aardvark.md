# 介绍 Aardvark：OpenAI 智能体安全研究员

>原文：[绍 Aardvark：OpenAI 智能体安全研究员](https://openai.com/index/introducing-aardvark/)
>
>译者：[madneal](https://github.com/madneal)
>
>welcome to star my [articles-translator](https://github.com/madneal/articles-translator/), providing you advanced articles translation. Any suggestion, please issue or contact [me](mailto:bing@stu.ecnu.edu.cn)
>
>LICENSE: [MIT](https://opensource.org/licenses/MIT)

今天，我们宣布推出 Aardvark，这是一家由 GPT-5 提供支持的智能体安全研究员。

软件安全是技术领域最关键、最具挑战性的前沿领域之一。每年，在企业和开源代码库中都会发现数以万计的新漏洞。防御者面临着在对手之前发现和修补漏洞的艰巨任务。在 OpenAI，我们正在努力使这种平衡向有利于防御者的方向倾斜。

Aardvark 代表了人工智能和安全研究的突破：一种自主代理，可以帮助开发人员和安全团队大规模发现和修复安全漏洞。Aardvark 现已推出私人测试版，以验证和完善其在该领域的能力。

## Aardvark 的工作原理

Aardvark 不断分析源代码仓库，以识别漏洞、评估可利用性、确定严重性优先级并提出有针对性的补丁。

Aardvark 的工作原理是监控代码库的提交和更改、识别漏洞、如何利用它们并提出修复建议。Aardvark 不依赖传统的程序分析技术，如模糊测试或软件组合分析。相反，它使用大模型驱动的推理和工具使用来理解代码行为并识别漏洞。Aardvark 像人类安全研究人员一样寻找错误：通过阅读代码、分析代码、编写和运行测试、使用工具等等。

![](https://images.ctfassets.net/kftzwdyauwt9/4VNAPycxZna8DOgUmlDFuL/e5f0c2773a181f456f186ead677a9e02/Aardvark_Overview_Diagram_Desktop_Light__1_.svg?w=3840&q=80)

Aardvark 依靠多阶段流水线来识别、解释和修复漏洞：

* 分析：它首先分析完整的仓库，以生成反映其对项目安全目标和设计的理解的威胁模型。
* 提交扫描：在提交新代码时，它通过针对整个仓库和威胁模型检查提交级别更改来扫描漏洞。首次连接仓库时，Aardvark 将扫描其历史记录以识别现有问题。Aardvark 逐步解释了它发现的漏洞，并注释代码以供人工审查。
* 验证：一旦 Aardvark 发现潜在漏洞，它将尝试在隔离的沙盒环境中触发它以确认其可利用性。Aardvark 描述了为帮助确保向用户返回准确、高质量和低误报见解而采取的步骤。
* 修复：Aardvark 与 OpenAI Codex 集成，帮助修复发现的漏洞。它将 Codex 生成和 Aardvark 扫描的补丁附加到每个发现中，以便人工审查和高效的一键式补丁。

Aardvark 与工程师一起工作，与 GitHub、Codex 和现有工作流程集成，在不减慢开发速度的情况下提供清晰、可作的见解。虽然 Aardvark 是为安全而构建的，但在我们的测试中，我们发现它还可以发现逻辑缺陷、不完整的修复和隐私问题等错误。


## 当下的真正影响

Aardvark 已经服务了几个月，在 OpenAI 的内部代码库和外部 alpha 合作伙伴的代码库中持续运行。在 OpenAI 内部，它暴露了有意义的漏洞，并助长了 OpenAI 的防御态势。合作伙伴强调了其分析的深度，Aardvark 发现了仅在复杂条件下才会出现的问题。

在对“黄金”存储库的基准测试中，Aardvark 识别了 92% 的已知和合成引入的漏洞，证明了高召回率和现实世界的有效性。


## Aardvark 开源

Aardvark 还应用于开源项目，它已经发现了许多漏洞，我们负责任地披露了许多漏洞，其中 10 个漏洞已获得公共漏洞和暴露 （CVE） 标识符。

作为数十年来开放研究和负责任披露的受益者，我们致力于回馈社会，贡献工具和发现，使数字生态系统对每个人来说都更安全。我们计划为选择非商业性开源存储库提供无偿扫描，为开源软件生态系统和供应链的安全做出贡献。

我们最近[更新](https://openai.com/index/scaling-coordinated-vulnerability-disclosure/)了[外部协同披露政策](https://openai.com/policies/outbound-coordinated-disclosure-policy/)，该政策采取了对开发人员友好的立场，专注于协作和可扩展的影响，而不是可能给开发人员带来压力的严格披露时间表。我们预计像 Aardvark 这样的工具将导致越来越多的问题被发现，并希望以可持续的方式合作以实现长期弹性。


## 为什么重要

软件现在是每个行业的支柱，这意味着软件漏洞对企业、基础设施和社会来说是一种系统性风险。仅在 2024 年，就报告了超过 40,000 个 CVE。我们的测试表明，大约 1.2% 的提交引入了错误——微小的更改可能会产生巨大的后果。

Aardvark 代表了一种新的防御者优先模式：代理安全研究人员通过随着代码的发展提供持续保护来与团队合作。通过及早发现漏洞、验证现实世界的可利用性并提供明确的修复程序，Aardvark 可以在不减缓创新的情况下增强安全性。我们相信扩大安全专业知识的获取范围。我们从私人测试版开始，并将随着我们的学习扩大可用性。

## 私人测试版现已开放

我们邀请精选合作伙伴加入 Aardvark 私人测试版。参与者将获得早期访问权，并直接与我们的团队合作，以完善检测准确性、验证工作流程和报告经验。

我们希望验证各种环境中的性能。如果您的组织或开源项目有兴趣加入，您可以在[这里](https://www.openai.com/form/aardvark-beta-signup)申请。