# Philosophy & Approach: Understanding the Foundation of Endpoint Analysis for Network Threat Hunting

## Network for Breadth, Endpoint for Depth

When you begin your journey as a network threat hunter, you'll quickly encounter a fundamental principle that will shape how you approach every investigation: **"Network for breadth, endpoint for depth."** This isn't just a catchy phrase - it's a strategic framework that defines the relationship between two complementary types of analysis.

### Understanding the Principle

Network analysis is your wide-angle lens. When you monitor network traffic flowing through your organization's infrastructure, you're observing thousands - sometimes millions - of connections simultaneously. You can see patterns emerging across entire departments, you can spot anomalies that affect multiple systems, and you can identify trends that might indicate a spreading threat. This is what we mean by "breadth." Network analysis gives you a panoramic view of your environment. It shows you _what_ is happening across your organization at scale.

But here's where network analysis reaches its natural limitation: while it can tell you that something suspicious _might_ be happening, it often cannot tell you that with a great degree of certainty, nor is it capable of telling you _why_ it's happening or _who_ is responsible. You might see an unusual connection to what appears to be a suspicious external IP address, but the network data alone won't reveal whether this is a legitimate business application calling home, a user browsing to a new website, or malware establishing a command-and-control channel.

This is where endpoint analysis becomes essential. When you transition from the network to the endpoint - to the actual computer or server where a suspicious connection originated - you gain depth. Endpoint analysis reveals the specific process or application that created the network connection, shows you how it was launched, tells you where it came from, and provides the detailed context needed to make an informed decision.

As a network threat hunter, our success is largely measured in terms of our accuracy - we don't want to be the "boy that cried wolf" and waste the time and resources of the Response Team on "maybe's". Transitioning from the network to the endpoint and performing a cursory "80/20" analysis is what will allow us to go from "maybe's" to "almost certainly's".

### The Integration: How Network and Endpoint Work Together

```
Network Analysis → Suspicious Connection Identified
         ↓
Endpoint Analysis → Process/Binary Investigation (Passive)
         ↓
Decision Point → Malware or Legitimate?
         ↓
Action → Alert IR Team or Clear
```

The real power emerges when you understand how these two types of analysis work together in a coordinated workflow. Your investigation will typically follow this pattern:

You begin with network analysis. Your monitoring tools flag something unusual. This is your starting point, your alert, your reason to investigate. The network has provided the breadth - it has identified that something potentially suspicious is occurring.

So you transition to endpoint analysis. You identify which computer or server generated the suspicious network traffic, and then you drill down to discover which specific process or application on that system created the connection. This is where you begin gathering the contextual information that will enable you to make an informed decision.

**Critically, this endpoint analysis is performed passively through log analysis.** You'll examine:

- The process that made the connection
- Where the executable file is located on the disk
- What the process is named
- How it was launched and what parameters were passed to it
- Which other process spawned it, creating the parent-child relationship that can reveal attack chains
- Whether this file's cryptographic hash matches any known malware samples in threat intelligence databases

All of this information is available through passive analysis of Sysmon logs, Windows Event Logs, and PowerShell ScriptBlock logs. You're not interacting with potentially malicious processes or opening suspicious binaries - you're examining telemetry that already exists.

This endpoint analysis provides the depth you need. It transforms a generic suspicious connection alert into a specific understanding: "Process X, located at path Y, launched by parent Z, with hash matching known malware, is making network connections to destination W." With this detailed context, you can now make an informed decision.

Finally, you reach your decision point. You synthesize the information from both the network and the endpoint. Does the process making this connection make sense? Based on this synthesis of network breadth and endpoint depth, you arrive at one of two conclusions: either this is malicious activity that requires immediate escalation to your incident response team, or this is legitimate activity that can be cleared and documented.


## The 80/20 Approach: Rapid Triage, Not Exhaustive Analysis

It's also worth touching on the type of endpoint analysis we'll be performing as a network threat hunter.

### What This Investigation Is Not

**You are not a malware reverse engineer.** You will not be loading malicious binaries into disassemblers like IDA Pro or Ghidra, stepping through assembly code, or analyzing the internal logic of how the malware functions. That level of deep technical analysis requires specialized skills, sophisticated tools, and most importantly, significant time - often hours or even days per sample.

You are not performing comprehensive static or dynamic analysis. You won't be running samples in sandboxes for extended periods, capturing complete behavioral profiles, or generating detailed malware analysis reports that document every file created, every registry key modified, or every API call made.

You are not conducting entropy analysis to identify packed or encrypted sections of executables, you're not performing detailed memory forensics to understand runtime behavior, and you're not attempting to identify the specific malware family or variant you're dealing with.

All of these activities have their place in a mature security program, but they're not your job in this particular role. They're tasks for specialized malware analysts, for forensic investigators, or for incident response teams during the deep-dive phase of an investigation.

### What This Investigation Is

So if you're not doing all of that, what are you actually doing? You're performing **rapid triage through passive analysis**. And rapid triage is absolutely critical to an effective security program.

Think of yourself as the emergency room triage nurse. When patients arrive at an emergency department, the triage nurse doesn't perform surgery, doesn't run extensive lab tests, doesn't make final diagnoses. Instead, they make rapid assessments: Is this person's condition life-threatening and requiring immediate attention? Is it serious but stable enough to wait? Or is it a minor issue that can be handled by a general practitioner?

This is your role as a network threat hunter performing endpoint analysis. You're making rapid assessments to answer one fundamental question: **"Do we alert the incident response team or not?"**

### The 80/20 Principle in Practice

The framework we use to achieve this rapid triage is based on the Pareto Principle, commonly known as the 80/20 rule. The principle states that roughly 80% of effects come from 20% of causes. In the context of threat hunting and endpoint analysis, we apply this principle to mean: you can achieve (at least) 80% confidence in your decision by examining 20% of the available endpoint indicators through passive analysis.

This is a pragmatic, efficiency-focused approach born from real-world necessity. In a typical enterprise environment, you might generate dozens or even hundreds of alerts per day. If you spent hours performing exhaustive analysis on each one, you'd never keep up with the queue. Worse, truly critical incidents would be delayed while you conducted unnecessarily deep analysis on less important alerts.

Instead, the 80/20 approach acknowledges that there's a subset of indicators - specific characteristics of processes and binaries available through passive log analysis - that, in combination with the preceding network indicators, are highly diagnostic. These indicators can quickly differentiate between malicious and legitimate activity in the vast majority of cases.

### Example

Let me give you a concrete example to illustrate this. Suppose your network monitoring alerts on a connection from a workstation to an external IP address. Using the 80/20 approach with passive analysis, you would:

First, quickly identify which process made the connection by querying Sysmon Event ID 3 (Network Connection) logs. Let's say it's a process called `svchost.exe` - a legitimate Windows system process.

Second, you correlate to Sysmon Event ID 1 (Process Creation) and check where this process is running from. You discover it's running from `C:\Users\JohnDoe\AppData\Local\Temp\svchost.exe`

At this point, you already have your answer through purely passive log analysis. Legitimate `svchost.exe` processes run exclusively from `C:\Windows\System32`. The fact that this one is running from a user's temporary folder is a massive red flag. You could stop your investigation right here with very high confidence that this is malicious - it's almost certainly malware masquerading as a Windows system process.

In this example, you examined just two indicators (process name and location) from passive log sources, spent perhaps 60 seconds on the investigation, and reached a high-confidence determination of malicious activity. You didn't need to interact with the binary, you didn't need to run it in a sandbox, you didn't need to identify which malware family it belongs to. You made a rapid, accurate triage decision using just 20% of the potentially available information through passive means, and you can now escalate this to your incident response team with confidence.






