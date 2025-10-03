## Google

Search for the following terms to get general info related to a specific port:
- `tcp port nnnn`, or
- `udp port nnnn`, or
- `firewall tcp port nnnn`, or
- `firewall udp port nnnn`, or


Search for the following terms to see if the port has been associated with any specific malware/campaigns. This is only applicable for unconventional ports, say 7707, not established ports like 443. 
- `port nnnn malware`, or
- `tcp port nnnn malware`, or
- `udp port nnnn malware`



## Wikipedia
- [See link](https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers) for a basic list of common ports.  
- There's no such thing as an exhaustive, complete, and perfect list.


## Zeek
- Zeek provides insightful reporting on the Connection State. 
- You can find information on this [here](https://docs.zeek.org/en/current/scripts/base/protocols/conn/main.zeek.html#type-Conn::Info). 

  

## Non-standard ports

- See [this link](https://attack.mitre.org/techniques/T1571/). 
- Adversaries may communicate using a protocol and port pairing that are typically not associated. For example, HTTPS over port 8088 as opposed to the traditional port 443. 
- Adversaries may make changes to the standard port used by a protocol to bypass filtering or muddle analysis/parsing of network data.
- Adversaries may also make changes to victim systems to abuse non-standard ports. For example, Registry keys and other configuration settings can be used to modify protocol and port pairings.

## Standard Ports, Non-Standard Traffic (Masquerading)

- Adversaries know that most firewalls allow outbound traffic on ports 80 (HTTP) and 443 (HTTPS) by default. They abuse this trust by tunnelling non-web traffic through these ports.
- Zeek can be useful to generate these alerts for example "Non-SSL traffic on port 443".
- Remember that any port can be used for any service!  The fact that you're seeing (say) UDP port 123 does not mean it's actually transmitting NTP traffic.  
- This is why Zeek's report of the actual protocol it's seeing is so valuable; Zeek is actually checking the payload to confirm it's actually NTP.

## Standard Ports, Unexpected Traffic

- Sometimes one might encounter an unusual port that is the standard for the application, but the application is not expected on the target network.
- For example one might have a web server in a DMZ that is communicating outbound via port TCP 11601.
- Googling this specific port one would find that it belongs to Ligolo-ng, an open-source tunneling application.
- The question then becomes - is there a specific use case for running an open-source tunneling application on the web server in question? 
- Do note however that the use of port 11601 is not definitive proof that it is Ligolo-ng, equally an attacker might use Ligolo-ng and use a different port altogether. 

  

## Unexpected Listening Ports on End-user Workstations

- In general, we expect end-user machines to behave as clients - they should be initiating connections outwards. 
- Any instance where they are acting servers and listening for inbound connections should be subjected to closer scrutiny. 
- This is best found with an Endpoint Detection and Response (EDR), or Network Detection and Response (NDR), tool that monitors running processes and their network sockets. 
- You can also spot this manually on a machine by running `netstat -anb` (Windows) or `netstat -anp` (Linux) to see which processes are in a LISTENING state.

___

[BACK TO MAIN](../README.md)