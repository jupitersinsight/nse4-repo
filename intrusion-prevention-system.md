## Intrusion Prevention System

- **Exploits**: exploits are known attack, with known patterns that can be matched by IPS, WAF or antivirus signatures
- **Anomalies**: anomalies are unusual behaviors in the network, such as higher-than-usual CPU usage or network traffic, that must be detected and monitored (and in some cases blocked or mitigated) because they may be symptoms of new emerging threats. Useful features: rate-based IPS signatures, DoS policies, protocol constraints inspection


The IPS engine is responsible for application control, web filtering, flow-based antivirus and email filtering, IPS and protocol decoders.

**Protocol decoders parse each packet according to the protocol specifications.** Some protocols decoders require a port number specification (configured on the CLI) but usually the protocol is automatically detected. If the packet does not conform to the specification (for example, it contains malformed data or invalid commands), the decoder detects the error (example, too many HTTP headers in request or BoF attempt). 

IPS often does not need IANA standard ports for protocol identification because decoders are automatically selected based for protocol at each OSI layer.   

IPS signatures are updated regularly (an initial set is bundled with every Fortigate) via Fortiguard. Protocol decoders are rarely updated because RFCs tend to remain the same for very long time. 

Updates frequency is different for each model of Fortigate but max interval time is one hour.  

IPS s a Fortiguard subscription and includes a botnet signature database. The botnet IP database is part of the ISDB updates. The botnet domain database is part of the AV updates. Only botnet signatures require the Fortiguard IPS license subscription. 

The IPS signature database is divided into two:

- the **regular** database which includes signatures for common attacks, those signatures cause rare or no false positives. It's a smaller database and its default action is to block the detected attack
- the **extended** database contains signatures for attacks that cause a significant impact on performance or don't support blocking. Useful in high-security networks, but not all Fortigate models support it (disk and RAM limits)

IPS sensors can be configured (**Security Profiles** > **Intrusion Prevention**) adding singatures to it o adding groups (or categories) of signatures to it.  
It is possible to block specific traffic matching a given signature or category for a duration (in seconds) when a set threshold is exceeded.  

The order of signatures in IPS sensor filter has the same meaning of orders for firewall policy.  Signatures are evaluated from top to bottom so placing the most specific or purpose-related signatures on top make Fortigate identify threats faster, reduces CPU usage and RAM usage.

IP addresses can be exempted from IPS scanning but only per signatury (not per category or group of signatures).

Actions on signature matching are:

- **Allow** : traffic is allowed
- **Block** : traffic is blocked
- **Monitor** : logs signature matches
- **Default** : default action for the signature
- **Reset** : reset the connection sending a TCP RST
- **Quarantine** : the IP of the attacker is blocked for a specified duration

If **Packet Logging** is enabled, Fortigate saves a copy of the packet that matches the signature.

IPS signature filter options include CVE pattern. The CVE pattern allows administrator to filter IPS signatures based on CVE IDs with a CVE wildcard ensuring that any signatures tagged with that CVE are automatically included. 

As part of Fortiguard IPS, administrators can enable scanning of botnet connections to maximize internal security.  

DoS protection/filtering is done early in the packet handling process, which is handled by the kernel.  
DoS sensosrs can identify different anomalies and act accordingly.  
DoS protection can be applied to four protocols: TCP, UDP, ICMP and SCTP.  
For each protocols there are four type of anomaly detection:
1. A flood sensor dectecs high volume of that specific protocol, or signal in the protocol
2. A sweep/scan detects probing attempts to map which of the host ports respond and might be vulnerable
3. Source signatures look for large volumes of traffic originating from a single IP address
4. Destination signatures look fo large volumes of traffic destined for a single IP address

Best practice is to first monitor and log network traffic (DoS policy) to identify a baseline from which tune a DoS policy that wouldn't impact on daily usage of network and resources.  

Limit threshold are expressed as sessions/packets per second (for anomalies 1 and 2) or as sessions (for anmalies 3 and 4).  
A threshold too high, can cause a DoS attack to impact performances, while a threshold too little can cause Fortigate to drop legit traffic.  

**Best practices**  
Create a network baseline. Know the network and the resources in it and fine-tune IPS sensors accordingly (example, if only Windows-machines are in use in the network, configure IPS only with Windows-related signatures). Unnecessary operations decrease the overall performance of the appliance. 
Some signatures are designed to protect clients while others are designed to protect servers.

IPS is not a **set-and-forget** implementation. Logs must be reviewed regularly to detect anomalous traffic patterns and adjust IPS profiles accordingly.  
It is also important to perform internal audit to identify if certain vulnerabilities still apply to the organization.  

Avoid implementation of IPS on internal-to-internal policies.  

Since some signatures only works against encrypted traffic, IPS (and WAF) works best when a Full SSL Inspection profile is in use.  

**Troubleshoot**
Fortiguard IPS updates relies on connection to `update.fortiguard.net:443`... any issues related to updates is mainly due to problems reaching that resource.   

Command `diagnose test application ipsmonitor <integer>` is useful while investigating high CPU usage issues related to IPS processes.  
(Option 2 toggle disable/enable IPS, option 5 toggle IPS bypass traffic analysis, option 99 restart all IPS engines and monitors).

IPS goes into fail-open when there is not enough available memory n the IPS socket buffer for new packets. If the `fail-open` settings is enabled, some new packets can pass through without being inspected. If it is disabled, new packets are dropped. 