## Understanding Sysmon Event IDs for Threat Hunting

### What Are Sysmon Event IDs?

Sysmon (System Monitor) generates telemetry by monitoring and logging specific system activities to the Windows event log. Each distinct type of activity Sysmon can monitor is assigned a unique **Event ID** - a numerical identifier that categorizes the nature of the logged event.

When Sysmon detects activity matching your configuration rules, it creates an event record in the `Microsoft-Windows-Sysmon/Operational` event log. The Event ID is the first piece of information that tells you what type of activity occurred: was it a process being created? A network connection established? A registry key modified? Each has its own Event ID.

Event IDs are your primary lens for understanding system behavior through Sysmon. They are the fundamental building blocks of your telemetry - the raw data streams from which threat detection, incident investigation, and behavioral analysis all flow.

### Event IDs as Fundamental Units of Telemetry

Think of Event IDs as different "sensors" or "cameras" positioned throughout the Windows operating system, each watching a specific type of activity:

- **Event ID 1** watches process creation - every time a new program starts
- **Event ID 3** watches network connections - every time a process communicates over the network
- **Event ID 13** watches registry writes - every time a value is set in the registry
- And so on...

Each event type captures different metadata relevant to that activity. A process creation event (ID 1) includes the command line, parent process, and user context. A network event (ID 3) includes source/destination IPs, ports, and protocols. A registry event (ID 13) includes the key path, value name, and data written.

By combining these different event streams, you can reconstruct the complete story of what happened on a system: what was executed, what it did, where it connected, what it modified, and how it persisted.

### The 29 Sysmon Event Types

As of Sysmon v15, there are 29 distinct event types. Below is a complete reference:

|Event ID|Event Name|Description|
|---|---|---|
|1|Process Creation|Process started (including command line arguments)|
|2|File Creation Time Changed|File creation timestamp modified (timestomping)|
|3|Network Connection|TCP/UDP connection initiated|
|4|Sysmon Service State Changed|Sysmon service started or stopped|
|5|Process Terminated|Process ended|
|6|Driver Loaded|Kernel driver loaded into memory|
|7|Image Loaded|DLL/module loaded by a process|
|8|CreateRemoteThread|Thread created in another process|
|9|RawAccessRead|Direct disk read bypassing filesystem|
|10|ProcessAccess|Process opened another process (for reading/writing memory)|
|11|FileCreate|File created or overwritten|
|12|RegistryEvent (Object)|Registry key/value created or deleted|
|13|RegistryEvent (Value Set)|Registry value modified|
|14|RegistryEvent (Rename)|Registry key or value renamed|
|15|FileCreateStreamHash|Alternate data stream created|
|16|ServiceConfigurationChange|Service configuration modified|
|17|PipeEvent (Created)|Named pipe created|
|18|PipeEvent (Connected)|Named pipe connection established|
|19|WmiEvent (Filter)|WMI event filter registered|
|20|WmiEvent (Consumer)|WMI event consumer registered|
|21|WmiEvent (Consumer-Filter)|WMI filter bound to consumer|
|22|DNSEvent|DNS query made|
|23|FileDelete|File deleted (sent to recycle bin or permanent)|
|24|ClipboardChange|Clipboard content changed|
|25|ProcessTampering|Process memory/image tampered with|
|26|FileDeleteDetected|File delete operation detected|
|27|FileBlockExecutable|Executable file block detected|
|28|FileBlockShredding|File shredding operation blocked|
|29|FileExecutableDetected|Executable file creation detected|



### Not All Event IDs Are Created Equal

While it's valuable to understand all 29 event types, **they are not equally relevant for threat hunting**. Some are universal and foundational - you'll use them in nearly every investigation. Others are highly specialized, designed to detect very specific attacker techniques or malware behaviou rs.

#### Universal/Foundational Events

These event types provide broad visibility and are relevant to most threat hunting activities:

- **Event ID 1 (Process Creation)** - The cornerstone of endpoint telemetry. Nearly every attack involves executing code.
- **Event ID 3 (Network Connection)** - Essential for detecting C2 communications, lateral movement, and exfiltration.
- **Event ID 11 (FileCreate)** - Critical for tracking malware drops, staging, and payload delivery.
- **Event ID 13 (Registry Value Set)** - Key for detecting persistence mechanisms and configuration changes.
- **Event ID 22 (DNS Query)** - Powerful for detecting C2 domains, beaconing, and infrastructure.

These five event types alone provide tremendous threat hunting value and should be prioritized in any Sysmon deployment.

#### Targeted/Specialized Events

Other event types are more specialized, designed to detect specific techniques:

- **Event IDs 17/18 (Named Pipes)** - Primarily useful for detecting specific malware families that use named pipes for C2 or inter-process communication (for ex Agent-to-Agent SMB beacons).
- **Event IDs 19/20/21 (WMI Events)** - Target WMI-based persistence and lateral movement techniques.
- **Event ID 8 (CreateRemoteThread)** - Detects a specific process injection technique.
- **Event ID 25 (ProcessTampering)** - Catches process hollowing and memory manipulation.

These are extremely valuable when hunting for specific attacker TTPs, but you may not reference them as frequently as the universal events.

#### Supporting/Context Events

Some events primarily provide supporting context or detect edge-case scenarios:

- **Event ID 5 (Process Terminated)** - Useful for determining process lifetime and calculating process duration.
- **Event ID 7 (Image Loaded)** - High volume, but essential for detecting DLL hijacking and suspicious module loads.
- **Event ID 2 (File Time Changed)** - Specifically detects anti-forensics (timestomping).
- **Event ID 24 (ClipboardChange)** - Niche, but can detect credential theft or data collection from clipboard.

### Event ID to Threat Hunting Matrix

The table below maps each event type to common adversary techniques and threat hunting categories.

|Event ID|Event Name|Execution|Persistence|Privilege Escalation|Defense Evasion|Credential Access|Discovery|Lateral Movement|Collection|C2|Exfiltration|Process Injection|
|---|---|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|1|Process Creation|**X**|**X**|**X**|**X**|**X**|**X**|**X**|||||
|2|File Time Changed||||**X**||||||||
|3|Network Connection|||||||**X**||**X**|**X**||
|4|Service State Changed||||**X**||||||||
|5|Process Terminated||||**X**||||||||
|6|Driver Loaded||**X**|**X**|**X**||||||||
|7|Image Loaded||**X**|**X**|**X**|**X**||||||**X**|
|8|CreateRemoteThread|||**X**||||||||**X**|
|9|RawAccessRead||||**X**|**X**|||||||
|10|ProcessAccess|||||**X**||||||**X**|
|11|FileCreate|**X**|**X**||**X**|||**X**|||||
|12|Registry Object||**X**|**X**|**X**||||||||
|13|Registry Value Set||**X**|**X**|**X**||||||||
|14|Registry Rename||||**X**||||||||
|15|FileCreateStreamHash||**X**||**X**||||||||
|16|Service Config Change||**X**|**X**|**X**||||||||
|17|Pipe Created|||||||**X**||**X**|||
|18|Pipe Connected|||||||**X**||**X**|||
|19|WmiEvent Filter|**X**|**X**|||||**X**|||||
|20|WmiEvent Consumer|**X**|**X**|||||**X**|||||
|21|WmiEvent Binding|**X**|**X**|||||**X**|||||
|22|DNS Query||||||**X**|||**X**|**X**||
|23|FileDelete||||**X**||||||||
|24|ClipboardChange|||||**X**|||**X**||||
|25|ProcessTampering|||**X**|**X**|||||||**X**|
|26|FileDeleteDetected||||**X**||||||||
|27|FileBlockExecutable|**X**|||**X**||||||||
|28|FileBlockShredding||||**X**||||||||
|29|FileExecutableDetected|**X**|**X**||**X**||||||||

**Reading the Matrix:**

- **Execution** - Initial code execution, scripting, command interpreters
- **Persistence** - Mechanisms to maintain access across reboots
- **Privilege Escalation** - Gaining higher-level permissions
- **Defense Evasion** - Avoiding detection, obfuscation, anti-forensics
- **Credential Access** - Dumping, stealing, or accessing credentials
- **Discovery** - Reconnaissance, enumeration, situational awareness
- **Lateral Movement** - Moving between systems in the network
- **Collection** - Gathering data of interest
- **C2 (Command and Control)** - Communication with attacker infrastructure
- **Exfiltration** - Stealing data from the environment
- **Process Injection** - Injecting code into other processes
