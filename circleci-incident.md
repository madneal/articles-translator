# CircleCI 20230104 安全事件报告

>原文：[CircleCI incident report for January 4, 2023 security incident(https://circleci.com/blog/jan-4-2023-incident-report/)
>
>译者：[madneal](https://github.com/madneal)
>
>welcome to star my [articles-translator](https://github.com/madneal/articles-translator/), providing you advanced articles translation. Any suggestion, please issue or contact [me](mailto:bing@stu.ecnu.edu.cn)
>
>LICENSE: [MIT](https://opensource.org/licenses/MIT)

[On January 4, 2023, we alerted customers](https://circleci.com/blog/january-4-2023-security-alert/) to a security incident. Today, we want to share with you what happened, what we’ve learned, and what our plans are to continuously improve our security posture for the future.

We would like to thank our customers for your attention to rotating and revoking secrets, and apologize for any disruption this incident may have caused to your work. We encourage customers who have yet to take action to do so in order to prevent unauthorized access to third-party systems and stores. Additionally, we want to thank our customers and our community for your patience while we have been conducting a thorough investigation. In aiming for responsible disclosure, we have done our best to balance speed in sharing information with maintaining the integrity of our investigation.

**This report will cover:**

* What happened?
* How do we know this attack vector is closed and it’s safe to build?
* Communication and support for customers
* How do I know if I was impacted?
* Details that may help your team with internal investigations
* What we learned from this incident and what we will do next
* A note on employee responsibility vs. systems safeguards
* Security best practices
* Closing thoughts

## What happened?

*All dates and times are reported in UTC, unless otherwise noted.*

On December 29, 2022, we were alerted to suspicious GitHub OAuth activity by one of our customers. This notification kicked off a deeper review by CircleCI’s security team with GitHub.

On December 30, 2022, we learned that this customer’s GitHub OAuth token had been compromised by an unauthorized third party. Although that customer was able to quickly resolve the issue, out of an abundance of caution, on December 31, 2022, we proactively initiated the process of rotating all GitHub OAuth tokens on behalf of our customers. Despite working with GitHub to increase API rate limits, the rotation process took time. While it was not clear at this point whether other customers were impacted, we continued to expand the scope of our analysis.

By January 4, 2023, our internal investigation had determined the scope of the intrusion by the unauthorized third party and the entry path of the attack. To date, we have learned that an unauthorized third party leveraged malware deployed to a CircleCI engineer’s laptop in order to steal a valid, 2FA-backed SSO session. This machine was compromised on December 16, 2022. The malware was not detected by our antivirus software. Our investigation indicates that the malware was able to execute session cookie theft, enabling them to impersonate the targeted employee in a remote location and then escalate access to a subset of our production systems.

Because the targeted employee had privileges to generate production access tokens as part of the employee’s regular duties, the unauthorized third party was able to access and exfiltrate data from a subset of databases and stores, including customer environment variables, tokens, and keys. We have reason to believe that the unauthorized third party engaged in reconnaissance activity on December 19, 2022. On December 22, 2022, exfiltration occurred, and that is our last record of unauthorized activity in our production systems. Though all the data exfiltrated was encrypted at rest, the third party extracted encryption keys from a running process, enabling them to potentially access the encrypted data.

While we are confident in the findings of our internal investigation, we have engaged third-party cyber security specialists to assist in our investigation and to validate our findings. Our findings to date are based on analyses of our authentication, network, and monitoring tools, as well as system logs and log analyses provided by our partners.

In response to this incident, we took the following actions:

* On January 4, 2023, at 16:35 UTC, we shut down all access for the employee whose account was compromised.
* On January 4, 2023, at 18:30 UTC, we shut down production access to nearly all employees, limiting access to an extremely small group for operational issues. At no time during this investigation did we have any evidence that the credentials of any other employees or their devices had been compromised, but we took this action in order to limit the potential surface area.
* On January 4, 2023, at 22:30 UTC, we rotated all potentially exposed production hosts to ensure clean production machines.
* On January 5, 2023, at 03:26 UTC, we revoked all Project API Tokens.
* On January 6, 2023, at 05:00 UTC, we revoked all Personal API Tokens that had been created prior to 00:00 UTC on January 5, 2023.
* On January 6, 2023, at 06:40 UTC we began work with our partners at Atlassian to rotate all Bitbucket tokens on behalf of our customers. This work was completed on January 6, 2023, at 10:15 UTC.
* On January 7, 2023, at 07:30 UTC, we completed the rotation of GitHub OAuth tokens that we began on December 31, 2022, at 04:00 UTC.
* On January 7, 2023, at 18:30 UTC, we began working with our partners at AWS to notify customers of potentially affected AWS tokens. We understand that those notifications were complete as of January 12, 2023, at 00:00 UTC.
In the intervening time, we have continued our forensic investigation with the support of external investigators, rolled out additional layers of security on our platform, and built and distributed additional tools (more details below) to support our customers in securing their secrets.


## How do we know this attack vector is closed and it’s safe to build?

We are confident that customers can safely build on CircleCI.

We have taken many steps since becoming aware of this attack, both to close the attack vector and add additional layers of security, including the following:

* Added detection and blocking through our MDM and A/V solutions for the specific behaviors exhibited by the malware used in this attack.
* Restricted access to production environments to a very limited number of employees as we implement additional security measures. We’re confident in our platform’s security, and we have no indication that any other employee’s device has been compromised.
* For the employees who retain production access, we have added additional step-up authentication steps and controls. This will help us prevent possible unauthorized production access, even in the case of a stolen 2FA-backed SSO session.
* Implemented monitoring and alerting for the specific behavior patterns we identified in this scenario, across multiple triggers and via a variety of third-party vendors.

We know that security work is never done. In addition to closing this particular vector, we have also performed enhanced and ongoing reviews to ensure a stronger defense against potential attacks.


## Communication and support for customers

Upon completion of the rotation of all production hosts on January 4, 2023 at 22:30 UTC, we were confident that we had eliminated both the attack vector and the potential of a lingering corrupted host.

On January 4, 2023, at 6:30 PM PST / January 5, 2023, at 02:30 UTC, we sent disclosure emails, [posted a security notification on our blog](https://circleci.com/blog/january-4-2023-security-alert/), notified customers via our social media accounts and our Discuss forum, and created a support article on how to perform the recommended security steps.

We recommended that all customers rotate their secrets, including OAuth tokens, Project API Tokens, SSH keys, and more (see blog post or Discuss post for more details).

This disclosure initiated an active and ongoing period of communication with our customers. We would like to thank our customers both for your concerted response to this incident as well as for helping us to identify opportunities to provide you with additional tooling. We took this feedback and in response built and released new tools and amended our existing tools to speed remediation for customers via the following:

* A [script for secret finding](https://github.com/CircleCI-Public/CircleCI-Env-Inspector) in order to create an actionable list for secrets rotation.
* Two key changes to the CircleCI API:
    * New functionality to return SHA-256 signatures for checkout keys in order to better match the GitHub UI.
    * The `updated_at` field was added to the Contexts API so that customers can verify the successful rotation of these variables.
* Audit logs were made accessible to all customers on both free and paid plans to help customers review CircleCI platform activity.

We appreciate all feedback from customers about where we could improve our communications including opportunities to make the incident more visible across our channels.


## How do I know if I was impacted?

### Is my data at risk?

In this incident, the unauthorized actor exfiltrated customer information on December 22, 2022, which included environment variables, keys, and tokens for third-party systems. If you stored secrets on our platform during this time period, assume they have been accessed and take the recommended mitigation steps. We recommend you investigate for suspicious activity in your system starting on December 16, 2022 and ending on the date you completed your secrets rotation after our disclosure on January 4, 2023. Anything entered into the system after January 5, 2023 can be considered secure.

### Did an unauthorized actor use that data to access any of my systems?

Because this incident involved the exfiltration of keys and tokens for third-party systems, there is no way for us to know if your secrets were used for unauthorized access to those third-party systems. We have provided some details below to aid customers in their investigations.

**At the time of publishing, fewer than 5 customers have informed us of unauthorized access to third-party systems as a result of this incident.**


## Details that may help your team with internal investigations

With the help of third-party forensic investigators, we have recently confirmed additional details that may help customers in their audits and investigations.

### Dates of impact:

* We see unauthorized third-party access on December 19, 2022, with exfiltration of data occurring on December 22, 2022.
* We have no evidence of customer-impacting activity prior to December 19, 2022. Out of an abundance of caution, we recommend you investigate the period between the date of corruption on December 16, 2022 through the date when you rotated your secrets following January 4, 2023 for unusual activity in your systems.

### IP addresses identified as being used by the threat actor:

* 178.249.214.10
* 89.36.78.75
* 89.36.78.109
* 89.36.78.135
* 178.249.214.25
* 72.18.132.58
* 188.68.229.52
* 111.90.149.55

### Data centers and VPN providers identified as being used by the threat actor:

* Datacamp Limited
* Globalaxs Quebec Noc
* Handy Networks, LLC
* Mullvad VPN

### Malicious files to search for and remove:

* /private/tmp/.svx856.log
* /private/tmp/.ptslog
* PTX-Player.dmg (SHA256: 8913e38592228adc067d82f66c150d87004ec946e579d4a00c53b61444ff35bf)
* PTX.app

### Block the following domain:

* potrax[.]com

### Review GitHub audit log files for unexpected commands such as:

* repo.download_zip

## What we learned from this incident and what we will do next

**We learned: While we have the tools in place to deter and detect attacks, there is always opportunity to strengthen our security posture.**

The authentication, security, and tracing tools we had in place allowed us to comprehensively diagnose and remediate the issue. As the sophistication of malicious actors increases, we are continuously evolving our security standards and pushing best practices to stay ahead of future threats. We will be increasingly aggressive in our use of security tools. Going forward, to support a more conservative stance and preclude bad actors from improperly accessing our systems, we will be optimizing the configuration of our existing tools to create additional layers of defense.

### Our plan:

To start, we will initiate periodic automatic OAuth token rotation for all customers. Our plans also include a shift from OAuth to GitHub apps, enabling us to enforce more granular permissions within tokens. We also plan to complete a comprehensive analysis of all of our tooling configurations, including a third-party review. We’re continuing to take additional steps including expanding alerting, reducing session trust, adding additional authentication factors, and performing more regular access rotations. Finally, we will be making our system permissions more ephemeral, severely restricting the target value of any tokens gained from a similar incident.

### We learned: We can make it easier for customers to adopt our most advanced security features.

Through the evolution of CircleCI, we have continued to introduce features to improve the security of our customers’ build pipelines. While advanced security features are available to customers, we can do more to improve adoption of those features.

### Our plan:

It must be easier for customers to seamlessly adopt the latest and most advanced security features available, including OIDC and IP ranges. There are also additional proactive steps we are exploring, for example, automatic token expiration and notifications for unused secrets. We will make it simpler and more convenient for our customers to create and maintain highly secure pipelines, enabling every advantage of the cloud while smartly managing risk.

## A note on employee responsibility vs. systems safeguards

We want to be clear. While one employee’s laptop was exploited through this sophisticated attack, a security incident is a systems failure. Our responsibility as an organization is to build layers of safeguards that protect against all attack vectors.

## Security best practices

Given the increasing presence of highly sophisticated and motivated bad actors, we are committed to sharing best practices with our customers in order to strengthen our collective defense against inevitable future attempts. Here are recommendations customers can take to increase pipeline security:

* Use [OIDC tokens](https://circleci.com/docs/openid-connect-tokens/) wherever possible to avoid storing long-lived credentials in CircleCI.
* Take advantage of [IP ranges](https://circleci.com/docs/ip-ranges/) to limit inbound connections to your systems to only known IP addresses.
* Use [Contexts](https://circleci.com/docs/contexts/) to consolidate shared secrets and restrict access to secrets to specific projects, which can then be [rotated automatically via API](https://circleci.com/docs/contexts/#rotating-environment-variables).
* For privileged access and additional controls, you may choose to use [runners](https://circleci.com/docs/runner-overview/#circleci-runner-use-cases), which allow you to connect the CircleCI platform to your own compute and environments, including IP restrictions and IAM management.

## Closing thoughts

We know there is no convenient time to respond to a security incident on a critical system, and we want to sincerely thank all of our customers who immediately took action following this incident. It is through this collective action that we will be in a stronger position to combat future threats. We also witnessed firsthand the strength and generosity of our customer community, with many of you heading to our Discuss forum to share knowledge and help one another. Thank you for your support and patience as we worked to resolve this incident.