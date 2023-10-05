## Logging and Monitoring

Fortigate logs types are:
- **Forward traffic**: includes information about traffic that Fortigate either accepted or blocked
- **Local traffic**: includes information about traffic from and to the Fortigate management IP addresses
- **Sniffer**: includes information related to traffic seen by the one-arm sniffer

**Events** logs record system and administrative events while **Security** logs record security events such as virus attacks and intrusion attempts, base on the security profile type (log=utm)

Log levels range from 0 (highest importance) to 6 (lowest importance). Level 7 is used only in specific scenarios as requested by Fortinet Support.  

|Levels|Description|
|-|-|
|0 - Emergency|System unstable|
|1 - Alert|Immediate action required|
|2 - Critical|Functionality affected|
|3 - Error|Error exists that can affect functionality|
|4 - Warning|Functionality could be affected|
|5 - Notification|Information about normal events|
|6 - Information|General system information|
|7 - Debug|Diagnostic information for investigating issues|

Generating logs reduce performances, so lots os logs translate into more CPU resources used and more storage space consumed.

Logging to disk can be enabled either from the CLI o via the GUI: **Log & Report** > **Log Settings**.  
In order to keep the system running, Fortigates reserve about 25% of total disk space for system and unexpected quota overflow. 

Fortigates can send log to different remote devices such as:
- **Syslog**: a logging server that is used as a central repository for networked devices
- **FortiCloud**: is a Fortinet subscription-based, hosted security management and log retention service
- **FortiSIEM**: provides unified event correlation and risk management that can collect, parse, normalize, index and store security logs
- **FortiAnalyzer**: provides high store capacity for logs as well as specific analyzing capabilities
- **FortiManager**: central management of multiple Fortigate devices

Fortigates can be configured to send logs to FortiManager or FortiAnalyzer both from the GUI and the CLI: **Log & Report** > **Log Settings**.

Should the FortiAnalyzer becoming unavailable for a SHORT period of time (es. firmware upgrade), Fortigate devices can store locally logs and once the connection to FortiAnalyzer is re-established, all cached logs are transmitted. The process in charge of this behaviour is **miglogd**.  

Under the same GUI section, administrators can configure logs to be sent to FortiCloud (after Fortinet account succesfully activated).  
From the CLI, Fortigates can be configure to send logs up to four Syslog of FortiSIEM devices.  

Log messages are stored on disk and trasmitted to FortiAnalyzer as plain text in LZ4 compressed format.  
By default UDP port 514 is used for log transmission, however TCP can be enabled as main transport protocol from the CLI.  

If logging to FortiAnalyzer or FortiManager is configured from the GUI, reliable logging is automatically enabled. If configured from the CLI, it must be manually enabled.  

Logging to FortiCloud uses TCP. From the CLI it is possible to set the encryption algorithm (set to **high** by default).  

**OFTPS** is the protocol used by FortiGates to send encrypted logs to FortiAnalyzer.  

Benefit of using more than one remote log server is the chance of categorize logs per type or security and dedicate each category an entire log server.  

The CLI offers a more granular set of filter options in order to optimize which logs must be sent to remote logging devices. 
Filters include:
- Severity \<level>
- Forward traffic [enable|disable]*
- Local Traffic *
- Multicast Traffic *
- Sniffer Traffic *
- Anomaly *
- VOIP *
- ZTNA-traffic *
- GTP *
- Filter [string]
- Filter Type [include | exclude]

Hardware acceleration affects logging:
- Traffic offloaded to NP6 and NP6lite processors does not traffic statistics
- Traffic offloaded to NP7 processors have improved logging of traffic statistics capabilities
    - Can disable hardware acceleration
    - Can enable NP packet logging

Logs can be backup up to remote destination or to USB devices. From the GUI is also possible to download them but only those visible in the current view( as such, filters might be useful to download specific subsets ot log messages). From the CLI the rest of more useful commands.