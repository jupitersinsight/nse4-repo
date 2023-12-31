## FSSO

SSO, Single Sign-On, is a process which allow users to be automatically logged in to every application after being identified, regardless of the platform, technology and domain.  

FSSO, Fortinet Single Sign-On, is a software agent that enables FortiGate to identify users for security policies, VPN access, in advanced deployments with FortiAuthenticator.  

When a user logs in to a directory service, its username, password, IP and group membership are collected and maintained by FortiGate in a local database. These information are then used to automatically identify users and apply the right policies and instructions.  

FSSO is typically used in conjuction with directory services such as Active Directory.  

FSSO for AD uses collector agents. DC agents may also be required, depending on the collector agent working mode.  
There are two working modes that monitor user sign-on activities in Windows. DC agent mode and polling mode. FortiGate also offers a polling mode that does not require a collector agent, which is intended for simple networks with a minimal number of users.  

There is another kind of DC agent used for Citrix and Terminal Services environments: terminal server (TS) agents.  
TS agents require the Windows AD collector agent or FortiAuthenticator to collect and send the login events to FortiGate.  

<ins>DC Agent Mode.</ins>  
DC agent mode is the recommend mode for FSSO and requires:

- One DC agent (`dcagent.dll`) installed on each Windows DC under `Windows\system32`
- A collector agent, which is another FSSO component  

The collector agent is installed on a Windows Server that is member of the DC, its purpose is to consolidate information received before forwarding them to FortiGate.  
The collector agent is responsible for group verification, workstation checks and FortiGate updates of login records. The FSSO collector agent can send domain local security group, OUs and global security group information to Fortigate devices. It can also be customized for DNS lookups.  

When the user logs in, the DC agent intercept the login event on the domain controller. It then resolves DNS of the client and sends to the collector agent on port UDP 8002.  

The collector receives the information and then performs a DNS lookup in order to check if the IP of the client has changed and sends information to FortiGate on port TCP 8000.  

In some configurations, double DNS resolution is a problem. In this case, you may configure a registry key on the domain controller that hosts the DC agent in order not to resolve the DNS.  
```
donot_resolve = (DWORD) 1 at HKLM/Software/Fortinet/FSAE/dcagent
```

<ins>Polling Mode</ins>
Polling mode can be agent-based or agentless.  

**Agent-based polling mode**, like in DC Agent Mode, requires a collector agent to be installed on a Windows Server but it doesn't require DC agents installed on each DC.  
In this mode the collector agent generates unnecessary traffic when there have been no login events.  

In Windows Event Log Polling, the most commonly  deployed polling mode, the collector uses TCP port 445 (SMB) to periodically request event logs from DCs. TCP ports 135, 139 and UDP port 137 are used fallbacks.  

Polling mode supports three methods for information collection:
- **WMI** (most recommended): is a Windows API that gets system information from a Windows Server. The DC returns all requested login events. The collector agent is a WMI client which sends WMI queries for login events to the DC, which is a WMI server. Since the DC returns login events, the collector does not have to search for them thus reducing the load on the network. 3 seconds interval.
- **WinSecLog**: polls all security event logs from the DC. In large networks may reuquire some time to send data to FortiGate. Only parses known event IDs by collector agent. 10 seconds interval. 
- **NetAPI** (less recommended): polls temporary sessions on DC of logged users. Since sessions are created and purged from RAM, this method may miss login events, especially if the DC is under heavy load. 9 seconds interval.  

**Agentless polling mode** consists in FortiGate devices to directly poll information from DCs. The connection happens through TCP port 445.  
This mode is similar to WinSecLog method of Agent-based polling mode but Fortigate only parses logs for IDs 4768 and 4769.  

Resource-intensive, event logging must be enalbed on DCs, workstations are not polled. 


For full functionality, for both FSSO mode, the collector agent must be able to poll workstations.  

Agentless polling mode uses LDAP, with DC administrators credentials to poll information from each DC specified.  

It is best practite to not collect information/events generated by network service accounts.  

**Troubleshoot**

- Ports needed: 139 (workstation verificatio), 445 (workstation verification and event log polling), 389 (LDAP), 445/636 (LDAPS), 3268-3269 (TLS)
- Guaranteed minimum bandwith of 64kbps between Fortigate and DCs
- In all-Windows environment, incative sessions must be flushed regularly
- DNS and IPs must be correct
- Never set workstation timer to 0, this prevents the collector agent from aging out stale entries
- Include all FSSO groups in the firewall policies when using passive authentication, if active authentication is used as a backup, do not include SSO_Guest_Users to policies