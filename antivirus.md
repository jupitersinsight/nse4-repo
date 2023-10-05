## Antivirus

Techniques used by Fortigate to detect viruses (performed in the following order when all active):

- **Antivirus scan**: simpler and faster. Look for viruses matching against signatures in the antivirus database

- **Grayware scan**: aims at identifying graywares, unsolicited programs installed without user's knowledge or consent. They are not technically viruses but are not that safe so are categorized as malware. Signature matching from Fortiguard is usually enough to detect them

- **Machine Learning (AI) scan**: these scans are based on probability, so they increase the possibility of false positives but they can also detect sero-day attacks. Resource-intensive, identified threats are identified with the `W32/AI.Pallas Suspicious` signature  

If AI scan is too uncertain, administrators can embed FortiSandbox (cloud or appliance) for malware analysis for high-certainty malware detection and zero-day viruses identification. The analysis is performed in isolated VMs. 

When Fortigate sends original files to FortiSandbox it uses a proxy-based feature named **Content Disarm and Reconstruction**.  
This feature remove malicious content from inspected files replacing it with content known to be safe so that traffic can pass. 
Content that can be scanned includes PDF and Microsoft Office documents leaving the network on CDR-supported protocols, such as HTTP, SMTP, IMAP, POP3.  
The original file is sent to FortiSandbox for inspection.  

Information needed to determine if a file is suspicious are update by Fortiguard based on the current threat climate.  
Administrators can enable settings to enable Fortigate to use FortiSandbox signatures to supplement Fotiguard antivirus database.  
**Securit Profile** > **Antivirus**.

In proxy inspection mode, Fortigate supports FortiSandbox inline scanning, (CLI activation).

Multiple antivirus databases exist, which administrators can configure from the CLI:
- **Extended database**: containes signatures for viruses that have been detected in recent months, it also detects viruses no longer active
- **Extreme database**: intended for use in high-security environments since it detects all known viruses, viruses for legacy systems included

Other than CDR, Antivirus profile includes two more useful feature:
- **Virus Outbreak Prevention**: requires a Fortigate Zero-Hour Virus license (ZHVO) which enables Fortigate to add hash-based virus detection for new threats that are not yet detected by the antivirus signatures
- **Malware Block List**: Fortigate can link a dynamic external malware block list, hosted on web server where HTTP/HTTPS is available, plaintext file composed by multiple encrypted signatures (MD5, SHA1, SHA256)

Antivirus can operate in flow-based or proxy-based inspection mode.  
Flow-based inspection mode uses a hybrid of proxy-based scanning mode (default scanning mode and legacy scanning mode).  
The default mode enchances the scanning od nested archive files without buffering the container archive file.  
The legacy mode buffers the full container and then scans it.  

The IPS engine reads the payload of each packet, caches a local copy, and forwards the packet ro the receiver at the same time.  
Antivirus flow-based mode is a CPU-intensive proces because the file is transmitted simultaneously , but on some models such operations can be offloaded to dedicated SPUs (chipsets) to improve performance.  
When the last packet of the file is received by Fortigate, it puts the packet on hold and sends the file to the IPS engine.  
The IPS engine extracts the payload and assembles the whole file which is sent to the AV engine.  

When a virus is detected on a TCP session, Fortigate resets the connection and does not send the last piece of data. Since the client never received the entire file, it cannot be opened.   
The IPS engine also caches the URL of the infected file, so that if a second attempt is made, the IPS engine will then send a block replacement message to the client immediately.

If the virus is detected at the start of the connection, the IPS engine sends the block replacement message immediately.  
**Security Profiles** > **Antvirus**.

Flow-based inspection mode for **performance**.  
Proxy-based inspection mode for **security**.

Proxy-based inspection mode makes Fortigate to buffer the whole file first before scanning it. If clean, it is forwarded but the client has to wait until the scan is finished.  
If files are bigger than the buffer they are not scanned, they are either passed of blocked.

To avoid session timeout for protocols such as HTTP and FTP, from the CLI admnistrators can enable _client comforting features_. In this way, Fortigate transmits little pieces of data in order to keep the session alive and avoid timeouts.  

Key-aspect of the antivirus running in proxy-based mode, is **stream-based scanning**.  
Stream-based scanning scans large archive files by decompressing the files and then scanning and extracting them at the same time. Viruses are detected even if they are in the middle or towards the end of these large files.  

**Policy & Objects** > **Protocol Options**.  
Configuring Protocol Options allows administrators the configuration of protocol port mappings, common options (block oversized email/file...), web and email options.  

Files too big for the buffer bypasses scans by default. The buffer size limit can be adjusted from the CLI but, a buffer size limit too small may results in more viruses bypassing the scan, while a buffer size limit too big will increase communications latency.  

From the CLI is possible to log oversized files so to keep track of how many oversized files are processed by Fortigate and derive a fair buffer size limit.  

Large files are often compressed. Then compressed files go through scanning, compression acts like encryption: the signature won't match.  
The first step is to identify the compression algorithm used, Fortigate can identify it (most often) from the header. If archives are password protected they cannot be decompressed and scanned.  

Decompressed files are store in RAM, incresing the buffer size limit enables Fortigate to process larger files but increases latency.  
In case of nested archive, Fortigate tries to decompress all layers (up tp 12 by default, 100 max supported).

**Best practices**

- Enable antivirus scanning on all Internet traffic, including internal-to-external and any VIP firewall policies
- Use deep-inspection instead of certificate-base inspection to ensure that full content inspection is performed
- Use FortiSandbox for protection against new viruses
- Do not increase the maximum file size to be scanned, unless there is good reason, or you need to do so in order to meet a network requirement
- Enable logging for oversized files
- Offload inspection to dedicated network processors (when NTurbo is supported) and offload content scanning to dedicated processors (CP8 and CP9), only for flow-based