
## What the HELK? SIGMA integration via Elastalert

![](https://cdn-images-1.medium.com/max/5524/1*Pn_nk6oHSpmRe1vFgtybZg.png)

Usually, while performing research on detecting new adversary techniques, I wonder if I would be able to find specific events that would allow me to easily detect the malicious activity in production right away, or If I would have to perform deeper analysis, and come up with other data analytics to compensate and enhance my detection approach. Sometimes, it is the former, and if I am able to define the specific event or a combination of events in a form of a rule that could trigger a high-fidelity alert or let me know when certain activity occurs, then there is not need for me to constantly run it manually. One of the objectives for a threat hunting program is to enhance the current detection capabilities of an organization, and usually this is done by providing context for the development of high-fidelity alerts or enhance current rules to monitor for potential adversarial activity.

The concept of developing rules is not new; however, being able to share rules via a common signature format with the community is. I am not talking about sharing rules for one tool only with one format (i.e. snort), but one format for several other rule-based systems. In this post, I will show you how I was able to take rules that describe **Windows event logs** from the [Sigma](https://github.com/Neo23x0/sigma) project and integrate them with my project [**HELK](https://github.com/Cyb3rWard0g/HELK)** via [**Elastalert](https://github.com/Yelp/elastalert)**. If you are using Elastalert and are considering on adding [**Sigma** **Windows](https://github.com/Neo23x0/sigma/tree/master/rules/windows)** rules to your stack, I hope this post gives you some ideas to expedite the process, and at least make it to your lab environment for you to do further research and testing before adding them to your production environment.

## What is Sigma?
>  Sigma is a generic and open signature format that allows you to describe relevant log events in a straightforward manner. The rule format is very flexible, easy to write and applicable to any type of log file. The main purpose of this project is to provide a structured form in which researchers or analysts can describe their once developed detection methods and make them shareable with others. Sigma is for log files what Snort is for network traffic and YARA is for files.

The project is yaml-based, and was developed primarily by [Thomas Patzke](https://github.com/thomaspatzke) and [Florian Roth](https://github.com/Neo23x0) focusing on a few of the issues that our industry faces when developing detection rules:

* Lack of standardization to describe log events from a rules creation perspective.

* Inefficiency to distribute and manage log signatures across different rule-based systems in an organization.

* Lack of flexibility to translate current vendor provided signatures to other rule-based systems in an organization.

## Sigma rules

Sigma distributes its [rule folders](https://github.com/Neo23x0/sigma/tree/master/rules) by applications, APT signatures, network, and operating systems. There is a Windows folder that contains several rules mainly categorized by log sources (Security, Application, System, Powershell, Sysmon, etc). The Windows folder will be part of the initial integration for HELK, since most of the log parsers available in the stack are for Windows event logs. As of today, there are 168 Windows rules available in the project, and they keep growing in numbers thanks to the community.

## Sigma Converter (Sigmac)

Something that I like about the Sigma project is that it also gives you resources to help you translate its current rules to other systems rule format (i.e Elastalert). This is done by a tool named Sigmac, and it already provides the following output target formats:

![](https://cdn-images-1.medium.com/max/2820/1*moLnfcTadsPbYSQ9OxIiTg.png)

You can get the output from above by running the following command with **Sigmac** after cloning the [**Sigma](https://github.com/Neo23x0/sigma)** project:

    sigma/tools/Sigmac -l

## Sigma Elastalert-Backend

The Elastalert-Backend was recently announced by Thomas Patzke [@blubbfiction](https://twitter.com/blubbfiction) in [one of his recent tweets](https://twitter.com/blubbfiction/status/1067919669452070913) . Thanks to the contribution by [Soumille Lucas @SouLuC13](https://twitter.com/SouLuC13), translating Sigma rules to an Elastalert rule format is now possible.

![](https://cdn-images-1.medium.com/max/2416/0*zTOgZppio55czf4h)

I had been thinking about adding Elastalert to the HELK, but I kept putting it off because I wanted to spend some time creating rules that I could provide with the project right out of the box. Now that Sigma can be translated to an Elastalert rule format, it makes it way easier to provide an initial set of rules for analyst to use and learn from. I used the following commands to translate each Sigma rule to an Elastalert rule format:

    tools/Sigmac -t elastalert -c field_index_mapping.yml -o /rules/elastalert_rule.yml Sigma_rule.yml

* **-t : **Output target format

* **-c: **Configuration with field name and index mappings

* **-o: **Output file or filename prefix if multiple files are generated

You can apply the same commands, but in a ‘**For’** loop and hit every single rule available in the project, but before we do that, what is Elastalert?

## What is Elastalert?
>  ElastAlert is a simple framework for alerting on anomalies, spikes, or other patterns of interest from data in Elasticsearch. It works by combining Elasticsearch with two types of components, rule types and alerts. Elasticsearch is periodically queried and the data is passed to the rule type, which determines when a match is found. When a match occurs, it is given to one or more alerts, which take action based on the match.

## Elastalert Global Configuration

Before getting into Elastalert rules format, it is important to understand its main global configuration to set the right Elasticsearch server, point to the right rules folder and even define how often a rule needs to be executed. An example of a global configuration can be found in the official Elastalert repo and it is named [config.yaml.example](https://github.com/Yelp/elastalert/blob/master/config.yaml.example). I took the comments and properties out of the config sample, and made a table out of it to make it easy to follow:

    rules_folder: example_rules
    run_every:
      minutes: 1
    buffer_time:
      minutes: 15
    es_host: elasticsearch.example.com
    es_port: 9200
    aws_region: us-east-1
    profile: test
    es_url_prefix: elasticsearch
    use_ssl: True
    verify_certs: True
    es_send_get_body_as: GET
    es_username: someusername
    es_password: somepassword
    verify_certs: True
    ca_certs: /path/to/cacert.pem
    client_cert: /path/to/client_cert.pem
    client_key: /path/to/client_key.key
    writeback_index: elastalert_status
    alert_time_limit:
      days: 2

![](https://cdn-images-1.medium.com/max/2552/1*cudbJ21P3GR_rp6ArJGzkQ.png)

## Elastalert Alert Types

When building elastalert rules, there are different types of alerts, known as subclasses of the alerter concept of Elastalert, that you can use for when it matches a certain rule logic . You can have more than one alert type per rule. Below is a table of the several types of alerts supported by Elastalert:

![](https://cdn-images-1.medium.com/max/2848/1*nExOALJkFDLMmOfgHFFa1Q.png)

## Elastalert Rule Format

Let’s take a look at an elastalert rule, and go through all its properties. The Elastalert shown in the example below is the results of the translation of the Sigma Windows rule [win_admin_share_access.yml](https://github.com/Neo23x0/sigma/blob/master/rules/windows/builtin/win_admin_share_access.yml).

    alert:
    - debug
    description: Detects access to $ADMIN share
    filter:
    - query:
      query_string:
        query: ((event_id:”5140" AND share_name:”Admin$”) AND NOT (user_name:”*$”))
    index: logs-endpoint-winevent-security-*
    name: Access-to-ADMIN$-Share_0
    priority: 4
    realert:
      minutes: 0
    type: any

![](https://cdn-images-1.medium.com/max/2840/1*_snSE3JqAR8tv_-V6mNTkw.png)

You can learn more about this in the [Elastalert documentation](https://elastalert.readthedocs.io/en/latest/ruletypes.html?highlight=alert#rule-types-and-configuration-options). I hope this helped a little bit to get you familiarized with the two open source projects that we will be integrating with [HELK](https://github.com/Cyb3rWard0g/HELK)

## HELK & Sigma Elastalert-Backend

One of the initial integrations of Elastalert with HELK started with Jordan Potty [@ok_by_now](https://twitter.com/ok_bye_now), but unfortunately, at that time, there were some compatibility issues with latest versions of Elastalert and the ELK stack. This new integration approach automatically imports Sigma rules to an Elastalert deployment and use them with HELK to enable alerting capabilities right out of the box, and automate the execution of pre-defined queries. HELK is built via docker as a proof of concept, so I added a new docker container which helped me to have everything ready and be able to share it with the community.

## New helk-elastalert Directory

This new [helk-elastalert folder](https://github.com/Cyb3rWard0g/HELK/tree/master/docker/helk-elastalert) available in the HELK github repo is used to build the new docker container and has the following directory structure:

![](https://cdn-images-1.medium.com/max/2824/1*B7QFds0gdSm7mgqMfDiNDg.png)

## Elastalert Field-Index Mapping Configuration

This configuration is one of the most important ones when translating Sigma rules to an Elastalert rule format. This is because, It allows you to map every field name defined in Sigma rules to your own standard field naming convention, and Sigma log sources defined in rules to indices names used in your own ELK stack. HELK is one of the first open source pipelines that follows its own common information model (CIM), and has indices per each data log source documented. Therefore, from a HELK perspective, it is important to make sure that the Elastalert rules follow the same [CIM that the project uses](https://github.com/Cyb3rWard0g/OSSEM/tree/master/common_information_model). Sigma was very nice to build an initial configuration for HELK, and I updated it to make sure it followed the latest compatible field and index mappings available in the project. You can find the [HELK config here](https://github.com/Neo23x0/sigma/blob/master/tools/config/helk.yml).

The two images below show how the field-index mapping config relates to Sigma rules and the Elastalert rule format (the result).

![](https://cdn-images-1.medium.com/max/3200/0*a9bS4v5AdWbsCuO6)

![](https://cdn-images-1.medium.com/max/3200/0*IaAjH8XKj6OkqO1x)

If you want to pull all the field names of every Windows Sigma rule to create your own field-index mapping file, you can run Sigmac in a loop with the target output format set to “**fieldlist”** and get all of them in a list. ****I did that to validate my own mappings.

    for rule_category in rules/windows/* ; do
      for rule in $rule_category/* ; do
        tools/Sigmac -t fieldlist $rule
      done
    done

## Translating Sigma rules to Elastalert format in HELK

Once we define the translation of Sigma rules to Elastalert rules format at the field and index level, we can start using Sigmac to perform the transformations. [All the code is available here](https://github.com/Cyb3rWard0g/HELK/blob/master/docker/helk-elastalert/scripts/pull-sigma.sh) for you to go over it, but the main part of the script that performs the translations are the following bash lines:

    for rule_category in rules/windows/* ; do
      for rule in $rule_category/* ; do
        tools/Sigmac -t elastalert -c Sigmac-config.yml -o /etc/elastalert//rules/Sigma_$(basename $rule) $rule
      done
    done

As you can see above, it goes through every Windows Sigma folder and every rule inside of each folder. It also names the Elastalert rule with the prefix “Sigma_” and the original Sigma rule name. At the end, you should be able to have all Windows Sigma rules translated to Elastalert rules .

![](https://cdn-images-1.medium.com/max/2868/0*W74zBskzPgHavVeJ)

The example below shows a Sigma rule translated to an Elastalert rule as shown before:

**Sigma rule**

![](https://cdn-images-1.medium.com/max/2704/0*1Z85wrhhfjgTUdXx)

**Elastalert alert file**

![](https://cdn-images-1.medium.com/max/2944/0*0xtJSEs82njOUuuH)

You can list all the Elastalert rules created in HELK by running the following command when your HELK is up and running:

    sudo docker exec -ti helk-elastalert ls /etc/elastalert/rules

You can also check any rules by following the following command:

    sudo docker exec -ti helk-elastalert cat /etc/elastalert/rules/helk_all_susp_powershell_commands.yml

    alert:
    - debug
    description: Detects potential suspicious powershell parameters
    filter:
    - query:
      query_string:
        query: (process_path:("*\\Powershell.exe") AND event_id:"1" AND process_command_line.keyword:( /.*\-w.*h.*/ /.*\-NoP.*/ /.*\-noni.*/ /.*\-ec.*/ /.*\-en.*/))
    index: logs-endpoint-winevent-*
    name: Windows-Suspicious-Powershell-commands_0
    priority: 2
    realert:
      minutes: 0
    type: any

## HELK Elastalert Workflow

Now that we have Elastalert rules ready in the right folder and elastalert running, the following is happening in the backend:

![](https://cdn-images-1.medium.com/max/3200/0*wLY_Pfptc9JeNXW9)

* Data flows through your pipeline

* Data gets stored in Elasticsearch

* Elastalert is constantly running queries defined in the Elastalert rule files against Elasticsearch

* Queries being run, matches found, and errors occurred in Elastalert are saved on specific Elasticsearch indices

* Kibana index patterns are already mapped to Elasticsearch indices. Therefore, security Analysts can see the queries being run, any matches found, and errors that occur in Elastalert via KIbana

Remember that Elastalert does not operate at the pipeline level. Therefore, alerting does not happen in real-time. It queries data already stored in Elasticsearch, and it does it from time to time depending on the time frequency set in the main global config.

## Elastalert Kibana indices

There are a few Kibana index patterns that are created by HELK that provide information about rules being executed against Elasticsearch, rules triggering alerts, and any error messages that occur in Elastalert. Elastalert writes to those indices allowing analysts to use Kibana and go through all that metadata. More information about Elastalert metadata index can be found [here](https://elastalert.readthedocs.io/en/latest/elastalert_status.html?highlight=elastalert_status#elastalert-metadata-index).

**elastalert_status** index: It is a log of information about every alert triggered:

![](https://cdn-images-1.medium.com/max/3200/0*g80Hx9FbwUmlzlp_)

**elastalert_status_status **index: It is a log of the queries performed for a given rule

![](https://cdn-images-1.medium.com/max/3200/0*05ZJE0nvxHbNDiC1)

**elastalert_status_error **index: it is log for errors that occur in Elastalert. Errors are written to both Elasticsearch and to stderr

![](https://cdn-images-1.medium.com/max/3200/0*VfAGfccQ2ZUk7TM1)

## HELK + Elastalert + Sigma + SLACK = 🍻💙

One aspect of this integration that I like a lot is that you can also select the type of alert that you want to trigger when it finds a match. ***I highly recommend to first have them go straight to your ES indices so that you can catch noisy ones and update them as you go. Also, remember that several Sigma rules are very broad so they might be more situated for situational awareness use cases rather than high fidelity alerts*.** Anyways, If you want to have alerts also being sent to a slack channel, I added a Slack integration with this too.

### Requirements:

* Admin rights to a Slack Workspace

* Slack App & Webhook URL

* Webhook URL Environment Variable

## Create Slack App & Webhook URL

Go to [https://api.slack.com/apps](https://api.slack.com/apps), and click on the **‘Create New App’** button as shown below

![](https://cdn-images-1.medium.com/max/3200/0*JC1QykWxvtAjHAdl)

Name your app and select the **Development Slack Workspace** from the drop down menu as shown below:

![](https://cdn-images-1.medium.com/max/2216/0*4OQDTkoQqHygzppE)

Once you create your app, you will see similar information from below:

![](https://cdn-images-1.medium.com/max/3200/0*SobBgwBSrIBRwTca)

Configure **Incoming Webhooks, **by clicking on **‘Incoming Webhooks’** as shown below:

![](https://cdn-images-1.medium.com/max/3112/0*SrvSNi7gO_wUorQ6)

Activate incoming Webhooks (turn it on)

![](https://cdn-images-1.medium.com/max/3112/0*lrH88y5lxd6sqTuP)

![](https://cdn-images-1.medium.com/max/2912/0*3si6MPEzq-s3W4QC)

Add a new Webhooks to Workspace by clicking on the button **‘Add New Webhook to Workspace’** and select the specific channel where you want to post the alerts being generated. For me since I am still testing this, I just select my own username.

![](https://cdn-images-1.medium.com/max/2000/0*xubXIgmcJCyweHhx)

Once you do that, click on **‘Authorize’**.

![](https://cdn-images-1.medium.com/max/2000/0*_KUZTTCKB9ye9CjQ)

After that, you will be taken back to the main page of your app. If you scroll down, you will be able to now see your **Webhook URL.** You will need that URL to post alerts to your Slack workspace from Elastalert. Also, one thing that you can do with your Slack app is give it an icon/logo. Click on ‘**Basic Information’** under ‘**Settings’ **on the left side of your App dashboard, and scroll down:

![](https://cdn-images-1.medium.com/max/3200/0*pnBJDct3vIM5raYi)

You can add an icon under the **‘App icon & Preview’** section like this:

![](https://cdn-images-1.medium.com/max/2836/0*OEottRcdYq5HjJl-)

## Webhook URL Environment Variable

HELK container has an environment variable that you can use when running the container. In my case, I use docker-compose so all I have to do is add it to the docker-compose file as shown below:

    helk-elastalert:
      build: helk-elastalert/
      container_name: helk-elastalert
      restart: always
      depends_on:
        — helk-elasticsearch
        — helk-kibana
      environment:
        ES_HOST: helk-elasticsearch
        ES_PORT: 9200
        **SLACK_WEBHOOK_URL: [https://hooks.slack.com/XXXXXXXXXXXXXX](https://hooks.slack.com/services/XXXXXXXXXXXXXX)**
     networks:
       helk:

When an Elastalert rule finds a match, it should send an alert to your Slack workspace. There is a [whoami rule from Sigma project](https://github.com/Neo23x0/sigma/blob/3288f6425b1a868c66f6f0a255956f8f041bc666/rules/windows/builtin/win_susp_whoami.yml) the got translated into an Elastalert rule. Let’s test that:

![](https://cdn-images-1.medium.com/max/2000/0*9PQJ7_iXqNUc71E0)

![](https://cdn-images-1.medium.com/max/3200/0*JSYDDIrmHLFtajpQ)

![](https://cdn-images-1.medium.com/max/2000/0*8xgdrQIRMW9aYUau)

Once again, some Sigma rules might be too broad to be high fidelity alerts, so be careful with this. I will continue working on only applying this feature to specific Elastalert rules that I believe are good candidates to be high fidelity alerts. So far it is applied to only **‘priority: 1’** Elastalert rules which translates to **‘level: critical’** in Sigma. This can be done by the following loop:

    for er in $ESALERT_HOME/rules/*; do
      priority=$(sed -n -e ‘s/^priority: //p’ $er)
      if [[ $priority = “1” ]]; then
        sed -i “s/- debug/- slack/g” $er
        sed -i “/- slack/a slack_webhook_url: $SLACK_WEBHOOK_URL” $er
      fi
    done

If you enable Slack or other alert type besides debug to every single rule in production, without the right testing, the following could happen:

## Powershell Substring Sigma Rule Use Case

I enabled Slack on every rule for this test. After deploying the new helk-elastalert container via HELK, I started to test a few rules, and my first instinct was to open a Powershell console and start running a few commands. I was not even finished writing a few basic commands and I got bombarded by hundreds of alerts via my initial Slack app (test) that I had created for testing.

![](https://cdn-images-1.medium.com/max/2000/0*Cj5wdR5zkFHy0sNw)

![](https://cdn-images-1.medium.com/max/2000/0*FMhc2z67FsnmlSQZ)

The alert going crazy after opening Powershell was **‘Suspicious-PowerShell-Parameter-Substring’**. I checked my Kibana **elastalert_status** index where I could see all the alerts being triggered, and I confirmed it was the only rule triggering several alerts in a few seconds:

![](https://cdn-images-1.medium.com/max/3200/0*EdpXmh1z8BhVmT2y)

I checked the Elastalert logic to get more context around it, and it was the one shown below:

    alert:
    - slack
    slack_webhook_url: [https://hooks.slack.com/services/XXXXXXXXXXXXX](https://hooks.slack.com/services/XXXXXXXXXXXXX)
    description: Detects suspicious PowerShell invocation with a parameter substring
    filter:
      - query:
        query_string:
          query: (process_path:”*\\powershell.exe” AND (“ \-windowstyle h “ OR “ \-windowstyl h” OR “ \-windowsty h” OR “ \-windowst h” OR “ \-windows h” OR “ \-windo h” OR “ \-wind h” OR “ \-win h” OR “ \-wi h” OR “ \-win h “ OR “ \-win hi “ OR “ \-win hid “ OR “ \-win hidd “ OR “ \-win hidde “ OR “ \-NoPr “ OR “ \-NoPro” OR “ \-NoProf “ OR “ \-NoProfi “ OR “ \-NoProfil “ OR “ \-nonin “ OR “ \-nonint” OR “ \-noninte “ OR “ \-noninter “ OR “ \-nonintera “ OR “ \-noninterac” OR “ \-noninteract “ OR “ \-noninteracti “ OR “ \-noninteractiv” OR “ \-ec “ OR “ \-encodedComman “ OR “ \-encodedComma “ OR “ \-encodedComm “ OR “ \-encodedCom” OR “ \-encodedCo “ OR “ \-encodedC “ OR “ \-encoded “ OR “ \-encode “ OR “ \-encod “ OR “ \-enco “ OR “\-en “))
    index: logs-endpoint-winevent-sysmon-*
    name: Suspicious-PowerShell-Parameter-Substring_0
    priority: 2
    realert:
      minutes: 0
    type: any

I also checked the Sigma rule where it came from, and it was from [sysmon_powershell_suspicious_parameter_variation.yml](https://github.com/Neo23x0/sigma/blob/d647a7de0730ad1e6789aa3f768b4e00b6a7a5f2/rules/windows/sysmon/sysmon_powershell_suspicious_parameter_variation.yml). Obviously, something was wrong with it. I decided to take the elastalert query and run it manually in the **logs-endpoint-winevent-sysmon-*** index.

After running the query, I was able to identify two problems. **First**, the query was matching on every string of every single event from Sysmon that had the **process_path** value ending in **“*\\powershell.exe”. Second**, the strings passed to the query were not triggering on the specific patterns provided. For example, instead of matching **‘-windows’** it was triggering on the string **‘window’** . Therefore, **“*\\powershell.exe” and “windows”** would trigger on several events mapped to **powershell** (Event IDs 1, 7, 12,13, etc.)

![](https://cdn-images-1.medium.com/max/3200/0*2go_PLY_GVc3E0aB)

I updated the query, and it definitely helped to match on the right patterns by filtering on **‘process_command_line’** and using the **.keyword field data type**.

    (process_path:(“*\\Powershell.exe”) AND event_id:”1" AND process_command_line.keyword:( /.*\-w.*h.*/ /.*\-NoP.*/ /.*\-noni.*/ /.*\-ec.*/ /.*\-en.*/))

Updated alert shown below:

    alert:
      - slack
    slack_webhook_url: [https://hooks.slack.com/services/XXXXXXXXXXXXX](https://hooks.slack.com/services/XXXXXXXXXXXXX)
    description: Detects potential suspicious powershell parameters
    filter:
      - query:
        query_string:
          query: (process_path:(“*\\Powershell.exe”) AND event_id:”1" AND process_command_line.keyword:( /.*\-w.*h.*/ /.*\-NoP.*/ /.*\-noni.*/ /.*\-ec.*/ /.*\-en.*/))
    index: logs-endpoint-winevent-*
    name: Windows-Suspicious-Powershell-commands_0
    priority: 2
    realert:
      minutes: 0
    type: any

![](https://cdn-images-1.medium.com/max/3200/0*as6x4SoEK_u2niL0)

![](https://cdn-images-1.medium.com/max/2064/0*oOzK7enq6kJnbM91)

One thing to remember is that the original Elastalert rule was exported by Sigmac, and might not be necessarily a problem caused by the way how the Sigma rule was written. I only submitted a PR to add the **process command line** field, but adding the **.keyword field data type** needed to be done on my end. I created a new Elastalert rule file, and I use it as a replacement of the original Elastalert rule.

## A few Patches Implemented

This Elastalert backend feature provided by Sigmac is new, so there are a few things that are still a work in progress to make the feature and the project itself more robust. In addition, there a few features from Sigma that do not apply to Elasticsearch. Therefore, I needed to apply a few patches for the helk-elastalert container to make the Sigma integration possible and smooth.

### Aggregation Operator ‘near’ not yet available:

In the process of translating Sigma rules, I got the following message on a few loops (example below is for when Sigmac tries to translate the Sigma rule [sysmon_mimikataz_inmemory_detection.yml](https://github.com/Neo23x0/sigma/blob/master/rules/windows/sysmon/sysmon_mimikatz_inmemory_detection.yml))

    An unsupported feature is required for this Sigma rule (rules/windows/sysmon/sysmon_mimikatz_inmemory_detection.yml): None : The ‘near’ aggregation operator is not yet implemented for this backend
    Feel free to contribute for fun and fame, this is open source :) -> [https://github.com/Neo23x0/Sigma](https://github.com/Neo23x0/sigma)

This behavior makes the Elastalert rule to be blank, and throws an error message when Elastalert starts. The fix is to delete those empty files before running Elastalert. According to the Sigma team, the aggregation operator ‘near’ cannot be used with Elasticsearch query strings and DSL. You can follow this issue [here](https://github.com/Neo23x0/sigma/issues/209)

### One Sigma rule with Two Log Sources = One Elastalert rule file with two rules in it

A few Sigma rules have two log sources defined such as Windows Sysmon and Windows security. This gets translated into two Elastalert rules, but in the same Elastalert rule file. Elastalert does not like that, and the alert never triggers. Therefore, the fix for now is to identify all the Elastalert rule files with two rules in them, and I split them into two new Elastalert rule files. You can follow this issue [here](https://github.com/Neo23x0/sigma/issues/205).

## That’s it? Profit?

Not yet! Projects like Sigma are great since they provide a lot of rules for an organization to consume right out of the box. However, it is very important to differentiate what rules are situated for high fidelity alerts, situational awareness or simply to know about your environment. You need to learn about every single rule and test them before you deploy them into production. You do not want to just dump all the rules without knowing what they do and expect them to magically work in your environment and send you an email when something triggers. ***Several Sigma rules are good to just notify you that some activity is happening, but not necessarily that an incident has happened. Elastalert helps to automate the process of running those queries from time to time and learn more about certain events in your environment.***

I hope this blog post was helpful for those that did not know how to integrate Sigma into their Elastalert deployments. I wanted to share my experience playing with it and what I had to do to add it to my project HELK. Remember to do your own research and test the rules before deploying Sigma via Elastalert to production. Some rules can be part of high fidelity alerts while others are very broad and could be used more for situational awareness. Also, every environment is different, so make sure you do the appropriate testing. If you believe rules can be improved, I encourage you to submit a PR and help the community in general. This new container is available in [HELK](https://github.com/Cyb3rWard0g/HELK) already so just follow the [installation instructions](https://github.com/Cyb3rWard0g/HELK/wiki/Installation) in the repo, and it will be run as part of the HELK stack automatically.

Feedback is greatly appreciated it!

## References
[**Cyb3rWard0g/HELK**
*The Hunting ELK. Contribute to Cyb3rWard0g/HELK development by creating an account on GitHub.*github.com](https://github.com/Cyb3rWard0g/HELK)
[**ElastAlert - Easy & Flexible Alerting With Elasticsearch - ElastAlert 0.0.1 documentation**
*At Yelp, we use Elasticsearch, Logstash and Kibana for managing our ever increasing amount of data and logs. Kibana is…*elastalert.readthedocs.io](https://elastalert.readthedocs.io/en/latest/elastalert.html#overview)

 <iframe src="https://medium.com/media/56512b060b0c82e469cb8f8e16cc8d78" frameborder=0></iframe>
[**Neo23x0/sigma**
*Generic Signature Format for SIEM Systems. Contribute to Neo23x0/sigma development by creating an account on GitHub.*github.com](https://github.com/Neo23x0/sigma)
