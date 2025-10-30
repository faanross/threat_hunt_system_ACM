# Why Perfect Coverage is the Enemy of Good Security

A new threat hunting program at a 10,000-employee organization faces a daunting reality: 15,000 endpoints, 2,000 servers, hundreds of applications, cloud infrastructure across three providers, remote users worldwide, millions of daily authentication events, and terabytes of logs. The hunting team consists of three people.

The math is brutal. Even with larger teams, comprehensive coverage remains fantasy. Yet the organization expects the hunting program to provide value, demonstrate effectiveness, and justify continued investment - ideally within months, not years.

This fundamental dilemma defines modern threat hunting: resources are finite while potential scope is effectively infinite. Every hour spent hunting one area is an hour not spent on others. How do you allocate scarce hunting resources to maximize security value?

The answer lies not in working harder or faster, but in embracing a counterintuitive truth: attempting to hunt everything means hunting nothing effectively. Success requires the discipline to prioritize ruthlessly, accepting that skewed coverage of critical areas provides more value than attempting comprehensive coverage of everything.


## Why Most Hunting Programs Fail Before They Start

The pressure on new hunting programs is immense. Leadership expects visible results within 3-6 months. Other security investments compete for the same budget. Early failures can doom a program before it finds its footing.

Meanwhile, the scope feels overwhelming. Three hunters might conduct 10 to 25 substantial hunts each monthly. But there are over 200 MITRE ATT&CK techniques worth investigating, thousands of systems, dozens of threat actors whose tactics should be tracked, and an unlimited stream of suspicious patterns worth examining. Attempting to distribute effort equally across this scope means spreading resources so thin that nothing receives adequate attention.

This leads to a predictable failure pattern: hunters tackle whatever seems interesting each day, jumping between investigations without clear priorities. Six months later, they've touched many areas superficially but have no compelling findings on critical systems. Leadership sees effort but not value. The program gets defunded or relegated to reactive incident response.

The hunters themselves burn out from facing impossibly large scope without clear direction. When everything seems important, nothing gets completed. The lack of visible progress is demoralizing. Without focus, it's hard to articulate impact, making the work feel meaningless.


## The Crown Jewels Framework: Protecting What Actually Matters

The most fundamental prioritization principle is deceptively simple: protect the most valuable and vulnerable assets first. Emergency rooms don't treat everyone equally - they triage patients based on severity. Hunting programs must do the same.

Start by identifying your "crown jewels" - assets whose compromise would cause disproportionate damage. This typically includes customer databases, intellectual property repositories, authentication infrastructure like Active Directory domain controllers, revenue-generating systems, and administrator workstations. The key is combining business impact with attack value: what matters most to your organization and to adversaries?

But identifying crown jewels requires more than listing sensitive systems. You need to assess vulnerability and existing detection coverage. An internet-facing customer database with weak logging represents higher risk than a well-monitored internal financial system. Crown jewels with minimal detection coverage become the highest priorities.

Consider a mid-sized healthcare organization with three hunters. They identified four crown jewels: the electronic health records database, pharmaceutical research data, domain controllers, and billing systems. Rather than attempting to hunt everywhere, they dedicated 50% of hunting time to these four assets - weekly hunting on each one. The remaining time covered critical attack paths like administrator workstations and VPN access, systematic technique hunting, and exploratory work.



The results were striking. Within three months, they identified suspicious authentication patterns on domain controllers that led to discovering a compromised administrator account. Six months in, they found evidence of data staging on the research repository that prevented intellectual property theft. These findings on genuinely critical systems demonstrated clear value, building credibility and securing continued investment.

Most importantly, they explicitly accepted lighter coverage elsewhere. General endpoints received quarterly hunting rather than weekly. Lower-value servers might be hunted semi-annually. This wasn't failure - it was strategic resource allocation aligned with actual risk.

## Beyond the Obvious: Risk-Based Thinking

Crown jewels provide the foundation, but threat hunting requires broader risk-based thinking. The fundamental equation is simple: **Risk = Likelihood × Impact × Exposure**. Hunt the highest-risk items first, accepting that lowest-risk items might receive minimal or no coverage.

Consider two scenarios. First, internet-facing web servers with known vulnerabilities containing customer data: high likelihood of targeting, high impact if compromised, high exposure. Second, internal development servers behind multiple security controls with no external access: moderate impact, low exposure, low likelihood. With limited resources, the first scenario demands attention; the second can wait.

This creates natural priority tiers. Highest priority combines high likelihood, high impact, and high exposure - think internet-facing systems with sensitive data. High priority has at least two of these factors - perhaps critical internal servers or privileged users frequently targeted by phishing. Medium priority has one factor - maybe development systems with moderate access. Lowest priority has none - general employee workstations with no special access or data.

The key insight is that distribution of hunting effort should mirror risk distribution, not be uniform. If 20% of your environment represents 80% of your risk, those assets should receive 80% of hunting attention. This seems obvious when stated explicitly, yet many programs default to equal distribution out of misplaced fairness or fear of missing something.


## The Technique Prioritization Problem

While asset-based prioritization addresses where to hunt, technique prioritization addresses what to hunt for. The MITRE ATT&CK framework catalogs over 200 adversary techniques. Comprehensive technique coverage would take years even for large teams.

The solution requires combining multiple factors. Start with prevalence: some techniques like credential dumping and lateral movement via remote services appear in most breaches. Hunt for common techniques before rare ones. Add organizational context: a company with extensive cloud infrastructure should prioritize cloud-specific techniques; one with primarily on-premises infrastructure focuses elsewhere. Layer in threat intelligence about active campaigns targeting your sector. Factor in detection coverage - hunt for techniques where automated detection is weak.

This creates a prioritized technique list aligned with actual threat landscape and organizational profile. A financial services firm might prioritize credential access and privilege escalation techniques, knowing they're heavily targeted for financial data. A healthcare organization might emphasize data exfiltration and initial access via medical devices, reflecting their threat profile.

The mistake is attempting systematic coverage of all techniques equally. Instead, establish a tier system: hunt for your top 30 techniques quarterly, the next 50 semi-annually, and lower-priority techniques opportunistically when capacity allows or intelligence indicates relevance.


## Embracing "Good Enough"

Perhaps the most psychologically difficult aspect of prioritization is accepting that hunting will never achieve comprehensive coverage. The perfect hunting program - one investigating every system, user, and technique continuously - is impossible. Pursuing it guarantees failure.

Instead, embrace a tiered approach: crown jewels receive intensive hunting (weekly or biweekly), critical paths get regular attention (monthly), high-risk systems are covered periodically (quarterly), and the general environment receives light, opportunistic hunting. This is "good enough" for organizational security needs.

The key is transparency. Don't claim comprehensive coverage when providing focused coverage. Instead: "We hunt our 50 most critical systems weekly, 200 high-risk systems monthly, and conduct quarterly technique hunts across the general environment." Explain why this prioritization makes sense, how it aligns with risk and resources, and what it would require to expand coverage. Demonstrate value by showing what current coverage is finding.

This honesty builds trust and manages expectations. Leadership understands trade-offs when they're explained clearly. What they can't accept is discovering after six months that the hunting program attempted to cover everything but actually covered nothing adequately.




## Balancing Depth and Breadth

A final prioritization dimension: should hunting be deep but narrow, or broad but shallow? Deep hunting means intensive investigation of few systems - thorough examination finding subtle threats and providing high confidence in specific areas. Broad hunting means light coverage across many systems - quick checks identifying obvious indicators and providing situational awareness.

The answer, unsurprisingly, is both - strategically allocated. Crown jewels demand deep hunting: thorough investigation, behavioural baselining, multiple hunting approaches, forensic capabilities. The general environment receives broad hunting: quick checks for obvious threats, cursory examination validating nothing obvious was missed. A practical distribution might allocate 60% of time to deep hunting of priorities, 30% to broad hunting for coverage, and 10% to exploratory work.

This balance should evolve as programs mature. Early-stage programs emphasize depth over breadth - focus intensively on crown jewels, demonstrate ability to find sophisticated threats, build expertise and credibility. Mature programs can afford more breadth as crown jewels receive strong automated detection and hunting expertise expands across broader environments.



## Making Prioritization Dynamic

The final critical element: priorities aren't static. New threat intelligence about campaigns targeting your sector requires adjusting priorities to hunt for specific threats. After incidents, prioritize hunting for similar threats and validate the incident wasn't broader. Critical vulnerability disclosures affecting your environment trigger focused hunting for exploitation. Organizational changes - new business initiatives, mergers, workforce changes - shift what constitutes crown jewels.

This requires regular priority reviews. Quarterly, reassess crown jewels, review whether threat patterns are shifting, evaluate whether priorities are receiving adequate hunting, and adjust accordingly. Annually, conduct comprehensive strategic reviews with major adjustments to hunting strategy and resource allocation planning.

Treat prioritization as an ongoing discipline, not a one-time decision. The security landscape evolves, organizational risks shift, and program capabilities mature. Priorities must evolve with them.

## The Path Forward

The counterintuitive secret of successful threat hunting is that it begins with accepting limitations. You cannot hunt everywhere. You cannot investigate everything. Resources are finite. Attempting comprehensive coverage means achieving inadequate coverage.

Success comes from the discipline to prioritize ruthlessly based on risk, focus intensively on what matters most, and explicitly accept lighter coverage elsewhere. It comes from transparently communicating these trade-offs rather than implying comprehensive coverage you cannot provide. It comes from demonstrating clear value on genuinely critical systems rather than diffuse effort across everything.

Most importantly, it comes from remembering that perfect is the enemy of good. A hunting program providing strong coverage of crown jewels, regular attention to critical paths, and opportunistic coverage of the broader environment delivers substantial security value. One attempting to hunt everything equally delivers none.

The question isn't whether you'll prioritize - you will, whether explicitly or implicitly. The question is whether you'll do so strategically, transparently, and effectively, or whether you'll attempt impossible comprehensiveness and fail to deliver value before the program is defunded.

Choose to hunt what matters. Accept that other things matter less. Focus your limited resources where they provide maximum security value. That discipline - not heroic effort or sophisticated tools - is what separates successful hunting programs from failed ones.



