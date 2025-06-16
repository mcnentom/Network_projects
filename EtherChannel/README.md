# EtherChannels

This a load balancing technique that aggregates multiple links between intermediary devices into bundles enabling a network to have redundancy, loop prevention and increased bandwidth.

In this setup the intermediary devices is configured to view the multiple bundles connected to it as one logical interface.

Occassionally, STP will tend to block the multiple physical links and use only one physical link in packet transmission.

If one physical link such as fa0/1 can transmit 100mbps, using two of this links will transmit 200mbps,the speed increases as you ncrease number of physical interface.

To overcome the issue with STP,etherchannel is configured.

One advantage of etherChannel is even if one of the physical link goes down the other links in the same channel group will still transmit data packets.

To configure etherchannel two protocols are used:

## PAGB

This is the Port Aggregation Protocol.

It is specific to cisco devices.

For configuration for instance using two switch S1 and S2 the following modes will work:
 

|  S1        |    S2           |Negotiation   | Channel      |
|:----------:|:---------------:|:------------:|:------------:|
|  On        |    On           |  NO          | Forms        |
|  On        | Auto/Desirable  |  NO          | Doesn't form |
|  Auto      |    Auto         |  NO          | Doesn't form |
|  Auto      |   Desirable     |   YES        | Forms        |
|  Desirable |  Desirable      |   YES        | Forms        | 
| Desirable  |   Auto          |   YES        | Forms        |

## LACP

This Link Aggregation Control Protocol

Works with both Cisco devices and other vendor devices.

For configurations:
|  S1        |    S2           |Negotiation   | Channel      |
|:----------:|:---------------:|:------------:|:------------:|
|  On        |    On           |  NO          | Forms        |
|  On        | Passsive/Active |   No         | Doesn't Forms|
|  Active    | Active/passive  |  YES         | forms        |
|  Passive   |   Passsive      |   No         | Doesn't Forms|
|  Passive   |   Active        |   Yes        |  Forms       |

## PAgB Commands

* _int range fa0/1-4_    
**Note:** This are the physical interfaces connected to the device and which are to be grouped into one group
* _channel-group 1 mode desirable_
* _exit_

**Note:** If this the configuration on one intermediary device say S1,then for the other device say S2 the _mode_ can be either _auto_ or _desirable_ for successful channel formation.

**Note:**the identifier is 1,for easier negotiation and avoiding confusion when troubleshooting it is advised that the identifier for the interfaces on S1 and S2 needed to form one group be identical.

* _int port-channel 1_
* _switchport mode trunk_

**Note:** if you are using VLANS add  this to allow vlan trunking between the devices.

* _switchport trunk allowed vlan 1,10,20,30_
* _exit_

**Note:** the vlans on one devices should be familiar to the other devices.For instance if S1 has vlan 10,20,30 then S2 hould be familiar with this vlans.

## LACP Commands

* _int range fa0/1-4_    
* _channel-group 1 mode active_
* _exit_

**Note:** If this the configuration on one intermediary device say S1,then for the other device say S2 the _mode_ can be either _active_ or _passive_ for successful channel formation.

* _int port-channel 1_
* _switchport mode trunk_

**Note:** if you are using VLANS add  this to allow vlan trunking between the devices.

* _switchport trunk allowed vlan 1,10,20,30_
* _exit_

**Note:** On layer 3 switch if there are vlans and inter-vlan routing need to be done,always enable routing using 
*_ip routing_
