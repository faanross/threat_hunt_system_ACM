## General Introduction to Threat Hunting
- [What is Threat Hunting](./thunt/what_is.md)
- [Historical Context and Evolution](./thunt/history.md)
- [The Philosophy of Assumed Breach](./thunt/breach.md)
- [Threat Hunting vs. Other Security Functions](./thunt/other.md)
- [The Threat Hunting Loop and Core Workflow](./thunt/loop.md)
- [Threat Hunting Frameworks and Methodologies](./thunt/frameworks.md)
- [The Threat Hunting Maturity Model](./thunt/maturity.md)
- [Organizational Readiness and Prerequisites](./thunt/readiness.md)
- [The Pyramid of Pain and Hunting Focus Areas](./thunt/pyramid.md)
- [Signatures vs. Behaviours: The Hunter's Mindset](./thunt/mindset.md)
- [Network-Based vs. Endpoint-Based Threat Hunting](./thunt/net_vs_end.md)
- [Asset Inventory and Environmental Knowledge](./thunt/asset.md)
- [Data Sources and Telemetry Requirements](./thunt/data.md)
- [The Role of Threat Intelligence in Hunting](./thunt/intelligence.md)
- [Threat Hunting and Detection Engineering A Symbiotic Relationship](./thunt/detection.md)
- [Machine Learning and AI in Threat Hunting]
- [Challenges of Implementing Threat Hunting at Scale]
- [Triage and Prioritization Strategies]
- [Measuring Threat Hunting Effectiveness]
- [Building and Sustaining a Hunting Culture]
- [Legal, Privacy, and Ethical Considerations]
- [The Future of Threat Hunting]






## Introduction to Network-Centric Threat Hunting
- [Goal](./introduction/00_goal.md)
- [Overview of System](./introduction/01_system.md)


## What Are We Looking For - Malware Network Communication
- Introduction to Malware Communication

### Beaconing Behaviour
- [Introduction to Histograms](./beacon/00_histograms.md)
- [Connection Interval Histogram](./beacon/01_histograms_interval.md)
- [Connection Size Histogram](./beacon/02_histograms_size.md)
- [Hourly-Connection Graph](./beacon/03_hourly.md)
- [Beaconing](./beacon/04_beacon.md)
- [Jitter](./beacon/05_jitter.md)
- [Data Jitter](./beacon/06_data_jitter.md)





## General Investigations
- [Investigating IP Addresses](./general/00_ips.md)
- [Investigating Ports](./general/01_ports.md)
- [Investigating Domains](./general/02_domains.md)
- [Investigating Certificates](./general/03_certs.md)

## Investigating Processes + Binaries
- [Philosophy + Approach](./binaries/philosophy.md)
- [The Active-Passive Line](./binaries/active_passive.md)
- [The Five Critical Indicators](./binaries/framework.md)
- [Investigation Workflow](./binaries/workflow.md)
- [Indicator 1 - Binary Location](./binaries/location.md)
- [Indicator 2 - Process Name](./binaries/name.md)



## Investigating C2 over HTTP/S
TODO

## Investigating C2 over DNS
TODO


## Investigating C2 Agent-to-Agent Communication
TODO

## Cloud Threat Hunting
TODO

## Core Toolbox
- [Zeek](./core/zeek/moc.md)
- [Sysmon](./core/sysmon/moc.md)
- [RITA](./core/rita/moc.md)
- [AC-Hunter](./core/ach/moc.md)
- [Suricata](./core/suricata/moc.md)



## Extended Toolbox
- Zui
- [WireShark](./core/wireshark/moc.md)
- PEBear
- [System Informer](./core/sysinformer/moc.md)
- [WEL](./core/wel_logs/moc.md)
- [PowerShell ScriptBlock Logs](./core/ps_sb_logs/moc.md)


## Threat Hunt Lab Setup
TODO




## Import Links + References
- [DFIR Report](https://thedfirreport.com)
- [Malware Traffic Analysis](https://www.malware-traffic-analysis.net)
