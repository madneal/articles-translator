# 为什么 2022 年是漏洞赏金奖破纪录的一年

>原文：[Why 2022 was a record-breaking year in bug bounty awards](https://about.gitlab.com/blog/2022/12/19/why-2022-was-a-record-breaking-year-in-bug-bounty-awards/)
>
>译者：[madneal](https://github.com/madneal)
>
>welcome to star my [articles-translator](https://github.com/madneal/articles-translator/), providing you advanced articles translation. Any suggestion, please issue or contact [me](mailto:bing@stu.ecnu.edu.cn)
>
>LICENSE: [MIT](https://opensource.org/licenses/MIT)

![](/images/blogimages/inside-gitLab-public-bug-bounty-program.png)

每年，GitLab 的[应用安全团队](https://about.gitlab.com/handbook/security/security-engineering/application-security/) 都会回顾 GitLab 漏洞赏金计划的亮点。

对于整个行业的安全团队来说，2022 年是忙碌的一年，我们很幸运收到了大量出色的报告，帮助我们确保 GitLab 及其客户的安全。 随着我们在 2021 年 11 月 [增加我们的漏洞赏金奖励金额](https://about.gitlab.com/blog/2021/11/01/3rd-annual-bug-bounty-contest/#-increased-bounties-across-all-bounty-ranges-)和研究人员参与度的提高，我们在 2022 年期间**奖励超过 100 万美元**，打破了新纪录！

如果没有我们的漏洞赏金社区的合作，我们就不会取得今天的成就，我们认为这些奖励非常有益，而且钱花得值。

2022 年的数字[](##2022-by-the-numbers)
------------------------------------------

* 在 221 份有效报告中获得总计 1,055,770 美元的奖金，高于去年的 337,780 美元！
* 三名研究人员在他们的多份报告中获得了 10 万美元以上的收入，另外七名研究人员的收入超过了 2 万美元。
* 2022年共收到424名研究人员的920份报告。
* 解决了 158 份有效报告并公开了 94 份 - 今年，我们收到了一些信息泄漏报告，与漏洞不同，这些报告不需要公开 GitLab 问题。
* 今年有 138 名安全研究人员提交了一份以上的报告，表明他们对我们的计划做出了积极的贡献。
* 向提交三份或更多有效报告的研究人员授予八份 GitLab Ultimate 许可证。

_注：数据为截至 2022 年 12 月 16 日。_

您可以在我们的 [HackerOne 项目主页](https://hackerone.com/gitlab) 上查看每天更新的项目统计信息。如果您想参与其中，这也是开始我们计划的地方！

脱颖而出的报告和上报者[](##reports-and-reporters-that-stand-out)
---------------------------------------------- --------------------------

**我们计划的最有效报告**。祝贺 [@joaxcar](https://hackerone.com/joaxcar) 在 2022 年提交了 22 份有效且现已解决的报告。

**来自我们计划新人的最有效报告**， 欢迎并祝贺 [@albatraoz](https://hackerone.com/albatraoz) 在 2022 年提出了七份有效且现已解决的报告。

**最佳书面报告**。干得好，谢谢 [@yvvdwf](https://hackerone.com/yvvdwf) 编写了一个非常有趣的 [远程代码执行错误](https://gitlab.com/gitlab-org/gitlab/-/issues/371098?_gl=1*w2k5zo*_ga*MTE4NzUwNTYwNC4xNjcyODE5MjA1*_ga_ENFH3X7M5Y*MTY3MzgyOTM5OS4zLjEuMTY3MzgyOTkzOC4wLjAuMA..)。代码和根本原因的 walkthrough、创建虚拟恶意服务器的脚本，以及在验证期间与我们的 AppSec 团队的协作非常棒！

**最具创新性的报告**。击掌，[@vakzz](https://hackerone.com/vakzz)，他用[新颖的本地 `git` 读取漏洞](https://gitlab.com/gitlab-org/gitlab/-/issues/372165)夺得了旗帜！ 他还对上述 `@yvvdwf` 的 RCE 进行了[简洁的跟进](https://gitlab.com/gitlab-org/gitlab/-/issues/371884)。

**最具影响力的发现**。我们很高兴认识到 [@taraszelyk](https://hackerone.com/taraszelyk)，他连续提交的信息披露导致 GitLab 内部发生了许多积极的安全变化。谢谢，Taras！

我们将与这些研究人员取得联系，寄出 [GitLab Swag Shop](https://shop.gitlab.com) 代金券以示感谢。

2022 年所做的更改[](##changes-made-in-2022)
------------------------------------------

* 我们采用了 HackerOne 的黄金标准安全港声明。请参阅 [来自 HackerOne 的公告](https://www.hackerone.com/press-release/hackerone-announces-gold-standard-safe-harbor-improve-protections-good-faith-security)。
* 我们推出了 [20,000 美元的夺旗奖金](https://hackerone.com/gitlab#user-content-capture-the-flag-for-20000)，[已经被获得了一次](https://gitlab.com/gitlab-org/gitlab/-/issues/372165)。
* 我们创建了 [HackerOne Questions](https://gitlab.com/gitlab-com/gl-security/appsec/hackerone-questions/)，这是一个专门用于在 HackerOne 报告之外与 AppSec 团队联系的空间。
* 创建了 ["Reproducible Vulnerabilities"](/handbook/security/security-engineering-and-research/application-security/reproducible-vulnerabilities.html)，这是我们手册中的全新学习资源，具有可扩展的提示部分，以便您可以挑战自己并学习如何找到真正的安全漏洞。
* 继续透明地迭代我们的 [HackerOne 分类流程](https://gitlab.com/gitlab-com/www-gitlab-com/-/commits/master/sites/handbook/source/handbook/security/security-engineering-and-research/application-security/runbooks/hackerone-process.html.md?_gl=1*q768y9*_ga*MTE4NzUwNTYwNC4xNjcyODE5MjA1*_ga_ENFH3X7M5Y*MTY3MzgyOTM5OS4zLjEuMTY3MzgyOTkzOC4wLjAuMA..)，以及我们的 [漏洞赏金计算器](https://gitlab.com/gitlab-com/gl-security/appsec/cvss-calculator/-/commits/master)，包括标准化非漏洞报告的数量，如信息泄漏。

今年，我们还继续提供有助于研究人员和其他运行漏洞赏金计划的组织的内容：

* GitLab 博客：[“想开始黑客攻击吗？这是快速深入的方法”](https://about.gitlab.com//blog/2022/07/27/cracking-our-bug-bounty-top-10/)
* GitLab 博客：[“GitLab 如何处理安全漏洞（及其重要性）”](https://about.gitlab.com//blog/2022/02/17/how-gitlab-handles-security-bugs/)
* YouTube：[NullCon 2022 视频座谈会：“CXO 座谈会：漏洞赏金？太棒了！现在怎么办？”](https://www.youtube.com/watch?v=uqvaiml1iV4)

一如既往，与我们行业最好的安全研究人员（包括许多新人）一起工作真的很高兴。 GitLab 的 AppSec 团队致力于在漏洞赏金计划和奖励的透明度方面成为行业领导者。 [让我们知道我们在做什么](https://gitlab.com/gitlab-com/gl-security/appsec/hackerone-questions/) 这样我们就可以迭代我们的程序流程。

为 2023 年干杯 - 快乐挖洞！

