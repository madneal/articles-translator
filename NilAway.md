# Go 的漏洞管理

>原文：[NilAway: Practical Nil Panic Detection for Go](https://www.uber.com/en-NL/blog/nilaway-practical-nil-panic-detection-for-go/)
>
>译者：[madneal](https://github.com/madneal)
>
>welcome to star my [articles-translator](https://github.com/madneal/articles-translator/), providing you advanced articles translation. Any suggestion, please issue or contact [me](mailto:bing@stu.ecnu.edu.cn)
>
>LICENSE: [MIT](https://opensource.org/licenses/MIT)

![](https://blog.uber-cdn.com/cdn-cgi/image/width=2048,quality=80,onerror=redirect,format=auto/wp-content/uploads/2023/11/cover_photo.jpg)

Uber 由于 [Go](https://go.dev/?uclick_id=6f537554-73b3-4559-9cd3-4ce624452b1f) 语言的[高性能](https://www.uber.com/blog/tech-stack-part-one-foundation/?uclick_id=6f537554-73b3-4559-9cd3-4ce624452b1f)，广泛采用其作为实现后端服务和库的主要编程语言。Uber 的 [Go monorepo](https://www.uber.com/blog/go-monorepo-bazel/?uclick_id=6f537554-73b3-4559-9cd3-4ce624452b1f) 是 Uber 最大的代码库，包含 9000 万行代码（并且还在增长）。这使得编写可靠 Go 代码的工具成为我们开发基础设施的重要组成部分。

[指针](https://www.golang-book.com/books/intro/8?uclick_id=6f537554-73b3-4559-9cd3-4ce624452b1f)（保存其他变量的内存地址而不是其实际值的变量）是 Go 编程语言的一个重要组成部分，有助于高效的内存管理和有效的数据操作。因此，程序员在编写 Go 程序时广泛使用指针，出于多种目的，如原地数据修改、并发编程、数据共享优化、内存使用优化以及支持接口和多态性。虽然指针功能强大且被广泛使用，但必须谨慎和明智地使用它们，以避免诸如空指针解引用导致的 nil panic等常见陷阱。

## nil panic 问题

A nil panic is a runtime panic that occurs when a program attempts to dereference a nil pointer. When a pointer is nil, it means that it does not point to any valid memory address, and attempting to access the value it points to will result in a panic (i.e., a runtime error) with the error message shown in Figure 1.

nil panic 是指程序尝试解引用一个 nil 指针时发生的运行时 panic。当一个指针为 nil 时，意味着它不指向任何有效的内存地址，尝试访问它指向的值将导致 panic（即运行时错误），错误信息如图 1 所示。

![](https://blog.uber-cdn.com/cdn-cgi/image/width=2048,quality=80,onerror=redirect,format=auto/wp-content/uploads/2023/11/figure_1.jpg)


图 2 显示了在实现 Go 标准库（特别是 net 包）中发现并解决的最近一次 [nil panic 问题](https://github.com/golang/go/pull/60823?uclick_id=6f537554-73b3-4559-9cd3-4ce624452b1f) 的示例。由于在第 1859 行直接调用了方法 `RemoteAddr()` 的返回值上的 `String()` 方法，假设它总是非 nil 的，如图2所示，从而引发了 panic。当接口类型 `net.Conn` 的字段 `c.rwc` 被分配给结构 `net.conn` 时导致了这个问题，因为如果发现连接 c 异常的话，它的 `RemoteAddr()` 的具体实现可以返回 nil 值（如图 3 所示）。具体来说，`RemoteAddr()` 可以在 L225 返回一个 [nil 接口值](https://go.dev/tour/methods/13?uclick_id=6f537554-73b3-4559-9cd3-4ce624452b1f#:~:text=A%20nil%20interface%20value%20holds,which%20concrete%20method%20to%20call.)，当被调用方法（`.String()`）时，由于 `nil` 值不包含任何指向可以调用的具体方法的指针，从而导致 nil panic。

![Figure 2: Fix commit from golang/go fixing a nil panic in its net package (PR #60823). The nil panic is caused by calling method String() on the return of RemoteAddr() on L1859, which can be a nil interface value (as shown in Figure 3)](https://blog.uber-cdn.com/cdn-cgi/image/width=2048,quality=80,onerror=redirect,format=auto/wp-content/uploads/2023/11/figure_2.jpeg)

![Figure 3: Excerpt from net/net.go showing the net.Conn interface and the implementation of the RemoteAddr() method by the struct net.conn, which can return nil, if c.ok() is false](https://blog.uber-cdn.com/cdn-cgi/image/width=1877,quality=80,onerror=redirect,format=auto/wp-content/uploads/2023/11/figure_3.jpg)

Nil panics are found to be an especially pervasive form of runtime errors in Go programs. Uber’s Go monorepo is no exception to this, and has witnessed several runtime errors in production because of nil panics, with effects ranging from incorrect program behavior to app outages, affecting Uber customers. Therefore, in order to maximize reliability and code quality, it is crucial for Uber to enable programmers to detect and fix nil panics early, before the buggy code gets deployed in production.

Nil panics can also cause denial of service attacks. For example, CVE-2020-29652 is due to a nil pointer dereference in the golang.org/x/crypto/ssh that allows remote attackers to cause a denial of service against SSH servers.


在 Go 程序中，nil panic 是一种特别[普遍](https://github.com/search?q=repo%3Agolang%2Fgo+nil+panic&type=issues&uclick_id=6f537554-73b3-4559-9cd3-4ce624452b1f)的运行时错误。Uber 的 Go monorepo 也不例外，因为 nil panic 在生产中出现了几次运行时错误，导致程序错误以及应用程序中断，影响了 Uber 的客户。因此，为了最大限度地提高可靠性和代码质量，Uber 需要确保程序员能够在有问题的代码部署到生产环境之前，尽早检测和修复 nil panic。

nil panic 还可能导致拒绝服务攻击。例如，CVE-2020-29652 就是由于在 golang.org/x/crypto/ssh 中的 nil 指针解引用，允许远程攻击者对 SSH 服务器发动拒绝服务攻击。

There exists an automated tool, nilness, offered by the Go distribution for detecting nil panics. This nilness checker is a lightweight static analysis technique that reports only simple errors, such as obvious sites of nil dereferences (e.g., if x == nil { print(*x) }). However, such simple checks fail to capture the complex nil flows in real programs, such as the one shown in Figure 2. Therefore, we need a technique that performs rigorous analysis and is effective on production code.

To deal with NullPointerExceptions (NPEs) in Java, Uber has developed NullAway. NullAway requires the code to be annotated with @Nullable annotations to guarantee NPE freedom during compile time. This limits the feasibility of directly adapting a NullAway-like technique for our purpose, since, unlike Java, Go does not have language support for annotations. Moreover, annotating a large codebase (e.g., Uber’s Go monorepo with 90 million lines of code) is a cumbersome task. Besides, Go’s various unique features and idiosyncrasies present their own unique challenges.

存在一个名为 nilness 的自动化工具，由 Go 发行版提供，用于检测 nil panic。这个 nilness 检查器是一种轻量级静态分析技术，仅报告简单错误，例如明显的nil解引用位置（例如，如果x == nil { print(*x) }）。然而，这种简单的检查无法捕捉到真实程序中复杂的nil流，如图2所示。因此，我们需要一种能够进行严格分析并在生产代码上有效的技术。

为了处理Java中的空指针异常（NPE），Uber开发了NullAway。NullAway要求代码使用@Nullable注解进行标注，以保证在编译时不出现NPE。这限制了我们直接改编类似NullAway技术的可行性，因为与Java不同，Go语言并不支持注解。此外，为大型代码库（例如，Uber的Go单体库，包含9000万行代码）添加注解是一项繁琐的任务。此外，Go语言的各种独特特性和特有习惯也带来了独特的

Our answer to overcome these limitations? NilAway.

We designed and developed NilAway for automatically detecting nil panics by employing sophisticated interprocedural static analysis and inferencing techniques. The design goal of NilAway was to have no annotation burden on developers, maintain minimal impact on local and CI build-times, and address the many challenges posed by Go language idioms in ways that are natural to Go developers.

## Core Idea of NilAway

Our main idea is that nilability flows in code can be modeled as a system of global typing constraints, which can then be solved using a 2-SAT algorithm to determine potential contradictions. At a high level, we capture both nilable and nonnil constraints at various program sites for struct fields, function parameters, and return values. An example of a nilable constraint is return x, where x is an uninitialized pointer, while the dereference, *x, is an example of a nonnil constraint. We then build a global implication graph modeling these program site-specific constraints. Finally, we traverse the implication graph – forward propagating known nilness values and backward propagating known nonnil values – to find contradictions. For a site, S, if a contradiction nilable(S) ^ nonnil(S) is discovered in a program path of the implication graph, then it implies that a nil value is witnessed to flow from a nil source to the site S, from where it reaches a dereference point, which can likely cause a nil panic. NilAway collects and reports these contradictions as potential nil panics to the developer.

![Figure 4: Excerpt from the implication graph built by NilAway representing the nil flow for the example in Figure 2.](https://blog.uber-cdn.com/cdn-cgi/image/width=2048,quality=80,onerror=redirect,format=auto/wp-content/uploads/2023/11/figure_4.jpg)

Figure 4 shows the path through the implication graph built by NilAway for the nil flow for the example presented in Figure 2. Here the nodes are program sites that could be a nilable type and edges are the nil flows between them. NilAway traverses the implication graph to find unsafe flows modeling them as contradictions. A flow is deemed unsafe if a witnessed nil value is found to flow through different program paths to a destination where that same value is expected to be nonnil, such as in the case of the nil value flowing from the concrete implementation net.conn.RemoteAddr() to its dereference via method invocation on interface declaration net.Conn.RemoteAddr(). NilAway reports a detailed error message for this nil panic (as shown in Figure 5) that allows developers to easily debug through the exact nil flow from evidenced nilability to its dereference, and apply the necessary fix to prevent the nil panic.

![Figure 5: Error message reported by NilAway for the unsafe flow in Figure 2](https://blog.uber-cdn.com/cdn-cgi/image/width=2048,quality=80,onerror=redirect,format=auto/wp-content/uploads/2023/11/figure_5.jpg)

Note that, in general, for practical static type systems, with or without global inference of types, there will always exist error-free programs that do not satisfy a valid static typing. In the case of NilAway, note that the above algorithm doesn’t capture cases where subtle inter-procedural invariants in the execution of the program would prevent the nil to nonnil flow from happening at runtime. For example, in Figure 3, it is possible that some shared program state is set up such that whenever c.ok() is called from conn.RemoteAddr(), it always returns true, in which case no nil panic exists in that code. However, in practice, NilAway’s false positive rate is low and the cases where such complex execution invariants inherently prevent inferring proper nilness constraints tend to be associated with likely code smells.

## Design and Implementation of NilAway

We designed and developed NilAway around the following four key requirements to make it a practical tool for Uber scale:

1. Low latency: NilAway should incur only a low overhead in performing its analysis on the large Go codebase. We want NilAway to give developers immediate feedback when they introduce a potential nil panic, thereby requiring NilAway to be fast enough to run with low latency at every stage of our development pipeline, even during local builds. A high overhead would mean higher latency (delayed feedback), thereby reducing developer productivity.
2. High effectiveness: NilAway should have a low false positive rate; inspecting false positive nil panics wastes developer time.
3. Fully automated: NilAway should be fully automated, requiring no additional input from developers (e.g., annotations as in NullAway, or contrived coding patterns).
4. Tailored to Go’s idiosyncrasies: NilAway should treat the idiosyncrasies in Go as first-class citizens and devise a system tailored to Go.

![Figure 6: Architecture of NilAway.](https://blog.uber-cdn.com/cdn-cgi/image/width=2048,quality=80,onerror=redirect,format=auto/wp-content/uploads/2023/11/figure_6.jpg)

NilAway is implemented in Go and uses the go/analysis framework for the analysis of code. Figure 6 shows an overview of NilAway’s architecture. NilAway takes as input standard Go code, in the form of a target package path containing the code, and returns as output the potential nil panic errors that it identifies through its analysis. NilAway is implemented as an analyzer that can be used as an independent tool or, optionally, can also be easily integrated into a build system, such as Bazel, with existing analyzer drivers, such as nogo.

Broadly, the implementation of NilAway can be divided into 3 components: the Analyzer Engine, the Inference Engine, and the Error Engine. The Analyzer Engine is responsible for identifying all potential nil flows within a function independently (i.e., intra-procedurally), while the Inference Engine is responsible for collecting witnessed nilability values for different program sites and propagating this information through inter-procedural flows by building the implication graph. Finally, the Error Engine accumulates the information from both the Analyzer Engine and the Inference Engine, and marks each potential nil flow (intra- and inter-procedural) as safe or unsafe. Unsafe nil flows are then reported to the user as potential nil panic errors.

Powered with the novel constraint-based approach to detect nil panics, NilAway aptly satisfies the four requirements listed above:

* NilAway is fast. Independent analysis of each function in the Analyzer Engine makes it amenable to parallelization, which is a major performance enhancer. Furthermore, we have designed NilAway to construct the global implication graph incrementally by leveraging build cache, avoiding expensive re-building of the dependencies. This careful engineering makes NilAway fast and scalable, making it suitable for large codebases. In our measurements at Uber, we have observed that NilAway added only a small overhead (less than 5%) to the normal build process.
* NilAway is practical. To keep NilAway precise, the Analyzer Engine is designed and implemented to support many common Go language idiosyncrasies. Our Error Engine is also carefully designed to only report errors when an unsafe nil flow is evidenced. Having said that, we don’t claim our approach to be either sound or complete, instead having practical bug finding as our northstar. NilAway may incur both false positives and false negatives. However, we are continuously striving hard to reduce them and make NilAway precise. NilAway has been observed to work well in practice when deployed at Uber (as discussed subsequently), catching most of the potential nil panics in new code, allowing NilAway to maintain a good balance between usefulness and performance overhead.
* NilAway is fully automated. Our constraint-based approach makes it a natural fit for inference, which allows NilAway to operate in a fully automated mode with no annotations required.

## Using NilAway at Uber

NilAway is deployed centrally in the Go monorepo, integrating tightly with the Bazel+Nogo framework, allowing it to run as a default linter on every build in the CI pipeline and local builds. The error reporting is, however, in the testing phase, where nil panic errors are only reported for services in the Go monorepo that are onboarded onto NilAway. 

For service owners, we currently offer two options of error reporting: (1) comprehensive and blocking, and (2) stop-the-bleed and non-blocking. 

In the first option, NilAway causes the build to fail, if any errors are found (suppressions are possible if needed, through //nolint:nilaway). NilAway comprehensively reports errors on all code, existing and new. This option is preferable to ensure a nil panic free codebase. However, it requires all reported nil panics in the service’s code to be addressed, before any build can be allowed to pass. This may incur a high upfront cost for the service’s development, which can cause friction among service owners.

To address the above problem, we offer a lightweight version in Option 2, in which we only report NilAway errors for changed code in the service. These errors are directly reported in a non-blocking way on every differential code revision (i.e., a pull request) of the onboarded service. This stop-the-bleed approach helps to prevent new nil panics from being introduced into the service code, while allowing teams to gradually address nil panics in existing code without the need for a development-slowing upfront onboarding effort.

We have onboarded several services at Uber onto NilAway, across both the options, and the overall feedback that we have received from the teams has been positive. One such happy user says “NilAway has helped their team catch issues early, preventing deployment rollbacks,” while another says “The comments left by NilAway are very actionable and it hasn’t caused any noise.” The users also actively report false positives that they may encounter and suggest usability improvements that we actively work upon.

### Impactful Example

We now discuss one interesting case, where NilAway reported an important error in a service that was logging over 3,000 nil panics per day in production code. Figure 7 shows a simplified and redacted excerpt of the code causing the nil panic. This example uses the message passing construct of Go called channel. On line L16, the function call to t.s.foo(…) returns a channel ch which is subsequently received by the variable a. Unfortunately, Go allows reading from a closed channel, in which case a zero-value (i.e., nil) would be returned. If the code path L7->L8->L5 is taken in the function foo, the channel would be closed without anything written to it. This will cause a nil panic at the dereference point a.Items[*id] on line L17. NilAway correctly reported this error since it witnessed an unsafe dereference on the variable that may be received from a closed channel.

![Figure 7: Simplified and redacted code excerpt from an internal Uber service logging 3000+ nil panics per day in production.](https://blog.uber-cdn.com/cdn-cgi/image/width=1955,quality=80,onerror=redirect,format=auto/wp-content/uploads/2023/11/figure_7.jpg)

The fix for this problem is to properly guard the receive from a closed channel, either using the ok construct of Go (e.g., if a, ok := <-t.s.foo(…); ok { … }) or by a nilness check on the result variable a (e.g., if a != nil { … }) before the dereference on L17. Our developers applied the nilness check fix right after NilAway reported this error, and the impact was remarkable: the service went from logging 3,000+ nil panics daily to 0, as shown in Figure 8.

![Figure 8: Complete addressal of the 3000+ nil panics being logged per day in production.](https://blog.uber-cdn.com/cdn-cgi/image/width=1252,quality=80,onerror=redirect,format=auto/wp-content/uploads/2023/11/figure_8.jpg)

## Using NilAway for Your Code

We are happy to announce that NilAway is now open source at https://github.com/uber-go/nilaway/. We believe NilAway will be useful for any individual or team that implements code in Go and wants to ensure a nil-panic-free codebase.

Setting up NilAway is fairly straightforward. It can be used as a standalone checker or integrated with existing drivers. Refer to the README and wiki for more details. 

Try NilAway today and let us know your experience. We also welcome contributions from the community.

## Acknowledgements

NilAway began as the internship project of Joshua Turcotti (Uber intern ’22) and benefited from the very significant contributions of the following Uber Ph.D. interns: Shubham Ugare, Narges Shadab, and Zhiqiang Zang. We also would like to thank the Go monorepo team at Uber for collaborating with us in building NilAway, with special thanks to Dmitriy Shirchenko.