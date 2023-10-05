## Application Control

Application control can be configured in proxy-based and flow-based firewall policies but, since **application control relies on the IPS engine** which uses flow-based inspection, inspection is always flow-based.  

The IPS engine used by Application Control matches patterns in the entire byte stream of the packet and then look for patterns.

The IPS engine cannot be bypassed using HTTP proxys.  
Most applications use a P2P logic, meaning they must work with none or minimum user (and administrator) interaction. Bypassing firewalls make them hard to detect. They do not depend on port forwarding.

The IPS engine can analyze packets for pattern matching and then look for patterns to detect P2P applications.  

Each peer in a P2P network is a server with a piece of the global information and small bandwidth to share.  
The more peers offer information the faster the download will be but the requesting peer can consume much more bandwidth than it would downloading the entire file from a single server.  

Fortigate application control signatures are part of the standard Forticare subcription plan which also includes updates to the IPS engine (holding the signatures). Settings about scheduled updates can be found in **System** > **Fortiguard**.  

Fortiguard research team analyzes applications prior to build signatures and assign the application a risk level which is Fortinet-only, not related to CVSS or other external system.  
[IPS signature online database](https://www.fortiguard.com/updates/ips).

The Fortiguard application control signatures are fine-tuned in order to let administrators blocks application (components) with more granularity: for example, blocking Facebook posting features but allowing Facebook chat platform.  

**IPS DATABASE AND APPLICATION CONTROL DATABASE ARE NOT THE SAME THING**

Application control profiles can be used when the Fortigate work in profile-based + flow-based inspection or profile-based + proxy-based inspection (note that the IPS engine **always** works in flow-based mode).  
Most cloud applications require SSL which cannot be inspected if full SSL inspection is not configured for the policy the profile will be applied to, and in general in the network.  
Applications that are not matched to a pattern are marked as **unknown application**.  

Google's protocol QUIC uses UDP for web browsing instead of TCP. Application control can block QUIC (looking for specific headers in packets) and force Google Chrome to use HTTP2/TLS1.2. Fortigate will log QUIC requests as blocked.

For blocked HTTP/HTTPS-based applications, Fortigate can display to users custom webpages. Non-HTTP/S-based applications are simply drop (drop packets or reset TCP connection).  

**Network protocol enforcement** allows blocking or monitoring of known services on unknown ports.

The IPS engine examines the traffic for a signature match, then Fortigate scans packets for matches, evaluating application or filter overrides as step number 1 and application categories as step number 2.  

Application control is performed **before** web filtering.  
Web filtering **can** block applications allowed by application control.  
Web filtering **cannot** bypass applications control actions.  

FortiOS uses a three-step process to perform NGFW policy-bases application filtering:

1. Allows all traffic while forwarding packets to the IPS engine. At the same time, FortiOS adds an entry in the session table allowing the traffic to pass and it adds a `may_dirty` flag to it

2. As soon as the IPS engine identifies the application, updates the session entry with flag `dirty` (instructs the kernel to re-evealuate the session entry) or with flag `app_valid` (indicates the IPS engine has validated the traffic) and and application ID

3. The FortiOS kernel performs a security policy lookup again to see if the application ID is listed in one of the policies. This time the kernel uses both layer4 and layer7 for policy matching.  
After the criteria matches a firewall policy rule, the FortiOS kernel applies the action configured on the policy to the application traffic

Since policy-based requires the use of Central SNAT, it is very important to arrange security policies so that the more specific policies are located at the top to ensure proper use of application control.

If administrator don't want to block applications, they can rate limit their bandwidth usage, such as for youtube videos.  
Under **Policy & Objects** > **Traffic Shaping** > **Traffic Shaping Policies**, it is possible to configure traffic shapers.  
**Shared shaper**: applies a total bandwidth to all traffic using that shaper  
**Per-IP shaper**: applies traffic shaping to all source IP addresses in the security policy, equally divided among the group  

A Shared shaper is applied from ingress to egress (useful to limit uploads) while a Reverse shaper is aplied from egress to ingress (useful to limit downloads)

Some best practices to follow:
- Do NOT apply application control scanning to internal-only traffic
- Apply application control only where needed
- Apply the same control policies on redundant/failover connections
- Policies must be as specific as possible
- Deep Inspection for SSL/TLS traffic (HTTPS...)