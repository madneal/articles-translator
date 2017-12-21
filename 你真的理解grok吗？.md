# Do you grok Grok?

>原文：[Do you grok Grok?(https://www.elastic.co/blog/do-you-grok-grok)
>
>译者：[neal1991](https://github.com/neal1991)
>
>welcome to star my [articles-translator ](https://github.com/neal1991), providing you advanced articles translation. Any suggestion, please issue or contact [me](mailto:bing@stu.ecnu.edu.cn)


>LICENSE: [MIT](https://opensource.org/licenses/MIT)
> ​grok (verb) 
> understand (something) intuitively or by empathy.

One the most common tasks when parsing log data is to decompose raw lines of text into a set of structured fields which other tools can manipulate. If you’re using the Elastic Stack, you can leverage Elasticsearch’s aggregations and Kibana’s visualizations to answer both business and operational questions from the information extracted in the logs, like ip addresses, timestamps, and domain specific data.

For Logstash, this deconstruction job is carried by logstash-filter-grok, a filter plugin that helps you describe the structure of your log formats.

There are over 200 grok patterns available which abstract concepts such as IPv6 addresses , UNIX paths and names of months. 
In order to match a line with the format:

2016-09-19T18:19:00 [8.8.8.8:prd] DEBUG this is an example log message
with the grok library, it’s only necessary to compose a handful of patterns to come up with:

%{TIMESTAMP_ISO8601:timestamp} \[%{IPV4:ip};%{WORD:environment}\] %{LOGLEVEL:log_level} %{GREEDYDATA:message}
Which will create the structure:

{
  "timestamp": "2016-09-19T18:19:00",
  "ip": "8.8.8.8",
  "environment": "prd",
  "log_level": "DEBUG",
  "message": "this is an example log message"
}
Easy right?

Yes!

Great! Are we done here? No! Because..

“I’m using grok and it’s super slow!!”
That is a very common remark! Performance is a topic that is often brought up from the community as, often enough, users or customers will create a grok expression that will greatly reduce the number of events per second being processed by the logstash pipeline.

As mentioned before, grok patterns are regular expressions, and therefore this plugin’s performance is severely impacted by the behaviour of the regular expression engine. In the following chapters, we’ll provide some guidelines on do’s and don’ts when creating grok expressions to match your log lines.

Measure, measure, measure
In order to validate decisions and experiments during grok expression design, we need a way to quickly measure performance between two or more expressions. For this, I created a small jruby script that uses the logstash-filter-grok plugin directly, bypassing the logstash pipeline.

You can fetch this script here. We’ll be using it to collect performance numbers to validate (or destroy!) our assumptions.

Beware of the performance impact when grok fails to match
Although it is very important to know how fast your grok pattern matches a log entry, it is also essential to understand what happens when it doesn’t. Successful matches can perform very differently than unsuccessful ones.

When grok fails to match an event, it will add a tag to the event. By default, this tag is _grokparsefailure.

Logstash allows you then to route those events somewhere where they can be counted and reviewed. For example, you can write all the failed matches to a file:

input { # ... }
filter {
  grok {
    match => { "message" => "%{TIMESTAMP_ISO8601:timestamp} \[%{IPV4:ip};%{WORD:environment}\] %{LOGLEVEL:log_level} %{GREEDYDATA:message}" }
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
If you find that there are multiple pattern match failures, you can benchmark those lines and find out their impact on the pipeline throughput.

We’ll now use a grok expression that is meant to parse apache log lines and study its behaviour. First, we start with an example log entry:

220.181.108.96 - - [13/Jun/2015:21:14:28 +0000] "GET /blog/geekery/xvfb-firefox.html HTTP/1.1" 200 10975 "-" "Mozilla/5.0 (compatible; Baiduspider/2.0; +http://www.baidu.com/search/spider.html)"
And use the following grok pattern to match it:

%{IPORHOST:clientip} %{USER:ident} %{USER:auth} \[%{HTTPDATE:timestamp}\] "%{WORD:verb} %{DATA:request} HTTP/%{NUMBER:httpversion}" %{NUMBER:response:int} (?:-|%{NUMBER:bytes:int}) %{QS:referrer} %{QS:agent}
Now, we’ll compare the matching speed of a successful match against three other log entries which don’t conform to the format, either at the start, the middle, or at the end of the line:

# beginning mismatch - doesn't start with an IPORHOST
'tash-scale11x/css/fonts/Roboto-Regular.ttf HTTP/1.1" 200 41820 "http://semicomplete.com/presentations/logs'

# middle mismatch - instead of an HTTP verb like GET or PUT there's the number 111
'220.181.108.96 - - [13/Jun/2015:21:14:28 +0000] "111 /blog/geekery/xvfb-firefox.html HTTP/1.1" 200 10975 "-" "Mozilla/5.0 (compatible; Baiduspider/2.0; +http://www.baidu.com/search/spider.html)"'

# end mismatch - the last element isn't a quoted string, but a number
'220.181.108.96 - - [13/Jun/2015:21:14:28 +0000] "GET /blog/geekery/xvfb-firefox.html HTTP/1.1" 200 10975 "-" 1'
These log lines were benchmarked using the script described at the start, and the result is presented below:

matching events per second

We can see that, for this grok expression, depending on the location of the mismatch, the time spent checking that a line doesn’t match can be up to 6 times slower than a regular (successful) match. This helps explain user reports on grok maximizing CPU usage when lines don’t match, like https://github.com/logstash-plugins/logstash-filter-grok/issues/37.

What can we do about it?

Fail Faster, Set Anchors
So now that we understand that match failures are dangerous to your pipeline’s performance, we need to fix them. In regular expression design, the best thing you can do to aid the regex engine is to reduce the amount of guessing it needs to do. This is why greedy patterns are generally avoided, but we’ll come back to that in a bit, as there’s a much simpler change that alters how your patterns are matched.

Let’s come back to our lovely apache log line…

220.181.108.96 - - [13/Jun/2015:21:14:28 +0000] "GET /blog/geekery/xvfb-firefox.html HTTP/1.1" 200 10975 "-" "Mozilla/5.0 (compatible; Baiduspider/2.0; +http://www.baidu.com/search/spider.html)"
…which is parsed by the grok pattern below:

%{IPORHOST:clientip} %{USER:ident} %{USER:auth} \[%{HTTPDATE:timestamp}\] "%{WORD:verb} %{DATA:request} HTTP/%{NUMBER:httpversion}" %{NUMBER:response:int} (?:-|%{NUMBER:bytes:int}) %{QS:referrer} %{QS:agent}
There’s a performance problem hiding in plain sight which exists due the natural expectations from the user of the grok plugin: the assumption that the grok expression we wrote will only match our log line from start to finish. In reality, what grok is being told is to “find this sequence of elements within a line of text”.

Wait, what? That’s right, “within a line of text”. This means that a line such as…

OMG OMG OMG EXTRA INFORMATION 220.181.108.96 - - [13/Jun/2015:21:14:28 +0000] "GET /blog/geekery/xvfb-firefox.html HTTP/1.1" 200 10975 "-" "Mozilla/5.0 (compatible; Baiduspider/2.0; +http://www.baidu.com/search/spider.html)" OH LOOK EVEN MORE STUFF
…will still match the grok pattern! The good news is that the fix is simple, we just need to add a couple of Anchors!

Anchors allow you to pin the regular expression to a certain position of the string. By adding the start and end of line anchors (^ and $) to our grok expression, we make sure that we’ll only match those patterns against the whole string from start to finish, and nothing else.

This is very important in the case of failure to match. If the anchors aren’t in place and the regex engine can’t match a line, it will start trying to find the pattern within substrings of the initial string, hence the performance degradation we saw above.

So, to see the performance impact, we benchmarked the previous expression against a new one, now with anchors:

^%{IPORHOST:clientip} %{USER:ident} %{USER:auth} \[%{HTTPDATE:timestamp}\] "%{WORD:verb} %{DATA:request} HTTP/%{NUMBER:httpversion}" %{NUMBER:response:int} (?:-|%{NUMBER:bytes:int}) %{QS:referrer} %{QS:agent}$
Here are the results:

using anchors

It’s a pretty dramatic change in behavior for the non matching scenarios! Not only we removed huge performance drops in the middle and end scenarios, but also made the initial match failure detection around 10 times faster. Sweet.

Beware of matching the same thing twice
You might be tempted to say: “well, all my lines are correctly formatted so we don’t have failed matched”, but this might not be the case.

Over time, we’ve seen a very common pattern of grok usage, especially when lines from multiple applications come through a single gateway like syslog which adds a common header to all messages. Let’s take an example: imagine that we have three applications which log using a “common_header: payload” format:

Application 1: '8.8.8.8 process-name[666]: a b 1 2 a lot of text at the end'
Application 2: '8.8.8.8 process-name[667]: a 1 2 3 a lot of text near the end;4'
Application 3: '8.8.8.8 process-name[421]: a completely different format | 1111'
A common grok setup for this would be to match the three formats in one grok:

grok {
  "match" => { "message => [
    '%{IPORHOST:clientip} %{DATA:process_name}\[%{NUMBER:process_id}\]: %{WORD:word_1} %{WORD:word_2} %{NUMBER:number_1} %{NUMBER:number_2} %{DATA:data}',
    '%{IPORHOST:clientip} %{DATA:process_name}\[%{NUMBER:process_id}\]: %{WORD:word_1} %{NUMBER:number_1} %{NUMBER:number_2} %{NUMBER:number_3} %{DATA:data};%{NUMBER:number_4}',
    '%{IPORHOST:clientip} %{DATA:process_name}\[%{NUMBER:process_id}\]: %{DATA:data} | %{NUMBER:number}'
    ] }
}
Now notice that even if your applications log correctly, grok will still sequentially try to match the incoming log line against the three expressions, breaking at the first match.

This means that it’s still important to ensure we skip to the right one as fast as possible, since you’ll always have one failed match for Application 2 and two failed matches for Application 3.

The first tactic we often see is to tier the grok matching: first match the header, overwrite the message field, then match only the bodies:

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
This alone provides an interesting performance boost, matching lines 2.5x faster than the initial approach. But what if we add our fellow anchors?

tiered match

Interesting! Adding anchors makes both architectures perform equally well! In fact, because of the greatly increased failed match performance, our initial single grok design performs slightly better since there is one less match being executed.

Ok, so how can I know things aren’t going well?
We’ve already come to the conclusion that monitoring the existence of “_grokparsefaiure” events is essential, but there is more that you can do:

Since version 3.2.0 of the grok plugin, there’s a couple of settings that help you understand when an event is taking a long time to match (or fail to). Using timeout_millis and tag_on_timeout it’s possible to set an upper bound time limit to the execution of the grok match. If this limit is reached, the match is interrupted and the event is tagged by default with _groktimeout.

Using the same conditional strategy we presented before, you can redirect those events to a file or a different index in elasticsearch for later analysis.

Another really cool thing that we will be introducing in Logstash 5.0 in the context of metrics is the ability to extract data on the pipeline performance and, most importantly, per plugin statistics. While logstash is running, you can query its API endpoint and see how much cumulative time logstash is spending on a plugin:

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
With this information, you can see if grok’s “duration_in_millis” is growing rapidly or not and if the number of failures is increasing, which could serve as a warning flag that some pattern is not well designed or consuming more time than expected.

Conclusion
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
