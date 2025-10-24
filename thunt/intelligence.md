
# The Role of Threat Intelligence in Hunting

## Intelligence as Context, Not Command

Imagine being a detective investigating crimes in your city. You receive reports from other cities about a particular burglar's methods - always enters through second-story windows, targets homes with security systems by disabling them first, leaves a specific calling card. This intelligence doesn't tell you whether this burglar is active in your city, but it tells you what to look for if they are. When you find a burglary with these characteristics, you have context about who might be responsible and what they might do next.

Threat intelligence serves a similar role in threat hunting. It provides context about adversary behaviours, tools, and techniques without dictating what you must hunt for or how to hunt. Intelligence describes what's happening in the broader threat landscape - active campaigns, emerging techniques, adversary tradecraft - enabling hunters to make informed decisions about what to investigate and how to recognize threats when found.

Yet the relationship between threat intelligence and threat hunting is frequently misunderstood. Some organizations treat intelligence as commands that must be acted upon immediately: "Intelligence says adversary X is active - drop everything and hunt for them!" Others dismiss intelligence as irrelevant: "We hunt based on our environment, not external intelligence." Both extremes miss the optimal balance: intelligence informs hunting without controlling it, providing context and priorities without replacing hunter judgment and creativity.

The most effective approach recognizes that intelligence and hunting exist in a synergistic relationship. Intelligence provides invaluable context and direction. Hunting validates intelligence and generates ground truth. Each enhances the other in a continuous feedback loop.


## Three Flavours of Intelligence

Threat intelligence comes in multiple forms, each serving different hunting purposes. Understanding these distinctions helps hunters apply intelligence appropriately.

**Strategic intelligence** addresses high-level questions about the threat landscape. It answers questions like: Which adversaries target our sector? What are their motivations and capabilities? What trends are emerging? This intelligence operates on a timeline of months to years, focusing on big-picture patterns rather than specific technical details. When intelligence reports that nation-state actors are increasingly targeting healthcare due to pandemic research value, or that ransomware groups are shifting from encryption to data theft and extortion, that's strategic intelligence.

For threat hunting, strategic intelligence informs program-level decisions rather than specific hunts. It helps answer: Which threats warrant hunting investment? What techniques should we prioritize? What assets require enhanced monitoring? What tools and solutions should we invest in? Strategic intelligence shapes overall hunting strategy but doesn't directly generate specific hunt plans.

**Tactical intelligence** describes adversary tactics, techniques, and procedures - the "how" of attacks. This intelligence operates on a weeks-to-months timeframe and provides the behavioural details that make effective hunting possible. When you learn that APT29 uses WMI for lateral movement and establishes persistence through COM hijacking, or that ransomware operators are exploiting unpatched VPN appliances for initial access before deploying Cobalt Strike for post-exploitation, that's tactical intelligence.

This is particularly valuable for threat hunting, since it allows hunters to generate specific hunting hypotheses about adversary techniques, provides behavioural patterns to search for, suggests investigation approaches for specific TTPs, and contextualizes suspicious findings. Tactical intelligence translates directly into hunt activities - it describes behaviours you can actually hunt for in your environment. This includes, significantly, helping overcome the initial (and often most severe) blockage in the hunting process - "where do I even begin?"




**Operational intelligence** provides specific details about active adversary operations and infrastructure. It operates on a days-to-weeks timeline and includes concrete technical indicators: active phishing campaign email subjects, command-and-control infrastructure IP addresses and domains, specific malware file hashes and behaviours. Operational intelligence is highly specific and immediately actionable.

For hunting, operational intelligence provides concrete targets to search for and active threat context for suspicious findings. However, it has severe limitations that must be understood honestly.

## The Brittleness of Operational Intelligence

Operational intelligence, particularly indicators, represents the most fragile form of threat detection. Once an IP address appears on an intelligence feed, competent adversaries have typically already moved on. Domains get burned and replaced within hours. Malware hashes change with each compilation. This brittleness - what the Pyramid of Pain framework describes as "easy for adversaries to change" - makes indicators fundamentally reactive. By the time intelligence documents an indicator, it's often already obsolete for detecting active operations.

This isn't to say indicators are useless. They're valuable for historical investigation, attribution context, and catching unsophisticated threats. But their limitations are real: they detect past campaigns better than current ones, sophisticated adversaries trivially evade indicator-based detection, and they provide no defense against novel threats or customized operations. An adversary group you've never seen before, using infrastructure not in any feed, remains completely invisible to indicator-based intelligence.

This brittleness is precisely why threat hunting and behavioural analysis exist. While indicators rapidly decay in value, adversary techniques persist much longer. An adversary might change IP addresses hourly, but their fundamental approach to lateral movement or credential theft remains recognizable across campaigns. Behavioral detection - hunting for how adversaries operate rather than specific artifacts they left behind - addresses the core weakness of indicator intelligence. You can't easily change fundamental operational requirements like credential access, privilege escalation, or command-and-control communication patterns.

Operational intelligence works best when automated systems handle indicator matching while hunters focus on behavioural detection that remains effective regardless of which specific infrastructure adversaries use today.



## Intelligence-Driven Hunting in Practice

Intelligence-driven hunting starts with external threat intelligence and searches for evidence of described threats in your environment. The process flows naturally: consume intelligence from various sources, assess which threats actually apply to your environment, translate that intelligence into testable hunting hypotheses, design investigations to detect those behaviours, execute the hunt, and feed results back to improve both hunting and intelligence.

The key is translating intelligence into hypotheses appropriate for your specific environment. Generic intelligence about adversary behaviour needs contextual translation. If intelligence describes lateral movement via WMI, you need to consider: Do we have visibility into WMI activity? What would legitimate WMI usage look like in our environment? What patterns would distinguish malicious from benign WMI operations? This translation process converts abstract threat descriptions into concrete hunting activities tailored to your infrastructure and logging capabilities.

Intelligence-driven hunting excels at covering known threats systematically. It ensures your program addresses documented adversary behaviours rather than hunting blindly. It provides focus when threats multiply and priorities compete. It validates whether known threats exist in your environment before they cause damage.

## Environment-Driven Hunting: The Other Half

Intelligence-driven hunting isn't the complete picture. Environment-driven hunting - starting from your environment rather than external intelligence - provides equal value through a different lens.

Environment-driven hunting begins by analyzing your actual environment, searching for anomalies, outliers, and suspicious patterns without preconceived notions about specific threats. This approach excels at discovering threats that don't match existing intelligence descriptions: zero-day techniques, insider threats following unexpected patterns, and adversaries using novel tradecraft.

The power of environment-driven hunting lies in its openness to the unexpected. While intelligence-driven hunting asks "Is known threat X present?", environment-driven hunting asks "What unusual patterns exist?" It finds threats intelligence hasn't documented yet, detects adversaries avoiding known detection patterns, and reveals environmental-specific attack variations.



The most effective hunting programs balance both approaches. Intelligence-driven hunting provides structure and ensures known threats receive attention. Environment-driven hunting provides flexibility and discovers novel threats. Neither approach alone is sufficient - they complement each other, covering different blind spots.

In practice, many hunts blend both approaches. You might start with intelligence about a technique, then pivot to environment-driven investigation when you find something unexpected. Or begin with environment-driven anomaly detection, then use intelligence to contextualize findings. The distinction matters less than maintaining both perspectives.

## Making Intelligence Actionable

The greatest challenge isn't obtaining threat intelligence - numerous sources provide free, high-quality intelligence. The challenge is operationalizing intelligence effectively without drowning in information overload or becoming enslaved to external priorities.

Start by being selective about intelligence sources. Subscribe to sector-specific ISACs, follow vendors who provide detailed technical analysis, monitor government advisories relevant to your geography and industry, and track threat actor groups known to target organizations like yours. Quality and relevance matter far more than quantity. **Three highly relevant intelligence sources beat thirty generic ones**.

Filter intelligence ruthlessly for relevance. Not all threats apply to all organizations. Intelligence about adversaries targeting operational technology matters immensely if you operate OT networks but provides minimal value if you're purely enterprise IT. Geographic targeting matters - North Korea-attributed adversaries targeting cryptocurrency exchanges have limited relevance if you're a healthcare provider. Ask constantly: Does this threat actually apply to us?

Focus on TTPs rather than pure indicators. Indicators change rapidly, but adversary techniques persist longer. Intelligence describing how adversaries move laterally using stolen credentials remains relevant longer than intelligence about specific command-and-control domains. Build your intelligence repository around behaviours and techniques rather than fleeting indicators. Automate indicator matching where possible and reserve hunting attention for behavioural detection.

Maintain manageable intelligence consumption routines. Daily morning intelligence review takes 15-30 minutes scanning overnight feeds for critical alerts and identifying immediate priorities. Weekly intelligence planning involves deeper review to generate hunting hypotheses and schedule hunts. Monthly intelligence analysis provides comprehensive threat landscape assessment and strategic adjustments. Without structure, intelligence consumption becomes either neglected or overwhelming.


## Common Intelligence Pitfalls

Organizations repeatedly make predictable mistakes integrating intelligence with hunting. Awareness helps avoid these traps.

The indicator obsession problem manifests as focusing exclusively on indicator matching rather than behavioural hunting. "Let's hunt for these IPs and hashes" becomes the entire program. The solution: automate indicator detection and focus hunting on TTPs and behaviours. Use indicators for context, not as the primary hunting target.

The passive consumption trap occurs when organizations only consume intelligence without validating it against their environment. Intelligence accumulates unquestioned and unused. The solution: every intelligence report should prompt a question: "What does this mean for us? How would we detect this?" Turn intelligence consumption into active engagement.

The intelligence slavery problem emerges when teams only hunt when intelligence directs, never conducting environment-driven exploration. "Intelligence didn't mention this, so we ignore it." The solution: balance intelligence-driven and environment-driven hunting. Use intelligence as input, not the sole driver.

Stale intelligence causes problems when teams use outdated intelligence without assessing current relevance. Adversaries evolve continuously - intelligence from two years ago may not reflect current behaviour. Always assess intelligence freshness and validate continued relevance.

Intelligence overload overwhelms teams subscribing to every possible source. The solution: curate aggressively. Prioritize quality over quantity.




## The Bidirectional Relationship

The most sophisticated programs recognize that intelligence and hunting have a bidirectional relationship. Intelligence informs hunting, but hunting should also inform intelligence.

When hunts validate intelligence - confirming described techniques in your environment - share that validation back to the intelligence community. When hunts find adversary behaviours not described in current intelligence, document and share those discoveries. When intelligence proves inaccurate or inapplicable in your environment, that's valuable negative feedback.

This sharing strengthens the broader security community. Threat intelligence improves when everyone contributes findings. Your discoveries help other organizations detect the same threats. Their discoveries help you. The community benefits from this reciprocity.

Even if formal intelligence sharing seems daunting, start small. Document findings for internal reference. Share sanitized discoveries with trusted peers. Contribute to sector-specific information sharing communities. Any contribution helps.

## The Bottom Line

Threat intelligence provides invaluable context for threat hunting - describing what adversaries do, what techniques they use, and what threats are active in your sector. But intelligence should inform and guide hunting, not control or replace it. The hunter's judgment about what to investigate and how remains paramount.

The most effective approach balances intelligence-driven and environment-driven hunting, uses tactical intelligence to generate specific hunting hypotheses, maintains selective and relevant intelligence sources, focuses on behaviours over fleeting indicators, and feeds hunting discoveries back to improve collective intelligence.

Intelligence answers "What should we consider hunting for?" Hunting answers "What actually exists in our environment?" Together, they create comprehensive threat detection that neither achieves alone.
