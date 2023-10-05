## Certificate operations

Fortigate uses certificate to:

- inspect traffic between to devices acting as a MiTM attack (generating certifcates on-demand) and as identification mechanism for people and devices. If the certificare is trusted by Fortigate, then the client is allowed to establish a full connection to/with the resource protected by Fortigate. On top of this behavior, other policies may alter determine whether the traffic is to be accepted or rejected
- enforce privacy using SSL/TLS for sensitive traffic encryption such as connections to web servers, FortiGuard, or web servers
- authenticate users using certificates issued and signed by trusted and known CA thus letting them access the network or establish VPN connections. Certificate can be used as a second factor authentication for administrators

A digital certificate is a digital document produced and signed by a CA. It identifies an end entity (such as a person), a device (such as example.com), a thing (such as a certificate revocation list).  

Fortigate identifies the person or device from the **Subject** field which is expressed as a distinguished name (DN).  
Fortigate can also use other identifiers such as **Subject Alternative Name** (network ID or email) and can establish the relationship between certificate and the issuer from fields **Subject Key Identifier** and **Authority Key Identifier**.  

Fortigate supports the X.509v3 certificate standard.

Before a certificate is trusted, it must pass several checks:

- Fortigate checks locally, against and updated CRL (Cetificate Revocation List) if the certificate is listed or not. If it is listed then it is no longer valid and not to be trusted. Fortigate also supports OCSP (Online Certificate Status Protocol) where FortiAuthentication acts as the OCSP responder
- reads the value in the **Issuer** field, if the CA certificate is missing then the certificate is not to be trusted. FortiOS uses the [Mozilla CA certificate store](https://wiki.mozilla.org/CA)
- checks the certification validity comparing current date to values from **Valid From** and **Valid to**
- validates the signature on the certificate (critical requirement)

Signature verification process can be broken down in 3 steps:

1. First (not actually related with Fortigate) the CA runs the content of the certificate through a hash function, resulting in a hash digest (referred to as **original hash result**). That hash is encrypted by the CA using its private key. The result is the digital signature
2. When Fortigate verifies the digital signature, it runs the certificate (digital signature) through a hash function (the same specified in the certificate)
3. Fortigate decrypts the hash result from step 2 using the CA public key, if the key cannot restore the encrypted hash result to its original value, then the verification fails
4. In the last step, Fortigate compared the original hash result to the local hash result, if they match then the integrity of the certificate is confirmed

Certificate-based user authentication uses an end-entity certificate which contains the user public key and the signature of the CA that issued the certificate. Fortigate performs all the checks above before succesfully authenticate the user.

Fortigate uses SSL to protect data privacy in communication over the network. SSL uses symmetric and asymmetric criptography:

- Symmetric criptography means that only one key is used to both encrypt and decrypt information and it is share between the two parties (or rather the data needed to produce it)
- Asymmetric criptography means that a set ot two keys is used, each with its own purpose: one for encryption and one for decryption. For example, Fortigate must send data over HTTPS so it uses the remote webserver publick key to encrypt the data while the webserver uses the private key (which must be kept private at any cost) to decrypt

Steps involved while establishin an SSL session (for example, with a remote web server configured to use SSL):

1. Initial hello message from the Fortigate which includes the SSL version number and the names of the cryptographic algorithms it supports
2. The web server receives the message from Fortigate and chooses the first suite of cryptographic algorithms included in the message and verifies that it is also supported by the web server. The Web server replies with the chosen SSL version and cipher suite and sends its certificate to Fortigate  
(The certificate is passed as plaintext over the public network because its content is typically public)
3. Fortigate validates the certificate performing a list of checks (described adove). If the certificate can be trusted then the SSL handshake proceeds
4. In the last step, Fortigate generates a value known as the **premaster secret** which is enrypted using the public key stored in the certificate. The encrypted premaster secret is sent to the web server which decrypts it using the private key. Since the encrypted file can be decrypted only by the private key associated to the public key, no one intercepting the encrypted premaster secret can decrypt it
5. The premaster secret is now a shared secret between Fortigate and the web server
6. Both Fortigate and the web server derive the **master secret** from the premaster secret
7. Based on the master secret value, both devices generates the session key which is a symmetric key. Symmetric key cryptography is much faster than asymmetric key criptography because requires less computational power using one key to both encrypt and decrypt files. Direct exchange of symmetric keys poses a severe security risks because if leaked or intercepted, an attacker can decrypt private messages or break the SSL tunnel acting as a Man in the Middle. **Exchange of secrets for generating symmetric keys over asymmetric cryptography is a much robust and safe way**.
8. As last step, both devices produce a summary (or digest) of the messages exchanged so far. The summary is encrypted using the session key. If the digests are the same then the SSL tunnel is secure, otherwise some other actor has altered the exchange of information in some way.

Once the SSL handshake is complete, both devices use the session key to encrypt and decrypt information exchanged.

If malwares or viruses manage to get through in encrypted traffic they become undetectable. That is why Fortigate can perform SSL traffic inspection.  

A Simple SSL Certificate Inspection parses only the header of a certificate trying to extract information about the sever name (from Subject or SAN field). This level of inspection can be associated with web filter and application control activities because packets are evaluated as single entitities. Fortigate does not inspect the flow of encrypted data between the outside web server and the internal client.  

A Full SSL Certificate Inspection, on the other hand, requires Fortigate to act as a web proxy and as a CA. The internal CA must generate a private key and certificate each time a client performs a connection and the entire process must place immediately to not delay the connection.  

The client web browser is not connected to the remote server but to Fortigate (web proxy). In order to do so, its CA certificate must have fields **cA=True** (identifies the certificate as a CA certificate) and **keyUsage=keyCertSign** (the corrisponding private key is permitted to sign certificates).

All Fortigate devices capable of Full Inspection are shipped with a self-signed Fortinet certificate **Fortinet_CA_SSL**. Internal CA certificate can also be installed in Fortigate.

Clients connecting to the web while Full Inspection is active must have installed the Fortigate certificate in order to trust the connection. Fortigate sends the chain of certificates to client (when custom CA certificate are installed) so they can build a chain of trust.

If the inspection is applied to outbound traffic, the option **Multiple clients connecting to multiple servers** must be enabled. On the contrary, the option for inbound traffic is **Protecting SSL Server**.  

Fortigate let administrators decide actions to be taken when web servers use _Unstrusted Certificates_:
- **Allow**: sends the browser a temporary self-signed certificate **Fortinet_CA_Unstrusted** which is not to be impoerted into the bworser's root CA store
- **Block**: the connection is blocked
- **Ignore**: sends the browser a temporary **Fortinet_CA_SSL** certifictae regardless of the SSL certificate status

When websites uses [HPKP](https://en.wikipedia.org/wiki/HTTP_Public_Key_Pinning) or when laws prohibits SSL inspection (such as for bank-related traffic), administrator ìs can exempt domains from full SSL Inspection or use Fortiguard always-updated list to bypass SSL for specific domains/categories.

Inbound Full SSL Inspection requires to import the server key pair into Fortigate which act as the final web server for the client, and as a client for the real web server.  
Fortigate can act as a MiTM for several server sharing the same external IP address. Based on the SNI in the certificate compared to the CN on the certificate, Fortigate uses the correct certificate and connect to the right web server. If no matching SNI is found, the first server certificate of the list is used. 

Other than web browsers, other applications use SSL encryption to protect their network communication such as Dropbox (which uses HPKP and must be exempted from SSL inspection) and Microsoft Outlook 365 (which requires the self-signed certificate to be imported into Windows root CA store).  
Lots of protocols use SSL encryption such as POP3S, STMPS, IMAPS, FTPS, LDAPS... which can suffer from misconfigured SSL inspection.  