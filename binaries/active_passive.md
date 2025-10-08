# The Active-Passive Analysis Line

## Understanding the Risk

When you investigate a potentially malicious process, you face a critical decision: how to gather information without revealing your presence to the attacker. This is what we call "crossing the active-passive line."

The distinction is straightforward. Passive analysis involves examining data that already exists - logs, telemetry, historical records. You're reviewing information that was captured during normal system operation. The malware has no way to detect that you're analyzing these records because you're not interacting with it directly.

Active analysis, in contrast, requires you to interact with the system or process in real-time. You might open the suspicious process in a monitoring tool, read the binary file from disk, inspect its memory, or attach debugging tools. These actions can potentially be detected by sophisticated malware that has been designed with anti-analysis capabilities.

The fundamental difference:

```
PASSIVE ANALYSIS          |  ACTIVE ANALYSIS
--------------------------|---------------------------
✓ Log review              |  ✓ Process inspection
✓ Sysmon data             |  ✓ Memory dumps
✓ EDR telemetry           |  ✓ Direct file access
✓ Network logs            |  ✓ System Informer queries
✓ Historical artifacts    |  ✓ Live debugging
```

## How Malware Detects Your Investigation

Modern malware, particularly tools used by experienced threat actors, often includes anti-analysis capabilities. These mechanisms allow the malware to detect when it's being investigated and respond accordingly.

Here's what malware can potentially detect:

**Process enumeration tools opening handles**: When you open System Informer or similar tools and inspect a process, your tool must request a handle to that process from the operating system. Malware can monitor for these handle requests and detect when another process is attempting to examine it.

**File reads on the malicious binary**: If you navigate to the malware's location on disk and open the file for analysis, the malware can detect file access events. Some malware even monitors its own executable file and triggers defensive actions if it detects unauthorized access.

**Memory inspection**: Advanced malware can detect attempts to read its process memory, which might indicate memory forensics or debugging attempts. Memory inspection often requires specific Windows API calls that can be monitored.

**Debugger attachment**: Malware commonly checks for debuggers using various techniques - from simple API calls like `IsDebuggerPresent()` to more sophisticated timing checks that detect the performance impact of debugging.

**API monitoring**: Some malware detects when API monitoring tools or behaviour analysis frameworks are present in the system, indicating a security research or analysis environment.

## Consequences of Detection

When sophisticated malware detects your investigation, it can respond in several ways, all of which are potentially detrimental to your organization:

- **Termination:** The simplest strategy is for the malware to just shut down. This prevents the analyst from observing its malicious activity.
- **Altered Behaviour:** The malware might continue running but change what it does. For example, it might stop communicating with its command-and-control server or refrain from encrypting files. This makes it appear benign to the analyst.
- **Implant destruction**: The attacker may immediately burn their access, deleting the malware and covering their tracks. While this removes the immediate threat, it also destroys evidence and prevents you from understanding the full scope of the compromise.
- **Evidence destruction**: The malware might rapidly delete logs, remove artifacts, or destroy other evidence of the attack before terminating, making forensic investigation significantly more difficult.
- **Crashing the Analysis Tool:** Some malware is designed to be aggressive and may try to crash the tool that's analyzing it.
- **Providing Misleading Information:** More sophisticated malware might feed the analysis tool fake or misleading data to confuse the analyst.
- **Accelerated lateral movement**: If the attacker realizes they've been detected, they may immediately attempt to spread to additional systems while they still have access, expanding the compromise before you can contain it.
- **Premature ransomware deployment**: Perhaps most dangerously, if an attacker with ransomware capabilities realizes they're about to lose access, they may deploy their ransomware immediately - before you're ready to respond - causing maximum damage.

## The Passive-First Principle

As a network threat hunter, you operate under a critical constraint: **you generally should not cross the line from passive to active analysis.**

### What This Means

**Passive analysis** means examining data that already exists - logs, telemetry, historical records. The suspicious process doesn't know you're investigating it because you're not interacting with it directly.

**Active analysis** means directly interacting with potentially malicious files or processes - opening binaries in analysis tools, inspecting running processes in real-time, extracting embedded strings from executables, or verifying digital signatures by accessing the file. These actions can potentially alert sophisticated malware to your investigation.

### Why Network Threat Hunters Stay Passive

As a threat hunter focused on rapid triage and validation of network detections, your role is to:

1. Quickly determine if a network alert represents genuine malicious activity
2. Gather sufficient evidence to confidently escalate or clear
3. Maintain operational tempo across many alerts

**You achieve 80%+ confidence through passive analysis alone.** The five core indicators available passively (process location, name, hash, command line, parent process) are sufficient for the vast majority of investigations.

### When Passive Analysis Isn't Enough

If you've completed your passive analysis and still cannot reach a confident decision, **escalate to IR or malware analysis specialists** rather than crossing into active analysis yourself. They have:

- Specialized training in safely conducting active analysis
- Controlled environments for examining malicious files
- Tools and expertise for deep malware analysis
- Time to conduct thorough investigations



## Primary Passive Sources

**Sysmon Logs**

Sysmon (System Monitor) is your most valuable passive data source. It captures detailed process and system activity and logs it to the Windows Event Log, where you can query it without any interaction with potentially malicious processes.

Key Event IDs:

- **Event ID 1 (Process Creation)**: Captures every process launch, including the full command line, parent process, user context, and file hashes (the five core indicators)
- **Event ID 3 (Network Connection)**: Logs every network connection made by any process, including source and destination IPs and ports
- **Event ID 7 (Image Loaded)**: Records DLL loads, which can reveal process injection
- **Event ID 10 (Process Access)**: Logs when one process accesses another, indicating potential credential dumping or injection
- **Event ID 22 (DNS Query)**: Captures DNS queries made by processes, revealing C2 domains even before connections are established

**EDR/XDR Telemetry**

If your organization uses an Endpoint Detection and Response (EDR) or Extended Detection and Response (XDR) platform, this provides another rich source of passive data:

- Historical process execution records, often with more context than Sysmon alone
- File creation, modification, and deletion events
- Registry modifications
- Often includes automatic behavioral analysis and threat intelligence correlation

**Windows Event Logs**

Standard Windows Event Logs provide additional context:

- **Security Log Event 4688**: Process creation events (requires Group Policy configuration to enable command line logging)
- **Application and System Logs**: May contain error messages or unusual events related to malware execution

**Prefetch Files**

Windows Prefetch files (`C:\Windows\Prefetch\`) maintain execution history:

- Provides a timeline of when programs were executed
- Lists files accessed by each program during execution
- Useful for establishing persistence and understanding program behavior

**PowerShell ScriptBlock Logs**

PowerShell ScriptBlock logging captures the actual content of PowerShell commands and scripts executed on the system, providing critical visibility into PowerShell-based attacks.

PowerShell is one of the most common attack vectors. Attackers use it for:

- Downloading and executing malware
- Credential theft
- Lateral movement
- Living-off-the-land techniques

ScriptBlock logging reveals exactly what PowerShell commands were executed, even if they were encoded or obfuscated. When you see `powershell.exe -enc [base64]` in Sysmon, ScriptBlock logs show you the decoded content.

### When Active Analysis Becomes Necessary

Despite the preference for passive analysis, there are legitimate situations where active analysis is needed. However, **as a network threat hunter, this is your signal to escalate to IR or malware analysis specialists** rather than conducting active analysis yourself.

**Escalate to specialists when:**

1. **Passive sources are insufficient**: You've exhausted your logs and telemetry and still cannot make a confident determination about whether the activity is malicious.
2. **Risk has been assessed**: The situation requires active analysis, and specialists can assess whether the intelligence value outweighs the risk of detection.
3. **Preparation for consequences**: Specialists have response plans ready in case the investigation triggers a defensive reaction from the malware.
4. **IR team coordination**: The incident response team needs to be coordinated and prepared for what might happen during active analysis.

The key is intentionality. Active analysis should never be your default first step as a threat hunter. Your role is rapid triage through passive means. When passive is insufficient, you escalate to those with specialized training and controlled environments for safe active analysis.

**Exception**: If you are also trained in malware analysis AND the risk assessment justifies it, then you may conduct active analysis yourself. But for standard threat hunting operations - stay passive, escalate when needed.

## Practical Workflow

In practice, your investigation workflow should follow this pattern:

**Phase 1: Passive Intelligence Gathering**

- Query Sysmon logs for process creation and network connection events
- Check Windows Event Logs + PS ScriptBlock Logs for relevant security events
- Examine prefetch files if needed for execution history

**Phase 2: Passive Analysis**

- Analyze all gathered data to understand the five core indicators (location, name, hash, command line, parent process)
- Research file hashes against threat intelligence databases (VirusTotal, MalwareBazaar)
- Map out parent-child process relationships from logs
- Identify network connections and destinations from passive network logs

**Phase 3: Decision Point**

- Can you make a determination based on passive data alone?
- If yes → Make your decision and proceed with appropriate action
- If no → Escalate to IR/malware analysis specialists rather than going active yourself

**Phase 4: Active Analysis (if necessary and authorized)**

- Specialists ensure system isolation if possible
- Specialists use tools like System Informer to inspect running processes
- Specialists use PEStudio to analyze binary files
- All active analysis steps are documented

This structured approach ensures that you minimize risk while maximizing the intelligence value of your investigation. The vast majority of your investigations will be resolved in Phases 1 and 2 through passive analysis of the five core indicators, never requiring active analysis at all.

___

