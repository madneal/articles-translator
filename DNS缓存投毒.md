
https://medium.com/iocscan/dns-cache-poisoning-bea939b5afaf

## DNS 缓存投毒

![](https://cdn-images-1.medium.com/max/2000/1*3kt_t9wSc7MP4Gu2rD7_gw.png)

DNS Spoofing is the result of alterations to a DNS server’s records resulting in the malicious redirection of traffic. DNS spoofing can be performed by a direct attack on the DNS server (what we will be talking about here) or through any form of a Man-in-the-Middle attack specifically targeting DNS traffic.

DNS Cache spoofing works explicitly in a way that exploits the way in which DNS communication is structure. When a DNS server attempts to perform a lookup on a domain, it will forward the request along to the root authoritative DNS and iteratively proceed down the chain of DNS servers until it reaches the DNS server authoritative over the domain. Since the local DNS server does not know which server is in charge of which domain, and does not know the full route to each authoritative server, it accepts replies to its queries from anywhere so long as the reply matches the query and is formatted correctly. The attacker can exploit this design by beating the actual Authoritative DNS server in replying to the local DNS server, and if it does so, the local DNS server will use the attacker’s DNS record instead of the actual Authoritative answer. Due to the nature of DNS, the local DNS server has no way of determining which reply is real and which is fake.

This attack is made worse by the fact that DNS servers will cache lookups internally so that they don’t have to waste time querying the authoritative servers each time the domain is requested. This poses another problem, because if an attacker can beat the Authoritative DNS server to a reply, then the attackers record will be cached by the local DNS server meaning that any user that uses the local DNS server will be given the attackers record, potentially redirecting all users using that local DNS server to the attacker’s website.

DNS 欺骗是 DNS 服务器记录更改导致恶意重定向流量的结果。 DNS 欺骗可以通过直接攻击 DNS 服务器（我们将在这里讨论）或通过任何形式的专门针对 DNS 流量的中间人攻击来执行。

DNS 缓存欺骗以一种利用 DNS 通信结构的方式明确地工作。当 DNS 服务器尝试在域上执行查找时，它会将请求转发到根权威 DNS，并迭代地沿着 DNS 服务器链向下行进，直到它到达域上的权威 DNS 服务器。由于本地 DNS 服务器不知道哪个服务器负责哪个域，并且不知道到每个权威服务器的完整路由，因此只要回复与查询匹配并且格式正确，它就会从任何地方接受对其查询的回复。攻击者可以通过在回复本地 DNS 服务器时击败实际的权威 DNS 服务器来利用此设计，如果这样做，本地 DNS 服务器将使用攻击者的 DNS 记录而不是实际的权威答案。由于 DNS 的性质，本地 DNS 服务器无法确定哪个回复是真实的，哪个是假的。

由于 DNS 服务器将在内部缓存查询，因此每次请求域时，他们不必浪费时间查询权威服务器，从而加剧了这种攻击。这带来了另一个问题，因为如果攻击者可以击败权威DNS 服务器进行回复，那么攻击者记录将被本地 DNS 服务器缓存，这意味着任何使用本地DNS服务器的用户都将获得攻击者记录，可能会重定向所有使用该本地 DNS 服务器的用户都可以访问攻击者的网站。

![DNS Cache Poisoning Workflow](https://cdn-images-1.medium.com/max/2000/1*iBZM5hvnXRvelyJ0E1KRig.png)

## DNS 缓存投毒的例子

**生日攻击的盲目响应伪造**

A DNS protocol exchange does not authenticate responses to recursive iterative queries. The only checks to validate the query are the 16-bit transaction ID and the source IP address and destination port of the response packet. Before 2008, all DNS revolvers used fixed port 53. Therefore with the exception of the transaction ID, all information necessary to spoof a DNS reply is predictable. Attacks on DNS exploiting this weakness are known as the “birthday paradox” and on average take ²⁸ or 256 attempts to guess the transaction ID. For the attack to succeed, the forged DNS reply must arrive at the target resolver before the legitimate authoritative response does. If the legitimate response arrives first, it will be cached by the resolver, and until its time-to-live (TTL) expires, the resolver will not ask the authoritative server to resolve the same domain name, preventing the attacker from poisoning the mapping for that domain until the TTL expires.

DNS协议交换不验证对递归迭代查询的响应。 验证查询的唯一检查是16位事务ID以及响应数据包的源IP地址和目标端口。 在2008年之前，所有DNS左轮手枪都使用固定端口53.因此，除了事务ID之外，欺骗DNS回复所需的所有信息都是可预测的。 利用这种弱点攻击DNS被称为“生日悖论”，平均需要⁸或256次来猜测交易ID。 为了使攻击成功，伪造的DNS回复必须在合法权威响应之前到达目标解析器。 如果合法响应首先到达，它将由解析器缓存，并且直到其生存时间（TTL）到期，解析器将不会要求权威服务器解析相同的域名，从而防止攻击者中毒映射 对于该域，直到TTL到期。

**Kaminsky 漏洞**

An extension of the Birthday Attack was revealed at Black Hat 2008, where the basic blind guessing technique remains the same. The attack exploits the underlying nature of DNS responses in that a DNS response can be an answer (the direct IP address of the request) or a referral (the server that is authoritative over a given zone). The Birthday Attack forges an answer that injects a false entry for a given domain record. The Kaminsky exploit utilizes a referral to make a false entry for an entire domain bypassing TTL on previous entries. The basic idea is that the attacker chooses the domain that they wish to compromise and then queries the target resolver for a sub-domain which is not already cached by the resolver (Targeting non-existent sub-domains is a good bet that the record is not cached by the DNS resolver). Since the sub-domain is not in the cache, the target resolver sends a query to the authoritative server for that domain. It is at this point that the attacker floods the resolver with a large number of forged responses, each with a different forged transaction ID number. If the attacker is successful in the injection of the forged response, then the resolver will cache a false mapping for an authoritative server. Future DNS queries to the target resolver of the compromised domain will result in all requests being forwarded to the attacker controller authoritative resolver enabling the attacker to provide malicious responses without a need to inject fake entries for each new DNS record.


生日攻击的拓展在2008年 Black Hat 得到了揭示，其中基本的盲猜技术保持不变。该攻击利用了 DNS 响应的基本特性，因为 DNS 响应可以是答案（请求的直接 IP地址）或引用（对给定区域具有权威性的服务器）。生日攻击伪造了一个为给定域记录注入错误条目的答案。 Kaminsky 漏洞使用引用来绕过先前条目上的 TTL 对整个域进行错误输入。基本思想是攻击者选择他们希望攻击的域，然后向目标解析器查询尚未被解析器缓存的子域（定位不存在的子域是一个很好的选择，记录是没有被DNS解析器缓存）。由于子域不在缓存中，因此目标解析器向该域的权威服务器发送查询。正是在这一点上，攻击者用大量伪造的响应来淹没解析器，每个伪造的响应都有不同的伪造交易ID号。如果攻击者成功注入伪造响应，则解析器将为权威服务器缓存错误映射。对受感染域的目标解析器的未来DNS查询将导致所有请求被转发到攻击者控制器权威解析器，使攻击者能够提供恶意响应，而无需为每个新DNS记录注入假条目。

**Eavesdropping**
Many new proposals for enhancing DNS security include source port randomization, 0x20 XOR encoding, WSEC-DNS, all depend on the asymmetrical accessibility of the components used for authentication. In other words, they provide security through obscurity rather than confidentiality through authentication and cryptography. Their only goal is to prevent BLIND attacks as discussed above. Using these security methods still leaves DNS vulnerable to trivial attacks of compromised servers and network eavesdroppers to break the obscurity and perform the same attacks as seen above, this time without blind guessing. Even in switched environments, ARP poisoning and similar techniques can be used to force all packets to a malicious computer and can defeat the obscurity.

## DNS Cache Poisoning Mitigation

**DNSSEC**
The best way to prevent Cache Poisoning of DNS revolvers is to implement a secure method of cryptography and authentication. DNS as a protocol is antiquated and as a backbone of the entire internet is surprisingly still an unencrypted protocol without any form of validation for entries and responses it receives.

The solution, of course, is to provide a method of verification and authentication known as DNS Secure or [DNSSEC](https://medium.com/iocscan/how-dnssec-works-9c652257be0). The protocol creates a unique cryptographic signature stored alongside the DNS records. The signature is then used by the DNS resolver to authenticate a DNS response ensuring the record was not tampered with. Additionally, it provides a chain of trust from the TLD all the way down to the domain authoritative zone, ensuring the whole process of DNS resolution is secured.

Despite these apparent benefits, DNSSEC has been slow to be adopted, and many of the less popular TLDs still do not make use of the security DNSSEC provides. The main issue is that DNSSEC is complex to set up and requires upgraded appliances to handle the new protocol, additionally due to the rarity and impotence of most DNS spoofing attacks historically, implementation of DNSSEC is not seen as a priority and is usually only performed once appliances reach the end of their life cycle.
