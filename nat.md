## NAT

### Policy SNAT

Policy SNAT can be configued to use the _Outgoing Interface Address_ (1) or a _Dynamic IP Pool_ (2).  

(1) Fortigate uses the egress interface address as the NAT IP for performing SNAT. If there are more than one device behind the interface, Fortigate performs a many-to-one NAT, known as PAT.  
If the source port cannot be translated due to a static configuration (source port must be the one configured) only a connection using that port can be established.

(2) IP Pools define a single address or a range of addresses to be used instead of the IP assigned to the egress interface (usually they are in the same subnet).
There are four types of pools:
- Overload
- One-to-one
- Fixed port range
- Port block allocation

IP Pool - Overload: similar to PAT but allows to define a range of IP addresses.  
IP Pool - One-to-one: NAT acts on a first come, first served basis meaning that IP addresses are assigned, one to each client, until the pool is exhausted.  
IP Pool - Fixed port range: few IP address to many internal IP addresses, blocks of ports are used as session identifier and allow administrators to track sessions without logging (for given internal IP address are assigned a fixed range of ports to use). IP addresses are randomly assigned as connections are made.  
IP Pool - Port block allocation: administrators are required to define the external IP address only, the port bock size and the number of blocks. 

### Central SNAT

Once enabled it simplifies firewall policy configuration.  
NAT rules are not assigned to firewall policies which will now reference destinations.
 
Central SNAT policies can, for example, be defined per protocol and source ip, mapping address translation to some (overloaded) ip-pools or translating to the external interface IP address.  