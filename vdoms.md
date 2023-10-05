## VDOMs

VDOMs split the Fortigate into multiple virtual devices each with its own policy table, routing table and so on.  

By default, inter-VDOMs traffic is not permitted. VDOMs can share a same IP address with no conflict.  
Fortigates support up to 10 VDOMs out-of-the-box.

DNS database can be per VDOM.

Multi-VDOM works well for managed service providers leveraging multi-tenant configurations, or large enterprise environments that desire departmental segmentation. Administrators can give each individual tenant or department visibility and control of their VDOM, keeping the others hidden and independent.  

Two types of VDOMs can be created in Multi-VDOM:
- **admin VDOM**: for FortiGate administration
- **traffic VDOM**: permits traffic to travel through the FortiGate

Traffic originating from FortiGate, such as traffic for Fortiguard updates or NTP, comes from the **management** VDOM. There can be only one management VDOM. By default the root VDOM is assigned the managemenet VDOM role, but it can be reassigned.  

The only requirement for the management VDOM to work properly is direct connection to Internet.  

**Independent VDOMs**  
Each VDOM, in a scenario where multiple customers share the same FortiGate appliance, need a separate physical connection to the Internet.  
Inter-VDOM routing is not possible unless traffic leaves the FortiGate and is rerouted back into another VDOM from the external server (Internet).  

**Meshed VDOMs**  
VDOMs connect to a dedicated VDOM which is the only one with Internet access. All traffic routed to external domains/IPs passes through the dedicated VDOM, where traffic from all VDOMs meets.  
Inter-VDOM routing is possible in the dedicated VDOM, or linking together VDOMs.  

More complex configuration and troubleshoot but better inter-VDOM routing performance due to shorter paths.  

**Routing through a single VDOM**  
VDOMs are not connected among them but each is connected to a dedicated VDOM for internet access. Again, inter-VDOM routing is possible but only through the dedicated VDOM.  
This scenario can be suitable for customers sharing the same FortiGate, each in their own VDOM.

Only the user named **admin** or administrators with role **super_admin** are global to all VDOMs.  

VDOMs administrators permissions are define in their profile, there can be multiple admininistrators per VDOM each with speficic duties, there can be multiple VDOMs per administrator. VDOM administrators are also responsible for VDOM configuration backup.  

**Global** > **System** > **Administrators**, GUI path for administrators creation, profile and VDOM assignment.  

Multi-VDOM can be activate both from the GUI (**System** > **Settings**) and the CLI.

The default inspection mode is flow-based. It is possible to switch between Profile-based and Policy-based mode.  
The former requires creation of security services profiles applied to policies, the latter requires the creation of policies in which administrators will configure security services settings.  
Proxy-based inspection mode is not possible when policy-based is active.  

Switching between inspection modes determine the loss of all configured policies. 

VDOM can operate as in NAT mode or Transparent mode.

While operating in transparent mode, FortiGate acts as a L2 switch passing traffic without changing the MAC header in the frame.

Transparent mode is useful when installing FortiGate in existing networks (large networks) where addressing scheme reconfiguration is not an option.  
Acting as a L2 switch, FortiGate populates its MAC table, determining where endpoints sit in the network realtive to itself. Each link determines a collision domain.  

By default, while in transparent mode, each VDOM forms a separate forward domain but interfaces do not: they still belong the to root VDOM even if are part of different VLANs. This require administrators to manually set the forward domain ID for each interface. If interfaces are not split into VDOMs, their broadcast is the same which means troubles... broadcast storm. 

In order to prevent accidental VDOM creation, administrators can enable a security feature which prompts administrators for confirmation upon VDOM creation.

Interfaces (physical and virtual) cannot be member of more than one VDOM at a time.  
**Global** > **Network** > **Interfaces**

|Global settings|Per-VDOM settings|
|-|-|
|Hostname|Operating mode (NAT/Transparent)|
|HA settings|NGFW mode (profile-based/policy-based)|
|Fortiguard settings|Routers and network interfaces|
|System time|Firewall policies|
|Administrative accounts|Security profiles|

While on the GUI, global-administrators can switch between VDOMs and the Global view using a dropdown menù.  
On the CLI, global settings are set in the `(global)` context while VDOM settings are set in the `(vdom-name)` context.  
Administrators can prepend their command with the `sudo` command to set settings across contexts, without the need of switching between contexts.

Security profiles can be global if the same profile is needed for more than one VDOM. This remove the need of creating the same profile in each VDOM. 

The following profiles can be global:
- Antivirus
- Application control
- Intrusion prevention
- Web filtering

Global profiles must be prefixed with substring "g-" and are read-only in VDOMs.  

Inter-VDOM links are virtual links whose purpose is to link a VDOM to another VDOM.  
|VDOM1|VDOM2|Link|
|-|-|-|
|NAT|NAT|Ok|
|NAT|Transparent|Ok|
|Transparent|NAT|Ok|
|Transparent|Transparent|No|

NAT-to-NAT links require both VDOMs to belong to the same IP subnet.

In order for inter-VDOM links to work properly, routers and firewall policies are required. The logic behind the setting is the same as if connecting FortiGate to an external device.  

**Global** > **Network** > **Interfaces** > **Create New** > **VDOM Link**

FortiGate devices with NP4 and NP6 processors include inter-VDOM links that FortiGate can use to accelerate inter-VDOM link traffic. When both processors are present, there are two inter-VDOM links each with two interfaces:

- **npu0_link**
    - npu0_vlink0
    - npu0_vlink1
- **npu1_link**
    - npu1_vlink0
    - npu1_vlink1

These interfaces are available both in the GUI and the CLI.  

**Best practices and Troubleshooting**

VDOMs all share the same resoucers by default with no limits on usage.  
Usage limits can be set per feature at the global level or per-VDOM.  
**Global** > **System** > **Global resources** to set global limits on critical objects such as sessions, policies...  
**Global** > **System** > **VDOM** to set per-VDOM quota on global resources, can set minimum guaranteed. (Displays also VDOM CPU and memory usage).  

If administrators have problems while logging-in:
- check administrator is allowed to access the VDOM
- check interface from which the administrator is reaching the VDOM is actually part of the VDOM
- check the connection protocol is allowed, such as SSH
- check trusted hosts and IPs, if set, and that there is no device in the middle performing NAT

In general, apply least-privilege.