# Multi_Access OSPF

Multi-access network is one where multiple routers share the same physical medium/subnet.

Example: Ethernet LANs, Frame Relay multipoint, etc.

Unlike point-to-point links (two routers only), here many OSPF neighbors can form on the same segment.

If every router on a multi-access segment tried to form a full OSPF adjacency with every other router, we would get an N² problem:

With 10 routers → 45 adjacencies,flooding LSAs (Link-State Advertisements) would overwhelm the network.

OSPF solves this using the Designated Router (DR) and Backup Designated Router (BDR):

#### DR (Designated Router)

* Acts as the “hub” for exchanging LSAs.

* All routers form a full adjacency only with the DR (and BDR).

* LSAs are sent to the DR → DR then floods them to everyone else.

#### BDR (Backup DR)

* Passive standby in case DR fails.

#### Other Routers are called DROthers.

* They form adjacencies only with DR/BDR, not with each other.

### DR/BDR Election

Election is based on OSPF priority (higher wins).

* _ip ospf priority <0-255>_

Priority 0 : router never becomes DR/BDR.

If priorities are equal : highest Router ID wins.

Once elected, DR/BDR stays until a failure (no re-election if a higher priority router comes online later).

### Configuring MUltiAccess OSPF

Take for instance all routers are in the same broadcast domain 10.1.1.0/24.

Assign IP Addresses

On all routers:

* _R1(config)# interface fa0/0_
* _R1(config-if)# ip address 10.1.1.1 255.255.255.0_
* _R1(config-if)# no shutdown_

* _R2(config)# interface fa0/0_
* _R2(config-if)# ip address 10.1.1.2 255.255.255.0_
* _R2(config-if)# no shutdown_

* _R3(config)# interface fa0/0_
* _R3(config-if)# ip address 10.1.1.3 255.255.255.0_
* _R3(config-if)# no shutdown_

* _R4(config)# interface fa0/0_
* _R4(config-if)# ip address 10.1.1.4 255.255.255.0_
* _R4(config-if)# no shutdown_

Configure OSPF in each router with the networks each is connected to. Use different Router-id for each router.

* _R1(config)# router ospf 1_
* _R1(config-router)# router-id 1.1.1.1_
* _R1(config-router)# network 10.1.1.0 0.0.0.255 area 0_

OSPF will elect a DR and BDR automatically with the highest priority winning (default = 1). If equal the one with highest Router ID wins.

For example in our case:

* R4 (Router ID 4.4.4.4) : DR

* R3 (Router ID 3.3.3.3) : BDR

* R1, R2 : Drother

If you want to force R1 as DR:

* _R1(config)# interface fa0/0_
* _R1(config-if)# ip ospf priority 200_


If you want a router never to be DR/BDR:

* _R2(config)# interface fa0/0_
* _R2(config-if)# ip ospf priority 0_

### Verification

Check neighbors:

* _R1# show ip ospf neighbor_

