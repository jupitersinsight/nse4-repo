Firewall traffic (also known as user traffic) s generated by clients.  
Local-out traffic is generated by FortiGate, examples are ping requests and requests to FortiGuard.  

Routing on Fortigate preceeds security actions since policies are applied to interfaces: they are applied per interface/destination.  

Routing information are stored in two tables:
- **RIB**: the Routing Information Base contains active (or the best) connected, static and dynamic routes
- **FIB**: the Forwarding Information Base is the routing table seen from the kernel's point of view, it built mostly out of RIB entries plus some system-specific entries required by FortiOS  

**When Fortigate performs a route lookup it inspects the FIB** not the RIB.  
RIB entries can be displayed both via GUI or CLI while FIB entries only via CLI.

Foreach session Fortigate performs two route lookups:
- for the first packet sent by the originator
- for the first reply packet received from the responder

After that, Fortigate writes the routing information in its session table. Subsequent packets are routed using the information from the session table, not from the routing table. All packets that belong to the same session follow the same path.  
If there is a change in the routing table that impacts the session, the routing information is removed from the session and Fortigate performs a new route lookup to rebuild the information.  

Policy routes take **precedence** over information from the routing table and that is why policy routes must be as much accurate as possible.  
When matching criteria of policy routes matches, Fortigate performs two actions, either forward the traffic (routing packets to the specified destination/gateway) or stop using the policy routes and the session is handled using the FIB. **Network** > **Policy Routes**.

Fortigate contains a so-called ISDB database (Internet Service Database) which include objects to services with lots of entries (IPs, protocols, port numbers...) such as cloud platform AWS, Azure...  
These objects can be referenced in static routes which act as policy routes. In this way it is possible to route traffic from/for specific application such as AWS or Netflix to and from specific WAN interface/address.  

The lower the distance of a route to a specific destination, the better it is in routing terms. This route will be installed in the routing table while other routes to the same destination but higher distance are installed in the routing table database.  

When Fortigate learns two equal-cost routes to the same destination but that are sourced from different (routing) protocols, in the routing table is installed the route that was learned last. It is best practive to **NOT** change distance to protocol to avoid installing the same route with same distance but from two different protocols. This misconfiguration can decrease performance of network traffic and cause protocol-related communications and synchronization issues. 

|type|distance|
|----|--------|
|Connected|0|
|Static (SD-WAN Zone)|1|
|Static (DHCP)|5|
|Static (Manual)|10|
|Static (Ike)|15|
|EBGP|20|
|OSPF|110|
|IS-IS|115|
|RIP|120|
|iBGP|200|

Metric is another value which routing relies on: the lower the metric the higher the preference, meaning the route will be installed in the routing table. Routes to the same destination with higher metric are installed in the routing database.  

**The Administrative Distance is used to determine the best route among routes learned through DIFFERENT protocols while the Metric is used to determine the best route among routes learned through the SAME protocol.**  

When there are two or more static routes that have the same distance, Fortigate installs all of them in the routing table. If they also have the same priority then the routes are known as ECMP (Equal Cost Multi-Path) routes.  
The lower the priority the higher the preference. 
The priority value **only applies** to static routes.  

ECMP routes are all installed in the routing table and Fortigate load-balance traffic among them.  

ECMP can load balance sessions using one of these algorithms:
- **Source IP**: (default) sessions sourced from the same IP address use the same route
- **Source-Destination IP**: sessions with the same source and destination pair use the same route
- **Weighted**: applies to static routes only, sessions are distributed between routes. The higher the weight of the route or interface, the more sessions are routed through the selected route
- **Usage (spillover)**: the first ECMP route is used until the specified bandwidth limit is reached, exceeding sessions are routed using the second ECMP route

ECMP configuration is done through the CLI.


## RPF

RPF check is a mechanism meant to protect a network from IP spoofing by checking for a return path to the source in the routing table.  

If FortiGate receives a packet to an interfcae whose source IP address is not reachable via routes installed in the FIB, it is 
likely to be a spoofed IP address.

Fortigate performs RPF check only on the first packet. If that packet is passes the check, then the entire session is allowed with no further RPF checks.  

RPF checks can be performed in two ways:

- **Feasible path**: FortiGate verifies that the incoming IP address and interface match an existing route in the routing table. The matching route doesn't have to be the best route.
- **Strict**: Fortigate verifies that the incoming IP address and interface match an existing route in the routing table which has to be the best route.


## Link Health Monitor

Link Health Monitor is a feature which allows FortiGate to send probes to a set remote gateway or server (best practice is to used at least to routers/servers). A total of 5 missing replies (or as much as set by the administrator) means the link is dead and FortiGate performs any set action, such as update the routing table or policy routing.  
Those actions are reverted back when a total of 5 good replies is received on a link, meaning it is alive.

Ping is the most used network protocol because it is supported by virtually all networks. Some providers though may configure their routers to drop ICMP echo requests. In those cases other protocols such as TCP echo, UDP echo and TWAMP (Two-Way Active Measurement Protocol) should be used.  

TCP echo and UDP echo both listen on port 7, a succesfull reply is a copy of the original request.  

TWAMP uses two sessions, one for authentication and one for quality testing of the link. By default authentication is disabled so Fortigate only generates the test session. Default port on Fortigate is 802, used by both sessions.  

HTTP is the last protocol that can be used for link monitoring. HTTP GET requests are sent out periodically, Fortigate waits for responses and, if configured can search for a specific string the HTML body of the response.  

