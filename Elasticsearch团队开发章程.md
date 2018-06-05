Elasticsearch 团队开发章程

# 前言

We, the team of Elasticsearch core developers, want to move as fast as we can toward a system that is reliable, robust, secure, scalable, and straightforward to use. We want to strive for innovation, replace legacy constructs and features, remove fragile code, and work toward a better user experience while keeping our users onboard with our rapid changes.

我们作为 Elasticsearch 核心开发人员团队希望尽可能快地向可靠，健壮，安全，可扩展且易于使用的系统迁移。 我们希望争取创新，取代传统的构造和功能，删除脆弱的代码，并致力于改善用户体验，同时让我们的用户随着我们的快速变化而加入。

It’s crucial for us to have a shared vision of where the team is heading and maybe even more importantly why the team is going down a certain path. When Elasticsearch was the *new kid on the block* it shone with endless flexibility, ease of use, and rich APIs. We formed a company around that young kid and suddenly its user base shot through the roof. The support organization could barely keep up with the growing number of customers, which is a good problem to have. Yet as the number of users grew, so did the chance of things going sideways, unfortunately much more quickly than we could ever hire support engineers. We learned that much of the flexibility came from leniency, from features that worked in most cases but not all. For instance, having scripts that users can send with the request is basically a remote code execution engine and if it goes wrong it’s fatal. Even the most basic features, like settings, were very flexible but enormously fragile. Specifying a number without a unit was perfectly fine except that many users didn’t know what the default unit was. We just tried to do the right thing, which turned out to not be the right thing all the time. 

对于我们来说，拥有一个团队的前进方向的共同认识是非常重要的，甚至更重要的是为什么团队要走上一条特定的道路。当 Elasticsearch 创立之初时，它具有无尽的灵活性，易用性和丰富的 API。我们在这帮年轻的团队成立了一家公司，并且突然用户数也井喷式发展。支持组织几乎不能跟上越来越多的客户，这是幸福的烦恼。然而，随着用户数量的增长，事情发生的可能性也越来越大，不幸的是，比我们聘用支持工程师的速度要快得多。我们了解到，大多数灵活性来自宽松处理，从大多数情况下可行的功能，但不是全部。例如，用户可以使用请求发送的脚本基本上是一个远程代码执行引擎，如果出错，它是致命的。即使最基本的功能，比如设置，也非常灵活，但非常脆弱。在没有单位的情况下指定一个数字是很好的，除非许多用户不知道默认单位是什么。我们只是试图做正确的事情，结果一直不是正确的事情。

Today we are in a different position. Our user base is much larger than it was in 2013 but our support organization hasn’t grown at the same rate. Yes, we handle an order of magnitude more support cases than in 2013, but this would not have been possible with the system we had back then. Now we’ve moved from a fragile but flexible system toward software that is narrower in scope. We have defined many more boundaries: stricter input validation, a security model that allows us fine grained control over permissions, and even a plugin model that provides great flexibility to add riskier features. 

现在我们处于不同的位置。 我们的用户群比 2013 年的用户群大得多，但我们的支持机构并没有以同样的速度增长。 是的，我们处理比 2013 年更多的支持案例，但这在我们当时的系统中是不可能的。 现在我们已经从一个脆弱而灵活的系统转向了范围较窄的软件。 我们定义了更多的边界：更严格的输入验证，允许我们对权限进行细粒度控制的安全模型，甚至还有一个插件模型，可以提供极大的灵活性来添加风险更高的功能。

But hold on, we are not there yet! There are still endless problems that can have fatal consequences. Aggregations can blow up servers with a single request. Users feel the need to run Elasticsearch with 30+ GB heaps. We are still offering 27 different ways of specifying a boolean value. And this list continues…

We have a massive responsibility to our users, the support organization, the cloud hosting teams, and third party providers to offer a system that is reliable, robust, secure, and straightforward to use. For this reason we all should strive for innovation, replace legacy constructs and features, remove fragile code, and improve the user experience. Our advantage over other companies is our innovation, and innovation requires velocity. We must move and embrace change to innovate without leaving the user behind.

The following sections are a collection of principles and guidelines for designing, refactoring, or removing code from the Elasticsearch codebase. These points are unordered and mostly uncategorized and should be seen as a constitution of software development within the Elasticsearch team.

但等等，我们还差得远呢！仍然有无穷无尽的问题会造成致命的后果。聚合可以通过一个请求来撑爆服务器。用户感觉需要运行 30+ G B堆的 Elasticsearch。我们仍然提供了 27 种指定布尔值的不同方式。这份名单还有其它内容...

我们对我们的用户，支持组织，云托管团队和第三方提供商负有巨大责任，以提供可靠，稳健，安全且易于使用的系统。出于这个原因，我们都应该努力创新，取代传统的构造和功能，删除脆弱的代码，并改善用户体验。我们与其他公司相比的优势是我们的创新，创新需要速度。我们必须在不留下用户的情况下采取行动并接受变革创新。

以下部分是用于设计，重构或从 Elasticsearch 代码库中删除代码的原则和指导原则的集合。这些点是无序的，大部分是未分类的，应该被看作是 Elasticsearch 团队内软件开发的一个组成部分。

## 设计特性

* *Progress over perfection.* We have followed this approach for many years now which allows us to make large changes over time without big bang commits originating from massive pull requests. For example, the completion suggester was added in the early days of Elasticsearch without support for realtime updates and specifically deletes. This means that deleting a document in Elasticsearch wasn’t immediately reflected in the suggestions. It was a hard problem at the time and about three years later we added support for bitset filters to the Lucene suggester as well as to Elasticsearch. Meanwhile it’s been an acceptable solution for many users out there, with many bugs fixed and the evolution toward a document based suggester. It was all about progress over perfection.
* *过程优于结果* 我们多年来一直遵循这种方法，这使我们能够随着时间的推移做出巨大的变化，而不会因大量的请求而产生巨大的响应。 例如，完成建议程序在 Elasticsearch 的早期版本中添加，而不支持实时更新和特定的删除。 这意味着删除 Elasticsearch 中的文档不会立即反映在建议中。 这是一个很难的问题，大约三年后，我们增加了对 Lucene 建议器和 Elasticsearch 的 bitset 过滤器的支持。 与此同时，对于许多用户来说，这是一个可以接受的解决方案，修复了许多错误，并朝着基于文档的建议者发展。 这就是过程优于结果。
* *为今天设计！谨慎使用抽象* Computer Science professors teach students to make extensive use of abstraction layers in the name of flexibility and information hiding. Certainly Elasticsearch makes extensive use of abstractions; no project involving several million lines of code could do otherwise and survive. But experience has shown that excessive or premature abstraction can be just as harmful as premature optimization. Abstraction should be used to the level required and no further.
* 计算机科学教授教育学生以灵活性和信息隐藏的名义广泛使用抽象层。 当然 Elasticsearch 广泛使用抽象; 没有任何涉及数百万行代码的项目可以以其他方式进行工作并生存 但经验表明，过早或过早的抽象可能与过早优化一样有害。 抽象应该用于所需的级别，不要再进一步。

As a simple exercise, consider a function which has an argument which is always passed as zero by all callers. One could retain that argument just in case somebody eventually needs to use the extra flexibility that it provides. By that time, though, chances are good that the code never noticed — because it has never been used.  Or when the need for extra flexibility arises, it does not do so in a way which matches the programmer's early expectation. We should routinely submit patches to remove unused arguments; in general they should not be added in the first place. (adopted from [https://www.kernel.org/doc/Documentation/development-process/4.Coding](https://www.kernel.org/doc/Documentation/development-process/4.Coding))

作为一个简单的练习，假设一个函数，它的参数总是被所有调用者传递为零。 人们可以保留这个观点，以防万一有人最终需要使用它提供的额外的灵活性。 但是那个时候，代码从来没有注意到的机会是好的 - 因为它从未被使用过。 或者当需要额外的灵活性时，它不会以符合程序员早期预期的方式进行。 我们应该定期提交补丁以删除未使用的参数; 一般而言，他们不应该添加在首位。（来源于 https://www.kernel.org/doc/Documentation/development-process/4.Coding）

* *开始简单; 不要聪明*。 每个人都希望编写一流的，前沿的，快速的代码，长时间保持鲁棒性，优雅和高效的代码。 不幸的是，这不会在一夜之间发生。 就像我们在跑步之前学会走路一样。 一项新功能应该以最简单的方式开始。 即使希望防止移动所有分片将索引从8分片缩减到4分片，最好有一个稳固的共享基础结构，它要求所有分片位于同一个节点上。 特殊分配逻辑可以在稍后的版本中发布。
* 注意：删除代码很困难。 即使删除最小的功能也是非常困难的。 将代码添加到代码库并明智地选择添加的内容时请注意这一点。 我们可能需要坚持多年，或者当我们试图将其删除时，将许多用户分散到那里。
* *严格，明确，可靠，简单。* Elasticsearch 有添加宽松，模糊，不可靠和复杂选项的历史。 关于这些在不久前发生了变化。 乍一看，它似乎有助于对用户友好，但是其成本巨大。 它带有一些组合式的选项和代码路径，它们没有经过测试，并且隐藏了错误。 一个完美的例子是指定一个布尔值的 gazillion 选项。 有人会认为这与将该值与字符串“true”或“false”进行比较一样简单，如果不匹配，我们会抛出异常。 不，它接受值“false”，“0”，“no”和“off”，如果它们不匹配，则将其解释为“true”。 什么可能会出错？ 如果你添加代码，请尽可能以最简单，最严格，明确和可靠的方式进行。
* *坚守核心职责* 我们的系统坚持*稳固*和*可靠*特性至关重要。 为此，我们需要牢记我们的核心责任是分布式可扩展搜索引擎。 例如，我们曾经提供一个名为 site-plugins 的有限 Web 服务器，但它并不代表我们的核心职责，所以我们将其删除。 当功能与使 Elasticsearch 成为更好的分布式可扩展搜索引擎相一致时，它变得更加坚实可靠。（同样的原则适用于我们所有的产品。）
* *你是专家; 就表现得像一名专家。* Elasticsearch 已经变得流行起来。 用户群非常庞大，呈指数级增长。 高级用户的一部分正在缩小，因此我们现在的核心职责之一就是简化 API 使用并降低“搬起石头砸自己的脚”的风险。 我们的核心 API 提供了很大的灵活性，这使得它们很容易被误用。其结果往往是性能下降，集群中断和错误结果。展望未来，我们应该利用我们的经验和对系统的深入了解来预防这些缺陷。 构建可以很好地完成一件事的 API 和功能。不要将其设计为解决其他问题的解决方法。
* *独立构建功能* 始终优先将功能添加为插件，而不是将功能添加到内核。制定明确的 API 和扩展的最好方法就是使用它们。为了达到可维护的核心，我们必须保持其精益。 我们的插件模型允许类加载器隔离以及对第三方组件的专用权限。一个孤立的实现总是可取的。如果需要通过发布来传送，它可以是一个模块。
* *先移除再修复。*通常情况下，因为没有替换它的功能，所以移除危险或不便的功能会停止。我们将移除这些功能并优先重新实现它们，就像它们是新功能一样。如果这些功能很重要，我们将会在下一个发布中推迟重新实现。如果不是，它可能会被重新实现，直到它们被f发布。或者它们可能永远不会被重新实现，随着时间的推移而被遗忘。消除危险功能对组织的成功至关重要。例如，*delete-by-query* 反复导致大量中断，需要数天才能进行调试和修复。它的移除可能为我们节省了大量的金钱和时间，而我们没有花费在我们的客户身上。鉴于我们的用户群增长迅速，即使该决定不受欢迎，我们也有责任为用户做出正确的决定。当涉及到安全性，群集稳定性和数据损坏时，*先移除*方法是强制性的。
* *Be fast by default; slow is optional.* Performance is key in our business. Slow is considered esoteric. This is a very difficult topic since for instance O(n) can be acceptable with 10k documents but not for 10m. A perfect example is `*_source`* access in scripts. There are some scripts (search for instance) that should simply not allow the access of `*_source`* since it loads the JSON source of every scored document. That is a round-trip to disk per doc the script is executed for. Features like this must be disabled by default or should not be available on performance critical parts of the system. There is always the argument of prototyping, small documents sets, little webshops etc. Yet our message here should focus on reindexing and future improvements to our defaults to make these APIs obsolete.
* *默认速度很快; 慢是可选的*。 性能是我们业务的关键。 慢被认为是深奥的。 这是一个非常困难的话题，因为例如 O(n)  对于10k 文件来说是可以接受，但不能用于 10m。 一个完美的例子是脚本中的 `_source` 访问。 有些脚本（搜索实例）应该不允许访问 `_source `，因为它会加载每个评分文档的 JSON 源。脚本每次执行都需要访问磁盘。像这样的功能必须默认禁用，或者不应该在系统的性能关键部分提供。 始终存在原型设计，小型文档集，小型网上商店等等的争论。然而，我们在这里的信息应该集中在重新定义和未来改进我们的默认设置，以使这些 API 过时。
* *Focus on upgrade experience.* With time-based release the upgrade experience is crucial for us since we want users to move to new releases ASAP. We had several problems in the past with breaking too much and users suffered from long cluster restarts. Our feature development and ideas for improvement should focus on smoothing the path forward.
* *关注升级体验*。 随着基于时间的发布，升级体验对我们来说至关重要，因为我们希望用户尽快转向新版本。 过去我们遇到了很多问题，导致用户遭遇了长时间的群集重启。 我们的功能开发和改进想法应着重于平滑前进的道路。
* *Break on majors, not on minors.* Breaking changes must be done on major releases. We try to scope the changes to not break too much at the same time especially if we are getting closer to the GA date. It’s OK to push out a breaking change to the next major if we are close. 
* *突破主要的，而不是小的*。 主要版本必须完成重大更改。 我们试图将变化的范围限制在不会同时突破太多，特别是如果我们正在接近 GA 日期。 如果我们接近，可以将突破性改变来推进到下一个大的变更。
* *Test bottom up.* If you write code, write unit tests first. Write many of them. Write code so you can write many of them. Integration testing is the last step. Focus on adding more tests that execute fast and are easy to debug, like unit tests. This is crucial for developer velocity. 
* 自下而上测试。 如果你编写代码，首先编写单元测试。 写很多。 编写单元测试代码，以便可以编写许多代码。 集成测试是最后一步。 专注于添加更多快速执行且易于调试的测试，如单元测试。 这对于开发者的速度来说至关重要。
* **Consider Java APIs expert APIs**. With the exception of the HTTP-Client and its dedicated APIs, all APIs in Elasticsearch, all extension points and plugins are expert APIs. Accordingly, expert users can handle API changes and removals. The most reliable way to make sure folks don’t use deprecated APIs is to remove them. Don’t hesitate, especially when it’s an internal API. Non-expert users should always go through a REST interface. 
* *Be critical, doubt all the code, and embrace mistakes.* Everybody writes code that must be fixed, refactored, or removed at some point. But most of the time the effective half-life of code is pretty short. Add comments describing why things are done in a certain way. We can never know the full extent of a problem or all the use cases when we develop a feature. 

When someone criticizes the code, they are not criticizing you, so don’t take it personally. Help them understand why you wrote it that way. When someone rewrites something you wrote, it’s not a rejection of your ideas. At one point Mike pushed a change to Lucene that Adrien obsoleted just two days later! It’s great when other people take interest in code you wrote: it means the code is alive. See improvements to your code as that code becoming a growing, thriving being.

Don’t fear making a mistake and, more importantly, don’t let fear paralyze you from adding something that might not be totally right. See mistakes and failures as *feedback*, *discovery *and *knowledge* that can make our product better. 

* *Don’t be afraid of big changes.* Often the solution to a problem is hard. The hardest part is to solve it correctly. It comes together with tons of work, risk, and changes in the system that will affect others — mostly users. Prefer incremental changes (see *Progress over perfection*), but be willing to make big changes in big chunks where incremental change is impossible. 

* *Don’t be afraid to say no.* Elasticsearch is at a point where it can’t accept every change. If we tried to make everybody happy we would stall and be paralyzed. There are certain things that just don’t work with a system like Elasticsearch. Think of *[joins](https://github.com/elastic/elasticsearch/pull/3278)* or *real type isolation*. To these we have to say *thanks, but no thanks!*

* *Only accept features that scale.* We often get feature requests that would work fine in the single-node case (e.g., joins or precise cardinality aggregations) but would be a disaster given the distributed nature of Elasticsearch. These feature requests should always be rejected because they violate our core Elasticsearch responsibilities of scalability and distributed nature. In principle we do not add features that only work in the single-node case.

* *Always start with a dream.* It’s wise to start with an idea of what a perfect solution would look like, even if it involves backward breaks or removing core features and replacing them later with better solutions. Sometimes it’s possible to implement the ideal even if it takes time. By the same token, it’s wise to think about the simplest possible solution, and in many cases the *biggest bang for the buck* is pretty close to the simplest solution.

* *Focus on error reporting.* In software development lots of things are binary. If it didn’t work as expected it should fail fast and hard. Focus on good error reporting; avoid swallowing exceptions, declaring checked exceptions, and forcing the caller to handle the situation. Guard preconditions for methods with actual checks. Exceptions are part of a method’s contract! If we don’t know or document what exceptions can happen, and when, then we have no idea really how the method behaves. Add checks/javadocs and try to make this better. Look at the JDK code for examples of this, even Lucene code which we use as a reference often is not a good example here. Understand how this can make your code faster, e.g. checking for array index up front in the method is not only more clear, it fails fast and hard, and can also eliminate bounds checks ("dominating test"). When reporting errors, ask yourself a) what message you would want to see if you are debugging a problem, and b) what information would give the operational production-support team enough insight to diagnose problems.

* *Document the code.* You may think your code is obvious, but it may not be. Give a high level overview of it to someone who is unfamiliar with those thousands of lines of code, such that you both would be able to divide and conquer. Document a summary of what things do at package, class, and method level. If you think your own code is tricky or hairy, do even more to try to compensate. Long living code is written once but read and reread many times!

* *Private by default.* Java’s access levels are a good way to encapsulate code: separate the interface/contract from the implementation detail. Private is best, package-private is good, public is last resort. Be careful about what you expose, so that your class or API is simple easy to use.

* *Every change deserves review.* Our system is complex and every change can have potential side effects. We expect everyone to work hard and think things through but there will be times when implications are missed. Every change should be proposed and receive at least one LGTM. For complex changes, two reviewers are better. On some teams two is the minimum number of LGTMs, three for complex changes. Coders and reviewers share responsibility for failures associated with a change; this encourages careful review. Sometimes a feature fails unexpectedly because something it depends on has changed or broken. We should all take ownership for failures and unexpected problems for customers, rather than blaming a few people.

* *Rules are meant to be broken.* Sometimes the code has to break the rules. Maybe it’s five times easier to understand if your comment contains a table 150 characters wide. Nasty abstractions to try to enforce "DRY" can end out far worse than a simple duplication of code!

## Interacting with people

* *Voice your opinion with precision and respect.* Always share what you have to say but leave room for another opinion. Always explain your reasons. Ultimatums kill conversation. Phrases like "This will never work" and “This is stupid” are lazy and imprecise. Say “I think this will be problematic in the case that ... because … “.  Don’t say “This is wrong”; say “I think this is wrong because…”.  Don’t say “Is this really needed?”; ask “Why is this needed?” Don’t say “I am not open to anything else” or “There is nothing to discuss”. See the point about vetoes instead.

* *Be kind.* The written form is hard. What you intend with your words may not be obvious to the reader. Make the effort to explain your reasoning clearly. Be quick to apologise if you haven’t made a good job of explaining. Assume misunderstanding rather than malice. When in any doubt about communicating your idea, get on video or voice chat.

From [Being Kind](http://boz.com/articles/be-kind.html):

*Being kind is fundamentally about taking responsibility for your impact on the people around you. It requires you be mindful of their feelings and considerate of the way your presence affects them. It isn’t the same as being nice. It isn’t about superficial praise. It doesn’t mean dulling your opinions. And it shouldn’t diminish the passion with which you present them.*

* *Thank people.* Say it when someone has done a good job. Take the time to include some detail on why you think it was a good job to make it sincere and specific.

(Beware of the bad audio of the recording: [How to Delegate, Like a Boss](https://www.youtube.com/watch?v=h3MPewsk5PU&t=5m55s))

* *With power comes responsibility.* You have the power to veto. A veto or in other words a `-1` is a strong call. Only use it as a last resort when you are 100% convinced that a certain change should not be made. Don’t use it if you only disagree or do not like a change. **Beware that a veto will kill progress of the issue/change and won’t be overruled unless retracted**, so be conscious of the seriousness of a veto. 

Use wise words to explain your objection. Your veto must come with a technical reason. Be prepared to discuss and explain. The vetoed change is guaranteed to be seen as good by the persons who proposed it and they deserve that discussion. Of course, they also deserve the chance to convince you of their reason. In that past, such healthy discussion actually ended with the a veto being pulled back.

* *Empathy for passion.* Some of your coworkers have unlimited passion. Unfortunately it doesn’t always come with unlimited patience. It’s always good to change communication channels if discussions go sideways. Face to face communication is crucial in such a development environment. For example, some of our code cleanup efforts last several months. Think of settings refactoring or removing google guice. If you argue on an issue, keep in mind that the other person might have spent months on this and you may be missing something in the big picture. It’s enormously hard to ensure both people are on the same page! If in doubt do /zoom on slack or go on aOn.

* *Empathy for the pressured.* You will face situations where an argument goes sideways. You will see situations where people do not use a polite voice. Don’t accept it but try to look behind the scenes, speak about it, and forgive.

* *Report abusive comments to our Code of Conduct team.* If you see abusive comments, even if you are not part of the conversation, please report it (push back). Don’t fuel a discussion, and don’t provide a forum for more abusive comments. End the discussion and notify others to back you up. If you don’t feel like doing this, it’s fine to ping others directly to do this for you. Use github emoji (and reaction emoji) to amplify pushback from others. Then move on with technical arguments. Be the bigger personality and help others to reduce the impact of abusive or aggressive comment.

* *Ask questions if in doubt.* There are many complex areas in Elasticsearch. If you are in doubt or you are not sure how to fix a certain problem, if you don’t even know how to approach the problem or you are stuck, go and get help. Having zoom sessions to explain the problem to others might even help in realizing that the solution to the problem is a totally different one.

* *Solve conflicts by speaking to each other. Accept decisions, even if not yours and move on.* We are all passionate and opinionated people. This is what makes us good at our job and moves the code forward. It also means we will not always agree. Talk things through and try to see the other side. Almost always there is also a third way that will make both parties happy. In the worst case, there will be times that consensus is not reached and leadership has to make a call. [Disagree and commit](https://www.google.com/url?q=https://www.amazon.jobs/principles&sa=D&ust=1470304258318000&usg=AFQjCNEtnDcPw2eh-GlszSmtsrGfZtSoMw). Remember: nothing is final and things can be changed if they have been proven wrong.

受启发于：

[Zen of Python](https://en.wikipedia.org/wiki/Zen_of_Python) 

[Contributor Covenant](http://contributor-covenant.org/)

[Amazon’s Leadership Principles](https://www.amazon.jobs/principles)

[Rust’s Code of Conduct](https://www.rust-lang.org/en-US/conduct.html), [Rust video on Conduct](https://youtu.be/dIageYT0Vgg?t=7m2s)

 

