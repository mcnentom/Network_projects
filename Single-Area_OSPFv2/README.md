# Single-Area_OSPFv2

OSPFv2 is the Interior Gateway Protocol (IGP) for IPv4.

Single area means all routers and links belong to Area 0 (the backbone).

Simpler to design and configure because:

* Only one LSDB (Link-State Database) exists.

* No need for ABRs (Area Border Routers).

To enable OSPF on Router

* _R1(config)# router ospf 1_              ! Start OSPF process ID 1
* _R1(config-router)# router-id 1.1.1.1_   ! Assign router ID

The Router ID is a unique 32-bit number. If not manually configured, OSPF will pick the highest loopback or highest active IP address.

Advertise Networks in Area 0

* _R1(config-router)# network 10.0.12.0 0.0.0.255 area 0_
* _R1(config-router)# network 10.0.14.0 0.0.0.255 area 0_
* _R1(config-router)# network 1.1.1.1 0.0.0.0 area 0_

network command tells OSPF:

* Which interfaces to run OSPF on.

* Which networks to advertise.

* Which area the interfaces belong to.

* 0.0.0.255 = wildcard mask (inverse of subnet mask).

### OSPF Hello and Neighbor Formation

OSPF sends Hello packets (multicast 224.0.0.5).

If neighbors match parameters (Hello/dead interval, area ID, authentication, subnet) they form adjacency.

Adjacencies exchange LSAs (Link State Advertisements).

### SPF Algorithm and Routing Table

Each router builds identical Link-State Database (LSDB).

Runs Dijkstra SPF algorithm to calculate the shortest path tree.

Installs the best routes in the routing table with O code:

* _R1# show ip route ospf_
O    10.0.23.0/24 [110/20] via 10.0.12.2, 00:00:12, FastEthernet0/0

### Verification

show ip ospf neighbor    ! See OSPF neighbors
show ip ospf database    ! Check LSDB
show ip protocols        ! OSPF process info
show ip route ospf       ! OSPF routes in table

