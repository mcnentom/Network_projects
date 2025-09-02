# DHCPv6

DHCPv6 is the IPv6 version of DHCP (like we use in IPv4 to give out IPs automatically).

It allows hosts to dynamically obtain IPv6 addresses and other parameters (DNS, domain name, etc.).

Works together with IPv6 autoconfiguration (SLAAC).

## Two Main Modes of IPv6 Address Assignment

### Stateless

The router (via RA – Router Advertisement) tells the host the prefix.

The host generates its own IPv6 address (EUI-64 or random).

DHCPv6 is used only for extra info (DNS, NTP, domain name).

Router advertises: Other-config-flag = 1.

### Stateful

Works just like DHCPv4.

The DHCPv6 server provides the full IPv6 address and info.

Router advertises: Managed-config-flag = 1.

## DHCPv6 Configuration on Cisco Router

### SLAAC (Stateless Address Autoconfiguration)

Host generates its IPv6 address from the router’s RA (Router Advertisement) prefix.

No DHCPv6 server needed.

Router only advertises the prefix.

Host derives full address (using MAC/EUI-64 or random).

Router Config

* _R1(config)# ipv6 unicast-routing_

* _R1(config)# interface g0/0_
* _R1(config-if)# ipv6 address 2001:db8:1::1/64_
* _R1(config-if)# ipv6 enable_

! By Default = SLAAC only (M=0, O=0 in RA)

PC Config

* _PC1(config)# interface g0/0_
* _PC1(config-if)# ipv6 address autoconfig_

### Stateless DHCPv6

Router gives the prefix via RA (like SLAAC).

Hosts generate their own addresses but use DHCPv6 for extra info (DNS, domain).

Router sets the Other-config flag = 1.

Router Config

* _R1(config)# ipv6 unicast-routing_

 Interface config

* _R1(config)# interface g0/0_
* _R1(config-if)# ipv6 address 2001:db8:1::1/64_
* _R1(config-if)# ipv6 enable_
* _R1(config-if)# ipv6 nd other-config-flag_   ! tells hosts to use DHCPv6

 DHCPv6 Pool

* _R1(config)# ipv6 dhcp pool Ip-POOL_
* _R1(config-dhcpv6)# dns-server 2001:db8::53_
* _R1(config-dhcpv6)# domain-name stateless.local_

 Bind pool to interface

* _R1(config)# interface g0/0_
* _R1(config-if)# ipv6 dhcp server Ip-POOL_

 PC Config
* _PC1(config)# interface g0/0_
* _PC1(config-if)# ipv6 address autoconfig_

Step 1: Enable IPv6 routing

* _R1(config)# ipv6 unicast-routing_

Step 2: Configure the interface toward LAN

* _R1(config)# interface g0/0_
* _R1(config-if)# ipv6 address 2001:db8:1::1/64_
* _R1(config-if)# ipv6 enable_
* _R1(config-if)# ipv6 nd managed-config-flag _  ! tells hosts to use DHCPv6 stateful

Step 3: Create a DHCPv6 pool

* _R1(config)# ipv6 dhcp pool DHCPv6-LAN_
* _R1(config-dhcpv6)# address prefix 2001:db8:1::/64_  ! prefix given to clients
* _R1(config-dhcpv6)# dns-server 2001:db8::53_          ! optional DNS server
* _R1(config-dhcpv6)# domain-name mynet.local_

Step 4: Bind DHCPv6 pool to the interface

* _R1(config)# interface g0/0_
* _R1(config-if)# ipv6 dhcp server DHCPv6-LAN_

DHCPv6 Client Configuration

* _PC1(config)# interface g0/0_
* _PC1(config-if)# ipv6 address dhcp_

