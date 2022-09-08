# Go 的漏洞管理

>原文：[Vulnerability Management for Go](https://go.dev/blog/vuln)
>
>译者：[madneal](https://github.com/madneal)
>
>welcome to star my [articles-translator](https://github.com/madneal/articles-translator/), providing you advanced articles translation. Any suggestion, please issue or contact [me](mailto:bing@stu.ecnu.edu.cn)
>
>LICENSE: [MIT](https://opensource.org/licenses/MIT)

我们很高兴地宣布 Go 对漏洞管理的新支持，这是我们帮助 Go 开发人员了解可能影响他们的已知漏洞的第一步。

这篇文章概述了当前可用的内容以及该项目的后续计划。

# 概述

Go 提供工具来分析你的代码库来发现已知漏洞。该工具由 Go 漏洞数据库提供支持，该数据库由 Go 安全团队规划。Go 的工具通过仅显示代码实际调用的函数中的漏洞来减少结果中的噪音。

![Architecture diagram of Go's vulnerability management system](https://go.dev/blog/vuln/architecture.png)

# Go 漏洞数据库

Go 漏洞数据库 (https://vuln.go.dev) 是有关公共 Go 模块中可导入包中已知漏洞的综合信息源。

漏洞数据来自现有来源（例如 CVE 和 GHSA）以及来自 Go 包维护者的直接报告。Go 安全团队会审查这些信息并将其添加到数据库中。

我们鼓励包维护者在他们自己的项目中[提供](https://go.dev/s/vulndb-report-new)有关公共漏洞的信息，并[更新](https://go.dev/s/vulndb-report-feedback)其 Go 包中漏洞的现有信息。我们的目标是使报告过程成为一个非常容易的过程，因此请向我们反馈任何改进的[建议](https://golang.org/s/vuln-feedback)。

Go 漏洞数据库可以在浏览器中的 pkg.go.dev/vuln 中查看。 有关数据库的更多信息，请参阅 go.dev/security/vuln/database。

# 使用 govulcheck 检测漏洞

新的 [govulncheck 命令](https://pkg.go.dev/golang.org/x/vuln/cmd/govulncheck)是一种低噪音、可靠的方式，让 Go 用户了解可能影响他们项目的已知漏洞。 Govulncheck 分析你的代码库并仅根据代码中的哪些函数传递调用易受攻击的函数来发现实际影响你的漏洞。 要开始使用 govulncheck，你可以从项目中运行以下命令：

```
$ go install golang.org/x/vuln/cmd/govulncheck@latest
$ govulncheck ./...
```

Govulncheck 是一个独立的工具，允许在我们收集用户反馈的同时进行频繁更新和快速迭代。从长远来看，我们计划将 govulncheck 工具集成到主要的 Go 发行版中。

为了将漏洞检查直接集成到其他工具和流程中，[vulncheck](https://pkg.go.dev/golang.org/x/vuln/vulncheck) 包将 govulncheck 的功能导出为 Go API。

# 集成

在开发和部署过程中尽早了解漏洞总是更好的。 为此，我们将漏洞检测集成到现有的 Go 工具和服务中，例如 [Go 包发现站点](https://pkg.go.dev/)。例如，[此页面](https://pkg.go.dev/golang.org/x/text?tab=versions)显示了 `golang.org/x/text` 的每个版本中的已知漏洞。 通过 VS Code Go 扩展的漏洞检查功能也即将推出。

# 下一步

我们希望你会发现 Go 对漏洞管理的支持很有用，并帮助我们改进它！

Go 对漏洞管理的支持是一项正在积极开发的新功能。你应该预料到一些错误和[限制](https://pkg.go.dev/golang.org/x/vuln/cmd/govulncheck#hdr-Limitations)。

我们希望你通过以下方式做出贡献并帮助我们进行改进：

* 为你维护的 Go 软件包[提供新的](https://golang.org/s/vulndb-report-new)和[更新](https://go.dev/s/vulndb-report-feedback)有关公共漏洞的现有信息
* [参加此问卷调查](https://golang.org/s/govulncheck-feedback)以分享你使用 govulncheck 的经验
* 向我们发送有关问题和功能请求的[反馈](https://golang.org/s/vuln-feedback)

我们很高兴与你合作，建立一个更好、更安全的 Go 生态系统。

