# Do you grok Grok?

>原文：[Do you grok Grok?](https://www.elastic.co/blog/do-you-grok-grok)
>
>译者：[neal1991](https://github.com/neal1991)
>
>welcome to star my [articles-translator ](https://github.com/neal1991), providing you advanced articles translation. Any suggestion, please issue or contact [me](mailto:bing@stu.ecnu.edu.cn)
>
>LICENSE: [MIT](https://opensource.org/licenses/MIT)




> grok (verb) 
>
> understand (something) intuitively or by empathy.

One the most common tasks when parsing log data is to decompose raw lines of text into a set of structured fields which other tools can manipulate. If you’re using the Elastic Stack, you can leverage Elasticsearch’s aggregations and Kibana’s visualizations to answer both business and operational questions from the information extracted in the logs, like ip addresses, timestamps, and domain specific data.

解析日志数据时最常见的任务是将原始文本行分解为其他工具可以操作的一组结构化字段。 如果你使用 Elastic Stack，则可以利用 Elasticsearch 的聚合和 Kibana 的可视化，从日志中提取的信息（如 IP 地址，时间戳和特定域的数据）解释业务和操作问题。

For Logstash, this deconstruction job is carried by [logstash-filter-grok](https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html), a filter plugin that helps you describe the structure of your log formats.

对于 Logstash，这个解构工作由 [logstash-filter-grok](https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html) 来承担，它是一个过滤器插件，可以帮助你描述日志格式的结构。

There are [over 200 grok patterns available](https://github.com/logstash-plugins/logstash-patterns-core/tree/master/patterns) which abstract concepts such as [IPv6 addresses](https://github.com/logstash-plugins/logstash-patterns-core/blob/v4.0.2/patterns/grok-patterns#L29) , [UNIX paths](https://github.com/logstash-plugins/logstash-patterns-core/blob/v4.0.2/patterns/grok-patterns#L38) and [names of months](https://github.com/logstash-plugins/logstash-patterns-core/blob/v4.0.2/patterns/grok-patterns#L52). 

[这里有超过200个 grok 模式](https://github.com/logstash-plugins/logstash-patterns-core/tree/master/patterns)对于一些概念进行概括，如[IPv6 地址](https://github.com/logstash-plugins/logstash-patterns-core/blob/v4.0.2/patterns/grok-patterns＃L29），[UNIX 路径](https://github.com/logstash-plugins/logstash-patterns-core/blob/v4.0.2/patterns/grok-patterns＃L38)和[月份名称](https://github.com/logstash-plugins/logstash-patterns-core/blob/v4.0.2/patterns/grok-patterns#L52)。

为了以 grok 库匹配下列一行的格式，只需要将一些模式组合在一起：
In order to match a line with the format:

`2016-09-19T18:19:00 [8.8.8.8:prd] DEBUG this is an example log message`
with the grok library, it’s only necessary to compose a handful of patterns to come up with:

`%{TIMESTAMP_ISO8601:timestamp} \[%{IPV4:ip};%{WORD:environment}\] %{LOGLEVEL:log_level} %{GREEDYDATA:message}`

这样就会生成结构化结果：
Which will create the structure:

```{
{
  "timestamp": "2016-09-19T18:19:00",

  "ip": "8.8.8.8",

  "environment": "prd",

  "log_level": "DEBUG",

  "message": "this is an example log message"

}

```


Easy right?

Yes!

Great! Are we done here? No! Because..

很简单，是不是？

是！

很棒！就到这了么？不！因为...

## “I’m using grok and it’s super slow!!”
## “我正在使用 grok 并且它非常慢”

That is a very common remark! Performance is a topic that is often brought up from the community as, often enough, users or customers will create a grok expression that will greatly reduce the number of events per second being processed by the logstash pipeline.

As mentioned before, grok patterns are regular expressions, and therefore this plugin’s performance is severely impacted by the behaviour of the regular expression engine. In the following chapters, we’ll provide some guidelines on do’s and don’ts when creating grok expressions to match your log lines.

这是一个非常普遍的说法！性能是一个经常从社区引发的话题，用户或客户通常会创建一个 grok 表达式，这将极大地减少 logstash 管道每秒处理的事件数量。

如前所述，grok 模式是正则表达式，因此这个插件的性能受到正则表达式引擎严重影响。 在接下来的章节中，我们将提供一些关于创建 grok 表达式来匹配日志行的操作指南。

## Measure, measure, measure
## 测量，测量，测量

In order to validate decisions and experiments during grok expression design, we need a way to quickly measure performance between two or more expressions. For this, I created a small jruby script that uses the logstash-filter-grok plugin directly, bypassing the logstash pipeline.
为了在 grok 表达式设计过程中验证决策和实验，我们需要一种方法来快速测量两个或更多表达式之间的性能。 为此，我创建了一个小的 jruby 脚本，它直接使用logstash-filter-grok 插件，绕过 logstash 管道。

You can fetch [this script here](https://gist.github.com/jsvd/a2613ea1ba00f02926a302781ca62f7b). We’ll be using it to collect performance numbers to validate (or destroy!) our assumptions.

你可以从[这](https://gist.github.com/jsvd/a2613ea1ba00f02926a302781ca62f7b)获取脚本。我们将使用它来收集性能数据来验证（或者推翻！）我们的假设。

## Beware of the performance impact when grok fails to match
## 留意 grok 匹配失败时的性能影响

Although it is very important to know how fast your grok pattern matches a log entry, it is also essential to understand what happens when it doesn’t. Successful matches can perform very differently than unsuccessful ones.

尽管知道 grok 模式与日志条目可以多快匹配非常重要，但是了解它在什么时候匹配失败也很重要。匹配成功和匹配失败的性能可能会差异很大。

When grok fails to match an event, it will add a tag to the event. By default, this tag is [_grokparsefailure](https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html#plugins-filters-grok-tag_on_failure).

当 grok 无法匹配一个事件的时候，它将会为这个事件添加一个 tag。默认这个 tag 是 [_grokparsefailure](https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html#plugins-filters-grok-tag_on_failure)。

Logstash allows you then to route those events somewhere where they can be counted and reviewed. For example, you can write all the failed matches to a file:

Logstash 允许你将这些事件路由到可以统计和检查的地方。 例如，你可以将所有失败的匹配写入文件：

```yaml
input { # ... }
filter {
  grok {
  match => { "message" => "%{TIMESTAMP_ISO8601:timestamp} [%{IPV4:ip};%{WORD:environment}] %        {LOGLEVEL:log_level} %{GREEDYDATA:message}" }
  }
}
output {
  if "_grokparsefailure" in [tags] {
    # write events that didn't match to a file
    file { "path" => "/tmp/grok_failures.txt" }
  } else {
     elasticsearch { }
  }
}
```


If you find that there are multiple pattern match failures, you can benchmark those lines and find out their impact on the pipeline throughput.

We’ll now use a grok expression that is meant to parse apache log lines and study its behaviour. First, we start with an example log entry:

如果发现有多个模式匹配失败，则可以对这些行进行基准测试，并找出它们对管道吞吐量的影响。

现在我们将使用 grok 表达式来解析 apache 日志行并研究其行为。 首先，我们从一个示例日志条目开始：

`220.181.108.96 - - [13/Jun/2015:21:14:28 +0000] "GET /blog/geekery/xvfb-firefox.html HTTP/1.1" 200 10975 "-" "Mozilla/5.0 (compatible; Baiduspider/2.0; +http://www.baidu.com/search/spider.html)"`

And use the following grok pattern to match it:
使用以下 grok 模式来匹配它：

`%{IPORHOST:clientip} %{USER:ident} %{USER:auth} \[%{HTTPDATE:timestamp}\] "%{WORD:verb} %{DATA:request} HTTP/%{NUMBER:httpversion}" %{NUMBER:response:int} (?:-|%{NUMBER:bytes:int}) %{QS:referrer} %{QS:agent}`

Now, we’ll compare the matching speed of a successful match against three other log entries which don’t conform to the format, either at the start, the middle, or at the end of the line:

现在，我们将比较成功匹配的匹配速度和不符合格式的其他三个日志条目，无论是在开始，中间还是在行尾：

```
beginning mismatch - doesn't start with an IPORHOST

'tash-scale11x/css/fonts/Roboto-Regular.ttf HTTP/1.1" 200 41820 "http://semicomplete.com/presentations/logs'

middle mismatch - instead of an HTTP verb like GET or PUT there's the number 111

'220.181.108.96 - - [13/Jun/2015:21:14:28 +0000] "111 /blog/geekery/xvfb-firefox.html HTTP/1.1" 200 10975 "-" "Mozilla/5.0 (compatible; Baiduspider/2.0; +http://www.baidu.com/search/spider.html)"'

end mismatch - the last element isn't a quoted string, but a number

'220.181.108.96 - - [13/Jun/2015:21:14:28 +0000] "GET /blog/geekery/xvfb-firefox.html HTTP/1.1" 200 10975 "-" 1'

```

These log lines were benchmarked using the script described at the start, and the result is presented below:
这些日志行在文章开头提到的脚本进行基准测试，结果如下：

[![jndER.md.png](https://s1.ax1x.com/2017/12/21/jndER.md.png)](https://imgchr.com/i/jndER)

matching events per second
每秒匹配的日志数

We can see that, for this grok expression, depending on the location of the mismatch, the time spent checking that a line doesn’t match can be up to 6 times slower than a regular (successful) match. This helps explain user reports on grok maximizing CPU usage when lines don’t match, like https://github.com/logstash-plugins/logstash-filter-grok/issues/37.
我们可以看到，对于这个 grok 表达式，取决于不匹配的位置，检查一行不匹配的时间可能比常规（成功）匹配慢6倍。 这有助于解释在行数不匹配时 grok 最大化 CPU 使用率的用户报告，如https://github.com/logstash-plugins/logstash-filter-grok/issues/37。

What can we do about it?
对此我们可以做什么呢？

## Fail Faster, Set Anchors
## 设置锚可以加速失败匹配

So now that we understand that match failures are dangerous to your pipeline’s performance, we need to fix them. In regular expression design, the best thing you can do to aid the regex engine is to reduce the amount of guessing it needs to do. This is why greedy patterns are generally avoided, but we’ll come back to that in a bit, as there’s a much simpler change that alters how your patterns are matched.

既然现在我们知道匹配失败对你的管道性能是很危险的，我们需要修复它们。 在正则表达式设计中，你可以做的最好的事情来帮助正则表达式引擎是减少它需要做的猜测工作。 这就是为什么通常会避免贪婪模式的原因，但是我们稍微回顾一下，因为有一个更简单的变化来改变模式的匹配。

Let’s come back to our lovely apache log line…
让我们回到我们可爱的 apache 日志行...

`220.181.108.96 - - [13/Jun/2015:21:14:28 +0000] "GET /blog/geekery/xvfb-firefox.html HTTP/1.1" 200 10975 "-" "Mozilla/5.0 (compatible; Baiduspider/2.0; +http://www.baidu.com/search/spider.html)"`

…which is parsed by the grok pattern below:
它由以下的 grok 模式来进行解析：

`%{IPORHOST:clientip} %{USER:ident} %{USER:auth} \[%{HTTPDATE:timestamp}\] "%{WORD:verb} %{DATA:request} HTTP/%{NUMBER:httpversion}" %{NUMBER:response:int} (?:-|%{NUMBER:bytes:int}) %{QS:referrer} %{QS:agent}`

There’s a performance problem hiding in plain sight which exists due the natural expectations from the user of the grok plugin: the assumption that the grok expression we wrote will only match our log line from start to finish. In reality, what grok is being told is to “find this sequence of elements within a line of text”.
由于grok插件的用户的自然期望，隐藏在表面上的性能问题显而易见：假设我们编写的 grok 表达式仅从开始到结束与我们的日志行匹配。 实际上，grok 被告知的是“在一行文本中找到这个元素序列”。

Wait, what? That’s right, “within a line of text”. This means that a line such as…
等一下，什么？就是它了，“在一行文本中”。这意味着比如一行数据...

`OMG OMG OMG EXTRA INFORMATION 220.181.108.96 - - [13/Jun/2015:21:14:28 +0000] "GET /blog/geekery/xvfb-firefox.html HTTP/1.1" 200 10975 "-" "Mozilla/5.0 (compatible; Baiduspider/2.0; +http://www.baidu.com/search/spider.html)" OH LOOK EVEN MORE STUFF`

…will still match the grok pattern! The good news is that the fix is simple, we just need to add a couple of Anchors!
将会依然匹配 grol 模式！好消息是修复很简单，我们只需要添加一些锚！

Anchors allow you to pin the regular expression to a certain position of the string. By adding the start and end of line anchors (^ and $) to our grok expression, we make sure that we’ll only match those patterns against the whole string from start to finish, and nothing else.

锚允许你将正则表达式固定到字符串的某个位置。 通过在我们的 grok 表达式中添加行锚点（^和$）的开始和结束，我们确保我们只会匹配整个字符串从开始到结束，而不包含其他的。

This is very important in the case of failure to match. If the anchors aren’t in place and the regex engine can’t match a line, it will start trying to find the pattern within substrings of the initial string, hence the performance degradation we saw above.

这在匹配失败的情况下非常重要。 如果锚点不在位，并且正则表达式引擎不能匹配一行日志，它将开始尝试在初始字符串的子字符串中查找该模式，因此我们在上面看到了性能下降。

So, to see the performance impact, we benchmarked the previous expression against a new one, now with [anchors](https://ruby-doc.org/core-1.9.3/Regexp.html#class-Regexp-label-Anchors):

因此，为了看到性能影响，我们产生一个新的使用[锚](https://ruby-doc.org/core-1.9.3/Regexp.html#class-Regexp-label-Anchors)的表达式与之前的表达式进行对比：

`^%{IPORHOST:clientip} %{USER:ident} %{USER:auth} \[%{HTTPDATE:timestamp}\] "%{WORD:verb} %{DATA:request} HTTP/%{NUMBER:httpversion}" %{NUMBER:response:int} (?:-|%{NUMBER:bytes:int}) %{QS:referrer} %{QS:agent}$`

Here are the results:
下面是结果：

[![jncKe.md.png](https://s1.ax1x.com/2017/12/21/jncKe.md.png)](https://imgchr.com/i/jncKe)

It’s a pretty dramatic change in behavior for the non matching scenarios! Not only we removed huge performance drops in the middle and end scenarios, but also made the initial match failure detection around 10 times faster. Sweet.
对于不匹配的场景，这是一个相当显著的变化！ 不仅我们在中端和末端场景中消除了巨大的性能下降，而且使初始匹配失败检测速度提高了 10 倍左右。 赞。

## Beware of matching the same thing twice
## 留意两次匹配相同的行

You might be tempted to say: “well, all my lines are correctly formatted so we don’t have failed matched”, but this might not be the case.

Over time, we’ve seen a very common pattern of grok usage, especially when lines from multiple applications come through a single gateway like syslog which adds a common header to all messages. Let’s take an example: imagine that we have three applications which log using a “common_header: payload” format:

你可能会说：“好吧，我的所有行都格式正确，所以我们没有匹配失败”，但情况可能并非如此。

随着时间的推移，我们已经看到了 grok 用法的一个非常常见的模式，尤其是当来自多个应用程序的日志行通过单个网关（如 syslog）向所有消息添加公共头时。 举一个例子：假设我们有三个使用“common_header：payload”格式的应用程序：

```
Application 1: '8.8.8.8 process-name[666]: a b 1 2 a lot of text at the end'

Application 2: '8.8.8.8 process-name[667]: a 1 2 3 a lot of text near the end;4'

Application 3: '8.8.8.8 process-name[421]: a completely different format | 1111'
```

A common grok setup for this would be to match the three formats in one grok:

一个常见的 grok 设置就是在一个 grok 中匹配三种格式：

```
grok {
  "match" => { "message => [
    '%{IPORHOST:clientip} %{DATA:process_name}\[%{NUMBER:process_id}\]: %{WORD:word_1} %{WORD:word_2} %{NUMBER:number_1} %{NUMBER:number_2} %{DATA:data}',
    '%{IPORHOST:clientip} %{DATA:process_name}\[%{NUMBER:process_id}\]: %{WORD:word_1} %{NUMBER:number_1} %{NUMBER:number_2} %{NUMBER:number_3} %{DATA:data};%{NUMBER:number_4}',
    '%{IPORHOST:clientip} %{DATA:process_name}\[%{NUMBER:process_id}\]: %{DATA:data} | %{NUMBER:number}'
    ] }
}
```

Now notice that even if your applications log correctly, grok will still sequentially try to match the incoming log line against the three expressions, breaking at the first match.

This means that it’s still important to ensure we skip to the right one as fast as possible, since you’ll always have one failed match for Application 2 and two failed matches for Application 3.

The first tactic we often see is to tier the grok matching: first match the header, overwrite the message field, then match only the bodies:

现在请注意，即使你的应用程序正确日志记录，grok 仍然会依次尝试将传入日志行与三个表达式进行匹配，从而在第一次匹配时中断。

这意味着确保我们尽可能快地跳到正确的位置仍然很重要，因为应用程序2总是有一个失败的匹配，应用程序3有两个失败的匹配。

我们经常看到的第一个策略是对Grok匹配进行分层：首先匹配 header，覆盖 message 字段，然后仅匹配 bodies：

```
filter {
  grok {
    "match" => { "message" => '%{IPORHOST:clientip} %{DATA:process_name}\[%{NUMBER:process_id}\]: %{GREEDYDATA:message}' },
    "overwrite" => "message"
  }
  grok {
    "match" => { "message" => [
      '%{WORD:word_1} %{WORD:word_2} %{NUMBER:number_1} %{NUMBER:number_2} %{GREEDYDATA:data}',
      '%{WORD:word_1} %{NUMBER:number_1} %{NUMBER:number_2} %{NUMBER:number_3} %{DATA:data};%{NUMBER:number_4}',
      '%{DATA:data} | %{NUMBER:number}'
    ] }
  }
)
```

This alone provides an interesting performance boost, matching lines 2.5x faster than the initial approach. But what if we add our fellow anchors?

[![jnTxS.md.png](https://s1.ax1x.com/2017/12/21/jnTxS.md.png)](https://imgchr.com/i/jnTxS)

Interesting! Adding anchors makes both architectures perform equally well! In fact, because of the greatly increased failed match performance, our initial single grok design performs slightly better since there is one less match being executed.

## Ok, so how can I know things aren’t going well?

We’ve already come to the conclusion that monitoring the existence of “_grokparsefaiure” events is essential, but there is more that you can do:

Since version 3.2.0 of the grok plugin, there’s a couple of settings that help you understand when an event is taking a long time to match (or fail to). Using [timeout_millis and tag_on_timeout](https://www.elastic.co/guide/en/logstash/5.0/plugins-filters-grok.html#plugins-filters-grok-timeout_millis) it’s possible to set an upper bound time limit to the execution of the grok match. If this limit is reached, the match is interrupted and the event is tagged by default with _groktimeout.

Using the same conditional strategy we presented before, you can redirect those events to a file or a different index in elasticsearch for later analysis.

Another really cool thing that we will be introducing in Logstash 5.0 in the context of metrics is the ability to extract data on the pipeline performance and, most importantly, per plugin statistics. While logstash is running, you can query its API endpoint and see how much cumulative time logstash is spending on a plugin:

```
$ curl localhost:9600/_node/stats/pipeline?pretty | jq ".pipeline.plugins.filters"
[
  {
    "id": "grok_b61938f3833f9f89360b5fba6472be0ad51c3606-2",
    "events": {
      "duration_in_millis": 7,
      "in": 24,
      "out": 24
    },
    "failures": 24,
    "patterns_per_field": {
      "message": 1
    },
    "name": "grok"
  },
  {
    "id": "kv_b61938f3833f9f89360b5fba6472be0ad51c3606-3",
    "events": {
      "duration_in_millis": 2,
      "in": 24,
      "out": 24
    },
    "name": "kv"
  }
]
```

With this information, you can see if grok’s “duration_in_millis” is growing rapidly or not and if the number of failures is increasing, which could serve as a warning flag that some pattern is not well designed or consuming more time than expected.

## Conclusion
Hopefully this blog post will help you understand how grok is behaving and how to improve its throughput. To sum up our conclusions:

Grok may not perform well when a match fails;
Monitor the occurrence of _grokparsefailures and then benchmark 
their cost;
Use anchors such as ^ and $ to remove ambiguity and aid the regex 
engine;
Tiered matching increases performance if you don’t use anchors, 
otherwise don’t bother. When in doubt, measure!
Using either timeout settings or the upcoming Metrics API allows you 
to get a better look into how grok is behaving, and serve a starting 
point for performance analysis.
