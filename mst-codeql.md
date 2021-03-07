# 微软开源对于 Solorigate 活动捕获的开源 CodeQL 查询

>原文：[微软 open sources CodeQL queries used to hunt for Solorigate activity](https://www.microsoft.com/security/blog/2021/02/25/microsoft-open-sources-codeql-queries-used-to-hunt-for-solorigate-activity/)
>
>译者：[madneal](https://github.com/madneal)
>
>welcome to star my [articles-translator](https://github.com/madneal/articles-translator/), providing you advanced articles translation. Any suggestion, please issue or contact [me](mailto:bing@stu.ecnu.edu.cn)
>
>LICENSE: [MIT](https://opensource.org/licenses/MIT)

Solorigate 攻击的一个关键方面是供应链攻击，这使攻击者可以修改 SolarWinds Orion 产品中的二进制文件。这些经过修改的二进制文件是通过以前合法的更新渠道分发的，并允许攻击者远程执行恶意活动，例如窃取凭据，提权和横向移动，以窃取敏感信息。该事件提醒组织不仅要考虑是否准备好应对复杂的攻击，还需要考虑自己代码库的弹性。

微软坚信以透明的方式进行领导并与社区共享情报，从而改善整个行业的安全实践和状况。在此博客中，我们将分享审查代码库的过程，重点介绍一种特定的技术：使用 [CodeQL](https://securitylab.github.com/tools/codeql) 查询来大规模分析我们的源代码，并排除存在代码级别的危威胁情报（IoCs）和与 Solorigate 相关的代码模式。我们正在将本次本调查中使用的 [CodeQL 查询](https://github.com/github/codeql/pull/5083)开源，以便其他组织可以执行类似的分析。请注意，我们在此博客中介绍的查询仅可用于查找与 Solorigate 植入程序中的源代码具有相似之处的源代码，无论是在语法元素（名称，字面量等）还是功能上。两者可能在良性代码中同时发生，因此所有发现都需要进行审查以确定它们是否可行。此外，不能保证恶意行为者在其他操作中被约束为相同的功能或编码风格，因此这些查询可能无法检测到与在 Solorigate 植入代码中看到的策略有明显差异的其他植入代码。这些应被视为只针对攻击[审计技术](https://techcommunity.microsoft.com/t5/azure-sentinel/solarwinds-post-compromise-hunting-with-azure-sentinel/ba-p/1995095)的一部分。

长期以来，微软一直采用完整性控制来验证分发给我们的服务器和客户的最终编译二进制文件在开发和发布周期的任何时候都没有被恶意修改。例如，我们验证编译器生成的源文件哈希是否与原始源文件匹配。尽管如此，在微软，我们仍然秉承 “assume breach” 的理念，该理念告诉我们，无论我们的安全实践多么勤奋和广泛，潜在的对手都可以同样地聪明并拥有大量资源。作为 Solorigate 调查的一部分，我们使用了自动和手动技术来验证我们的源代码，构建环境以及生产二进制文件和环境的完整性。

微软在 Solorigate 调查期间的贡献反映了我们对 [Githubification of InfoSec](https://medium.com/@johnlatwc/the-githubification-of-infosec-afbdbfaad1d1) 中描述的基于社区的共享愿景的承诺。为了保持我们对防御者知识的了解并加快社区对复杂威胁的响应的愿景，微软团队在此次事件期间公开透明地共享了[威胁情报](https://techcommunity.microsoft.com/t5/azure-active-directory-identity/understanding-quot-solorigate-quot-s-identity-iocs-for-identity/ba-p/2007610)，[详细的攻击分析和 MITER ATT＆CK 技术](https://www.microsoft.com/security/blog/2020/12/18/analyzing-solorigate-the-compromised-dll-file-that-started-a-sophisticated-cyberattack-and-how-microsoft-defender-helps-protect/)，[高级狩猎查询](https://techcommunity.microsoft.com/t5/azure-sentinel/solarwinds-post-compromise-hunting-with-azure-sentinel/ba-p/1995095)，[事件响应指南](https://www.microsoft.com/security/blog/2020/12/21/advice-for-incident-responders-on-recovery-from-systemic-identity-compromises/)以及[风险评估工作簿](https://techcommunity.microsoft.com/t5/azure-active-directory-identity/azure-ad-workbook-to-help-you-assess-solorigate-risk/ba-p/2010718)。微软鼓励其他安全组织开源自己的威胁知识和防御者技术来共享 “Githubification” 愿景，以加速防御者的洞察力和分析。如前所述，我们已在 https://aka.ms/solorigate 上收集了全面的资源，以提供有关攻击的技术详细信息，威胁情报和产品指南。作为微软全面调查 Solorigate 的一部分，我们检查了自己的环境。正如我们之前所[分享](https://msrc-blog.microsoft.com/2020/12/31/microsoft-internal-solorigate-investigation-update/)的那样，这些调查发现有少量内部帐户存在活动，并且一些帐户已用于查看源代码，但是我们没有发现任何对源代码，构建基础结构，已编译的二进制文件或生产环境进行任何修改的[证据](https://www.microsoft.com/security/blog/2021/02/18/turning-the-page-on-solorigate-and-opening-the-next-chapter-for-the-security-community/)。

## CodeQL 入门以及微软如何使用它

[CodeQL](https://securitylab.github.com/tools/codeql) 是一种功能强大的语义代码分析引擎，现在已经是 GitHub 的一部分。与许多分析解决方案不同，它在两个不同的阶段工作。首先，作为将源代码编译为二进制文件的一部分，CodeQL 建立了一个捕获编译代码模型的数据库。对于解释型语言，由于没有编译器，因此它将解析源并构建自己的抽象语法树模型。其次，该数据库一旦构建，便可以像其他任何数据库一样反复查询。CodeQL 语言是专用于构建可轻松从数据库中选择复杂的代码条件。

在微软我们发现 CodeQL 中发现如此多的实用性的原因之一，尤其是因为这种两阶段的方法释放了许多有用的场景，包括不仅可以将静态分析用于主动安全开发生命周期分析，而且还可以用于整个企业的反应性代码检查。我们将微软的各种构建系统或管道生成的 CodeQL 数据库聚合到一个集中式基础结构中，在该基础结构中，我们能够立即查询整个 CodeQL 数据库的范围。聚合 CodeQL 数据库使我们能够在众多代码库中进行语义搜索，并根据构建的一部分特定代码查找可能跨越多个程序集，库或模块的代码条件。我们拥有可以在描述的变体后数小时内分析成千上万的资源库的能力，以查找新描述的漏洞变体，但是它也使我们能够同样快速地对 Solorigate 植入模式进行首次通过调查。

![](https://www.microsoft.com/security/blog/wp-content/uploads/2021/02/Figure-1b-process.png)

我们正在开源这些代码级威胁情报的多个 C# 查询，目前可以在 [CodeQL GitHub 代码仓库](https://aka.ms/Solorigate-CodeQL-Queries)中找到它们。该仓库中的 [Solorigate-Readme.md](https://aka.ms/Solorigate-CodeQL-ReadMe) 包含每个查询的详细说明以及每个查询试图查找的代码级威胁情报。它还包含给其他查询作者的指南，这些指南涉及对那些查询进行调整或编写在查找模式时采用不同策略的查询。

GitHub 即将发布有关如何为现有 CodeQL 客户部署这些查询的指南。提醒一下，CodeQL 对于 GitHub 托管的开源项目是免费的。

## 我们使用 CodeQL 寻找代码级威胁情报的方法

在寻找代码级 Solorigate 威胁情报时，我们使用了两种不同的策略。一种方法是寻找在 Solorigate 代码级威胁情报中脱颖而出的特定语法。另一种方法则针对代码级威胁情报中存在的技术寻找整体语义模式。

与可比较的正则表达式搜索相比具有很多优势，语法查询的编写和执行速度非常快。但是，它们对于恶意角色更改其使用的名称和字面量很弱。语义模式寻找植入程序中使用的总体技术，例如哈希处理名称，联系 C2 服务器之前的时间延迟等。这些可以承受实质性的变化，但是它们在编写时更加复杂并且在一次分析很多代码库时更加耗费计算资源。

![](https://www.microsoft.com/security/blog/wp-content/uploads/2021/02/Figure-2a-sample-code.png)

通过组合这两种方法，查询能够检测到恶意行为者更改了技术但使用了相似语法，或者更改了语法但采用了相似技术的场景。由于恶意行为者可能会更改语法和技术，因此 CodeQL 只是我们大量调查工作的一部分。

## 使用 CodeQL 的下一步

我们在此博客中共享并在 [Solorigate-Readme.md](https://aka.ms/Solorigate-CodeQL-ReadMe) 目标模式中描述特别与 Solorigate 代码级威胁情报相关联的查询，但 CodeQL 还提供了许多其他选项来查询后门功能和逃避检测技术。

这些查询的编写速度相对较快，与使用源代码的文本搜索相比，我们能够在我们的 CodeQL 数据库中更准确地寻找模式，并且用更少的精力手动审查发现的结果。CodeQL 是一个功能强大的开发人员工具，我们希望这篇文章能激发组织探索如何使用它来改善反应式安全响应并充当入侵检测工具。

在以后的博客文章中，我们将分享微软使用 CodeQL 的更多方式。我们还将继续在CodeQL的基础上进行开放源代码的查询和实用程序，以便其他人可以从中受益并进一步建立在它们之上。

