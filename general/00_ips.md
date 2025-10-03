## ACM List

Reference the IP in the ACM hint list and see if there is an entry for it. (TODO: Link)

## ipinfo.io

Use [ipinfo.io](http://ipinfo.io/) to look at a variety of important pieces of information related to the IP.


### ASN, ASN type, and Company
- This tells us who controls the network this IP address lives on, which helps to frame the context of the connection. 
- Is this a residential internet connection, a university, a business, or a server in a data center? Adversaries often use servers from hosting providers/data centers for their infrastructure.
- If the Company was "DigitalOcean," "OVH," "Hetzner," or any other VPS/dedicated server provider, one should have a closer look. While millions of legitimate websites use these, they are also the primary choice for attackers to quickly stand up malicious servers.
- If the Company is a known "bulletproof" hoster (one that ignores abuse complaints), this would be a major red flag. See [Spamhaus Project](https://www.spamhaus.org/blocklists/do-not-route-or-peer/) for lists of known BPHs.


### Hostname
- Many hastily configured malicious servers don't have a reverse DNS (PTR) record set up, so this field would be blank. In other words an outbound connection directly to an IP, and not a domain, should be viewed with some level of suspicion. 
- A hostname like `c2.evil-domain.ru` or `update.system-check.xyz` is an obvious indicator of malicious intent.
- A hostname like `server1.some-random-hosting.com` or a random string of characters might indicate a generic, non-production server, which is more typical of temporary malicious infrastructure.
- Something else to look for is the number of domains associated with the IP - a very high number of hostnames (like >1000) associated with a single IP address can be an indicator of something malicious, but it requires context.



### Geolocation
- While not always definitive, the geographic location provides crucial context.
- It helps you answer the question: "Do we have any business connecting to a machine in this location?"
- In particular, if the company was a Russian or Chinese hosting provider and your business has no reason to connect to such services, it would be suspicious.

  
___

[BACK TO MAIN](../README.md)