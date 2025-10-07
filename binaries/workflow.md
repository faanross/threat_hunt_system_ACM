# Investigation Workflow: From Detection to Decision

Every investigation follows a structured workflow designed to move you efficiently from initial network detection to a confident decision. This workflow incorporates the passive-first principle, the 80/20 analysis approach, and the seven critical indicators into a practical sequence of steps.

Understanding this workflow is essential because it prevents you from jumping directly to time-consuming deep analysis when a quick triage might resolve the issue. It also ensures you don't skip critical steps that might reveal malicious activity early in the process.

## Phase 1: Discovery (Network Analysis)

Your investigation begins when your network monitoring infrastructure identifies something that warrants attention. This is the "breadth" component - network analysis casting a wide net to detect potentially suspicious activity.

```
Network Monitoring Tool
         ↓
Suspicious Connection Detected
         ↓
Extract Connection Details:
  - Source IP/Port
  - Destination IP/Port
  - Destination domain + X509 (if applicable)
  - Protocol
  - Connection Interval Histogram (AC-Hunter)
  - Data Transfer Histogram (AC-Hunter) 
  - etc.
         ↓
Identify Associated Process
```

### What Triggers an Investigation

Network monitoring tools generate alerts based on various detection logic, since that is discussed elsewhere and not the focus of this section, let's move right ahead.


### Identifying the Associated Process

The network alert tells you that traffic occurred, but it doesn't tell you which application or process created that traffic. This is the critical link between network breadth and endpoint depth.

We can typically correlate the connection with a process ID (PID) or process name using Sysmon Event ID 3 (Network Connection).

Once you've identified the process, you transition from Phase 1 to Phase 2 - from network analysis to endpoint analysis.



## Phase 2: Initial Triage (Passive)

Phase 2 is where you begin examining the endpoint, but you're still operating entirely through passive means. You're querying logs and telemetry, not interacting with any potentially malicious processes.

```
Query Logs
         ↓
Extract Process Details:
  - Process name
  - PID
  - Binary path
  - Command line
  - Parent process
  - User context
  - Timestamp
         ↓
Quick Assessment:
  - Known good process?
  - Expected location?
  - Legitimate parent?
    ↓
    ALL YES → Evaluate Network Behavior Context
    ↓
    ├─ Network behavior matches expected? → Document & Clear
    │
    ├─ Network behavior highly suspicious? → Continue to Phase 3
    │    (Process injection, LOLBin abuse, or DLL hijacking possible)
    ↓
    ANY NO/UNSURE → Continue to Phase 3
```



### Querying Your Data Sources

By correlating Sysmon Event ID 3 (Network Connection) to its corresponding Sysmon Event ID 1 (Process Creation) using the GUID we can determine exactly what process was responsible for initializing the connection of interest.

### Critical Process Details

From this log, extract seven key pieces of information:

**Process name**: What is the process called? This is your first clue about what you're dealing with.

**PID (Process ID)**: The unique identifier for this specific process instance. Useful for correlating events across different log sources.

**Binary path**: The full file system path to the executable. This is one of your seven critical indicators - where the binary is located matters tremendously.

**Command line**: The complete command line used to launch the process, including all arguments and parameters. This often reveals malicious intent even when the process itself appears legitimate.

**Parent process**: Which process spawned this one? Parent-child relationships are highly diagnostic for detecting attack techniques.

**User context**: Which user account was the process running under? This helps determine if the activity aligns with expected user behavior.

**Timestamp**: When the process was created. Useful for timeline analysis and correlating with other events.

### Quick Initial Assessment

With this information in hand, perform a rapid initial assessment by asking three questions:

**Is this a known good process?** If you immediately recognize this as legitimate corporate software or a standard Windows process running in its expected configuration, you might be able to clear the alert right here. For example, if the process is `chrome.exe` running from `C:\Program Files\Google\Chrome\Application\` with `explorer.exe` as the parent, connecting to `google.com`, this is almost certainly legitimate browsing activity.

**Is the location expected?** Even if you don't immediately recognize the process, check whether it's running from an expected location. Processes in `C:\Windows\System32\` or `C:\Program Files\` from recognized vendors are less suspicious than processes in temporary folders or user directories.

**Is the parent process legitimate?** Does the parent-child relationship make sense? Standard applications launched by users should have `explorer.exe` as their parent. Windows services should descend from `services.exe`. Unusual parent-child pairs (like Microsoft Word spawning PowerShell) are immediate red flags.

### Decision Point: When "Legitimate" Doesn't Mean "Safe"

This is a critical juncture in your investigation where analyst judgment becomes essential. Even when all the basic process characteristics appear legitimate, you cannot automatically clear the alert if the network behavior that triggered your investigation remains unexplained.

**The Three Quick Assessment Questions**:

1. Is this a known good process (recognized vendor, common application)?
2. Is it running from its expected location?
3. Does it have a legitimate parent process?

**If all three answers are YES**, you might initially think the alert is a false positive. However, you must now consider the network evidence that brought you here in the first place.

**Evaluating Network Behavior in Context**:

Ask yourself: Does the network behavior that triggered this investigation make sense for this process?

**Scenario 1: Network Behavior Matches Expectations**

```
Process: chrome.exe
Location: C:\Program Files\Google\Chrome\Application\chrome.exe
Parent: explorer.exe
Signature: Valid (Google LLC)
Network: Multiple HTTPS connections to various domains (google.com, youtube.com, etc.)

Assessment: Everything aligns - legitimate browser making expected connections
Decision: Document and clear
```

**Scenario 2: Network Behavior Highly Suspicious Despite Legitimate Process**

```
Process: chrome.exe
Location: C:\Program Files\Google\Chrome\Application\chrome.exe
Parent: explorer.exe  
Signature: Valid (Google LLC)
Network: Regular HTTPS beaconing to 185.220.101.50 every 60 seconds

Assessment: Process appears completely legitimate BUT:
  - Beaconing pattern is not normal browser behavior
  - Single destination despite supposedly "active browsing"
  - Regular 60-second intervals suggest automated C2 communication
  
Possible Explanations:
  1. Process injection - malicious code injected into legitimate Chrome
  2. Malicious browser extension
  3. Compromised Chrome instance (though hash matches legitimate)

Decision: Continue to Phase 3 for deeper investigation
Reasoning: Network behavior is so anomalous it overrides surface legitimacy
```

**Scenario 4: Legitimate RMM/DFIR Tools - Authorization Question**

```
Process: ScreenConnect.ClientService.exe (or AnyDesk, TeamViewer, Velociraptor, etc.)
Location: C:\Program Files (x86)\ScreenConnect Client\
Parent: services.exe (running as Windows service)
Signature: Valid (ScreenConnect vendor signature)
Network: HTTPS connections to relay.screenconnect.com

Assessment: Process is completely legitimate remote management tool BUT:
  - Does our organization use/authorize ScreenConnect?
  - If yes, is this instance connecting to our authorized ScreenConnect server?
  - If no, this is likely attacker-installed instance

Required Investigation:
  1. Check if tool is in approved software list
  2. If approved, verify destination matches company infrastructure
  3. Check installation date/timeline - recently installed = suspicious
  4. Verify with IT/Security team if this installation was authorized
  5. Check user context - should this user/system have this tool?

Decision Path:
  - Tool NOT approved → HIGH SUSPICION
  - Tool approved BUT wrong destination → HIGH SUSPICION (unauthorized instance)
  - Tool approved AND correct destination → Verify with IT team, then likely clear
```

**Common RMM/DFIR Tools That Require Authorization Verification**:

Remote Management:

- ScreenConnect (ConnectWise Control)
- AnyDesk
- TeamViewer
- LogMeIn
- GoToMyPC
- Remote Desktop tools (third-party)

DFIR/Security Tools:

- Velociraptor
- GRR Rapid Response
- OSQuery


**The Critical Questions**:

1. **Is this tool authorized in our environment?**

    - Check with IT/Security team
    - Consult approved software list
    - Verify against standard deployments
2. **Even if authorized, is THIS instance legitimate?**

    - Connecting to company infrastructure or external?
    - Installation date matches known deployment?
    - Present on expected systems or unexpected?
    - Proper service account or user account?
3. **Does the context make sense?**

    - IT admin workstation with RMM tool = expected
    - Random user workstation with DFIR tool = unexpected
    - Server with unauthorized remote access = very suspicious

This scenario demonstrates that "legitimate and signed" doesn't always mean "authorized and safe." Organizational context is essential for tools that have both legitimate and malicious use cases.
