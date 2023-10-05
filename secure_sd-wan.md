## Secure SD-WAN

Secure SD-WAN is the Fortinet SD-WAN solutions. Is is named _Secure_ because it embeds FortiOS security features and allows administrators to define how Fortigate steers traffic across the WAN based on multiple factors such as the protocol, service or application identified and the quality of the links. SD-WAN controls egress traffic not ingress traffics, meaning that ingress traffic may use a different link from the one SD-WAN chose for egress.  

One benefit of SD-WAN is effective WAN usage. that is administrators can use public (broadband, LTE ...) and private (MPLS ...) links to securely steer traffic to different destinations: internet, public cloud, private cloud and internal network. This _hybrid_ approach reduces costs mainly because administrators tend to steer most of the traffic to low-cost fast link than high-cost slow link (meaning that private links, such as MPLS, are used for critical traffic only or as failover link.)  

Another benefit of SD-WAN is an application experience improvement because application traffic is steered to the fastest links or those meeting the application requiremenets.  

During traffic congestation, traffic shaping helps in prioritizing sensitive and critical applications over less important ones. Also the support of ADVPN (auto-discovery VPN) shortcuts enables SD-WAN to use direct IPSEC tunnels between sites to steer traffic, resulting in lower latency for traffic between the sites (spokes) and the central locations (hubs).

The most common use case is **direct Internet access** (DIA).  
A site has more than one Internet link and administrators steer internet traffic across the links (members). 

Usually, the best performing links are used for sensitive traffic while non sestitive traffic is distrubuted across one or more links using a best effort approach. Costly internet links are commonly used as backup routes or for critical traffic only. 

A typical configuration makes use of static default routes. However in some cases BGP is used between the ISP and FortiGate. 

**Network** > **SD-WAN** > **SD-WAN Rules**.  
Software-defined rules requires a source, a matching criteria and one or more members to steer traffic to (if the perfroming metric is specified, it indicates the member eligiiblity for the rule).  
The are eveualted from top-to-bottom and **their purpose is to steer traffic, not allow/disallow traffic**.  
That means firewall policies are required to allow SD-WAN traffic.  

The implict SD-WAN rule instructs FortiGate to perform standard routing on traffic.  
SD-WAN rules are policy routes, which means they are installed in the routing table, have precedence over FIB entries but not over regular policy routes.  

When SD-WAN is enabled, the ECMP algorithm is no longer controled via the command `v4-ecmp-mode` which is replaced by the command `load-balance-mode` under `config system sdwan`.  
Differences between the two:
- `load-balance-mode` supports the volume algorithm, `v4-ecmp-mode` does not
- `load-balance-mode` uses the weight defined in the SD-WAN member configuration, `v4-ecmp-mode` uses the weight defined in the static route
- `load-balance-mode` uses the spillover thresholds defined under the SD-WAN member configuration, `v4-ecmp-mode`the spillover thresholds defined in the interface settings

A SD-WAN member volume is used in conjuction with the weight to determine how sessions are load-balanced between members: the higher the volume and the weight, the more sessions are sent to the member.  
The volume is the cumulative number of bytes tracked by FortiGate per member.