# Signatures vs. Behaviours: The Hunter's Mindset

## The Two Ways of Seeing

This distinction between signatures and behaviours represents perhaps the most important conceptual shift for threat hunters. New hunters often approach hunting as "searching for bad things" - specific malware, known malicious indicators, documented threats. This signature-focused mindset produces limited value because adversaries constantly change these specific artifacts. Mature hunters think differently - they hunt for suspicious behaviours, adversary methodologies, and attack patterns that persist even when specific tools and artifacts change.

This chapter explores the signature-versus-behaviour distinction in depth, explains why behavioural thinking is fundamental to effective hunting, and describes how to develop the "hunter's mindset" that focuses on adversary methodology rather than specific artifacts. This shift is challenging - it requires changing how you think about threats, detection, and investigation - but it's perhaps the single most important leap toward becoming an effective threat hunter.

## Defining Signatures and behaviours

Before diving deeper, let's clearly define these terms as they apply to threat hunting:

### Signatures (Indicator-Based Detection)

Signatures are specific, concrete artifacts that identify known threats with high precision:

**Technical Signatures**: File hashes, IP addresses, domain names, URLs, email addresses, registry keys, file paths, mutex names, service names, and other specific atomic indicators.

**Pattern Signatures**: Specific byte patterns in files, particular strings in network traffic, exact command-line arguments, specific API call sequences, or other precise patterns that match known threats.

**Behavioural Signatures**: Even "behavioural" detections can become signatures when they're overly specific - for instance, "detect exactly this PowerShell command with these specific arguments executed from this specific path."

The key characteristic of signatures is specificity and exactness. Signatures match specific known artifacts or patterns, providing high-confidence detection but limited flexibility.

### Behaviours (TTP-Based Detection)

Behaviours are patterns of activity that indicate adversary techniques, tactics, or methodologies:

**Technique Indicators**: Evidence that specific MITRE ATT&CK techniques are being used - lateral movement via WMI, credential dumping from LSASS, persistence via scheduled tasks - regardless of specific tools or exact implementation.

**Procedural Patterns**: Sequences of activities that indicate adversary methodologies - discovery commands followed by credential access attempts followed by lateral movement, even when specific commands vary.

**Anomalous Patterns**: Deviations from established baselines that suggest adversary activity - unusual authentication patterns, abnormal process relationships, suspicious network communication patterns.

**Contextual behaviours**: Activities that are suspicious in context even if individually benign - administrative tools used by non-administrators, system access from unusual locations, data movement in unexpected directions.

The key characteristic of behaviours is abstraction and flexibility. behavioural detection recognizes classes of activity that indicate threats, providing resilience against adversary variation and evolution.

## The Fundamental Differences

Understanding the deep differences between these approaches clarifies why behavioural hunting is so valuable:

```
Signatures vs. Behaviours: Core Differences
═══════════════════════════════════════════════════════════

SIGNATURES                    BEHAVIOURS
══════════                    ═════════

SPECIFICITY
• Exact matches               • Pattern recognition
• High precision              • Contextual judgment
• Binary (match/no match)     • Probabilistic (likely/unlikely)

COVERAGE
• Known threats only          • Known and unknown threats
• Specific variants           • Entire technique classes
• Point-in-time               • Persistent over time

RESILIENCE
• Brittle - easily evaded     • Resilient - harder to evade
• Adversary changes trivially • Adversary must change methodology
• Short lifespan              • Long lifespan

DEVELOPMENT
• Easy to create              • Challenging to develop
• Simple logic                • Complex analytics
• Quick implementation        • Requires tuning

FALSE POSITIVES
• Very low                    • Can be higher
• Clear matches               • Requires investigation
• High confidence             • Contextual assessment

INVESTIGATION
• Detection is answer         • Detection is question
• Known threat identified     • Requires investigation
• Minimal context needed      • Environmental context critical

AUTOMATION
• Easily automated            • Harder to automate
• Machine-efficient           • Often requires human judgment
• Scales naturally            • Scaling challenges

VALUE OVER TIME
• Depreciates quickly         • Appreciates or maintains
• Requires constant updates   • Improves with refinement
• Reactive                    • Proactive
```

These differences aren't just technical - they reflect fundamentally different philosophies about detection and hunting.

## Why Signatures Fail at Hunting

If signatures provide precise, high-confidence detection, why aren't they sufficient for threat hunting? Several fundamental limitations make signature-based hunting ineffective:

### Limitation 1: The Adversary Adaptation Cycle

Adversaries continuously adapt to evade signature-based detection. The cycle works like this:

1. Defenders deploy signature-based detection for known threat X
2. Adversaries learn signature is detecting threat X
3. Adversaries trivially modify X to create variant X' that evades signature
4. Defenders discover variant X' and create new signature
5. Adversaries create variant X'', repeating the cycle indefinitely

This is an arms race defenders cannot win. Adversaries need to succeed only once with each variant before detection catches them, while defenders must create new signatures for every variant. The economics favor adversaries - signature evasion is cheaper than signature development.

Mature adversaries specifically design tools and procedures to evade signature detection. Polymorphic malware automatically generates variants with different signatures. Sophisticated adversaries test their tools against common security products before deployment, iterating until signatures don't match.

### Limitation 2: The Zero-Day Problem

Signatures by definition detect only known threats. Zero-day exploits, custom malware, and novel attack techniques have no signatures because they're unknown to defenders. Yet these are exactly the threats most dangerous and most worth detecting.

Signature-based hunting fundamentally cannot address zero-day threats. You can't hunt for signatures that don't exist yet. This means signature-focused hunting misses precisely the threats that automated signature-based detection also misses - providing no incremental value.

### Limitation 3: The Coverage Gap

Even if you had perfect signatures for all known threats, coverage gaps would remain:

**Living-off-the-Land**: Adversaries increasingly use legitimate system tools (PowerShell, WMI, PsExec, built-in utilities) that have no malicious signatures because they're legitimate software.

**Fileless Attacks**: Malware executing entirely in memory leaves no files to hash, no signatures to match.

**Encrypted Communications**: Even if you know malicious domains or IPs, encrypted communications hide the specific content that signatures might match.

**Insider Threats**: Malicious insiders use legitimate credentials and access, generating no signature matches because they're using authorized accounts and tools.

Signature-based hunting offers nothing in these scenarios because the adversary artifacts you'd signature-match simply don't exist or aren't accessible.

### Limitation 4: The Investigation Value

Perhaps most importantly, signature matches provide minimal investigative value. When a signature matches, you learn: "This specific artifact is present." You don't learn:

- How did the adversary gain access?
- What have they done since initial compromise?
- What other systems might be compromised?
- What is their ultimate objective?
- What techniques and procedures are they using?

Signature detection is the beginning of investigation, not the end. But signature-focused hunting invests human effort in activity that automated systems handle more efficiently, leaving little time for actual investigation that provides context and understanding.

## Why Behavioural Hunting Succeeds

Behavioural hunting addresses each limitation of signature-based approaches:

### Advantage 1: Resilience to Adversary Adaptation

Behavioural detection targets adversary methodologies that are difficult or costly to change. When you detect "credential dumping from LSASS process" behaviourally (unusual process access to LSASS combined with suspicious read operations), adversaries can't simply change a hash or IP address to evade detection. They must either:

- **Abandon the technique entirely**: Find different ways to steal credentials, potentially less effective or more detectable
- **Significantly modify their approach**: Develop new tools or procedures, requiring development time, testing, and operational risk
- **Accept detection risk**: Continue using the technique despite knowing it might be detected

Each option causes adversaries real pain - time, resources, operational constraints. Unlike signature evasion (trivial), behavioural evasion is costly.

### Advantage 2: Detection of Unknown Threats

Behavioural hunting detects adversary techniques regardless of whether specific malware or indicators are known. When you hunt for "unusual authentication patterns consistent with pass-the-hash attacks," you catch:

- Known malware using pass-the-hash
- Unknown malware using pass-the-hash
- Custom tools using pass-the-hash
- Legitimate tools misused for pass-the-hash
- Novel implementations of pass-the-hash

The behavioural pattern - unusual authentication with particular characteristics - remains detectable regardless of specific implementation. This enables detection of zero-days and novel attacks that signature-based approaches miss entirely.

### Advantage 3: Comprehensive Coverage

Behavioural hunting addresses gaps that signatures can't fill:

**Living-off-the-Land Detection**: Behavioural analysis distinguishes malicious from legitimate use of system tools through context - who's using the tool, when, for what purpose, in what sequence with other activities.

**Fileless Attack Detection**: Behavioural analysis detects adversary activities in memory through suspicious process behaviours, unusual memory allocations, or anomalous inter-process interactions.

**Encrypted Traffic Analysis**: Behavioural analysis examines traffic patterns, connection characteristics, and timing rather than content - detecting command and control through beaconing patterns even when communications are encrypted.

**Insider Threat Detection**: Behavioural analysis detects unusual patterns in otherwise-authorized access - unusual times, locations, resources accessed, or data volumes that suggest malicious intent.

### Advantage 4: Rich Investigative Context

Behavioural hunting provides understanding, not just detection. When you identify suspicious behaviours, you gain context:

- **Techniques being used**: Understanding what adversaries are doing
- **Attack progression**: Recognizing where adversaries are in their campaign
- **Adversary sophistication**: Assessing skill level based on techniques and procedures
- **Operational patterns**: Identifying adversary tradecraft and methodologies
- **Scope assessment**: Understanding breadth of compromise based on technique coverage

This context enables effective response, comprehensive scoping, and strategic security improvements. behavioural hunting isn't just about detection - it's about understanding adversary operations thoroughly enough to respond comprehensively.


## Thinking Like an Adversary

The core of behavioural hunting is thinking like an adversary. Rather than asking "What bad artifacts might be present?" ask "How would I accomplish malicious objectives in this environment?"

### The Adversary Perspective

Skilled adversaries think operationally:

**Objective-Oriented**: Adversaries don't randomly use tools or techniques. Every action serves an objective - gaining access, establishing persistence, moving laterally, stealing data, causing disruption. Understanding objectives helps predict techniques.

**Constraint-Aware**: Adversaries operate within constraints - their tools' capabilities, their skill levels, their operational security requirements, the target environment's characteristics. Understanding constraints helps narrow possible techniques.

**Risk-Conscious**: Sophisticated adversaries balance effectiveness against detection risk. They choose techniques that accomplish objectives while minimizing exposure. Understanding this calculus helps predict adversary choices.

**Adaptive**: When one technique fails or is detected, adversaries adapt - trying alternative techniques, modifying procedures, or changing operational approaches. Understanding adaptation patterns helps predict next moves.

### Applying Adversary Thinking to Hunting

Adopting this adversary perspective transforms hunting:

**Instead of**: "Let me search for known malicious PowerShell commands" 
**Think**: "If I wanted to execute code without dropping files, I'd use PowerShell. What would that look like in my environment? What would distinguish malicious from legitimate PowerShell use?"

**Instead of**: "Let me hunt for connections to known-malicious IPs" 
**Think**: "If I needed command and control communications, what would be most effective in this environment? How would I disguise those communications to look legitimate?"

**Instead of**: "Let me search for known persistence mechanisms" 
**Think**: "If I compromised a system, how would I maintain access for weeks or months? What persistence mechanisms are available? How could I make them look legitimate?"

This shift from "what artifacts?" to "how would I?" fundamentally changes investigation approaches and success rates.

## Developing Behavioural Hunting Hypotheses

How do you generate hypotheses focused on behaviours rather than signatures?

### Start with Techniques, Not Artifacts

**Poor Hypothesis**: "Search for hash X in our environment" 
**Better**: "Search for evidence of credential dumping techniques"

**Poor Hypothesis**: "Hunt for connections to malicious IP addresses from last week's intel report" 
**Better**: "Hunt for beaconing network behaviour consistent with command and control"

**Poor Hypothesis**: "Look for known malware families mentioned in recent reports" 
**Better**: "Look for lateral movement using remote execution tools"

The pattern: move from specific artifacts to general techniques and behaviours.

### Use MITRE ATT&CK as Framework

MITRE ATT&CK provides an excellent framework for behavioural hypothesis generation. Rather than inventing hypotheses from scratch:

1. Review ATT&CK techniques relevant to your environment
2. Select techniques based on recent threat intelligence, organizational concerns, or systematic coverage planning
3. Generate hypotheses about how those techniques would manifest in your specific environment
4. Design investigations to detect those manifestations

For example, taking ATT&CK technique T1003 (OS Credential Dumping):

**Hypothesis**: "Adversaries may be dumping credentials from LSASS process memory. In our Windows environment, this would involve unusual processes accessing LSASS, loading suspicious DLLs into credential processes, or execution of known credential dumping tools like Mimikatz. We should hunt for processes accessing LSASS from unexpected sources, unusual DLL loads, and credential tool execution patterns."

### Consider Environmental Context

Effective behavioural hypotheses incorporate environmental context:

**Generic Hypothesis**: "Hunt for lateral movement" 
**Contextualized**: "In our environment, domain administrators routinely use Remote Desktop to manage servers. Non-admin accounts should never initiate remote sessions to multiple systems. Hunt for non-admin accounts establishing remote sessions, or admin accounts accessing systems outside their normal management scope."

Environmental context transforms abstract techniques into specific, actionable investigations that distinguish malicious from legitimate behaviour.

### Think About Adversary Tradecraft

Different adversary groups have characteristic tradecraft - preferred techniques, typical tool combinations, operational patterns. When threat intelligence describes adversary behaviour:

**Don't Just Extract IOCs**: "Here's a malicious IP address to block" 
**Extract Behavioural Patterns**: "This adversary typically uses spear-phishing for initial access, establishes persistence via scheduled tasks, uses WMI for lateral movement, and stages data in hidden directories before exfiltration. Let's hunt for these patterns."

Even without knowing specific artifacts, understanding tradecraft enables hunting for characteristic behaviour patterns.


## Practical behavioural Hunting Techniques

How does behavioural hunting work practically? Several approaches prove effective:

### Baseline Deviation Detection

Establish baselines for normal behaviour, then hunt for deviations:

**Authentication Patterns**: Baseline typical user authentication times, locations, frequency, and target systems. Hunt for deviations - authentications at unusual times, from unusual locations, to unusual systems, or at unusual frequency.

**Process Relationships**: Baseline normal parent-child process relationships. Hunt for unusual relationships - cmd.exe spawned by Excel, PowerShell spawned by browser, unusual service creation patterns.

**Network Communications**: Baseline which systems normally communicate with which destinations. Hunt for unusual communications - systems communicating that normally don't, connections to unusual external destinations, unusual protocol or port usage.

**Data Movement**: Baseline normal data flows within environment. Hunt for unusual movements - data flowing to unexpected locations, large transfers to external destinations, unusual file access patterns.

The power of baseline deviation detection is its environment specificity. What's suspicious in your environment depends on your normal patterns, not generic signatures.


### Sequence and Correlation Analysis

Hunt for sequences of behaviours that individually seem benign but collectively indicate attack progression:

**Discovery → Credential Access → Lateral Movement**: This sequence is characteristic of adversary progression. Individual activities might be legitimate (admin running discovery commands, legitimate credential operations, normal remote access), but the sequence from unexpected user accounts suggests compromise.

**Initial Access → Persistence → Defense Evasion**: This sequence suggests adversary entrenchment. Hunt for scenarios where initial access events are quickly followed by persistence mechanism creation and suspicious process behaviours.

**Collection → Staging → Exfiltration**: This sequence indicates data theft. Hunt for unusual file gathering activities followed by data movement to staging locations and subsequent external transfers.

Behavioural hunting excels at correlation across time and systems - connecting individual events into narratives that reveal adversary campaigns.

### Anomaly-Based Hunting

Hunt for statistically unusual behaviours even without knowing specific adversary techniques:

**Volume Anomalies**: Unusually large numbers of failed authentications, unusual volumes of DNS queries, unusual network traffic volumes.

**Timing Anomalies**: Activities at unusual times - system access during off-hours, scheduled tasks at odd times, data transfers at 3 AM.

**Pattern Anomalies**: Behaviours that break from typical patterns - alphabetically sequential file access (suggest automated scripts), highly regular timing patterns (suggest beaconing), unusual process execution patterns.

Statistical anomaly detection requires establishing normal ranges, then hunting for outliers. Not all anomalies are malicious, but adversaries often create statistical anomalies that behavioural hunting detects.

### Tool behaviour Profiling

Rather than detecting tools by signatures, detect them by characteristic behaviours:

**Cobalt Strike Beacons**: Characteristic named pipes, specific service creation patterns, particular injection techniques, recognizable memory artifacts.

**Mimikatz**: Characteristic LSASS access patterns, specific DLL loading, particular authentication artifacts left behind.

**PowerShell Empire**: Characteristic staging patterns, specific PowerShell command combinations, recognizable network traffic patterns.

Even when adversaries modify tools to evade signatures, behavioural patterns often persist because they reflect fundamental tool functionality that's difficult to change without breaking capability.

## The Investigation Mindset

Behavioural hunting requires different investigation approaches than signature-based detection:


### Start with Questions, Not Answers

**Signature Mindset**: "Does this hash exist in my environment?" (Yes/No answer) 
**Behavioural Mindset**: "Are there signs that adversaries are dumping credentials?" (Requires investigation)

Behavioural hunting starts with questions that require investigation, analysis, and judgment rather than simple matching.

### Embrace Ambiguity

**Signature Mindset**: Clear matching - artifact is present or not 
**Behavioural Mindset**: Probabilistic assessment - behaviour is suspicious to some degree, requires contextual evaluation

Behavioural hunters must be comfortable with ambiguity and graduated suspicion rather than binary determinations.

### Follow Where Evidence Leads

**Signature Mindset**: Check for specific artifact, move to next artifact if not found 
**Behavioural Mindset**: Investigate initial suspicion, let findings suggest next investigation paths, pivot based on discoveries

Behavioural investigation is exploratory and iterative rather than checklist-based.

### Consider Multiple Hypotheses

**Signature Mindset**: Looking for specific known thing 
**Behavioural Mindset**: Could be adversary technique A, or technique B, or legitimate activity that resembles technique C - must investigate to determine

Good behavioural hunters maintain multiple working hypotheses simultaneously, letting evidence gradually strengthen or eliminate each possibility.

## Building behavioural Detection Rules

One valuable outcome of behavioural hunting is creating detection rules that catch similar behaviours automatically in the future:

### From Hunt to Detection Rule

When behavioural hunting discovers threats:

1. **Document the behaviour**: Exactly what behavioural patterns indicated the threat?
2. **Identify distinguishing characteristics**: What made this behaviour malicious vs. legitimate?
3. **Abstract the pattern**: Can you generalize from this specific instance to a broader pattern?
4. **Design detection logic**: How could automated systems recognize similar behaviours?
5. **Test and tune**: Validate detection catches threats while minimizing false positives
6. **Deploy and monitor**: Implement detection, monitor effectiveness, refine over time

This cycle transforms hunting discoveries into persistent detection capabilities - exactly what the Pyramid of Pain recommends.

### Balancing Detection and Evasion

When creating behavioural detections, balance detection coverage against evasion resistance:

**Too Specific**: Detection matches only exact behaviour observed, easily evaded by slight variations 
**Too General**: Detection triggers on many benign activities, overwhelming with false positives

Good behavioural detection is specific enough to minimize false positives while general enough to catch variations and evolutions of the technique.

### Living Documentation

Behavioural detections require ongoing refinement:

- Adversaries evolve techniques, requiring detection updates
- Environments change, requiring baseline adjustments
- False positive investigation reveals edge cases requiring rule tuning
- New tools and technologies create new legitimate behaviours that resemble threats

Treat behavioural detection rules as living documentation requiring continuous maintenance rather than static signatures deployed once and forgotten.

## Common Challenges and Solutions

Behavioural hunting faces several common challenges:

### Challenge 1: High False Positive Rates

**The Problem**: Behavioural detection often triggers on legitimate activities that resemble adversary techniques.

**Solutions**:

- Develop comprehensive baselines so you understand legitimate behaviours
- Incorporate contextual factors (user, system, time, business process) into detection
- Triage efficiently - distinguish false positives quickly without deep investigation
- Refine detection rules iteratively based on false positive patterns
- Accept some false positives as cost of detecting unknown threats

### Challenge 2: Environmental Complexity

**The Problem**: Complex environments with diverse systems, users, and business processes make behavioural analysis difficult.

**Solutions**:

- Start hunting in well-understood areas before expanding to complex areas
- Build documentation of legitimate behaviours alongside hunting activities
- Collaborate with system owners and business units to understand legitimate use cases
- Segment hunting by environment zones (production vs. development, critical vs. general)
- Accept that perfect behavioural understanding takes time to develop

### Challenge 3: Adversary behavioural Evasion

**The Problem**: Sophisticated adversaries intentionally blend malicious behaviours with legitimate activities to evade behavioural detection.

**Solutions**:

- Look for subtle anomalies in otherwise-normal behaviours
- Correlate across multiple behavioural dimensions (timing + volume + destination)
- Hunt for "too perfect" behaviours that actually stand out through their normalcy
- Leverage deception technologies that detect adversary reconnaissance behaviours
- Accept that the cat-and-mouse game continues at the behavioural level

### Challenge 4: Scaling behavioural Hunting

**The Problem**: Behavioural hunting is time-intensive and difficult to scale across large environments.

**Solutions**:

- Prioritize hunting on critical assets and high-risk user populations
- Automate behavioural pattern recognition where possible (ML/analytics)
- Create playbooks that make behavioural hunting more efficient
- Focus on high-value TTPs rather than comprehensive coverage
- Build feedback loops where hunting discoveries become automated detection

## The Cultural Shift

Perhaps the hardest part of moving from signatures to behaviours isn't technical - it's cultural:

### From Certainty to Probability

Signature-based security provides comfortable certainty - clear matches with definitive answers. behavioural hunting requires accepting probability and ambiguity. This shift makes some stakeholders uncomfortable: "You're saying you're not certain if this is malicious?"

**Managing Expectations**: Help stakeholders understand that behavioural hunting trades certainty for resilience. You detect more threats (including unknown threats) but with more investigation required.

### From Point Solutions to Continuous Process

Signatures provide point solutions - deploy signature, detect that specific threat. behavioural hunting is continuous process requiring ongoing effort, refinement, and investment.

**Managing Expectations**: Position behavioural hunting as operational capability requiring sustained investment, not one-time deployment.

### From Tool-Centric to Analyst-Centric

Signatures enable tool-centric security - deploy tool, let it detect. behavioural hunting is analyst-centric - human judgment, creativity, and expertise are essential.

**Managing Expectations**: Emphasize that behavioural hunting requires skilled analysts and can't be fully automated. This isn't weakness - it's acknowledging reality that detecting sophisticated threats requires human intelligence.

### Measuring Success Differently

Signature-based detection has easy metrics - detection rates, false positive rates, coverage counts. behavioural hunting requires different metrics - detection of unknown threats, improvement in detection capabilities, reduction in dwell time for novel attacks.

**Managing Expectations**: Develop metrics that capture behavioural hunting's value: techniques covered, detection rules created, novel threats discovered, security improvements enabled.

## The Signature-behaviour Balance

This chapter has strongly advocated for behavioural hunting, but the message isn't "abandon signatures entirely." The optimal approach balances both:

**Signatures Handle Known Threats**: Automated signature-based detection efficiently handles known threats at scale. This remains valuable and shouldn't be abandoned.

**Behaviours Handle Unknown Threats**: Human behavioural hunting discovers unknown threats and novel techniques that signatures miss. This provides unique value.

**Hunting Improves Signatures**: behavioural hunting discoveries feed back into signature creation, expanding automated detection coverage. Each hunt that finds new techniques should produce new signatures or behavioural detection rules.

**Signatures Free Hunting**: Effective automated signature detection frees human hunters from searching for known indicators, allowing focus on behavioural hunting where humans add unique value.

The relationship is complementary and symbiotic. Don't view this as "signatures vs. behaviours" but rather "signatures for automation, behaviours for hunting, both improving security together."


