---
title:  "Active Directory Lab"
layout: post
---
## Current Structure
* Domain Controller
    * Windows 2019 Datacenter 1809
* Windows Workstation
    * [Invoke-AtomicRedTeam]
    * Windows 10 Enterprise 1909
* Splunk
    * Single server: Indexer, Search Head, Deployment Server
    * Centos 7
* XSOAR (Demisto) 
    * Alerts created from [TA_Saved_Searches] and fed from Splunk
    * Centos 7
* Kali
    * Management and Attacking machine
* [Velocidex Velociraptor][velociraptor-homepage] Server
    * Awesome open-source project for 'endpoint monitoring, digital forensic investigations and cyber incident response.'
    * Ubuntu 18.04
* [SANS Slingshot C2 Matrix Edition][slingshot-homepage]
    * Using [VECTR][vectr-homepage], another awesome open-source project, for detection validation tracking
    * Testing [Sliver C2]

### Windows Machine Configurations
* Splunk Universal Forwarder installed
* GPO for logging configured using [guidance from the Australian Signals Directorate][asd-guidance]

[asd-guidance]:             https://www.cyber.gov.au/publications/windows-event-logging-and-forwarding
[Invoke-AtomicRedTeam]:     https://github.com/redcanaryco/invoke-atomicredteam
[slingshot-homepage]:       https://www.sans.org/blog/introducing-slingshot-c2-matrix-edition/
[Sliver C2]:                https://github.com/BishopFox/sliver
[TA_Saved_Searches]:        https://github.com/teddy-ROxPin/TA_Sigma_Searches
[vectr-homepage]:           https://vectr.io/
[velociraptor-homepage]:    https://www.velocidex.com/
