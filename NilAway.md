# NilAway：实用的 Go Nil Panic 检测方式

>原文：[NilAway: Practical Nil Panic Detection for Go](https://www.uber.com/en-NL/blog/nilaway-practical-nil-panic-detection-for-go/)
>
>译者：[madneal](https://github.com/madneal)
>
>welcome to star my [articles-translator](https://github.com/madneal/articles-translator/), providing you advanced articles translation. Any suggestion, please issue or contact [me](mailto:bing@stu.ecnu.edu.cn)
>
>LICENSE: [MIT](https://opensource.org/licenses/MIT)

![](https://blog.uber-cdn.com/cdn-cgi/image/width=2048,quality=80,onerror=redirect,format=auto/wp-content/uploads/2023/11/cover_photo.jpg)

Uber 由于 [Go](https://go.dev/?uclick_id=6f537554-73b3-4559-9cd3-4ce624452b1f) 语言的[高性能](https://www.uber.com/blog/tech-stack-part-one-foundation/?uclick_id=6f537554-73b3-4559-9cd3-4ce624452b1f)，广泛采用其作为实现后端服务和库的主要编程语言。Uber 的 [Go monorepo](https://www.uber.com/blog/go-monorepo-bazel/?uclick_id=6f537554-73b3-4559-9cd3-4ce624452b1f) 是 Uber 最大的代码库，包含 9000 万行代码（并且还在增长）。这使得编写可靠 Go 代码的工具成为我们开发基础设施的重要组成部分。

[指针](https://www.golang-book.com/books/intro/8?uclick_id=6f537554-73b3-4559-9cd3-4ce624452b1f)（保存其他变量的内存地址而不是其实际值的变量）是 Go 编程语言的一个重要组成部分，有助于高效的内存管理和有效的数据操作。因此，程序员在编写 Go 程序时广泛使用指针，出于多种目的，如原地数据修改、并发编程、数据共享优化、内存使用优化以及支持接口和多态性。虽然指针功能强大且被广泛使用，但必须谨慎和明智地使用它们，以避免诸如空指针解引用导致的 nil panic 等常见陷阱。

## nil panic 问题

nil panic 是指程序尝试解引用一个 nil 指针时发生的运行时 panic。当一个指针为 nil 时，意味着它不指向任何有效的内存地址，尝试访问它指向的值将导致 panic（即运行时错误），错误信息如图 1 所示。

![](https://blog.uber-cdn.com/cdn-cgi/image/width=2048,quality=80,onerror=redirect,format=auto/wp-content/uploads/2023/11/figure_1.jpg)

<p align="center">图 1：Nil panic 报错</p>

图 2 显示了在实现 Go 标准库（特别是 *net* 包）中发现并解决的最近一次 [nil panic 问题](https://github.com/golang/go/pull/60823?uclick_id=6f537554-73b3-4559-9cd3-4ce624452b1f) 的示例。由于在第 1859 行直接调用了方法 `RemoteAddr()` 的返回值上的 `String()` 方法，假设它总是非 nil 的，如图2所示，从而引发了 panic。当接口类型 `net.Conn` 的字段 `c.rwc` 被分配给结构 `net.conn` 时导致了这个问题，因为如果发现连接 c 异常的话，它的 `RemoteAddr()` 的具体实现可以返回 nil 值（如图 3 所示）。具体来说，`RemoteAddr()` 可以在 L225 返回一个 [nil 接口值](https://go.dev/tour/methods/13?uclick_id=6f537554-73b3-4559-9cd3-4ce624452b1f#:~:text=A%20nil%20interface%20value%20holds,which%20concrete%20method%20to%20call.)，当被调用方法（`.String()`）时，由于 `nil` 值不包含任何指向可以调用的具体方法的指针，从而导致 nil panic。

![Figure 2: Fix commit from golang/go fixing a nil panic in its net package (PR #60823). The nil panic is caused by calling method String() on the return of RemoteAddr() on L1859, which can be a nil interface value (as shown in Figure 3)](https://blog.uber-cdn.com/cdn-cgi/image/width=2048,quality=80,onerror=redirect,format=auto/wp-content/uploads/2023/11/figure_2.jpeg)

<p align="center">图2：来自 <i>golang/go</i> 的修复提交，修复了其 <i>net</i> 包中的 nil panic（<a href="https://github.com/golang/go/pull/60823?uclick_id=620f1ca1-e871-4193-9f25-2bc76417cfa7">PR #60823</a>）。nil panic 是由于在 L1859 调用 <i>RemoteAddr()</i> 的返回值的 <i>String()</i> 方法引起的，该返回值可能是一个 nil 接口值（如图3所示）。</p>

![Figure 3: Excerpt from net/net.go showing the net.Conn interface and the implementation of the RemoteAddr() method by the struct net.conn, which can return nil, if c.ok() is false](https://blog.uber-cdn.com/cdn-cgi/image/width=1877,quality=80,onerror=redirect,format=auto/wp-content/uploads/2023/11/figure_3.jpg)
<p align="center">图3：<i>net/net.go</i> 的摘录，展示了 <i>net.Conn</i> 接口以及结构体 <i>net.conn</i> 对 <i>RemoteAddr()</i> 方法的实现，如果 <i>c.ok()</i> 为 false，则可以返回 nil。</p>

在 Go 程序中，nil panic 是一种特别[普遍](https://github.com/search?q=repo%3Agolang%2Fgo+nil+panic&type=issues&uclick_id=6f537554-73b3-4559-9cd3-4ce624452b1f)的运行时错误。Uber 的 Go monorepo 也不例外，因为 nil panic 在生产中出现了几次运行时错误，导致程序错误以及应用程序中断，影响了 Uber 的客户。因此，为了最大限度地提高可靠性和代码质量，Uber 需要确保程序员能够在有问题的代码部署到生产环境之前，尽早检测和修复 nil panic。

nil panic 还可能导致拒绝服务攻击。例如，[CVE-2020-29652](https://www.cvedetails.com/cve/CVE-2020-29652/?uclick_id=620f1ca1-e871-4193-9f25-2bc76417cfa7) 就是由于在 [golang.org/x/crypto/ssh](https://cs.opensource.google/go/x/crypto?uclick_id=620f1ca1-e871-4193-9f25-2bc76417cfa7) 中的 nil 指针解引用，允许远程攻击者对 SSH 服务器发动拒绝服务攻击。

存在一个名为 [nilness](https://pkg.go.dev/golang.org/x/tools/go/analysis/passes/nilness?uclick_id=620f1ca1-e871-4193-9f25-2bc76417cfa7) 的自动化工具，由 Go 发行版提供，用于检测 nil panic。这个 nilness 检查器是一种轻量级静态分析技术，仅报告简单错误，例如明显的 nil 解引用位置（例如，*if x == nil { print(*x) }*）。然而，这种简单的检查无法捕捉到真实程序中复杂的 nil 流，如图2所示。因此，我们需要一种能够进行严格分析并在生产代码上有效的技术。

为了处理 Java 中的空指针异常（NPE），Uber开发了 [NullAway](https://www.uber.com/blog/nullaway/?uclick_id=620f1ca1-e871-4193-9f25-2bc76417cfa7)。NullAwa 要求代码使用 `@Nullable` 注解进行标注，以保证在编译时不出现 NPE。这限制了我们直接改编类似 `NullAway` 技术的可行性，因为与 Java 不同，Go 语言并不支持注解。此外，为大型代码库（例如，Uber 的 Go 单体库，包含 9000 万行代码）添加注解是一项繁琐的任务。此外，Go语言的各种独特特性和特有习惯也带来了独特的挑战。

我们克服这些限制的答案是？**NilAway**。

我们设计并开发了 NilAway，通过采用复杂的跨过程静态分析和推断技术，自动检测 nil panic。NilAway 的设计目标是使开发者没有注解负担，尽量减少对本地和持续集成构建时间的影响，并以对 Go 开发者自然的方式解决 Go 语言习惯带来的诸多挑战。

## NilAway 的核心理念

我们的主要想法是，代码中的 nilability 流可以建模为一个全局类型约束系统，然后可以使用 [2-SAT](https://en.wikipedia.org/wiki/2-satisfiability?uclick_id=620f1ca1-e871-4193-9f25-2bc76417cfa7) 算法来解决潜在的矛盾。在高层次上，我们捕捉到各种程序位置的可空和非空约束，包括结构体字段、函数参数和返回值。可空约束的一个例子是返回x，其中x是一个未初始化的指针，而解引用 **x* 是非空约束的一个例子。接着，我们构建一个全局[蕴含图](https://en.wikipedia.org/wiki/Implication_graph?uclick_id=620f1ca1-e871-4193-9f25-2bc76417cfa7)，建模这些特定于程序位置的约束。最后，我们遍历蕴含图 —— 向前传播已知的可空值，向后传播已知的非空值 —— 以寻找矛盾。对于某个位置 *S*，如果在蕴含图的程序路径中发现矛盾 *nilable(S) ^ nonnil(S)*，那么这意味着可空值从一个可空源流向位置 S，并从那里到达解引用点，这可能导致 nil panic。NilAway 收集并报告这些矛盾作为潜在的 nil panic 给开发者。

![Figure 4: Excerpt from the implication graph built by NilAway representing the nil flow for the example in Figure 2.](https://blog.uber-cdn.com/cdn-cgi/image/width=2048,quality=80,onerror=redirect,format=auto/wp-content/uploads/2023/11/figure_4.jpg)
<p align="center">图4：NilAway构建的蕴含图摘录，表示图2中示例的 nil 流。</p>

图4 展示了 NilAway 为图2 中示例构建的 nil 流的推理图路径。在这里，节点是可能为 nilable 类型的程序位置，边则是它们之间的 nil 流。NilAway 遍历推理图以查找不安全的流，并将其建模为矛盾。如果发现一个被证实的nil值通过不同的程序路径流向一个期望该值为非nil的目的地，则该流被视为不安全，例如在nil值从具体实现net.conn.RemoteAddr()流向其通过接口声明net.Conn.RemoteAddr()的方法调用的解引用的情况下。NilAway为这个 nil panic 报告了详细的错误信息（如图5所示），使开发者能够轻松调试从证据证明的 nilability 到其解引用的确切 nil 流，并应用必要的修复以防止 nil panic。

![Figure 5: Error message reported by NilAway for the unsafe flow in Figure 2](https://blog.uber-cdn.com/cdn-cgi/image/width=2048,quality=80,onerror=redirect,format=auto/wp-content/uploads/2023/11/figure_5.jpg)

<p>图5：NilAway 报告的图2 中不安全流的错误信息</p>

请注意，一般对于实际的静态类型系统，无论是否进行全局类型推断，总会存在一些不满足有效静态类型的无错误程序。在 NilAway 的情况下，请注意上述算法未能捕捉到程序执行中的微妙跨过程不变性会在运行时阻止 nil 到非 nil 流。例如，在图3 中，某些共享程序状态可能被设置为每当从 *conn.RemoteAddr()* 调用 *c.ok()* 时，它总是返回 *true*，在这种情况下，该代码中不存在 nil panic。然而，实际上，NilAway 的误报率较低，而那些复杂的执行不变性本质上阻止推断适当的 nil 性约束的情况往往与可能的代码异味相关。

## NilAway 的设计和实现

我们围绕以下四个关键要求设计和开发了 NilAway，使其成为一个适用于 Uber 规模的实用工具：

1. 低延迟：NilAway在对大型Go代码库进行分析时应仅产生低开销。我们希望 NilAway 在开发者引入潜在的 nil panic 时能够立即反馈，因此 NilAway 需要在开发流水线的每个阶段都能快速运行，甚至在本地构建过程中也要保持低延迟。高开销意味着更高的延迟（反馈延迟），从而降低开发者的生产力。
2. 高效性：NilAway 应具有低误报率；检查误报的 nil panic 会浪费开发者的时间。
3. 完全自动化：NilAway 应完全自动化，无需开发者提供额外输入（例如，像 NullAway 那样的注释或人为的编码模式）。
4. 针对 Go 的特性量身定制：NilAway 应将 Go 中的特性视为一等公民，并设计一个专门针对 Go 的系统。

![Figure 6: Architecture of NilAway.](https://blog.uber-cdn.com/cdn-cgi/image/width=2048,quality=80,onerror=redirect,format=auto/wp-content/uploads/2023/11/figure_6.jpg)
<p align="center">图6：NilAway 的架构</p> 

NilAway 是用 Go 实现的，并使用 [go/analysis](https://pkg.go.dev/golang.org/x/tools/go/analysis?uclick_id=620f1ca1-e871-4193-9f25-2bc76417cfa7) 框架进行代码分析。图 6 显示了 NilAway 的架构概述。NilAway 以标准 Go 代码作为输入，输入形式为包含代码的目标包路径，并通过分析返回它识别的潜在 nil panic 错误。NilAway 被实现为一个分析器，可以作为独立工具使用，或者可选地，轻松集成到构建系统中，例如 [Bazel](https://bazel.build/?uclick_id=620f1ca1-e871-4193-9f25-2bc76417cfa7)，配合现有的分析器驱动程序，如 [nogo](https://github.com/bazelbuild/rules_go/blob/master/go/nogo.rst?uclick_id=620f1ca1-e871-4193-9f25-2bc76417cfa7)。

总体而言，NilAway 的实现可以分为三个组件：分析引擎、推理引擎和错误引擎。**分析引擎**负责独立识别函数内的所有潜在 nil 流（即过程内），而**推理引擎**负责收集不同程序位置的 nil 值，并通过构建推理图在过程间流动中传播这些信息。最后，**错误引擎**汇总来自分析引擎和推理引擎的信息，并将每个潜在的 nil 流（过程内和过程间）标记为安全或不安全。不安全的 nil 流随后被报告给用户，作为潜在的 nil panic 错误。

凭借新颖的基于约束的方法来检测 nil panic，NilAway 恰当地满足了上述四个要求：

* **NilAway 速度快**。分析引擎对每个函数的独立分析使其适合并行化，这是一个主要的性能提升因素。此外，我们设计了 NilAway，通过利用构建缓存逐步构建全局推理图，避免了对依赖项的昂贵重建。这种精心的工程设计使 NilAway 快速且可扩展，适合大型代码库。在我们在 Uber 的测量中，我们观察到 NilAway 仅为正常构建过程增加了很小的开销（不到 5%）。
* **NilAway 实用**。为了保持 NilAway 的精确性，分析引擎的设计和实现支持许多常见的 Go 语言特性。我们的错误引擎也经过精心设计，仅在有不安全的 nil 流证据时报告错误。尽管如此，我们并不声称我们的方法是完全可靠或完整的，而是将实际的错误发现作为我们的指导方针。NilAway 可能会产生假阳性和假阴性。然而，我们正在不断努力减少这些情况，使 NilAway 更加精确。NilAway 在 Uber 部署时表现良好（如后文所述），能够捕捉到新代码中的大多数潜在 nil panic，从而使 NilAway 在实用性和性能开销之间保持良好的平衡。
* **NilAway 完全自动化**。我们的基于约束的方法使其自然适合推理，这使得 NilAway 能够在完全自动化模式下运行，无需任何注释。

## 在 Uber 使用 NilAway

NilAway 在 Go 单体库中集中部署，与 Bazel+Nogo 框架紧密集成，允许其作为 CI 流水线和本地构建中的默认代码检查工具运行。然而，错误报告目前处于测试阶段，仅对已接入 NilAway 的 Go 单体库中的服务报告 nil panic 错误。

对于服务拥有者，我们目前提供两种错误报告选项：(1) 全面且阻塞，(2) 停止出血且非阻塞。

在第一种选项中，如果发现任何错误，NilAway 会导致构建失败（如有需要，可以通过 //nolint:nilaway 进行抑制）。NilAway 会全面报告所有代码的错误，包括现有代码和新代码。该选项更可确保代码库中没有 nil panic。然而，它要求在服务代码中报告的所有 nil panic 必须得到解决，才能允许任何构建通过。这可能会给服务的开发带来较高的前期成本，从而导致服务拥有者之间的摩擦。

为了解决上述问题，我们提供了第二种轻量级版本，在该版本中，我们只对服务中更改的代码报告 NilAway 错误。这些错误会在每个已接入服务的差异代码修订（即拉取请求）中以非阻塞方式直接报告。这种停止出血的方法有助于防止新的 nil panic 被引入到服务代码中，同时允许团队逐步解决现有代码中的 nil panic，而不需要耗时的前期接入工作。

我们已在 Uber 的多个服务中接入了 NilAway，涵盖了这两种选项，团队反馈总体积极。一位满意的用户表示：“NilAway 帮助他们的团队及早发现问题，防止了部署回滚。”另一位用户则表示：“NilAway 留下的评论非常可操作，并没有造成任何噪音。”用户们也积极报告他们可能遇到的误报，并建议可用性改进，我们正在积极进行改进。

### Impactful Example

### 影响深远的例子

我们现在讨论一个有趣的案例，NilAway 报告了一个重要错误，该服务在生产代码中每天记录超过 3,000 次 nil panic。图 7 显示了导致 nil panic 的代码的简化和编辑摘录。这个例子使用了 Go 语言中的消息传递构造 —— [通道](https://go.dev/tour/concurrency/2?uclick_id=620f1ca1-e871-4193-9f25-2bc76417cfa7)。在 L16 行，函数调用 *t.s.foo(…)* 返回一个通道 *ch*，随后被变量 *a* 接收。不幸的是，Go 语言允许从关闭的通道读取，在这种情况下将返回一个零值（即 nil）。如果在函数 foo 中执行代码路径 L7->L8->L5，通道将被关闭而没有任何写入。这将在 L17 行的解引用点 *a.Items[*id]* 处导致 nil panic。NilAway 正确地报告了这个错误，因为它观察到了可能从关闭通道接收的变量上的不安全解引用。

![Figure 7: Simplified and redacted code excerpt from an internal Uber service logging 3000+ nil panics per day in production.](https://blog.uber-cdn.com/cdn-cgi/image/width=1955,quality=80,onerror=redirect,format=auto/wp-content/uploads/2023/11/figure_7.jpg)

<p align="center">图7：来自内部Uber服务的简化和删减代码摘录，显示在生产环境中每天记录超过3000个nil panic。</p>

The fix for this problem is to properly guard the receive from a closed channel, either using the ok construct of Go (e.g., if a, ok := <-t.s.foo(…); ok { … }) or by a nilness check on the result variable a (e.g., if a != nil { … }) before the dereference on L17. Our developers applied the nilness check fix right after NilAway reported this error, and the impact was remarkable: the service went from logging 3,000+ nil panics daily to 0, as shown in Figure 8.

解决此问题的方法是正确地防止从关闭的通道接收数据，可以使用 Go 的 ok 结构（例如，*if a, ok := <-t.s.foo(…); ok { … }*）或在结果变量 a 上进行 nil 检查（例如，在 L17 解引用之前，*if a != nil { … }*）。我们的开发人员在 NilAway 报告此错误后立即应用了 nil 检查修复，效果显著：服务的每日 nil panic 日志从 3000+ 降至 0，如图 8 所示。

![Figure 8: Complete addressal of the 3000+ nil panics being logged per day in production.](https://blog.uber-cdn.com/cdn-cgi/image/width=1252,quality=80,onerror=redirect,format=auto/wp-content/uploads/2023/11/figure_8.jpg)

<p align="center">图8：在生产环境中，全面处理每天记录的3000多个 nil panic</p>

## 在你的代码中使用 NilAway

我们很高兴地宣布，NilAway 现已开源，地址为 https://github.com/uber-go/nilaway/。我们相信，NilAway 对于任何使用 Go 语言编写代码并希望确保代码库无 nil panic 的个人或团队都将非常有用。

设置 NilAway 相对简单。它可以作为独立的检查工具使用，也可以与现有的驱动程序集成。有关更多详细信息，请参阅 [README](https://github.com/uber-go/nilaway/?uclick_id=620f1ca1-e871-4193-9f25-2bc76417cfa7) 和 [wiki](https://github.com/uber-go/nilaway/wiki?uclick_id=620f1ca1-e871-4193-9f25-2bc76417cfa7)。

今天就试用 NilAway，并告诉我们您的使用体验。我们也欢迎社区的贡献。

## 致谢

NilAway 最初是 Joshua Turcotti（Uber intern ’22）的实习项目，并得到了以下 Uber 博士实习生的重大贡献：Shubham Ugare, Narges Shadab, and Zhiqiang Zang。我们还要感谢 Uber 的 Go 单体仓库团队与我们合作开发 NilAway，特别感谢 Dmitriy Shirchenko。