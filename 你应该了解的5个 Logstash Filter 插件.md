# 你应该了解的5个 Logstash Filter 插件

> 原文：[5 Logstash Filter Plugins You Need to Know About](https://logz.io/blog/5-logstash-filter-plugins/#ult-fs-search)
>
> 译者：[neal1991](https://github.com/neal1991)
>
> welcome to star my [articles-translator](https://github.com/neal1991/articles-translator/), providing you advanced articles translation. Any suggestion, please issue or contact [me](mailto:bing@stu.ecnu.edu.cn)
>
> LICENSE: [MIT](https://opensource.org/licenses/MIT)

![logstash filter](https://logz.io/wp-content/uploads/2017/08/logstash_filter2.jpg)

在 [ELK](https://logz.io/learn/complete-guide-elk-stack/) 中， Logstash 处理资源繁重的日志聚合和处理的任务。 Logstash 执行的处理工作确保我们的日志消息被正确解析和结构化，并且这种结构使你能够更容易地在 Elasticsearch 中进行索引分析和数据可视化。

对数据执行什么精确处理由你在 Logstash 配置文件的 filter 部分确定。 在本节中，你可以从大量的官方支持和社区 filter 插件中选择从而决定如何转换日志。 最常用的过滤器插件是 grok，但是还有一些其他非常有用的插件可以使用。

你使用的插件当然取决于日志本身，但本文尝试列出你最有可能在涉及 Logstash 的任何日志处理中找到的五个插件。

## # 1 grok

如上所述，grok 是 Logstash 中最常用的过滤器插件。 尽管事实上它不容易使用，但是 grok 非常受欢迎，因为它允许你将非结构化日志结构化。

以下面的随机日志消息为例：

```
2016-07-11T23:56:42.000+00:00 INFO
[MySecretApp.com.Transaction.Manager]:Starting transaction for session
-464410bf-37bf-475a-afc0-498e0199f00
```

我们使用的 grok 正则就跟下面一样：

```
filter {
	grok {
    	match => { "message" =>"%{TIMESTAMP_ISO8601:timestamp} %{LOGLEVEL:log-level} [%{DATA:class}]:%{GREEDYDATA:message}" }
        }
}
```

处理之后，日志消息就会处理成下面这样：

```
{ 
"timestamp" => "2016-07-11T23:56:42.000+00:00",
"log-level" =>"INFO",
"class" =>"MySecretApp.com.Transaction.Manager"
"message" => "Starting transaction for session   -464410bf-37bf-475a-afc0-498e0199f008" 
}
```

这就是 Elasticsearch 如何索引日志消息。 以此格式排序，日志消息已被分解成逻辑命名的字段，可以更容易地查询，分析和可视化。

在[这篇文章中](https://logz.io/blog/logstash-grok/)可以找到更多关于 grok 如何工作和使用的信息。

## # 2 mutate 

另一个常见的 Logstash filter 插件是 *mutate*。 顾名思义，这个 filter 允许你通过“改变”各个字段真正地转换你的日志消息。 例如，你可以使用 filter 来更改字段，将它们拼接在一起，重命名它们等等。

使用上面的日志作为示例，使用 *mutate* 插件的 *lowercase* 配置选项，我们可以将“log-level”字段转换为小写：

```
filter { 
	grok {...}
	mutate {   lowercase => [ "log-level" ]  }
}
```

mutate 插件是更改日志格式的好方法。 [这里](https://www.elastic.co/guide/en/logstash/current/plugins-filters-mutate.html)列出了插件的不同配置选项的完整列表。

## # 3 date 

如果分析日志和事件未按时间顺序排列怎么办？

Logstash *date* filter 插件可用于从日志消息中提取时间和日期，并将其定义为日志的时间戳字段（@timestamp）。 一旦定义，这个时间戳字段将以正确的时间顺序排列日志，并帮助你更有效地分析它们。

有几十种（如果不是数百种）不同的方式可以在日志中格式化时间和日期。

以下是Apache访问日志的示例：

```
200.183.100.141 - - [25/Nov/2016:16:17:10 +0000] "GET
/wp-content/force-download.php?file=../wp-config.php HTTP/1.0" 200
3842 "http://hack3r.com/top_online_shops" "Mozilla/4.0 (compatible;
MSIE 8.0; Windows NT 5.1; Trident/4.0; YTB720; GTB7.2; .NET CLR 1.1.4322; .NET CLR 2.0.50727; .NET CLR 3.0.4506.2152; .NET CLR 3.5.30729)"
```

使用 date filter 如下，我们可以提取日期和时间正则，并将其定义为@timestamp字段，并根据此所有日志将按以下排序：

```
filter {
grok {
	match => { "message" => "%{COMBINEDAPACHELOG}"}
}
date {
	match => ["timestamp", "dd/MMM/yyyy:HH:mm:ss Z"]
    target => "@timestamp"     }
}
```

请注意，如果不使用日期过滤器，Logstash将根据输入时间自动设置时间戳。

在[这里](https://www.elastic.co/guide/en/logstash/5.4/plugins-filters-date.html)阅读有关其他配置选项。

## # 4 json 

[JSON](http://www.json.org/) 是一种非常受欢迎的日志格式，因为它允许用户编写可以轻松读取和分析的结构化和标准化的消息。

为了维护整个消息或特定字段的 JSON 结构，Logstash *json* filter 插件使你能够在日志消息中提取和维护 JSON 数据结构。

下面的示例是一个格式为 JSON 的 Apache 访问日志：

```
{
"time":"[30/Jul/2017:17:21:45 +0000]",
"remoteIP":"192.168.2.1",
"host":"my.host.local",
"request":"/index.html",
"query":"",
"method":"GET",
"status":"200",
"userAgent":"Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 5.1; Trident/4.0; YTB720; GTB7.2; .NET CLR 1.1.4322; .NET CLR 2.0.50727; .NET CLR 3.0.4506.2152; .NET CLR 3.5.30729)", "referer":"-" 
}
```

我们可以使用 json filter 来保留数据结构，而不是将日志平铺成一行：

```
filter { json { source =>"message"        target => "log"   }  } }
```

source 配置选项定义日志中的哪个字段是你要解析的 JSON。 在这个例子中，整个消息字段是一个 JSON。 我还使用目标选项将 JSON 扩展为名为  log 的字段中的数据结构。

在[这里](https://www.elastic.co/guide/en/logstash/5.4/plugins-filters-json.html)阅读有关其他配置选项。

## # 5 kv 

键值对或 KVP 是另一种常用的日志格式。 像 JSON 一样，这种格式主要是因为它是可读的，Logstash kv filter 插件允许你自动解析消息或以这种方式格式化的特定字段。

以此日志为例：

```
2017-07025 17:02:12 level=error message="connection refused" service="listener" thread=125 customerid=776622 ip=34.124.233.12 queryid=45
```

我可以使用以下 kv filter 来指示 Logstash 如何处理它：

```
filter {  
kv {
source => "metadata"
trim => "\""
include_keys => [ "level","service","customerid",”queryid” ]
target => "kv"
	}
}
```

请注意配置选项的使用。 我正在使用 source 来定义字段来执行 key = value 搜索，trim 以忽略特定字符，include_keys指定应该添加到日志中的解析 key，并且定位到所有 key 对象的容器， 再放入值。

在[这里](https://www.elastic.co/guide/en/logstash/5.4/plugins-filters-kv.html#plugins-filters-kv-target)阅读有关其他配置选项。

## 总结

正如我在文章开头所说，有大量的 Logstash filter 插件可供你使用。 你使用哪一个当然取决于您要处理的具体日志消息。

值得一提的其他非常有用的过滤器插件是 geoip（用于添加IP字段的地理数据）和 csv（用于解析CSV日志）插件。

虽然这些插件中的每一个都是有用的，但是当它们一起用于解析日志时，它们的全部功能被释放。 实际上，在大多数情况下，你最有可能使用 grok 和至少一个或两个附加插件的组合。 这种组合使用将保证你的日志在 Logstash 的另一端完美格式化！



