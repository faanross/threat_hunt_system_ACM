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

- **Implant destruction**: The attacker may immediately burn their access, deleting the malware and covering their tracks. While this removes the immediate threat, it also destroys evidence and prevents you from understanding the full scope of the compromise.
- **Evidence destruction**: The malware might rapidly delete logs, remove artifacts, or destroy other evidence of the attack before terminating, making forensic investigation significantly more difficult.
- **Accelerated lateral movement**: If the attacker realizes they've been detected, they may immediately attempt to spread to additional systems while they still have access, expanding the compromise before you can contain it.
- **Premature ransomware deployment**: Perhaps most dangerously, if an attacker with ransomware capabilities realizes they're about to lose access, they may deploy their ransomware immediately - before you're ready to respond - causing maximum damage.

**These consequences are why the passive-first approach is critical**. Every active analysis action you take must be weighed against these potential risks.

## When to Use Each Approach

Your decision to use passive or active analysis should be based on the specific situation and threat context:

| Situation                                             | Recommended Approach                              | Rationale                                                                                                                                           |
| ----------------------------------------------------- | ------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| Initial discovery, unsure of threat scope             | **Passive only**                                  | You don't know what you're dealing with yet. Preserve operational security while you gather intelligence about the threat.                          |
| Low-confidence IOC, likely false positive             | **Active acceptable**                             | If you're confident the alert is probably benign, the risk of active investigation is minimal since there's likely no attacker to alert.            |
| High-confidence C2, unknown threat actor capabilities | **Passive first, coordinate before going active** | You know there's a real threat, but you don't know if it has anti-analysis capabilities. Consult with IR team before risking detection.             |
| Contained system, network isolated                    | **Active acceptable**                             | If the compromised system has been isolated from the network, the attacker cannot receive alerts from the malware or respond to your investigation. |
| Active incident response in progress                  | **Active acceptable**                             | Once you're in full response mode and the attacker likely knows they're detected, stealth is no longer the priority - speed is.                     |

## Best Practice: Passive First

The guiding principle is simple: always exhaust passive sources before considering active analysis. Most investigations can be completed entirely through passive means, and this approach minimizes risk while maintaining investigative effectiveness.

### Primary Passive Sources

**Sysmon Logs**

Sysmon (System Monitor) is your most valuable passive data source. It captures detailed process and system activity and logs it to the Windows Event Log, where you can query it without any interaction with potentially malicious processes.

Key Event IDs:

- **Event ID 1 (Process Creation)**: Captures every process launch, including the full command line, parent process, user context, and file hashes
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

Despite the preference for passive analysis, there are legitimate situations where you need to go active. The decision should be based on a clear risk assessment:

**Consider active analysis when:**

1. **Passive sources are insufficient**: You've exhausted your logs and telemetry and still cannot make a confident determination about whether the activity is malicious.
2. **You've assessed the risk**: You understand the potential consequences and have determined that the intelligence value outweighs the risk of detection.
3. **You're prepared for consequences**: You have a response plan ready in case the investigation triggers a defensive reaction from the malware.
4. **IR team is coordinated**: You've consulted with your incident response team and everyone is prepared for what might happen when you go active.

The key is intentionality. Active analysis should never be your default first step. It should be a calculated decision made after careful consideration of the specific circumstances.

## Practical Workflow

In practice, your investigation workflow should follow this pattern:

**Phase 1: Passive Intelligence Gathering**

- Query Sysmon logs for process creation and network connection events
- Check Windows Event Logs + PS ScriptBlock Logs for relevant security events
- Examine prefetch files if needed for execution history

**Phase 2: Passive Analysis**

- Analyze all gathered data to understand process characteristics
- Research file hashes against threat intelligence databases (VirusTotal, MalwareBazaar)
- Map out parent-child process relationships from logs
- Identify network connections and destinations from passive network logs

**Phase 3: Decision Point**

- Can you make a determination based on passive data alone?
- If yes → Make your decision and proceed with appropriate action
- If no → Assess whether active analysis is warranted and consult with senior analysts or IR team

**Phase 4: Active Analysis (if necessary)**

- Ensure system isolation if possible
- Use tools like System Informer to inspect running processes
- Use PEStudio to analyze binary files
- Document all active analysis steps taken

This structured approach ensures that you minimize risk while maximizing the intelligence value of your investigation. The vast majority of your investigations will be resolved in Phases 1 and 2, never requiring active analysis at all.



