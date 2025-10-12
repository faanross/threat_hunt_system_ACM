# Threat Hunting Frameworks and Methodologies

## Why Frameworks Matter (And Why They Don't)

In the previous chapter, we explored the fundamental hunting loop that underlies all threat hunting activities. That conceptual foundation is essential, but it leaves many practical questions unanswered: How exactly do you generate good hypotheses? What's the detailed process for moving from investigation to actionable detection rules? How do you structure your team's hunting activities to ensure systematic coverage?

This is where frameworks come in. A threat hunting framework provides structured methodology, detailed process steps, and operational guidance that helps translate the conceptual hunting loop into repeatable practice. Good frameworks help teams avoid reinventing approaches that others have already refined, provide common language for discussing hunting activities, and ensure systematic rather than haphazard hunting.

But here's the paradox: frameworks are simultaneously helpful and potentially limiting. They're helpful because structure enables consistency, teachability, and organizational scaling. They're potentially limiting because threat hunting fundamentally requires creative investigation that doesn't always fit predetermined steps. The best hunting often happens when skilled practitioners adapt or abandon formal processes to follow where evidence leads.

The solution to this paradox is to view frameworks as tools in your toolkit rather than rigid prescriptions. Understand multiple frameworks, recognize their strengths and intended contexts, and then flexibly apply whichever approach best fits your current investigation. Sometimes you'll follow a framework closely; other times you'll blend elements from multiple frameworks; occasionally you'll set frameworks aside entirely and simply follow your investigative instincts.

This chapter presents the major threat hunting frameworks that have emerged over the past decade. Rather than prescribing one "correct" approach, it aims to give you a comprehensive toolkit so you can choose - or create - the methodology that works best for your organization, team, and specific investigations. The best way to approach it is with Bruce Lee's quote in mind: "**Absorb what is useful, discard what is useless and add what is specifically your own"**.



## The PEAK Framework

PEAK (Prepare, Execute, Act, Knowledge) is a structured framework developed to provide clear phases and deliverables for threat hunting activities. It's particularly popular in organizations that want well-defined processes and clear accountability.

### The Four Phases

```
PEAK Framework Structure
═══════════════════════════════════════════════════════════

┌─────────────────────────────────────────────────────────┐
│                      PREPARE                            │
│  • Define scope and objectives                          │
│  • Identify data sources and tools                      │
│  • Develop hypotheses                                   │
│  • Plan investigation approach                          │
└────────────────────┬────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────┐
│                     EXECUTE                             │
│  • Collect and analyze data                             │
│  • Test hypotheses                                      │
│  • Follow evidence trails                               │
│  • Document findings as you go                          │
└────────────────────┬────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────┐
│                      ACT                                │
│  • Respond to identified threats                        │
│  • Implement remediation                                │
│  • Create detection rules                               │
│  • Update security controls                             │
└────────────────────┬────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────┐
│                   KNOWLEDGE                             │
│  • Document lessons learned                             │
│  • Share findings with stakeholders                     │
│  • Update playbooks and procedures                      │
│  • Generate new hunting hypotheses                      │
└────────────────────┬────────────────────────────────────┘
                     ↓
              (Return to PREPARE)
```

**Prepare Phase**: This is the most distinctive element of PEAK - the emphasis on preparation before investigation begins. Teams explicitly define what they're hunting for, why it matters, what success looks like, and what resources they'll need. This planning reduces wasted effort and ensures investigations are purposeful.

The Prepare phase includes scoping decisions: Which systems or user populations will you focus on? What time period will you examine? What data sources are available and relevant? It also includes hypothesis development and validation - ensuring your hypothesis is testable and meaningful before investing investigation time.

**Execute Phase**: This is where actual investigation occurs - querying data, analyzing results, following leads, and testing hypotheses. PEAK emphasizes disciplined execution: stay focused on your defined scope, document as you go, and periodically reassess whether you're making progress toward your objectives.

The Execute phase often iterates internally - initial queries lead to refined queries, early findings suggest new investigation paths, and hunters pivot based on evidence. But throughout, the Prepare phase decisions about scope and objectives provide guardrails that prevent endless rabbit holes.

**Act Phase**: When threats are discovered or security improvements are identified, the Act phase ensures appropriate action is taken. This might mean escalating to incident response, implementing remediation, creating detection rules, or updating security controls. PEAK emphasizes that hunting without action provides limited value - findings must translate to improved security posture.

Even when hunts find no threats, action is still required: documenting what was validated, updating baseline understandings, and potentially adjusting future hunting priorities based on what you learned.

**Knowledge Phase**: The final phase captures and shares knowledge gained from the hunt. This includes formal documentation, briefings to stakeholders, updates to hunting playbooks, and generation of new hypotheses for future hunts. PEAK views knowledge capture as essential, not optional - without it, organizational learning is lost and teams repeatedly hunt for the same things.

### Strengths and Ideal Use Cases

PEAK's structured approach provides several advantages:

**Clarity and Accountability**: Each phase has clear objectives and deliverables, making it easy to understand where you are in the process and what should happen next. This is particularly valuable for newer hunting teams or organizations that need to demonstrate systematic methodology to leadership.

**Resource Planning**: The Prepare phase helps estimate time and resource requirements before beginning investigation, enabling better capacity planning and prioritization across multiple hunting activities.

**Completeness**: The framework's explicit phases reduce the risk of skipping important steps - particularly knowledge capture and action implementation, which are sometimes neglected in less structured approaches.

**Teachability**: PEAK's clear structure makes it relatively easy to train new hunters or explain hunting methodology to stakeholders who aren't hunters themselves.

**PEAK works best when:**

- You have clear, specific hunting objectives defined in advance
- You're conducting planned, scheduled hunting activities rather than reactive investigations
- You need to demonstrate systematic methodology to leadership or auditors
- Your team benefits from structured process guidance
- You're conducting complex hunts requiring coordination across multiple team members

**It's less ideal when:**

- You need to respond rapidly to emerging threats without time for extensive preparation
- You're conducting exploratory hunting without clear objectives
- Your investigation scope needs to be highly fluid based on findings
- Overhead of formal phase documentation outweighs benefits

## The TaHiTI Model

TaHiTI (Targeted Hunting integrating Threat Intelligence) is a framework developed by David J. Bianco that explicitly bridges threat intelligence and threat hunting. It's particularly valuable for organizations with mature threat intelligence programs looking to operationalize that intelligence through hunting.

### The Intelligence-Hunting Connection

TaHiTI recognizes that effective hunting requires context about adversary behavior, but translating threat intelligence into actionable hunting is often challenging. Intelligence reports describe attacks abstractly or focus on specific malware samples, while hunters need concrete investigation approaches for their environments.

TaHiTI provides a structured methodology for:

1. Consuming threat intelligence about adversary TTPs
2. Translating those TTPs into environment-specific hunting hypotheses
3. Designing investigations that could detect those TTPs in your environment
4. Executing hunts and generating findings
5. Feeding findings back to intelligence teams to refine future collection

### The TaHiTI Process Flow

The framework operates through several connected activities:

**Intelligence Analysis**: Start with threat intelligence - whether from commercial feeds, ISACs, open source reporting, or internal intelligence teams. The key is understanding adversary behavior at the TTP level, not just indicators like IP addresses or file hashes.

**Contextualization**: Assess whether the intelligence is relevant to your environment. Not all threats apply to all organizations. An attack targeting Linux servers is less relevant if you're primarily Windows-based. Techniques requiring physical access are less relevant if you're concerned about remote adversaries.

**Hypothesis Generation**: Translate relevant intelligence into specific, testable hypotheses about your environment. If intelligence describes adversaries using WMI for lateral movement, your hypothesis becomes: "Are we seeing evidence of WMI-based lateral movement in our environment?"

**Analytics Development**: Design specific analytics or queries that could detect the TTPs described in intelligence. This is where you translate abstract adversary behavior into concrete data analysis. What log sources would show WMI lateral movement? What patterns would indicate malicious vs. legitimate use?

**Hunting Execution**: Run your analytics, investigate results, and either discover threats or validate their absence. This phase closely resembles the Execute phase in PEAK or the Investigation phase in the core hunting loop.

**Intelligence Feedback**: Findings from hunting feed back to intelligence teams. Did the intelligence accurately describe adversary behavior in real environments? Were there detection blind spots? What additional context would make the intelligence more actionable? This feedback loop improves future intelligence collection and analysis.

### Distinctive Elements

What makes TaHiTI distinctive is its explicit focus on the intelligence-operations pipeline. While other frameworks acknowledge the role of threat intelligence, TaHiTI makes it central - viewing hunting as the operational application of threat intelligence.

This focus brings several benefits:

**Prioritization**: Not all possible hunting activities are equally valuable. By starting with intelligence about active threats and adversary campaigns, you prioritize hunts most likely to discover real threats in your environment.

**Efficiency**: Rather than generating hypotheses from scratch, you leverage the research and analysis that intelligence teams have already conducted. This is particularly valuable for smaller organizations that can't maintain extensive research capabilities.

**Continuous Improvement**: The feedback loop means your intelligence consumption becomes increasingly relevant over time. Intelligence teams learn what kinds of information operations teams need, and operations teams get better at translating intelligence into action.

### When TaHiTI Fits Best

TaHiTI is ideal when:

- You have access to quality threat intelligence (commercial, ISAC, or internal)
- Your organization faces threats from known adversary groups with documented TTPs
- You want to demonstrate that security investments are addressing real, documented threats
- You have separate intelligence and operations teams that need structured interaction
- You operate in sectors frequently targeted by sophisticated adversaries

It's less applicable when:

- You lack access to relevant threat intelligence
- Your primary threats are opportunistic rather than targeted campaigns
- You're conducting exploratory hunting not driven by specific intelligence
- Your environment is unique enough that external intelligence has limited applicability

## Sqrrl's Hunting Loop

Sqrrl (later acquired by Amazon) developed one of the earliest explicit threat hunting frameworks and helped popularize threat hunting as a distinct discipline. Their hunting loop framework is notable for its simplicity and focus on continuous iteration.

### The Three-Phase Loop

Sqrrl's framework describes hunting as a continuous three-phase loop:

**Create Hypothesis**: Based on threat intelligence, environment knowledge, or investigation of anomalies, formulate a hunting hypothesis. Sqrrl emphasizes that hypotheses should be specific enough to guide investigation but flexible enough to evolve based on findings.

**Investigate via Tools and Techniques**: Use available security tools and data sources to search for evidence related to your hypothesis. This investigation phase is explicitly iterative - initial findings lead to refined queries, which lead to new findings, in a continuous process of discovery.

**Uncover New Patterns and TTPs**: Investigation reveals either threats (requiring immediate response) or new understanding of environment behavior. These patterns and insights inform future hypotheses, continuing the loop.

### Emphasis on Continuous Iteration

What distinguishes Sqrrl's framework is its explicit emphasis on the never-ending nature of hunting. There's no separate "conclusion" or "knowledge" phase because every hunt naturally flows into the next. Each discovered pattern immediately becomes input for new hypotheses.

This reflects Sqrrl's philosophy that hunting isn't a series of discrete projects but rather an ongoing operational capability. You're never "done" hunting - you're always in the loop, continuously cycling through hypothesis, investigation, and discovery.

### The Role of Automation and Tools

Sqrrl's framework also emphasizes the relationship between human hunting and automated detection. They advocate for:

**Hunt → Detect → Automate**: When hunters discover threats through manual investigation, detection engineers create automated rules to catch similar threats. These automated detections free hunters to focus on new, unknown threats rather than repeatedly hunting for the same things.

**Continuous Refinement**: As automated detections generate alerts and hunters investigate them, the investigation often reveals ways to improve detection rules - tuning them to reduce false positives or expand coverage. This creates a virtuous cycle of continuous improvement.

### Practical Application

In practice, Sqrrl's framework is often the most flexible and least prescriptive. It provides philosophical guidance rather than detailed process steps. Teams using Sqrrl's approach might:

- Maintain a running list of hypotheses in a shared document
- Conduct hunts opportunistically as time allows rather than on fixed schedules
- Move fluidly between different investigations as findings suggest new directions
- Emphasize speed and iteration over comprehensive documentation
- Treat hunting as deeply integrated with daily security operations rather than a separate function

This flexibility makes Sqrrl's framework popular with experienced hunting teams who don't need detailed process guidance but appreciate a simple mental model that captures hunting's essential nature.

## Other Notable Frameworks and Approaches

Beyond these major frameworks, several other methodologies deserve mention:

### The F3EAD Cycle (Military Intelligence Framework)

Adapted from military intelligence operations, F3EAD (Find, Fix, Finish, Exploit, Analyze, Disseminate) provides a more aggressive, military-flavored approach to hunting. While less commonly used in corporate environments, some organizations - particularly those with personnel from military or intelligence backgrounds - find its terminology and approach familiar.

**Find**: Identify indicators of threat activity **Fix**: Pinpoint the threat's location and characteristics **Finish**: Neutralize the threat (in cyber context, this means contain and eradicate) **Exploit**: Extract intelligence from the incident **Analyze**: Understand the broader implications **Disseminate**: Share findings with relevant stakeholders

F3EAD is particularly popular in organizations with offensive security operations or those conducting threat hunting in adversarial contexts where active disruption of threats is a primary goal.

### The Diamond Model Approach

Some organizations structure hunting around the Diamond Model of Intrusion Analysis, which views security events as having four components: Adversary, Infrastructure, Capability, and Victim. Hunting then focuses on discovering connections between these components.

This approach is particularly effective for advanced persistent threat investigations where understanding the full scope of adversary operations - their infrastructure, tools, and targeting - is crucial for complete remediation.

### The NIST Cybersecurity Framework Alignment

Some organizations align hunting activities explicitly with NIST Cybersecurity Framework functions (Identify, Protect, Detect, Respond, Recover). Hunting becomes a structured activity within the Detect function, with clear handoffs to Respond when threats are found.

This alignment is particularly valuable for organizations already using NIST CSF for security program organization, as it integrates hunting naturally into existing structure rather than treating it as a separate activity.

## Comparing and Choosing Frameworks

With multiple frameworks available, how do you choose? The honest answer is: you don't have to choose exclusively. Many mature hunting programs blend elements from multiple frameworks based on context.

```
Framework Selection Decision Tree
═══════════════════════════════════════════════════════════

Start: What's your context?
           │
           ↓
    ┌──────┴──────┐
    │             │
Planned Hunt   Reactive/
  │           Exploratory
  ↓              │
  │              ↓
  │         Sqrrl Loop
  │         (Flexible)
  ↓
Strong Intel?
  │
  ├─Yes─→ TaHiTI
  │       (Intel-driven)
  │
  └─No─→ PEAK
         (Structured)

Reality: Most teams blend approaches!
• Use PEAK for planned hunts requiring coordination
• Use TaHiTI when consuming threat intelligence
• Use Sqrrl for day-to-day opportunistic hunting
• Use Hypothesis-driven thinking throughout
```

### Selection Factors

Several factors influence which framework fits your needs:

**Team Experience**: Newer teams often benefit from more structured frameworks (PEAK, TaHiTI) that provide detailed guidance. Experienced teams may prefer more flexible approaches (Sqrrl, hypothesis-driven) that don't constrain their investigation.

**Organizational Culture**: Organizations that value structured processes, clear deliverables, and formal documentation gravitate toward PEAK. Organizations that value speed and flexibility prefer Sqrrl or hypothesis-driven approaches.

**Intelligence Maturity**: Organizations with mature threat intelligence programs and access to quality intelligence benefit significantly from TaHiTI's structured approach to operationalizing that intelligence.

**Hunting Maturity**: Early-stage hunting programs often start with more structured frameworks to establish systematic practices. As programs mature, they typically become more flexible and adaptive in their approach.

**Stakeholder Expectations**: If leadership or auditors want to see systematic methodology, structured frameworks like PEAK provide clear evidence of process. If stakeholders care primarily about outcomes, more flexible approaches may be appropriate.

### The Blended Approach

In practice, most successful hunting programs blend elements from multiple frameworks rather than rigidly following one. You might:

- Use TaHiTI principles when consuming threat intelligence to generate hypotheses
- Follow PEAK's Prepare phase when planning complex hunts requiring team coordination
- Apply Sqrrl's continuous loop philosophy for day-to-day hunting activities
- Use hypothesis-driven thinking throughout all investigations regardless of framework
- Adopt F3EAD terminology when working with military or intelligence backgrounds

This blended approach takes the best from each framework while remaining pragmatic about your specific needs and constraints. Frameworks should serve your operations, not constrain them.

## The Meta-Framework: Adaptive Methodology

Perhaps the most mature approach isn't choosing a specific framework but developing what might be called an "adaptive methodology" - the ability to recognize what any given situation requires and apply appropriate structure, rigor, or flexibility.

This meta-framework asks:

**What's the goal of this hunt?** Different objectives (respond to intelligence, explore new environment, validate detection coverage, investigate anomaly) suggest different approaches.

**What constraints exist?** Time pressure, stakeholder visibility, team coordination needs, and other constraints influence how structured or flexible your approach should be.

**What's the risk if we get it wrong?** High-stakes hunts (critical assets, recent breach indicators, sensitive investigations) deserve more rigor and documentation than routine coverage hunts.

**What resources are available?** Team size, tools, data sources, and time availability all influence which framework elements are practical.

Answering these questions helps you select appropriate rigor, structure, and approach for each specific hunt rather than treating all hunts identically.

## Avoiding Framework Dogmatism

The greatest risk with frameworks is dogmatism - insisting that a particular framework must be followed rigidly regardless of context. This leads to several problems:

**Process Over Outcomes**: Teams focus on following framework steps rather than actually finding threats or improving security posture. The framework becomes bureaucratic overhead rather than value-adding structure.

**Constrained Creativity**: Rigid framework adherence can prevent hunters from following where evidence leads. If your framework says "complete Prepare phase before Execute phase" but investigation findings suggest immediate action, following the framework strictly means worse outcomes.

**False Security**: Organizations sometimes believe that following a framework ensures effective hunting, regardless of team skill, data availability, or investigation quality. Frameworks don't make bad hunting good - they provide structure for skilled practitioners.

**Framework Wars**: Teams debate which framework is "best" rather than pragmatically applying whichever approach fits their needs. All mainstream hunting frameworks are sound; the question isn't which is best but which fits your context.

The solution is viewing frameworks as tools: useful, important, but not sacred. Learn multiple frameworks, understand their strengths and contexts, and then flexibly apply them as needed. The goal is effective hunting, not framework compliance.

## Developing Your Organization's Approach

As your hunting program matures, you'll likely develop your own methodology - influenced by established frameworks but customized for your environment, team, and organizational culture. This is not just acceptable but desirable. The best hunting methodologies are those that actually get used consistently and effectively.

When developing your approach, consider:

- What structure does your team actually need to be effective?
- What documentation do stakeholders require to have confidence in your work?
- How does hunting integrate with your SOC, incident response, and detection engineering?
- What tools and data sources do you have, and what workflows do they support?
- How do you ensure knowledge capture and continuous improvement?

Document your methodology clearly enough that new team members can learn it and stakeholders can understand it, but not so rigidly that it constrains investigation. Build in regular reviews where you assess whether your methodology is serving your needs and adjust as necessary.

Remember that methodology should evolve as your program matures, your team gains experience, and your organizational context changes. What works for a three-person hunting team will need adjustment when you scale to ten people. What works when hunting is new will need refinement after two years of operation.



