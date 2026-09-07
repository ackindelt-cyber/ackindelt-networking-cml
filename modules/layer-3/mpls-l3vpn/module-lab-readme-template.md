# Lab Guide — MPLS L3VPN Service Provider Lab

## Overview

This lab simulates a small service provider network delivering Layer 3 VPN connectivity between two customer sites using MPLS.

The provider core uses IS-IS for internal routing, LDP for MPLS label distribution, and MP-BGP to exchange customer VPN routes between provider edge routers. VRFs are used on the provider edge routers to maintain customer routing separation while MPLS transports traffic across the shared provider infrastructure.

The lab demonstrates how customer traffic can traverse a provider-managed MPLS L3VPN while the internal provider router remains unaware of customer LAN routes. This models a common service provider WAN architecture and provides hands-on practice with the control-plane and forwarding relationships between IS-IS, LDP, MPLS, VRFs, and MP-BGP.

Lab Status: In Progress

End-to-End Verification: Not Tested

---

## Objectives
* [ ] Configure IS-IS as the provider core routing protocol.
* [ ] Configure MPLS and LDP across the provider core.
* [ ] Configure customer VRFs on the provider edge routers.
* [ ] Configure MP-BGP VPNv4 peering between provider edge routers.
* [ ] Exchange customer site routes across the MPLS L3VPN.
* [ ] Verify MPLS label distribution and label-switched forwarding.
* [ ] Verify customer routes are installed in the correct VRFs.
* [ ] Validate end-to-end connectivity between the two customer sites.
* [ ] Confirm the provider core router does not require customer LAN routes.

---

## Topology

This lab uses two simulated customer sites connected across a small MPLS service provider network.

Each customer site contains an access switch with an SVI used as the end-to-end test endpoint, a distribution switch providing the customer LAN gateway, and a customer edge router connecting the site to the provider.

The provider network consists of two provider edge routers and one internal provider router. PE1 and PE2 provide MPLS L3VPN services to the customer sites, while P1 operates only as an MPLS core router and does not maintain customer LAN routes.

![Topology Diagram](topology/<diagram-file-name>)

---

## Tables

### Physical Links

| Local Device | Local Interface      | Peer Device | Peer Interface       | Purpose                              |
| ------------ | -------------------- | ----------- | -------------------- | ------------------------------------ |
| TSA1         | GigabitEthernet0/0   | TSD1        | GigabitEthernet0/0   | Site A access-to-distribution link   |
| TSD1         | GigabitEthernet0/1   | TSE1        | GigabitEthernet0/0   | Site A internal routed uplink        |
| TSE1         | GigabitEthernet0/1   | PE1         | GigabitEthernet0/0   | Site A CE-to-PE WAN link             |
| PE1          | GigabitEthernet0/1   | P1          | GigabitEthernet0/0   | Provider MPLS core link              |
| P1           | GigabitEthernet0/1   | PE2         | GigabitEthernet0/0   | Provider MPLS core link              |
| PE2          | GigabitEthernet0/1   | TSE2        | GigabitEthernet0/1   | Site B PE-to-CE WAN link             |
| TSE2         | GigabitEthernet0/0   | TSD2        | GigabitEthernet0/1   | Site B internal routed uplink        |
| TSD2         | GigabitEthernet0/0   | TSA2        | GigabitEthernet0/0   | Site B distribution-to-access link   |

### Customer Site Addressing

| Site | Device | Interface / SVI     | VLAN | IP Address    | Prefix | Purpose                     |
| ---- | ------ | ------------------- | ---- | ------------- | ------ | --------------------------- |
| A    | TSA1   | Vlan10              | 10   | 10.10.10.10   | /24    | Site A test endpoint        |
| A    | TSD1   | Vlan10              | 10   | 10.10.10.1    | /24    | Site A LAN gateway          |
| A    | TSD1   | GigabitEthernet0/1  | —    | 10.10.255.1   | /30    | TSD1-to-TSE1 transit        |
| A    | TSE1   | GigabitEthernet0/0  | —    | 10.10.255.2   | /30    | TSE1-to-TSD1 transit        |
| B    | TSA2   | Vlan20              | 20   | 10.20.20.10   | /24    | Site B test endpoint        |
| B    | TSD2   | Vlan20              | 20   | 10.20.20.1    | /24    | Site B LAN gateway          |
| B    | TSD2   | GigabitEthernet0/1  | —    | 10.20.255.1   | /30    | TSD2-to-TSE2 transit        |
| B    | TSE2   | GigabitEthernet0/0  | —    | 10.20.255.2   | /30    | TSE2-to-TSD2 transit        |

### Provider Addressing

| Device | Interface             | IP Address   | Prefix | VRF / Role                                  |
| ------ | --------------------- | ------------ | ------ | ------------------------------------------- |
| TSE1   | GigabitEthernet0/1    | 172.16.1.1   | /30    | Site A customer edge                        |
| PE1    | GigabitEthernet0/0    | 172.16.1.2   | /30    | CUSTOMER_A                                  |
| PE1    | Loopback0             | 1.1.1.1      | /32    | Provider loopback / MP-BGP / LDP router ID  |
| PE1    | GigabitEthernet0/1    | 10.0.12.1    | /30    | Provider core                               |
| P1     | GigabitEthernet0/0    | 10.0.12.2    | /30    | Provider core                               |
| P1     | Loopback0             | 2.2.2.2      | /32    | Provider loopback / LDP router ID           |
| P1     | GigabitEthernet0/1    | 10.0.23.1    | /30    | Provider core                               |
| PE2    | GigabitEthernet0/0    | 10.0.23.2    | /30    | Provider core                               |
| PE2    | Loopback0             | 3.3.3.3      | /32    | Provider loopback / MP-BGP / LDP router ID  |
| PE2    | GigabitEthernet0/1    | 172.16.2.1   | /30    | CUSTOMER_A                                  |
| TSE2   | GigabitEthernet0/1    | 172.16.2.2   | /30    | Site B customer edge                        |

### Customer Routing

| Device | Route / Relationship              | Next Hop / Peer | Purpose                                          |
| ------ | --------------------------------- | --------------- | ------------------------------------------------ |
| TSA1   | Default gateway                   | 10.10.10.1      | Sends non-local traffic to TSD1                  |
| TSD1   | Default route                     | 10.10.255.2     | Sends non-local Site A traffic toward TSE1       |
| TSE1   | 10.10.10.0/24                     | 10.10.255.1     | Provides return reachability to Site A LAN       |
| TSE1   | Default route                     | 172.16.1.2      | Sends remote-site traffic toward PE1             |
| PE1    | 10.10.10.0/24 in CUSTOMER_A       | 172.16.1.1      | Imports Site A LAN reachability into the L3VPN   |
| PE2    | 10.20.20.0/24 in CUSTOMER_A       | 172.16.2.2      | Imports Site B LAN reachability into the L3VPN   |
| TSE2   | Default route                     | 172.16.2.1      | Sends remote-site traffic toward PE2             |
| TSE2   | 10.20.20.0/24                     | 10.20.255.1     | Provides return reachability to Site B LAN       |
| TSD2   | Default route                     | 10.20.255.2     | Sends non-local Site B traffic toward TSE2       |
| TSA2   | Default gateway                   | 10.20.20.1      | Sends non-local traffic to TSD2                  |

### IS-IS Underlay

| Device | IS-IS Process | NET                         | Level   | Loopback0  | Enabled Links                  |
| ------ | ------------- | --------------------------- | ------- | ---------- | ------------------------------ |
| PE1    | CORE          | 49.0001.0001.0001.0001.00   | Level 2 | 1.1.1.1/32 | Loopback0, PE1–P1              |
| P1     | CORE          | 49.0001.0002.0002.0002.00   | Level 2 | 2.2.2.2/32 | Loopback0, P1–PE1, P1–PE2      |
| PE2    | CORE          | 49.0001.0003.0003.0003.00   | Level 2 | 3.3.3.3/32 | Loopback0, PE2–P1              |

### LDP / MPLS Transport

| Device | LDP Router ID | MPLS-Enabled Links   | LDP Neighbors          | Role                      |
| ------ | ------------- | -------------------- | ---------------------- | ------------------------- |
| PE1    | 1.1.1.1       | PE1–P1               | 2.2.2.2                | Provider edge LSR         |
| P1     | 2.2.2.2       | P1–PE1, P1–PE2       | 1.1.1.1, 3.3.3.3       | Transit LSR               |
| PE2    | 3.3.3.3       | PE2–P1               | 2.2.2.2                | Provider edge LSR         |

### MP-BGP VPNv4

| Device | Local AS | Peer | Peer Address | Update Source | Address Family | Purpose                                          |
| ------ | -------- | ---- | ------------ | ------------- | -------------- | ------------------------------------------------ |
| PE1    | 65000    | PE2  | 3.3.3.3      | Loopback0     | VPNv4          | Exchanges customer VPN routes and VPN labels     |
| PE2    | 65000    | PE1  | 1.1.1.1      | Loopback0     | VPNv4          | Exchanges customer VPN routes and VPN labels     |

### L3VPN Service

| Device | VRF        | RD        | Import RT | Export RT | Customer Prefix |
| ------ | ---------- | --------- | --------- | --------- | --------------- |
| PE1    | CUSTOMER_A | 65000:101 | 65000:100 | 65000:100 | 10.10.10.0/24   |
| PE2    | CUSTOMER_A | 65000:102 | 65000:100 | 65000:100 | 10.20.20.0/24   |


---

## Configuration Steps

> **Note:** The CLI examples below are annotated for readability. Clean device configurations are available in [`configs/`](configs/).

> **Design note:** Add any important design, protocol, or lab-specific warning here. Keep this specific to the lab.

**TSA1**

```bash
# Basic device configuration
enable # Enters privileged EXEC mode.
configure terminal # Enters global configuration mode.
hostname TSA1 # Sets hostname.
no ip domain-lookup # Disables DNS lookup for mistyped or unrecognized CLI commands.

# Create VLAN 10
vlan 10 # Creates VLAN 10 for the Site A customer LAN
name SITE_A_LAN # Assigns a descriptive name to VLAN 10

# Place interface into VLAN 10
interface gigabitethernet0/0 # Enters the interface connected to TSD1
description LINK_TO_TD1 # Identifies the interface as the uplink to TSD1
switchport mode access # Configures the TSD1-facing interface as a static Layer 2 access port.
switchport access vlan 10 # Assigns TSA1-facing access port to VLAN 10
no shutdown # Enables the physical interface toward TSD1

#Creates and configures SVI vlan10
interface vlan10 # Enters the Layer 3 SVI for VLAN 10
description SITE_A_TEST_ENDPOINT # Identifies the SVI as the Site A test endpoint.
ip address 10.10.10.10 255.255.255.0 # Assigns the Site A test endpoint IP address to Vlan10.
no shutdown # Enables the Vlan10 SVI

#Sets default gateway
ip default-gateway 10.10.10.1 # Configures TSD1 as TSA1's gateway for non-local traffic

end # Returns to privileged EXEC mode.
write memory # Saves the running configuration.
```

**TSD1**

```bash
# Basic device configuration
enable # Enters privileged EXEC mode.
configure terminal # Enters global configuration mode.
hostname TSD1 # Sets hostname.
no ip domain-lookup # Disables DNS lookup for mistyped or unrecognized CLI commands.

# Create VLAN 10
vlan 10 # Creates VLAN 10 for the Site A Customer LAN
name SITE_A_LAN # Assigns a descriptive name to VLAN 10.

# Place interface into VLAN 10
interface GigabitEthernet0/0 # Enters the interface connected to TSA1.
description LINK_TO_TSA1 # Identifies the interface as the link toward TSA1.
switchport mode access # Configures the TSA1-facing interface as a static Layer 2 access port.
switchport access vlan 10 # Assigns the TSA1-facing access port to VLAN 10.
no shutdown # Enables the physical interface toward TSA1

#Creates and configures SVI vlan10
interface Vlan10 # Enters the Layer 3 SVI for the Site A customer LAN.
description SITE_A_LAN_GATEWAY # Identifies Vlan10 as the default gateway interface for the Site A LAN.
ip address 10.10.10.1 255.255.255.0 # Assigns the Site A LAN gateway address to Vlan10.
no shutdown # Enables the Vlan10 SVI.

ip routing # Enables Layer 3 routing on TSD1.

interface gigabitEthernet0/1 # Enters the routed uplink interface connected to TSE1
description LINK_TO_TSE1 # Identifies the interface as the routed uplink toward TSE1.
no switchport # Converts GigabitEthernet0/1 from a Layer 2 switchport into a Layer 3 routed interface.
ip address 10.10.255.1 255.255.255.252 # Assigns the TSD1-to-TSE1 transit address to the routed uplink.
no shutdown # Enables the routed uplink interface toward TSE1.

ip route 0.0.0.0 0.0.0.0 10.10.255.2 # Configures a default route toward TSE1 for all non-local traffic.

end # Returns to privileged EXEC mode.
write memory # Saves the running configuration.
```

**TSE1**

```bash
# Basic device configuration
enable # Enters privileged EXEC mode.
configure terminal # Enters global configuration mode.
hostname TSE1 # Sets hostname.
no ip domain-lookup # Disables DNS lookup for mistyped or unrecognized CLI commands.

interface GigabitEthernet0/0 # Enters the interface connected to TSD1.
description LINK_TO_TSD1 # Identifies the interface as the routed link toward TSD1.
ip address 10.10.255.2 255.255.255.252 # Assigns TSE1's address on the routed transit link to TSD1.
no shutdown # Enables the routed interface toward TSD1.

ip route 10.10.10.0 255.255.255.0 10.10.255.1 # Adds a static route to the Site A LAN through TSD1.

interface GigabitEthernet0/1 # Enters the interface connected to PE1.
description LINK_TO_PE1 # Identifies the interface as the WAN link toward PE1.
ip address 172.16.1.1 255.255.255.252 # Assigns TSE1's address on the CE-to-PE WAN link.
no shutdown # Enables the CE-to-PE WAN interface toward PE1.

ip route 0.0.0.0 0.0.0.0 172.16.1.2 # Configures a default route toward PE1 for all non-local traffic.


end # Returns to privileged EXEC mode.
write memory # Saves the running configuration.
```

**PE1**

```bash
# PE1 Configuration Block
enable # Enters privileged EXEC mode.
configure terminal # Enters global configuration mode.
hostname PE1 # Sets the device hostname.
no ip domain-lookup # Disables DNS lookup for mistyped or unrecognized CLI commands.

# Configures IS-IS as PE1's dynamic routing protocol for Layer 3 reachability across the provider core.
router isis PROVIDER_CORE # Creates the IS-IS process used for routing inside the provider core network.
net 49.0001.0001.0001.0001.00 # Assigns PE1's unique IS-IS NET within the provider IS-IS domain.
is-type level-2-only # Restricts PE1 to Level 2 IS-IS routing for the provider backbone.
exit # Returns to global configuration mode.

# Creates Loopback0 as PE1's stable control-plane endpoint for provider services.
interface Loopback0 # Enters the provider loopback interface used as a stable control-plane endpoint.
description PROVIDER_LOOPBACK # Identifies Loopback0 as PE1's provider control-plane interface.
ip address 1.1.1.1 255.255.255.255 # Assigns PE1's stable /32 control-plane address.
ip router isis PROVIDER_CORE # Advertises Loopback0 into the PROVIDER_CORE IS-IS process.
no shutdown # Enables Loopback0.
exit # Returns to global configuration mode.

# Configures PE1's routed provider-core link toward P1.
interface GigabitEthernet0/1 # Enters PE1's provider-core-facing interface toward P1.
description LINK_TO_P1 # Identifies the interface as PE1's provider-core link toward P1.
ip address 10.0.12.1 255.255.255.252 # Assigns PE1's address on the point-to-point provider core link to P1.
ip router isis PROVIDER_CORE # Enables IS-IS on the PE1-to-P1 link so the routers can form an adjacency.
mpls ip # Enables MPLS forwarding and LDP on PE1's provider-core-facing link toward P1.
no shutdown # Enables the physical provider-core interface toward P1.
exit # Returns to global configuration mode.

# Configures PE1's LDP control-plane settings from global configuration mode.
mpls ldp router-id Loopback0 force # Uses PE1's Loopback0 address as its stable LDP router ID.
mpls label protocol ldp # Sets LDP as the protocol used to distribute MPLS transport-label bindings.

# Creates and configures the CUSTOMER_A VRF.
ip vrf CUSTOMER_A # Creates and enters submode for a separate routing and forwarding table for Customer A.
rd 65000:101 # Gives PE1's CUSTOMER_A VPN routes a unique Route Distinguisher in the shared VPNv4 control plane.
route-target export 65000:100 # Tags routes exported from CUSTOMER_A with RT 65000:100 for remote VPN membership decisions.
route-target import 65000:100 # Imports VPNv4 routes tagged with RT 65000:100 into PE1's CUSTOMER_A VRF.
exit # Returns to global configuration mode.

# Configures PE1's customer-facing interface toward TSE1 and places it into the CUSTOMER_A VRF.
interface GigabitEthernet0/0 # Enters PE1's customer-facing interface toward TSE1.
description LINK_TO_TSE1 # Identifies GigabitEthernet0/0 as PE1's customer-facing link toward TSE1.
ip vrf forwarding CUSTOMER_A # Associates this interface with the CUSTOMER_A routing and forwarding table.
ip address 172.16.1.2 255.255.255.252 # Assigns PE1's address on the CUSTOMER_A CE-to-PE link toward TSE1.
no shutdown # Enables PE1's customer-facing interface toward TSE1.
exit # Returns to global configuration mode.

# Adds CUSTOMER_A's Site A LAN route from global configuration mode.
ip route vrf CUSTOMER_A 10.10.10.0 255.255.255.0 172.16.1.1 # Adds Site A's LAN route to the CUSTOMER_A VRF through TSE1.

# Enters PE1's provider BGP process from global configuration mode and defines PE2 as an iBGP neighbor.
router bgp 65000 # Creates PE1's BGP process using provider AS 65000.
neighbor 3.3.3.3 remote-as 65000 # Defines PE2's Loopback0 address as an iBGP neighbor in the same provider AS.
neighbor 3.3.3.3 update-source Loopback0 # Sources the iBGP session from PE1's stable Loopback0 address.

# Configures the shared MP-BGP VPNv4 address family used by PE1 and PE2 to exchange MPLS L3VPN routes.
address-family vpnv4 # Enters the VPNv4 address-family submode.
neighbor 3.3.3.3 activate # Enables PE2 as a VPNv4 neighbor.
neighbor 3.3.3.3 send-community extended # Sends extended communities, including Route Targets, with VPNv4 advertisements.
exit-address-family # Returns to BGP router configuration mode.

# Configures CUSTOMER_A's IPv4 BGP address family so local VRF routes can be injected into the VPNv4 control plane.
address-family ipv4 vrf CUSTOMER_A # Enters the CUSTOMER_A-specific IPv4 BGP address-family submode.
redistribute static # Injects CUSTOMER_A's static routes into BGP so they can be advertised through MP-BGP VPNv4.
exit-address-family # Returns to BGP router configuration mode.

exit # Leaves BGP router configuration mode and returns to global configuration mode.

end # Returns to privileged EXEC mode.
write memory # Saves the running configuration.
```

**P**

```bash
# <Device 2> Configuration Block
enable # Enters privileged EXEC mode.
configure terminal # Enters global configuration mode.
hostname P1 # Sets hostname.

router isis PROVIDER_CORE # Creates the IS-IS process used for Layer 3 routing inside the provider core network.
net 49.0001.0002.0002.0002.00 # Assigns P1's unique IS-IS NET within the provider IS-IS domain.
is-type level-2-only # Restricts P1 to Level 2 IS-IS routing for the provider core.

interface Loopback0 # Creates P1's stable control-plane interface for IS-IS reachability and its LDP router ID.
description PROVIDER_LOOPBACK # Identifies Loopback0 as P1's stable provider control-plane interface.
ip address 2.2.2.2 255.255.255.255 # Assigns P1's stable /32 loopback address used as its provider control-plane identifier.
ip router isis PROVIDER_CORE # Advertises P1's Loopback0 into the provider IS-IS process so the other provider routers can reach 2.2.2.2.

interface GigabitEthernet0/0 # Enters P1's provider-core-facing interface toward PE1.
description LINK_TO_PE1 # Identifies GigabitEthernet0/0 as P1's provider-core link toward PE1.
ip address 10.0.12.2 255.255.255.252 # Assigns P1's IP address on the point-to-point provider-core link toward PE1.
ip router isis PROVIDER_CORE # Enables IS-IS on P1's link toward PE1 so the two routers can form an IS-IS adjacency and exchange provider routes.
mpls ip # Enables MPLS forwarding and LDP on P1's core-facing link toward PE1.

interface GigabitEthernet0/1 # Enters P1's provider-core-facing interface toward PE2.
description LINK_TO_PE2 # Identifies GigabitEthernet0/1 as P1's provider-core link toward PE2.
ip address 10.0.23.1 255.255.255.252 # Assigns P1's IP address on the point-to-point provider-core link toward PE2.
ip router isis PROVIDER_CORE # Enables IS-IS on P1's link toward PE2 so the two routers can form an IS-IS adjacency and exchange provider routes.
mpls ip # Enables MPLS forwarding and LDP on P1's core-facing link toward PE2.

mpls ldp router-id Loopback0 force # Forces P1 to use its stable Loopback0 address, 2.2.2.2, as its LDP router ID.
mpls label protocol ldp # Sets LDP as the protocol P1 uses to distribute MPLS label bindings with its provider neighbors.

end # Returns to privileged EXEC mode.
write memory # Saves the running configuration.
```

**PE2**

```bash
# PE2 Configuration Block
enable # Enters privileged EXEC mode.
configure terminal # Enters global configuration mode.
hostname PE2 # Sets the device hostname.
no ip domain-lookup # Disables DNS lookup for mistyped or unrecognized CLI commands.

# Configures IS-IS as PE2's dynamic routing protocol for Layer 3 reachability across the provider core.
router isis PROVIDER_CORE # Creates the IS-IS process used for routing inside the provider core network.
net 49.0001.0003.0003.0003.00 # Assigns PE2's unique IS-IS NET within the provider IS-IS domain.
is-type level-2-only # Restricts PE2 to Level 2 IS-IS routing for the provider backbone.
exit # Returns to global configuration mode.

# Creates Loopback0 as PE2's stable control-plane endpoint for provider services.
interface Loopback0 # Enters the provider loopback interface used as a stable control-plane endpoint.
description PROVIDER_LOOPBACK # Identifies Loopback0 as PE2's provider control-plane interface.
ip address 3.3.3.3 255.255.255.255 # Assigns PE2's stable /32 control-plane address.
ip router isis PROVIDER_CORE # Advertises Loopback0 into the PROVIDER_CORE IS-IS process.
no shutdown # Enables Loopback0.
exit # Returns to global configuration mode.

# Configures PE2's routed provider-core link toward P1.
interface GigabitEthernet0/0 # Enters PE2's provider-core-facing interface toward P1.
description LINK_TO_P1 # Identifies the interface as PE2's provider-core link toward P1.
ip address 10.0.23.2 255.255.255.252 # Assigns PE2's address on the point-to-point provider core link to P1.
ip router isis PROVIDER_CORE # Enables IS-IS on the PE2-to-P1 link so the routers can form an adjacency.
mpls ip # Enables MPLS forwarding and LDP on PE2's provider-core-facing link toward P1.
no shutdown # Enables the physical provider-core interface toward P1.
exit # Returns to global configuration mode.

# Configures PE2's LDP control-plane settings from global configuration mode.
mpls ldp router-id Loopback0 force # Uses PE2's Loopback0 address as its stable LDP router ID.
mpls label protocol ldp # Sets LDP as the protocol used to distribute MPLS transport-label bindings.

# Creates and configures the CUSTOMER_A VRF.
ip vrf CUSTOMER_A # Creates a separate routing and forwarding table for Customer A.
rd 65000:102 # Gives PE2's CUSTOMER_A VPN routes a unique Route Distinguisher in the shared VPNv4 control plane.
route-target export 65000:100 # Tags routes exported from CUSTOMER_A with RT 65000:100 for remote VPN membership decisions.
route-target import 65000:100 # Imports VPNv4 routes tagged with RT 65000:100 into PE2's CUSTOMER_A VRF.
exit # Returns to global configuration mode.

# Configures PE2's customer-facing interface toward TSE2 and places it into the CUSTOMER_A VRF.
interface GigabitEthernet0/1 # Enters PE2's customer-facing interface toward TSE2.
description LINK_TO_TSE2 # Identifies GigabitEthernet0/1 as PE2's customer-facing link toward TSE2.
ip vrf forwarding CUSTOMER_A # Associates this interface with the CUSTOMER_A routing and forwarding table.
ip address 172.16.2.1 255.255.255.252 # Assigns PE2's address on the CUSTOMER_A CE-to-PE link toward TSE2.
no shutdown # Enables PE2's customer-facing interface toward TSE2.
exit # Returns to global configuration mode.

# Adds CUSTOMER_A's Site B LAN route from global configuration mode.
ip route vrf CUSTOMER_A 10.20.20.0 255.255.255.0 172.16.2.2 # Adds Site B's LAN route to the CUSTOMER_A VRF through TSE2.

# Enters PE2's provider BGP process from global configuration mode and defines PE1 as an iBGP neighbor.
router bgp 65000 # Creates PE2's BGP process using provider AS 65000.
neighbor 1.1.1.1 remote-as 65000 # Defines PE1's Loopback0 address as an iBGP neighbor in the same provider AS.
neighbor 1.1.1.1 update-source Loopback0 # Sources the iBGP session from PE2's stable Loopback0 address.

# Configures the shared MP-BGP VPNv4 address family used by PE1 and PE2 to exchange MPLS L3VPN routes.
address-family vpnv4 # Enters the VPNv4 address-family submode.
neighbor 1.1.1.1 activate # Enables PE1 as a VPNv4 neighbor.
neighbor 1.1.1.1 send-community extended # Sends extended communities, including Route Targets, with VPNv4 advertisements.
exit-address-family # Returns to BGP router configuration mode.

# Configures CUSTOMER_A's IPv4 BGP address family so local VRF routes can be injected into the VPNv4 control plane.
address-family ipv4 vrf CUSTOMER_A # Enters the CUSTOMER_A-specific IPv4 BGP address-family submode.
redistribute static # Injects CUSTOMER_A's static routes into BGP so they can be advertised through MP-BGP VPNv4.
exit-address-family # Returns to BGP router configuration mode.

exit # Leaves BGP router configuration mode and returns to global configuration mode.

end # Returns to privileged EXEC mode.
write memory # Saves the running configuration.
```

**CE2**

```bash
# CE2 Configuration Block

enable # Enters privileged EXEC mode.
configure terminal # Enters global configuration mode.
hostname CE2 # Sets the device hostname.
no ip domain-lookup # Disables DNS lookup for mistyped or unrecognized CLI commands.

# Configures CE2's routed link toward D2.
interface GigabitEthernet0/0 # Enters CE2's customer-LAN-facing routed interface toward D2.
description LINK_TO_D2 # Identifies the interface as the routed link toward D2.
ip address 10.20.255.2 255.255.255.252 # Assigns CE2's address on the CE2-to-D2 point-to-point network.
no shutdown # Enables the interface.
exit # Returns to global configuration mode.

# Configures CE2's routed WAN handoff toward PE2.
interface GigabitEthernet0/1 # Enters CE2's provider-facing interface toward PE2.
description LINK_TO_PE2 # Identifies the interface as the customer-to-provider handoff toward PE2.
ip address 172.16.2.2 255.255.255.252 # Assigns CE2's address on the CE-to-PE point-to-point network.
no shutdown # Enables the interface.
exit # Returns to global configuration mode.

# Adds routing for Site B's local LAN and remote destinations.
ip route 10.20.20.0 255.255.255.0 10.20.255.1 # Routes Site B's LAN toward D2.
ip route 0.0.0.0 0.0.0.0 172.16.2.1 # Sends remote traffic toward PE2 and the provider MPLS L3VPN.

end
write memory
```

**D2**

```bash
# D2 Configuration Block
enable # Enters privileged EXEC mode.
configure terminal # Enters global configuration mode.
hostname D2 # Sets the device hostname.
no ip domain-lookup # Disables DNS lookup for mistyped or unrecognized CLI commands.
ip routing # Enables Layer 3 routing on D2.

# Creates Site B's LAN VLAN.
vlan 20 # Creates VLAN 20 for the Site B LAN.
name SITE_B_LAN # Assigns a descriptive name to VLAN 20.
exit # Returns to global configuration mode.

# Configures D2's Layer 2 link toward TSA2.
interface GigabitEthernet0/0 # Enters D2's interface toward TSA2.
description LINK_TO_TSA2 # Identifies the interface as the link toward TSA2.
switchport mode access # Configures the interface as a Layer 2 access port.
switchport access vlan 20 # Places the interface into Site B VLAN 20.
no shutdown # Enables the interface.
exit # Returns to global configuration mode.

# Creates Site B's Layer 3 gateway.
interface Vlan20 # Enters the Site B VLAN 20 SVI.
description SITE_B_LAN_GATEWAY # Identifies the SVI as the default gateway for Site B.
ip address 10.20.20.1 255.255.255.0 # Assigns the Site B LAN gateway address.
no shutdown # Enables the SVI.
exit # Returns to global configuration mode.

# Configures D2's routed uplink toward CE2.
interface GigabitEthernet0/1 # Enters the routed interface toward CE2.
description LINK_TO_CE2 # Identifies the interface as the routed uplink toward CE2.
no switchport # Converts the interface from Layer 2 switching to Layer 3 routing.
ip address 10.20.255.1 255.255.255.252 # Assigns D2's address on the D2-to-CE2 point-to-point network.
no shutdown # Enables the interface.
exit # Returns to global configuration mode.

# Sends traffic for remote networks toward CE2.
ip route 0.0.0.0 0.0.0.0 10.20.255.2 # Uses CE2 as D2's default next hop for remote destinations.

end
write memory
```

**TSA2**

```bash
# TSA2 Configuration Block
enable # Enters privileged EXEC mode.
configure terminal # Enters global configuration mode.
hostname TSA2 # Sets the device hostname.
no ip domain-lookup # Disables DNS lookup for mistyped or unrecognized CLI commands.

# Creates Site B's LAN VLAN.
vlan 20 # Creates VLAN 20 for the Site B LAN.
name SITE_B_LAN # Assigns a descriptive name to VLAN 20.
exit # Returns to global configuration mode.

# Configures TSA2's Layer 2 link toward D2.
interface GigabitEthernet0/0 # Enters TSA2's uplink toward D2.
description LINK_TO_D2 # Identifies the interface as TSA2's link toward D2.
switchport mode access # Configures the interface as a Layer 2 access port.
switchport access vlan 20 # Places the interface into Site B VLAN 20.
no shutdown # Enables the interface.
exit # Returns to global configuration mode.

# Creates TSA2's test endpoint SVI.
interface Vlan20 # Enters TSA2's VLAN 20 SVI.
description SITE_B_TEST_ENDPOINT # Identifies the SVI as the Site B end-to-end test endpoint.
ip address 10.20.20.10 255.255.255.0 # Assigns TSA2's test endpoint address.
no shutdown # Enables the SVI.
exit # Returns to global configuration mode.

# Configures D2 as TSA2's default gateway.
ip default-gateway 10.20.20.1 # Sends off-subnet traffic toward D2.

end
write memory
```

---

## Customer LAN Verification

See [`verification/verification_commands.md`](verification/verification_commands.md) for recorded command output.

### Site A — Layer 2 Verification

**TSA1**

```bash
show interfaces GigabitEthernet0/0 status # Verifies the physical link toward TSD1 is connected and operational.
show interfaces GigabitEthernet0/0 switchport # Verifies Gi0/0 is operating as an access port assigned to VLAN 10.
show vlan brief # Verifies VLAN 10 exists, is active, and includes Gi0/0.
```

**TSD1**

```bash
show interfaces GigabitEthernet0/0 status # Verifies the physical link toward TSA1 is connected and operational.
show interfaces GigabitEthernet0/0 switchport # Verifies Gi0/0 is operating as an access port assigned to VLAN 10.
show vlan brief # Verifies VLAN 10 exists, is active, and includes Gi0/0.
```

### Site A — Layer 3 Verification

**TSA1**

```bash
show ip interface brief # Verifies Vlan10 is up/up with the expected 10.10.10.10 address.
show running-config | include ip default-gateway # Verifies TSA1 is configured to use TSD1 at 10.10.10.1 as its default gateway.
ping 10.10.10.1 # Verifies Layer 3 reachability from TSA1 to TSD1's VLAN 10 default gateway.
```

**TSD1**

```bash
show ip interface brief # Verifies Vlan10 and the routed link toward CE1 are up/up with the expected IP addresses.
show ip route 0.0.0.0 # Verifies TSD1 has a default route pointing toward CE1 at 10.10.255.2.
ping 10.10.255.2 # Verifies Layer 3 reachability from TSD1 to CE1 across the customer-edge routed link.
```

### Site B — Layer 2 Verification

**TSA2**

```bash
show interfaces GigabitEthernet0/0 status # Verifies the physical link toward TSD2 is connected and operational.
show interfaces GigabitEthernet0/0 switchport # Verifies Gi0/0 is operating as an access port assigned to VLAN 20.
show vlan brief # Verifies VLAN 20 exists, is active, and includes Gi0/0.
```

**TSD2**

```bash
show interfaces GigabitEthernet0/0 status # Verifies the physical link toward TSA2 is connected and operational.
show interfaces GigabitEthernet0/0 switchport # Verifies Gi0/0 is operating as an access port assigned to VLAN 20.
show vlan brief # Verifies VLAN 20 exists, is active, and includes Gi0/0.
```

### Site B — Layer 3 Verification

**TSA2**

```bash
show ip interface brief # Verifies Vlan20 is up/up with the expected 10.20.20.10 address.
show running-config | include ip default-gateway # Verifies TSA2 is configured to use TSD2 at 10.20.20.1 as its default gateway.
ping 10.20.20.1 # Verifies Layer 3 reachability from TSA2 to TSD2's VLAN 20 default gateway.
```

**TSD2**

```bash
show ip interface brief # Verifies Vlan20 and the routed link toward CE2 are up/up with the expected IP addresses.
show ip route 0.0.0.0 # Verifies TSD2 has a default route pointing toward CE2 at 10.20.255.2.
ping 10.20.255.2 # Verifies Layer 3 reachability from TSD2 to CE2 across the customer-edge routed link.
```

---

## Customer Edge Verification

### CE1 Verification

```bash
show ip interface brief # Verifies the routed interfaces toward TSD1 and PE1 are up/up with the expected IP addresses.
show ip route 10.10.10.0 # Verifies CE1 has a route to the Site A LAN through TSD1 at 10.10.255.1.
show ip route 0.0.0.0 # Verifies CE1 has a default route toward PE1 at 172.16.1.2.
ping 10.10.255.1 # Verifies Layer 3 reachability from CE1 toward TSD1.
ping 172.16.1.2 # Verifies Layer 3 reachability across the CE1-to-PE1 provider handoff.
```

### CE2 Verification

```bash
show ip interface brief # Verifies the routed interfaces toward TSD2 and PE2 are up/up with the expected IP addresses.
show ip route 10.20.20.0 # Verifies CE2 has a route to the Site B LAN through TSD2 at 10.20.255.1.
show ip route 0.0.0.0 # Verifies CE2 has a default route toward PE2 at 172.16.2.1.
ping 10.20.255.1 # Verifies Layer 3 reachability from CE2 toward TSD2.
ping 172.16.2.1 # Verifies Layer 3 reachability across the CE2-to-PE2 provider handoff.
```

---

## Provider Verification

### IS-IS Verification

**PE1**

```bash
show isis neighbors # Verifies PE1 has formed an IS-IS adjacency with P1.
show ip route isis # Verifies PE1 is learning provider routes through IS-IS, including reachability toward PE2.
ping 3.3.3.3 source 1.1.1.1 # Verifies end-to-end provider underlay reachability from PE1's loopback to PE2's loopback.
```

**P1**

```bash
show isis neighbors # Verifies P1 has formed IS-IS adjacencies with both PE1 and PE2.
show ip route isis # Verifies P1 is learning the PE loopback and provider-link routes through IS-IS.
ping 1.1.1.1 source 2.2.2.2 # Verifies provider underlay reachability from P1's loopback to PE1's loopback.
ping 3.3.3.3 source 2.2.2.2 # Verifies provider underlay reachability from P1's loopback to PE2's loopback.
```

**PE2**

```bash
show isis neighbors # Verifies PE2 has formed an IS-IS adjacency with P1.
show ip route isis # Verifies PE2 is learning provider routes through IS-IS, including reachability toward PE1.
ping 1.1.1.1 source 3.3.3.3 # Verifies end-to-end provider underlay reachability from PE2's loopback to PE1's loopback.
```

### LDP Verification

**PE1**

```bash
show mpls ldp neighbor # Verifies PE1 has formed an LDP neighbor relationship with P1.
show mpls ldp bindings # Verifies PE1 has local and remote label bindings for provider FECs learned through LDP.
```

**P1**

```bash
show mpls ldp neighbor # Verifies P1 has formed LDP neighbor relationships with both PE1 and PE2.
show mpls ldp bindings # Verifies P1 has local and remote label bindings for provider FECs learned through LDP.
```

**PE2**

```bash
show mpls ldp neighbor # Verifies PE2 has formed an LDP neighbor relationship with P1.
show mpls ldp bindings # Verifies PE2 has local and remote label bindings for provider FECs learned through LDP.
```

### MPLS Forwarding Verification

**PE1**

```bash
show mpls forwarding-table # Verifies PE1 has active MPLS forwarding entries for provider FECs.
```

**P1**

```bash
show mpls forwarding-table # Verifies P1 has active MPLS forwarding entries and label switching toward both provider edges.
```

**PE2**

```bash
show mpls forwarding-table # Verifies PE2 has active MPLS forwarding entries for provider FECs.
```

### MP-BGP VPNv4 Verification

**PE1**

```bash
show ip bgp vpnv4 all summary # Verifies PE1's VPNv4 iBGP session with PE2 is established and receiving VPNv4 prefixes.
show ip bgp vpnv4 all # Verifies PE1's VPNv4 table contains the expected local and remote customer VPN routes, including RD-qualified prefixes and BGP next hops.
```

**PE2**

```bash
show ip bgp vpnv4 all summary # Verifies PE2's VPNv4 iBGP session with PE1 is established and receiving VPNv4 prefixes.
show ip bgp vpnv4 all # Verifies PE2's VPNv4 table contains the expected local and remote customer VPN routes, including RD-qualified prefixes and BGP next hops.
```

### VRF Route Verification

**PE1**

```bash
show ip route vrf CUSTOMER_A # Verifies PE1's CUSTOMER_A VRF contains the expected local Site A route and remote Site B route learned through the MPLS L3VPN.
```

**PE2**

```bash
show ip route vrf CUSTOMER_A # Verifies PE2's CUSTOMER_A VRF contains the expected local Site B route and remote Site A route learned through the MPLS L3VPN.
```

---

## End-to-End Customer Verification

**TSA1 → TSA2**

```bash
ping 10.20.20.10 # Verifies end-to-end CUSTOMER_A connectivity from Site A to Site B across the MPLS L3VPN.
```

**TSA2 → TSA1**

```bash
ping 10.10.10.10 # Verifies end-to-end CUSTOMER_A connectivity from Site B back to Site A across the MPLS L3VPN.
```

---
## Packet Captures

- IS-IS control plane — confirm adjacency maintenance on a provider-core link.
- LDP control plane — confirm LDP discovery/session traffic and label mapping exchanges.
- MP-BGP VPNv4 control plane — confirm TCP/179 between PE loopbacks and VPNv4 route exchange.
- MPLS L3VPN data plane — ping TSA1 to TSA2 and inspect the outer transport label, inner VPN label, and label changes across P1.

## Troubleshooting

> **Note:** These are quick-reference checks for this lab and are not intended to be an exhaustive troubleshooting guide. After correcting an issue, re-run the relevant verification steps.

```bash
# Customer LAN or CE reachability issue.
show ip interface brief # Check that expected routed interfaces and SVIs are up/up with the correct IP addresses.
show ip route # Confirm the required connected, static, and default routes are present.
ping <next-hop-ip> # Test reachability to the directly connected next hop before troubleshooting farther into the path.

# IS-IS provider underlay issue.
show isis neighbors # Confirm PE/P routers have formed the expected IS-IS adjacencies.
show ip route isis # Confirm provider loopbacks and transit prefixes are being learned through IS-IS.
ping <remote-loopback> source <local-loopback> # Confirm provider loopback-to-loopback reachability across the underlay.

# LDP or MPLS transport issue.
show mpls ldp neighbor # Confirm expected LDP neighbor relationships are established.
show mpls ldp bindings # Confirm label bindings exist for provider FECs.
show mpls forwarding-table # Confirm active MPLS forwarding entries and expected next hops are installed.

# MP-BGP VPNv4 issue.
show ip bgp vpnv4 all summary # Confirm the PE-to-PE VPNv4 iBGP session is established.
show ip bgp vpnv4 all # Confirm expected local and remote VPNv4 customer routes are present.

# VRF or customer VPN routing issue.
show ip route vrf CUSTOMER_A # Confirm expected local and remote CUSTOMER_A routes are installed in the VRF.
show running-config | section ip vrf CUSTOMER_A # Check RD and Route Target configuration if expected VPN routes are missing.

# End-to-end customer connectivity issue.
ping <remote-customer-ip> # Confirm whether the complete MPLS L3VPN path is working after lower-layer checks pass.
```
---

## Artifacts

| Type           | Location                                                                         |
| -------------- | -------------------------------------------------------------------------------- |
| Configurations | [`configs/`](configs/)                                                           |
| Diagram        | [`topology/<diagram-file-name>`](topology/<diagram-file-name>)                   |
| Topology File  | [`topology/topology.yaml`](topology/topology.yaml)                               |
| Verification   | [`verification/verification_commands.md`](verification/verification_commands.md) |

---

## Document Metadata

| Field        | Value         |
| ------------ | ------------- |
| Lab Version  | 1.0           |
| Last Updated | <YYYY-MM-DD>  |
| Author       | Aaron Kindelt |
