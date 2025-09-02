# IPsec vpn tunneling
IPsec(internet protocol secure) is a suite of protocols used to secure IP communications by authenticating and encrypting each IP packet in a communication session.
The original IP packet (with source/destination IPs) is encrypted and encapsulated.

A new IP header is added so the packet can travel across the intermediate network.

At the other end, the packet is decrypted and the original IP packet is delivered.

## Modes of IPsec Tunneling
### Transport Mode
* Encrypts only the payload of the IP packet (not the original IP header).
* Used for end-to-end communication (e.g., host-to-host).

### Tunnel Mode
* Encrypts the entire original IP packet (header + payload).
* A new IP header is added.
* Used for network-to-network or host-to-network VPNs.

IPsec tunneling occurs in two main/negotiation phases that is:

### Phase 1:Establishing the IKE(Internet Key Exchange) Security Association (IKE SA)

Builds a secure, authenticated channel for negotiating further security.

The devices (peers) agree on a security policy:

* Encryption algorithm (e.g., AES, 3DES)

* Hashing/integrity algorithm (e.g., SHA, MD5)

* Authentication method (Pre-Shared Key(PSK), RSA certificates, etc.)

* Diffie-Hellman group (for key exchange strength)

* Lifetime of the IKE SA

They use Diffie-Hellman to generate a shared secret key over the insecure internet.

Authentication is performed (e.g. checking the PSK).

After Phase 1, both peers have a secure encrypted channel (IKE SA) to negotiate IPsec parameters.

Two modes for Phase 1:

* Main Mode : More secure, hides identities, uses 6 messages.

* Aggressive Mode : Faster (only 3 messages), but less secure (identity not protected).

### Phase 2 : Establishing the IPsec Security Association (IPsec SA)

Negotiate the actual parameters for protecting user data traffic (the tunnel).

Uses the secure channel from Phase 1 to negotiate:

* Which IPsec protocol: ESP (Encapsulating Security Payload) or AH (Authentication Header).

* Encryption algorithm (e.g., AES, DES, 3DES).

* Integrity algorithm (SHA, MD5).

* Perfect Forward Secrecy (optional additional Diffie-Hellman exchange).

* Lifetime of the IPsec SA.

After this, the peers establish IPsec SAs, and the actual data (like packets from LAN-to-LAN) starts flowing securely.

Here only Quick Mode exists in Phase 2 (3 messages).

## Configuring IPsec
* _sh version_

//Check if the package for vpn is enabled,most wil be securityk9 package

* _license boot module c2900 technology-package securityk9_ 

 **note:** Enables the vpn package,always reload for effect
 
* _access-list 101 permit ip source net id subnet mask dest net id_

Phase1
* _crypto isakmp policy 10_

**note:** policy id can be 10,20,30; lower no eq high priority.

**note:** isakmp: Internet Security Association and Key Management Protocol

* _encryption aes_

**note:** set encryption to Advanced Encryption Standard

* _hash sha_

**note:** use sha for hashing.

* _authentication pre-share_

**note:** will use a preshared key for authentication with the other router/ASA

* _group 2_

**note:** selects use of deffie hellman group will be used allowing the two peers to generate a shared secret key over secure network.

**note:** The group number represents the strength and type of the DH key exchange i.e. the size of the prime modulus used in the calculation. inclde group 1,2,5,1419,20,24.

**note:** higher group strong encryption,more cpu power. Both peers must have a compatible group number.

* _lifetime 86400_

* _crypto isakmp key MYKEY address 200.1.1.2_

**note:** attaches/Associates the pre-shared key "MYKEY" with the ip of the other router int facing the internet.

Phase2
* _crypto ipsec transform-set MYSET esp-aes esp-sha-hmac_

**note:** Creates a transform set named MYSET. A transform set tells IPsec how to protect the data (encryption + integrity algorithms used in Phase 2)

**note:** esp-aes: choose AES for encryption (confidentiality). If unspecified size, AES-128 is the usual default; you can specify esp-aes 256 for AES-256.

**note:** esp-sha-hmac: choose (Hash-based Message Authentication Code) HMAC-SHA (SHA-1 HMAC) for integrity/authentication. Stronger options exist and are recommended where supported such as esp-sha256-hmac, esp-sha384-hmac, esp-sha512-hmac

**note:** Alternatively you can use GCM which does not need hmac specification as it laready provides integrity i.e crypto ipsec transform-set MYSET esp-gcm 256

**note:** the above command will choose 

* _crypto map MYMAP 10 ipsec-isakmp_

**note:** Creates a crypto map named MYMAP with sequence number 10 and specifies the type ipsec-isakmp (IKEv1 bind).

**note:**A crypto map is a container that ties together: peer, transform-set(s), ACL (interesting traffic) and optional settings (PFS, NAT traversal, etc.).

**note:** Sequence number (10) orders entries within that crypto map. Lower numbers are evaluated first. This number is local only peers don't need matching sequence numbers.

**note:** ipsec-isakmp: Indicates this map will use IKE (classic IKEv1) to negotiate keys/SAs. (IKEv2 configurations are done differently on newer IOS.)

* _set peer 200.1.1.2_

**note:** Defines the remote peer IP address for this crypto-map entry. This is the public/WAN IP of the other VPN endpoint.

**note:**You can set a DNS name or use dynamic peers (wildcard/dynamic crypto maps) in different configs but set peer <ip> is the most common static case.

**note:** If the peer has a dynamic IP (e.g., remote dial-in client), you use a different crypto map setup (e.g., crypto map ... set peer identity or a dynamic crypto map entry). For two site-to-site routers, static peer IP is typical.

* _set transform-set MYSET_

**note:** Attaches the previously defined transform-set MYSET to this crypto-map entry.

**note:** When traffic matches the ACL (next line), this crypto-map entry will offer MYSET (ESP-AES + HMAC-SHA) in Phase 2 negotiation. Both sides must negotiate a mutually-acceptable transform.

* _match address 101_

**note:** Tells this crypto-map entry which traffic to protect by referencing an access control list (ACL) 101.

**note:** This ACL defines the "interesting traffic" (source network -> destination network) that should be encrypted and sent through the tunnel.

