# Top 5 Commercial SIEM Tools
Jun 14th, 2018 | [Write a comment](#disqusComments) | by Daniel Berman

![commercial siem tools](https://logz.io/wp-content/uploads/2018/06/top_commercial_siem_tools_-_article.jpg)

*   [Home](https://logz.io "Home")
*   >
*   [Blog](https://logz.io/blog/)
*   >
*   [DevOps](https://logz.io/blog/devops/)
*   >
*   **Top 5 Commercial SIEM Tools**

Following our comprehensive introduction to [SIEM systems](https://logz.io/blog/what-is-siem/), we looked at the available [open source SIEM platforms](https://logz.io/blog/open-source-siem-tools/). In this third article in our SIEM series, we review five of the most popular commercial offerings in this space. We evaluate them by looking at their intended audience and market segment, deployment model, SIEM features (threat intelligence, reporting, etc.), and each solution’s pros and cons. At the end of the article, we provide a summary of the reviewed product and make recommendations.

1\. AlienVault: Unified Security Management (USM)
-------------------------------------------------

[AlienVault USM](https://www.alienvault.com) is intended for small and medium organizations. AlienVault can be deployed as both a virtual or hardware appliance, as well as an Amazon Web Services (AWS) based, cloud version of the product. USM provides a comprehensive set of basic SIEM features and functionality, such as event data collection and logging.

USM offers threat intelligence information from the OSSIM community, and from commercial third-party services and vendors. Unlike the other systems we review, USM’s provides only 150 built-in reports. These reports can be customized and cover a range of compliance standards, such as HIPAA and SOX.

USM is a very powerful general solution based on an open architecture. USM not only provides a highly flexible solution that can be modified over time, flexible architecture also allows organizations with constrained budgets to gain control over their deployment and operational costs.

However, USM is still a relatively new and immature product. As such, it lacks features that its competitors provide, such as network forensics, and has less mature functionality in areas, such as event correlation, reporting, and threat intelligence. In addition, as a product aimed at small organizations, installing and configuring USM requires detailed technical knowledge of databases and networking. For organizations without the necessary in-house expertise, this will require hiring external consultants.

2\. Micro Focus: ArcSight ESM
-----------------------------

From a small, niche player, let’s look at a very different animal. [ArcSight ESM](https://software.microfocus.com/en-us/products/siem-security-information-event-management/overview), is a mature SIEM product for large enterprises working in the commercial and regulated sectors. As a result, Micro Focus supports a server-based software deployment model that these organizations are comfortable with. Its basic SIEM features are designed for handling and monitoring a large range and number of data sources in real time. ArcSight ESM provides threat intelligence based on feeds from third-party vendors. Since it is designed to meet the needs of the enterprise, it can report data using dashboards or via a large range of security compliance reports. In addition to reporting, ArcSight also lets organizations act on reported data. The system also provides a machine learning option that can be used to process collected information, analyze and present insights.

For large organizations, ArcSight ESM has much to recommend it. For a start, it is a mature product from a company with a long history of building and supporting enterprise software. As a product, ArcSight ESM provides a complete solution for corporations with large national and global infrastructures. Moreover, it not only detects and investigates potential and actual threats, it also includes the necessary tools to handle these problems. While ArcSight aims to provide a complete enterprise SIEM solution, there are a number of areas in which it is deficient. Currently, it does not provide network forensics, just limited host logging, and has a confusing user interface. It is also an expensive product that requires extensive support and training.

3\. IBM Security: QRadar
------------------------

IBM is another big, established enterprise software company that serves the same market segment as MicroFocus. That being said, there are a number of key differences between ArcSight ESM and [QRadar](https://www.ibm.com/security/security-intelligence/qradar). The first noticeable difference is that the product can be deployed as hardware, software, or a virtual appliance. QRadar threat intelligence can use both open feed, and the IBM Security X-Force paid subscription service. QRadar provides a similar range of security compliance initiatives such as ArcSight, but also includes a Report Builder Wizard. Perhaps QRadar’s biggest differentiator is not the number of features it offers, but its extensibility. If the basic product doesn’t have what you need out of the box, you can choose from modules for data ingestion, risk management, vulnerability management, forensics analysis, and incident response. QRadar also supports integrations with third-party products.

QRadar is a modular, scalable, appliance-based SIEM solution. It is designed to be both robust and monitor the volume and velocity of data that an enterprise system must handle. Overall it’s a good solution, but it’s not perfect. Many users find QRadar overwhelming. It already has a large feature set, and additional modules have made the system even more complex and overwhelming. Many of the extra features and modules are the result of acquisitions by IBM, and need better integration with the host system. Like many IBM products, it is expensive, and not particularly user friendly. While QRadar has powerful search functionality, many users may not be able to take advantage of them without knowledge of regular expressions.

4\. Splunk Enterprise
---------------------

Splunk specializes in the commercial enterprise and regulated industries markets. Its [Splunk Enterprise](https://www.splunk.com/en_us/software/splunk-enterprise.html) product can be deployed locally or as a cloud service. It provides the standard, basic SIEM and reporting features, but lacks many of the advanced features available from other vendors’ products. However, it is by far the most flexible and extensible of the solutions reviewed in this article. In fact, many of these missing features are available as add-ins. From third-party tools that ingest threat intelligence data, reporting tools, and other advanced security-related functionality. In addition, you can build your own add-on with its Enterprise Security App framework.

Apart from being highly extensible, Splunk Enterprise is able to create a central repository of monitored and log data. This centralized data can be analyzed with a range of tools to provide new insights and correlations. The central repository can then be connected to additional data sources and searched with easy-to-use tools. The system also provides security controls and filters that let administrators restrict user access to sensitive information. Splunk’s flexibility is also its biggest drawback. As we have already noted, the base system lacks core functionality provided by its competitors. Adding missing functionality requires additional extensions that then need to be configured.

5\. LogRhythm
-------------

[LogRhythm](https://logrhythm.com/) is another small player servicing medium-to-large enterprises. Its solution also provides a number of deployment options including bundled or distributed components, hardware-based appliances, server-based software, and virtual appliances. LogRhythm has many features that its competitors lack. LogRhythm’s threat detection not only uses threat intelligence feeds, it also detect threats based on geolocation. It monitors and detects security events at system endpoints, on a host PC’s file system, and can monitor process execution. For PCs, it can also monitor the Windows’ registry. As with the standard version, it also has two additional advantages over other systems. First, it has built-in network forensics. Second, it supports eight hundred report formats, more than any other solution in this article.

Overall, LogRhythm is a compelling product. It has a flexible, decentralized architecture, and the widest range of security features and reporting capabilities. As good as it is, it is a powerful and large-scale solution that is designed for big companies that can afford its subscription pricing model. After installing the system, it requires extensive application and networking configuration both of which require expert knowledge. In addition, most operations can only be performed using a native client. LogRhythm does provide a web console, but this only provides a subset of the native clients’ features and functionality.

Conclusion: All Things Considered
---------------------------------

We have looked at five very different systems and compared them according to a common set of criteria.

AlienVault USM is a powerful general solution and suited for small-to-medium organizations; however, it is missing features and functionality and is difficult to configure. Micro Focus ArcSight is a complete solution for corporate infrastructure that detects threats and lets you take action; but, it is expensive and overly complicated. IBM Security QRadar is a robust, scalable, and extensible solution; but, it is overwhelming and not user-friendly. Splunk Enterprise is a highly extensible system with advanced data management and analysis; but, requires add-ons and extensive customization to provide features and functionality. LogRhythm provides the widest range of security features, and reporting capabilities; but, it is expensive and requires configuration.

Pick a system based on your companies’ current size and needs. For small organizations, AlienVault USM is probably the best fit. If you need a mature solution from an enterprise vendor, your best bet is ArcSight ESM. However, if you need a robust and scalable enterprise system, QRadar is a better choice. Splunk’s limited feature and flexible extensibility make it the ideal choice for companies that want to build a system that they can customize to their needs, it also has the best correlation and analysis tools of the systems we’ve reviewed.

Finally, LogRhythm is a good choice for enterprises that require extensive detection and reporting tools and can afford an expensive solution.

Looking for a secure ELK solution? Try Logz.io!

[Learn More](https://logz.io/product/soc-2-compliance/)

[Share](http://www.facebook.com/share.php?u=https%3A%2F%2Flogz.io%2Fblog%2Ftop-5-commercial-siem-tools%2F)

[Tweet](https://twitter.com/intent/tweet?text=Top+5+Commercial+SIEM+Tools&url=https%3A%2F%2Flogz.io%2Fblog%2Ftop-5-commercial-siem-tools%2F)

[Share](https://www.linkedin.com/cws/share?url=https%3A%2F%2Flogz.io%2Fblog%2Ftop-5-commercial-siem-tools%2F)

Tags:

*   [SIEM](https://logz.io/tag/siem/)
