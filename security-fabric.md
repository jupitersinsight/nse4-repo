## Security Fabric

Fortinet solution to centrally manage the network (from physical component to virtual devices in the cloud) and automate defence.  

The Security Fabric has the following attributes:

- **Broad**: it provides visibility of the entire digital attack surface
- **Integrated**: reduce complexity in supporting multiple point products
- **Automated**: threat intelligence is exhanged between network components in real-time allowing for automated response to threats

The API and protocol are available for other vendors to join and for partner integration, so that Security Fabric is extended to third-party devices.  

Core implementation of Security Fabric (minimum accordignly to Fortinet):
- minimum of two Fortigate devices: one root, and one or more downstream
- at least one of: FortiAnalyzer, FortiAnalyzer Cloud or FortiGate Cloud

Recommended implementation (previous +):
- FortiManager, FortiAP, FortiSwitch, FortiClient, FortiClient EMS, FortiSandbox, FortiMail, FortiWeb, FortiNDR, FortiDeceptor

Extended implementation (previous +):
- Other Fortinet products and API for third-party devices

To configure Security Fabric, on the root firewall enable **Security Fabric Connection** on all interfaces participating and then enable the **Security Fabric Connector** and select **Serve as Fabric Root**. FortiAnalyzer is required, this setting will be pushed to downstream FortiGates.  

On downstream devices, enable **Security Fabric Connection** and **Device detection**. On the **Security Fabric Connector** select **Join Existing Fabric**. 

The last step requires administrators to authorize downstream devices (it is possible to pre-authorize them adding thei serial numbers to the root firewall).

Synchronization flows from root Fortigate to downstream devices. To stop synchronization, on root Fortigate use the command `set fabric-object-unification local`.  

Downstream devices can be forced to not sync with root but still push sync objects downstream, `set configuration-sync local`.

It is possible to specify sync settings per object (only the supported ones).

Only VDOMs with ports assigned are displayed in the Security Fabric. VDOMs cannot join different Fabrics.  
Devices identification works agentless (identifying devices based on TCP fingerprint, HTTP user agent, MAC address...) or with agents (FortiClient).  