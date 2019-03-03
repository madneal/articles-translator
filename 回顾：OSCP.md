# Review: Offensive Security 认证专家 (OSCP) / 基于 Kali Linux 的渗透测试 (PWK)

>原文：[Review: Offensive Security Certified Professional (OSCP) / Penetration Testing with Kali Linux (PWK)](https://aminbohio.com/offensive-security-certified-professional-oscp-penetration-testing-with-kali-linux-pwk-review/)
>
>译者：[neal1991](https://github.com/neal1991)
>
>welcome to star my [articles-translator](https://github.com/neal1991/articles-translator/), providing you advanced articles translation. Any suggestion, please issue or contact [me](mailto:bing@stu.ecnu.edu.cn)
>
>LICENSE: [MIT](https://opensource.org/licenses/MIT)

![](https://aminbohio.com/storage/2019/02/oscp-emblem.png)

Offensive Security has been providing the best security courses and certifications in the industry for a very long time now. I have been a fan of their work ever since I came to know about them which was back in 2010. I have been following their certification path in a very non usual way. I usually see people starting out with OSCP/OSWP and then move on to OSCE or other courses however I took OSCE first which was back in October. I wrote a review and a study guide for it which has been helpful to many people. I decided to take OSCP because of various reasons but mostly because I was convinced by many people how different and great experience it was for them. This is a review and my experience of how it was. I won’t be giving any hints or give out any solutions but if this still does interest you, read ahead!

Offensive Security 已经在很长一段时间内提供业内最好的安全课程和认证。自从我在 2010年 开始了解他们以来，我一直是他们工作的粉丝。我一直在以不平常的方式遵循他们的认证路径。我通常会看到人们从 OSCP/OSWP 开始，然后转到 OSCE 或其他课程，但我首先考虑了 10 月份的 OSCE。我写了一篇回顾和一份学习指南，对很多人都很有帮助。由于种种原因，我决定参加 OSCP，但主要是因为很多人都相信我对他们有多么不同和丰富的经验。这是一篇回顾和我的经历。我不会给出任何提示或给出任何解决方案，但如果这仍然让你感兴趣，请继续阅读！

## 介绍

Penetration Testing with Kali Linux (PWK) is the most popular course offered by Offensive Security which when completed and passing the exam, gives you the OSCP certificate. It has a very high regard in the Information Security industry. An OSCP, by definition, is able to identify existing vulnerabilities and execute organized attacks in a controlled and focused manner, write simple Bash or Python scripts, modify existing exploit code to their advantage, perform network pivoting and data ex-filtration, and compromise poorly written PHP web applications.

基于 Kali Linux 进行渗透测试（PWK）是 Offensive Security 提供的最受欢迎的课程，在完成并通过考试后，会为你提供 OSCP 证书。 它在信息安全行业备受推崇。 根据定义，OSCP 能够以受控且集中的方式识别现有漏洞并执行有组织的攻击，编写简单的 Bash 或 Python 脚本，修改现有漏洞利用代码，执行网络透视和数据过滤，以及攻击编写得差得 PHP Web 应用程序。

## 注册流程

Unlike OSCE/CTP, you are not required to complete a registration task before signing up for the course. Offensive Security asks for the following from the students taking PWK and they are not joking!

与 OSCE/CTP 不同，你无需在注册课程之前完成注册任务。 Offensive Security 要求接受 PWK学生遵守以下内容，他们不是在开玩笑！

**学生要求**：

* 学生需要熟悉 Linux 环境。我们将使用 Kali Linux 作为整个课程的攻击平台。浏览目录，执行脚本和工具以及编写基本的 bash 脚本是所有学生所需要的技能。
* 对于 TCP/IP 以及多种网络服务（DNS, DHCP 等）有较强的理解。
* 了解脚本语言（Perl，Python, Ruby）是推荐的，但不是必须的。
* 需要对信息安全细节以及概念有较强的理解。
* 耐心并且能够坚持。这不是开玩笑，我们是想让你通过。

![](https://aminbohio.com/storage/2019/02/2019-02-15T1630060500.png)

Following is the pricing structure for PWK

下面是 PWK 的价格介绍

![](https://aminbohio.com/storage/2019/02/2019-02-15T1559160500.png)

I personally went with 30 days labs as I felt confident enough to put in the required time but I would recommend 60 or 90 days labs depending upon the experience you have being a penetration tester. If you are good with scripting, logic building, and have worked with exploits and shells, you are good to go with 30 days. To get the most out of this course, I would encourage you to put in maximum time in the labs and try everything you can.

我自己注册了 30 天的 lab，因为我有足够的信心投入所需的时间，但我建议 60 或 90天 的 lab，这取决于你渗透测试的经验。如果你擅长脚本编写，逻辑构建，并且已经使用了漏洞利用程序和 shell，那么你可以使用 30 天。 为了充分利用这门课程，我鼓励你在 lab 中投入最多时间并尝试一切。

## 课程资料

Once you have registered yourself, it takes about 15 days for your course work to start. You get downloadable course material which comprises of a PDF book and a set of videos both of which are watermarked with your name and OSID. You are not allowed to share this course material with anyone at all because you can be permanently banned for doing so.

The PDF guide and the video tutorials are almost identical but the PDF guide has much more detailing to cover the topics in a very good depth. The course in itself is very nicely designed but you are assumed to know a lot of things beforehand like Networking, Linux, etc.

Although I cannot share the exact content with you, you can get an idea about what to expect in the course and the labs through this syllabus made available by Offensive Security.

你一旦注册了，你的课程大概会 15 天后开始。你可以获得可下载的课程资料，其中包含 PDF 书籍和一组视频，这些视频都带有你的姓名和 OSID 的水印。你不得与任何人共享此课程资料，因为这是被永久禁止的行为。

PDF 指南和视频教程几乎完全相同，但 PDF 指南有更多细节，以深入介绍主题。该课程本身设计得非常好，但你可以事先了解很多事情，比如网络，Linux等。

虽然我无法与你分享确切的内容，但您可以通过 Offensive Security 提供的课程大纲了解课程和实验中的内容。

Penetration Testing: What You Should Know
Getting Comfortable with Kali Linux
The Essential Tools
Passive Information Gathering
Active Information Gathering
Vulnerability Scanning
Buffer Overflows
Win32 Buffer Overflow Exploitation
Linux Buffer Overflow Exploitation
Working with Exploits
File Transfers
Privilege Escalation
Client Side Attacks
Web Application Attacks
Password Attacks
Port Redirection and Tunneling
The Metasploit Framework
Bypassing Antivirus Software
Assembling the Pieces: Penetration Test Breakdown

## Labs

The labs are nicely designed to simulate a real corporate network where you have different departments and several machines across multiple networks behind a firewall and you have to compromise them. Following is the network diagram of the lab network where you practice and implement the tricks taught in the course material.

![](https://aminbohio.com/storage/2019/02/pwk-lab-network.png)

Students are given a VPN pack through which you login to these labs and the initial starting point is through the Public Network where you have got several machines waiting to be pawned. The end goal is to reach the administrative department by exploiting and gaining access to other networks.

The lab network contains a large number of machines which range in difficulty. Every machine that you compromise gives you a proof.txt file which contains a unique flag. Some of the lab machines contain a network-secret.txt which contains a flag which needs to be submitted in the lab control panel to unlock other networks.

I had to complete my labs alongside my 9 to 5 job which made the process very hard and I had to put in a lot of sleepless nights to make sure I did not miss any fun. It took me about 27 or 28 days in total to completely gain access to all networks and machines with the only exception of a single machine which took a lot of time and in the end my lab expired before I could get administrative access to that machine but rest assured, the labs were real fun and interesting and I would recommend anyone taking OSCP to put as much time as they can in the labs.

## Proctored Exam

The exam starts exactly 15 minutes before your scheduled time to verify your identity in front of the proctor. You need to make sure to setup your webcam, microphone, and Kali VM ready before the exam. You need to have Java 8 or 9 installed in order for the Screen Connect to work which is the software they use in order to proctor you. You are asked to scan the entire room with the webcam and provide your ID card and also run a few scripts inside the VM before you can begin your exam. Once that is done, you get the email with your connectivity pack and login credentials. You are also given a set of targets to compromise.

Once the proctor has verified your identity and can see you through the webcam, you are asked to begin the exam. The webcam feed does stop from time to time and they ask you to refresh the application again and again which is a little annoying but you cannot continue without it because the VPN connection is disconnected if your webcam feed has gone off for too long.

The exam consists of several targets and objectives. After reading several reviews on the internet, I knew that I had to grab the low hanging fruit first which was a machine with the lowest points and I was able to get administrative access on it within an hour after the scans. I then moved on to the machine which has one of the highest points and requires a lot of work and requires attention. I was very comfortable doing this one because of what I had faced in OSCE but still took me an hour and a half to do this. After that, I went on to get my hands on 2 other machines having the same points and was stuck for a while before I got into one of them but it was probably due to not taking a break which is something I had promised myself that I would take after every 2 hours. These machines were fairly easy to get administrative access on once I had code execution on them. This left only the last one which was very hard to get root on and in the end, I gave up on it as I already had enough points to pass the exam. Little did I KNOW!

I will not lie to you and tell you that I passed the exam first time around because I did not. It was not because I could not compromise the exam machines or complete tasks but rather that I did not follow rules outlined by Offensive Security. They strictly tell you not to use Metasploit in the exam which is something I missed from the rules the first time around and even though I got the perfect score, I failed. Offensive Security’s response when I sent them the query why I failed

![](https://aminbohio.com/storage/2019/02/2019-02-15T1720040500.png)

While I was upset to hear that, I did not let that stop me for taking it again and this time around, I promised myself not to touch metasploit and do everything manually. Just like the previous exam, the objectives were clear and I followed the same path but this time, I made sure not to use metasploit even for a bit. With 8 hours in, I had completed the exam with a perfect score 100/100 and by the 13th hour, I had completed the report for the exam. I submitted the exam report after telling the proctor to end my exam and waited for the mail from offsec!

And I got the mail on Tuesday, 12th February around 2:00 AM in the morning telling me that I passed the certification exam.

![](https://aminbohio.com/storage/2019/02/2019-02-15T1517280500.png)
