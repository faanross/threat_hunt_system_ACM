
## **Configuration Deep Dive**

### Overview


Creating, maintaining, and refining the configuration file is arguably the most important skill when working with Sysmon. While you must obviously be skilled at locating and interpreting logs during investigations, the logs themselves are only as useful as the configuration that generates them.

As mentioned in the previous section, the SwiftOnSecurity config provides an excellent baseline to start with. It will get you up and running quickly and offers a significant improvement over the default configuration - or not running Sysmon at all. However, once Sysmon is operational with that config, you should immediately familiarize yourself with each section and learn to confidently make changes suited to your specific environment and threat hunting approach.

There are two key areas of knowledge required:

- Understanding the language (XML) and logical structure of the config, including rule syntax
- Understanding the 29 specific Event IDs, their relevance to threat detection, and how to write rules that continuously optimize their value

In this section, we'll focus primarily on the former. Following this, we'll cover each Event ID in depth, discussing both its relevance to threat hunting and how to write specific rules for it.





### **Understanding Sysmon Configuration Structure**

Sysmon's behaviour is controlled by an XML configuration file that defines:

1. Which event types to monitor
2. What filters to apply (include or exclude specific events)
3. What additional context to capture (hashes, network details, etc.)

The configuration uses a two-phase filtering model:

**Include Filters**: Only events matching these rules are logged  
**Exclude Filters**: Events matching these rules are discarded

This allows for flexible rule creation. You might include all network connections (Event ID 3) but exclude connections to known-good internal services to reduce noise.
