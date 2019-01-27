# Do you grok Grok?

>原文：[Do you grok Grok?](https://www.elastic.co/blog/do-you-grok-grok)
>
>译者：[neal1991](https://github.com/neal1991)
>
>welcome to star my [articles-translator](https://github.com/neal1991/articles-translator/), providing you advanced articles translation. Any suggestion, please issue or contact [me](mailto:bing@stu.ecnu.edu.cn)
>
>LICENSE: [MIT](https://opensource.org/licenses/MIT)


> grok (verb) 
>
> understand (something) intuitively or by empathy.

解析日志数据时最常见的任务是将原始文本行分解为其他工具可以操作的一组结构化字段。 如果你使用 Elastic Stack，则可以利用 Elasticsearch 的聚合和 Kibana 的可视化，从日志中提取的信息（如 IP 地址，时间戳和特定域的数据）解释业务和操作问题。

对于 Logstash，这个解构工作由 [logstash-filter-grok](https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html) 来承担，它是一个过滤器插件，可以帮助你描述日志格式的结构。

[这里有超过200个 grok 模式](https://github.com/logstash-plugins/logstash-patterns-core/tree/master/patterns)对于一些概念进行概括，如[IPv6 地址](https://github.com/logstash-plugins/logstash-patterns-core/blob/v4.0.2/patterns/grok-patterns#L29)，[UNIX 路径](https://github.com/logstash-plugins/logstash-patterns-core/blob/v4.0.2/patterns/grok-patterns#L38)和[月份名称](https://github.com/logstash-plugins/logstash-patterns-core/blob/v4.0.2/patterns/grok-patterns#L52)。

为了以 grok 库匹配下列一行的格式，只需要将一些模式组合在一起：

`2016-09-19T18:19:00 [8.8.8.8:prd] DEBUG this is an example log message`

`%{TIMESTAMP_ISO8601:timestamp} \[%{IPV4:ip};%{WORD:environment}\] %{LOGLEVEL:log_level} %{GREEDYDATA:message}`

这样就会生成结构化结果：

```{
{
  "timestamp": "2016-09-19T18:19:00",

  "ip": "8.8.8.8",

  "environment": "prd",

  "log_level": "DEBUG",

  "message": "this is an example log message"

}

```

很简单，是不是？

是！

很棒！就到这了么？不！因为...

## “我正在使用 grok 并且它非常慢”

这是一个非常普遍的说法！性能是一个经常从社区引发的话题，用户或客户通常会创建一个 grok 表达式，这将极大地减少 logstash 管道每秒处理的事件数量。

如前所述，grok 模式是正则表达式，因此这个插件的性能受到正则表达式引擎严重影响。 在接下来的章节中，我们将提供一些关于创建 grok 表达式来匹配日志行的操作指南。

## 测量，测量，测量

为了在 grok 表达式设计过程中验证决策和实验，我们需要一种方法来快速测量两个或更多表达式之间的性能。 为此，我创建了一个小的 jruby 脚本，它直接使用logstash-filter-grok 插件，绕过 logstash 管道。

你可以从[这](https://gist.github.com/jsvd/a2613ea1ba00f02926a302781ca62f7b)获取脚本。我们将使用它来收集性能数据来验证（或者推翻！）我们的假设。

## 留意 grok 匹配失败时的性能影响

尽管知道 grok 模式与日志条目可以多快匹配非常重要，但是了解它在什么时候匹配失败也很重要。匹配成功和匹配失败的性能可能会差异很大。

当 grok 无法匹配一个事件的时候，它将会为这个事件添加一个 tag。默认这个 tag 是 [_grokparsefailure](https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html#plugins-filters-grok-tag_on_failure)。

Logstash 允许你将这些事件路由到可以统计和检查的地方。 例如，你可以将所有失败的匹配写入文件：

```yaml
input { # ... }
filter {
  grok {
  match => { 
  "message" => "%{TIMESTAMP_ISO8601:timestamp} [%{IPV4:ip};%{WORD:environment}] %{LOGLEVEL:log_level} %{GREEDYDATA:message}" }
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

如果发现有多个模式匹配失败，则可以对这些行进行基准测试，并找出它们对管道吞吐量的影响。

现在我们将使用 grok 表达式来解析 apache 日志行并研究其行为。 首先，我们从一个示例日志条目开始：

`220.181.108.96 - - [13/Jun/2015:21:14:28 +0000] "GET /blog/geekery/xvfb-firefox.html HTTP/1.1" 200 10975 "-" "Mozilla/5.0 (compatible; Baiduspider/2.0; +http://www.baidu.com/search/spider.html)"`

使用以下 grok 模式来匹配它：

`%{IPORHOST:clientip} %{USER:ident} %{USER:auth} \[%{HTTPDATE:timestamp}\] "%{WORD:verb} %{DATA:request} HTTP/%{NUMBER:httpversion}" %{NUMBER:response:int} (?:-|%{NUMBER:bytes:int}) %{QS:referrer} %{QS:agent}`

现在，我们将比较成功匹配的匹配速度和不符合格式的其他三个日志条目，无论是在开始，中间还是在行尾：

```
beginning mismatch - doesn't start with an IPORHOST

'tash-scale11x/css/fonts/Roboto-Regular.ttf HTTP/1.1" 200 41820 "http://semicomplete.com/presentations/logs'

middle mismatch - instead of an HTTP verb like GET or PUT there's the number 111

'220.181.108.96 - - [13/Jun/2015:21:14:28 +0000] "111 /blog/geekery/xvfb-firefox.html HTTP/1.1" 200 10975 "-" "Mozilla/5.0 (compatible; Baiduspider/2.0; +http://www.baidu.com/search/spider.html)"'

end mismatch - the last element isn't a quoted string, but a number

'220.181.108.96 - - [13/Jun/2015:21:14:28 +0000] "GET /blog/geekery/xvfb-firefox.html HTTP/1.1" 200 10975 "-" 1'

```

这些日志行在文章开头提到的脚本进行基准测试，结果如下：

[![jndER.md.png](https://s1.ax1x.com/2017/12/21/jndER.md.png)](https://imgchr.com/i/jndER)

每秒匹配的日志数

我们可以看到，对于这个 grok 表达式，取决于不匹配的位置，检查一行不匹配的时间可能比常规（成功）匹配慢6倍。 这有助于解释在行数不匹配时 grok 最大化 CPU 使用率的用户报告，如https://github.com/logstash-plugins/logstash-filter-grok/issues/37。

对此我们可以做什么呢？

## 设置锚可以提升匹配失败的性能

既然现在我们知道匹配失败对你的管道性能是很危险的，我们需要修复它们。 在正则表达式设计中，你可以做的最好的事情来帮助正则表达式引擎是减少它需要做的猜测工作。 这就是为什么通常会避免贪婪模式的原因，但是我们稍微回顾一下，因为有一个更简单的变化来改变模式的匹配。

让我们回到我们可爱的 apache 日志行...

`220.181.108.96 - - [13/Jun/2015:21:14:28 +0000] "GET /blog/geekery/xvfb-firefox.html HTTP/1.1" 200 10975 "-" "Mozilla/5.0 (compatible; Baiduspider/2.0; +http://www.baidu.com/search/spider.html)"`

它由以下的 grok 模式来进行解析：

`%{IPORHOST:clientip} %{USER:ident} %{USER:auth} \[%{HTTPDATE:timestamp}\] "%{WORD:verb} %{DATA:request} HTTP/%{NUMBER:httpversion}" %{NUMBER:response:int} (?:-|%{NUMBER:bytes:int}) %{QS:referrer} %{QS:agent}`

由于grok插件的用户的自然期望，隐藏在表面上的性能问题显而易见：假设我们编写的 grok 表达式仅从开始到结束与我们的日志行匹配。 实际上，grok 被告知的是“在一行文本中找到这个元素序列”。

等一下，什么？就是它了，“在一行文本中”。这意味着比如一行数据...

`OMG OMG OMG EXTRA INFORMATION 220.181.108.96 - - [13/Jun/2015:21:14:28 +0000] "GET /blog/geekery/xvfb-firefox.html HTTP/1.1" 200 10975 "-" "Mozilla/5.0 (compatible; Baiduspider/2.0; +http://www.baidu.com/search/spider.html)" OH LOOK EVEN MORE STUFF`

将会依然匹配 grok 模式！好消息是修复很简单，我们只需要添加一些锚！

锚允许你将正则表达式固定到字符串的某个位置。 通过在我们的 grok 表达式中添加行锚点（^和$）的开始和结束，我们确保我们只会匹配整个字符串从开始到结束，而不包含其他的。

这在匹配失败的情况下非常重要。 如果锚点不在位，并且正则表达式引擎不能匹配一行日志，它将开始尝试在初始字符串的子字符串中查找该模式，因此我们在上面看到了性能下降。


因此，为了看到性能影响，我们产生一个新的使用[锚](https://ruby-doc.org/core-1.9.3/Regexp.html#class-Regexp-label-Anchors)的表达式与之前的表达式进行对比：

`^%{IPORHOST:clientip} %{USER:ident} %{USER:auth} \[%{HTTPDATE:timestamp}\] "%{WORD:verb} %{DATA:request} HTTP/%{NUMBER:httpversion}" %{NUMBER:response:int} (?:-|%{NUMBER:bytes:int}) %{QS:referrer} %{QS:agent}$`

下面是结果：

[![jncKe.md.png](https://s1.ax1x.com/2017/12/21/jncKe.md.png)](https://imgchr.com/i/jncKe)

对于不匹配的场景，这是一个相当显著的变化！ 不仅我们在中端和末端场景中消除了巨大的性能下降，而且使初始匹配失败检测速度提高了 10 倍左右。 赞。

## 留意两次匹配相同的行

你可能会说：“好吧，我的所有行都格式正确，所以我们没有匹配失败”，但情况可能并非如此。

随着时间的推移，我们已经看到了 grok 用法的一个非常常见的模式，尤其是当来自多个应用程序的日志行通过单个网关（如 syslog）向所有消息添加公共头时。 举一个例子：假设我们有三个使用“common_header：payload”格式的应用程序：

```
Application 1: '8.8.8.8 process-name[666]: a b 1 2 a lot of text at the end'

Application 2: '8.8.8.8 process-name[667]: a 1 2 3 a lot of text near the end;4'

Application 3: '8.8.8.8 process-name[421]: a completely different format | 1111'
```

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

仅仅这一个就是一个有趣的性能提升，匹配行比初始方法快了2.5倍。 但是如果我们添加我们的同伴锚呢？

[![jnTxS.md.png](https://s1.ax1x.com/2017/12/21/jnTxS.md.png)](https://imgchr.com/i/jnTxS)

有意思！添加锚点使得两个架构的性能同样出色！ 事实上，由于失败的匹配性能大大提高，我们最初的单杆设计稍微好一点，因为有一个比较少的匹配正在执行。

## 好的，那么我们如何知道事情进行得如何？


我们已经得出结论，监控“_grokparsefaiure”事件的存在是必不可少的，但是你可以做更多的事情：

自从版本 3.2.0 grok 插件，已经有很多设置可以帮助你什么时候事件需要花费长时间来匹配（或者失败匹配）。使用[timeout millis 以及 timeout 标签](https://www.elastic.co/guide/en/logstash/5.0/plugins-filters-grok.html#plugins-filters-grok-timeout_millis)能够对于 grok 匹配的时间设置一个上限。如果达到了限制时间，这次匹配就会被中断并且被打上 `_groktimeout` 标签。

使用我们之前介绍的相同的条件策略，你·可以将这些事件重定向到 elasticsearch 中的文件或不同的索引，以供日后分析。

另一个非常酷的事情，我们将在 Logstash 5.0 中引入度量的上下文是能够提取管道性能的数据，最重要的是，每个插件的统计数据。 在 logstash 运行时，你可以查询其 AP I端点，并查看 logstash 在一个插件上花费的累积时间：

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

有了这些信息，你可以看到grok的“duration_in_millis”是否快速增长，如果失败的数量在增加，可以作为警告标志，表明某些模式设计不好，或者消耗的时间比预期的多。

## 总结

希望这篇博文能够帮助你理解 grok 的行为，以及如何提高吞吐量。 总结我们的结论：

1. grok 匹配失败的时候性能可能表现不好。
2. 监测发生 `_grokfailures`的情况并且对于他们的消耗进行基准测试。
3. 使用锚比如 `^` 以及`$` 避免歧义并且帮助正则引擎。
4. 如果你不使用锚的话使用分层匹配会提高性能。如果怀疑的话，直接测量。
5. 使用超时设置或者即将推出的 Metrics API 能够让你更好地了解 grok 是如何工作的，并且是性能分析的第一点。
