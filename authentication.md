## Authentication

### Remote server authentication

It is possible to configure Fortigate authentication using RADIUS, TACACS+ and LDAP. There two ways to configurw and use external servers:

1. Create users accounts on Fortigate and select the remote server type (RADIUS, TACACS+ or LDAP). Point the Fortigate to the preconfigured remote authentication server and add the user to an appropriate group (usually done when 2fa will be enabled for users)
2. Add the remote authentication server to user groups. This method requires administrators to create a user group and add the preconfigured remote server to the group. In this way, any user who has an account on the remote server can authenticate. 


LDAP is a directory based protocol which stores objects and their status in a directory tree. The most common schema is Active Directory.  

RADIUS and TACACS+ are AAA services (Authentication, Authorization, Accounting). A typical handshake is like:
- client to server : ACCESS-REQUEST
- server to client: ACCESS-ACCEPT/ACCESS-REJECT/ACCESS-CHALLENGE

**ACCESS-ACCEPT**: everything is ok  
**ACCESS-REJECT**: credentials are wrong  
**ACCESS-CHALLENGE**: the server is requesting a secondary password ID token (such as 2fa tokens) or certificate

In order for the Fortigate to access the LDAP server, under **User & Authentication** > **LDAP Servers**

In order for the Fortigate to access the RADIUS server, under **User & Authentication** > **RADIUS Servers**

Under **User & Authentication** > **User Groups** it is possible to create guests account which are temporary. Guests accounts are useful for events and/or wireless clients. Guest Management Administrators only serve the sole purpose of managing guests. Fortigates can create guests accounts in bulk to reduce time.

**The only service that is allowed through firewall policies when users have not authenticated yet (or have failed the authentication) is DNS (and those services which include it, such as ALL)**

Active authotization requires credentials to be passed to a prompt, this is the case for local password authentication, server-based password authentication and 2fa.   
Passive authentication, on the other hand, tries to identify users behind the scene, not requiring credentials to be actively passed by users.

Active: LDAP, Radius, TACACS+
Passive: FSSO (Fortinet Single Sign-On), RSSO (RADIUS Single Sign-On)

Link: https://community.fortinet.com/t5/FortiGate/Technical-Tip-Active-and-passive-authentication-behavior/ta-p/192967

The presence of a fall-through policy (policy to dest X with no authentication after policy to dest X with authentication) implies that users are never asked to actively pass credentials.  
In order to force Fortigate to ask for credentials:
- active or passive authentication must be enabled on ALL policies  
OR  
- authentication must be force from the CLI (passive auth policies on top of active auth policies)  
OR  
- captive portal on the ingress interface so that users must authenticate BEFORE accessing resources

For security reasons, it is better to set a timeout on authentication preventing malicious actors to use the IP of a legitimate atuhenticated user. It also frees up memory space by forgetting information.  

- **Idle**: if no traffic is generated in a given timeframe, the user is logged out
- **Hard**: users are logged out no matter what when the timer hits the given value (maximum allowed time). The counter starts counting after successful login
- **New Session**: even if traffic is generated on existingin communication and established channel, if no new sessions are generated from the host device in a given timeframe, the user if logged out