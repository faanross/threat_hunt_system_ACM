# The Pyramid of Pain and Hunting Focus Areas

## The Question of Where to Hunt

Imagine you're a threat hunter facing an environment with millions of log entries, thousands of systems, and hundreds of potential threat scenarios. Where do you start? What should you hunt for? These aren't just practical questions - they're strategic ones that determine whether your hunting efforts produce meaningful value or consume resources without impact.

Many new threat hunters instinctively focus on what seems concrete and specific: hunting for known malicious IP addresses, searching for file hashes from malware reports, or looking for domain names associated with command and control infrastructure. These hunts feel productive because they're specific and measurable - "We searched for 500 malicious IPs and found three matches!" But this approach fundamentally misunderstands what makes threat hunting valuable and what adversaries find difficult to evade.

This is where David Bianco's Pyramid of Pain becomes essential. First published in 2013, this simple but profound model revolutionized how defenders think about threat indicators and detection strategy. It provides a framework for understanding which adversary artifacts are trivial to change (and therefore low-value to hunt for) versus which are deeply embedded in adversary operations (and therefore high-value hunting targets).

Understanding the Pyramid of Pain transforms how you approach threat hunting. It shifts focus from hunting for specific artifacts that adversaries easily change to hunting for behavioural patterns and techniques that adversaries can't easily abandon. This chapter explores the Pyramid of Pain model, its implications for threat hunting strategy, and why focusing on TTPs (Tactics, Techniques, and Procedures) rather than atomic indicators is fundamental to effective hunting.

## The Pyramid of Pain: Structure and Concept

The Pyramid of Pain visualizes different types of adversary indicators arranged by how much "pain" (difficulty, cost, operational impact) changing them causes adversaries:

![pyramid_of_pain](./img/pyramid.png)



The pyramid's power lies in its simplicity: adversaries easily change things at the bottom (hash values, IP addresses) but struggle to change things at the top (tactics, techniques, procedures). Therefore, defensive focus should concentrate toward the pyramid's top - detecting and hunting for TTPs rather than atomic indicators.

## Level 1: Hash Values (Trivial)

At the pyramid's base are hash values - cryptographic signatures of files, typically malware samples. Security tools use hashes to identify known malicious files through exact matching.

### Why Hashes Are Trivial

Changing a file's hash requires only trivial modifications. An adversary can:

- Recompile malware source code with minor changes
- Append random bytes to executable files
- Modify resources or metadata within files
- Use polymorphic or metamorphic techniques that automatically generate variants

Each modification produces a new hash. For sophisticated adversaries, generating new hashes is essentially free - automated, instantaneous, and requiring no operational changes.

### The Hash Paradox

Here's the paradox: hashes are simultaneously very useful and very limited.

**Useful**: When you have a hash of known malware and find that exact hash in your environment, you have high-confidence detection. False positive rate is near zero - matching hashes definitively identifies known malicious files.

**Limited**: Adversaries easily evade hash-based detection by trivially modifying malware. Hash detection only catches adversaries who use unchanged malware or tools - typically less sophisticated actors using commodity malware.

### Implications for Threat Hunting

Should threat hunters search for malicious hashes? Generally no, for several reasons:

**Automated Detection Handles This**: SIEM correlation rules, EDR systems, and antivirus can automatically compare file hashes against threat intelligence feeds. This happens at machine speed across millions of files. Human hunting adds no value to this automated process.

**Low Discovery Rate**: Sophisticated adversaries - exactly those automated detection misses and hunting should target - don't reuse unchanged malware. Hunting for hashes finds only unsophisticated threats that automated tools should already catch.

**Poor Resource Allocation**: Time spent hunting for hashes is time not spent hunting for behavioral patterns sophisticated adversaries use. It's focusing hunting effort where it's least likely to find threats that evaded automated detection.

**Exception**: Hash hunting makes sense in limited scenarios - for instance, searching for hashes of legitimate tools that adversaries abuse (living-off-the-land binaries) or when specific intelligence indicates particular malware family is active in your sector and automated tools aren't detecting it. But these are exceptions, not routine hunting focus.

## Level 2: IP Addresses (Easy)

The next level comprises IP addresses - the numeric addresses adversaries use for command and control servers, data exfiltration destinations, or attack infrastructure.

### Why IP Addresses Are Easy to Change

Adversaries maintain libraries of compromised servers, purchased hosting, and proxy services. Burning one IP address means simply switching to another from their available pool. For sophisticated adversaries:

- Cloud hosting provides on-demand IP addresses
- Compromised home routers or IoT devices provide disposable infrastructure
- Proxy and VPN services obscure true source addresses
- Domain fronting and CDN abuse hide command and control behind legitimate services

Changing IP addresses requires no malware modification, no operational procedure changes, and minimal cost. It's operationally easy.

### The Detection Value

IP-based detection is more valuable than hash-based detection because:

**Context Matters**: An IP address in logs provides investigative leads - what else communicated with this IP? When? What data was transferred? Hashes just confirm "this is malicious."

**Network Perimeter**: Blocking malicious IPs at network perimeter prevents communication even if endpoint is compromised. This provides defense-in-depth that hash blocking doesn't.

**Pattern Analysis**: Analyzing IP communication patterns (frequency, volume, timing) reveals behavioral indicators that individual IP addresses don't show.

### Implications for Threat Hunting

Hunting specifically for known malicious IPs has limited value for the same reasons as hash hunting - adversaries easily change them, and automated blocking/detection handles this efficiently. However, IP-based analysis is valuable when:

**Pattern Hunting**: Instead of hunting for specific malicious IPs, hunt for suspicious IP communication patterns - destinations with very few connections, communications to unusual countries, beaconing behaviors.

**Infrastructure Mapping**: When you identify one malicious IP, investigate related infrastructure - IPs in same netblock, domains hosted on same server, SSL certificates shared across IPs.

**Historical Analysis**: Search historical logs for past communications with now-known-malicious infrastructure to understand attack timeline and scope.

The key distinction: don't hunt for specific malicious IPs (automated tools do this), but do use IP analysis as part of broader behavioural hunting.


## Level 3: Domain Names (Annoying)

Domain names - the human-readable addresses adversaries use for command and control or phishing - sit higher in the pyramid because changing them causes adversaries more pain.

### Why Domains Are Annoying to Change

Domain changes create actual operational friction:

**Registration Costs**: Domains cost money and require payment methods that potentially leave trails. Bulk registration draws registrar attention.

**Propagation Delays**: DNS changes take time to propagate. During transition, some malware may lose command and control connectivity.

**Operational Coordination**: Adversaries must update malware configuration, inform operators of new domains, and coordinate changes across distributed operations.

**Detection Risk**: Newly registered domains or domain registration patterns often appear suspicious, potentially triggering defensive detection.

While not catastrophic, these frictions cause genuine operational annoyance - more pain than simply changing an IP address.

### Detection and Prevention Value

Domain-based detection provides several advantages:

**Longer Lifetime**: Adversaries reuse domains longer than IPs because changing causes operational friction. This makes domain-based detection more durable.

**Behavioral Patterns**: Domain naming patterns, registration characteristics, and DNS query behaviors provide detection opportunities beyond simple blocklists.

**Sinkholing**: Law enforcement or security vendors can seize domains, disrupting adversary operations more significantly than blocking IPs.

### Implications for Threat Hunting

Domain-based hunting is more valuable than IP hunting but still shouldn't focus on specific known-malicious domains (automated tools handle that). Instead:

**Pattern-Based Hunting**: Hunt for domains with suspicious characteristics - newly registered, unusual TLDs, algorithmically generated names (DGAs), DNS tunneling patterns, excessive subdomain levels.

**Infrastructure Analysis**: Investigate domain registration patterns, shared WHOIS information, passive DNS histories to understand adversary infrastructure.

**Behavioral Anomalies**: Hunt for unusual DNS query patterns - high volume queries to single domains, queries to domains that don't exist, unusual query timing patterns.

**Living-off-the-Land**: Hunt for abuse of legitimate services - adversaries increasingly use legitimate cloud services, file sharing platforms, and collaboration tools to blend command and control with normal traffic.



## Level 4: Network and Host Artifacts (Challenging)

Network and host artifacts include patterns embedded in adversary tools and techniques - URI patterns, user-agent strings, registry keys, file paths, service names, and other distinctive artifacts that appear across multiple operations.

### Why Artifacts Are Challenging to Change

These artifacts are often embedded deeply in adversary tooling:

**Tool Dependencies**: Malware may depend on specific registry keys for persistence or particular file paths for configuration. Changing these requires modifying and testing tools.

**Documentation and Training**: Operational procedures document specific artifacts. Changes require updating documentation and retraining operators.

**Compatibility Concerns**: Changing artifacts risks breaking functionality across different target environments or causing unexpected behaviour.

**Development Effort**: Unlike changing an IP (configuration change) or domain (registration change), modifying embedded artifacts requires development work, testing, and validation.

While possible, changing these artifacts causes genuine pain - requiring time, resources, and introducing risk of operational failures.

### Detection Value

Artifact-based detection is powerful because:

**Cross-Campaign Persistence**: Adversaries often reuse artifacts across campaigns, making detection more durable than indicators like IPs or domains.

**Tool Identification**: Specific artifacts identify particular tools or malware families, enabling threat intelligence correlation and attribution.

**Behavioural Context**: Artifacts often relate to specific techniques - certain registry keys indicate persistence mechanisms, particular user-agents indicate specific exploitation frameworks.

### Implications for Threat Hunting

Artifact hunting becomes genuinely valuable at this pyramid level:

**Technique-Focused Hunting**: Hunt for artifacts associated with specific attack techniques - scheduled task names typical of persistence mechanisms, PowerShell command patterns indicating fileless malware, WMI event subscription configurations.

**Tool Profiling**: Develop profiles of known adversary tools and hunt for their distinctive artifacts. Unlike hash-based detection, artifact patterns may persist across tool variants.

**Environmental Baselining**: Understanding normal artifacts in your environment (legitimate scheduled tasks, common user-agents, standard registry keys) enables detecting anomalous artifacts.

**Correlation Hunting**: Hunt for combinations of artifacts that individually seem benign but collectively indicate compromise - for instance, specific DLL combinations that indicate particular attacker frameworks.

## Level 5: Tools (Painful)

Tools - the malware, exploitation frameworks, scripts, and utilities adversaries use - represent significant investment and operational capability.

### Why Tools Are Painful to Change

Changing tools causes substantial pain:

**Development Investment**: Custom malware or modified frameworks represent months or years of development effort. Abandoning tools means losing substantial investment.

**Operational Capability**: Adversary operators develop expertise with specific tools. Switching tools means retraining and potential capability loss during transition.

**Tool Uniqueness**: Sophisticated adversaries' custom tools provide unique capabilities or evasion features that alternatives don't match. Generic tools may not meet operational needs.

**Testing and Validation**: New tools require extensive testing across target environments to ensure reliability. Premature deployment risks operational failures and detection.

**Infrastructure Dependencies**: Tools may be tightly integrated with adversary infrastructure, command and control systems, and data processing pipelines. Replacing tools means rebuilding these integrations.

### Detection Value

Tool-based detection is powerful because:

**Behavioural Signatures**: Tools exhibit distinctive behavioural patterns - particular sequences of API calls, specific persistence mechanisms, characteristic network communications.

**Operational Disruption**: Detecting and defending against specific tools forces adversaries into costly tool replacement or modification.

**Attribution**: Tool usage often enables threat actor attribution, providing strategic intelligence about who is attacking and why.

### Implications for Threat Hunting

Tool-focused hunting produces significant value:

**Behaviour Profiling**: Hunt for behavioural patterns associated with known adversary tools - Cobalt Strike's named pipes, Mimikatz's LSASS access patterns, PowerShell Empire's staging behaviours.

**Cross-Tool Families**: Hunt for techniques common across tool families rather than specific tools. For instance, many post-exploitation frameworks use similar privilege escalation or lateral movement patterns.

**Living-off-the-Land Tools**: Increasingly, adversaries use legitimate administrative tools (PowerShell, WMI, PsExec) rather than custom malware. Hunt for suspicious uses of legitimate tools - administrative tools run by non-administrators, unusual combinations of legitimate tools, or legitimate tools used in suspicious sequences.

**Tool Capability Detection**: Rather than hunting for specific tools, hunt for the capabilities they provide - credential dumping, lateral movement, data staging, exfiltration. This remains effective even when specific tools change.


## Level 6: TTPs (Tough/Challenging)

At the pyramid's apex are Tactics, Techniques, and Procedures - the fundamental methodologies adversaries use to achieve objectives. This is where threat hunting should focus most effort.

### Understanding TTPs

The term "TTPs" deserves unpacking:

**Tactics**: High-level adversary objectives during operations - initial access, execution, persistence, privilege escalation, defense evasion, credential access, discovery, lateral movement, collection, exfiltration, impact. The MITRE ATT&CK framework organizes adversary behaviour around these tactical objectives.

**Techniques**: Specific methods adversaries use to achieve tactical objectives. For instance, within the "Persistence" tactic, techniques include scheduled tasks, registry run keys, service creation, DLL hijacking, etc.

**Procedures**: Specific implementations of techniques - exactly how particular adversaries execute techniques, including tool choices, specific commands, timing patterns, and operational sequences.

### Why TTPs Are Tough to Change

Changing TTPs causes adversaries maximum pain:

**Operational Methodology**: TTPs represent core adversary methodology developed through experience and training. They're not arbitrary choices but refined approaches that work reliably.

**Organizational Knowledge**: In organized adversary groups, TTPs are documented, taught to operators, and embedded in standard operating procedures. Changing them requires organizational retraining.

**Capability Dependencies**: Some TTPs depend on unique capabilities or access that alternatives don't provide. For instance, adversaries with particular exploitation capabilities can't simply abandon them without losing tactical options.

**Success History**: Adversaries continue using TTPs that work. Forcing them to change means they must adopt untested approaches with unknown effectiveness.

**Fundamental Trade-offs**: Many TTPs involve fundamental trade-offs - stealth vs speed, reliability vs complexity, broad access vs deep persistence. Changing TTPs means accepting different trade-offs that may not suit operational objectives.

### Why TTP-Focused Hunting Is Most Valuable

Focusing hunting on TTPs provides maximum value for several reasons:

**Durability**: TTP-based detection remains effective across multiple campaigns and even across different adversary groups using similar methodologies. Unlike indicators that adversaries quickly change, TTPs persist.

**Broad Coverage**: Detecting TTPs catches not just known threats but unknown threats using similar techniques. You're detecting behaviors rather than specific artifacts.

**Operational Impact**: When detection forces adversaries to change TTPs, they face genuine operational difficulty - requiring retraining, accepting capability loss, or abandoning approaches they know work.

**Intelligence Value**: Understanding adversary TTPs provides strategic intelligence about adversary capabilities, priorities, and likely future actions - far more valuable than knowing they used a particular IP address.

**Scalable Learning**: As hunters develop expertise in detecting TTPs, that expertise transfers across different threat scenarios. Learning to detect credential dumping techniques applies to all adversaries using those techniques, not just specific tools or campaigns.

### TTP Hunting in Practice

What does TTP-focused hunting look like practically?

**Technique-Based Hypotheses**: Generate hunting hypotheses around specific MITRE ATT&CK techniques: "Are adversaries using WMI for lateral movement?" "Is there evidence of pass-the-hash authentication?" "Are there signs of data staging before exfiltration?"

**Behavioral Pattern Recognition**: Hunt for behavioral patterns that indicate techniques regardless of specific tools - unusual authentication patterns suggesting credential theft, suspicious process relationships indicating code injection, anomalous network flows suggesting command and control.

**Procedural Analysis**: When investigating incidents or anomalies, document not just what artifacts were found but how adversaries operated - their procedures, tool usage patterns, timing, and sequencing. This procedural understanding enables hunting for similar operations in the future.

**Technique Stacking**: Hunt for combinations of techniques that comprise complete attack chains. Single techniques might appear benign, but specific sequences indicate sophisticated attacks - for instance, discovery commands followed by credential access attempts followed by lateral movement.

**Environmental Context**: Apply environmental understanding to recognize when techniques are used maliciously versus legitimately. Administrators legitimately use many "adversary" techniques - what makes them suspicious is context: who's using them, when, why, and in what sequence.

## The Strategic Shift: From Indicators to Behaviours

The Pyramid of Pain advocates a fundamental strategic shift in defensive thinking - from indicator-based detection to behaviour-based detection.

### Traditional Indicator-Based Approach

Traditional security operations focus heavily on indicators of compromise (IOCs) - specific artifacts like hashes, IPs, domains that identify known threats. This approach has strengths:

- **Precision**: Matching specific indicators provides high-confidence detections with few false positives
- **Simplicity**: Indicator matching is computationally simple and scales easily
- **Shareability**: Indicators are easy to share through threat intelligence feeds

But fundamental weaknesses limit effectiveness:

- **Brittleness**: Adversaries trivially change indicators, making detection quickly obsolete
- **Reactive**: Indicator-based detection only catches known threats, missing novel attacks
- **No Context**: Matching indicators provides detection but limited understanding of adversary objectives or procedures

### Behaviour-Based Approach

Behaviour-based detection focuses on adversary techniques and procedures - the TTPs at the pyramid's top. This approach flips the strengths and weaknesses:

**Challenges**:

- **Complexity**: Detecting behaviors requires sophisticated analytics, environmental baselines, and contextual judgment
- **False Positives**: Legitimate activities sometimes resemble adversary techniques, requiring investigation to distinguish
- **Development Effort**: Behaviour-based detection requires significant development, testing, and tuning

**Strengths**:

- **Durability**: Behaviour-based detection remains effective across campaigns and even adversary groups
- **Proactive**: Detecting techniques catches both known and unknown threats using similar methods
- **Intelligence**: Behavior detection provides understanding of adversary methodology, enabling strategic response

### The Balanced Approach

The Pyramid of Pain isn't suggesting abandoning indicator-based detection - it's arguing for proper allocation of resources and effort:

**Automate Indicator Detection**: Let automated systems (SIEM rules, EDR detection, threat intelligence feeds) handle indicator matching. These systems do it efficiently at scale, freeing humans for higher-value work.

**Human Focus on Behaviors**: Allocate human expertise - particularly threat hunting - to detecting behaviors and TTPs. This is where human judgment, creativity, and contextual understanding add unique value that automation can't replicate.

**Continuous Improvement Cycle**: When hunting discovers new adversary techniques, feed those discoveries back into automated detection systems. What hunters discover manually should become automated detection, raising the bar and freeing hunters to discover new techniques.

This balanced approach plays to each component's strengths - automation for scale and consistency, humans for insight and adaptation.

## Practical Application to Threat Hunting

How does the Pyramid of Pain translate into practical threat hunting decisions?

### Hypothesis Generation

When generating hunting hypotheses, bias toward pyramid's top:

**Poor Hypothesis**: "Search for known malicious IP addresses from last month's threat intelligence report"

- Why poor: IP addresses are easy to change, automated tools should handle this, unlikely to find sophisticated threats

**Better Hypothesis**: "Search for beaconing network traffic patterns consistent with command and control communications seen in recent compromises"

- Why better: Focuses on behavioural pattern (beaconing) rather than specific indicators, catches both known and unknown C2 channels

**Best Hypothesis**: "Investigate whether adversaries are using WMI for lateral movement, examining unusual WMI process creation events from non-administrative accounts"

- Why best: Focuses on specific technique (WMI lateral movement), considers environmental context (non-admin users), targets TTP that's difficult for adversaries to abandon

### Investigation Approach

When investigating, move up the pyramid:

If you start with an indicator (say, a suspicious IP address), don't stop at validating the indicator. Ask:

- What technique does this indicator represent? (Perhaps command and control communication)
- What tools might have produced this indicator? (Specific malware family or framework)
- What procedures does this suggest? (Particular adversary operational pattern)
- What other TTPs might this adversary be using? (Likely lateral movement, credential access, etc.)

This pyramid ascent transforms indicator investigation into TTP understanding, providing broader context and enabling proactive hunting for related activities.

### Coverage Planning

When planning hunting coverage, prioritize systematically:

**High Priority**: TTP coverage - work through MITRE ATT&CK techniques relevant to your environment, ensuring you can detect each technique or have hunted for evidence of its use.

**Medium Priority**: Tool profiling - understand what tools might target your environment and how to detect their use behaviorally, not just through signatures.

**Lower Priority**: Artifact hunting - hunt for distinctive artifacts when they're particularly durable or closely tied to TTPs, but don't make this primary focus.

**Automated**: Indicator matching - ensure automated systems handle hash, IP, and domain indicators efficiently; don't waste hunting time on this.

### Value Demonstration

When communicating hunting value to leadership, emphasize pyramid top:

**Weak Value Claim**: "We hunted for 1,000 malicious hashes and found three matches"

- Why weak: This work should be automated, finding only three matches suggests most threats use different hashes

**Strong Value Claim**: "We hunted for lateral movement using WMI and discovered a technique our automated detection wasn't catching. We've created detection rules and searched for historical evidence, finding two previously undetected compromises"

- Why strong: Focuses on TTP detection, shows detection gap closure, demonstrates both immediate discovery and detection improvement



## Common Mistakes and Misconceptions

Several common mistakes occur when applying (or misapplying) the Pyramid of Pain:

### Mistake 1: Abandoning Lower Levels Entirely

Some interpret the Pyramid as "don't bother with indicators at all." This is wrong. Indicators have value - just not as hunting focus. Maintain indicator-based detection through automation, but don't allocate human hunting time to searching for known indicators.

### Mistake 2: TTP Hunting Without Environmental Understanding

Hunting for TTPs without understanding your environment leads to overwhelming false positives. Administrators legitimately use many "adversary techniques." The key is contextual understanding - who should be doing what, when, and why. Develop this understanding before aggressive TTP hunting.

### Mistake 3: Over-Focusing on Attribution

Some interpret TTP hunting as "identify which adversary group is attacking us." While attribution can emerge from TTP analysis, it's not the primary goal. Focus on detecting and disrupting adversary operations regardless of who's conducting them. Attribution is byproduct, not objective.

### Mistake 4: Ignoring Tool Reality

While theoretically adversaries could abandon tools, in practice they don't do so lightly. Tool-focused hunting (level 5) provides significant value between abstract TTP hunting and concrete indicator matching. Don't skip this level in rush toward pure TTP focus.

### Mistake 5: Forgetting the Baseline

TTP hunting requires understanding normal behaviour. Without baselines, every administrative action looks potentially malicious. Before aggressive TTP hunting, invest in environmental baselining so you can recognize what's actually anomalous.


## The Future: Moving Beyond the Pyramid

While the Pyramid of Pain remains foundational, defensive thinking continues evolving:

**Behavioural Analytics**: Machine learning and advanced analytics increasingly detect TTP-level patterns automatically, potentially automating some behaviour detection that currently requires human hunting.

**Deception Technologies**: Honeypots and deception platforms detect adversary reconnaissance and lateral movement techniques, providing TTP detection through adversary interaction with fake assets.

**Assumed Compromise**: The maturity of assumed breach thinking (Chapter 3) means increasingly sophisticated defenders assume TTPs are being used and hunt to prove or disprove that assumption rather than waiting for indicators.

**Cross-Domain TTPs**: As environments become more complex (cloud, containers, IoT), TTPs increasingly span domains. Future hunting may focus on cross-domain behavioural patterns that no single security tool sees completely.

These evolutions don't invalidate the Pyramid of Pain - they extend and reinforce its core insight: focus on adversary behaviours that are difficult to change rather than artifacts they easily modify.




