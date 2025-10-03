## **Why Suricata for Network Threat Hunting?**

### **The Signature vs. Behaviour Paradigm**

To understand Suricata's value, we must first distinguish between two fundamental approaches to threat detection:

**Behavioural Analysis** (Zeek/RITA/AC-Hunter approach):

- Examines _patterns_ in network connections over time
- Identifies anomalies: unusual beaconing, rare ports, long connections, strange User-Agents etc
- Excels at detecting novel threats and zero-day attacks
- Generates fewer false positives for unknown threats but requires expertise to investigate anomalies
- Cannot identify _what_ specific threat is present per se, only that behaviour is suspicious

**Signature-Based Detection** (Suricata approach):

- Matches traffic against _known patterns_ of malicious activity
- Identifies specific threats: malware families, exploit techniques, C2 frameworks
- Excels at detecting known threats with high confidence
- Generates high-confidence alerts with specific threat identification
- May miss novel attacks that don't match existing signatures

Neither approach is superior - they're complementary. Behavioural analysis catches novel, unknown threats that signatures miss. Signature-based detection provides immediate, high-confidence identification of known threats. A mature threat hunting program leverages both.

### The Ideal Deployment

In a comprehensive network threat hunting architecture:

1. **Zeek generates metadata logs** capturing connection characteristics, protocol details, and timing information
2. **RITA and AC-Hunter analyze Zeek logs** to identify behavioural anomalies: beaconing, long connections, unusual protocols
3. **Suricata inspects packet payloads** matching against thousands of threat signatures
4. **Cross-correlation reveals the complete picture**:
    - RITA alerts on suspicious beaconing → Suricata identifies the C2 framework (Cobalt Strike, Metasploit)
    - Suricata alerts on exploit attempt → Zeek logs show which internal host was targeted and subsequent connections
    - Zeek identifies unusual SSL certificate → Suricata matches associated traffic to malware family
    - etc.

This layered approach maximizes detection coverage: behavioural analysis catches novel threats, signature detection provides immediate identification of known threats.




### **Suricata's Unique Value Propositions**

**1. Known Threat Identification**

When Suricata alerts on `ET MALWARE Cobalt Strike Beacon Observed`, you immediately know:

- The specific threat (Cobalt Strike)
- The threat category (C2 beacon)
- The confidence level (known signature match)

Compare this to a RITA alert showing "96% beacon score to 165.22.159.5:8443" - you know something is beaconing, but not _what_. Suricata bridges this gap.

**2. Payload Analysis**

Suricata can detect threats present in encrypted payloads by:

- Matching on TLS certificate characteristics
- Identifying protocol violations in TLS handshakes
- Detecting known malicious certificates
- Analyzing JA3/JA3S fingerprints to identify malware families

**3. Exploit Detection**

Suricata rules detect exploit attempts in real-time:

- Buffer overflow attempts
- SQL injection patterns
- Command injection strings
- CVE-specific exploit signatures

These attacks may succeed or fail, but Suricata alerts regardless - providing early warning of targeting.

**4. Lateral Movement Detection**

Suricata's SMB protocol analyzer identifies:

- Suspicious file transfers between internal hosts
- PsExec execution patterns
- Admin share access from unexpected sources
- Credential relay attacks

Combined with Sysmon (Event IDs 3, 17, 18), you can map complete lateral movement chains.

**5. Data Exfiltration**

File extraction capabilities enable:

- Capturing files uploaded/downloaded via HTTP, FTP, SMB
- Calculating file hashes for malware identification
- Detecting sensitive data patterns (SSN, credit cards) in exfiltrated data
- Integrating with sandbox analysis (Cuckoo, CAPE)


### **Real-World Detection Scenarios**

**Scenario 1: Detecting Cobalt Strike Beacons**

- **RITA/AC-Hunter**: Identifies host 192.168.1.55 beaconing to 143.198.3.13:8443 with 97% beacon score
- **Suricata**: Matches traffic to rule `ET MALWARE Cobalt Strike Beacon HTTP`, confirming the C2 framework
- **Zeek**: http.log shows User-Agent and URI patterns consistent with Cobalt Strike
- **Sysmon**: Event ID 3 identifies the process responsible (unknown binary in %TEMP%)

Result: Complete attack chain identified with high confidence - behavioural anomaly (beaconing) + signature match (Cobalt Strike) + process context (malicious binary).

**Scenario 2: Detecting Exploitation Attempt**

- **Suricata**: Alerts on `ET EXPLOIT Apache Log4j RCE Attempt` targeting web server 192.168.1.100
- **Zeek**: http.log captures full HTTP request with malicious JNDI payload
- **Suricata**: (hours later) Alerts on `ET MALWARE Generic Reverse Shell` from same web server
- **Sysmon**: Shows java.exe spawning bash → suspicious network connection

Result: Detected exploit attempt in real-time, confirmed successful compromise via subsequent C2 beacon, identified compromised process via endpoint telemetry.

**Scenario 3: Detecting Lateral Movement**

- **Suricata**: Alerts on `ET POLICY PsExec Service Start` from 192.168.1.25 to 192.168.1.40
- **Sysmon**: Event ID 3 shows suspicious SMB connections between same hosts
- **Sysmon**: Event IDs 17/18 reveal named pipe creation/connection consistent with PsExec
- **Suricata**: (minutes later) Both hosts now beaconing to external C2

Result: Lateral movement detected via Suricata signature, confirmed via Sysmon endpoint telemetry, C2 expansion identified.

