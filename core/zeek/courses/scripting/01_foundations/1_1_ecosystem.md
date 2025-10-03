# **LESSON 1.1: ZEEK ECOSYSTEM OVERVIEW**

## **PART 1: THE EVOLUTION OF ZEEK - FROM RESEARCH PROJECT TO ENTERPRISE STAPLE**

### **The Birth of Something Different**

In 1995, a PhD student at UC Berkeley named Vern Paxson was working on his dissertation at Lawrence Berkeley National Laboratory. The laboratory had a problem that was becoming increasingly common in the mid-1990s: their network was under constant probing and attack, but existing intrusion detection systems weren't giving them the visibility they needed. These systems, which primarily relied on signature matching, could tell you if a known attack pattern appeared in your traffic, but they couldn't tell you much about what was actually happening on the network moment to moment.

Paxson approached the problem differently. Instead of building yet another signature-matching system, he asked himself: "What if we built a system that could continuously observe network activity, understand the protocols being used, track the state of connections, and provide a rich stream of information about what's really happening?" This question led to the creation of Bro - originally standing for "Big Brother," a reference to Orwell's all-seeing surveillance system, though in this case watching networks rather than people.

The early version of Bro that emerged from Paxson's research had several revolutionary characteristics that set it apart from everything else available at the time. First, it was designed to be completely passive. Rather than sitting inline and potentially becoming a bottleneck or single point of failure, Bro would observe a copy of network traffic without interfering with it. Second, it **separated policy from mechanism**. The core engine would handle the hard work of parsing protocols and tracking connection state, while separate scripts - written in a custom scripting language - would implement the detection logic. This meant that security analysts could customize detection behaviour without modifying the core system.

Third, and perhaps most importantly, Bro was designed to generate rich logs about network activity whether or not any suspicious behaviour was detected. This was a radical departure from traditional intrusion detection systems, which primarily generated alerts when signature matches occurred. Paxson understood that for real network security monitoring, you needed to understand baseline behavior, track long-term trends, and have detailed forensic data available when investigating incidents.


### **From Academic Tool to Production System**

Throughout the late 1990s and early 2000s, Bro was primarily deployed in academic and research environments. Universities and national laboratories were ideal environments for the tool because they had the technical expertise to manage it and appreciated its flexibility and depth. But Bro was not an easy system to deploy. It required significant expertise to configure, tune, and operate effectively. The scripting language was powerful but had a steep learning curve. The system generated enormous volumes of logs that needed to be managed and analyzed.

Despite these challenges, organizations that invested in deploying Bro discovered something remarkable: they could detect sophisticated attacks that their signature-based systems completely missed. When an attacker used a novel technique or modified existing malware to evade signatures, Bro's behavioural analysis capabilities could still identify anomalous activity. A connection that established itself and then sat open for hours sending periodic small packets might look innocuous to a signature-based system, but Bro could identify it as potential command-and-control traffic based on its behavioural characteristics.

As Bro matured through the 2000s and early 2010s, several major improvements expanded its capabilities. The introduction of cluster architecture meant that Bro could scale to monitor high-bandwidth networks by distributing the analysis workload across multiple systems. The development of the Broker communication framework provided efficient mechanisms for different Bro instances to share data and coordinate analysis. The scripting language evolved to support more sophisticated detection logic and statistical analysis.

By 2013, when Bro reached version 2.0, it had become a production-grade system capable of monitoring some of the world's most demanding networks. But it still carried the "Bro" name, which was increasingly seen as a barrier to broader adoption. Some organizations were hesitant to deploy a security tool with a name that could be seen as unprofessional or insensitive.


### **The Transformation to Zeek**

In October 2018, the Bro project announced that it would be rebranding to Zeek. The name, derived from the prophet Ezekiel who had visions of future events, symbolized the system's ability to provide foresight into network threats through its analytical capabilities. More practically, the name change was part of a broader effort to make the project more accessible and welcoming to a wider audience.

The rebranding to Zeek marked more than just a name change. It represented a maturation of the project from a research tool with an academic pedigree into a system that organizations of all types could adopt. The documentation was improved and reorganized. The package management system was enhanced to make it easier to discover and install community-contributed scripts. Commercial support options became available through companies like Corelight, which was founded by the original Zeek developers.

Today, Zeek is deployed in environments ranging from small businesses to Fortune 500 companies, from university networks to critical infrastructure providers. It's used by government agencies for national security missions and by managed security service providers offering network monitoring services to their clients. The tool that began as one graduate student's research project has become a cornerstone of modern network security monitoring.

---

## **PART 2: UNDERSTANDING ZEEK'S PHILOSOPHICAL FOUNDATION**

### **Behavioural Analysis vs. Signature Matching**

To truly understand what makes Zeek powerful - and why you're investing time in learning it - you need to grasp the fundamental philosophical difference between signature-based detection and behavioural analysis. This isn't just an academic distinction; it will shape how you think about threat hunting and how you write detection scripts throughout this course.

Traditional intrusion detection systems, and signature-based tools like Snort and Suricata, operate on a conceptually straightforward principle. They examine network traffic looking for specific patterns - signatures - that indicate known malicious activity. If a packet or series of packets matches a signature, the system generates an alert. This approach has significant advantages: it's fast, it's deterministic, and when properly tuned it generates relatively few false positives.

But signature-based detection has a fundamental limitation: it can only detect what it knows about. If an attacker uses a new technique, modifies existing malware to change its network signatures, or exploits a zero-day vulnerability for which no signature exists, signature-based systems will remain silent. They're excellent at detecting known threats, but they struggle with novel or sophisticated attacks.

Zeek takes a completely different approach, one that's rooted in behavioural analysis. Rather than looking for specific malicious patterns, Zeek focuses on understanding what's actually happening on the network. It doesn't ask "Does this traffic match a known bad pattern?" Instead, it asks "What protocols are being used? What are the characteristics of these connections? Are there unusual patterns in timing, volume, or behavior?"

Let me give you a concrete example that illustrates this distinction. Imagine an attacker has deployed a Remote Access Trojan (RAT) on one of your network endpoints. The RAT needs to communicate with its server to receive instructions and send back stolen data.

A signature-based system would look for known indicators of this specific RAT. Perhaps previous analysis of the malware revealed that it uses a particular HTTP User-Agent string, or sends requests to a specific URI path like `/api/v2/beacon`, or includes a distinctive string in its POST data. The security vendor would create signatures for these patterns and distribute them. Your IDS would watch for those specific indicators and alert if they appear.

This works well  - until the malware author makes a minor modification. They change the User-Agent string to mimic a legitimate browser. They modify the URI path to `/images/logo.png`. They encode the POST data differently. Suddenly, your signatures no longer match, and the RAT communicates freely without detection.

Now consider how Zeek would approach the same scenario. Rather than looking for specific strings or patterns, Zeek analyzes the behavioural characteristics of the communication. It notices that one internal host has established an HTTP connection to an external IP address, and this connection exhibits unusual characteristics. The requests happen with remarkable regularity - one every sixty seconds, with very low jitter. The size of the requests is unusually consistent, always around 147 bytes. The connection persists for hours or days. The bidirectional byte ratio suggests a command-and-response pattern rather than normal web browsing.

None of these individual characteristics might be definitive proof of malicious activity, but together they form a behavioural signature that strongly suggests command-and-control traffic. More importantly, this behavioural analysis works regardless of what specific strings appear in the traffic. The attacker can change their User-Agent, modify their URI paths, and encode their data however they want, but as long as they maintain this periodic beaconing behaviour - which is fundamental to how the RAT operates - Zeek can detect it.

This is the power of behavioural analysis, and it's why Zeek has become an indispensable tool for threat hunting. When you're searching for sophisticated adversaries or novel threats, you need to look beyond signatures. You need to understand the behavioral patterns that indicate malicious activity, even when those patterns don't match any known signatures.


### **The Event-Driven Architecture**

Zeek's approach to behavioural analysis is enabled by its event-driven architecture, which is quite different from how traditional packet inspection systems work. Understanding this architecture is crucial because it will directly affect how you write detection scripts later in this course.

In a traditional packet-based intrusion detection system, the analysis flow is relatively straightforward. The system captures packets from the network, and for each packet, it runs through a series of signature checks. If any signature matches, an alert is generated. The system maintains minimal state between packets - it's essentially asking "Does this packet match a known bad pattern?" for each packet independently.

Zeek works completely differently. Rather than operating at the packet level, Zeek operates at the protocol and connection level through an event-driven model. Here's how it works: as packets arrive, Zeek's protocol analyzers parse them and reconstruct the higher-level protocols being used. When significant protocol-level events occur - a connection is established, an HTTP request is made, a DNS query is issued, a file transfer begins - Zeek generates events.

These events are the fundamental building blocks of Zeek's analysis capabilities. An event represents something meaningful happening on the network at the protocol level. For example, when someone browses to a website, Zeek doesn't just see a series of TCP packets. It sees a connection establishment event, followed by HTTP request events, followed by HTTP response events, and eventually a connection termination event. Each of these events carries rich contextual information about what's happening.

Your detection scripts operate by responding to these events. Rather than examining individual packets, you write functions that are called when specific events occur. For instance, you might write a function that's called every time an HTTP request event fires. This function receives detailed information about the request - the URL being accessed, the User-Agent header, the size of the request, the time it occurred, and much more. Based on this information, your script can make intelligent decisions about whether the activity is suspicious.

The event-driven model provides several crucial advantages for threat hunting. First, it gives you access to high-level protocol information without needing to manually parse packets. Zeek has already done the hard work of extracting HTTP headers, DNS query names, SSL certificate details, and hundreds of other protocol elements. Second, it allows you to maintain state across multiple related events. You can track characteristics of a connection over its entire lifetime, building up a behavioural profile that wouldn't be possible if you were only examining individual packets. Third, it enables sophisticated analysis based on timing and patterns. You can detect that a host is making DNS queries at regular intervals, or that a connection is exhibiting unusual periodicity in its communications.



### **Protocol Context and Connection State**

One of Zeek's most powerful capabilities is its ability to maintain rich protocol context and connection state. This is intimately tied to the event-driven architecture, but it deserves special attention because it's what enables much of the sophisticated analysis you'll be doing later.

When Zeek observes network traffic, it doesn't just pass events to your scripts and then forget about them. It maintains detailed state about ongoing connections, tracking everything from basic TCP connection parameters to application-layer protocol details. This state information is made available to your scripts through data structures that represent connections and the protocols they're using.

Consider an HTTP connection as an example. When Zeek sees HTTP traffic, it doesn't just generate isolated events for each request and response. It maintains a complete understanding of the HTTP session. It tracks all the requests made over the connection, all the responses received, the headers exchanged, the content types, the methods used, the status codes returned, and much more. When your script responds to an HTTP event, it has access to this complete context. You can ask questions like "Is this the first request on this connection, or have there been previous requests?" or "What was the User-Agent header from the initial request in this session?" or "Has this connection been reused for multiple requests?"

This contextual awareness is what enables behavioural analysis that would be impossible with simpler packet-level inspection. You can detect subtle anomalies like connections that claim to be HTTP but exhibit unusual characteristics, or sessions where the client and server behavior doesn't match expected patterns for the protocols they're supposedly using.

The same principle applies to other protocols. For DNS, Zeek tracks queries and their responses, building a complete picture of DNS activity for each host. For SSL/TLS, it extracts and validates certificates, tracks cipher suites, and identifies potential security issues. For file transfers, it can extract the files themselves and compute hashes for malware analysis.

This level of protocol awareness and state tracking is what sets Zeek apart from simpler packet capture tools. It provides you with a high-level, semantically meaningful view of network activity that forms the perfect foundation for behavioural analysis and threat hunting.

---



## **PART 3: ZEEK IN THE NETWORK SECURITY TOOL ECOSYSTEM**

### **Understanding Where Zeek Fits**

Now that you understand Zeek's philosophical approach and architecture, it's important to understand where it fits in the broader ecosystem of network security tools. Zeek isn't meant to replace every other security tool in your environment-it's designed to complement other systems and fill specific gaps that other tools don't address well.

The network security monitoring landscape includes several categories of tools: signature-based intrusion detection systems, packet capture and analysis tools, network flow analyzers, and security information and event management platforms. Each has its strengths and ideal use cases. Understanding where Zeek excels - and where other tools might be more appropriate - will help you architect comprehensive security monitoring solutions.



### **Zeek and Signature-Based IDS: Complementary, Not Competing**

Let's start by examining the relationship between Zeek and signature-based intrusion detection systems like Snort and Suricata. At first glance, these might seem like competing technologies - they all monitor network traffic looking for threats. But in practice, they're highly complementary, and mature security operations centres often deploy both.

Snort, which has been around since 1998 and is one of the most widely deployed open-source IDS systems, excels at real-time detection of known threats. It processes packets at line rate, comparing them against thousands of signatures that describe known attacks. When traffic matches a signature, Snort generates an alert immediately. It can also operate in prevention mode, sitting inline and actively blocking traffic that matches malicious signatures. This makes Snort excellent for defending against commodity attacks and known threats.

Snort's strength is also its limitation. Because it relies on signatures, it needs to know what to look for. When a new vulnerability is discovered and exploited in the wild, there's a window of time before signatures are developed and deployed. During this window, Snort will be blind to the attack. Similarly, when attackers customize malware or use novel techniques, they may evade signature-based detection entirely.

This is where Zeek shines. Because Zeek focuses on behavioural analysis and generates comprehensive logs of network activity, it can detect suspicious patterns even when no signature exists. More importantly, when an incident occurs, Zeek's detailed logs provide the forensic data needed to understand what happened. You can reconstruct the attacker's actions, identify the scope of a compromise, and gather the intelligence needed to improve your defenses.

In a well-designed security architecture, Snort and Zeek work together. Snort provides the first line of defense against known threats, generating real-time alerts and optionally blocking malicious traffic. Zeek provides deep visibility into network behaviour, enabling the detection of sophisticated threats and providing the rich forensic data needed for investigation and response. When Snort generates an alert, you can pivot to Zeek's logs to get the full context of what was happening on the network at that time.

Suricata represents an interesting middle ground. Originally developed as a modernized alternative to Snort, Suricata includes many improvements over traditional signature-based IDS. It has native multi-threading support, can generate rich JSON logs, and includes some protocol analysis capabilities that go beyond simple signature matching. In some ways, Suricata tries to bridge the gap between Snort's signature-based approach and Zeek's behavioural analysis.

However, Suricata's protocol analysis capabilities, while improving, don't match Zeek's depth. Zeek was designed from the ground up for protocol analysis and behavioural monitoring, with a full-featured scripting language that gives you complete control over detection logic. Suricata's scripting capabilities are more limited, primarily using Lua for simple programmatic rules. For complex behavioral analysis, custom protocol parsers, or sophisticated statistical detection, Zeek remains the better choice.



### **Zeek and Packet Capture: Different Layers of Abstraction**

Another important comparison is between Zeek and traditional packet capture tools like tcpdump and Wireshark. These tools serve very different purposes, but they're often used together in complementary ways.

Tools like tcpdump and Wireshark capture complete packets from the network and allow you to examine them in detail. This is incredibly valuable when you need to perform deep forensic analysis of specific network activity. You can see exactly what bytes were sent, examine protocol headers at the lowest level, and track the precise sequence of network events.

However, packet capture tools have significant limitations when it comes to continuous network monitoring. Capturing full packets requires enormous amounts of storage, especially on high-bandwidth networks. A one-gigabit network generating even modest traffic can produce terabytes of packet capture data per day. This makes long-term storage impractical for most organizations. Additionally, analyzing packet captures is largely a manual process. While you can write display filters in Wireshark or process captures with command-line tools, you're still working at a very low level of abstraction.

Zeek takes a completely different approach. Rather than capturing complete packets, Zeek extracts high-level information and generates structured logs. An HTTP transaction that might consume megabytes in a packet capture might be represented by a few hundred bytes in Zeek's HTTP log, containing all the important details-source and destination, requested URL, response code, content type, sizes, and timing information-but discarding the raw packet bytes.

This approach provides several enormous advantages. First, Zeek's logs are compact enough that you can realistically store months or even years of network activity. This long-term retention is crucial for threat hunting and incident investigation. Second, Zeek's logs are structured and easily searchable, enabling rapid analysis of large time periods. Third, Zeek's automated analysis can identify suspicious patterns across millions of connections, something that would be impossible with manual packet analysis.

The typical deployment pattern is to use Zeek for continuous monitoring and automated analysis, while using targeted packet capture for specific investigations. Zeek can trigger packet capture when it detects something suspicious, giving you the best of both worlds: efficient long-term monitoring with detailed packet-level data available when needed. Many organizations use Zeek's notice framework to automatically initiate packet capture whenever high-priority alerts are generated, ensuring that the raw packet data is available for deep forensic analysis.



### **Zeek and SIEM: Complementary Components of a Security Architecture**

Perhaps the most important relationship to understand is between Zeek and Security Information and Event Management (SIEM) platforms like Splunk, Elastic Stack, or commercial solutions like QRadar and ArcSight. This relationship is often misunderstood, with people sometimes asking whether they should deploy Zeek or a SIEM. The answer is that they serve fundamentally different purposes and are most powerful when used together.

A SIEM platform is designed to aggregate security data from many different sources across your environment - firewalls, intrusion detection systems, endpoint security tools, authentication logs, application logs, and more. The SIEM normalizes this data into common formats, stores it in a searchable database, provides correlation capabilities to identify patterns across different data sources, and offers dashboards and alerting to help analysts make sense of it all.

Zeek is not a SIEM. Zeek is a network visibility tool that generates structured data about network activity. It's one of the sources that feeds data into your SIEM. In a typical architecture, Zeek sits at strategic points in your network, analyzing traffic and generating logs. These logs are then forwarded to your SIEM, where they're combined with data from other sources for comprehensive security monitoring.

This architecture is powerful because it combines Zeek's deep network visibility with the SIEM's ability to correlate across multiple data sources. For example, Zeek might detect a host making a suspicious outbound connection. That information flows into your SIEM, which correlates it with endpoint logs showing the initial infection vector, authentication logs showing subsequent lateral movement, and firewall logs showing data exfiltration attempts. The SIEM gives you the complete picture of the attack campaign, while Zeek provides the critical network component of that picture.

Many organizations structure their Zeek-to-SIEM pipeline using log shipping tools like Filebeat or Logstash. Zeek generates its logs in tab-separated or JSON format, a log shipper monitors these files and forwards them to the SIEM in real-time, and the SIEM ingests and indexes them. This provides near-real-time visibility into network activity within your central security monitoring platform.

It's worth noting that for organizations without a SIEM, Zeek's logs can still be incredibly valuable. You can analyze them directly using command-line tools, Python scripts, or even spreadsheet software. However, as your Zeek deployment grows and you begin generating significant volumes of data, having a proper SIEM or log management platform becomes increasingly important for making that data actionable.

---


## **PART 4: THE ZEEK COMMUNITY AND ECOSYSTEM**

### **The Foundation: Official Documentation and Resources**

One of Zeek's greatest strengths is its vibrant community and extensive ecosystem of resources, scripts, and tools. As you embark on your Zeek learning journey, understanding how to navigate these resources is just as important as understanding the tool itself. The community has developed an enormous body of knowledge, shared scripts, and collaborative tools that you'll leverage throughout your career working with Zeek.

Let's start with the official documentation, which serves as your primary reference throughout this course and beyond. 
The Zeek documentation site, located at [docs.zeek.org](https://docs.zeek.org/en/master/), is comprehensive and well-organized, but it can be overwhelming when you're first starting out. Understanding its structure will help you find information quickly as you develop your skills.


The documentation is organized into several major sections, each serving a different purpose. The [Getting Started](https://docs.zeek.org/en/master/get-started.html) section provides introductory material about Zeek's concepts and basic operation. This is where you'll find explanations of how Zeek works, tutorials for common tasks, and guidance on your initial deployment. As you progress through this course, you'll reference this section less frequently, but it's invaluable for building your mental model of how Zeek operates.



The "Scripting" section is where you'll spend most of your time as you develop detection capabilities. This section documents Zeek's scripting language in detail, including data types, operators, control structures, and the vast library of built-in functions. Every time you're writing a script and need to know how a particular function works or what data types are available, you'll come here. The scripting documentation also includes information about Zeek's frameworks - structured collections of functionality for common tasks like threat intelligence integration, file analysis, and logging.

The "Frameworks" section deserves special mention because it documents some of Zeek's most powerful capabilities. The Intelligence Framework, which allows you to integrate threat intelligence feeds and automatically match them against network activity, is documented here. The Logging Framework, which you'll use to create custom log files, has its own comprehensive documentation. The Input Framework, used to read data from external files and databases, is explained in detail. As you tackle more sophisticated detection scenarios, you'll rely heavily on these frameworks.

Finally, the reference documentation provides complete details about every built-in event, function, type, and variable in Zeek. When you're writing scripts and need to know exactly what parameters a particular event provides, or what fields are available in a connection record, this is where you'll find that information. The reference documentation is exhaustive but can be dry - it's meant as a lookup resource rather than tutorial material.

Beyond the official documentation, the Zeek project maintains several other critical resources. The GitHub repository at github.com/zeek/zeek contains Zeek's source code and serves as the hub for development activity. If you encounter a bug or want to request a feature, this is where you'll file an issue. The repository also includes the issue tracker's history, which can be valuable when you're troubleshooting problems-chances are someone else has encountered a similar issue before.



### **The Power of Community: Packages and Shared Scripts**

One of Zeek's most valuable aspects is the community's culture of sharing detection scripts and protocol analyzers. Rather than each organization building everything from scratch, the community has developed a rich ecosystem of packages that extend Zeek's capabilities. Understanding how to find, evaluate, and leverage these packages will dramatically accelerate your threat hunting capabilities.

The central hub for community packages is [packages.zeek.org](https://packages.zeek.org), which serves as a searchable repository of contributed scripts and analyzers. At the time of this writing, the repository contains hundreds of packages covering everything from protocol parsers for obscure protocols to sophisticated detection logic for specific attack techniques. Many of these packages represent hundreds or thousands of hours of development effort, freely shared with the community.



Let's walk you through some of the most important packages that you'll likely use in your threat hunting work, because understanding what's available will inform how you approach detection problems later in this course.

The JA3 package, developed by researchers at Salesforce, is one of the most widely deployed community packages. JA3 provides fingerprinting of SSL/TLS clients and servers based on the parameters used during the TLS handshake. This might sound esoteric, but it has profound implications for threat detection. Many malware families use consistent TLS parameters when establishing encrypted connections, creating unique fingerprints that can identify them even when the traffic itself is encrypted. By generating JA3 hashes for TLS connections observed on your network and comparing them against databases of known-malicious fingerprints, you can detect malware communications over encrypted channels. This is particularly valuable given the increasing prevalence of encrypted malware command-and-control traffic.

Similarly, the HASSH package provides fingerprinting for SSH communications. SSH is another protocol where implementation details create unique fingerprints. Attackers who establish SSH backdoors or use SSH for lateral movement often have distinctive SSH fingerprints that differ from legitimate administrative tools. HASSH enables you to detect these anomalies.

The BZAR package, maintained by MITRE, focuses on detecting specific attack techniques from the ATT&CK framework. BZAR includes detection logic for techniques like credential dumping, lateral movement using SMB, and various reconnaissance activities. Rather than building these detections from scratch, you can deploy BZAR and immediately gain coverage for a range of common attack techniques. As you become more proficient with Zeek, you might customize BZAR's detections or use them as templates for your own detection logic.

Several packages focus on detecting specific types of threats. The zeek-httpattacks package includes signatures and behavioural detections for web application attacks like SQL injection and cross-site scripting. The zeek-log4j package was rapidly developed in response to the Log4j vulnerability and can detect exploitation attempts. The zeek-EternalSafety package detects exploitation attempts related to the EternalBlue vulnerability that was used in the WannaCry ransomware attack.

Other packages extend Zeek's protocol coverage. While Zeek includes native support for many common protocols, it can't possibly include every protocol in existence. The community has developed packages for analyzing protocols ranging from industrial control systems (Modbus, DNP3, BACnet) to messaging protocols (MQTT, AMQP) to various proprietary protocols. If you need to monitor a protocol that Zeek doesn't support natively, there's a good chance someone in the community has already developed a parser for it.

The Zeek Package Manager, called `zkg`, makes it straightforward to discover and install these packages. The command `zkg list` shows all available packages, `zkg search <keyword> `helps you find packages related to specific topics, and `zkg install <package-name>` installs a package and makes it available to Zeek. We'll work with `zkg` hands-on in a later lesson, but it's important to know now that this ecosystem exists and that you don't need to build everything yourself.



### **Threat Intelligence Integration: Leveraging Community Feeds**

A critical component of effective threat hunting is threat intelligence - knowledge about indicators of compromise, attacker techniques, and emerging threats. Zeek's Intelligence Framework provides powerful capabilities for integrating threat intelligence into your monitoring, and the community has made numerous high-quality intelligence feeds available for free or at low cost.

Understanding what intelligence feeds are available and how they can enhance your detection capabilities will inform how you approach threat hunting throughout this course. When you write detection scripts later, you'll often combine behavioural analysis with intelligence feed matching to create highly accurate detections with low false positive rates.

Let's explore some of the most valuable intelligence feed sources that you'll want to integrate with your Zeek deployment.

[Abuse.ch](https://abuse.ch) operates several freely available feeds that are particularly valuable for network security monitoring. Their [URLhaus](https://urlhaus.abuse.ch) feed provides real-time intelligence about malicious URLs being used for malware distribution. When your users browse the web, Zeek can automatically check accessed URLs against this feed and alert if anyone attempts to visit a known-malicious site.

The [Feodo Tracker](https://feodotracker.abuse.ch) feed from Abuse.ch focuses on command-and-control infrastructure for banking trojans and other malware families. These are IP addresses and domains actively being used by malware to communicate with attacker-controlled servers. By loading this feed into Zeek's Intelligence Framework, you can detect if any systems on your network are communicating with known C2 infrastructure - a strong indicator of compromise.

The [SSL Blacklist](https://sslbl.abuse.ch) feed, also from Abuse.ch, tracks malicious SSL certificates. When Zeek observes SSL/TLS connections, it extracts certificate details and can compare the certificate fingerprints against this blacklist. Malware often uses distinctive certificates for its encrypted communications, making certificate-based detection particularly effective.

AlienVault's [Open Threat Exchange (OTX)](https://otx.alienvault.com) is a community-driven threat intelligence platform that aggregates indicators from thousands of contributors worldwide. Security researchers, malware analysts, and security teams share intelligence about threats they've observed in the form of "pulses" - collections of related indicators describing specific threats or campaigns. OTX provides APIs that allow you to programmatically retrieve intelligence and feed it into Zeek. The indicators include IP addresses, domains, URLs, file hashes, and more, all tagged with information about the associated threats.

The [MISP Project](https://www.misp-project.org) (Malware Information Sharing Platform) is an open-source threat intelligence platform widely used by CERTs, security teams, and intelligence organizations for sharing structured threat information. Many organizations operate MISP instances to share intelligence within trusted communities. MISP can export intelligence in formats that Zeek's Intelligence Framework can consume, enabling automated sharing of threat intelligence between organizations and tools.




Commercial threat intelligence providers also offer feeds compatible with Zeek. Companies like Recorded Future, ThreatConnect, and others provide curated intelligence feeds that can be integrated through Zeek's Intelligence Framework or through custom scripts. While these commercial feeds require subscriptions, they often provide intelligence that's earlier, more accurate, or more comprehensive than free community feeds.

Beyond external feeds, many organizations generate their own internal threat intelligence from incident response activities, malware analysis, and threat research. When your team analyzes a malware sample and identifies its C2 infrastructure, those indicators can be fed into Zeek to detect other systems that might be compromised by the same malware. When you investigate a phishing campaign and identify the attacker's infrastructure, that intelligence can be used to detect future attacks from the same adversary.




## **PART 5: PRACTICAL EXERCISES**

Now it's time to move from theory to practice. The exercises in this section are designed to familiarize you with the resources we've discussed and help you begin building your personal knowledge base about Zeek. Unlike later lessons where you'll be writing scripts and analyzing traffic, these exercises focus on navigation, research, and preparation. They're building the foundation you'll need for the hands-on work ahead.

### **Exercise 1: Deep Dive into Official Documentation**

Your first task is to become comfortable navigating Zeek's official documentation. This isn't about reading every page - that would take weeks. Instead, you're going to explore the documentation's structure and locate specific types of information. This skill will save you countless hours as you develop detection capabilities later in the course.

Open your web browser and navigate to [docs.zeek.org](https://docs.zeek.org). Take a few minutes to explore the main navigation menu and understand how the documentation is organized. You'll see sections for "Getting Started," "Scripting," "Frameworks," and reference documentation.


Now let's find specific information to understand how this documentation works. Navigate to the Scripting section and locate the page that describes Zeek's built-in data types. You should find descriptions of types like addr (representing IP addresses), port (representing network ports), time (representing timestamps), and many others. Read through the description of the addr type. Notice how the documentation explains not just what the type is, but how to use it, what operations it supports, and common patterns for working with it.

Next, let's explore the event reference. In many places throughout the documentation, you'll see references to events that Zeek generates. Find the reference documentation for events and browse through the list. It's quite extensive - Zeek generates events for dozens of protocols and network activities. Locate the `connection_state_remove` event. This event fires when a connection terminates, and you'll use it frequently in your detection scripts. Read its documentation and notice what information is provided to your script when this event fires.



Now let's look at the Framework documentation. Navigate to the Intelligence Framework documentation and read through the overview. Even though you won't be writing intelligence framework scripts yet, understanding how the framework operates will inform your threat hunting strategy. Notice how the framework allows you to specify indicator types, matching conditions, and actions to take when indicators are observed.

Finally, familiarize yourself with the reference documentation's structure. Find the page that documents the connection record -a data structure that represents a network connection. This record is passed to many events and contains fields describing the connection. Read through the field descriptions and note the wealth of information available about each connection: source and destination addresses and ports, protocol, service identification, duration, byte counts, connection state, and more.

Your deliverable for this exercise is to document your findings. Create a text file or document where you write down the following:

1. Three events that would be useful for detecting command-and-control traffic. For each event, write a sentence explaining why it would be useful. Think about what characteristics of C2 traffic these events would help you observe.
2. Five fields from the connection record that would be valuable for behavioural analysis. Explain what each field represents and how you might use it in threat hunting.
3. Two frameworks that seem particularly relevant for your threat hunting goals, with a brief note about what each framework does.

This exercise should take you about thirty minutes. Don't rush - the goal is to become comfortable finding information in the documentation, not to memorize everything you see.

### **Exercise 2: Exploring the Package Ecosystem**

Now let's explore the community package repository and identify tools that will support your threat hunting objectives. Open your browser and navigate to [packages.zeek.org](https://packages.zeek.org). You'll see a list of available packages, each with a brief description.



Start by using the search functionality to find packages related to specific topics.
- Search for "malware" and see what comes up. You should find packages designed to detect various types of malware behaviour.
- Search for "TLS" or "SSL" and examine the packages related to encrypted traffic analysis.
- Search for "HTTP" and look at packages focused on web traffic analysis.

Now let's investigate some specific packages in detail. Find the JA3 package and click through to its full description. You'll typically find a link to the package's GitHub repository where you can see the actual code, read more detailed documentation, and understand how to use it. Read about what JA3 fingerprinting is and how it works. Even if you don't understand all the technical details yet, grasp the core concept: JA3 creates a fingerprint of TLS clients based on the parameters they use, allowing identification of malware even in encrypted traffic.

Locate the BZAR package. Read about how it implements detections for ATT&CK techniques. Visit its GitHub repository and look at the README file. Notice how the package documentation explains what it detects and provides examples of the alerts it generates. This is the kind of documentation you should aim for when you eventually share your own scripts.


Your deliverable for this exercise is a list of five packages you plan to install and use with your Zeek sensor. For each package, document:

1. The package name and where to find it
2. What problem it solves or what capability it provides
3. Why it's relevant to your threat hunting objectives
4. Any prerequisites or dependencies it has

This exercise should take about twenty to thirty minutes. The goal is to understand what the community has already built so you can leverage that work rather than reinventing solutions.




### **Exercise 3: Threat Intelligence Feed Research**

Let's explore the threat intelligence feeds that you can integrate with Zeek. Understanding what intelligence is available will shape how you approach detection - many sophisticated detection strategies combine behavioural analysis with intelligence feed matching.

Start by visiting abuse.ch, one of the most valuable sources of free threat intelligence. Navigate to their URLhaus project and examine the feed. Click through to see examples of the malicious URLs they track. Notice the level of detail provided: the URL itself, the malware family it's associated with, when it was first observed, the threat type, and more. Download a sample of the feed data and examine the format. You'll notice it's structured in a way that can be consumed by automated tools like Zeek.

Now explore their Feodo Tracker feed, which focuses on C2 infrastructure. Look at the types of information provided about each C2 server: the IP address or domain, the port, the malware family, the first and last times it was observed online, and confidence levels. This is exactly the kind of intelligence that's valuable for detecting compromised systems communicating with C2 infrastructure.

Visit AlienVault's Open Threat Exchange at otx.alienvault.com. You'll need to create a free account to fully explore the platform. Once logged in, browse through recent "pulses" - curated collections of indicators related to specific threats. Look for pulses related to RATs. A typical pulse might include IP addresses used for C2, domains registered by the attackers, file hashes of malware samples, and YARA rules for detecting the malware. Notice how different indicator types work together to provide comprehensive intelligence about a threat.

Finally explore the MISP Project website at misp-project.org. While you won't set up a full MISP instance right now, understanding what MISP is and how organizations use it will inform your intelligence strategy. Read about how MISP enables structured threat intelligence sharing and how communities use it to collaborate on threat information. Note that many organizations and CERTs operate MISP instances that share intelligence feeds in formats Zeek can consume.

Your deliverable for this exercise is documentation of three threat intelligence feeds you plan to integrate with Zeek. For each feed, document:

1. The feed source and how to access it
2. What types of indicators it provides (IPs, domains, URLs, file hashes, etc.)
3. How frequently it's updated
4. How you'll integrate it with Zeek (we'll cover the technical details later, but make note of the feed format and delivery mechanism)
5. Why this feed is particularly valuable for your threat hunting objectives

This exercise should take about twenty to thirty minutes. The goal is to identify intelligence sources that will enhance your detection capabilities before you begin writing scripts.


## **KNOWLEDGE VALIDATION**

Before moving on to the next lesson, take a few minutes to test your understanding of the concepts covered in this lesson. These questions aren't meant to be tricky - they're designed to ensure you've grasped the fundamental concepts that will inform everything that follows.

Think about the fundamental philosophical difference between Zeek and Snort. If you were explaining to a colleague who's familiar with Snort why they should also deploy Zeek, what key points would you make? How would you explain the complementary nature of signature-based detection and behavioral analysis without suggesting that one is better than the other?

Consider the three types of threat intelligence indicators that Zeek's Intelligence Framework can consume. Can you name at least three different types and explain why each type is valuable? Think about how different indicator types address different aspects of threat detection.

Here's a true-or-false question that gets at an important misconception: Zeek can operate in inline IPS mode to actively block malicious traffic. Take a moment to consider why the answer is false and what implications this has for Zeek's role in a security architecture.

Reflect on the primary advantage of Zeek's event-driven architecture for detecting unknown threats. How does operating at the event level rather than the packet level enable detection of novel attacks? Think about the role of state tracking and protocol context in this capability.

Consider the two community packages we discussed that help detect malware in encrypted TLS traffic. Can you name them and explain at a high level how they work despite traffic being encrypted? This question tests whether you understand that analysis of encrypted traffic metadata can still reveal malicious activity.

Finally, think about why Zeek is often deployed alongside a SIEM rather than as a replacement for one. What unique capabilities does each system bring, and how do they complement each other? This question gets at understanding Zeek's role in a comprehensive security architecture.

Take your time with these questions. If you're uncertain about any of them, review the relevant sections of this lesson before proceeding. Understanding these foundational concepts is crucial for everything that follows.

---


