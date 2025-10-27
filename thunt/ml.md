
# Machine Learning and AI in Threat Hunting

## The Promise and the Reality

The marketing materials are seductive: "AI-powered automated threat hunting detects unknown threats in real-time! Machine learning eliminates the need for manual investigation!" Yet organizations that deploy these solutions often discover a jarring disconnect. The algorithms generate thousands of alerts demanding human investigation. The "automated hunting" still requires analysts to separate malicious activity from benign anomalies. The promised efficiency gains evaporate because machines can't replicate the contextual judgment, creative thinking, and investigative intuition that define effective threat hunting.

This gap between promise and reality has pushed organizations toward two extremes. Some over-invest in AI solutions expecting them to replace human hunters, only to discover they've automated data processing but not actual hunting. Others reject AI entirely, dismissing it as hype that distracts from proven manual techniques.

Both extremes miss the optimal position: machine learning excels at what machines do well - processing massive data volumes, identifying statistical patterns, flagging anomalies at scale. Humans excel at tasks requiring judgment, context, creativity, and adaptation. The question isn't whether AI can replace human hunters, but how to forge an effective partnership where each amplifies the other's strengths.



## What Machine Learning Actually Does

Before exploring applications, we need clarity about what machine learning means in practice. ML systems analyze data to identify patterns, make predictions, or classify activities without being explicitly programmed for every scenario. In security operations, this typically manifests in three primary approaches:

**Supervised learning** trains on labeled data - examples marked as "threat" or "benign" - then classifies new data based on learned patterns. Training a model with known malware and legitimate software to classify unknown files exemplifies this approach.

**Unsupervised learning** finds patterns in unlabeled data without predefined categories, identifying clusters or anomalies. This might cluster network traffic to reveal unusual communication patterns without predefining what "unusual" means.

**Anomaly detection** identifies data points that deviate significantly from normal patterns. This could detect authentication attempts that differ statistically from typical user behaviour or process executions that don't match system baselines.


Crucially, ML doesn't "understand" security the way humans do. It identifies statistical patterns without comprehending meaning or context. It's not autonomous - it requires training data, tuning, validation, and ongoing oversight. It's not perfect - false positives and false negatives are inevitable. And some techniques, particularly deep learning, are "black boxes" that make predictions without explaining their reasoning.

Understanding these realities positions us for the plateau of productivity, where AI provides genuine value for specific tasks when properly implemented and combined with human analysis.  



## Implementation Approaches: The Practical Landscape

Organizations face three primary paths for deploying ML in threat hunting, each with distinct trade-offs that align with different operational realities.

**Commercial platforms with vendor-trained models** represent the most common approach. EDR, NDR, and UEBA vendors like CrowdStrike, Darktrace, or Exabeam provide solutions with pre-trained models built from threat data across their customer base. You deploy the platform, it begins baselining your environment, and within weeks you're receiving ML-generated alerts.

The advantages are speed and expertise: immediate deployment without ML expertise, models trained on broad threat landscapes beyond your visibility, continuous vendor updates as threats evolve, and integrated tooling that connects detection to response. The trade-offs involve control and customization: limited ability to modify core algorithms, potential misalignment with your specific environment or threats, vendor lock-in for both technology and threat intelligence, and costs that scale with deployment size.

This path suits most organizations. Unless you have specific requirements that commercial tools can't address or significant ML expertise in-house, vendor platforms provide the fastest route to value.

**Fine-tuning pre-trained models** offers a middle ground for organizations with some ML capability. You start with foundation models - whether security-specific models from research communities or general-purpose models like transformer architectures - and adapt them to your environment. This might mean taking an open-source malware classification model and retraining it on samples relevant to your industry, or adapting a network anomaly detection model to your traffic patterns.

The advantages combine breadth and specificity: leverage training from broader datasets than you could create alone, customize for your threat landscape and environment, maintain more control over model behaviour, and potentially reduce costs compared to commercial platforms. The challenges involve capability and maintenance: requires ML expertise to fine-tune effectively, demands quality labeled data from your environment, creates ongoing maintenance burden as you must update models, and still depends on the original model's architecture and limitations.

This approach fits organizations with ML talent who face unique requirements - specific adversaries, unusual environments, or compliance constraints - that generic commercial tools don't address well.

**Fully custom models trained locally** represents the highest-effort path. Your team builds models from scratch, trains them entirely on your data, and maintains complete control over algorithms, features, and deployment. You might develop custom behavioural analytics specifically for your organization's unique application stack or build detection models for proprietary systems that commercial tools don't support.

The advantages are control and specificity: complete customization for your exact requirements, full transparency into model operation and decision logic, no vendor dependencies or data sharing, and ability to rapidly adapt to emerging threats specific to your organization. The costs are substantial: requires dedicated ML expertise and infrastructure, demands extensive high-quality training data from your environment, creates significant ongoing operational burden, and involves longer time-to-value than other approaches.

This path makes sense only for large organizations with dedicated ML teams, highly specialized environments, or specific regulatory requirements that preclude commercial or shared models. For most organizations, the cost-benefit doesn't justify fully custom development.

**The practical reality** for most threat hunting teams combines approaches: commercial platforms handle core detection and analytics, fine-tuning addresses specific gaps or enhances commercial tools, and custom models fill narrow niches where nothing else works. Start with commercial solutions for baseline capability, then selectively invest in customization where it delivers clear value beyond what vendors provide.

The goal isn't purity of approach - it's effective threat detection. Use the simplest implementation that solves your problem.



## Where ML Delivers Real Value

Machine learning provides measurable value in three primary areas that align with its genuine capabilities.

### Anomaly Detection at Scale

ML excels at processing massive datasets to identify statistical outliers that would be impractical for humans to find manually. Modern NDR platforms like Darktrace and ExtraHop leverage unsupervised learning to baseline network traffic patterns across thousands of endpoints, flagging unusual volumes, destinations, protocols, or beaconing behaviours consistent with command-and-control communications. Authentication monitoring gains precision: profiling typical login times, locations, and patterns enables detection of impossible travel scenarios or credential stuffing attempts. EDR solutions such as CrowdStrike and SentinelOne apply anomaly detection to process execution, baselining typical processes to identify unusual parent-child relationships or unexpected binary executions.


The value for hunters is strategic: rather than manually searching entire datasets for anomalies, they investigate ML-flagged outliers. This transforms an impossible task into a manageable one.

The limitations are significant but addressable. High false positive rates mean many anomalies are benign. Accurate baselines are essential - garbage in, garbage out. Sophisticated adversaries craft activity to appear normal. And ML cannot distinguish malicious from benign anomalies without human investigation. But these limitations don't negate the value, they clarify ML's role as hypothesis generator rather than verdict deliverer.


### Behavioural Profiling

ML builds sophisticated behavioural profiles that capture normal patterns more comprehensively than human-defined rules. UEBA platforms like Microsoft Defender for Identity and Exabeam profile each user's typical applications, file access, and authentication patterns, then identify significant changes that might indicate account compromise or insider threats. Entity behaviour analytics profiles system behaviours - typical processes, network connections, resource usage - flagging inconsistencies that suggest compromise. Application behaviour profiling learns typical communication patterns and identifies exploitation indicators.

The critical advantage is context-aware alerting. Rather than generic "this activity is suspicious" flags, behavioural profiling says "this specific user or system is behaving uncharacteristically." This context dramatically reduces false positives and prioritizes investigations.

The trade-offs are temporal and adversarial. Building accurate profiles requires significant training periods. Legitimate behaviour changes generate false alerts until profiles update. Sophisticated adversaries can operate within behavioural norms. But for detecting most threats and prioritizing investigative efforts, behavioural profiling provides substantial value.

### Pattern Recognition and Classification

ML identifies complex patterns in data that indicate known threat categories. Malware classification analyzes file characteristics to categorize threats and identify families. Phishing detection examines email content, metadata, and links to flag suspicious messages at scale. Platforms like Vectra AI apply pattern recognition to identify MITRE ATT&CK techniques in telemetry data - lateral movement, credential dumping, exfiltration patterns - accelerating hypothesis testing during investigations.

This accelerates triage and investigation. Rather than manually analyzing every suspicious file or email, hunters investigate ML-classified items requiring deeper analysis. Rather than searching logs manually for specific TTPs, they query ML-tagged events.

## Making ML Work: Implementation Realities

The gap between ML's potential and realized value typically stems from implementation failures rather than technology limitations. Success requires deliberate approaches to common challenges.

**Start with clear problems, not technology.** Identify specific hunting challenges where ML's strengths apply: "We can't manually analyze authentication logs for anomalies at our scale" or "We need to prioritize which systems show signs of compromise." Deploy ML to address these specific problems rather than hoping it will generically improve hunting. As Dave Hoelzer of SANS has emphasized in threat hunting training, technology should solve defined operational problems, not dictate hunting methodology.



**Accept that tuning is the real work.** ML deployment isn't a one-time installation. Initial configuration establishes baselines and parameters. Continuous refinement reduces false positives by excluding known benign anomalies, adjusts sensitivity based on results, incorporates analyst feedback, and updates models as environments evolve. Organizations that underinvest in tuning generate analyst fatigue as teams ignore noisy ML outputs.

**Maintain human primacy through human-in-the-loop design.** ML should surface findings for human review, not make autonomous decisions. Effective implementations provide rich context with ML outputs - not just scores, but relevant details enabling informed investigation. Humans retain ability to override or modify ML outputs, and the system includes transparency about how ML reached conclusions. This isn't just reviewing outputs - it's continuous learning where humans improve ML and ML enhances human effectiveness.

**Measure effectiveness honestly.** Track whether ML actually improves hunting: time saved through prioritization, threats found that were ML-flagged versus discovered through other means, false positive trends over time, analyst satisfaction, and detection coverage improvements. If ML requires more investigation time than it saves, generates high false positive rates that don't improve, or leads analysts to routinely ignore outputs, reassess implementation or reconsider whether ML fits your use cases.


## Common Pitfalls

The path to effective ML implementation is littered with predictable failures. **Over-reliance** on ML reduces human vigilance, assuming algorithms will catch everything - treat ML as assistant, not replacement, maintaining human-driven hunting independent of ML. **Under-investment in tuning** expects perfect immediate performance - budget significant time and resources for initial tuning and ongoing refinement. **Black box acceptance** takes ML outputs on faith without understanding reasoning - insist on explainable AI and demand transparency about decision logic. **Ignoring feedback loops** means ML never improves from analyst findings - implement systematic mechanisms where investigation results refine models. **Vendor hype belief** trusts marketing about "fully automated threat hunting" - evaluate capabilities realistically through proof-of-concept testing with your data. **Static models** train once but never update as threats evolve - implement continuous retraining as an ongoing operational requirement.


## The Future Landscape

Several emerging applications show promise while maintaining realistic expectations. Natural language processing is improving for automatically analyzing threat intelligence reports, extracting TTPs, and generating hunting hypotheses - though human validation remains essential. Adversarial machine learning research aims to understand how adversaries evade ML-based detection, potentially making systems more robust. Automated investigation workflows could suggest next investigation steps based on current findings, accelerating investigations while maintaining human control. Explainable AI developments aim to make ML reasoning transparent rather than opaque, increasing trust and understanding.

These developments may enhance ML's hunting role, but the fundamental need for human judgment, creativity, and context will persist.


## The Partnership Paradigm

"Automated threat hunting" is marketing fiction. What vendors offer is advanced detection and analytics - valuable, but distinct from hunting. The creative hypothesis generation, contextual judgment, adaptive investigation, and synthesis of insights that define hunting remain inherently human activities.

The path forward is human-AI partnership where each does what it does best: machines handle data processing and pattern recognition at scale, humans provide judgment, creativity, and context. Organizations that maintain this balance - using AI as a powerful assistant while preserving essential human elements - build the most effective hunting programs.

The goal isn't automation replacing hunters. It's augmentation making hunters more effective. When AI and humans work in true partnership, threat hunting capabilities exceed what either could achieve alone. ML processes the haystack; humans recognize the needle.

