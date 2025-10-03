

## **Configuration Deep Dive**

Suricata's behaviour is controlled by `/etc/suricata/suricata.yaml`- a YAML configuration file with hundreds of options. Understanding key sections is essential for effective threat hunting deployment.

### **Core Configuration Structure**

The `suricata.yaml` file is organized into logical sections:

1. **vars**: Network and port variables
2. **default-log-dir**: Where logs are written
3. **stats**: Statistics collection and reporting
4. **outputs**: Log formats and destinations
5. **af-packet / pcap / nfqueue**: Packet acquisition methods
6. **stream**: TCP stream reassembly settings
7. **app-layer**: Protocol parsers and settings
8. **detect**: Detection engine tuning
9. **threading**: CPU and threading configuration