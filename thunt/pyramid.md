# The Pyramid of Pain and Hunting Focus Areas

## The Question of Where to Hunt

Imagine you're a threat hunter facing an environment with millions of log entries, thousands of systems, and hundreds of potential threat scenarios. Where do you start? What should you hunt for? These aren't just practical questions - they're strategic ones that determine whether your hunting efforts produce meaningful value or consume resources without impact.

Many new threat hunters instinctively focus on what seems concrete and specific: hunting for known malicious IP addresses, searching for file hashes from malware reports, or looking for domain names associated with command and control infrastructure. These hunts feel productive because they're specific and measurable - "We searched for 500 malicious IPs and found three matches!" But this approach fundamentally misunderstands what makes threat hunting valuable and what adversaries find difficult to evade.

This is where David Bianco's Pyramid of Pain becomes essential. First published in 2013, this simple but profound model revolutionized how defenders think about threat indicators and detection strategy. It provides a framework for understanding which adversary artifacts are trivial to change (and therefore low-value to hunt for) versus which are deeply embedded in adversary operations (and therefore high-value hunting targets).

Understanding the Pyramid of Pain transforms how you approach threat hunting. It shifts focus from hunting for specific artifacts that adversaries easily change to hunting for behavioural patterns and techniques that adversaries can't easily abandon. This chapter explores the Pyramid of Pain model, its implications for threat hunting strategy, and why focusing on TTPs (Tactics, Techniques, and Procedures) rather than atomic indicators is fundamental to effective hunting.

## The Pyramid of Pain: Structure and Concept

The Pyramid of Pain visualizes different types of adversary indicators arranged by how much "pain" (difficulty, cost, operational impact) changing them causes adversaries:

[!pyramid_of_pain](./img/pyramid.png)

