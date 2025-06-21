# Port-Security

A mitigation technique used to limit the number of mac address connected on a port which helps prevent layer 2 attacks such as MAC addresses table flooding.

## MAC Table Flooding Attack

This a type of Layer 2 (Data Link Layer) network attack where an attacker floods a switch with tons of fake MAC addresses from a single port using tools like _Macof_. 

Eventually it overwhelms the switch's MAC address table, which maps MAC addresses to physical ports.

How It Works

* A switch learns and stores MAC addresses dynamically as devices communicate.
* The MAC address table has limited space (e.g., 8,000 entries).
* A malicious device sends thousands of fake MAC addresses.
* The switch's table fills up so new legitimate MAC addresses can't be learned.
* As a result, the switch starts broadcasting all frames out of all ports, not just the destination port.
* The attacker can then sniff (capture) the traffic meant for other devices in a form of eavesdropping.

With port-security enabled the number of MAC addresses learnt from a port is limited with excessive or unkonown traffic causing the port to go into:

* shutdown
* Restriction
* Dropping the packets or protect mode.

Configuring port security:

* _interface fa0/1_

 **Note:** one can use the range here,it depends on the number of access port.For range: int range fa0/1-5
* _switchport mode access_

**Note:** Sets the port to an access port for end devices
* _switchport port-security_

**Note:** Intializes port security configuration
* _switchport port-security maximum 1_

**Note:** sets the maximum number of Mac addresses to be read from the port
* _switchport port-security violation shutdown_

**Note:** When the port security is violated it turns down the switch port
* _switchport port-security mac-address sticky_

**Note:** Enables the switch learn Mac addresses dynamically

## VLAN Attacks
VLANs are used to segment network traffic for better security, performance, and organization. However, if not properly secured, they can be exploited using VLAN attacks, primarily:

### VLAN Hopping Attacks
These allow an attacker to send traffic into a VLAN they are not authorized to access.

There are of two main types:

#### Switch Spoofing (Auto Trunking Attack)

* Most switches by default use Dynamic Trunking Protocol (DTP) to automatically negotiate trunk links.

* If a malicious user connects to an access port and pretends to be a switch, the port may become a trunk port.

Now, the attacker using tools like _Yersinia_ can send tagged traffic for any VLAN and access restricted VLANs.

Prevention: Disable DTP / Auto-Trunking

Here is how you do it on the switch:

* _switchport mode trunk_ //For the ports used for trunking
* _switchport nonegotiate_  //Disables DTP negotiation on that port

#### Double Tagging Attack
The attacker tags a packet with two VLAN IDs:

Outer tag = native VLAN (e.g., VLAN 1)

Inner tag = target VLAN (e.g., VLAN 10)

Switch strips off the outer tag (native VLAN) and forwards it to the next trunk.

The inner tag remains, and the next switch forwards it to VLAN 10.

Prevention:

Set the native VLAN to an unused VLAN, like VLAN 999.

Here is how you do it on the switch:

* _switchport trunk native vlan 999_     //Makes all the trunk links to have a vlan that is not the default one.

Full setup Configuration of the Project:

* _int range fa0/3-5_
* _switchport mode access_
* _switchport access vlan 20_
* _switchport port-security_
* _switchport port-security maximum 2_
* _switchport port-security mac-address sticky_
* _switchport port-security violation shutdown_
* _exit_
* _int range fa0/6-24_
* _switchport mode access_
* _switchport access vlan 1001_
* _shutdown_
* _exit_
* _int fa0/2_
* _switchport mode trunk_
* _switchport nonegotiate_
* _switchport trunk native vlan 999_
* _exit_

**Note:**
* port fa0/3 to 5 were access port
* Port fa0/6-24 were the unused that is why they are turned off. it is a good practice to keep this ports off.
* Port fa0/2 was the trunk port where now DTP is disabled.



