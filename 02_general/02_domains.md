

## ACM List

Reference the hostname in our list and see if there is an entry for it.



## Google

Search for "scam domain_name" and see what comes up.



## whois.com

Use [whois.com](http://whois.com/) to look at a variety of important pieces of information related to the IP.



### Domain Age (Registered On)

- Trust is established over time -> the younger the domain, the more suspicious you should be. 
- Cybercriminals operate on a "cheap and disposable" model, they rarely invest in or use old domains.
- If a domain was registered within the last 90 days, it should be treated as highly suspicious until proven otherwise. It has no history and no established trust - this is the hallmark of a phishing or malware campaign.

### Registrant Information (and Privacy Services)

- While hiding one's identity isn't automatically malicious, the way it's hidden provides significant clues.
- For example it's extremely common for legitimate businesses to use "Domains By Proxy, LLC", GoDaddy's well-known and widely used domain privacy service. For a valuable domain like this, using a major registrar's privacy service is a standard and expected security practice. It prevents the owner's real name, address, and phone number from being harvested by spammers and scammers.
- What could potentially be suspicious then is when the registrant details are not private, but they are clearly fake (e.g., Name: "Domain Owner," Address: "123 Fake Street," City: "New York," Phone: "555-555-5555"). 
- Or, if the details are public but don't make sense together (e.g., a US address with a Russian phone number).
- Finally, also be on the lookout for the use of BPHs here, see [Spamhaus Project](https://www.spamhaus.org/blocklists/do-not-route-or-peer/).



## Registrar Information

- Consider the reputation of the registrar - for ex. GoDaddy.com with well-established abuse policies vs. a BPH with no abuse policies. 



### Domain Status Codes

- This indicates whether the domain is active, locked, expired, or has been taken down for malicious activity.
- The status includes multiple client... prohibited flags. This is a sign of good security. These are locks put on the domain by the owner (via the registrar) to prevent it from being hijacked (i.e., deleted, updated, or transferred without authorization). This shows the owner is protecting their valuable asset.
- Conversely, serverHold or clientHold are considered "punitive" - the registrar or registry has actively taken the domain offline. This is often done in response to confirmed malicious activity (phishing, C2) or non-payment. 

  
  