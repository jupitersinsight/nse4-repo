**Disable account _maintainer_**
```
config sys global
    set admin-maintainer disable
end
```
___
**Create new admin profile**
```
config system accprofile
    edit NEW-PROFILE
    [...]
end
```
**Create profile to log read access only**
```
config system accprofile
	edit log_reader
		set loggrp read
		set system-diagnostics disable
	next
end
```

___
**Create new admin**
```
config system admin
    edit NEW-ADMIN
    [...]
end
```
**Configure log_reader admin**
```
config system admin
    edit "log1"
        set accprofile "log_reader"
        set password ****
    next
end
```
___
**Get current firmware**
```
get system status
```
___
**Enable denied session to be added into the session table**
```
config system settings
    set ses-denied-traffic enable
end
```
**For optimum performance, adjust the global block-session-timer.**
```
config system global
    set block-session-timer <1-300>  (default = <30>)
end
```
___
**Edit policy**
```
config firewall policy
    edit <policy_id>
end
```

**Set policy to block all traffic, VIPs as well**
```
config firewall policy
    edit <policy_id>
        set match-vip enable
        set dstaddr <VIP>
end
```

___
**Diagnose NAT - Fixed Port Range**
```
diagnose firewall ippool list
diagnose firewall ippool-all list
diagnose firewall ippool-all stats <Optional IP Pool Name>
diagnose firewall ippool-fixed-range list natip <ip-address>
diagnose firewall ippool-fixed-range list natip <ip-address> <port-number>
```

___
**Diagnose LDAP and RADIUS query**
```
diagnose test authserver ldap <server-name> <username> <password>
diagnose test authserver ldap <server-name> <scheme> <username> <password>
```

___
**Enforce active authentication**
```
config user setting
    set auth-on-demand always|implicit
end
```

___
**Set Authentication timeout**
```
config user setting
    set auth-auth-timeout-type idle|hard|new-session
end
```

___
**Enable performance statistic logging for remote logging devices on Fortigate**
```
config system global
    set sys-perf-log-interval <number from 0 to 15>
end
```

___
**Enable logging to disk and change maximum log-on-disk retention time**
```
config log disk setting
    set status enable
    set maximum-log-age <integer>
```
**Determine amount of space reserved for system**
```
diagnose sys logdisk usage
```

___
**Configure Fortigate to send logs to FortiManager/FortiAnalyzer**
**Determine upload option**
```
config log [fortianalyzer|fortianalyzer-cloud|fortianalyzer2|fortianalyzer3] setting
    set status enable
    set server <server_IP>
    set upload-option [store-and-upload|realtime/1-minute/5-minute]
end
```

___
**Display statistic for the miglogd process**
```
diagnose test application miglogd 6
```
**Display statistics for log cache memory being full (increased `failed-log` value)**
```
diagnose log kernel-stats
```

___
**Configure Syslog/FortiSIEM (up to four)**
```
config log [syslogd | syslogd2 | syslogd3 | syslogd4] setting
    set status enable
    set server <syslog_IP>
end
```
**Configure encryption for logs to FortiGuard**
```
config log fortiguard setting
    set status enable
    set source-ip <src IP used to connect to FortiCloud>
    set upload-option [store-and-upload|realtime/1-minute/5-minute]
    set enc-algorithm [high-medium | high |low]
end
```
**Configure format of logs for Syslog servers**
```
config log syslogd3 setting
    set format [default | csv | cef]
end
```
**Enable TCP for log transmission**
```
config log fortianalyzer setting
    set reliable enable
end
```
___
**Set log filter**
```
config log syslogd|fortianalyzer filter
        set severity <level>
end
```

___
**Anonymize usernames in logs**
```
configure log setting
    set user-anonymize enable
end
```

___
**Display log messaged from the CLI**
```
execute log filter # what you will see
execute log display # display messages with above filter applied
```

___
**Configure alert email (up to 3 recipients)**
```
config alertemail setting
    [many triggers]
end
```

___
**Backup log messages**
```
execute backup disk alllogs [ftp | tftp | usb]
```
OR 
```
execute backup disk log <logtype> [ftp | tftp | usb]
```
**Configure logs rolling (zipping and eventually archive)**
```
config log disk setting
    [many options]
end
```

___
**Enable search engines safe-search**
```
config webfilter profile
    edit "default"
        config web
            set safe-search url header
        end
    next
end
```
**Enable and set video filter and YouTube API key**
```
config videofilter youtube-key
    edit 1
        set key "youtube_api_key"
    next
end
```
**Debug Fortiguard connection issues**
```
diagnose debug rating
```

___
**Force FortiGate to check for new application control updates**
```
execute update-now
```

___
**Enable GUI interface for FortiSandbox Cloud**
```
set-gui-fortigate-cloud-sandbox enable
```
**Enable FortiSandbox inline scanning**
```
config system fortisandbox
    set inline-scan {enable | disable}
end
```
**Verifiy signatures versions on GUI or CLI commands**
```
diagnose autoupdate status
diagnose autoupdate versions
```
**Choose antivirus database**
```
config antivirus settings
    set use-extreme-db {enable | disable}
end
```
**Configure Client Comforting Features**  
**Configure Protocol Options such as port mappings per protocol**  
_Used by many security profiles... antivirus, web filter, dns filter, data loss prevention..._  
**Configure buffer size limit and logging for oversized files**  
**Configure buffer size limit for archive decompression**
```
config firewall profile-protocol-options
    edit <profile-name>
        config <protocol-name>
        set options oversize
        set oversize-limit <integer>
        set oversize-log {enable | disable}
        set uncompressed-oversize-limit [1-<model_limit>]
        set uncompressed-nest-limit [1-<model-limit>]
    end
end
```
**Get hardware information, containing details about processing units installed**
```
get hardware status
```

**Enable offload to hardware specific processors**
```
config ips global
    set np-accel-mode {none | basic}
    set cp-accel-mode {none | basic | advanced}
end
```
**Check new AV updates**
+
**Debug AV issues**
```
diagnose debug application update -1
diagnose debug enable
execute update-av
```
**Advaced debugging**
```
get system performance status
diagnose antivirus database-info
diagnose autoupdate versions
diagnose antivirus test "get scantime"
execute update-av
```

___
**Configure CVE patterns**
```
config ips sensor
    edit "cve"
        set comment "cve"
            config entries
                edit 1
                    set cve "cve xxxx-xxxx"
                    set status enable
                    set log-packet enable
                    set action block
                next
            end
        next
    end
```

**Troubleshoot IPS high CPU usage**
```
diagnose test application ipsmonitor <integer>
```

**IPS fail-open setting**
```
config ips global
    set fail-open {enable | disable}
    ...
end
```

___
**Security Fabric sync options**
```
config system csf
    set status enable
    set group-name "<name>"
    set configuration-sync {default | local}
    set fabric-object-unification {default | local}
end
```

___
**Display FIB entries**
```
get router info kernel
```
**Get routing information**
```
get router info routing-table all
```
**ECMP configuration**  
SD-WAN disabled
```
config system settings
    set v4-ecmp-mode {source-ip-based | weight-based | usage-based | source-dest-ip-based}
end
```
SD-WAN enabled
```
config system sdwan
    set load-balance-mode

```
Configure weight for the interface
```
config system interface
    edit <interface name>
        set weight <0-255>
    next
end
```
Configure weight for the route
```
configure router static
    edit <id>
        set weight <0-255>
    next
end
```
Configure spillover thresholds (kbps)
```
config system interface
    edit <interface name>
        set spillover-threshold <0-16776000>
        set ingress-spillover-threshold <0-16776000>
    next
end
```
**Change RPF settings**
```
config system settings
    set strict-src-check {disable | enable}
end
```
```
config system interface
    edit <interface>
        set src-check disable
    next
end
```
**Set Link Health Monitor**
```
config system link-monitor
    edit <profile>
        set srcintf <port>
        set <server1> <server2>... # up to four
        set gateway <gateway>
        set protocol <protocol>
        set update-cascade-interface {enable | disable}
        set update-static-route {enable | disable}
        set update-policy-route {enable | disable}
    next
end
```
```
config system interface
    edit <profile>
        set fail-detect {enable | disable}
        set fail-detect-option detectserver
        set fail-alert-method link-down
        set fail-alert interfaces <"port">
    next
end
```
**Display active routes**
```
get router info routing-table all
```
[AD/Metric] - [Priority/Weight]  
- Metric only for dynamic routes
- Weight is always 0 for dynamic routes
**Display all routes**
```
get router info routing-table database
```
**Display policy routes**
```
diagnose firewall proute list
```
- regular policy routes: ID <= 65535
- ISDB policy routes: ID > 65535
- SD-WAN policy routes: ID > 65535 and vwl_service field
**Packet sniffer**
```
diagnose sniffer packet <interface> '<filter>' <verbosity> <count> <timestamp> <frame.size>
```
- **interfcae**: `any` or specific
- **filter**: follows `tcpdump` format
- **verbosity**: specifies how much information to capture (3 or 6 to convert into pcap for analysis in Wireshark)
- **count**: number of packets to capture
- **timestamp**: print timestamp information
    - **-a**: absolute timestamp
    - **-l**: local timestamp
- **frame-size**: specify length (MTU)

___
**Configure VDOMs**
```
config system global
    set vdom-mode {no-vdom | multi-vdom}
end
```
**Change per-VDOM settings**
```
config vdom
edit <vdom>
    config system settings
        set vdom-type {admin | traffic}
        set opmode {nat | transparent}
    end
```
**Set forward-domain ID per interface when multi-VDOM is used in FortiGate transparent mode**
```
config system interface
    edit <interface_name>
        set forward-domain <domain_ID>
end
```
**Enable VDOM creation confirmation prompt**
```
config system global
    set edit-vdom-prompt {enable | disable}
end
```
**Configure interface VDOM membership**
```
config system interface
    edit <interface_name>
        set vdom <vdom_name>
end
```
**Set global, per-VDOM, and cross-contexts settings**  
Set global settings
```
config global
```
Set VDOM settings
```
config vdom
    edit <vdom-name>
```
Set settings across contexts
```
sudo [global | vdom-name] [diagnose | execute | show | get]
```
**VDOM traffic troubleshoot**
```
diagnose sniffer packet <interface_name> <'filter'> <verbose> <count>
```
More details, as NAT and policies applied
```
diagnose debug enable
diagnose debug flow filter addr <source>
diagnose debug flow trace start <number of packets>
```
___

**Debug FSSO**  
List currently logged-in users
```
diagnose debug authd fsso list
```
Refresh user group information
```
execute fsso refresh
```
```
diagnose debug enable
diagnose debug authd fsso server-status
```
Detailes about agentless polling results
```
diagnose debug fsso-polling detail
```
Flushes information about all active FSSO users
```
diagnose debug fsso-polling refresh-user
```
Real-time polling-mode debug
```
diagnose debug application fssod -1
```

___
**Debug ZTNA**
```
diagnose endpoint record list <optional IP address>
```
**Configure ZTNA options**
```
config firewall access-proxy
    edit <name>
        set client-cert enable
        set empty-cert-action {block | accept}
    end
```

___
**Config SSL VPN**
```
config vpn ssl web portal
    edit <portal-name>
        set tunnel-mode {enable | disable}
        set web-mode {enable | disable}
        set host-check {none | av | fw | av-fw | custom}
        set host-check-interval <seconds>
    end
```
**VPN settings***
```
conf vpn ssl settings
# Determine IP allocation method

    set tunnel-addr-assigned-method first-available/round-robin 

# Timers

    set login-timeout <10-180>
    set dtls-hello-timeout <10-60>

# Can help against DoS Attack partial HTTP requests, such as SlowIoris and R-U-Dead-Yet

    set http-request-header-timeout <1-60>
    set http-request-body-timeout <1-60>
end
```
**Create PKI user**
```
config user peer
    edit pki
        set ca "CA_Cert_1"
        set cn "FGVM01TM905"
    end
```
**Client integrity check**
```
config vpn ssl web host-checksoftware
    show
```
**Debug SSL VPNs**
```
diagnose debug enable
diagnose vpn ssl list | info | statistics | debug-filter | hw-acceleration-status | tunnel-test | web-mode-test

diagnose debug application sslvpn -1
diagnose debug enable
```

___
**IKE custom port**
```
config system settings
    set ike-port <port>
end
```
**Disable IPsec hardware offload**
```
config vpn ipsec phase1-interface
    edit <tunnel-name>
        set npu-offload disable
    next
end
```

## Diagnostics
```
get system status

get hardware nic

get system arp

diagnose debug flow filter
`diagnose debug enable
diagnose debug flow trace start 
diagnose debug flow stop
```