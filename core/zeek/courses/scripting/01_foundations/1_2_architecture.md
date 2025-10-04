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


