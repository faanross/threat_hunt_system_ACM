# Data Sources and Telemetry Requirements

## The Data Paradox

Threat hunting is fundamentally a data analysis activity. Hunters search through logs, network traffic, and system telemetry looking for evidence of compromise. The logic seems straightforward: more data means better visibility. More logging captures more adversary activity, creating more opportunities for detection. So the solution appears simple - log everything, retain it forever, and hunt through all of it.

Except this approach fails spectacularly in practice. Organizations that "log everything" quickly discover a harsh reality: storage costs become prohibitive, query performance degrades to unusability, signal drowns in noise, and hunters spend more time managing data infrastructure than actually hunting. The supposed advantage - comprehensive visibility - becomes a liability when data volume makes analysis impractical.

This is the data paradox of threat hunting: you need extensive data to detect sophisticated threats, but too much data makes detection impossible. The solution isn't simply "more data" or "less data" - it's the _right_ data, collected at the right fidelity, retained for appropriate durations, and accessible through efficient query mechanisms. A small amount of high-quality, readily-searchable data enables more effective hunting than massive volumes of poor-quality or inaccessible data.

Understanding telemetry requirements isn't just a technical exercise - it directly informs infrastructure investments, budget allocation, and realistic assessment of hunting capabilities. Organizations that get this right build sustainable hunting programs. Those that get it wrong either starve hunters of necessary data or drown them in unsearchable data lakes.


## What Data Actually Matters

Three data categories form the foundation for most corporate hunting programs: network telemetry, endpoint telemetry, and authentication/directory services. These work in concert - network data provides the zoomed-out bird's-eye view revealing suspicious patterns, then you pivot to endpoint and authentication logs to zoom in and build the complete narrative. For cloud-heavy environments, add cloud service logs as a fourth pillar. These aren't abstract recommendations - they're the data sources that consistently enable detection of real attacks.

**Network telemetry** provides the highest vantage point for threat hunting. From this elevated perspective, you can observe communications patterns, identify anomalies, and spot suspicious behaviours across your entire infrastructure. When something looks wrong from the network view, you pivot to endpoint and authentication data to investigate the details.

Network flow data (NetFlow, IPFIX) offers basic visibility - source and destination addresses, ports, protocols, byte counts, and timestamps - but it was never designed for security analysis. Flow data shows _that_ communication happened but provides minimal context about _what_ happened. In a perfect world, full packet capture would give us everything, but PCAP's storage requirements make comprehensive deployment impractical for most organizations.

The sweet spot for security-focused network telemetry is **Zeek**. Zeek extracts and logs all relevant metadata from network traffic - connections, DNS queries, HTTP requests, file transfers, SSL certificates - in a security-focused format. Unlike generic flow data, Zeek was built for security analysis. It's fully customizable through scripting and even supports writing custom protocol parsers via Spicy for analyzing proprietary protocols. Tools like RITA and AC-Hunter then interpret this Zeek data to recognize patterns like for example command-and-control beaconing and dns tunneling.


Proxy and web gateway logs capture another critical attack vector. Web traffic serves as both initial compromise channel and exfiltration route. Understanding what URLs users access, what files they download, and what data exits through web protocols helps detect both inbound threats and outbound data theft.

**Endpoint telemetry** provides the ground-level detail that network visibility cannot. When network data reveals suspicious patterns, endpoint logs show exactly what happened on those systems - what processes executed, what files were created, what registry keys were modified.

EDR platforms provide comprehensive endpoint visibility, but **Sysmon** deserves specific attention as a powerful open-source alternative or complement. Sysmon logs detailed process execution data including command-line arguments and parent-child relationships, file creation and modification events, registry operations, network connections initiated by processes, named pipe creation, and much more. Sysmon's Event ID 3 network connection data is particularly valuable - it bridges endpoint and network telemetry, allowing you to correlate Zeek network logs with Sysmon endpoint data to build comprehensive attack narratives showing exactly which process on which system created specific network traffic.

Windows Event Logs (WEL), like NetFlow, were never designed for security analysis. They're noisy, inconsistent, and often lack the detail needed for effective hunting. Sysmon and EDR fill the gaps that WEL leaves, providing security-focused logging that actually enables investigation.

File system activity matters enormously - malware deployment, data staging, and tool operations all leave traces. Registry operations are indispensable in Windows environments where adversaries abuse the registry for persistence, configuration, and defense evasion. PowerShell and script logging has become essential as fileless attacks and living-off-the-land techniques rely on scripting. Comprehensive script block logging is a necessity, not a luxury.


**Authentication and directory services** logs track the who and when of access. Active Directory authentication events, Kerberos ticketing, VPN connections, and privileged account usage all reveal account compromise and credential abuse. Since adversaries inevitably need to authenticate and move laterally through environments, these logs enable detecting that critical activity. When network data shows suspicious lateral movement and endpoint data reveals the processes involved, authentication logs show which accounts were used - completing the picture.

**Cloud service logs** have become equally essential as infrastructure shifts to the cloud. API calls to AWS, Azure, or GCP reveal resource manipulation, configuration changes, and access attempts. SaaS application logs from Office 365, Google Workspace, and other cloud applications show data access, sharing, and administrative actions. Attackers increasingly target cloud infrastructure and applications, making these logs fundamental rather than supplemental.

The power emerges from how these data sources work together. Network telemetry spots the anomaly - unusual beaconing patterns, strange DNS queries, unexpected lateral movement. You pivot to endpoint data to see what processes created that traffic, what files were involved, what registry changes occurred. Authentication logs show which accounts were used. The combination creates a complete narrative that no single data source provides alone.




## The Quality Over Quantity Principle

Here's where many organizations go wrong: they focus on volume rather than quality. Collecting terabytes of data daily sounds impressive until you try to analyze it. The critical factors that actually determine hunting effectiveness are data quality, accessibility, and retention - not raw volume.

**Data quality** means having the necessary fields to perform analysis. But it's once again about finding the balance - we need sufficient detail to enable hunting, but excessive detail creates noise. The goal is finding the sweet spot where you capture what matters without drowning in irrelevant information.

**Accessibility** determines whether data can actually be searched. The most comprehensive data collection becomes worthless if query performance makes hunting impractical. If a search takes hours to complete, hunters can't iterate hypotheses effectively. They can't explore data, test ideas, or respond to developing situations. Query performance is fundamental to hunting. Subsecond searches enable experimentation; hour-long searches kill investigation momentum.

The query interface matters immensely. Hunters need expressive query languages that support filtering, aggregation, correlation, and statistical analysis. Simple keyword searches aren't sufficient for sophisticated hunting. The platform should enable complex multi-source correlation, pattern matching with regular expressions, and statistical outlier detection. Poor query interfaces become hunting bottlenecks, forcing hunters to export data and analyze it elsewhere - a workflow that defeats the purpose of centralized logging.

**Retention** involves careful balancing. Longer retention enables hunting for slow-moving threats and investigating historical compromises, but storage costs scale with retention. Different data types warrant different retention periods. Endpoint telemetry typically requires 90-180 days for effective hunting - enough to catch most threats before they age out. Network data like Zeek can be retained longer (6-12 months) since it's less voluminous. Authentication logs should persist for at least a year since investigating account compromise often requires historical context. Security alerts benefit from even longer retention (18-24 months) since they're low volume but high value.

The tiered storage approach resolves this tension: keep recent data in fast, searchable storage for active hunting, then migrate older data to cheaper storage for historical investigation. This "hot, warm, cold" model provides both immediate accessibility and long-term retention without prohibitive costs.


## The Data Lake Trap

"Big data" technologies promised to solve security analytics challenges through massive data lakes - storing everything affordably and querying flexibly. This sounds ideal for threat hunting, but data lakes have become traps for many organizations.

The most common problem is the unsearchable lake. Data gets poured into the lake without proper indexing, schemas, or organization. "We have the data" becomes the organization's mantra, but that data remains effectively unsearchable - a massive morass that's not only expensive to store but impossible to analyze efficiently.

Performance problems plague many data lake implementations. Data lake technologies often sacrifice query performance for storage economy. When queries take hours, hunting becomes impractical. Hunters need interactive analysis - the ability to quickly test hypotheses, refine searches, and follow investigative threads. Hour-long query times kill this investigative flow.

The schema swamp creates another challenge. Without consistent schemas and data normalization, hunters spend more time cleaning and parsing data than actually analyzing it. Every hunt turns into a data engineering project.

Data lakes aren't inherently bad for hunting, but they require thoughtful architecture. They work best as storage layers beneath security-specific platforms that provide hunting-optimized interfaces. Use the lake for long-term retention and periodic deep dives, not as your primary real-time hunting platform. The winning pattern combines recent data in fast storage (SIEM or similar) with historical data in lakes where slower access is acceptable.

## Making Practical Decisions

The core challenge remains balancing visibility against practical constraints of cost, performance, and operational capacity. Perfect visibility is impossible, so prioritization becomes essential.

Start by asking: what techniques actively target our organization or sector? Data enabling detection of those techniques takes first priority. Then ensure comprehensive coverage of common attack techniques - the MITRE ATT&CK framework provides excellent guidance here. After covering the fundamentals, address less common but high-impact scenarios. Nice-to-have visibility comes last.

Accept that some gaps will remain. This isn't defeat - it's honest risk assessment. Detailed logging of low-risk systems? That's an acceptable gap. Highly specialized attack techniques rarely seen in the wild? Probably acceptable. But core visibility into critical assets, data required for detecting common techniques, and logs needed for regulatory compliance? Those are unacceptable gaps that warrant investment.

Don't treat data collection as "set and forget." Instead, embrace continuous improvement: hunt with current data, identify detection gaps encountered during hunting, prioritize gap remediation based on risk and feasibility, deploy additional data sources to address high-priority gaps, and optimize or retire sources based on actual hunting value. This cycle ensures your telemetry evolves with both the threat landscape and your program's maturity.

## The Bottom Line

Data is threat hunting's foundation, but more data doesn't equal better hunting. The right data, properly collected, efficiently accessible, and appropriately retained - that's what enables effective hunting. Organizations succeeding at threat hunting understand this distinction. They invest in quality over quantity, accessibility over volume, and practical usability over theoretical comprehensiveness.

Start with the fundamentals: endpoint visibility, network telemetry, authentication logs, and cloud data. Ensure query performance supports interactive investigation. Implement tiered retention matching data value and investigation needs. Then iterate based on actual hunting experience - letting real detection gaps guide your investments rather than abstract completeness goals.

The data paradox resolves when you stop trying to collect everything and start focusing on collecting what actually enables detection. That shift - from maximum volume to optimal value - separates sustainable hunting programs from those that collapse under the weight of their own data.