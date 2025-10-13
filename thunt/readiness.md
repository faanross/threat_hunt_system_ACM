# Organizational Readiness and Prerequisites

## The Readiness Reality Check

Imagine trying to perform surgery without proper medical equipment, training, or even a basic understanding of human anatomy. The attempt would be worse than useless - it would be actively harmful. Yet organizations regularly attempt threat hunting without the foundational capabilities that make hunting possible, let alone effective. They hire hunters, deploy hunting platforms, and launch hunting programs, only to discover that they lack the data, tools, processes, or organizational structure needed for hunting to succeed.

The results are predictable: frustrated hunters unable to do their jobs, wasted investment in tools and personnel, leadership disappointment when hunting fails to deliver promised value, and organizational cynicism about security initiatives that creates barriers to future improvements. These failures aren't because threat hunting doesn't work - they're because organizations attempted hunting before they were ready.

This chapter provides a reality check: what prerequisites must be in place before launching a threat hunting program? What technical capabilities, organizational structures, and cultural attributes are required? How do you honestly assess whether you're ready, and what should you do if you're not ready yet? Most importantly, how do you avoid the costly mistake of premature hunting investment that undermines both immediate effectiveness and future security initiatives?

The core message is simple but critical: threat hunting is resource-intensive and requires solid fundamentals. Organizations should only undertake hunting when prerequisites are in place. If prerequisites are lacking, investing in hunting directly is putting cart before horse - better to invest first in building the foundation that makes hunting possible.


## Technical Prerequisites: The Data Foundation

Threat hunting is fundamentally data analysis. Hunters search through logs, network traffic, endpoint telemetry, and other data sources looking for evidence of compromise. Without adequate data collection, retention, and accessibility, hunting is simply impossible. This seems obvious, yet insufficient data infrastructure is the single most common reason hunting programs fail.

### Comprehensive Logging and Telemetry

The first prerequisite is comprehensive data collection across critical systems and security domains. What does "comprehensive" mean practically?

**Endpoint Telemetry**: Modern endpoint detection and response (EDR) tools provide detailed visibility into process execution, file operations, registry changes, network connections, and user activity on endpoints. This telemetry is essential for hunting - without it, you're blind to most adversary activity. At minimum, you need visibility into:

- Process creation events with full command lines, directories, timestamps, and parent-child relationships (PID + PPID)
- Network connections initiated from endpoints
- File creation, modification, and deletion events
- PowerShell and script execution
- Authentication and privilege escalation activities
- DLL/module loading patterns


**Network Visibility**: Network-based hunting requires visibility into network traffic flows. Though full packet capture would be ideal, in most cases it's absolutely not possible. In this case we recommended you run Zeek, which will capture most data relevant to security operations.

Some ritical elements include:

- Which internal hosts communicated to which external hosts
- DNS queries and responses
- Proxy logs showing web access patterns
- Firewall logs documenting allowed and blocked connections
- IDS/IPS logs capturing suspicious network patterns
- etc.

Without network visibility, hunters miss command and control communications, lateral movement, and data exfiltration - all critical adversary activities.

**Authentication and Access Logs**: Understanding who is accessing what, from where, and when is fundamental to hunting. Required logging includes:

- Successful and failed authentication attempts across all systems
- Privilege escalation and administrative access events
- VPN and remote access connections
- Cloud service authentication (Office 365, AWS, etc.)
- Service account usage patterns

Authentication anomalies are often early indicators of compromise - but only if you're collecting the data.

**Application and System Logs**: Beyond security-specific logs, application and system logs provide context. Web server logs, database audit trails, and application-specific logging often contain evidence of compromise that security tools miss.

### Data Retention and Accessibility

Collecting data is necessary but insufficient - you also need adequate retention and efficient access.

**Retention Periods**: Many attacks have long dwell times measured in weeks or months. If you only retain logs for seven days, you can't investigate attacks that began three weeks ago. Practical hunting requires:

- **Minimum 30 days** of detailed logs for active investigation
- **90+ days** for comprehensive hunting and historical analysis
- **One year or more** for select data types where storage is economical

Retention requirements must be balanced against storage costs, but inadequate retention creates blind spots that make hunting ineffective.

**Query Performance**: Data must be searchable efficiently. Waiting hours for query results makes iterative hunting impractical. Modern hunting requires:

- Sub-second to few-minute query response times for typical searches
- Ability to search across multiple data sources simultaneously
- Flexible query languages that support complex filtering and correlation

This typically means centralized logging infrastructure (SIEM, data lakes, or specialized hunting platforms) with indexed data and optimized search capabilities.

**Data Integration**: Data from different sources must be correlatable. If endpoint logs, network logs, and authentication logs can't be linked by common identifiers (IP addresses, hostnames, user accounts, timestamps, UIDs), hunting becomes fragmented and inefficient.


### The Data Readiness Test

A practical test of data readiness: can you answer these hunting questions with your current data?

1. "Show me all processes that made external network connections in the past 24 hours"
2. "Which users authenticated to domain controllers from unusual locations this week?"
3. "What PowerShell commands were executed on this server in the past month?"
4. "Which systems communicated with this suspicious IP address, and when?"
5. "Show me all new scheduled tasks created on endpoints in the past 30 days"

If you can't answer these questions within minutes using your existing data infrastructure, you lack the data foundation needed for effective hunting. Before investing in hunting personnel or programs, invest in building this foundation.


## Baseline Security Operations Prerequisites

Threat hunting doesn't exist in isolation - it depends on and builds upon baseline security operations. Organizations must have certain operational capabilities in place before hunting can be effective.

### Functional SOC Operations

A Security Operations Center (or equivalent function) must be operating effectively before adding hunting capability. This means:

**Alert Triage Under Control**: If your SOC is drowning in alerts, unable to keep up with basic triage, adding hunting just makes the problem worse. Hunters need SOC operational stability so that discovered threats can be handled through existing response processes.

**Basic Detection Coverage**: Automated detection systems (SIEM rules, EDR detections, IDS/IPS) should be detecting known threats reliably. Hunting focuses on unknown threats - if you're not catching known threats with automation, adding hunting is premature.

**Incident Response Capability**: When hunters discover threats, someone must respond. Without functional incident response, hunting discovers threats that then go unaddressed - wasting hunting effort and leaving threats active.

The relationship between SOC operations and hunting is symbiotic: SOC provides the foundation and response capability that hunting depends on, while hunting feeds back improved detection rules and environmental knowledge that make SOC more effective. But this symbiosis requires SOC to be functional first.

### Asset Management and Inventory

You can only hunt across systems you know exist. Comprehensive asset inventory is a prerequisite for effective hunting:

**Complete Asset Inventory**: All servers, workstations, network devices, and other assets should be catalogued with sufficient detail to enable hunting. This includes:

- System hostnames and IP addresses
- Operating systems and versions
- System owners and business functions
- Criticality classifications
- Network locations and segment information

**Configuration Management**: Understanding standard configurations helps identify anomalies. What software should be installed? What services should run? What accounts should exist? Deviations from standards often indicate compromise.

**Asset Lifecycle Tracking**: Knowing when systems were deployed, modified, or decommissioned prevents false positives when hunting. That "suspicious" authentication to a server might be legitimate if you knew it was recently deployed.

Organizations with poor asset management find hunting extremely frustrating. You discover suspicious activity on a system but can't determine what that system is, what it should be doing, or who owns it. Without this context, distinguishing malicious from legitimate activity becomes nearly impossible.

### Environmental Baselines

Threat hunting fundamentally involves identifying deviations from normal. But you must first understand what "normal" looks like:

**Network Traffic Baselines**: What systems normally communicate with each other? What external destinations are regularly accessed? What protocols and ports are used? Without baselines, everything looks potentially suspicious.

**User Behavior Baselines**: What are typical authentication patterns? When do users normally log in? From what locations? What resources do they access? Understanding normal user behavior is essential for detecting account compromise.

**System Behavior Baselines**: What processes normally run on different system types? What scheduled tasks exist? What services should be running? Baseline understanding enables anomaly detection.

These baselines needn't be perfect or comprehensive when starting hunting - they improve through hunting activities. But some baseline understanding is prerequisite. Starting with zero understanding of normal makes distinguishing signal from noise extremely difficult.


### Threat Intelligence Capability

While organizations can hunt without extensive threat intelligence, having at least basic threat intelligence access significantly improves hunting effectiveness:

**External Threat Feeds**: Indicator feeds, vulnerability databases, and technique descriptions provide context for hunting. When you discover suspicious activity, threat intelligence helps determine whether it's known attacker infrastructure or benign.

**Tactical Intelligence**: Understanding current attack campaigns, active vulnerability exploitation, and trending adversary techniques helps prioritize hunting activities and generate relevant hypotheses.

**Intelligence Consumption Process**: Having threat intelligence is insufficient if you can't operationalize it. Processes for consuming intelligence, determining relevance, and translating it into hunting activities should exist - even if informal initially.

Organizations sometimes overestimate threat intelligence needs, assuming they must have extensive commercial feeds before hunting. In reality, open source intelligence (OSINT) and industry sharing provide sufficient context for initial hunting programs. Having some intelligence capability is useful, but the lack ofcommercial threat intelligence should never be used as an excuse to delay the initial implementation of a threat hunting program. 

