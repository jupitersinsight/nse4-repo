## IPSec VPN

IPSec is a vendor-free standard that allows data encryption and IPSec VPNs provide:
- authentication to verify the identity of both ends
- data integrity (or HMAC) to prove that encapsulated data has not been tampered with as it crosses a potentially hostile network
- confidentiality (or encryption) to make sure that only the intended recipient can read the message

**IPsec is a suite** of different protocols:
- **Internet Key Enchange** (IKE): used to authenticate peers, exchange keys and negotiate the encryption and checksums that will be used. **IKE = Control Channel**
- **AH**: the Authentication Header, the checksum that verify the integrity of the data
- **Encapsulating Security Payload** (ESP): the encapsulated security payload, the encrypted payload. **ESP = Data Channel**

**Since AH does not provide encryption, FortiGate does not use AH. Therefore, there is no need to allow _AH IP protocol (51)_.**
___
### How IPSec works?

Ports needed:  

**NAT-T**
- IKE UDP (IP protocol 17):
    - UDP 500 ([RFC 2409](https://datatracker.ietf.org/doc/html/rfc2409) IKEv1)
    - UDP 4500 ([RFC 4306](https://datatracker.ietf.org/doc/html/rfc4306) IKEv2) for rekey, quick mode, mode-cfg
- ESP UDP (IP protocol 17):
    - UDP 4500 ([RFC 4303](https://datatracker.ietf.org/doc/html/rfc4303)) for encapsulation  

**No NAT**
- IKE UDP (IP protocol 17)
    - UDP 500 (IKE)
- ESP UDP (IP protocol 50)


On the CLI administrators can specify a custom port both for IKE and NAT-T, FortiGate will always listens on port 4500 regardless to be able to negotiate NAT-T tunnels.  

IPSec works at Layer 3 (Network Layer). During tunnel establishment, both ends negotiate encryption and authentication algorithms to use.  
After the tunnel has been negotiated and is up, data is encrypted and encapsulated into ESP packets.  

_Encapsulation_  
- **IPSec transport mode: TCP/UDP wrapped**. Encapsulates and protects Layer 4 and above. The original IP header is not protected and no additional IP header is added.
- **IPSec tunnel mode: additional IP layer, TCP/UDP wrapped**. The whole IP packet is encapsulated and a new IP header is added at the beginning. After the IPSec packet reaches the remote LAN and is unwrapped, the original packet can continue on its journey.   

**A VPN-related transport mode packet cannot be transmitted any further once it is unwrapped. That is due to the lack of a second IP header making the packet not routable. For that reason, this mode is usually used only for end-to-end (or client-to-client) VPNs.**  

_Negotiation_  
- Authentication
- Handshake to exchange keys, settings

___
### What is IKE?

IKE uses UDP port 500. If NAT-T is enabled in a NAT scenario, IKE uses UDP port 4500.  

IKE establishes an IPSec VPN Tunnel. FortiGate uses IKE to negotiate with the peer and determine the IPSec security association (SA).  
The IPSec SA defines the authentication, keys and settings that FortiGate uses to encrypt and decrypt that peer's packets. It is based on the Internet Security Association and Key Management Protocol (ISAKMP).  

IKE defines two phases: phase 1 and phase 2.  
IKEv2 is simpler but the old IKEv1 is used the most. 

IKEv2 supports many extensions, such as timed re-authentication and quick-crash-detecion.  
IKEv1 does not support those extensions, but Fortinet did implemented a custom vendor ID (which works only between Fortigates and with Main Mode because encryption is needed) which allows the use of QCD technology in IKEv1.  

IKEv1 fragmentation is enabled by default and may prove useful when large PSK or certificates are used which push the packet size beyond 1500 bytes (max MTU).  
FortiGate performs IKE fragmentation only when it is explicitly set in phase1 options, the packet is larger than the minimun MTU size (576 bytes for IPv4 and 1280 for IPv6), the packet is being re-transmitted.  

IKEv2 fragmentation requires each fragment to be encrypted and if set, larger packets are preemeptively fragmented and ecnrypted.
___
### Negotiation - Security Associations

A Security Association is the bundle of algorithms and parameters being used to encrypt and authenticate data travelling through the tunnel.  

Both sides of the tunnel must agree on the security rules; in a normal two-way traffic this exchange is secured by a pair of SAs, one for each direction.  
No agreement, no tunnel.  

IKE uses two phases: IKE phase 1 and IKE phase 2.  
Each phase negotiates different SA types. The SA negotiated during phase 1 is named IKE SA, while the SA negotiated during phase 2 is IPSec SA.  
FortiGate uses IKE SAs to secure channels to negotiate IPSec SAs, and uses IPSec SAs to encrypt and decrypt the data sent and received.  

FortiGate matches the most secure proposl to negotiate with the peer.  


___
### VPN Topologies - Remote Access

Remote users (dial-up) with dynamic IP addresses using FortiClient or supported software to establish a VPN connection with FortiGate (set as a dial-up server), only clients can initiate the VPN.  
One VPN profile can fit many remote users.  

Remote users may be reuired to authenticate themselves against a username and password request (xauth) or against a certificate check (X.509). In this case administrators need to issue a unique certificate for each user. The certificate can be verified by the subject field, common name or the principal name in the Subject Alternative Name (SAN).


### VPN Topologies - Site-to-Site

One-to-One VPN connection is the simplest scenario. Multiple locations can be connected based on hub-and-spoke or meshed topologies.  

Hub-and-spoke topologies requires branch offices to connect to a single HQ. FortiGate at HQ must be powerful enough to maintain all spokes' VPNs at the same time. If HQ fails the VPN failure is company-wide. Branch office to branch office traffic might be slow, especially if HQ is far away.  
This topology cause little administration overhead.  

Full-mesh topologies mean every location is connected to every other location. More powerful FortiGates needed, more overhead, more expensive but faster and fault-tolerant.  
Partial-mesh topologies, useful compromise when not every location needs to connect to every other location directly.

**Auto-Discrovery VPN** (ADVPN) is a FortiGate feature that aims at making full-mesh topologies administration easier.  
The initial configuration requires the setup for a hub-and-spoke or partial-mesh configuration. Tunnel between branches/spokes are created automatically.  

Routing protocols can be used to deply ADVPN. Dynamic protocols are preferred, BGP is the most used and useful.  

___
### IPSec Phase 1

Phase 1 takes place when each peer of the tunnel connects and begins to set up the VPN. The initiator is the peer that starts the phase 1 connection, while the responder is the peer that responds to the initiator request.  

The first "connection" between the two peers is not secure, an attacker sitting in the middle of the conversation can intercept traffic. Before exchanging sensitive private keys, both peers must create a secure tunnel.  

The purpose of phase 1 is to authenticate peers and set up a secure channel for negotiating the phase 2 SAs that are later used to encrypt and decrypt traffic between the peers. To establish this secure channel, the peers negotiate a phase 1 SA. This SA is called the IKE SA and is bidirectional. 

To authenticate each other, peers use two methods: PSK or digital signature. XAuth can be enabled to enchance authetication.  

In IKEv1, there are two possible modes in which the IKE SA negotiation happens: main and aggressive. Both peers must agree on the terms of negotiation otherwise the phase 1 negotiation fails.  

At the end of phase 1, the negotiated IKE SA is used to negotiate the DH keys that are used in phase 2. DH uses the publick key (known by both ends) plus a mathematical factor called a none, in order to generate a common private key. With DH, attackers listenin to the messages containing public keys, they cannot determine the secret key.  

**IKE Mode Config** is an alternative to DHCP over IPSEC. The dial-up server passes network configuration (such as virtual addresses, networks and DNS) over to dial-up clients.  
IKE MOde Config is also compatible with IKEv2.  


**NAT-T**  
Since the ESP protocol does not use port numbers to differentiate one tunnel from another (no TCP or UDP), it cannot pass NAT. To solve the problem, **NAT-T** was introduced in IPSec. When peers detect NAT, **IKE negotiation switches to UDP port 4500 and ESP packets are encapsulated in UDP port 4500.**  
The **KeepAlive Frequency** is the time-interval at which FortiGate sends keepalive probes to determine if tunnels are dead. 

**DPD**  
Once a tunnel is up, peers don't usually exchange IP SAs until they expire. This means that if there is a network disruption along the path and the tunnel goes down, peers keep send data through the tunnel because they only know the tunnel is up.  
**DPD** probes are sent to detect a failed or dead tunnel and bring it down before IPSec SAs expire. **Useful when there are backup routes to switch over and reduce network issues due to dead links/tunnel**.  

FortiGate can send DPD probes when there is only outbound traffic (**On Demand**), when there is no traffic (**OnIdle**, resource intesive when there are many tunnels).  
DPD probes can be disabled.  

**Authentication**  
Supported authentication method are PSK-based and Signature-based.  
The former requires both peers to known the same PSK, while the latter requires certificates to be locally installed.  


IKEv1 supports _aggressive_ and _main_ authentication mode, while IKEv2 don't.  

|Aggressive|Main|
|-|-|
|PSK hash not encryted|PSK hash encrypted|
|3 packets exchanged, faster|6 packets exchaged, slower|
|Peer ID exchanged in the first packet*|Peer ID exchanged in the last packet|

Peer ID used for dial-up clients and since with aggressive mode that information is contained within the first packet, FortiGate uses that packet to match a client with the correct dial-up tunnel.  
When both ends know each othet IP addresses or FQDNs, tunnels are matched by IP addresses. 

**Phase 1 proposal** needs the set up of encryption algorithms that will be used to encrypt and decrypt data, and authentication algorithms that will be used to verify integrity and authenticity of the data. 
**DH groups**, at least 1. The higher the group number the more secure is the phase 1 negotiation but higher is also the compute time.  
**Key Lifetime** determine the expiration of IKE SAs.  

**Extended Authentication (XAuth)** requires client peers to authenticate themselves against the remote peers using a set of credentials (username and password).  
When FortiGate acts a dial-up server, administrators can choose the type of XAuth authentication server (auto, PAP, CHAP or disabled) and the user group matching (choose or inherit).  
The _Choose_ option requires different VPN profiles for every group and different network access policies (more control), while the _Inherit from policy_ option requires multiple policies for different user groups (simpler).  

___
### IPSec Phase 2

In phase 2, administrators must define the encryption domain of the IPSec tunnel. The encryption domain refers to the traffic to protect and it is determined by the phase 2 selector configuration.  

To have more granular control over traffic, multiple selectors can be configured.  
An encryption domain is defined by:
- **Local address** and **Remote address**
- **Protocol**
- **Local port** and **Remote port**

For every phase 2 selectors, appropriate phase 2 proposals must be configured.  

The **Enable Replay Detection** enables [anti-replay attacks](https://en.wikipedia.org/wiki/Anti-replay) protection.  

[**Perfect Forward Secrecy**](https://en.wikipedia.org/wiki/Forward_secrecy) protects data because every time a phase 2 expires, new secret keys are recalculated (based on DH). In this way new keys are not derived from older keys and past communications cannot be compromised.  
IPSec SAs lifetime is based on seconds or volume (bytes exchanged) or both. The lifetime thresholds do not have to match on both ends, peers agree to use the lowest one.  

The **Auto-Negotiate** key force FortiGate to generate new keys before the expiration lifetime is reached and to use them right away (old keys are always discarded). This prevents delays or traffic loss due to keys renegotiation.  

**Autokey Keep ALive** keeps the tunnel up.  

IPSec hardware offloading is enabled by default on supported models for the supported algorithms. Hardware offloading can be disabled per tunnel. 

___
### Route-based IPSec VPNs

Policy-based IPSec VPNs are a legacy IPSec VPN configuration supported only for backward compatibility and their use **is not recommended**.  One policy can allow traffic in both directions.

Route-based IPSec VPNs are the standard to which FortiGate refers when dealing with IPSec VPNs profiles. For each tunnel there is a virtual interface which can be referenced in policies, routing protocols, and other configurations.  

Each policy defines actions on traffic in one direction, inbound or outbound. A minimum of two policies is required to let the traffic passing through.  

Route-based IPSec VPNs support L2TP-over-IPSEC, GRE-over-IPSEC and dynamic routing.  

Static routes are automatically created by FortiGate if the `add-route` setting under phase1-interface configuration is set to `enable`. Static routes are added only after phase 2 is up, the destination is the local network presented by the dial-up client at phase 2 and the default route has AD of 15.  
Once the phase2 is down, routes are deleted from the routing table.  

When `add-route` is seto to disable, other dynamic protocols must be in place to take care of routing.  

When the remote gateway is either set to Static IP or Dynamic DNS, static routes must be manually configured.  

___
### Redundant VPNs

Two types of redundant VPNs:
- partially redundant: one peer has 2 WAN links while the other peer has 1 WAN link
- fully-redundat: each peer has 2 WAN links

Steps:
- add one phase 1 for each path + DPD on both ends
- add at least one phase 2 selector for each phase 1
- add at least one route for each VPN. Routes for the primary VPN have lower AD (or lower priority) than the backup
- allow traffic from policy for both VPNs