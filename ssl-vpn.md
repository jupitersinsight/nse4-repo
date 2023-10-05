## SSL VPN

Tunnel mode supports the most protocols but require installation of a dedicated network adapter or VPN client.

Web mode does not support as much protocols as Tunnel mode does (supported protocols: FTP, HTTP/HTTPS, RDP, SMB/CIFS, SSH, Telnet, VNC, Ping) but the onyl requirement, client-side, is a web browser.  

**SSL Web Mode** Bookmarks contains links to all or some of the recources available for user to access. The Quick Connection widget allows users to specify hostname or IP Address of the server to reach. Applications local to the user endpoint cannot send data through the VPN.  

Different users may access different SSL Portals, each with a set of bookmarks and access permissions.  

FortiGate acts as a reverse proxy or HTTPS gateway, authenticating users' credentials and connecting to right SSL Portal. Incoming remote user IP address is translated into internal FortiGate IP address.  

**SSL Tunnel Mode** requires FortiClient in order to work which installs on endpoints a net adapter named _fortissl_.  
All traffic is encapsulated/encrypted using SSL/TLS, if split tunnel is not enabled, all traffic is sent through the VPN Tunnel.  

**FortiGate can act as a SSL VPN client**. Routes to subnets returnd by the SSL VPN Server are dynamically added to the routing table (default route from Fortigate SSL client to Fortigate SSL server added as ECMP route along with already existing default route).  

Hub-and-spoke VPN topology is possible via SSL VPN protocol and it is suitable when ESP packets may be blocked, UDP ports 500 and 4500 may be blocked, peers don't support IKE fragmentation causing IKE negotiation when large certificates are used.  

FortiGate SSL VPN server requires a proper CA certificate to be used while FortiGate SSL VPN client/user uses PSK and PKI client certificate to authenticate.  All traffic is encrypted and sent through the tunnel.  

To configure FortiGate as a SSL VPN Serverthe first step is to create to accounts local/remote and PKI.  
The PKI menu is only available in the GUI after a PKI user has been created using the CLI, and a CN can only be configured in the CLI. If no CN is specified, then any certificate that is signed by the CA will be valid and matched.
To configure Fortigate as a SSL VPN Client, create user _pki_, using the same CN if PKI user on the Server has CN configured, select the CA certificate that allows the FortiGate to complete the certificate chains and verify the server's certificate. 

---
SSL VPN supports different authentication methods, local to Fortigate or against remote authentication servers (RADIUS, TACACS++, LDAP). FSSO is not supported.  

SSL interfaces on FortiGate are created per VDOM using the naming convention _ssl.<VDOM_NAME>_.  
Firewall policies are mandatory to detemine to which local interface(s) remote users can access to and to which resource(s).

Client integrity checking is only available in Tunnel mode and for Windows endpoints. Windows Security Center is queried to extract information about antivirus/firewalls or other security applications running in order to determine if a client may pose a threat or not.  
The check is performed while the VPN connection is still establishing, just after authentication has finished. If the required software is missing from the client, the connection is rejected even with valid user credentials.

Client integrity check can configured both via GUI or CLI but a list of recognized software is available on the CLI only.  
The list is divided into three categories: antivirus, firewall and custom. Windows registry plays an important role since applications are recognized by their GUID (windows registry string unique for each application).  

---
**System Events** >> **VPN Events**  
**System Events** >> **User Events**

Change SSL VPN timers to prevent users from timing-out (logout) when connecting over high-latency links.

**Best practices** 

Web Mode:
- Cookies enabled on clients and Internet privacy profiles set to HIGH
- SSL server URL (scheme:url:port) set right

Tunnel Mode:
- Forticlient right version, compatible with FortiOS in use
- Split tunnel or policy for external resources access

General:
- Users connet to the right port number
- Firewall policies inlude groups, users and destination address
- Timeout to flush inactive sessions
- Users encouraged to log out if access to corporate resource is no longer needed