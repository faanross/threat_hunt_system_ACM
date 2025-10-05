# **LESSON 1.2: ARCHITECTURE DEEP DIVE**


## **Welcome to the Inner Workings of Zeek**

In Lesson 1.1, you learned about Zeek's philosophical approach and its place in the network security ecosystem. You understand that Zeek performs behavioural analysis through an event-driven architecture, but we haven't yet explored what that really means at a technical level. In this lesson, we're going to open the hood and examine exactly how Zeek works internally.

This isn't just academic knowledge. Understanding Zeek's architecture will directly impact your success in deploying and using the system. When you're planning your deployment, architectural knowledge will help you determine how many sensors you need and where to place them. When you're writing detection scripts, understanding the processing pipeline will help you write efficient code that doesn't degrade performance. When something goes wrong - and at some point, something always goes wrong - architectural knowledge will help you troubleshoot effectively.

We're going to explore three major aspects of Zeek's architecture. First, we'll examine the single-instance processing pipeline: how packets flow from the network interface through protocol analysis to script execution. Second, we'll dive deep into cluster architecture, understanding how Zeek scales to handle high-bandwidth networks by distributing work across multiple machines. Finally, we'll discuss memory management and performance considerations that will inform both your deployment decisions and your scripting practices.



---

## **PART 1: THE SINGLE-INSTANCE PROCESSING PIPELINE**

### **Understanding How Zeek Processes Network Traffic**

Let's start by understanding how a single instance of Zeek processes network traffic. Even if you eventually deploy Zeek in a distributed cluster (and most production deployments do), understanding the single-instance architecture is crucial because every node in a cluster runs this same processing pipeline.

Here's a high-level view of the pipeline:


```
┌─────────────────────────────────────────────────────────────┐
│                    NETWORK INTERFACE                        │
│         (Packets arriving from monitored network)           │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│                  PACKET ACQUISITION                         │
│   (Libpcap, AF_PACKET, PF_RING, or other capture method)    │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│                   PACKET FILTER                             │
│        (BPF filter to exclude irrelevant traffic)           │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│                  EVENT ENGINE                               │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐   │
│  │         Protocol Analysis & Parsing                  │   │
│  │  (TCP/IP stack, HTTP, DNS, SSL, FTP, SSH, etc.)      │   │
│  └──────────────────┬───────────────────────────────────┘   │
│                     │                                       │
│                     ▼                                       │
│  ┌──────────────────────────────────────────────────────┐   │
│  │          Connection State Management                 │   │
│  │    (Tracking ongoing connections and their state)    │   │
│  └──────────────────┬───────────────────────────────────┘   │
│                     │                                       │
│                     ▼                                       │
│  ┌──────────────────────────────────────────────────────┐   │
│  │            Event Generation                          │   │
│  │    (Creating events from protocol activities)        │   │
│  └──────────────────┬───────────────────────────────────┘   │
└─────────────────────┼───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│                    SCRIPT LAYER                             │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐   │
│  │           Event Handlers                             │   │
│  │    (Your detection scripts respond to events)        │   │
│  └──────────────────┬───────────────────────────────────┘   │
│                     │                                       │
│                     ▼                                       │
│  ┌──────────────────────────────────────────────────────┐   │
│  │           Logging Framework                          │   │
│  │      (Generate structured log files)                 │   │
│  └──────────────────┬───────────────────────────────────┘   │
│                     │                                       │
│                     ▼                                       │
│  ┌──────────────────────────────────────────────────────┐   │
│  │          Notice Framework                            │   │
│  │    (Generate alerts for suspicious activity)         │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│                      OUTPUT                                 │
│       (Log files, notices, exported data)                   │
└─────────────────────────────────────────────────────────────┘
```




This diagram shows the complete path that network traffic takes through Zeek. Let's examine each stage in detail, understanding not just what happens but why it's designed this way and what implications it has for your deployment and scripting.

### **Stage 1: Packet Acquisition - Getting Traffic Into Zeek**

The first challenge Zeek must solve is getting packets from the network into the application for analysis. This might seem straightforward - after all, operating systems provide mechanisms for applications to capture network traffic - but packet acquisition is actually one of the most critical performance considerations in network monitoring. If Zeek can't acquire packets fast enough to keep up with your network's traffic volume, it will drop packets, resulting in incomplete visibility.

Zeek supports several different packet acquisition methods, each with different performance characteristics and use cases. Understanding these options will help you choose the right approach for your deployment.



#### **Libpcap: The Default and Most Portable Option**

The default packet acquisition mechanism is `libpcap` (or WinPcap on Windows, though Zeek is rarely deployed on Windows). Libpcap is a portable C library that provides a consistent API for packet capture across different operating systems. When you install Zeek without any special configuration, it uses `libpcap` to capture packets from your network interface.

Here's how libpcap works at a high level:

```
Network Interface → Kernel Network Stack → Packet Filter (BPF) → Userspace Buffer → Zeek
```

Packets arrive at your network interface and are processed by the kernel's network stack. A packet filter written in BPF (Berkeley Packet Filter) bytecode runs in kernel space, allowing you to discard irrelevant traffic before it's copied to userspace. The packets that pass the filter are copied to a buffer in userspace where Zeek can access them.

**Advantages of libpcap:**

- **Portability**: Works on Linux, BSD, macOS, and other Unix-like systems with minimal differences
- **Maturity**: Extremely well-tested and reliable
- **Simplicity**: Easy to set up with no special configuration required
- **Flexibility**: Works with any network interface your OS supports

**Limitations of libpcap:**

- **Performance ceiling**: On very high-bandwidth networks (10+ Gbps), libpcap may struggle to keep up
- **Single-threaded**: Libpcap delivers packets to a single processing thread, creating a bottleneck
- **Kernel copying overhead**: Every packet must be copied from kernel space to userspace

For many deployments, especially those monitoring networks below 1 Gbps, libpcap performs perfectly well. It's the right choice when you're starting out or when you don't need maximum performance.




#### **AF_PACKET: Linux's Native High-Performance Alternative**

On Linux systems, Zeek can use AF_PACKET instead of libpcap for significantly better performance. AF_PACKET is a Linux-specific interface that allows applications to capture packets with lower overhead than libpcap.

The key advantage of AF_PACKET is its use of memory-mapped buffers. Instead of copying each packet from kernel space to userspace, AF_PACKET sets up a shared memory region that both the kernel and Zeek can access. Packets are written directly into this shared memory, eliminating the copy overhead.

**AF_PACKET packet flow:**

```
Network Interface → Kernel NIC Driver → Memory-Mapped Ring Buffer → Zeek
                                         (Shared between kernel and userspace)
```

**Advantages of AF_PACKET:**

- **Better performance**: Lower CPU overhead and higher packet rates compared to libpcap
- **Fewer dropped packets**: More efficient handling reduces the risk of packet loss
- **Native to Linux**: No additional software required on modern Linux systems

**Considerations:**

- **Linux-only**: Not portable to other operating systems
- **Requires proper configuration**: You need to specify fanout modes and buffer sizes appropriately
- **More complex setup**: Not as simple as the default libpcap option

For production deployments on Linux, especially those monitoring networks above 1 Gbps, AF_PACKET is generally the better choice. We'll configure it when we install Zeek in Lesson 1.3.


#### **PF_RING: The High-Performance Commercial Option**

For the most demanding environments-networks running at 10 Gbps or higher-there's PF_RING, a high-performance packet capture framework developed by ntop. PF_RING provides even better performance than AF_PACKET through several optimizations.

**Key PF_RING features:**

- **Kernel bypass capabilities**: Can deliver packets directly from the network card to userspace, bypassing much of the kernel network stack
- **Hardware acceleration**: Supports offloading filtering and load balancing to compatible network cards
- **Better distribution**: More sophisticated algorithms for distributing packets across multiple processing threads

**The PF_RING trade-offs:**

```
┌─────────────────────────────────────────────────────────────┐
│                      PERFORMANCE                            │
│                                                             │
│  Libpcap     <     AF_PACKET     <     PF_RING              │
│  (Baseline)        (2-3x better)       (3-5x better)        │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│                     COMPLEXITY                              │
│                                                             │
│  Libpcap     <     AF_PACKET     <     PF_RING              │
│  (Simple)          (Moderate)          (Complex)            │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│                        COST                                 │
│                                                             │
│  Libpcap     <     AF_PACKET     <     PF_RING              │
│  (Free)            (Free)              (License required)   │
└─────────────────────────────────────────────────────────────┘
```

For most learning and even production deployments, PF_RING isn't necessary. We'll use AF_PACKET in this course, which provides excellent performance for networks up to several gigabits per second without the complexity or cost of PF_RING.


### **Stage 2: Packet Filtering - Reducing Unnecessary Load**

Before Zeek processes packets, they pass through a packet filter implemented using BPF (Berkeley Packet Filter). This filter runs in kernel space and can discard packets before they're delivered to Zeek, dramatically reducing the processing load.

You might wonder why you'd want to filter packets when the goal is comprehensive network visibility. The answer is that not all network traffic is relevant to security monitoring, and processing irrelevant traffic wastes resources that could be used for analyzing important traffic.

**Common filtering scenarios:**

|Traffic Type|Should Monitor?|Reasoning|
|---|---|---|
|Internet-bound traffic|Yes|Critical for detecting C2, data exfiltration|
|Internal server traffic|Yes|Important for lateral movement detection|
|Printer to file server|Probably not|High volume, low security relevance|
|Backup traffic|Probably not|High volume, legitimate file transfers|
|Network management (SNMP)|Maybe|Depends on your monitoring goals|

BPF filters are specified using a simple syntax. Here are some examples:

```
# Monitor only TCP traffic
tcp

# Monitor web traffic (HTTP and HTTPS)
port 80 or port 443

# Monitor all traffic except to/from backup server
not host 192.168.1.50

# Monitor only outbound traffic from internal network
src net 192.168.0.0/16 and dst net not 192.168.0.0/16

# Complex: Monitor HTTP(S) and DNS, but exclude internal DNS server
(port 80 or port 443 or port 53) and not (host 192.168.1.10 and port 53)
```

The key principle is to be thoughtful about what you filter out. Filter aggressively enough to reduce unnecessary load, but not so aggressively that you lose visibility into important security events. When in doubt, err on the side of capturing more rather than less-you can always adjust your filters based on experience.

In case you wanted to alter your BPF filter, this can typically be done directly **within Zeek's own configuration files**.

Think of it as giving Zeek a set of instructions on what traffic to even look at in the first place.

#### Where and How to Configure BPF Filters

The most common place to set your BPF filters is in the `local.zeek` file. This is the primary file for site-specific Zeek configurations. If you're running a Zeek cluster, you'll typically find this file in `<zeek_install_dir>/share/zeek/site/`.

You'll use the `restrict_filters` variable to define your BPF rules. Let's take a look at how you'd implement the examples from above.

```
# Redefine the restrict_filters to add our custom BPF rules.
redef restrict_filters += {
    # Monitor only TCP traffic
    ["only-tcp"] = "tcp",

    # Monitor web traffic (HTTP and HTTPS)
    ["web-traffic"] = "port 80 or port 443",

    # Monitor all traffic except to/from backup server
    ["exclude-backup-server"] = "not host 192.168.1.50",

    # Monitor only outbound traffic from internal network
    ["outbound-only"] = "src net 192.168.0.0/16 and not dst net 192.168.0.0/16",

    # Complex: Monitor HTTP(S) and DNS, but exclude internal DNS server
    ["web-and-dns-filtered"] = "(port 80 or port 443 or port 53) and not (host 192.168.1.10 and port 53)"
};
```


**Key things to note:**

- **`redef restrict_filters +=`**: This is important. The `+=` appends your filters to any existing ones. If you use a single `=`, you'll overwrite the default filters.
- **`["filter-name"]`**: This is just a descriptive name for your filter. It's good practice to give each filter a unique and meaningful name.
- **`"bpf-syntax"`**: This is where you place your actual BPF filter, just like in your course examples.

#### Another Method: Using `PacketFilter::exclude`

You can also use the `PacketFilter::exclude` function within the `zeek_init()` event in a seperate Zeek script. This can be useful for more dynamic or complex filtering scenarios.

Here's an example of how you might use this method in a custom script:


```
event zeek_init() {
    # Exclude traffic to and from the backup server
    PacketFilter::exclude("exclude-backup-server", "host 192.168.1.50");
}
```

This achieves the same goal as the `restrict_filters` example. For most common use cases, sticking with `restrict_filters` is perfectly fine and often easier to manage.

After you've made changes to your configuration, you'll need to restart Zeek for the new BPF filters to take effect. You can verify that your filters have been applied by using the command `zeekctl diag`.


### **Stage 3: The Event Engine - The Heart of Zeek**

Now we reach the most complex and interesting part of Zeek's architecture: the event engine. This is where Zeek transforms raw packets into the high-level events that your scripts respond to. The event engine is what makes Zeek fundamentally different from packet inspection tools.

The event engine has three main responsibilities: protocol analysis, connection state management, and event generation. Let's examine each.

#### **Protocol Analysis: Understanding What's Being Communicated**

When packets arrive at the event engine, Zeek doesn't just look at their raw bytes. Instead, it parses the protocols being used, extracting meaningful information from protocol headers and payloads. Zeek includes built-in analyzers for dozens of protocols, from low-level protocols like IP and TCP up through application protocols like HTTP, DNS, SSH, SSL, and many others.

**Zeek's protocol analysis stack:**

```
┌─────────────────────────────────────────────────────────────┐
│                   APPLICATION PROTOCOLS                     │
│   HTTP, DNS, SSL/TLS, SSH, FTP, SMTP, SMB, RDP, etc.        │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ├─→ Extract: URLs, headers, status codes
                     ├─→ Extract: Query names, response IPs
                     ├─→ Extract: Certificates, cipher suites
                     └─→ Extract: Commands, file transfers
                     │
┌────────────────────▼────────────────────────────────────────┐
│                  TRANSPORT PROTOCOLS                        │
│                    TCP, UDP, ICMP                           │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ├─→ Track: Sequence numbers, ACKs
                     ├─→ Track: Connection states
                     └─→ Reassemble: Fragmented packets
                     │
┌────────────────────▼────────────────────────────────────────┐
│                   NETWORK PROTOCOLS                         │
│                      IP, IPv6                               │
└────────────────────┬────────────────────────────────────────┘
                     │
                     └─→ Extract: Addresses, TTL, flags
                     │
┌────────────────────▼────────────────────────────────────────┐
│                    LINK LAYER                               │
│                 Ethernet, VLAN Tags                         │
└─────────────────────────────────────────────────────────────┘
```

Let's walk through an example to make this concrete. Imagine someone on your network visits a website. Here's what Zeek's protocol analysis does:




**Step 1: Link Layer Processing**

```
Ethernet Frame → Extract source/destination MAC addresses
              → Identify protocol (IPv4)
              → Strip Ethernet header
```

**Step 2: Network Layer Processing**

```
IP Packet → Extract source IP: 192.168.1.100
         → Extract destination IP: 93.184.216.34
         → Identify protocol (TCP)
         → Strip IP header
```

**Step 3: Transport Layer Processing**

```
TCP Segment → Extract source port: 52847
           → Extract destination port: 443
           → Track sequence numbers
           → Handle TCP flags (SYN, ACK, etc.)
           → Reassemble packets into data stream
           → Identify this as an HTTPS connection
```

**Step 4: Application Layer Processing**

```
TLS/SSL → Parse ClientHello
        → Extract TLS version: 1.3
        → Extract cipher suites offered
        → Parse ServerHello
        → Extract chosen cipher suite
        → Parse certificate
        → Extract subject: www.example.com
        → Extract issuer: DigiCert Inc.
        → Validate certificate chain
```

All of this analysis happens automatically. By the time your detection scripts see this connection, Zeek has already extracted every meaningful piece of information from it. Your scripts work with high-level abstractions like "HTTP request" or "SSL certificate" rather than raw packet bytes.

This multi-layer protocol analysis is what enables behavioural detection. You can ask questions like "Show me HTTPS connections where the certificate subject doesn't match the SNI hostname" or "Find HTTP POST requests with unusually large payloads sent to recently-registered domains." These questions would be nearly impossible to answer working at the packet level, but they're straightforward with Zeek's protocol analysis.




#### **Connection State Management: Tracking the Full Picture**

The second major responsibility of the event engine is maintaining state about network connections. Unlike stateless packet inspection, which examines each packet independently, Zeek tracks connections from establishment through termination, building a complete picture of each communication session.

For every connection, Zeek maintains a connection record that includes:

**Core Connection Information:**

```
┌──────────────────────────────────────────────────────┐
│ Connection UID: Unique identifier (C9x4Cz3...)       │
├──────────────────────────────────────────────────────┤
│ Source IP: 192.168.1.100                             │
│ Source Port: 52847                                   │
│ Destination IP: 93.184.216.34                        │
│ Destination Port: 443                                │
│ Protocol: TCP                                        │
├──────────────────────────────────────────────────────┤
│ Start Time: 2025-10-03 14:32:18.423                  │
│ Duration: 127.456 seconds                            │
│ Last Activity: 2025-10-03 14:34:25.879               │
├──────────────────────────────────────────────────────┤
│ Originator Bytes: 4,826                              │
│ Responder Bytes: 128,472                             │
│ Originator Packets: 89                               │
│ Responder Packets: 143                               │
├──────────────────────────────────────────────────────┤
│ Connection State: Established → Data Transfer →      │
│                   Closing → Closed                   │
│ State Flags: SF (normal establishment and close)     │
├──────────────────────────────────────────────────────┤
│ Service: SSL                                         │
│ Service Confidence: High                             │
└──────────────────────────────────────────────────────┘
```

But Zeek doesn't just track basic connection metadata. For protocols it understands, it maintains protocol-specific state:


**HTTP Connection State:**

```
┌──────────────────────────────────────────────────────┐
│ HTTP Transaction History:                            │
│                                                      │
│ Request 1:                                           │
│   Method: GET                                        │
│   URI: /index.html                                   │
│   User-Agent: Mozilla/5.0...                         │
│   Response Code: 200                                 │
│   Response Size: 4,826 bytes                         │
│                                                      │
│ Request 2:                                           │
│   Method: GET                                        │
│   URI: /style.css                                    │
│   User-Agent: Mozilla/5.0...                         │
│   Response Code: 200                                 │
│   Response Size: 12,340 bytes                        │
│                                                      │
│ [... additional requests ...]                        │
└──────────────────────────────────────────────────────┘
```






This stateful tracking enables sophisticated analysis. You can detect:

- Connections that established but never transferred data (potential scanning)
- Connections with unusual byte ratios (potential C2 beaconing)
- HTTP sessions where user agents change mid-session (potential session hijacking)
- Long-lived connections that periodically exchange small amounts of data (potential backdoors)

The state management also handles complex real-world scenarios. Zeek correctly handles:

- **TCP retransmissions**: Recognizing and deduplicating retransmitted packets
- **Out-of-order packets**: Reassembling them into the correct sequence
- **Fragmented packets**: Reassembling IP fragments and TCP segments
- **Connection reuse**: Tracking multiple HTTP transactions over a persistent connection
- **Timeouts**: Expiring state for connections that go idle


#### **Event Generation: Translating Protocol Activity Into Events**

The final responsibility of the event engine is generating events based on protocol activity. An event is a signal that something happened on the network - a connection was established, an HTTP request was made, a DNS query was issued, a file transfer began. Events are the interface between Zeek's core protocol analysis and your detection scripts.

Zeek generates events at multiple levels of granularity:

**Low-Level Network Events:**

```
new_connection()        - TCP connection established
connection_attempt()    - SYN packet seen (connection attempt)
connection_rejected()   - Connection refused (RST received)
connection_state_remove() - Connection finished
```

**Protocol-Specific Events:**

```
http_request()          - HTTP request observed
http_reply()            - HTTP response observed
dns_request()           - DNS query seen
dns_reply()             - DNS response received
ssl_established()       - SSL/TLS handshake completed
x509_certificate()      - X.509 certificate observed
smtp_request()          - SMTP command sent
smtp_reply()            - SMTP response received
```

**File Transfer Events:**

```
file_new()              - New file transfer detected
file_over_new_connection() - File transferred over new connection
file_hash()             - File hash calculated
```

**Connection Analysis Events:**

```
conn_stats()            - Periodic connection statistics
weird()                 - Protocol violation detected
```

When an event fires, it carries rich contextual information. For example, when the `http_request()` event fires, your script receives:

```
event http_request(c: connection, method: string, original_URI: string,
                   unescaped_URI: string, version: string)
```

Where:

- `c` is the full connection record with all state information
- `method` is the HTTP method (GET, POST, etc.)
- `original_URI` is the requested URI exactly as seen
- `unescaped_URI` is the URI with URL encoding resolved
- `version` is the HTTP version (1.0, 1.1, 2.0)

Your detection script can access all this information plus the full connection context. You might check if the URI matches a known-malicious pattern, if the requesting IP is on a watchlist, if the User-Agent is suspicious, if this is an unusual destination for this source, and more - all with the context Zeek has already gathered and structured for you.



### **Stage 4: The Script Layer - Where Your Detection Logic Lives**

After the event engine has parsed protocols, tracked state, and generated events, control passes to the script layer. This is where your detection logic executes. The script layer is implemented in Zeek's scripting language, which we'll explore in detail in Module 2, but it's important to understand its role in the architecture now.

#### **Event Handlers: Responding to Network Activity**

The primary way your scripts interact with Zeek is by defining event handlers - functions that are called when specific events occur. When you write a detection script, you're essentially telling Zeek "When this type of event happens, execute this code."

Here's a simple conceptual example (don't worry about the exact syntax yet):

```zeek
# When any HTTP request is observed...
event http_request(c: connection, method: string, original_URI: string,
                   unescaped_URI: string, version: string)
{
    # Check if the request is a POST to a suspicious path
    if ( method == "POST" && /\/api\/beacon/ in original_URI )
    {
        # This looks like potential C2 traffic
        # Generate an alert
        print fmt("Suspicious POST to %s from %s", original_URI, c$id$orig_h);
    }
}
```



When Zeek processes HTTP traffic, the event engine generates `http_request()` events. Each event triggers your handler function, passing in all the relevant information. Your code executes, performs whatever analysis you've programmed, and returns. The event engine continues processing traffic, generating more events, and calling your handlers.


#### **Execution Model: Single-Threaded Event Processing**

It's crucial to understand that Zeek's script layer is single-threaded within each Zeek process. Events are processed sequentially, one at a time. When an event handler is executing, no other events are processed until it completes.

```
Packet Arrives → Event Generated → Handler Executes → Handler Completes → Next Event
                                         ↓
                              Script execution blocks
                              other event processing
```

This has important implications:

**Performance Impact:** If your event handler is slow, it will slow down all event processing. Imagine you write a handler that does an expensive operation - perhaps querying a remote database - on every HTTP request. If each query takes 100 milliseconds and you're processing 1000 HTTP requests per second, you'd need 100 seconds to process one second of traffic. Zeek would fall hopelessly behind, dropping packets and missing events.

**Best Practices:**

- Keep event handlers fast and efficient
- Avoid blocking operations (network calls, disk I/O) in event handlers
- Use Zeek's built-in mechanisms for expensive operations (like the Intel Framework for lookups)
- Aggregate data and perform expensive analysis periodically rather than on every event

**State Sharing:** The single-threaded model does have an advantage: you don't need to worry about concurrency. All your event handlers share the same state, and you can modify shared data structures without locking or synchronization. When you track connection patterns across multiple events, you know your data structures won't be modified by another thread while you're using them.



#### **Frameworks: Structured Functionality for Common Tasks**

Zeek includes several frameworks - structured collections of scripts that provide sophisticated functionality for common security monitoring tasks. These frameworks execute in the script layer, but they're substantial enough that they feel like core Zeek features.

**Key Frameworks:**

|Framework|Purpose|Common Use Cases|
|---|---|---|
|**Intel Framework**|Match network activity against threat intelligence|Detect connections to known-malicious IPs, domains, URLs|
|**Notice Framework**|Generate and manage alerts|Create actionable alerts with context, suppression, and escalation|
|**Logging Framework**|Create custom log streams|Log specific detected behaviors in structured format|
|**File Analysis Framework**|Extract and analyze files|Extract executables, compute hashes, submit to sandboxes|
|**Input Framework**|Read data from external sources|Load configuration, read indicator lists, import data|
|**Cluster Framework**|Coordinate distributed deployment|Manage communication between cluster nodes|
|**SumStats Framework**|Perform statistical analysis|Count events, track thresholds, detect volumetric anomalies|

We'll work with these frameworks extensively in later modules. For now, understand that they provide powerful capabilities that your detection scripts can leverage. Rather than implementing statistical analysis from scratch, you can use the SumStats Framework. Rather than building threat intelligence matching logic, you can use the Intel Framework.




#### **The Logging System: Structured Output Generation**

The final stage in the script layer is logging. Based on the events processed and the analysis performed, Zeek generates structured log files that document network activity and detected threats.

Zeek's logs are tab-separated value (TSV) files with a specific structure:

```
#separator \x09
#set_separator	,
#empty_field	(empty)
#unset_field	-
#path	conn
#open	2025-10-03-14-32-18
#fields	ts	uid	id.orig_h	id.orig_p	id.resp_h	id.resp_p	proto	service	duration	orig_bytes	resp_bytes	conn_state	local_orig	local_resp	missed_bytes	history	orig_pkts	orig_ip_bytes	resp_pkts	resp_ip_bytes	tunnel_parents
#types	time	string	addr	port	addr	port	enum	string	interval	count	count	string	bool	bool	count	string	count	count	count	count	set[string]
1696348338.423000	C9x4Cz3...	192.168.1.100	52847	93.184.216.34	443	tcp	ssl	127.456	4826	128472	SF	-	-	0	ShADadFf	89	8744	143	135294	(empty)
```

**Standard Log Files:**

Zeek generates many log types by default:

|Log File|Contents|
|---|---|
|`conn.log`|All network connections with metadata|
|`http.log`|HTTP requests and responses|
|`dns.log`|DNS queries and responses|
|`ssl.log`|SSL/TLS connection parameters|
|`x509.log`|Certificates observed|
|`files.log`|File transfers across protocols|
|`smtp.log`|Email transactions|
|`ssh.log`|SSH connections|
|`weird.log`|Protocol violations and anomalies|
|`notice.log`|Alerts generated by detection scripts|

Your detection scripts can also create custom log files to record specific behaviors you're tracking. We'll learn how to do this in Module 3.

The logs are designed to be both human-readable and machine-parseable. You can examine them with standard Unix tools (`less`, `grep`, `awk`), but they're also structured enough to be easily ingested by log analysis platforms, SIEMs, and custom analysis scripts.

---



## **PART 2: CLUSTER ARCHITECTURE - SCALING TO HIGH-BANDWIDTH NETWORKS**

### **Why Clusters Are Necessary**

The single-instance architecture we've explored works well for many networks, but it has inherent scalability limits. A single Zeek process running on one machine can typically handle somewhere between 500 Mbps and 2 Gbps of traffic, depending on the hardware, the traffic mix, and the complexity of your detection scripts. What happens when you need to monitor a 10 Gbps network? Or a 100 Gbps data center?

The answer is Zeek's cluster architecture, which distributes the analysis workload across multiple machines or processors. Rather than trying to build a bigger, faster single system (vertical scaling), Zeek scales horizontally by adding more systems that work together.

**Scaling approaches compared:**

```
┌─────────────────────────────────────────────────────────────┐
│                    VERTICAL SCALING                         │
│              (Single Faster Machine)                        │
│                                                             │
│   Traffic → │████████████│ → Analysis                       │
│             │One Powerful│                                  │
│             │   Zeek     │                                  │
│                                                             │
│   Limits: Hardware ceiling, cost increases exponentially    │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                   HORIZONTAL SCALING                        │
│                (Multiple Machines)                          │
│                                                             │
│                ┌─────────┐                                  │
│   Traffic →    │Worker 1 │ → Analysis                       │
│          ↓     └─────────┘                                  │
│          →     ┌─────────┐                                  │
│          ↓     │Worker 2 │ → Analysis                       │
│          →     └─────────┘                                  │
│          ↓     ┌─────────┐                                  │
│          →     │Worker 3 │ → Analysis                       │
│                └─────────┘                                  │
│                                                             │
│   Limits: Coordination overhead, but scales much further    │
└─────────────────────────────────────────────────────────────┘
```



### **Cluster Roles: Division of Labor**

A Zeek cluster consists of multiple nodes, each playing a specific role. Understanding these roles and how they interact is essential for designing effective deployments. There are four different types of nodes.


#### **1. Workers: The Packet Processing Engines**

Workers are the nodes that actually capture and analyze network traffic. In a cluster, traffic is distributed across multiple workers, with each worker handling a portion of the total traffic volume. Workers perform all the packet acquisition, protocol analysis, and event generation we discussed in the single-instance architecture.

**Worker characteristics:**

- Directly capture packets from network interfaces or load balancers
- Run the full event engine and script layer
- Generate logs from their portion of the traffic
- Send notices and summaries to the manager
- Require significant CPU and memory resources

**Typical worker deployment:**

```
Network → Load Balancer → Worker 1 (handles 25% of traffic)
                      ├─→ Worker 2 (handles 25% of traffic)
                      ├─→ Worker 3 (handles 25% of traffic)
                      └─→ Worker 4 (handles 25% of traffic)
```

The number of workers you need depends on your traffic volume and the complexity of your analysis. A good rule of thumb is that each worker can handle 500 Mbps to 2 Gbps, so a 10 Gbps network might need 5-10 workers depending on configuration and hardware.

#### **2. Manager: The Coordination Hub**

The manager node coordinates the cluster, receives notices from workers, and makes cluster-wide decisions. Unlike workers, the manager doesn't directly analyze network traffic. Instead, it performs administrative and coordination tasks.

**Manager responsibilities:**

```
┌─────────────────────────────────────────────────────────────┐
│                   MANAGER NODE DUTIES                       │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ► Receive notices from all workers                         │
│    (Workers send alerts to manager for deduplication)       │
│                                                             │
│  ► Aggregate and deduplicate notices                        │
│    (If multiple workers detect same threat, merge alerts)   │
│                                                             │
│  ► Make cluster-wide policy decisions                       │
│    (Determine if behavior is significant across all workers)│
│                                                             │
│  ► Distribute updated intelligence/configuration            │
│    (Push new indicators to all workers)                     │
│                                                             │
│  ► Generate summary statistics                              │
│    (Aggregate metrics from all workers)                     │
│                                                             │
│  ► Coordinate response actions                              │
│    (Orchestrate cluster-wide responses to threats)          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

The manager is typically a less powerful machine than the workers since it's not processing high-volume traffic. However, it requires good network connectivity to all workers and sufficient resources to handle the volume of notices and coordination messages.

#### **3. Proxy: The Optional Load Distribution Layer**

Proxies are optional intermediary nodes that sit between workers and the manager. They aggregate data from multiple workers before forwarding it to the manager, reducing the connection and processing load on the manager.

**When proxies are valuable:**

|Cluster Size|Proxy Recommendation|Reasoning|
|---|---|---|
|1-5 workers|Not needed|Manager can handle direct connections|
|6-20 workers|One proxy|Reduces manager load significantly|
|20+ workers|Multiple proxies|Essential for scalability|

**Proxy communication flow:**

```
Worker 1  ┐
Worker 2  ├─→ Proxy 1 ─┐
Worker 3  ┘            ├─→ Manager
                       │
Worker 4  ┐            │
Worker 5  ├─→ Proxy 2 ─┘
Worker 6  ┘
```

Proxies aggregate and deduplicate data before sending it to the manager. For example, if three workers behind a proxy all detect connections to the same malicious IP, the proxy can aggregate these into a single notice with a count, rather than sending three separate notices to the manager.

#### **4. Logger: The Centralized Log Collector**

The logger node receives log data from all workers and writes consolidated log files. Rather than having each worker write its own log files (which would result in fragmented logs across multiple machines), the logger centralizes logging.

**Logger architecture:**

```
Worker 1 ─┐
Worker 2 ─┼─→ Logger ─→ Consolidated Logs
Worker 3 ─┘              (conn.log, http.log, etc.)
```

**Advantages of centralized logging:**

- Single location for all log data (easier analysis)
- Consistent log file rotation and management
- Reduced storage requirements (no duplication)
- Simplified log forwarding to SIEM

**Storage considerations:** The logger needs substantial storage capacity. As a rough estimate:

- 1 Gbps network: ~50-100 GB per day
- 10 Gbps network: ~500 GB - 1 TB per day
- Storage needs vary greatly based on traffic composition and retention requirements


### **Cluster Communication: How Nodes Coordinate**

The nodes in a Zeek cluster need to communicate with each other to coordinate analysis, share state, and aggregate results. This communication is handled by Zeek's Broker library, a high-performance pub/sub messaging system designed specifically for Zeek.

**Broker communication model:**

```
┌──────────────────────────────────────────────────────────────┐
│                    ZEEK CLUSTER COMMUNICATION                │
│                                                              │
│                        Manager                               │
│                           ▲                                  │
│                           │                                  │
│            ┌──────────────┼──────────────┐                   │
│            │              │              │                   │
│            ▼              ▼              ▼                   │
│         Proxy 1        Proxy 2        Proxy 3                │
│            ▲              ▲              ▲                   │
│       ┌────┼────┐    ┌────┼────┐    ┌────┼────┐              │
│       │    │    │    │    │    │    │    │    │              │
│       ▼    ▼    ▼    ▼    ▼    ▼    ▼    ▼    ▼              │
│      W1   W2   W3   W4   W5   W6   W7   W8   W9              │
│                                                              │
│  Communication Types:                                        │
│  ════════════════════                                        │
│  W→P: Logs, Notices, Metrics (high volume)                   │
│  P→M: Aggregated data (moderate volume)                      │
│  M→P: Policy updates, Intel feeds (low volume)               │
│  P→W: Distributed policy (low volume)                        │
└──────────────────────────────────────────────────────────────┘
```

**What gets communicated:**

|Data Type|Direction|Purpose|Volume|
|---|---|---|---|
|Log entries|Workers → Logger|Centralized logging|Very High|
|Notices|Workers → Manager|Alerts and detections|Medium|
|Metrics|Workers → Manager|Performance monitoring|Low|
|Intel updates|Manager → Workers|Threat intelligence|Low|
|State synchronization|Bidirectional|Shared analysis state|Variable|

### **Load Balancing: Distributing Traffic Across Workers**

One of the most critical aspects of cluster deployment is how you distribute network traffic across workers. If the distribution is uneven, some workers will be overwhelmed while others sit idle. If it doesn't maintain connection affinity, workers won't have complete visibility into connections.

**Load balancing requirements:**

```
┌──────────────────────────────────────────────────────────────┐
│          REQUIREMENTS FOR EFFECTIVE LOAD BALANCING           │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. BALANCE                                                  │
│     Each worker should receive approximately equal traffic   │
│     volume to prevent overload                               │
│                                                              │
│  2. AFFINITY                                                 │
│     All packets from the same connection must go to the      │
│     same worker so it can maintain complete state            │
│                                                              │
│  3. STABILITY                                                │
│     Load distribution should be consistent - same connection │
│     should always go to same worker                          │
│                                                              │
│  4. SCALABILITY                                              │
│     Should support adding/removing workers without           │
│     completely redistributing all traffic                    │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

**Common load balancing methods:**

#### **Method 1: Hardware Load Balancers**

Expensive network switches and load balancers can distribute traffic based on connection 5-tuples (source IP, source port, destination IP, destination port, protocol). They use hashing to ensure connection affinity while distributing load.

**Advantages:**

- Purpose-built hardware with line-rate performance
- No overhead on Zeek workers
- Supports very high bandwidth (40 Gbps+)

**Disadvantages:**

- Expensive
- Requires specialized knowledge
- Single point of failure (unless redundant)


#### **Method 2: AF_PACKET Fanout**

On Linux, `AF_PACKET` supports "fanout" mode, which distributes packets across multiple processes on the same host. Each worker process gets a portion of the traffic from a shared network interface.

```
┌────────────────────────────────────────────────────────────┐
│               Single Host with Multiple Workers            │
│                                                            │
│   Network Interface (10 Gbps)                              │
│         │                                                  │
│         ├──────────┬──────────┬──────────┬──────────       │
│         │          │          │          │                 │
│         ▼          ▼          ▼          ▼                 │
│     Worker 1   Worker 2   Worker 3   Worker 4              │
│     (2.5G)     (2.5G)     (2.5G)     (2.5G)                │
│                                                            │
│   AF_PACKET fanout distributes packets with connection     │
│   affinity maintained through hash of 5-tuple              │
└────────────────────────────────────────────────────────────┘
```

**Configuration:**

```
# In node.cfg
[worker-1]
type=worker
host=sensor-01
interface=af_packet::eth0
af_packet_fanout_id=23
af_packet_fanout_mode=PACKET_FANOUT_HASH
```

**Advantages:**

- No additional hardware required
- Excellent performance
- Built into Linux kernel
- Free

**Disadvantages:**

- Limited to single host (can't distribute across multiple machines)
- Requires modern Linux kernel


#### **Method 3: PF_RING Clustering**

`PF_RING` includes sophisticated clustering capabilities that can distribute traffic across multiple processes or even multiple hosts.

**Advantages:**

- Best performance for very high bandwidth
- Can distribute across multiple physical hosts
- Supports hardware offload with compatible NICs

**Disadvantages:**

- Requires PF_RING license
- More complex setup
- Additional cost

### **Cluster Deployment Patterns**

Let's look at some common cluster deployment patterns for different scales and requirements.

#### **Small Cluster (1-5 Gbps)**

```
┌─────────────────────────────────────────────────────────────┐
│              SMALL CLUSTER ARCHITECTURE                     │
│                                                             │
│                   ┌───────────┐                             │
│                   │ Manager   │                             │
│                   │ + Logger  │ (Combined on one host)      │
│                   └─────┬─────┘                             │
│                         │                                   │
│             ┌───────────┼───────────┐                       │
│             │           │           │                       │
│             ▼           ▼           ▼                       │
│        ┌────────┐  ┌────────┐  ┌────────┐                   │
│        │Worker 1│  │Worker 2│  │Worker 3│                   │
│        └────────┘  └────────┘  └────────┘                   │
│                                                             │
│  Hardware:                                                  │
│  - Manager/Logger: 4 cores, 16GB RAM, 1TB storage           │
│  - Each Worker: 8 cores, 32GB RAM, modest storage           │
│                                                             │
│  Best for: Small to medium enterprises, branch offices      │
└─────────────────────────────────────────────────────────────┘
```

#### **Medium Cluster (5-20 Gbps)**

```
┌─────────────────────────────────────────────────────────────┐
│              MEDIUM CLUSTER ARCHITECTURE                    │
│                                                             │
│               ┌──────────┐        ┌──────────┐              │
│               │ Manager  │        │  Logger  │              │
│               └────┬─────┘        └────┬─────┘              │
│                    │                   │                    │
│                    └─────────┬─────────┘                    │
│                              │                              │
│                    ┌─────────┴─────────┐                    │
│                    │                   │                    │
│                    ▼                   ▼                    │
│               ┌────────┐          ┌────────┐                │
│               │Proxy 1 │          │Proxy 2 │                │
│               └───┬────┘          └───┬────┘                │
│                   │                   │                     │
│        ┌──────────┼──────┐   ┌────────┼──────────┐          │
│        │          │      │   │        │          │          │
│        ▼          ▼      ▼   ▼        ▼          ▼          │
│    ┌───────┐ ┌───────┐ ┌───────┐ ┌───────┐ ┌───────┐        │
│    │Work 1 │ │Work 2 │ │Work 3 │ │Work 4 │ │Work 5 │        │
│    └───────┘ └───────┘ └───────┘ └───────┘ └───────┘        │
│        ...                           ...                    │
│                                                             │
│  Hardware:                                                  │
│  - Manager: 8 cores, 32GB RAM                               │
│  - Logger: 4 cores, 16GB RAM, 5TB+ storage                  │
│  - Proxies: 4 cores, 16GB RAM each                          │
│  - Workers: 12+ cores, 64GB RAM each                        │
│                                                             │
│  Best for: Large enterprises, service providers             │
└─────────────────────────────────────────────────────────────┘
```

#### **Large Cluster (20+ Gbps)**

```
┌──────────────────────────────────────────────────────────────┐
│               LARGE CLUSTER ARCHITECTURE                     │
│                                                              │
│                      ┌──────────┐                            │
│                      │ Manager  │                            │
│                      └────┬─────┘                            │
│                           │                                  │
│        ┌──────────────────┼──────────────────┐               │
│        │                  │                  │               │
│        ▼                  ▼                  ▼               │
│   ┌────────┐        ┌────────┐        ┌────────┐             │
│   │Proxy 1 │        │Proxy 2 │        │Proxy 3 │             │
│   └───┬────┘        └───┬────┘        └───┬────┘             │
│       │                 │                 │                  │
│    [Workers]         [Workers]         [Workers]             │
│    Rack 1            Rack 2            Rack 3                │
│    W1-W8             W9-W16            W17-W24               │
│                                                              │
│              Separate Logging Infrastructure:                │
│                                                              │
│                      ┌──────────┐                            │
│                      │ Logger   │                            │
│                      │ Frontend │                            │
│                      └────┬─────┘                            │
│                           │                                  │
│              ┌────────────┼────────────┐                     │
│              │            │            │                     │
│              ▼            ▼            ▼                     │
│         ┌────────┐   ┌────────┐   ┌────────┐                 │
│         │Storage │   │Storage │   │Storage │                 │
│         │ Node 1 │   │ Node 2 │   │ Node 3 │                 │
│         └────────┘   └────────┘   └────────┘                 │
│                                                              │
│  Best for: Data centers, ISPs, critical infrastructure       │
└──────────────────────────────────────────────────────────────┘
```

---



## **PART 3: MEMORY MANAGEMENT AND PERFORMANCE OPTIMIZATION**

### **Understanding Zeek's Memory Usage**

Zeek is a memory-intensive application because it maintains rich state about network connections and protocol sessions. Understanding how Zeek uses memory will help you size your systems appropriately and write scripts that don't cause memory problems.

**Major memory consumers:**

```
┌──────────────────────────────────────────────────────────────┐
│                ZEEK MEMORY USAGE BREAKDOWN                   │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  CONNECTION STATE (40-60% of memory)                         │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━                         │
│  • Connection records for all active connections             │
│  • Protocol-specific state (HTTP transactions, DNS queries)  │
│  • Packet reassembly buffers                                 │
│                                                              │
│  SCRIPT STATE (20-40% of memory)                             │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━                         │
│  • Tables and sets maintained by scripts                     │
│  • Intel framework indicator database                        │
│  • Statistical tracking structures                           │
│  • Custom data structures in your scripts                    │
│                                                              │
│  PACKET BUFFERS (10-20% of memory)                           │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━                         │
│  • Buffers for incoming packets                              │
│  • Reassembly buffers for fragmented traffic                 │
│                                                              │
│  ZEEK CORE (5-10% of memory)                                 │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━                         │
│  • Zeek executable and libraries                             │
│  • Event engine data structures                              │
│  • Protocol analyzers                                        │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

**Memory usage estimation:**

Here's a rough formula for estimating Zeek's memory requirements:

```
Base Memory = 2 GB (Zeek core + base scripts)

Connection Memory = (Active Connections) × (Memory per Connection)
                  = (Active Connections) × ~10 KB

Script Memory = Variable, depends on your detection scripts
              = 1-4 GB for typical deployments

Total = Base + Connection Memory + Script Memory
```

**Example calculations:**

|Network Type|Concurrent Connections|Estimated Memory|
|---|---|---|
|Small office (100 Mbps)|~5,000|2 + (5K × 10KB) = ~2.5 GB|
|Medium enterprise (1 Gbps)|~50,000|2 + (50K × 10KB) = ~3 GB|
|Large enterprise (10 Gbps)|~500,000|2 + (500K × 10KB) = ~7 GB|
|Data center (100 Gbps)|~5,000,000|2 + (5M × 10KB) = ~50 GB|

Add 50-100% headroom to these estimates for safety and to account for script memory usage.


### **Performance Tuning: Getting the Most from Your Hardware**

Zeek's performance depends on several factors: hardware capabilities, traffic characteristics, configuration choices, and script complexity. Let's explore the key performance considerations and how to optimize them.

#### **CPU Considerations**

Zeek is CPU-intensive, particularly for protocol parsing and script execution. CPU speed and architecture significantly impact performance.

**CPU requirements scale with:**

- Traffic volume (more packets = more processing)
- Traffic complexity (application protocols require more parsing than simple TCP)
- Script complexity (sophisticated detection logic uses more CPU)
- Enabled features (file extraction, statistical analysis add overhead)

**CPU optimization strategies:**

|Strategy|Impact|When to Use|
|---|---|---|
|**Higher clock speed**|Significant|Single-worker deployments|
|**More cores**|Significant|Cluster deployments|
|**Modern CPU architecture**|Moderate|New deployments|
|**Disable unused analyzers**|Moderate|Specialized monitoring|
|**Optimize scripts**|Variable|Always|

#### **Memory Performance**

Memory speed and capacity affect Zeek's ability to maintain state for large numbers of connections.

**Memory optimization:**

- Use fast RAM (DDR4-3200 or better)
- Ensure sufficient capacity to avoid swapping (swapping kills performance)
- Consider memory channels (more channels = better bandwidth)

**Connection timeout tuning:**

One of the most effective ways to control memory usage is adjusting connection timeouts. Zeek expires connection state for inactive connections based on configured timeouts:

```zeek
# Default timeouts (in /usr/local/zeek/share/zeek/base/init-bare.zeek)
redef tcp_inactivity_timeout = 5 min;  # TCP connections
redef udp_inactivity_timeout = 1 min;  # UDP "connections"
redef icmp_inactivity_timeout = 1 min; # ICMP
```

**Tuning considerations:**

```
┌──────────────────────────────────────────────────────────────┐
│            CONNECTION TIMEOUT TRADE-OFFS                     │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  SHORTER TIMEOUTS (e.g., 1-2 minutes)                        │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━    │
│  ✓ Lower memory usage                                        │
│  ✓ Faster state cleanup                                      │
│  ✗ May prematurely expire legitimate long connections        │
│  ✗ Could miss attacks that span long periods                 │
│                                                              │
│  LONGER TIMEOUTS (e.g., 10-15 minutes)                       │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━    │
│  ✓ Complete tracking of long-lived connections               │
│  ✓ Better detection of persistent threats                    │
│  ✗ Higher memory usage                                       │
│  ✗ Slower to reclaim memory from idle connections            │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

For high-volume networks where memory is constrained, shorter timeouts help. For threat hunting where you need to track persistent connections, longer timeouts are better.

#### **Disk I/O Performance**

Logging generates significant disk I/O. On a busy network, Zeek might write several gigabytes of logs per hour.

**Storage recommendations:**

|Storage Type|Performance|Use Case|
|---|---|---|
|**NVMe SSD**|Excellent|High-volume logging, logger nodes|
|**SATA SSD**|Good|Medium-volume logging|
|**HDD RAID**|Adequate|Archival storage, budget deployments|
|**Network storage**|Variable|Centralized log storage (watch latency)|




**I/O optimization strategies:**

- Use fast local storage for active logs
- Rotate logs frequently to prevent huge files
- Compress and move old logs to slower archival storage
- Consider separate file systems for logs vs OS

#### **Network Performance**

For cluster deployments, network connectivity between nodes affects performance.

**Network requirements:**

```
┌──────────────────────────────────────────────────────────────┐
│             CLUSTER NETWORK REQUIREMENTS                     │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  MANAGEMENT NETWORK (between cluster nodes)                  │
│  • Minimum: 1 Gbps                                           │
│  • Recommended: 10 Gbps for large clusters                   │
│  • Low latency critical for coordination                     │
│  • Separate from monitored network                           │
│                                                              │
│  MONITORING NETWORK (where packets are captured)             │
│  • Must match or exceed monitored network speed              │
│  • Often uses SPAN/TAP ports (receive-only)                  │
│  • No IP configuration needed (promiscuous mode)             │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### **Monitoring Zeek's Performance**

To ensure Zeek is performing well, you need to monitor its health and performance metrics.

**Key metrics to track:**

|Metric|What It Indicates|Warning Threshold|
|---|---|---|
|**Packet drop rate**|Can't keep up with traffic|>1%|
|**Memory usage**|Approaching capacity|>80%|
|**CPU usage**|Processing load|>80% sustained|
|**Disk I/O wait**|Storage bottleneck|>10%|
|**Event queue depth**|Script processing lag|>1000 events|

**Zeek provides performance statistics:**

```bash
# Check for dropped packets
zeek-cut ts dropped captured < capture_loss.log

# Monitor Zeek's internal stats
zeek-cut ts mem event_queue_size < stats.log
```


**Common performance problems and solutions:**

```
┌──────────────────────────────────────────────────────────────┐
│           TROUBLESHOOTING PERFORMANCE ISSUES                 │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  SYMPTOM: Dropping packets                                   │
│  CAUSES: CPU overload, memory exhaustion, network bottleneck │
│  SOLUTIONS:                                                  │
│    • Add more workers to cluster                             │
│    • Optimize or disable expensive scripts                   │
│    • Increase memory                                         │
│    • Improve packet acquisition (AF_PACKET, PF_RING)         │
│                                                              │
│  SYMPTOM: High memory usage                                  │
│  CAUSES: Too many concurrent connections, memory leaks       │
│  SOLUTIONS:                                                  │
│    • Reduce connection timeouts                              │
│    • Check scripts for unbounded table growth                │
│    • Add more memory                                         │
│    • Implement table expiration policies                     │
│                                                              │
│  SYMPTOM: Event queue growing                                │
│  CAUSES: Slow event handlers blocking processing             │
│  SOLUTIONS:                                                  │
│    • Profile scripts to find slow handlers                   │
│    • Optimize expensive operations                           │
│    • Remove or disable unused scripts                        │
│    • Reduce analysis granularity                             │
│                                                              │
│  SYMPTOM: High disk I/O wait                                 │
│  CAUSES: Slow storage, excessive logging                     │
│  SOLUTIONS:                                                  │
│    • Upgrade to faster storage (SSD/NVMe)                    │
│    • Reduce logging verbosity                                │
│    • Enable log compression                                  │
│    • Use separate file system for logs                       │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## **PART 4: PRACTICAL EXERCISES**

Now let's apply this architectural knowledge to practical scenarios. These exercises will help you think through deployment decisions and performance considerations.

### **Exercise 1: Design Your Network Architecture**

For this exercise, you're going to design a Zeek deployment for a hypothetical organization. This will help you think through the architectural decisions we've discussed.

**Scenario:**

You're deploying Zeek for a mid-sized company with the following characteristics:

- **Network:** 2 Gbps internet connection, 10 Gbps internal backbone
- **Endpoints:** 500 workstations, 50 servers
- **Traffic pattern:** Heavy web browsing, moderate email, some file sharing
- **Monitoring goals:** Detect C2 traffic, ransomware propagation, data exfiltration
- **Budget:** Moderate (can afford multiple servers but not enterprise appliances)

**Your task:**

Design a Zeek deployment by answering these questions and creating a diagram:


1. **Sensor placement:** Where will you deploy Zeek sensors?

    - At the internet gateway? (captures all external traffic)
    - On the internal backbone? (captures server-to-server traffic)
    - Both? (comprehensive but more complex)

   Think about what visibility you need for your detection goals. C2 detection requires seeing outbound traffic. Ransomware propagation detection requires seeing internal traffic.

2. **Cluster vs. single instance:** Will you deploy a cluster or a single Zeek instance?

    - Calculate: 2 Gbps internet + potentially 10 Gbps internal
    - Consider: A single instance can handle ~1-2 Gbps, so you likely need a cluster
3. **Cluster design:** If using a cluster, how many nodes of each type?

    - Workers: How many do you need for 2-3 Gbps total?
    - Manager: How many? (hint: usually just one)
    - Proxies: Do you need them for this scale?
    - Logger: How many? Will it be dedicated or combined with manager?
4. **Hardware specifications:** What specs for each node?

    - Consider CPU, memory, storage based on the formulas we discussed
    - Think about growth (plan for 50% traffic increase over 2 years)
5. **Load balancing:** How will you distribute traffic across workers?

    - Hardware load balancer? (expensive but performant)
    - AF_PACKET fanout? (free but requires multiple workers on same host)
    - Other approach?

**Deliverable:**

Create a document that includes:

- A network diagram showing where Zeek sensors are placed
- A cluster architecture diagram showing node types and connections
- A table of hardware specifications for each node
- Written justification for your design decisions

Spend at least 30 minutes on this exercise. There's no single "correct" answer-the goal is to think through the trade-offs and design a solution that meets the requirements.



### **Exercise 2: Plan Sensor Placement for Maximum Visibility**

This exercise focuses specifically on sensor placement strategy, which is often overlooked but critically important.

**Scenario:**

You have the following network topology:

```
                    Internet
                       │
                       ▼
                 ┌─────────┐
                 │Firewall │
                 └────┬────┘
                      │
              ┌───────┴───────┐
              │               │
              ▼               ▼
        ┌─────────┐     ┌─────────┐
        │  DMZ    │     │Internal │
        │ Servers │     │ Network │
        └─────────┘     └────┬────┘
                             │
                    ┌────────┼────────┐
                    │        │        │
                    ▼        ▼        ▼
               ┌────────┐ ┌────────┐ ┌────────┐
               │Worksta-│ │Servers │ │ Guest  │
               │ tions  │ │        │ │ WiFi   │
               └────────┘ └────────┘ └────────┘
```

**Your task:**

Decide where to place Zeek sensors to achieve these objectives:

1. **Detect external threats:** C2 traffic, inbound attacks, data exfiltration
2. **Detect lateral movement:** Compromised workstations attacking servers
3. **Detect ransomware:** Propagating between workstations
4. **Monitor DMZ:** Public-facing servers under attack

For each potential sensor placement, consider:

**Placement Option A: Between Internet and Firewall**

- What visibility: All traffic entering/leaving organization
- What you'll detect: External C2, data exfiltration, inbound attacks
- What you'll miss: Internal lateral movement, workstation-to-workstation attacks
- Traffic volume: All internet traffic (could be high)
- Not great since it is pre-NAT, unable to distinguish between different internal target hosts

**Placement Option B: Behind Firewall on Internal Network**

- What visibility: All internal traffic
- What you'll detect: Lateral movement, internal reconnaissance, ransomware spread
- Traffic volume: Potentially very high (all internal communications)

**Placement Option C: DMZ segment**

- What visibility: Traffic to/from DMZ servers
- What you'll detect: Attacks against public services, compromised DMZ servers
- What you'll miss: Most internal activity, direct workstation threats
- Traffic volume: Moderate

**Placement Option D: Multiple sensors (combined approach)**

- What visibility: Comprehensive
- What you'll detect: Everything
- What you'll miss: Nothing, but complexity increases
- Traffic volume: Must handle multiple traffic streams

**Questions to answer:**

1. If you could only deploy ONE sensor, where would you place it and why?
2. What are the minimum number of sensors needed to meet all four objectives?
3. For each sensor you propose, estimate the traffic volume it will see
4. How would your placement strategy change if:
    - Your primary concern was ransomware detection?
    - Your primary concern was APT/espionage detection?
    - You had unlimited budget vs. tight budget?

**Deliverable:**

Write a 1-2 page document explaining:

- Your sensor placement strategy with justifications
- A diagram showing sensor locations
- Analysis of what each sensor will and won't see
- Traffic volume estimates for each sensor
- How your strategy addresses each of the four objectives

### **Exercise 3: Calculate Resource Requirements**

This exercise helps you practice sizing Zeek deployments based on traffic characteristics.

**Scenario 1: Small Office**

- **Internet bandwidth:** 100 Mbps
- **Employees:** 50
- **Peak concurrent connections:** ~2,000
- **Traffic pattern:** 80% web browsing, 15% email, 5% other

Calculate:

1. How much RAM does Zeek need? (Use the formula from earlier)
2. How much disk space for logs per day?
3. How many CPU cores are recommended?
4. Can this run on a single instance or need a cluster?

**Scenario 2: Medium Enterprise**

- **Internet bandwidth:** 5 Gbps
- **Employees:** 1,000
- **Peak concurrent connections:** ~50,000
- **Traffic pattern:** 60% web, 20% cloud services, 10% email, 10% other

Calculate:

1. How many worker nodes needed?
2. Total RAM across all workers?
3. Logger disk space for logs (per day, per week, per month)?
4. Do you need proxy nodes?

**Scenario 3: Large Data Center**

- **Network bandwidth:** 40 Gbps aggregate
- **Connections:** ~500,000 concurrent
- **Traffic pattern:** Mixed (web services, APIs, databases, file storage)

Calculate:

1. How many workers needed?
2. Is this a candidate for PF_RING?
3. Total cluster memory requirements?
4. Logger capacity requirements (assuming 30-day retention)?

**Deliverable:**

Create a spreadsheet or table with your calculations for all three scenarios. Show your work so you understand how you arrived at each number. Include:

- Worker count
- CPU cores per worker
- RAM per worker (and total)
- Storage requirements
- Any special hardware (load balancers, PF_RING, etc.)

---


## **KNOWLEDGE VALIDATION**

Let's test your understanding of the architectural concepts covered in this lesson:

**Question 1: The Processing Pipeline**

Trace the path of a single HTTP packet through Zeek's architecture. Start from "packet arrives at network interface" and describe each stage it passes through, what happens at each stage, and what output is ultimately produced. This tests your understanding of the complete pipeline.

**Question 2: Event-Driven vs. Packet-Based**

Explain why Zeek's event-driven architecture is better suited for detecting C2 beaconing than packet-by-packet analysis. Think about what information is available at the event level that isn't available when examining individual packets.

**Question 3: Cluster Roles**

You're deploying a 15-worker Zeek cluster. Do you need proxy nodes? Why or why not? What factors would influence this decision? This tests your understanding of when proxies provide value.

**Question 4: Memory Management**

Your Zeek sensor is using 16 GB of RAM to monitor a 2 Gbps network with approximately 50,000 concurrent connections. Is this reasonable? If you needed to reduce memory usage, what configuration changes could you make? This tests your ability to apply the memory sizing formulas and understand tuning options.

**Question 5: Performance Troubleshooting**

You notice your Zeek sensor is dropping 5% of packets. What are the three most likely causes, and what would you check to diagnose each one? How would you fix each problem? This tests your understanding of performance bottlenecks.

**Question 6: Sensor Placement**

You want to detect lateral movement (compromised workstations attacking internal servers). Should you place your Zeek sensor at the internet gateway or on the internal network? Justify your answer. This tests your understanding of how sensor placement affects visibility.

Take time to think through these questions. If you're unsure about any of them, review the relevant sections before moving on.

---

## **PREPARING FOR LESSON 1.3**

Congratulations! You've completed the architecture deep dive and now have a comprehensive understanding of how Zeek works internally. Let's summarize what you've learned and prepare for the next lesson.

**What You've Mastered:**

You now understand Zeek's complete processing pipeline, from packet acquisition through protocol analysis to event generation and script execution. You've learned why the event-driven architecture enables behavioural analysis that would be impossible with packet-level inspection. You understand how Zeek maintains rich state about connections and protocols, providing the context necessary for sophisticated threat detection.

You've explored Zeek's cluster architecture, understanding how multiple nodes work together to analyze high-bandwidth networks. You know the roles of workers, managers, proxies, and loggers, and how they coordinate through the Broker communication framework. You understand load balancing strategies and can design cluster deployments for different scales and requirements.

You've learned about memory management and performance optimization, understanding how Zeek uses memory, how to size deployments appropriately, and how to tune configuration for optimal performance. You can diagnose performance problems and understand the trade-offs in different optimization strategies.

Most importantly, you've applied this knowledge through practical exercises, designing network architectures, planning sensor placements, and calculating resource requirements. These are the skills you'll use when deploying Zeek in real environments.

**What's Coming in Lesson 1.3:**

In the next lesson, we're moving from theory to hands-on practice. You're going to install Zeek on your Ubuntu droplet, going through the complete installation and configuration process. We'll cover three different installation methods (package manager, source compilation, and containers) so you understand the trade-offs and can choose the right approach for different scenarios.

You'll configure network interfaces for monitoring, set up AF_PACKET for improved performance, and make your first captures of network traffic. You'll learn to start and stop Zeek, monitor its operation, and troubleshoot common issues. By the end of Lesson 1.3, you'll have a fully functional Zeek sensor capturing and analyzing traffic.

**Before the Next Lesson:**

Make sure your Digital Ocean droplet is ready. If you haven't created it yet, do so now:

- Ubuntu 22.04 LTS
- At least 4 vCPUs
- At least 8 GB RAM
- At least 80 GB storage
- Note the IP address for SSH access

Think about what network traffic you'll analyze. Do you have:

- A home network you can monitor?
- Lab VMs you can use to generate traffic?
- Sample PCAP files for analysis?

Having traffic to analyze will make the next lesson much more engaging than just installing software.

Review your notes from this lesson, especially the cluster architecture and performance tuning sections. This knowledge will inform the configuration choices you make during installation.

**Final Thought:**

Architecture might seem like dry material, but it's the foundation everything else is built on. When you're writing detection scripts in later modules and wondering why something works the way it does, you'll refer back to this architectural knowledge. When you're troubleshooting a production issue, understanding the architecture will help you diagnose and fix it quickly. The time you've invested understanding Zeek's internals will pay dividends throughout your career.

Take a break, let this knowledge settle, and when you're ready, we'll dive into Lesson 1.3 and get our hands dirty with an actual Zeek installation!

---



















