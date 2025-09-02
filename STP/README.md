# Spanning Tree Protocol

This is a Layer 2 protocol defined in IEEE 802.1D. that prevent loops in a switched Ethernet network.

Without STP, redundant links in a switch network cause:

* Broadcast storms (infinite broadcast flooding).

* Multiple frame copies.

* MAC address table instability.

So STP automatically blocks some links to form a loop-free tree topology, while keeping redundancy for failover.

### How STP Works

#### Bridge ID (BID)

Each switch has a BID = Bridge Priority (default 32768) + MAC address.

Lowest BID wins and becomes Root Bridge.

#### Root Bridge Election

The switch with the lowest BID is chosen as the Root Bridge.

All ports on the root bridge are Designated Ports (forwarding).

#### Root Ports (RP)

On non-root switches, the port with the lowest path cost to the root bridge becomes the Root Port (forwarding).

Path cost depends on link speed (e.g., FastEthernet = 19, Gigabit = 4, etc.).

#### Designated Ports (DP)

For each network segment, the switch port with the lowest cost to root is Designated (forwarding).

Others on the segment are placed in Blocking state.

#### Blocked Ports

Any port that is neither Root Port nor Designated Port → Blocking to prevent loops.

### STP Port States

STP transitions ports through these states:

* Blocking, listens for BPDUs, doesn’t forward.

* Listening , processes BPDUs, not forwarding yet.

* Learning, builds MAC address table, not forwarding yet.

* Forwarding, forwards traffic normally.

* Disabled, administratively down.

### Basic Configs

Check STP status:

* _SW1# show spanning-tree_


Set Bridge Priority (to influence root election):

* _SW1(config)# spanning-tree vlan 1 priority 4096_

(default = 32768; lower wins)

Set Port Priority (influence designated port selection):

* _SW2(config-if)# spanning-tree vlan 1 port-priority 64_

Set Path Cost (influence root port selection):

* _SW2(config-if)# spanning-tree vlan 1 cost 19_