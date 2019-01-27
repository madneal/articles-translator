# 使用ELK Stack建设SIEM

>原文：[Using the ELK Stack for SIEM](https://logz.io/blog/elk-siem/)
>
>译者：[neal1991](https://github.com/neal1991)
>
>welcome to star my [articles-translator](https://github.com/neal1991/articles-translator/), providing you advanced articles translation. Any suggestion, please issue or contact [me](mailto:bing@stu.ecnu.edu.cn)
>
>LICENSE: [MIT](https://opensource.org/licenses/MIT)

![](https://logz.io/wp-content/uploads/2018/06/using_the_elk_stack_for_siem_-_article.jpg)

任何 SIEM 系统的核心都是日志数据。有很多种。无论是来自服务器，防火墙，数据库还是网络路由器，日志都为分析人员提供了深入了解 IT 环境中发生事件的原始资料。

然而，在将这些材料转化为资源之前，需要采取几个关键步骤。数据需要收集，处理，规范化，增强和存储。这些步骤通常在术语“日志管理”下组合在一起，是任何 SIEM系统中必备的组件。

因此，ELK Stack 是当今世界上最流行的开源日志分析和管理平台，这绝非巧合，它是当前大多数[开源 SIEM 解决方案](https://logz.io/blog/open-source-siem-tools)的重要组成部分。 ELK 负责收集，分析，存储和分析，部分架构来源于 OSSEC Wazuh，SIEMonster 和 Apache Metron。

如果日志管理和日志分析是 SIEM 中唯一的组件，则 ELK Stack 可被视为有效的开源解决方案。但是当我们定义[SIEM 系统实际是什么](https://logz.io/blog/what-is-siem/)时，除了日志管理之外，还列出了很多组件列表。本文将尝试深入探讨 ELK Stack 是否可用于S IEM，缺少什么以及将其扩展到全功能 SIEM 解决方案所需的内容。

日志收集
--------------

如上所述，SIEM 系统涉及汇总来自多个数据源的数据。这些数据源将根据你的环境而有所不同，但很可能您将从你的应用程序，基础设施级别（例如服务器，数据库），安全控制（例如防火墙，VPN），网络基础设施（如路由器，DNS）外部安全数据库（例如威胁情报）。

这需要 ELK Stack 非常适合处理的聚合能力。使用[Beats](https://logz.io/blog/beats-tutorial/)和[Logstash](https://logz.io/blog/logstash-tutorial/)的组合，你可以构建日志记录体系结构由多个数据管道组成。 Beats 是轻量级日志转发器，可用作边缘主机上的代理来跟踪和转发不同类型的数据，最常见的 beat 是用于转发日志文件的 Filebeat。 Logstash 然后可用于聚合来自 beat 的数据，对其进行处理（见下文）并将其转发给流水线中的下一个组件。

由于涉及的数据量很大，并且需要挖掘不同的数据源，因此很可能需要多个 Logstash 实例来确保更具弹性的数据管道。不仅如此，还需要部署排队机制来确保处理数据突发，并且管道中各个组件之间的断开连接不会导致数据丢失。 Kafka 通常是在这种情况下使用的工具，在 Logstash 之前安装（其他工具，如 Redis 和 RabbitMQ 也被使用）。

因此，单独使用 ELK Stack 很可能不足以满足你的业务需求，并且其生成的数据也会增长。希望使用 ELK 进行 SIEM 的组织必须了解需要部署其他组件才能增加堆栈。

日志处理
--------------

收集数据并转发它当然只是 Logstash 在日志记录管道中的一部分。另一个关键任务，也是 SIEM中 非常重要的一个任务，就是处理和解析数据。

上面概述的所有数据源类型都以不同的格式生成数据。要在下一步中取得成功 - 即搜索数据和分析数据 - 数据需要进行标准化。这意味着将不同的日志消息分解为有意义的字段名称，在 Elasticsearch中 正确映射字段类型，并在必要时丰富特定字段。

人们不能忽略这一步骤的重要性。如果没有正确的解析，当你试图在 Kibana中 分析时，你的数据将毫无意义。 Logstash 是一个强大的工具，可以帮助你完成此关键任务。 Logstash 支持大量不同的过滤器插件，可以分解日志，使用地理信息丰富特定字段，例如，删除字段，添加字段等。

再一次，诸如 SIEM 系统所需的日志架构可能变得复杂。特别是，配置 Logstash 以处理各种日志类型将需要多个 Logstash 配置文件和 Logstash 实例。重复处理是复杂过滤器配置的结果，会影响 Logstash 性能。监控 Logstash 管道非常重要，监控 API（例如用于识别具有高 CPU 的 Java 线程的 Hot Thread API）可用于此目的。

存储和保留
---------------------

从不同数据源收集的日志数据需要存储在数据存储中。在使用 ELK 的情形下，[Elasticsearch](https://logz.io/blog/elasticsearch-tutorial/)扮演数据索引和存储的角色。

Elasticsearch 是目前最流行的数据库之一，事实上 - 它是 Linux 内核之后第二大下载的开源软件。这种积极的原因是多种多样的 - 它是开源的，相对容易建立，快速，可扩展并且有一个巨大的社区支持它。

当然，部署 Elasticsearch 集群只是第一步。由于我们正在讨论索引的大量数据集，随着时间的推移，这些数据的数量将不断增加，因此用于 SIEM 的任何 Elasticsearch 部署都需要具有极高的可扩展性和容错性。

这需要许多特定的子任务。我们已经提到使用排队机制来确保数据在丢失或数据突发时不会丢失，但是你还需要关注关键的 Elasticsearch 性能指标，如索引速率和节点 JVM 堆和 CPU。再次，你可以使用监控 API 来达到此目的。容量规划也很重要，如果你部署在云上，则自动扩展策略很可能是确保你有足够的资源进行索引所必需的。

另一个考虑是数据保留。

为了进行高效的事后取证和调查，你需要一个长期的存储策略。例如，如果你注意到源自特定 IP 的流量大幅增加，你需要比较这些历史数据以验证它是否为异常行为。一些攻击可能会在几个月内缓慢演变，并且作为分析师，拥有这些历史数据是成功检测模式和趋势的关键。

毋庸置疑，ELK Stack 不支持开箱即用的归档功能，因此你需要弄清楚自己保留数据的体系结构。理想情况下，不会在财务上削弱你的组织。

查询
--------

一旦你的数据在 Elasticsearch中 收集，分析并建立索引，下一步就是查询数据。 你可以使用 Elasticsearch REST API 来做到这一点，但很可能你会为此使用Kibana。

在[Kibana](https://logz.io/blog/kibana-tutorial/)中，使用 Lucene 语法查询数据。 例如，常见的搜索类型是字段级搜索。 例如，假设我正在查找组织中某个人执行的操作所生成的所有日志消息。 因为我在所有数据源中标准化了一个名为 _username_ 的字段，所以我可以使用这个简单的查询：

`username:”Daniel Berman”`

这种查询可以使用逻辑符，比如 AND, OR, NOT

`username:”Daniel Berman” AND type:login ANS status:failed`

同样，如果你想使用 ELK Stack 用于 SIEM，你将需要利用 Logstash 的分析能力来处理你的数据 - 你如何设法做到这一点会影响你轻松浏览你导入的多个数据源的查询方式。 

仪表盘
----------

Kibana 以其可视化功能而闻名，支持各种不同的可视化类型，并允许用户以他们喜欢的任何方式对其数据进行切片和裁切。您可以创建饼图，图表，地图，单个度量标准，数据表等等，结果非常有效。

以下是针对AWS环境在Kibana中构建的SIEM仪表板的示例：

![仪表盘](https://logz.io/wp-content/uploads/2018/06/image1.png)

在 Kibana中 创建仪表板不是一项简单的任务，需要熟悉数据和构建日志消息的不同字段。更有甚者，Kibana 还缺少特定的功能，例如可视化中的动态链接。有[解决方法](https://logz.io/blog/kibana-hacks/)，但内置功能将会让你受益良多。

Kibana 也不支持安全共享对象。如果你发现安全漏洞并希望与同事共享仪表板或单个可视化文件，则 Kibana 中的共享链接不会被标记。你可以在 Kibana（X-Pack）或可以使用的开源解决方案之上实施商业附加组件。

关联
-----------

SIEM 的另一个关键要素是事件关联。 正如我们在[之前的文章](https://logz.io/blog/what-is-siem/)中已经定义的那样，事件关联是将来自不同数据源的信息连接成一种模式， 表明在安全方面有问题。 相关性规则定义了形成这种模式的特定事件序列。

例如，可以创建规则以识别何时在特定时间段内从特定 IP 范围和端口发送超过x个请求量。 关联规则的另一个示例将与特权帐户的创建一起寻找异常数量的失败登录。

这些关联规则由各种 SIEM 工具提供或针对不同的攻击情景预定义。 ELK Stack 当然没有内置的关联规则，因此分析人员可以根据使用 Logstash 执行的解析和处理来使用 Kibana 查询来关联事件。

警报
------

没有警报，关联规则就没有什么意义。 在识别可能的攻击模式时发出警报是 SIEM 系统的关键组成部分。

继续上面的例子，如果你的系统记录了来自特定 IP 范围的大量请求或异常数量的登录失败，则需要将警报发送给组织中正确的人员或团队。 速度是关键 - 通知发送得越快，缓解成功的机会就越大。

ELK Stack 以其开放源代码形式，没有提供内置的警报机制。 为了增加这个功能，ELK Stack 需要增加一个警报插件或附件。 再次，X-Pack 是一种选择。 另一个选择是添加 [ElastAlert](https://github.com/Yelp/elastalert)- 一个可以添加到 Elasticsearch 之上的开源框架。

事件管理
-------------------

问题明确后，分析人员发出警报。 现在怎么办？ 你的组织如何对事件做出响应将决定结果。 SIEM 系统旨在帮助安全人员的下一步 - 包含事件，必要时升级它，缓解它并扫描漏洞。

ELK Stack 在帮助分析人员识别事件但对管理事件没有太多帮助时非常棒。 即使在堆栈顶部实施警报附加功能，为了有效管理事件，也需要管理触发警报的方法。 否则，可能会迷失在众多警报中并且错过重要事件。 自动化升级过程和创建票据对于有效的事件处理也很重要

总结
-------------

那么，ELK Stack 可以用于 SIEM 吗？

这个问题的答案很简单。 在其原始形式中，由 Logstash，Elasticsearch，Kibana 和 Beats 组成 - ELK Stack **不是** SIEM解决方案。

我们来总结一下上面的关键点：

![chart](https://logz.io/wp-content/uploads/2018/06/chart-1.png)

虽然这是一个非常强大的集中日志记录工具，但 ELK Stack 不能直接用于 SIEM。 缺少内置警报功能，关联规则和缓解功能 - ELK Stack 无法完成安全分析人员所需的完整工具箱。

当然，ELK Stack 可以增加其他平台和服务。 这正是市场上的几种[开源SIEM解决方案](https://logz.io/blog/open-source-siem-tools/)所做的。 但是这需要组织的巨大工程技术专长。 将 ELK Stack 与其他附加组件和平台合并所需的资源和技术知识的数量，更不用说财务成本，因此选择商业 SIEM 也成为一个不错的选择。
