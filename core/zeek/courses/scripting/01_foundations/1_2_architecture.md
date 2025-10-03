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

