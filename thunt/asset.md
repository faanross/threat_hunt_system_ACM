# Asset Inventory and Environmental Knowledge

## The Foundation of "Normal"

Imagine being a detective investigating a crime scene in a house you've never visited. You notice a laptop on the kitchen counter, clothes scattered in the bedroom, and an open window on the second floor. Are these signs of a burglary, or is this simply how the residents live? Without knowing what "normal" looks like for this household, distinguishing evidence of crime from ordinary life becomes nearly impossible.

Threat hunters face the same fundamental challenge. When you discover a scheduled task running on a server, is it malicious persistence or legitimate automation? When you see authentication from an unusual location, is it account compromise or a traveling employee? When you observe large data transfers, is it exfiltration or routine backup? The answer depends entirely on understanding what's normal in your environment.

This is why asset management and environmental baselining aren't just good security practices - they're fundamental prerequisites for effective threat hunting. You cannot identify abnormal without first understanding normal. You cannot investigate suspicious activity on systems you don't know exist. You cannot distinguish malicious behavior from legitimate operations without context about how your environment functions.

Yet asset management and baselining are frequently neglected, treated as mundane administrative tasks rather than the critical foundation they represent. Organizations invest in sophisticated hunting tools and skilled personnel, then wonder why hunting produces overwhelming false positives or misses obvious threats. Usually, the root cause is simple: the hunters don't truly know their environment.

This chapter explores why environmental knowledge is so critical, what specific knowledge threat hunters need, and practical approaches to building and maintaining that knowledge. Most importantly, it addresses the uncomfortable truth that many organizations must confront: **you may need to pause hunting activities to build foundational environmental knowledge before effective hunting becomes possible**.


## The Principle: You Can Only Protect What You Know

This principle seems obvious yet is probably the most neglected in practice.  Consider these scenarios that occur regularly in organizations attempting threat hunting:

**Scenario 1: The Unknown Server** A hunter discovers suspicious network traffic from an IP address. Investigation reveals it's a server in the organization's data center. But nobody knows what this server does, who owns it, or what applications run on it. Without this context, determining whether the traffic is malicious or legitimate becomes guesswork.

**Scenario 2: The Undocumented Process** A hunter identifies an unusual process running on critical systems. Security teams spend hours investigating, only to eventually discover it's a legitimate monitoring agent that was deployed months ago but never documented. Time wasted, and even worse superficially similar processes might be dismissed as "probably legitimate" in the future, potentially missing actual threats.

**Scenario 3: The Mystery Account** A hunter observes authentication activity from a service account. The account name suggests it's related to backup operations, but investigation reveals no documentation about this account - who created it, what it's authorized to access, or what it should be doing. Every authentication event from this account now looks suspicious, but there's no baseline to compare against.

**Scenario 4: The Unexpected Communication** A hunter detects two systems communicating that "shouldn't" communicate. Incident response is initiated, impacting business operations. Later investigation reveals this communication is part of a new application deployment that IT implemented but never communicated to security. False alarm, wasted resources, and damaged credibility.

These scenarios share a common root cause: lack of environmental knowledge. The organization attempted threat hunting without first understanding what assets exist, what they do, who owns them, and how they normally behave. And as we saw, this lack almost always stems from either a lack of communication, and/or documentation. A solid security foundation is built on the "boring stuff".

Further, and as trite as it perhaps sounds - **clarity creates consistency**. If you want consistent results, you need clarity not only around your threat hunting strategy and techniques, but equally around what you are looking for. And at the risk of sounding repetitive - you cannot know what abnormal looks like if you do not know what normal looks like. 


## What Environmental Knowledge Do Hunters Need?

Effective threat hunting requires multiple layers of environmental knowledge:

### Asset Inventory: The Foundation

The most basic layer is knowing what assets exist in your environment:

**Systems and Devices**:

- All servers, workstations, network devices, mobile devices
- Operating systems, versions, patch levels
- Physical locations or network segments
- Hardware specifications and configurations

**Applications and Services**:

- What applications run on which systems
- Application versions and configurations
- Inter-application dependencies
- Service accounts associated with applications

**Network Architecture**:

- Network segments and their purposes
- Firewall rules and access controls
- Internal routing and switching topology
- External connections and partner networks

**User Accounts**:

- Human user accounts and their purposes
- Service accounts and what services use them
- Privileged accounts and who should have them
- Account lifecycle (creation, modification, deletion)

**Data Assets**:

- Critical data repositories and their locations
- Data classification and sensitivity
- Data flows between systems
- Backup and recovery systems

Without comprehensive asset inventory, you can't even enumerate what you're supposed to be protecting, let alone hunt for threats effectively.


### Ownership and Purpose: The Context

Knowing assets exist isn't sufficient - you need context:

**Business Ownership**:

- Who owns each asset from business perspective
- What business function does it support
- Who to contact with questions or concerns
- Who authorizes changes to the asset

**Technical Ownership**:

- Who maintains the asset technically
- Who has administrative access and why
- Who troubleshoots when problems occur
- Who performs routine maintenance

**Purpose and Function**:

- What is the asset's intended purpose
- What users or services should access it
- What external connections should it make
- What scheduled operations should occur

This context transforms raw asset data into actionable intelligence. When you observe suspicious activity, you can quickly assess "Should this system be doing this? Should this user have access? Is this communication expected?"

### Configuration Baselines: The Standard

Understanding standard configurations helps identify anomalous configurations:

**System Configuration**:

- Standard operating system configurations by role (workstation, server type, etc.)
- Approved software installations
- Standard security settings and hardening
- Expected services and startup items

**Security Configuration**:

- Firewall rules and expected network access
- Authentication mechanisms and requirements
- Logging configurations and retention
- Security tool installations and settings

**Application Configuration**:

- Standard application settings
- Expected file locations and directory structures
- Configuration files and their standard contents
- Integration points with other systems

Deviations from configuration baselines often indicate compromise - unauthorized software, disabled security tools, modified logging, unusual startup items.

### Behavioral Baselines: The Patterns

Perhaps most critical for threat hunting is understanding normal behavioral patterns:

**Network Behavior**:

- Which systems normally communicate with which destinations
- Typical traffic volumes and patterns
- Expected protocols and ports usage
- Normal external connections (cloud services, partners, etc.)

**Authentication Patterns**:

- Normal login times for users
- Expected login locations (offices, VPN, etc.)
- Typical authentication frequency
- Standard privilege escalation patterns for administrators

**Process Execution Patterns**:

- Normal processes for different system types
- Expected parent-child process relationships
- Standard command-line arguments for legitimate tools
- Typical PowerShell, script, and automation usage

**User Behavior**:

- Typical file access patterns
- Normal application usage
- Expected data movement and transfers
- Standard working hours and activity levels

**System Behavior**:

- Normal resource utilization (CPU, memory, disk)
- Expected disk I/O patterns
- Typical network bandwidth usage
- Standard scheduled task execution

These behavioural baselines enable anomaly detection - the core of behavioral threat hunting. Without baselines, everything looks potentially suspicious, or nothing does.

```
Layers of Environmental Knowledge
═══════════════════════════════════════════════════════════

             ┌─────────────────────────────┐
             │   BEHAVIORAL BASELINES      │
             │   "How things normally act" │
             └──────────────┬──────────────┘
                            │
             ┌──────────────▼──────────────┐
             │  CONFIGURATION BASELINES    │
             │  "How things should be set" │
             └──────────────┬──────────────┘
                            │
             ┌──────────────▼──────────────┐
             │   OWNERSHIP & PURPOSE       │
             │   "Why things exist"        │
             └──────────────┬──────────────┘
                            │
             ┌──────────────▼──────────────┐
             │     ASSET INVENTORY         │
             │     "What exists"           │
             └─────────────────────────────┘

Each layer builds on the previous:
• Can't baseline behavior without knowing assets
• Can't understand purpose without inventory
• Can't detect anomalies without behavioral baselines

Missing any layer undermines threat hunting effectiveness
```



## The Cost of Environmental Ignorance

What happens when organizations attempt threat hunting without adequate environmental knowledge?

### False Positive Overload

Without baselines, legitimate activities look suspicious. The hunter identifies:

- A scheduled task (might be malicious persistence, might be legitimate automation)
- Unusual authentication (might be compromise, might be traveling employee)
- Large data transfer (might be exfiltration, might be backup)
- Unfamiliar process (might be malware, might be legitimate software)

Each requires investigation to determine legitimacy. Without environmental knowledge, investigation is time-consuming - contacting system owners, researching purposes, checking with IT. Multiply this by hundreds of potential anomalies, and hunters drown in false positives, unable to distinguish signal from noise.

Organizations experiencing this often respond by raising investigation thresholds, dismissing more potential threats without investigation. This reduces false positive burden but increases false negative rate - real threats are dismissed as "probably legitimate" to manage workload.

### Missed Real Threats

The opposite problem also occurs: without baselines, real threats blend into noise. If you don't know what normal authentication looks like, pass-the-hash attacks look like routine authentications. If you don't know what processes typically run, malware execution blends with legitimate processes. If you don't know what network traffic is normal, command and control communications go unnoticed.

Sophisticated adversaries specifically design activity to blend with normal operations. This technique - blending in with legitimate activity - is highly effective when defenders don't actually know what legitimate looks like.

### Investigation Inefficiency

Even when hunters correctly identify suspicious activity, investigation becomes inefficient without environmental knowledge:

- Significant time spent determining basic context: "What is this system? Who owns it? What should it be doing?"
- Unable to quickly assess severity or urgency without knowing asset criticality
- Difficulty scoping investigations without understanding asset relationships
- Challenges determining appropriate response without knowing business impact

What should be quick triage - "This is suspicious and impacts critical assets, escalate immediately" or "This is anomalous but low risk, document and monitor" - becomes prolonged investigation.

### Eroded Credibility

When hunting produces excessive false positives or asks basic questions that should be answerable ("What does this server do?"), organizational credibility suffers:

- Leadership questions hunting program value
- IT teams become frustrated with interruptions
- Business units lose confidence in security team
- Future security initiatives face skepticism

Building credibility requires demonstrating competence. Competence requires environmental knowledge.

## Building Asset Inventory

How do you build comprehensive asset inventory if you don't have one?

### Discovery Methods

Multiple approaches exist for discovering assets:

**Network Scanning**:

- Active scanning (nmap, vulnerability scanners) discovers systems responding to network probes
- Passive monitoring (analyzing network traffic) identifies active systems without sending probes
- Advantages: Can discover unauthorized/forgotten systems
- Disadvantages: May miss systems not on network during scans, may not identify offline systems

**Integration with IT Systems**:

- Configuration Management Databases (CMDB) often contain asset information
- Endpoint management systems (SCCM, Jamf, etc.) track managed endpoints
- Cloud management consoles (AWS, Azure, GCP) inventory cloud resources
- Virtual infrastructure management (VMware vCenter, Hyper-V) tracks VMs
- Advantages: Authoritative sources, often include ownership and purpose
- Disadvantages: Only track managed/known systems, may be outdated

**Agent-Based Discovery**:

- Endpoint agents (EDR, asset management) report inventory from systems
- Advantages: Detailed system information, continuous updates
- Disadvantages: Only work on systems where agents can be deployed

**Authentication and Access Logs**:

- Analyze authentication logs to identify active user accounts
- Review access control lists to identify systems and resources
- Advantages: Reveals actual usage patterns
- Disadvantages: Only shows systems/accounts that authenticate

**Financial and Procurement Records**:

- Purchase orders and invoices reveal acquired assets
- Software licensing records show deployed applications
- Advantages: High-level organizational view
- Disadvantages: May not reflect actual deployment, may be outdated


### Effective Inventory Requires Multiple Sources

No single method provides complete inventory. Effective approaches combine multiple sources:

1. **Start with authoritative sources**: CMDB, asset management systems, cloud consoles
2. **Augment with discovery**: Network scanning, traffic analysis to find unmanaged assets
3. **Validate with agents**: Deploy monitoring where possible for detailed information
4. **Cross-reference**: Identify discrepancies between sources (systems in CMDB but not on network, systems on network not in CMDB)
5. **Continuous update**: Inventory is never "done" - it requires ongoing maintenance

### Critical Inventory Attributes

What information should asset inventory include?

**Minimum Required**:

- Asset identifier (hostname, IP address, etc.)
- Asset type (server, workstation, network device, etc.)
- Operating system and version
- Network location (segment, VLAN, physical location)
- Status (active, offline, decommissioned)

**Highly Valuable**:

- Business owner and technical owner contact information
- Purpose and function description
- Criticality/risk classification
- Dependencies (what this asset depends on, what depends on it)
- Installed applications and versions
- Access requirements (who should access this asset)

**Useful for Context**:

- Deployment date
- Last known configuration change date
- Maintenance schedule
- Compliance requirements
- Physical location
- Hardware specifications

Start with minimum required, progressively add valuable attributes as inventory matures.

### Maintaining Inventory Accuracy

Inventory degrades without maintenance. Systems are deployed, decommissioned, reconfigured. Strategies for maintaining accuracy:

**Automated Updates**:

- Continuous scanning or agent reporting updates inventory automatically
- Integration with deployment systems (adds assets automatically when deployed)
- Lifecycle hooks (removes assets when decommissioned)

**Periodic Reconciliation**:

- Regular reviews comparing inventory against actual environment
- Scheduled audits identifying discrepancies
- Cleanup of stale or inaccurate entries

**Change Management Integration**:

- Asset inventory updates as part of change management process
- System deployments require inventory registration
- Decommissions require inventory removal

**Ownership Accountability**:

- Asset owners responsible for maintaining accurate information
- Periodic owner confirmation of asset accuracy
- Escalation processes for outdated information


## Establishing Behavioural Baselines

Asset inventory tells you what exists. Behavioural baselines tell you what's normal.

### The Baseline Development Challenge

Establishing baselines sounds straightforward but presents challenges:

**Complexity**: Modern environments are complex - thousands of systems, millions of processes, endless variations in legitimate behavior. Baselining everything comprehensively is unrealistic.

**Dynamics**: Environments change constantly - new applications deploy, users change roles, business processes evolve. Baselines require continuous updating or they become obsolete.

**Legitimate Variability**: Even for single users or systems, behavior varies legitimately. Users travel, work unusual hours, take on new responsibilities. Systems experience varying loads, maintenance activities, updates.

**Lack of History**: New hunting programs may lack historical data to establish baselines. You can't analyze patterns you never collected data about.

The solution isn't perfect comprehensive baselines - it's strategically focused baselining that covers most important patterns while accepting some uncertainty. Be guided by the idea that you should not let "perfect be the enemy of good enough".

### Prioritized Baselining Approach

Rather than attempting comprehensive baselining, prioritize:

**Start with Critical Assets**:

- Baseline behavior for crown jewel systems first
- Critical servers, domain controllers, data repositories
- Privileged user accounts
- Systems storing or processing sensitive data

**Focus on Security-Relevant Behaviors**:

- Authentication patterns (most attacks involve authentication)
- Network communications (C2, lateral movement, exfiltration)
- Process execution (malware, living-off-land attacks)
- File operations on sensitive systems

**Use Statistical Approaches**:

- Identify typical ranges rather than exact values
- Focus on outliers and extreme deviations
- Accept that baselines will evolve

**Leverage Existing Knowledge**:

- Interview system owners about expected behaviour
- Document known legitimate patterns
- Start with explicit allowlists for well-understood systems

### Practical Baselining Techniques

Several practical approaches establish behavioural baselines:

**Statistical Profiling**:

- Collect data over time (30-90 days minimum)
- Calculate statistical measures (mean, median, standard deviation)
- Identify patterns (daily cycles, weekly patterns, seasonal variations)
- Define "normal" as within statistical ranges (e.g., within 2 standard deviations)
- Flag outliers for investigation

Example: Authentication patterns by user

- Average logins per day: 3.2 (±1.5)
- Typical login times: 7:00 AM - 6:00 PM weekdays
- Common locations: Office A (80%), Office B (15%), Home VPN (5%)
- Unusual: Logins outside hours, from unexpected locations, excessive frequency

**Communication Mapping**:

- Document which systems communicate with which destinations
- Identify legitimate internal system-to-system communications
- Catalog approved external destinations (cloud services, partners)
- Create expected communication matrix

This enables lateral movement detection (unexpected internal communications) and C2/exfiltration detection (unexpected external communications).

**Process Allowlisting/Baselining**:

- Catalog processes that run on different system types
- Document expected parent-child relationships
- Record typical command-line arguments
- Note legitimate PowerShell and script usage

Deviations indicate potential compromise - unusual processes, abnormal parentage, suspicious commands.

**User Behavior Profiling**:

- Document typical file access patterns by user role
- Baseline application usage
- Profile data access (what users access what data)
- Track typical work patterns

Deviations suggest account compromise or insider threat - unusual file access, application misuse, data exfiltration patterns.

### The Progressive Baselining Approach

Baselining is iterative, not one-time:

**Phase 1: Initial Baseline (Months 1-3)**:

- Focus on highest-priority systems and behaviors
- Use coarse-grained baselines (broad ranges, simple patterns)
- Document obvious normal patterns
- Identify and investigate major outliers

**Phase 2: Refinement (Months 4-6)**:

- Refine baselines based on initial hunting experience
- Add more detailed behavioral patterns
- Expand coverage to additional systems
- Reduce false positive rates through tuning

**Phase 3: Expansion (Months 7-12)**:

- Expand baselining to broader environment
- Develop more sophisticated behavioral profiles
- Integrate machine learning for pattern recognition
- Automate baseline updates

**Phase 4: Maturity (Year 2+)**:

- Comprehensive behavioral understanding
- Automated baseline maintenance
- Predictive modeling of normal behavior
- Minimal manual baseline tuning required

Accept that early hunting will have higher false positive rates as baselines mature. This is normal and expected - don't let it discourage investment in hunting.


## Documentation and Knowledge Management

Environmental knowledge must be documented and accessible to be useful:

### What to Document

**Asset Information**:

- Inventory databases with all asset attributes
- Network diagrams showing system relationships
- Application architecture documentation
- Data flow diagrams

**Baseline Information**:

- Statistical profiles for key behaviours
- Allowlists of expected processes, communications, etc.
- Known legitimate patterns and exceptions
- Historical trends and patterns

**Operational Context**:

- Maintenance windows and schedules
- Change control history
- Known issues and workarounds
- Business process descriptions

**Investigation History**:

- Past hunting findings (threats and false positives)
- Investigation notes and outcomes
- Lessons learned and techniques discovered
- Contacts and escalation paths

### Documentation Formats

Different knowledge types suit different formats:

**Structured Databases**:

- Asset inventory
- Configuration management databases
- Baseline statistical data

**Wiki or Knowledge Base**:

- System purposes and ownership
- Investigation procedures
- Known legitimate patterns
- Troubleshooting guides

**Diagrams and Visualizations**:

- Network architecture diagrams
- Data flow visualizations
- System dependency maps
- Communication relationship graphs

**Runbooks and Playbooks**:

- Hunting procedures
- Investigation workflows
- Escalation procedures
- Response playbooks

### Making Knowledge Accessible

Documentation is worthless if hunters can't find it when needed:

- **Centralized Repository**: Single source of truth rather than scattered documentation
- **Search Functionality**: Keyword search across all documentation 
- **Regular Updates**: Scheduled review cycles to keep information current 
- **Version Control**: Track documentation changes over time 
- **Role-Based Access**: Appropriate access controls while ensuring hunting team has full access 
- **Integration with Tools**: Link documentation from hunting tools where context is needed

## Practical Strategies for Building Environmental Knowledge

How do organizations with poor environmental knowledge build it while starting threat hunting?

### Strategy 1: Hunt to Learn

Use hunting activities themselves to build environmental knowledge:

**Document as You Hunt**:

- Every hunting investigation teaches something about environment
- Document systems encountered, their purposes, normal behaviors
- Record false positives and what made them legitimate
- Build knowledge progressively through hunting activities

**Collaborative Investigation**:

- Partner with system owners during investigations
- Learn about legitimate uses while investigating anomalies
- Document conversations and insights gained
- Build relationships that enable future quick context

**Systematic Coverage**:

- Deliberately hunt across different environment areas over time
- Each area hunted improves knowledge of that area
- Progressive coverage builds comprehensive understanding
- Document findings and baselines as you go


### Strategy 2: Focused Baseline Campaigns

Conduct deliberate campaigns to baseline specific areas:

**Pick Priority Areas**:

- Critical systems
- High-risk user populations
- Important business processes

**Intensive Data Collection**:

- Collect detailed data over 30-90 days
- Analyze patterns and establish baselines
- Document findings thoroughly

**Progressive Expansion**:

- Complete one area before moving to next
- Build comprehensive knowledge of priority areas
- Expand to less critical areas over time

### Strategy 3: Leverage Existing Documentation

Mine existing organizational knowledge:

**IT Documentation**:

- Network diagrams
- System documentation
- Runbooks and procedures
- Configuration standards

**Business Documentation**:

- Business process descriptions
- Organizational charts
- Application portfolios
- Vendor contracts

**Operational Data**:

- Ticketing system histories
- Change management records
- Incident reports
- Audit findings

Much knowledge exists but may be scattered or outdated. Consolidate and validate it.

### Strategy 4: Collaboration and Interviews

Human knowledge often exceeds documented knowledge:

**System Owner Interviews**:

- What does this system do?
- What's normal behavior?
- Who accesses it and why?
- What external connections are legitimate?

**Administrator Collaboration**:

- What maintenance activities occur?
- What legitimate tools and scripts run?
- What scheduled operations happen?
- What variations occur and why?

**Business Stakeholder Input**:

- What business processes use systems?
- What data flows are expected?
- What seasonal variations occur?
- What change plans exist?

Document these conversations - tribal knowledge becomes organizational knowledge.

### Strategy 5: Start Narrow, Expand Progressively

Rather than attempting comprehensive environmental knowledge immediately:

**Phase 1: Critical Systems Only**

- Hunt only on crown jewel systems
- Build deep knowledge of these systems
- Establish comprehensive baselines for critical assets
- Demonstrate value on limited scope

**Phase 2: Expand to Important Systems**

- Add high-value but not critical systems
- Leverage techniques learned from critical systems
- Continue building progressive knowledge

**Phase 3: Broader Environment**

- Expand hunting across general environment
- Accept higher false positive rates on less-known systems
- Continue progressive learning and documentation

This approach delivers value early (finding threats on critical systems) while building knowledge progressively.

## The Relationship Between Hunting and Environmental Knowledge

Environmental knowledge and threat hunting have symbiotic relationship:

**Hunting Requires Knowledge**:

- Baselines enable anomaly detection
- Context enables investigation efficiency
- Understanding enables distinguishing threats from legitimate activity

**Hunting Builds Knowledge**:

- Investigations teach about environment
- False positives reveal legitimate patterns
- Coverage builds comprehensive understanding
- Documentation captures learning

This creates virtuous cycle:

1. Initial knowledge enables initial hunting
2. Hunting discovers gaps in knowledge
3. Gaps are filled through investigation and documentation
4. Improved knowledge enables more effective hunting
5. Repeat

Organizations shouldn't delay hunting until knowledge is perfect - but should recognize that early hunting investments partly build foundational knowledge.

## When to Pause and Build Foundations

Some organizations discover their environmental knowledge is so poor that hunting produces minimal value. When should you pause hunting to build foundations?

**Indicators That Foundations Are Insufficient**:

- False positive rate exceeds 90%
- Most investigations end with "we don't know what this system is"
- Hunters spend more time gathering context than actually hunting
- No ability to distinguish normal from abnormal
- Investigations consistently lack necessary information

**The Hard Decision**: If foundations are truly insufficient, pausing hunting activities to build asset inventory and baselines may be more productive than continuing ineffective hunting. This is difficult - it means admitting you're not ready and delaying promised capabilities.

**The Alternative Approach**: Rather than completely pausing, dramatically narrow hunting scope:

- Hunt only on best-understood systems
- Focus on clearest behavioral indicators
- Accept slower progress and learning curve
- Build foundations while hunting limited scope

Most organizations can pursue this middle path - hunting on limited scope while building broader foundations.

## Measuring Environmental Knowledge Maturity

How do you assess whether environmental knowledge is sufficient for effective hunting?

**Asset Inventory Maturity**:

- Can you enumerate all assets in scope?
- Do you know purpose and ownership for each?
- Is inventory kept current automatically?
- Can you quickly get context for any asset encountered?

**Baseline Maturity**:

- Do baselines exist for critical behaviors?
- Can you distinguish normal from abnormal?
- Are baselines updated as environment evolves?
- Do false positive rates decline over time?

**Documentation Maturity**:

- Is knowledge documented and accessible?
- Can hunters quickly find needed information?
- Is documentation current and accurate?
- Does documentation grow through hunting activities?

**Operational Maturity**:

- Can you quickly assess investigation priority?
- Can you efficiently scope incidents?
- Do you have appropriate contacts and escalation paths?
- Does hunting efficiency improve over time?

Honest self-assessment against these criteria reveals readiness for effective hunting.


