# DHCP Attacks

DHCP is a dynamic ip assigning protocol that atomatically configures host devices with IPs.

## Types of Attacks

### DHCP starvation

A malicious device sends many DHCP requests using spoofed MAC addresses until the DHCP server runs out of IP addresses in its pool.

The effects will be the denial of IPs for legitimate clients resulting in a denial of service attack.

By use of tools like Yersinia, Macof or custom scripts,an attacker can spoof mac addresses then floood the server with dhcp discover which will eventually exhaust the server's ip limit.

To mitigate this attack on layer 2 device, just configure port security as this will limit the number of Mac address the switch reads from a specific port.

#### configuring port security

* _interface fa0/1_
* _switchport mode access_
* _switchport port-security_
* _switchport port-security maximum 1_
* _switchport port-security mac-address sticky_
* _switchport port-security violation shutdown_


### DHCP Spoofing

This a type of attack where a hacker will introduce a rogue server into the network by connecting it to the switch hence the host devices get fake ips from this server.

With the attacker responding faster than the legitimate server it will trick he device to:

* Use attackers device as the gateway
* Send traffic through the attacker allowing them to steal data and intercept communicationa man-in-the-middle attack.

This form of attack can not be only solved with port security, thus the integration of dhcp snooping.

DHCP snooping allows access to only authorized traffic controller in our case a real DHCP server through:

* Allowing DHCP offers from trusted ports (Ports that do not connect the swith to end devices).
* Blocking DHCP replies from untrusted ports, thise connected to host devices.

#### Configuring DHCP Snooping

* _ip dhcp snooping_
**Note:** Initializes dhcp snooping
* _ip dhcp snooping vlan 10,20_
**Note:** if you are using vlan if not:
* _int range fa0/2_   //uplink ports or thse connected to server
* _ip dhcp snooping trust_
* _exit_

On the untrusted ports or access ports limit the number of dhcpdiscovery message,no needto specify as untrusted as the ports by default are.

* _ip dhcp snooping limit rate 4_

To verify:
* _show ip dhcp snooping_ //in Priviledge exec mode


# ARP Attacks

ARP (Address Resolution Protocol) is a protocol used to map IP addresses to MAC addresses in a local network. It helps devices know "Who has IP 192.168.1.1? Tell me your MAC."

## ARP Spoofing/Poisoning

This occurs when an attacker sends fake ARP messages to associate their own MAC address with the IP address of another device like the default gateway or another host.

The Attacker says: "I am 192.168.1.1 (the gateway)." leading to all devices sending traffic meant for the gateway to the attacker instead.
In the ARP table it will be notted that the MAC address is that of the attacker and not the real gateway.

This allows for: 

* Man-in-the-Middle (MitM) attacks

* Eavesdropping on private communications

* Session hijacking

* Denial of Service (DoS)

This type of attack is mitigated through: DAI

### Dynamic ARP Inspection (DAI)

DAI checks ARP messages on untrusted ports and blocks malicious ones.

It works with DHCP Snooping by validating MAC IP bindings.
HOW DAI works:

* When a host gets an IP via DHCP, the switch's DHCP Snooping feature records:

    MAC address

    IP address

    VLAN

    Interface it came from.

* This creates a trusted DHCP Binding Table a "known good" list.

* When DAI is enabled, the switch checks every ARP request/reply that arrives on untrusted ports.

* It compares the source IP and MAC in the ARP packet against the binding table.

* If the ARP info matches its allowed

* If it doesn't it is dropped 

Trusted ports:

Usually uplinks to other switches or routers.

DAI doesn't inspect ARP on these we assume they're safe.

Untrusted ports:

Edge access ports where end hosts connect these are inspected by DAI.

* _ip dhcp snooping_
* _ip dhcp snooping vlan 10 20_  //if you have vlans if not just configure the access ports
* _ip arp inspection vlan 10 20_
* _interface gig0/1_ 
* _ip dhcp snooping trust_
* _ip arp inspection trust_
* _ip arp inspection validatesrc-mac dst-mac ip_ 

# STP ATTACKS

Spanning Tree Protocol (STP) is used in switches to prevent Layer 2 loops by electing a root bridge and placing redundant ports into a blocking state.

In an STP attack, an attacker injects malicious Bridge Protocol Data Units (BPDUs) to:

* Force a new root bridge election (attacker becomes root).

* Disrupt network topology by causing ports to flap or go down.

* Redirect traffic through their device, enabling sniffing or man-in-the-middle (MitM) attacks.

This is often called a BPDU spoofing attack.

 #### Mitigation: PortFast

PortFast is configured on access ports (ports connected to end devices, not other switches).

It allows the port to skip the usual STP listening/learning states and transition immediately to the forwarding state.

PortFast should never be enabled on trunk or uplink ports.

Why it's important:

Prevents long delays when end devices boot up.

Doesn't protect against STP attacks alone, but is required to enable BPDU Guard.

* _int fa0/3_
* _spanning-tre porfast_

#### Mitigation 2:BPDU Guard

Monitors access ports with PortFast enabled for incoming BPDUs.

If a BPDU is received on an access port, it is assumed to be malicious, and the port is shut down immediately thus Preventing a rogue switch from participating in the STP process.

 if you have enabled Portfast you can enable BPDU guard on these ports by:

 * _spanning-tree portfast bpduguard default_

Or in an interface by:

* _int fa0/3_
* _spanning-tree bpduguard enable_








