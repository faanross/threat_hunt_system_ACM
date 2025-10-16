# Network-Based vs. Endpoint-Based Threat Hunting

## Two Windows into Adversary Activity

Imagine trying to understand what's happening inside a building. You have two options: stand outside and watch everyone who enters and exits, observing their comings and goings, or walk through the building's interior, examining each room and what's happening inside them. Each perspective reveals different information. The external observer sees movement patterns, frequency of visits, and connections between different visitors. The internal observer sees detailed activities within rooms and what they're actually doing.

Threat hunting faces a similar choice: hunt using network telemetry (observing communications and movements between systems) or endpoint telemetry (observing detailed activities on individual systems). Each approach provides unique visibility into adversary operations. Each has distinct strengths and limitations that compliment one another.

This distinction between network-based and endpoint-based hunting isn't just a technical consideration - it fundamentally shapes how you hunt, what threats you detect most effectively, and what blind spots remain. Organizations that understand these differences can make informed decisions about where to invest, how to structure hunting programs, and when to expand from one domain to the other. Those that don't risk investing in capabilities that don't address their actual threat profile or leaving critical visibility gaps.

This chapter explores network-based versus endpoint-based threat hunting comprehensively: what each approach reveals, what it misses, what threats each best detects, and how to decide where your organization should focus. Most importantly, it explains why mature hunting programs eventually need both perspectives to achieve comprehensive threat visibility.

## Understanding Network-Based Hunting

Network-based threat hunting focuses on analyzing network traffic - the communications between systems, both internal and external. It views the environment through the lens of network flows, protocols, and communication patterns.

### What Network Hunting Reveals

Network telemetry provides several distinct types of visibility:

**Communication Patterns**: Network data shows which systems communicate with which destinations, how frequently, at what times, using what protocols and ports. This reveals patterns - normal and anomalous - in how systems interact.

**External Connections**: Network visibility shows all communications leaving your environment - to the internet, cloud services, partner networks. This is critical for detecting command and control, data exfiltration, and external reconnaissance.

**Lateral Movement**: Internal network traffic reveals systems communicating with each other within your environment. Unusual patterns often indicate adversary lateral movement between compromised systems.

**Protocol Analysis**: Deep packet inspection reveals how protocols are being used - whether HTTP contains suspicious requests, whether DNS queries show tunneling patterns, whether SMB traffic indicates file transfers or authentication attempts.

**Volume and Timing**: Network data reveals how much data moves between systems and when. Unusual volumes or timing patterns often indicate malicious activity - large data transfers to external destinations, regular beaconing consistent with command and control.

**Encrypted Traffic Metadata**: Even when traffic is encrypted (increasingly common), network analysis reveals metadata - destination IP addresses, connection frequency, packet sizes, timing patterns. This metadata often reveals adversary activity despite encryption.

### Network Data Sources

Network-based hunting leverages several data sources:


**Flow Data (NetFlow/IPFIX)**: Summarized records of network connections - source/destination IPs and ports, protocols, byte counts, timestamps. Flow data provides high-level visibility across entire networks efficiently.

**Full Packet Capture (PCAP)**: Complete capture of network traffic including packet contents. Provides maximum detail but is storage-intensive and limited to shorter retention periods or specific network segments.

**Zeek Logs**: Protocol-aware network monitoring that generates structured logs from deep packet inspection.  Provides protocol-level visibility without full packet capture storage requirements, making it effective for detecting anomalous protocol usage, and malware communication.

**DNS Logs**: Records of DNS queries and responses. DNS is critical for detecting command and control (through domain lookups), data exfiltration (through DNS tunneling), and adversary reconnaissance.

**Proxy Logs**: Web proxy logs showing HTTP/HTTPS requests, user agents, URLs accessed. Provides visibility into web-based threats and data exfiltration through web channels.

**Firewall Logs**: Records of connections allowed and blocked at network perimeter. Shows external connection attempts and provides context for what's being permitted or denied.

**IDS/IPS Alerts**: Intrusion detection/prevention system alerts about suspicious network patterns. While not raw data, these alerts provide valuable starting points for network hunting.

**SSL/TLS Inspection**: When implemented, decryption of SSL/TLS traffic enables inspection of encrypted communications. Privacy and technical considerations limit deployment but provides unique visibility where implemented.

### Network Hunting Strengths

Network-based hunting excels at detecting specific threat types:

**Command and Control Detection**: Most malware requires some form of outbound communication, which is often mediated by C2. Network hunting detects can detect this through beaconing patterns (regular, predictable communications), unusual external destinations, suspicious DNS queries, or protocol anomalies.

**Data Exfiltration**: Large data transfers to external destinations, especially to unusual locations or using suspicious protocols, indicate data theft. Network visibility is essential for detecting exfiltration because data must leave via network.

**Lateral Movement**: Adversaries moving between systems create network connections. Unusual internal communications - systems communicating that normally don't, unusual port usage, suspicious SMB or WMI traffic - indicate lateral movement.

**External Reconnaissance**: Adversaries scanning your environment from external locations generate network traffic. Unusual connection attempts, port scans, and reconnaissance patterns are visible in network data.

**Encrypted Malware Communications**: When adversaries use encrypted C2 channels, endpoint visibility may see only encrypted traffic. Network analysis can detect suspicious patterns in encrypted traffic through metadata analysis.

### Network Hunting Limitations

Network-based hunting has significant blind spots:

**No On-Host Visibility**: Network data shows communications between systems but not what's happening on systems. You see that system A connected to system B but not what processes made connections, what files were accessed, or what commands executed.

**Encryption Limitations**: Increasing encryption means packet content is often opaque. While metadata remains visible, you lose protocol-level detail that was once available.

**East-West Traffic Challenges**: In modern cloud and virtualized environments, significant traffic flows "east-west" (between systems in same data center) potentially bypassing traditional network monitoring points.

**Endpoint-Only Threats**: Threats that never create network traffic - malware executing locally without C2, insider threats using local access, attacks on isolated systems - are invisible to network hunting.

**Granularity Limits**: Network data aggregates activity. You see "system X connected to IP Y" but not which user, which process, or what specific activity occurred. This limits investigation depth.

**Volume Challenges**: Large organizations generate massive network traffic volumes. Storing, processing, and analyzing this data efficiently presents significant technical challenges.

```
Network Hunting: Strengths and Blind Spots
═══════════════════════════════════════════════════════════

EXCELLENT VISIBILITY          LIMITED/NO VISIBILITY
═══════════════════          ═══════════════════

✓ Command & Control          ✗ On-host activity details
✓ Data Exfiltration          ✗ Process execution
✓ Lateral Movement           ✗ File system operations
✓ External Connections       ✗ Registry modifications
✓ Protocol Anomalies         ✗ Memory operations
✓ Communication Patterns     ✗ Local malware execution
✓ Beaconing Behavior         ✗ User actions on endpoints
✓ Reconnaissance             ✗ Credential theft details
                             ✗ Encrypted content

Network hunting answers:     Network hunting struggles with:
• "What is communicating     • "What process made this
  with what?"                  connection?"
• "Where is data going?"     • "What commands executed?"
• "How much data moved?"     • "What files were accessed?"
• "What external destinations• "What user took actions?"
  are contacted?"            • "What happened locally?"
```

## Understanding Endpoint-Based Hunting

Endpoint-based threat hunting focuses on analyzing activity on individual systems - workstations, servers, and other endpoints. It examines processes, files, registry entries, memory, and user actions to detect threats.

### What Endpoint Hunting Reveals

Endpoint telemetry provides different but complementary visibility:

**Process Execution**: Detailed records of what processes run, with command-line arguments, parent-child relationships, and execution context. This reveals exactly what code executes on systems.

**File Operations**: Records of files created, modified, deleted, or accessed. Shows malware deployment, data collection, and adversary tool staging.

**Registry Activity**: On Windows systems, registry modifications reveal persistence mechanisms, configuration changes, and system alterations adversaries make.

**Authentication Events**: Logon attempts, privilege escalations, and credential operations. Essential for detecting credential theft, account compromise, and privilege abuse.

**Memory Analysis**: Advanced endpoint tools can inspect process memory, revealing fileless malware, injected code, and in-memory-only threats invisible to file-based detection.

**User Behaviour**: Actions users take on systems - applications launched, documents accessed, websites visited. Distinguishes malicious from legitimate activity through behavioural context.

**System Configuration**: Changes to system settings, services, scheduled tasks, and startup items. Adversaries often modify configurations for persistence or defense evasion.

**Local Network Connections**: Which processes on endpoints initiate or receive network connections. Bridges endpoint and network visibility by showing exactly which local processes create network traffic.


### Endpoint Data Sources

Endpoint-based hunting leverages different data sources than network hunting:

**EDR (Endpoint Detection and Response)**: Modern EDR platforms provide comprehensive endpoint telemetry - process execution, file operations, network connections, registry activity, and more. If available, EDR is the primary data source for endpoint hunting.

**Sysmon**: Microsoft's System Monitor (Sysmon) provides enhanced Windows logging with a focus on security - detailed process creation, network connections, and system events. Free tool that dramatically improves Windows visibility. If no EDR is available, Sysmon serves as the primary data source for endpoint hunting.

**Windows Event Logs**: System, security, and application event logs on Windows systems. Provide authentication events, system changes, and application activities. Available natively but require configuration for comprehensive coverage, and has terrible blindspots.

**Linux Auditd**: Linux audit framework providing system call monitoring, file access tracking, and user activity logging. Linux equivalent to Sysmon for enhanced logging.

**Osquery**: Cross-platform endpoint visibility tool that exposes system information as SQL-queryable tables. Enables flexible endpoint querying across Windows, Linux, and macOS.

**Application Logs**: Logs from security-relevant applications - web servers, PowerShell, databases, authentication systems. Provide context about application-level activities.

**Memory Forensics**: Tools like Volatility analyze memory dumps to detect fileless malware, code injection, and other in-memory threats. Typically used in-depth investigations rather than routine hunting.

### Endpoint Hunting Strengths

Endpoint-based hunting excels at detecting different threat types than network hunting:

**Execution Detection**: See exactly what processes run, with full command-line arguments and execution context. Detect malicious code execution, suspicious PowerShell commands, and unusual binaries.

**Persistence Mechanisms**: Detect adversary persistence through scheduled tasks, registry run keys, service creation, DLL hijacking, and other mechanisms. Endpoint visibility is essential because these are local system modifications.

**Credential Theft**: Detect credential dumping tools accessing LSASS memory, suspicious authentication patterns, and credential material access. Endpoint telemetry shows the detailed process behaviours indicating credential theft.

**Privilege Escalation**: Observe processes requesting elevated privileges, exploitation of privilege escalation vulnerabilities, and unauthorized privilege grants. Endpoint context reveals the local activities involved in privilege escalation.

**Defense Evasion**: Detect adversary attempts to disable security tools, modify logging, clear audit trails, or otherwise evade detection. These activities manifest as local system changes visible in endpoint telemetry.

**Living-off-the-Land**: Detect malicious use of legitimate system tools (PowerShell, WMI, PsExec, etc.). Endpoint visibility provides context - who executed commands, with what arguments, in what sequence - that distinguishes malicious from legitimate tool use.

**Fileless Attacks**: Detect malware executing entirely in memory without dropping files to disk. Endpoint tools with memory inspection capabilities detect these threats that bypass traditional file-based detection.


### Endpoint Hunting Limitations

Endpoint-based hunting also has blind spots:

**Limited Network Context**: Endpoint data shows that a process made a network connection but often lacks detail about where, what was communicated, or how much data transferred. Network context requires network data.

**Scalability Challenges**: Endpoint agents on every system generate massive telemetry volumes. Collecting, storing, and analyzing this data across thousands of endpoints requires significant infrastructure.

**Agent Dependencies**: Endpoint visibility requires agents installed on systems. Unmanaged devices, legacy systems unable to run modern agents, or systems where agent deployment is restricted create blind spots.

**Cloud and Container Challenges**: Traditional endpoint agents may not work in containerized or serverless cloud environments. Endpoint visibility in modern cloud architectures requires different approaches.

**Cross-System Correlation**: Endpoint data is system-centric. Understanding adversary campaigns that span multiple systems requires correlating endpoint data across systems - often challenging without additional tooling.

**Performance Impact**: Comprehensive endpoint monitoring consumes system resources. Organizations must balance visibility against performance impact, sometimes limiting what data is collected.

**Encrypted Communications**: Like network hunting, endpoint hunting struggles with encrypted communications. You see processes making encrypted connections but not communication content without additional decryption capabilities.

```
Endpoint Hunting: Strengths and Blind Spots
═══════════════════════════════════════════════════════════

EXCELLENT VISIBILITY          LIMITED/NO VISIBILITY
═══════════════════          ═══════════════════

✓ Process execution          ✗ Network path details
✓ File operations            ✗ Traffic volumes
✓ Registry modifications     ✗ External destinations
✓ Memory operations          ✗ Communication patterns
✓ Credential access          ✗ Data exfil volume
✓ Persistence mechanisms     ✗ Cross-system traffic flow
✓ User actions               ✗ Network reconnaissance
✓ Local authentication       ✗ Perimeter activities
✓ Defense evasion attempts   ✗ Encrypted content

Endpoint hunting answers:    Endpoint hunting struggles with:
• "What process ran?"        • "Where did data go?"
• "What commands executed?"  • "How much data left?"
• "What files changed?"      • "What systems were connected?"
• "Who took actions?"        • "What communication patterns
• "What persistence exists?"   exist?"
• "What memory injections?"  • "What external destinations
                               contacted?"
```

## Complementary Perspectives: Why You Need Both

The relationship between network and endpoint hunting isn't competitive - it's complementary. Each provides visibility the other lacks, and together they create comprehensive threat detection:

### The Investigation Scenario

Consider investigating suspicious activity. With only network data:

"System X contacted external IP address Y, transferring 500 MB of data over encrypted connection. The destination is a cloud storage provider. The connection occurred at 2 AM."

This raises questions: What process made this connection? What user account? What files were uploaded? Was this authorized or malicious? Network data alone can't answer these questions.

Now add endpoint data:

"Process 'svchost.exe' spawned by an unusual parent process made the connection. The process was running under a compromised user account. Before the upload, the process accessed hundreds of files from sensitive directories and staged them in a hidden temp folder."

Now you understand this is data exfiltration by a compromised system. The combination of network and endpoint data tells the complete story.

### The Blind Spot Elimination

Network and endpoint hunting eliminate each other's blind spots:

- **Network blind spot**: Can't see on-host details 
- **Endpoint solution**: Provides detailed process, file, and user context

- **Endpoint blind spot**: Limited network visibility 
- **Network solution**: Shows communication patterns, destinations, and volumes

- **Network blind spot**: Struggles with encrypted traffic content 
- **Endpoint solution**: Shows unencrypted process activity before encryption

- **Endpoint blind spot**: System-centric view limits cross-system understanding 
- **Network solution**: Provides environment-wide communication patterns

- **Network blind spot**: Limited granularity on who/what 
- **Endpoint solution**: Provides user, process, and execution context

- **Endpoint blind spot**: Agent deployment gaps 
- **Network solution**: Provides visibility for unmanaged systems via network observation


## Bridging Network and Endpoint Hunting

The most powerful hunting combines both perspectives. How do you bridge them effectively?

### Correlation Strategies

**Pivot from Network to Endpoint**:

- Network hunting detects suspicious connection
- Identify endpoint system involved
- Query endpoint data for process making connection
- Examine process creation context, parent, command line
- Investigate files accessed before/after connection
- Determine if connection is malicious or legitimate

**Pivot from Endpoint to Network**:

- Endpoint hunting detects suspicious process execution
- Identify network connections made by process
- Query network data for destinations contacted
- Examine volumes transferred, timing patterns
- Investigate other systems contacting same destinations
- Determine scope of potential compromise

### Unified Investigation Platforms

Modern security platforms increasingly unify network and endpoint visibility:

**XDR (Extended Detection and Response)**: Platforms combining EDR, network detection, cloud security, and other telemetry sources. Enable cross-domain hunting from single interface.

**SIEM with Broad Data Integration**: Security Information and Event Management systems ingesting both network and endpoint data, enabling correlated analysis.

**Data Lakes**: Centralized repositories containing both network and endpoint telemetry, queryable through unified analytics platforms.

These unified platforms reduce friction in cross-domain hunting, enabling hunters to pivot seamlessly between network and endpoint data without switching tools or correlating manually.

### The Hybrid Hunter

As hunting programs mature, consider developing "hybrid hunters" comfortable in both domains:

- **Cross-Training**: Train endpoint specialists in network fundamentals and vice versa 
- **Rotation**: Rotate hunters between network and endpoint focus areas 
- **Paired Hunting**: Pair network and endpoint specialists for joint investigations
- **Knowledge Sharing**: Regular sessions where specialists share techniques across domains

Hybrid capability doesn't mean every hunter must be equally expert in both domains, but building some cross-domain understanding dramatically improves hunting effectiveness.


## Common Pitfalls

Several mistakes occur when organizations navigate network vs. endpoint hunting:

**Pitfall 1: Analysis Paralysis** Endlessly debating which domain to start with instead of starting. Either domain provides value - choose based on practical factors and begin.

**Pitfall 2: Shallow in Both** Attempting both domains simultaneously without sufficient resources, producing superficial capability in each rather than depth in one.

**Pitfall 3: Ignoring Correlation** Operating network and endpoint hunting as completely separate activities, missing the power of combined analysis.

**Pitfall 4: Tool Over Strategy** Buying expensive unified platforms without developing hunting methodology in each domain first.

**Pitfall 5: Abandoning First Domain** Successfully implementing second domain and neglecting first, losing hard-won capability and creating blind spots.

**Pitfall 6: Domain Dogmatism** Declaring one domain "better" than the other rather than recognizing complementary strengths.

