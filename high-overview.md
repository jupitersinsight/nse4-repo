# Administering the Fortigate

## Administrator Profiles

Default administrator profiles are:
- super_admin: global administrator
- prof_admin: VDOM administrator

It is always possible to create custom profiles with privilege limitations.

Administrators can be global and so having access to the entire device, or per VDOM (or multiple VDOMs).  
Per VDOM administrators have power only in their VDOM.

### Reset lost Administrator password

After a hard power cycle, for 60 seconds, it is possible to log-in as a special user only via the console port.  

User: `maintainer`  
PSW: `bcpb\<serial-number>`

This procedure is available for all Fortigate devices.  
It can be disabled for security concerns:

```
config sys global
    set admin-maintainer disable
end
```

For VM instances this option is not valid. Administrators must rely on snapshots.

## Admin access

### Trusted sources

It is possible to configure trusted sources for admin access under **System** > **Administrator** > **Restrict login to trusted hosts**.

### Trusted ports and password

Best practice is to change ports for web access to the GUI, SSH and (disable) Telnet, enforce encrypted protocols and enforce strong password requirements: **System** > **Settings**

Admin idle-timeouts can be configured per profile.

___
## VLANs

Native VLAN: ID 0.

___
## Configuration backup

File configuration backup can be encrypted ot not.

Even if you choose not to encrypt the file, which is the default, the passwords stored in the file are hashed, and, therefore, obfuscated. The passwords that are stored in the configuration file would include passwords for the administrative users and local users, and preshared keys for your IPSec VPNs. It may also include passwords for the FSSO and LDAP servers.  

An encrypted backup file protects its content in a very strong way because decryption will require the password used for encryption and a device of the same model and firmware.  

Backup files can be created per VDOM and globally.

___
## Policies

It is possible to create **zones** which are groups of interfaces sharing the same purpose, thus the same policies. Interfaces which are part of a zone cannot be referenced as single interfaces.  

By default, from the GUI, policies only allow one interface as destination. In order to use multi-interface policies, the feature must be enabled. There is no such limitations for policies configured from the CLI.  

Sources can be:
- IP Address or range
- Subnet (IP/Mask)
- FQDN (DNS must be properly configured)
- Geography
- Dynamic (Fabric connector address)
- MAC address
- Users (optional)
    - Local firewall accounts
    - Accounts of a remote server (Active Directory, LDAP, RADIUS)
    - FSSO (Fortinet Single Sign-On)
    - Personal certificate (PKI-authenticated) users

When creating policies from the GUI, by default, FortiOS does require a unique name. From the CLI, the name is not required.  
Unnamed policies are allowed from the CLI, while from the GUI they must be enabled.  

Policies are automatically assigned UUIDs which is what FortiOS and other tools from Fortinet use to keep track of policies and to share information among them.  

**Fortigates are stateful firewalls which means they remember the source-destination pair of a communication and allow for replies to pass. There is no need to configure a return-policy-.**  

To reduce CPU loads when a deny policy is being hit, it is possible from the CLI to enable the creation of a session table entry of dropped traffic. This means that when a session is dropped, all its packets are also denied, saving the Fortigate from the process of a lookup policy for each new packet matching the denied session.  
This option is identified by the command `ses-denied-traffic`.
It is also possible to define a block duration, by default it is 30 seconds.  

Policies are identified by _PolicyIDs_ which cannot be modified once assigned.

**ALL_ICMP service is not subject to web filter and antivirus scan**.

When using VIPs, by default the fortigate object **all** (0.0.0.0/0) does not include VIPs destinations. So, when denying traffic, in order to include VIPs, on the CLI must be set the option `match-vip enable`.

### Traffic Shapers

A shared traffic shaper applies a total bandwidth to all traffic using that shaper. The scope can be per policy or for all policies referencing that shaper.

Per IP traffic shapers applies limitations only to specified IP addresses.  

Be careful about UDP communications, since UDP is stateless, packet losses can do harm.

___
## Inspection mode and NGFW features

**Flow-based inspection mode** means that each packet is a processed and forwarded without waiting for the complete file or web page. This means that advanced feature involving content modification cannot be performed but response time is faster. 
It is the default inspection mode and uses single-pass direct filter (DFA) pattern matching to identify possible threats.   

**The 3-way handshake is performed between client and final destination with Fortigate sitting in the middle, WITHOUT TAKING PART IN THE HANDSHAKE**

**Proxy-based inspection mode** means that Fortigate analyzes the traffic _as a whole_ before actions are taken. This inspection mode is also called **transparent proxy** because Fortigate is not the final destination and **does not alter IP addresses** involved but intercepts traffic.  

**The 3-way handshake is perform between the client and Fortigate and between Fortigate and the final destination, TAKING PART IN THE HANDSHAKE**

Fortigate provides two NGFW feature per single VDOM:

- **Profile-based mode**: requires administrators to create profiles for web filter and application control which are then applied to policies, either flow-based and proxy-based inspection modes are supported
- **Policy-based mode**: administrators can apply web filter and application control configuration directly to a security policy, flow-based inspection mode only is supported

Antivirus configuration is always profile-based.  
To switch between Profile-based and Policy-based mode, **System** > **Settings** but **ALL** existing policies are removed.  

When policy-based mode is in use, Fortigate or the VDOM must have a few polcies to allow traffic:

- **SSL Inspection & Authentication (consolidated) policy**: this policy allows traffic from a specific user or user group matching criteria withing the consolidated policy, SSL traffic inspection using the SSL inspection specified profile selected. Fortigate can ether accept or deny the traffic (**Policy & Objects** > **SSL Inspection & Authentication**)

- **Security policy**: is traffic is allowed according to the consolidated policy, than Fortigate performs other checks as specified in the security policies such as web filtering, application control, URL categorization, AV, file filter, IPS... (**Policy & Objects** > **Security Policy**)

___  
## Diagnostics

A baseline is very important to determine when an abnormal event is happening. Monitor:
- CPU usage
- Memory usage
- Traffic volume
- Traffic directions
- Protocols and port numbers
- Traffic pattern and distribution

Network diagrams are helpful:
- Physical diagram: L1 and L2 relationships
- Logical diagram: L3 relationsips

Useful tools:
- Security Fabric
- SNMP
- Dashboard (GUI)
- Alert email
- Logging/Syslog/Fortianalyzer
- CLI debug commands

```
get system status
```

Command used to display the Fortigate interface hardware and status information.
```
get hardware nic <interface>
```
Command to display the ARP table
```
get system arp
```

To debug packet flow: (available also in the GUI)
- define a filter: `diagnose debug flow filter <filter>`
- enable debug output: `diagnose debug enable`
- start the trace: `diagnose debug flow trace start `
- stop the trace: `diagnose debug flow stop`

Useful while troubleshooting firewall actions on packets or packet routing.

When memory usage is too high, FortiGate may enter conserve mode impacting user traffic.  
If memory usage keeps increasing, the firewall refuses to accept any new connection.  
Exit conserve mode when memory usage drops below 82%.  

Under `ips global` settings, administrators can configure the ips to put the firewall in a fail-open state when memory usage is too high. In this way packets are allowed through the firewall without being inspected by the IPS (not recommeded because the IPS engine powers many firewall feature, such as flow-based traffic inspection, network application identification...) 

Hardware (CPU and memory) usage:
### CPU
```
get system performance status
diagnose sys top -1
```

While in conserve mode the `av-failopen` setting defines the action that is applied to any proxy-based inspected traffic, it applies to flow-based traffic as well.  
```
config system global
    set av-failopen <off | pass | one-shot>
end
```
- **off**: all new sessions with content-scanning enabled are not passed but Fortigate processes the current active sessions
- **pass** (default): all new sessions pass without inspection untile Fortigate exits conserve-mode
- **one-shot**: similar to pass but it will keep bypassing inspection even after Fortigate exits conserve-mode. Administrators are required to restart the unit or the antivirus service.  

If the memory usage exceeds the memory threshold, new sessions are dropped regardless the unit configuration in use.  

To determine if a unit is in conserve-mode `diagnose hardware sysinfo conserve`

When a proxy on Fortigate runs out of available sockets to process more proxy-based sessions, the unit goes into a fail-open session mode. New sessions are dropped until new sockets are available, the `av-failopen-session` overrides the default behaviour. 

```
config system global
    set av-failopen-session <enable | disable>
```
- **enable**: sessions are allowed
- **disable**: sessions that require proxy-based inspection are blocked

To run hardware checks `diagnose hardware test suite all`.

To read unit's crash logs `diagnose debug crashlog history`.  
Entries in the crash logs are most of the time related to legit firewall processes, but still useful to know they are there in case of need.  

