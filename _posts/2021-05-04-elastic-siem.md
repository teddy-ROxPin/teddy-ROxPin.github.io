---
title: "Elastic Security: Process Diagrams and Detections" 
layout: post
---

<p align="center">
  <img src="/assets/posts/2021-05-04-elastic-siem/twitter_process_diagram.png">
</p>

## Introduction
After seeing a [Twitter post][process-diagram-post] from [@SBousseaden][SBousseaden-twitter] that had some awesome process diagrams, one is pictured above, I wanted to test them out for myself. I also noticed Detections while poking around in Elastic Security and decided to test them as well. There are [prerequisites and requirements][detections-on-prem-requirements] that must be fulfilled before the Detections feature can be used. 

I'm not too familiar with the inner-workings of ELK so it was painful process to get everything setup correctly. I have documented the steps here to hopefully spare someone else that pain.

## Step-by-Step ELK Setup (on Ubuntu with Docker and Docker Compose)
1. ~~~
cd /opt/
~~~
2. ~~~
git clone --branch tls https://github.com/deviantony/docker-elk.git
~~~
* The TLS branch must be used to if you'd like to use Detections since HTTPS communication between Elasticsearch and Kibana is a [requirement][detections-on-prem-requirements].
3. Edit */opt/docker-elk/docker-compose.yml* to increase the resources. 2g memory allocation in this example:
~~~
ES_JAVA_OPTS: "-Xmx2g -Xms2g"
~~~
* Fyi, I set it to 12g on a VM with a total of 16g and things were still pretty sluggish. Unusable once the Detection rules were enabled.
4. Add the following line to */opt/docker-elk/kibana/config/kibana.yml*
~~~
xpack.encryptedSavedObjects.encryptionKey: '6oh8wzpr5it4iyr0xi0h762so7a6xx7g'
~~~
* An alphanumeric value of at least 32 characters for the encryption key is a Detections [requirement][detections-on-prem-requirements].
5. ~~~
cd /opt/docker-elk/
~~~
6. ~~~
docker-compose up -d
~~~

## Shipping Sysmon EVTX Logs via Winlogbeat
1. Configure your Winlogbeat .yml file.
* Important part for the process diagrams is the "processors" section.
* An example, named "winlogbeat-evtx.yml", based on these [instructions][evtx-via-winlogbeat]:
^
~~~
winlogbeat.event_logs:
- name: ${EVTX_FILE}
    no_more_events: stop
    processors:
    - script:
        lang: javascript
        id: sysmon
        file: ${path.home}/module/sysmon/config/winlogbeat-sysmon.js

winlogbeat.shutdown_timeout: 30s
winlogbeat.registry_file: evtx-registry.yml

output.elasticsearch:
hosts: ["10.1.1.75:9200"]
protocol: "https"
ssl.enabled: true
ssl.verification_mode: none
username: "elastic"
password: "changeme"
~~~
2. ~~~
.\winlogbeat.exe -e -c .\winlogbeat-evtx.yml -E EVTX_FILE=c:\path\to\sysmon.evtx
~~~
* This should be "2." but I'm not willing to spend any more time fighting with markdown.

## Process Diagrams 
1. Stack Management -> Index Patterns -> Create index pattern -> winlogbeat-* 
 * Wait a minute or two after creating the index pattern.
2. Security -> Hosts -> Events
 * You can then click on a process event's "Analyze event" icon to view the process diagrams.
<p align="center">
  <img src="/assets/posts/2021-05-04-elastic-siem/process_diagram2.png">
</p>

## Detections
1. Security -> Detections -> Manage detection rules
   * From here you're able to "Load prebuilt rules", "Import rules", and "Create new rules".

### Detections Notes
I started with importing the prebuilt rules. Unfortunately the prebuilt rules I wanted to test were hardcoded to only look at logs with a timestamp of -5 minutes from when the rule runs. This would not work for me since I was ingesting logs that had taken place about a week prior and the prebuilt rules' schedule cannot be changed.

Elastic does have rules in [Github][detection-rules] along with a cli tool to process the rules into a format that can be imported into Detections. However, according to this [pull request][cli-fixes-pull] the cli tool's "export-rules" command is currently broken so I was unable to use this method.

Ultimately, I had to: 
1. Export the prebuilt rules
2. Set each rule's "timestamp_override" to "event.created"
3. Import the rules
4. Enable the rules
5. Ship in the logs

Once I was able to get the rules working, the process diagram would not load for any of the detections. To view the process diagram I would have to search for the specific process in Security -> Hosts -> Events.

Enabling the rules also crushed the VMs resources to the point that it was unusable.

## Conclusion
I found that getting to a particular process diagram and extracting meaningful information from it took longer than performing my normal KQL queries. I will keep an eye on how this progresses and plan to revisit it in the future.

I don't think I'll be revisiting the Detection rules. Too much wasted time all around to get things working just to end up with the detections crushing all available resources and process diagrams that would not load.

For now, I'll stick to using [HELK][helk-homepage] when processing evtx files for analysis.




[process-diagram-post]:                         https://twitter.com/SBousseaden/status/1383404194703417358
[SBousseaden-twitter]:                          https://twitter.com/SBousseaden
[detections-on-prem-requirements]:		        https://www.elastic.co/guide/en/security/current/detections-permissions-section.html#detections-on-prem-requirements
[evtx-via-winlogbeat]:                          https://www.elastic.co/guide/en/beats/winlogbeat/current/reading-from-evtx.html
[detection-rules]:                              https://github.com/elastic/detection-rules
[cli-fixes-pull]:                               https://github.com/elastic/detection-rules/pull/1073
[helk-homepage]:                                https://thehelk.com/intro.html