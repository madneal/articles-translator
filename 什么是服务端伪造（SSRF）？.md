# 什么是服务端伪造（SSRF）？

> 原文：[What is Server Side Request Forgery (SSRF)?](https://www.acunetix.com/blog/articles/server-side-request-forgery-vulnerability/)
>
> 译者：[neal1991](https://github.com/neal1991)
>
> welcome to star my [articles-translator](https://github.com/neal1991/articles-translator/), providing you advanced articles translation. Any suggestion, please issue or contact [me](mailto:bing@stu.ecnu.edu.cn)
>
> LICENSE: [MIT](https://opensource.org/licenses/MIT)

服务端伪造（SSRF）指的是攻击者从一个具有漏洞的web应用中发送的一个伪造的请求的攻击。SSRF通常适用于针对在防火墙后一般对于外部网络的攻击者是无法访问的内部系统。另外，攻击者也可能利用SSRF来访问监听回送地址接口（127.0.0.1）的服务。

典型的SSRF发生在web应用发送请求的时候，攻击者对这个发送的请求具有全部或者部分的控制。一个通用的例子就是攻击者能够控制全部或者部分web应用向第三方服务发送请求的URL。

![SSRF](https://user-images.githubusercontent.com/12164075/28745292-2d209ce2-74a8-11e7-9858-214153c97aa2.png)

下面的是PHP中容易收到SSRF的一个例子。

```php
<?php

/**
* Check if the 'url' GET variable is set
* Example - http://localhost/?url=http://testphp.vulnweb.com/images/logo.gif
*/
if (isset($_GET['url'])){
$url = $_GET['url'];

/**
* Send a request vulnerable to SSRF since
* no validation is being done on $url
* before sending the request
*/
$image = fopen($url, 'rb');

/**
* Send the correct response headers
*/
header("Content-Type: image/png");

/**
* Dump the contents of the image
*/
fpassthru($image);
}
```

在上面的例子中，因为攻击者对于url参数具有完整的控制，因此能够对于网上的任何网站都能够发送任意的GET请求。攻击者也能够向服务器中的资源发送请求。

比如，攻击者可能能够访问本地的服务。在下面的例子中，攻击者通过开启`mod_status`（默认开启）能够在Apache HTTP服务器上发送下面的请求。

```php
GET /?url=http://localhost/server-status HTTP/1.1
Host: example.com
```

类似的，SSRF能够被用来请求这个web服务器可以访问的其它内部资源，但是这些资源不是对公共开放的。这个样的例子比如是在[Amazon EC2](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html) 以及 [OpenStack](https://docs.openstack.org/admin-guide/compute-networking-nova.html) 上访问实例元数据。这个服务仅仅向服务器开放而不是外部世界。攻击者甚至会有更多发现通过使用这个方法[在内部网络中运行端口扫描](https://www.acunetix.com/blog/articles/ssrf-vulnerability-used-to-scan-the-web-servers-network/)。

```php
GET /?url=http://169.254.169.254/latest/meta-data/ HTTP/1.1
Host: example.com
```

除了通过`http://`以及·`https://`URL协议，攻击者可能也利用少数人或者遗留的URL协议来访问内部网络的本地系统中的文件。

下面的例子就是利用`file:///`URL协议来发送这样的请求。

```php
GET /?url=file:///etc/passwd HTTP/1.1
Host: example.com
```

根据应用如何产生请求，攻击者能够利用URL协议而不是文件以及HTTP。比如，如果`cURL`被用来产生请求（上面的例子这就是利用`fopen()`来发送请求），可能能够利用`dist URL`协议向任何主机的任何端口发送请求并且发送自定义的数据。

```php
GET /?url=dict://localhost:11211/stat HTTP/1.1
Host: example.com
```

上述的请求会造成应用链接到主机的11211端口并且发送字符串"stat"。端口11211是[Memcached](https://memcached.org/)（通常不会暴露）使用的端口。对于一个可以利用的综合攻击列表以及URL协议，ONSec实验室维护了一个具有关于SSRF攻击有用的[详细文档](https://docs.google.com/document/d/1v1TkWZtrhzRLy0bYXBcdLUedXGb9njTNIJXa3u9akHM)。

## 检测SSRF

为了自动检测SSRF，我们需要依靠中介服务，因为检测到这样一个漏洞需要一个带外和延时的向量。Acunetix通过在自动扫描是讲[AcuMonitor](https://www.acunetix.com/vulnerability-scanner/acumonitor-technology/) 作为它的中介服务来解决这个问题。

在这个扫描期间，Acunetix将会产生一个包含一个特殊AcuMonitor URL的请求。如果AcuMonitor接受到一个包含以上一个特殊URL的请求，它会发送一个通知到Acunetix告诉它应该对于SSRF发出警告。

下面的是Acunetix利用AcuMonitor扫描来检测SSRF的结果。这个警告包含了正在执行的HTTP请求的信息，包括发送这个请求的IP地址以及这个请求使用的User-agent字符串。这个信息可以帮助开发者识别问题的来源并且进行修复。

![image](https://user-images.githubusercontent.com/12164075/28749646-cc68858c-7500-11e7-8d17-25fb43657f1c.png)

## 避免SSRF

### 白名单和DNS解析

直接在用户的输入上实时简单的黑名单或者正则表达式来过滤IP地址或者域发送的这个请求，这对于避免SSRF是一个**坏**方法。

通常，黑名单都是一个糟糕的安全控制因为总是会有开发者忽视的漏网之鱼有。在这样的情况下，攻击者就能够利用这样的旁路来产生HTTP重定向，一个通配符DNS服务比如xip.io或者甚至是[可用的IP编码](http://www.pc-help.org/obscure.htm).

相反，最通用的解决SSRF的方式是使用你的应用需要访问DNS名称或者IP地址的白名单。如果白名单方案不适合你的用户案例，那么你必须依赖黑名单，适当地验证你的用户输入是非常重要的。一个例子就是不允许向私有IP地址（非路由）发送请求（详细参考[RFC 1918](https://tools.ietf.org/html/rfc1918)），然而，在使用黑名单的情况下，正确的采取什么样的避免措施往往取决于具体的应用。换句话说，对于SSRF没有一个通用的修复方法，因为它非常依赖于应用的功能以及商业需求。

### 响应处理

确保响应是远程服务器接收的响应是它应该接收的是非常重要的，这对于阻止响应数据意想不到的泄露给攻击者是非常重要的。以上其他的，无论在任何情况下，服务器发送的原生响应体都不应该直接发送给客户端。

### 关闭无用的URL协议

如果你的应用仅仅使用HTTP或者HTTPS来发送请求，那么应该就仅仅允许这些URL协议。关闭不使用的的URL协议能够阻止web应用使用潜在危险的URL协议，比如`file:///`，`dict://`，`ftp://`以及`gopher`。

### 认证内部服务

服务比如Memcached，Redis，Elasticsearch以及MongoDB默认不需要认证的。SSRF漏洞可以提供给攻击者一个没有任何认证阻拦的机会来访问这些服务。因此，最好实在人地方都使用认证，这也是一个防护机制。

