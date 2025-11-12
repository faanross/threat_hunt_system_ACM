# Hunt What Hurts: The Pyramid of Pain

## The Tyranny of Infinite Choice

Threat hunting is fundamentally different from other security operations. When an alert fires in the SIEM, or when the EDR flags suspicious activity, the SOC analyst's first step is decided for them - investigate the notification. Security incident responders face a similar clarity: they're handed an incident and must contain, eradicate, and recover. The starting point is given; the path forward, while complex, begins at a defined location.


Threat hunters face a different challenge entirely.

Hunting is proactive, not reactive. No alert has fired. No system has flagged anything. You're staring at millions of log entries, thousands of systems, and hundreds of potential threat vectors - all while nothing appears obviously wrong. You must decide not only what to investigate, but where to begin, how to prioritize, and why one hunting hypothesis deserves attention over countless alternatives.

This is the hunter's dilemma: the tyranny of infinite choice.

You could hunt for suspicious network traffic, anomalous authentication patterns, unusual process executions, irregular PowerShell usage, abnormal file modifications, or a thousand other possibilities. Each path seems equally valid - or equally arbitrary. Without some organizing principle, this freedom becomes paralyzing. You end up either hunting randomly (wasting time on low-value activities) or falling back to what feels concrete and measurable: searching for known malicious IPs, checking file hashes against threat intel feeds, matching domains to reputation databases.

But this reactive approach - hunting for specific known-bad indicators - misunderstands what makes threat hunting valuable and what adversaries actually find difficult to evade. It treats hunting like automated detection with a human manually running the queries.

What threat hunters need is something - a mental model, a framework, a systematic approach, or even just a set of guiding principles - to help navigate that critical first step. Something to distinguish high-value hunting activities from security theater. A way to prioritize where to focus limited time and expertise based on strategic impact rather than tactical convenience or what simply feels productive.

Different organizations solve this differently. Some develop internal methodologies tailored to their specific environment and threats. Others adopt established frameworks with detailed processes and metrics. On a more abstract level, some may also employ mental models - conceptual tools that shape how hunters think about the problem without prescribing specific techniques.

The Pyramid of Pain is one such mental model. [Created by David Bianco](https://youtu.be/3Xrl6ICxKxI?si=w3W-8MxdIKRyn7-s) in 2013 while working at Sqrrl, it offers a way to think about detection strategy through a simple lens: adversaries easily change some artifacts (file hashes, IP addresses) but struggle to change others (their operational techniques and procedures). The model suggests prioritizing hunting efforts on what hurts adversaries most to modify - not what's easiest for us to detect.

## Understanding the Pyramid

The Pyramid of Pain arranges adversary artifacts by how much "pain" changing them causes:


![pyramid_of_pain](./img/pyramid.png)



At the bottom sit hash values and IP addresses - trivial for adversaries to change. At the top sit tactics, techniques, and procedures (TTPs) - the operational methods adversaries depend on and can't easily abandon. The model inverts conventional thinking: instead of hunting signatures, hunt behaviors.


## Hash Values - Trivial to Change

At the pyramid's base sit hash values - cryptographic signatures of files, typically malware. Security tools use these to identify known malicious files through exact matching.

But hashes are trivial to change. An adversary need only alter a single bit, and since any competent attacker can implement polymorphic techniques to automatically generate variants, this change is essentially free and effortless.

Though hashes have limited use for threat hunting, they still hold some value for automated detection. While easy to change, hashes aren't always modified, and when you find an exact hash match, you have high-confidence detection with near-zero false positives.

Should we as threat hunters search for malicious hashes? Generally no. SIEM rules, EDR systems, and antivirus automatically compare file hashes against threat intelligence feeds at machine speed. Human hunting adds no value to this automated process. Time spent hunting for hashes is time not spent hunting for behavioral patterns that sophisticated adversaries use - exactly those that evaded automated detection.

## IP Addresses - Easy to Change

IP addresses - the numeric addresses adversaries use for command and control, exfiltration, or other attack infrastructure - rank only slightly higher on the Pyramid of Pain since they remain easy to change.

Many adversaries use VPS providers and major cloud platforms that offer on-demand infrastructure with minimal friction. They may also maintain pools of compromised servers for disposable infrastructure. Commercial proxy services and residential proxy networks provide ready access to rotating IP addresses. Bulletproof hosting providers specifically cater to malicious use by ignoring abuse complaints. Changing IPs requires no source code modification - adversaries simply point their C2 to a new address from their available pool, a process that takes seconds to minutes and costs next to nothing.

As with hash values, searching for specific known-malicious IPs is best left to automated systems. Your firewall, IDS/IPS, and SIEM can efficiently match IPs against threat intelligence feeds and block or alert on matches at machine speed. Human hunters add minimal value manually searching logs for known-bad IPs that automated detection should already catch.

The exception is when you're investigating patterns of IP behavior. For example, destinations that were only connected to from a single host, communications to unexpected geographic regions, or connections made using regular beaconing intervals etc. Here human judgment and contextual understanding provide value that automation cannot easily replicate.

## Domain Names - Annoying to Change

Domain names - the human-readable addresses for malicious infrastructure - sit higher on the pyramid because changing them creates genuine operational friction.

Unlike hash values and IP addresses, domains are annoying to change. Registration costs money and requires payment methods that create financial trails. DNS propagation takes time, potentially causing C2 connectivity gaps during transitions. Adversaries must update malware configurations, coordinate changes across their operational team, and manage the shift across distributed infrastructure. Newly registered domains often trigger defensive scrutiny from reputation systems and threat intelligence feeds.

Despite this elevated position, hunting for specific known-malicious domains still offers limited value - automated detection handles reputation checking and blocklisting efficiently. The real hunting value lies in domain analysis for patterns.

Be on the lookout for behavioral indicators like domain generation algorithms (DGAs) producing similar-looking domains, unusually high subdomain counts associated with a single domain, or registration clustering where multiple domains share creation dates or registrar information. Hunt for domain relationships and behaviors that expose adversary tradecraft, not individual known-bad domains that automated systems should already catch.

## Network & Host Artifacts - Challenging to Change

Here things start becoming more interesting for us as threat hunters. Network and host artifacts - URI patterns, certificate characteristics, process creation sequences, registry modifications, specific HTTP headers - are tied to how malware actually operates.

Changing them means modifying the malware's operational characteristics, not just swapping infrastructure. An adversary can easily point malware at a new IP, but changing these artifacts requires deeper alterations to the malware itself, redeployment through existing compromised systems, and potentially re-establishing command and control channels - all while maintaining operational security and avoiding detection during the transition.

This is where threat hunting begins showing real value.

Instead of hunting for specific malicious IPs, hunt for beaconing patterns - regular communication intervals suggesting automated check-ins. Instead of known file hashes, hunt for suspicious process creation chains where legitimate tools spawn each other unexpectedly (cmd.exe spawning PowerShell spawning wscript.exe). Look for unusual registry Run key modifications from non-standard processes. Investigate HTTP requests with suspicious User-Agent strings or MIME type discrepancies. Hunt for processes making network connections from unexpected parent processes (Word connecting outbound). 

  

We're now beginning to hunt for what malware does, not what it is.



## Tools - Difficult to Change

Tools - malware frameworks, lateral movement utilities, credential theft tools - rank even higher because adversaries invest significant effort developing or customizing them.

While theoretically adversaries could drop any tool in favor of a new one, in practice they don't do so lightly. Tools represent operational capability they've built, tested, and trained their operators to use. Suitable replacements are almost always available, but the cost to transition is significant.

Tool-focused hunting provides substantial value. Profile how tools behave - what artifacts they create, what network patterns they exhibit, what system interactions they perform. When you understand tool behavior rather than just signatures, you can detect that tool even after the adversary modifies easily-changed artifacts like hashes or IPs.

[Consider a Cobalt Strike Beacon](https://thedfirreport.com/2021/08/29/cobalt-strike-a-defenders-guide) - rather than hunting for specific beacon payload hashes, hunt for its characteristic named pipes (patterns like \.\pipe\MSSE-###-server), SMB beaconing on port 445, or distinctive sleep/jitter from known malleable profiles using network behavioral analysis. 

Tool-based hunting is practical and effective - specific enough for actionable detections, yet abstract enough to survive infrastructure changes. Yet, adversaries forced to abandon one tool can ultimately adopt another - it happens all the time. So the most resilient target of our hunting efforts should be the operational behaviours themselves.



## TTPs - Most Painful to Change

At the pyramid's apex sit TTPs - tactics, techniques, and procedures - the fundamental operational methods adversaries use to accomplish their mission.

A tactic is the adversary's goal: establish C2, move laterally, exfiltrate data. A technique is the specific method: using WMI for lateral movement, leveraging stolen credentials for privilege escalation, employing DNS tunneling for exfiltration. A procedure is the adversary's specific implementation - their exact operational tradecraft.

Changing TTPs requires fundamentally altering not only what adversaries do, but how they do it. TTPs are integrated into operational muscle memory, validated through actual compromises, and often dictated by target environment constraints. If an adversary reliably uses WMI because it works in their target's environment and their operators know it well, forcing them to abandon WMI means retraining operators, finding alternative techniques, and validating new procedures under operational conditions. This represents real cost and operational risk.

For us as threat hunters, TTPs hold the most value, but also require the highest level of expertise. Anyone can copy and paste a hash value from a CTI feed and search for it. But diving deep into logs and data sources to recognize the emergent patterns that reveal specific TTPs is both an art and a science built upon deep domain mastery.

Skilled hunters can spot techniques like lateral movement via WMI, credential dumping with LSASS access, or data staging in unusual directories. They can hunt for behavioral patterns that reveal adversary procedures. The real power of hunting here is that it can identify both known and unknown threats using similar methods, and can provide an understanding of adversary methodology that will deeply support subsequent response efforts. 



## The Balanced Reality

To be clear: The Pyramid of Pain doesn't suggest that you abandon indicator-based detection entirely. 

Indicators have value - just not as the primary target of hunting efforts. Instead, they are the ideal fodder for automated detection - it’s cheap, fast, and tends to minimize false positives. So if you're spending human hunting time searching for known indicators, you're optimizing for the wrong adversaries - the ones your automated defenses should already stop.

Instead, the model advocates that we invest precious human expertise where it adds unique value: hunting for behaviors that require contextual understanding by generating creative hypotheses, and investigating in an adaptive, dynamic manner. This is where human knowledge, intuition, and contextual understanding can't be replicated by automation, at least not without generating substantial false positives.

And importantly - ensure systems are in place so that threat hunting insights aren't ephemeral. When hunting discovers new adversary indicators, feed those discoveries back into automated detection systems. What hunters discover manually should become automated detection, raising the defensive baseline and freeing hunters to discover novel behaviors. 

## Conclusion

The Pyramid of Pain delivers one essential insight: optimize your defensive strategy around what's expensive for adversaries to change, not what's easy for you to detect.

This seems counterintuitive since it’s natural to hunt for concrete, specific things - this hash, that IP, these domains. It feels productive and measurable, it lends itself well to checking boxes and writing neat reports. But this is merely a charade that produces vanity metrics and will have no lasting impact on security posture by threat hunting efforts.



So break that pattern and shift your hunting focus up the pyramid. In doing so you not only become better at finding current threats, you also make future threats less likely to occur. **Threat Hunting’s true value proposition is not to find something bad, it’s to make it increasingly harder for something bad to occur at all**.
