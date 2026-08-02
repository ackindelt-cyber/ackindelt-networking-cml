# Lab Guide — Edge Firewall HA

## Overview

This lab demonstrates the configuration and validation of a Cisco ASAv Active/Standby failover pair in an enterprise-style edge topology. The firewall pair connects to an upstream edge router through a shared outside transit VLAN, connects to redundant distribution switches through a shared inside transit VLAN, and uses a dedicated failover and state link for HA communication, configuration synchronization, and connection-state synchronization.

The design combines ASAv high availability, HSRP, Spanning Tree, static routing, ICMP inspection, and dynamic PAT to provide resilient connectivity between an internal test VLAN and a simulated external destination. Validation includes normal operation, manual firewall switchover, monitored-interface failures, firewall node failure, distribution-switch failure, and restoration of the preferred operating state.

Lab Status: Planned

End-to-End Verification: Not Tested

---

## Objectives

* [ ] Configure the shared outside transit VLAN between R1 and both ASAv outside interfaces.
* [ ] Configure an ASAv Active/Standby failover pair with role-based inside and outside addressing.
* [ ] Configure a dedicated failover and state link for HA communication and state synchronization.
* [ ] Configure VLAN 99 as the shared firewall transit network between the ASAv pair and D1/D2.
* [ ] Configure HSRP on VLANs 10 and 99 and align the VLAN 10 Spanning Tree root with the preferred HSRP active switch.
* [ ] Configure static routing, ICMP inspection, and dynamic PAT for the internal test network.
* [ ] Verify Layer 2, Layer 3, NAT, HA synchronization, and end-to-end connectivity.
* [ ] Validate manual switchover and automatic recovery from firewall-link, firewall-node, and distribution-switch failures.


---

## Topology

The topology consists of a simulated ISP router, an enterprise edge router, a shared outside Layer 2 switch, two Cisco ASAv firewalls operating as an Active/Standby pair, two distribution switches, and a dual-connected access switch.

R1 connects to both firewall outside interfaces through OS1 and VLAN 100. FW1 and FW2 use a dedicated point-to-point link for failover communication and state synchronization. Each firewall connects to a different distribution switch through VLAN 99, which provides the shared inside transit network.

D1 and D2 provide HSRP default gateways for VLANs 10 and 99. They are interconnected by a trunk carrying both VLANs and each provides an independent VLAN 10 uplink to A1. Spanning Tree selects the preferred forwarding path while retaining the second uplink for redundancy.

A1 serves as the internal test endpoint through its VLAN 10 SVI. Its default gateway is the VLAN 10 HSRP virtual IP address. Traffic from A1 crosses the active distribution path, the active firewall, R1, and the simulated ISP before reaching the external test loopback.

![Topology Diagram](topology/<diagram-file-name>)

---

## Link Tables

### Physical Links

| Local Device | Local Interface | Peer Device | Peer Interface | Description                              |
|--------------|-----------------|-------------|----------------|------------------------------------------|
| ISP          | Gi0/0           | R1          | Gi0/1          | Simulated ISP connection to R1           |
| R1           | Gi0/0           | OS1         | Gi0/0          | R1 connection to shared outside segment  |
| OS1          | Gi0/1           | FW1         | Gi0/0          | FW1 outside connection                   |
| OS1          | Gi0/2           | FW2         | Gi0/0          | FW2 outside connection                   |
| FW1          | Gi0/1           | FW2         | Gi0/1          | Dedicated failover and state link        |
| FW1          | Gi0/2           | D1          | Gi0/0          | FW1 inside connection to D1              |
| FW2          | Gi0/2           | D2          | Gi0/0          | FW2 inside connection to D2              |
| D1           | Gi0/1           | D2          | Gi0/1          | Distribution inter-switch trunk          |
| D1           | Gi0/2           | A1          | Gi0/0          | A1 uplink to D1                          |
| D2           | Gi0/2           | A1          | Gi0/1          | A1 uplink to D2                          |

### Logical Links

| Logical Interface / Relationship | Participating Devices | Member Interfaces / Networks             | Purpose                                                  |
|----------------------------------|-----------------------|------------------------------------------|----------------------------------------------------------|
| Outside HA segment               | R1, OS1, FW1, FW2     | VLAN 100 and both firewall outside ports | Provides the shared outside subnet required for HA       |
| Failover and state link          | FW1, FW2              | FW1 Gi0/1 and FW2 Gi0/1                  | Carries HA control and connection-state information      |
| Firewall transit VLAN            | FW1, FW2, D1, D2      | VLAN 99 and both firewall inside ports   | Provides the shared inside subnet required for HA        |
| VLAN 99 HSRP relationship        | D1, D2                | VLAN 99 SVIs                             | Provides the firewall pair a redundant internal next hop |
| VLAN 10 HSRP relationship        | D1, D2                | VLAN 10 SVIs                             | Provides A1 a redundant default gateway                  |

---

## Addressing and Design

### IP Addressing

| Device / Role          | Interface / Role       | IP Address / Prefix | Purpose                                           |
|------------------------|------------------------|---------------------|---------------------------------------------------|
| ISP                    | Gi0/0                  | 198.51.100.1/30     | ISP side of the R1 point-to-point link            |
| ISP                    | Loopback0              | 192.0.2.100/32      | Simulated external test destination               |
| R1                     | Gi0/1                  | 198.51.100.2/30     | R1 side of the ISP point-to-point link            |
| R1                     | Gi0/0                  | 203.0.113.1/29      | Gateway for the shared outside subnet             |
| Firewall active role   | Gi0/0 / `outside`      | 203.0.113.2/29      | Outside address used by the Active firewall       |
| Firewall standby role  | Gi0/0 / `outside`      | 203.0.113.3/29      | Outside address used by the Standby firewall      |
| FW1 primary unit       | Gi0/1 / failover       | 10.255.255.1/30     | Primary-unit failover and state-link address      |
| FW2 secondary unit     | Gi0/1 / failover       | 10.255.255.2/30     | Secondary-unit failover and state-link address    |
| D1/D2                  | VLAN 99 HSRP VIP       | 10.255.0.1/29       | Redundant internal next hop for the firewall pair |
| D1                     | VLAN 99 SVI            | 10.255.0.2/29       | D1 firewall-transit address                       |
| D2                     | VLAN 99 SVI            | 10.255.0.3/29       | D2 firewall-transit address                       |
| Firewall active role   | Gi0/2 / `inside`       | 10.255.0.4/29       | Inside address used by the Active firewall        |
| Firewall standby role  | Gi0/2 / `inside`       | 10.255.0.5/29       | Inside address used by the Standby firewall       |
| D1/D2                  | VLAN 10 HSRP VIP       | 10.10.10.1/24       | Default gateway for the internal test VLAN        |
| D1                     | VLAN 10 SVI            | 10.10.10.2/24       | D1 internal test VLAN address                     |
| D2                     | VLAN 10 SVI            | 10.10.10.3/24       | D2 internal test VLAN address                     |
| A1                     | VLAN 10 SVI            | 10.10.10.10/24      | Internal source used for traffic validation       |

### VLANs

| VLAN ID | Name             | Segment Participants | Purpose                                                           |
|---------|------------------|----------------------|-------------------------------------------------------------------|
| 10      | INTERNAL_TEST    | D1, D2, A1           | Internal network used to validate HSRP, PAT, and HA               |
| 99      | FIREWALL_TRANSIT | FW1, FW2, D1, D2     | Shared inside segment between the firewall and distribution pairs |
| 100     | OUTSIDE_TRANSIT  | R1, OS1, FW1, FW2    | Shared outside segment between R1 and the firewall pair           |

### Layer 2 Design

| Link                        | Switch-Side Mode | VLANs | Purpose                                              |
|-----------------------------|------------------|-------|------------------------------------------------------|
| R1 Gi0/0 to OS1 Gi0/0       | Access           | 100   | Connects R1 to the shared outside HA segment         |
| OS1 Gi0/1 to FW1 Gi0/0      | Access           | 100   | Connects FW1 outside to the shared outside segment   |
| OS1 Gi0/2 to FW2 Gi0/0      | Access           | 100   | Connects FW2 outside to the shared outside segment   |
| FW1 Gi0/2 to D1 Gi0/0       | Access           | 99    | Connects FW1 inside to the shared firewall transit   |
| FW2 Gi0/2 to D2 Gi0/0       | Access           | 99    | Connects FW2 inside to the shared firewall transit   |
| D1 Gi0/1 to D2 Gi0/1        | Trunk            | 10,99 | Carries the internal and firewall transit VLANs      |
| D1 Gi0/2 to A1 Gi0/0        | Trunk            | 10    | Provides A1 a VLAN 10 uplink through D1              |
| D2 Gi0/2 to A1 Gi0/1        | Trunk            | 10    | Provides A1 a VLAN 10 uplink through D2              |

### HSRP Parameters

| VLAN | HSRP Group | Virtual IP  | D1 Priority | D2 Priority | Preferred Active | Preemption |
|------|------------|-------------|-------------|-------------|------------------|------------|
| 10   | 10         | 10.10.10.1  | 110         | 100         | D1               | D1 and D2  |
| 99   | 99         | 10.255.0.1  | 110         | 100         | D1               | D1 and D2  |

### Routing Parameters

| Device / Role | Destination     | Next Hop       | Purpose                                                    |
|---------------|-----------------|----------------|------------------------------------------------------------|
| ISP           | 203.0.113.0/29  | 198.51.100.2   | Routes the shared firewall outside subnet through R1       |
| R1            | 0.0.0.0/0       | 198.51.100.1   | Sends unknown destinations toward the simulated ISP        |
| Firewall pair | 0.0.0.0/0       | 203.0.113.1    | Sends outbound traffic toward R1                           |
| Firewall pair | 10.10.10.0/24   | 10.255.0.1     | Routes VLAN 10 through the VLAN 99 HSRP virtual IP         |
| D1            | 0.0.0.0/0       | 10.255.0.4     | Sends upstream traffic to the active-role firewall address |
| D2            | 0.0.0.0/0       | 10.255.0.4     | Sends upstream traffic to the active-role firewall address |
| A1            | Default gateway | 10.10.10.1     | Sends off-subnet traffic through the VLAN 10 HSRP VIP      |

### Firewall HA Parameters

| Parameter                    | Value                      | Purpose                                                      |
|------------------------------|----------------------------|--------------------------------------------------------------|
| Failover mode                | Active/Standby             | Provides stateful firewall appliance redundancy              |
| Primary unit                 | FW1                        | Identifies FW1 as the primary physical unit                  |
| Secondary unit               | FW2                        | Identifies FW2 as the secondary physical unit                |
| Baseline Active unit         | FW1                        | Defines the intended normal operating state                  |
| Outside interface            | Gi0/0 / `outside`          | Connects both firewalls to the shared VLAN 100 segment       |
| Inside interface             | Gi0/2 / `inside`           | Connects both firewalls to the shared VLAN 99 segment        |
| Failover and state interface | Gi0/1                      | Carries HA control and connection-state synchronization      |
| FW1 failover-link address    | 10.255.255.1/30            | Unit-specific failover address assigned to FW1               |
| FW2 failover-link address    | 10.255.255.2/30            | Unit-specific failover address assigned to FW2               |
| Outside Active-role address  | 203.0.113.2/29             | Outside address used by the Active firewall                  |
| Outside Standby-role address | 203.0.113.3/29             | Outside address used by the Standby firewall                 |
| Inside Active-role address   | 10.255.0.4/29              | Inside address used by the Active firewall                   |
| Inside Standby-role address  | 10.255.0.5/29              | Inside address used by the Standby firewall                  |
| Internal route next hop      | 10.255.0.1                 | VLAN 99 HSRP VIP used to reach VLAN 10                       |
| PAT source network           | 10.10.10.0/24              | Defines the internal network translated by the firewall pair |
| PAT translated address       | Active `outside` interface | Uses the Active firewall’s outside interface address         |
| External test destination    | 192.0.2.100                | Provides a stable end-to-end connectivity target             |

---

## Configuration Steps

**Note:** The CLI examples below are annotated for readability. Clean device configurations are available in [`configs/`](configs/).

**Design note:** Configure the shared firewall interfaces, routing, NAT, and inspection policy on FW1. After FW2 is bootstrapped into the failover pair, the shared configuration synchronizes automatically. Primary/secondary unit identity and failover-link addressing remain unit-specific, while the inside and outside addresses follow the Active and Standby roles.

**Save note:** After FW2 reaches `Standby Ready`, save the synchronized configuration from the Active firewall.

---

**ISP**

```bash
# Start basic device prep block
enable # Enters privileged EXEC mode.
configure terminal # Enters global configuration mode.
hostname ISP # Sets the device hostname.
no ip domain lookup # Prevents mistyped commands from triggering DNS lookup.

# Start ISP-to-R1 point-to-point link block
interface gigabitethernet0/0 # Selects the interface connected to R1 Gi0/1.
description R1 Gi0/1 - Customer Edge Link # Documents the connected device and link purpose.
ip address 198.51.100.1 255.255.255.252 # Assigns the ISP side of the point-to-point subnet.
no shutdown # Enables the interface.

# Start external test destination block
interface loopback0 # Creates the simulated external test interface.
description INTERNET_TEST Destination # Documents the loopback purpose.
ip address 192.0.2.100 255.255.255.255 # Assigns the simulated Internet test address.

# Start return route block
ip route 203.0.113.0 255.255.255.248 198.51.100.2 # Routes the shared firewall outside subnet through R1.

end # Returns to privileged EXEC mode.
write memory # Saves the running configuration.
```

---

**R1**

```bash
# Start basic device prep block
enable # Enters privileged EXEC mode.
configure terminal # Enters global configuration mode.
hostname R1 # Sets the device hostname.
no ip domain lookup # Prevents mistyped commands from triggering DNS lookup.

# Start R1-to-ISP point-to-point link block
interface gigabitethernet0/1 # Selects the interface connected to ISP Gi0/0.
description ISP Gi0/0 - Provider Uplink # Documents the connected device and link purpose.
ip address 198.51.100.2 255.255.255.252 # Assigns the R1 side of the ISP point-to-point subnet.
no shutdown # Enables the interface.

# Start shared outside segment block
interface gigabitethernet0/0 # Selects the interface connected to OS1 Gi0/0.
description OS1 Gi0/0 - Firewall Outside Segment # Documents the connected device and link purpose.
ip address 203.0.113.1 255.255.255.248 # Assigns R1 as the gateway for the shared firewall outside subnet.
no shutdown # Enables the interface.

# Start default route block
ip route 0.0.0.0 0.0.0.0 198.51.100.1 # Sends destinations not found in the routing table toward the simulated ISP.

end # Returns to privileged EXEC mode.
write memory # Saves the running configuration.
```

---

**OS1**

```bash
# Start basic device prep block
enable # Enters privileged EXEC mode.
configure terminal # Enters global configuration mode.
hostname OS1 # Sets the device hostname.
no ip domain lookup # Prevents mistyped commands from triggering DNS lookup.

# Start outside transit VLAN block
vlan 100 # Creates the shared firewall outside VLAN.
name OUTSIDE_TRANSIT # Assigns a descriptive name to VLAN 100.

# Start R1-facing access port block
interface gigabitethernet0/0 # Selects the interface connected to R1 Gi0/0.
description R1 Gi0/0 - Outside Transit # Documents the connected device and link purpose.
switchport mode access # Configures the interface as a Layer 2 access port.
switchport access vlan 100 # Places the interface in the shared outside VLAN.
no shutdown # Enables the interface.

# Start FW1-facing access port block
interface gigabitethernet0/1 # Selects the interface connected to FW1 Gi0/0.
description FW1 Gi0/0 - Outside Transit # Documents the connected device and link purpose.
switchport mode access # Configures the interface as a Layer 2 access port.
switchport access vlan 100 # Places the interface in the shared outside VLAN.
no shutdown # Enables the interface.

# Start FW2-facing access port block
interface gigabitethernet0/2 # Selects the interface connected to FW2 Gi0/0.
description FW2 Gi0/0 - Outside Transit # Documents the connected device and link purpose.
switchport mode access # Configures the interface as a Layer 2 access port.
switchport access vlan 100 # Places the interface in the shared outside VLAN.
no shutdown # Enables the interface.

end # Returns to privileged EXEC mode.
write memory # Saves the running configuration.
```

---

**FW1**

```bash
# Start basic device prep block
enable # Enters privileged EXEC mode.
configure terminal # Enters global configuration mode.
hostname FW-HA # Sets the shared hostname used by both members of the failover pair.
prompt hostname priority state # Displays the shared hostname, unit identity, and current failover state.

# Start outside interface block
interface gigabitethernet0/0 # Selects the physical interface connected to OS1.
description OS1 - OUTSIDE_TRANSIT # Documents the shared outside Layer 2 segment.
nameif outside # Assigns the logical outside role to the interface.
security-level 0 # Assigns the lowest default interface trust level.
ip address 203.0.113.2 255.255.255.248 standby 203.0.113.3 # Assigns the active-role and standby-role outside addresses.
no shutdown # Enables the outside interface.

# Start inside interface block
interface gigabitethernet0/2 # Selects the physical interface connected to the distribution layer.
description DISTRIBUTION_PAIR - FIREWALL_TRANSIT # Documents the shared inside transit segment.
nameif inside # Assigns the logical inside role to the interface.
security-level 100 # Assigns the highest default interface trust level.
ip address 10.255.0.4 255.255.255.248 standby 10.255.0.5 # Assigns the active-role and standby-role inside addresses.
no shutdown # Enables the inside interface.

# Start default route block
route outside 0.0.0.0 0.0.0.0 203.0.113.1 # Sends destinations not found in the routing table toward R1.

# Start internal route block
route inside 10.10.10.0 255.255.255.0 10.255.0.1 # Routes VLAN 10 through the VLAN 99 HSRP virtual IP.

# Start dynamic PAT block
object network INTERNAL_TEST # Creates a network object for the internal test subnet.
subnet 10.10.10.0 255.255.255.0 # Defines VLAN 10 as the source network to translate.
nat (inside,outside) dynamic interface # Translates internal traffic to the active outside interface address.
exit # Returns to global configuration mode.

# Start ICMP inspection block
policy-map global_policy # Selects the global inspection policy.
class inspection_default # Selects the default inspection traffic class.
inspect icmp # Enables stateful inspection of ICMP requests and replies.
exit # Returns to policy-map configuration mode.
exit # Returns to global configuration mode.
service-policy global_policy global # Applies the inspection policy globally.

# Start Active/Standby failover block
failover lan unit primary # Identifies this physical ASAv as the primary unit.
failover lan interface FAILOVER gigabitethernet0/1 # Assigns Gi0/1 as the dedicated failover communication interface.
failover interface ip FAILOVER 10.255.255.1 255.255.255.252 standby 10.255.255.2 # Assigns primary-unit and secondary-unit failover-link addresses.

interface gigabitethernet0/1 # Selects the dedicated failover interface.
description DEDICATED_FAILOVER_STATE_LINK # Documents the link without using unit-specific peer information.
no shutdown # Enables the failover interface.

failover link FAILOVER gigabitethernet0/1 # Uses the failover interface for stateful connection synchronization.
failover # Enables Active/Standby failover.

end # Returns to privileged EXEC mode.
write memory # Saves the running configuration after failover sychronization
```

---

**FW2**

```bash
# Start basic device prep block
enable # Enters privileged EXEC mode.
configure terminal # Enters global configuration mode.

# Start Active/Standby failover bootstrap block
failover lan unit secondary # Identifies this physical ASAv as the secondary unit.
failover lan interface FAILOVER gigabitethernet0/1 # Assigns Gi0/1 as the dedicated failover communication interface.
failover interface ip FAILOVER 10.255.255.1 255.255.255.252 standby 10.255.255.2 # Defines the same failover-link addresses configured on FW1.

interface gigabitethernet0/1 # Selects the dedicated failover interface.
description DEDICATED_FAILOVER_STATE_LINK # Documents the interface purpose using a pair-neutral description.
no shutdown # Enables the failover interface.

failover link FAILOVER gigabitethernet0/1 # Uses Gi0/1 for state synchronization as well as failover control.
failover # Enables failover and begins communication and configuration synchronization with FW1.

end # Returns to privileged EXEC mode. Run write memory on FW1 after failover synchronization
```

---

**D1**

```bash
# Start basic device prep block
enable # Enters privileged EXEC mode.
configure terminal # Enters global configuration mode.
hostname D1 # Sets the device hostname.
no ip domain lookup # Prevents mistyped commands from triggering DNS lookup.
ip routing # Enables Layer 3 routing between SVIs and toward the firewall.

# Start VLAN creation block
vlan 10 # Creates the internal test VLAN.
name INTERNAL_TEST # Assigns a descriptive name to VLAN 10.
vlan 99 # Creates the firewall transit VLAN.
name FIREWALL_TRANSIT # Assigns a descriptive name to VLAN 99.

# Start VLAN 10 spanning tree block
spanning-tree vlan 10 priority 0 # Makes D1 the preferred STP root for VLAN 10.

# Start FW1-facing access port block
interface gigabitethernet0/0 # Selects the interface connected to FW1 Gi0/2.
description FW1 Gi0/2 - FIREWALL_TRANSIT # Documents the connected device and link purpose.
switchport mode access # Configures the interface as a Layer 2 access port.
switchport access vlan 99 # Places the firewall-facing interface in VLAN 99.
no shutdown # Enables the interface.

# Start D2-facing trunk block
interface gigabitethernet0/1 # Selects the inter-switch link connected to D2 Gi0/1.
description D2 Gi0/1 - DISTRIBUTION_TRUNK # Documents the connected device and link purpose.
switchport mode trunk # Configures the interface as a static trunk.
switchport trunk allowed vlan 10,99 # Allows the internal and firewall transit VLANs across the trunk.
no shutdown # Enables the interface.

# Start A1-facing trunk block
interface gigabitethernet0/2 # Selects the interface connected to A1 Gi0/0.
description A1 Gi0/0 - ACCESS_UPLINK # Documents the connected device and link purpose.
switchport mode trunk # Configures the interface as a static trunk.
switchport trunk allowed vlan 10 # Allows only the internal test VLAN toward A1.
no shutdown # Enables the interface.

# Start VLAN 10 SVI and HSRP block
interface vlan 10 # Selects the Layer 3 interface for the internal test VLAN.
description INTERNAL_TEST_DEFAULT_GATEWAY # Documents the SVI purpose.
ip address 10.10.10.2 255.255.255.0 # Assigns D1's physical Layer 3 address in VLAN 10.
standby 10 ip 10.10.10.1 # Configures the shared HSRP default-gateway address.
standby 10 priority 110 # Gives D1 a higher priority than D2.
standby 10 preempt # Allows D1 to reclaim the active HSRP role after recovery.
no shutdown # Enables the SVI.

# Start VLAN 99 SVI and HSRP block
interface vlan 99 # Selects the Layer 3 interface for the firewall transit VLAN.
description FIREWALL_TRANSIT_NEXT_HOP # Documents the SVI purpose.
ip address 10.255.0.2 255.255.255.248 # Assigns D1's physical Layer 3 address in VLAN 99.
standby 99 ip 10.255.0.1 # Configures the shared HSRP next hop used by the firewall pair.
standby 99 priority 110 # Gives D1 a higher priority than D2.
standby 99 preempt # Allows D1 to reclaim the active HSRP role after recovery.
no shutdown # Enables the SVI.

# Start default route block
ip route 0.0.0.0 0.0.0.0 10.255.0.4 # Sends destinations not found in the routing table to the active firewall address.

end # Returns to privileged EXEC mode.
write memory # Saves the running configuration.
```

---

**D2**

```bash
# Start basic device prep block
enable # Enters privileged EXEC mode.
configure terminal # Enters global configuration mode.
hostname D2 # Sets the device hostname.
no ip domain lookup # Prevents mistyped commands from triggering DNS lookup.
ip routing # Enables Layer 3 routing between SVIs and toward the firewall.

# Start VLAN creation block
vlan 10 # Creates the internal test VLAN.
name INTERNAL_TEST # Assigns a descriptive name to VLAN 10.
vlan 99 # Creates the firewall transit VLAN.
name FIREWALL_TRANSIT # Assigns a descriptive name to VLAN 99.

# Start VLAN 10 spanning tree block
spanning-tree vlan 10 priority 4096 # Makes D2 the secondary STP root for VLAN 10.

# Start FW2-facing access port block
interface gigabitethernet0/0 # Selects the interface connected to FW2 Gi0/2.
description FW2 Gi0/2 - FIREWALL_TRANSIT # Documents the connected device and link purpose.
switchport mode access # Configures the interface as a Layer 2 access port.
switchport access vlan 99 # Places the firewall-facing interface in VLAN 99.
no shutdown # Enables the interface.

# Start D1-facing trunk block
interface gigabitethernet0/1 # Selects the inter-switch link connected to D1 Gi0/1.
description D1 Gi0/1 - DISTRIBUTION_TRUNK # Documents the connected device and link purpose.
switchport mode trunk # Configures the interface as a static trunk.
switchport trunk allowed vlan 10,99 # Allows the internal and firewall transit VLANs across the trunk.
no shutdown # Enables the interface.

# Start A1-facing trunk block
interface gigabitethernet0/2 # Selects the interface connected to A1 Gi0/1.
description A1 Gi0/1 - ACCESS_UPLINK # Documents the connected device and link purpose.
switchport mode trunk # Configures the interface as a static trunk.
switchport trunk allowed vlan 10 # Allows only the internal test VLAN toward A1.
no shutdown # Enables the interface.

# Start VLAN 10 SVI and HSRP block
interface vlan 10 # Selects the Layer 3 interface for the internal test VLAN.
description INTERNAL_TEST_DEFAULT_GATEWAY # Documents the SVI purpose.
ip address 10.10.10.3 255.255.255.0 # Assigns D2's physical Layer 3 address in VLAN 10.
standby 10 ip 10.10.10.1 # Configures the shared HSRP default-gateway address.
standby 10 priority 100 # Gives D2 a lower priority than D1.
standby 10 preempt # Allows D2 to assume the active role when no higher-priority router is available.
no shutdown # Enables the SVI.

# Start VLAN 99 SVI and HSRP block
interface vlan 99 # Selects the Layer 3 interface for the firewall transit VLAN.
description FIREWALL_TRANSIT_NEXT_HOP # Documents the SVI purpose.
ip address 10.255.0.3 255.255.255.248 # Assigns D2's physical Layer 3 address in VLAN 99.
standby 99 ip 10.255.0.1 # Configures the shared HSRP next hop used by the firewall pair.
standby 99 priority 100 # Gives D2 a lower priority than D1.
standby 99 preempt # Allows D2 to assume the active role when no higher-priority router is available.
no shutdown # Enables the SVI.

# Start default route block
ip route 0.0.0.0 0.0.0.0 10.255.0.4 # Sends destinations not found in the routing table to the active firewall address.

end # Returns to privileged EXEC mode.
write memory # Saves the running configuration.
```

---

**A1**

```bash
# Start basic device prep block
enable # Enters privileged EXEC mode.
configure terminal # Enters global configuration mode.
hostname A1 # Sets the device hostname.
no ip domain lookup # Prevents mistyped commands from triggering DNS lookup.

# Start VLAN creation block
vlan 10 # Creates the internal test VLAN.
name INTERNAL_TEST # Assigns a descriptive name to VLAN 10.

# Start D1-facing trunk block
interface gigabitethernet0/0 # Selects the uplink connected to D1 Gi0/2.
description D1 Gi0/2 - DISTRIBUTION_UPLINK # Documents the connected device and link purpose.
switchport mode trunk # Configures the interface as a static trunk.
switchport trunk allowed vlan 10 # Allows only the internal test VLAN across the uplink.
no shutdown # Enables the interface.

# Start D2-facing trunk block
interface gigabitethernet0/1 # Selects the uplink connected to D2 Gi0/2.
description D2 Gi0/2 - DISTRIBUTION_UPLINK # Documents the connected device and link purpose.
switchport mode trunk # Configures the interface as a static trunk.
switchport trunk allowed vlan 10 # Allows only the internal test VLAN across the uplink.
no shutdown # Enables the interface.

# Start VLAN 10 management and test SVI block
interface vlan 10 # Selects the Layer 3 interface for the internal test VLAN.
description INTERNAL_TEST_ENDPOINT # Documents the SVI as the lab traffic source.
ip address 10.10.10.10 255.255.255.0 # Assigns A1 its VLAN 10 address.
no shutdown # Enables the SVI.

# Start default gateway block
ip default-gateway 10.10.10.1 # Sends off-subnet traffic to the VLAN 10 HSRP virtual IP.

end # Returns to privileged EXEC mode.
write memory # Saves the running configuration.
```

## Verification

See [`verification/verification_commands.md`](verification/verification_commands.md) for recorded command output and detailed expected results.

---

**ISP**

```bash
# Basic device prep verification
show running-config | include ^hostname # Confirms the configured ISP hostname.
show running-config | include ^no ip domain lookup # Confirms DNS lookup is disabled.

# Interface verification
show ip interface brief # Confirms Gi0/0 and Loopback0 are up with the expected addresses.
show running-config interface gigabitethernet0/0 # Confirms the R1-facing interface configuration.
show running-config interface loopback0 # Confirms the external test loopback configuration.

# Routing verification
show ip route connected # Confirms the directly connected R1 link and loopback route.
show ip route static # Confirms the return route toward the firewall outside subnet.
show ip route 203.0.113.0 # Confirms 203.0.113.0/29 resolves through R1.

# Connectivity verification
ping 198.51.100.2 # Confirms direct connectivity to R1 Gi0/1.
ping 203.0.113.1 # Confirms routed connectivity to R1's shared outside interface.
ping 203.0.113.2 # Confirms connectivity to the active firewall outside address.
```      


---

**R1**

```bash
# Basic device prep verification
show running-config | include ^hostname # Confirms the configured R1 hostname.
show running-config | include ^no ip domain lookup # Confirms DNS lookup is disabled.

# Interface verification
show ip interface brief # Confirms Gi0/0 and Gi0/1 are up with the expected addresses.
show running-config interface gigabitethernet0/0 # Confirms the OS1-facing outside-transit configuration.
show running-config interface gigabitethernet0/1 # Confirms the ISP-facing point-to-point configuration.

# Routing verification
show ip route connected # Confirms the directly connected ISP and outside-transit networks.
show ip route static # Confirms the default route toward the ISP.
show ip route 192.0.2.100 # Confirms the external test destination resolves through the default route.
show ip route 203.0.113.0 # Confirms the firewall outside subnet is directly connected.

# Connectivity verification
ping 198.51.100.1 # Confirms direct connectivity to the ISP router.
ping 192.0.2.100 # Confirms routed connectivity to the simulated external destination.
ping 203.0.113.2 # Confirms Layer 3 reachability to the active firewall outside address.
show ip arp 203.0.113.2 # Confirms R1 has resolved the active firewall address to a MAC address.
```

---

**OS1**

```bash
# Basic device prep verification
show running-config | include ^hostname # Confirms the configured OS1 hostname.
show running-config | include ^no ip domain lookup # Confirms DNS lookup is disabled.

# VLAN verification
show vlan brief # Confirms VLAN 100 exists and Gi0/0 through Gi0/2 are assigned to it.

# Access-port verification
show interfaces gigabitethernet0/0 switchport # Confirms the R1-facing port is a static access port in VLAN 100.
show interfaces gigabitethernet0/1 switchport # Confirms the FW1-facing port is a static access port in VLAN 100.
show interfaces gigabitethernet0/2 switchport # Confirms the FW2-facing port is a static access port in VLAN 100.
show interfaces status # Confirms all three outside-transit links are physically connected.

# Layer 2 forwarding verification
show mac address-table vlan 100 # Confirms OS1 learns MAC addresses from R1 and the firewall pair.
show spanning-tree vlan 100 # Confirms the VLAN 100 ports are forwarding and no unexpected STP blocking exists.
```

---

**FW1**

```bash
# Basic device prep verification
show running-config hostname # Confirms the shared FW-HA hostname.
show running-config prompt # Confirms the prompt displays priority and active/standby state.

# Data interface verification
show interface ip brief # Confirms the outside and inside interfaces are enabled with the expected addresses.
show nameif # Confirms Gi0/0 is named outside and Gi0/2 is named inside.
show running-config interface gigabitethernet0/0 # Confirms the outside interface and active/standby addressing.
show running-config interface gigabitethernet0/2 # Confirms the inside interface and active/standby addressing.

# Routing verification
show route # Confirms the connected, default, and internal static routes.
show route 0.0.0.0 # Confirms the default route points outside through 203.0.113.1.
show route 10.10.10.0 # Confirms VLAN 10 is reachable inside through 10.255.0.1.

# NAT verification
show running-config object network INTERNAL_TEST # Confirms the 10.10.10.0/24 network object.
show nat detail # Confirms dynamic PAT from inside to the outside interface.
show xlate # Displays active translations after test traffic is generated.
show conn # Displays active firewall connections after test traffic is generated.

# ICMP inspection verification
show running-config policy-map # Confirms inspect icmp is present under global_policy.
show running-config service-policy # Confirms global_policy is applied globally.
show service-policy # Confirms the inspection policy is active and displays runtime counters.

# Failover configuration and health verification
show running-config failover # Confirms FW1's failover and state-link configuration.
show failover # Confirms unit identity, active/standby state, peer health, interfaces, and state-link status.
show failover state # Summarizes the current state of both failover units.
show monitor-interface # Confirms the inside and outside interfaces are monitored.
show interface gigabitethernet0/1 # Confirms the dedicated failover/state interface is operational.

# Local connectivity verification
ping 203.0.113.1 # Confirms connectivity to R1 through the outside interface.
ping 10.255.0.1 # Confirms connectivity to the VLAN 99 HSRP virtual IP.
ping 10.10.10.10 # Confirms routed connectivity to A1 through the distribution layer.
```

---

**FW2**

```bash
# Shared configuration synchronization verification
show running-config hostname # Confirms FW2 received the shared FW-HA hostname from FW1.
show running-config prompt # Confirms FW2 received the shared prompt configuration.
show running-config failover # Confirms FW2 retains its secondary identity and matching failover-link configuration.

# Failover role and peer-health verification
show failover # Confirms FW2 is Secondary, Standby Ready, and communicating with FW1.
show failover state # Confirms the current primary/secondary and active/standby states.
show failover interface # Confirms the dedicated failover/state link is operational.
show monitor-interface # Confirms the inside and outside data interfaces are monitored.

# Failover/state interface verification
show interface gigabitethernet0/1 # Confirms the dedicated failover/state interface is physically operational.

# Synchronized data interface verification
show interface ip brief # Confirms the synchronized outside and inside interface addressing and status.
show nameif # Confirms Gi0/0 is outside and Gi0/2 is inside.
show running-config interface gigabitethernet0/0 # Confirms the synchronized outside configuration.
show running-config interface gigabitethernet0/2 # Confirms the synchronized inside configuration.

# Synchronized routing and policy verification
show route # Confirms FW2 received the connected and static routes from FW1.
show nat detail # Confirms FW2 received the dynamic PAT rule.
show running-config policy-map # Confirms the synchronized ICMP inspection configuration.
show running-config service-policy # Confirms the global policy is applied.
```

---

**D1**

```bash
# Basic device prep verification
show running-config | include ^hostname # Confirms the configured D1 hostname.
show running-config | include ^no ip domain lookup # Confirms DNS lookup is disabled.
show running-config | include ^ip routing # Confirms Layer 3 routing is enabled.

# VLAN verification
show vlan brief # Confirms VLANs 10 and 99 exist and Gi0/0 is assigned to VLAN 99.

# Trunk verification
show interfaces trunk # Confirms Gi0/1 carries VLANs 10 and 99 and Gi0/2 carries VLAN 10.
show interfaces gigabitethernet0/1 switchport # Confirms the D2-facing interface is a static trunk.
show interfaces gigabitethernet0/2 switchport # Confirms the A1-facing interface is a static trunk.

# Firewall-facing access-port verification
show interfaces gigabitethernet0/0 switchport # Confirms the FW1-facing interface is an access port in VLAN 99.
show interfaces status # Confirms the physical links are connected.

# SVI verification
show ip interface brief | include Vlan # Confirms the VLAN 10 and VLAN 99 SVIs are up with the expected addresses.
show running-config interface vlan 10 # Confirms VLAN 10 addressing and HSRP configuration.
show running-config interface vlan 99 # Confirms VLAN 99 addressing and HSRP configuration.

# HSRP verification
show standby brief # Confirms D1 is the expected HSRP Active router for VLANs 10 and 99.
show standby vlan 10 # Displays detailed VLAN 10 HSRP state, timers, priority, and peer information.
show standby vlan 99 # Displays detailed VLAN 99 HSRP state, timers, priority, and peer information.

# Spanning Tree verification
show spanning-tree vlan 10 # Confirms D2's root and designated VLAN 10 ports are forwarding.

# Routing verification
show ip route connected # Confirms VLANs 10 and 99 appear as directly connected networks.
show ip route static # Confirms the default route toward the active firewall address.
show ip route 0.0.0.0 # Confirms the default route uses 10.255.0.4 as its next hop.

# Connectivity verification
ping 10.10.10.3 # Confirms VLAN 10 connectivity to D2.
ping 10.10.10.10 # Confirms VLAN 10 connectivity to A1.
ping 10.255.0.3 # Confirms VLAN 99 connectivity to D2.
ping 10.255.0.4 # Confirms connectivity to the active firewall inside address.
ping 203.0.113.1 source 10.10.10.2 # Confirms connectivity through the firewall using an address covered by PAT.
ping 192.0.2.100 source 10.10.10.2 # Confirms end-to-end connectivity using a source address covered by PAT.
```

---

**D2**

```bash
# Basic device prep verification
show running-config | include ^hostname # Confirms the configured D2 hostname.
show running-config | include ^no ip domain lookup # Confirms DNS lookup is disabled.
show running-config | include ^ip routing # Confirms Layer 3 routing is enabled.

# VLAN verification
show vlan brief # Confirms VLANs 10 and 99 exist and Gi0/0 is assigned to VLAN 99.

# Trunk verification
show interfaces trunk # Confirms Gi0/1 carries VLANs 10 and 99 and Gi0/2 carries VLAN 10.
show interfaces gigabitethernet0/1 switchport # Confirms the D1-facing interface is a static trunk.
show interfaces gigabitethernet0/2 switchport # Confirms the A1-facing interface is a static trunk.

# Firewall-facing access-port verification
show interfaces gigabitethernet0/0 switchport # Confirms the FW2-facing interface is an access port in VLAN 99.
show interfaces status # Confirms the physical links are connected.

# SVI verification
show ip interface brief | include Vlan # Confirms the VLAN 10 and VLAN 99 SVIs are up with the expected addresses.
show running-config interface vlan 10 # Confirms VLAN 10 addressing and HSRP configuration.
show running-config interface vlan 99 # Confirms VLAN 99 addressing and HSRP configuration.

# HSRP verification
show standby brief # Confirms D2 is the expected HSRP Standby router for VLANs 10 and 99.
show standby vlan 10 # Displays detailed VLAN 10 HSRP state, timers, priority, and active-peer information.
show standby vlan 99 # Displays detailed VLAN 99 HSRP state, timers, priority, and active-peer information.

# Spanning Tree verification
show spanning-tree vlan 10 # Confirms VLAN 10 port roles and the expected blocked and forwarding paths.

# Routing verification
show ip route connected # Confirms VLANs 10 and 99 appear as directly connected networks.
show ip route static # Confirms the default route toward the active firewall address.
show ip route 0.0.0.0 # Confirms the default route uses 10.255.0.4 as its next hop.

# Connectivity verification
ping 10.10.10.2 # Confirms VLAN 10 connectivity to D1.
ping 10.10.10.10 # Confirms VLAN 10 connectivity to A1.
ping 10.255.0.2 # Confirms VLAN 99 connectivity to D1.
ping 10.255.0.4 # Confirms connectivity to the active firewall inside address.
ping 192.0.2.100 source 10.10.10.3 # Confirms end-to-end connectivity using a source address covered by the PAT rule.
```

---

**A1**

```bash
# Basic device prep verification
show running-config | include ^hostname # Confirms the configured A1 hostname.
show running-config | include ^no ip domain lookup # Confirms DNS lookup is disabled.

# VLAN verification
show vlan brief # Confirms VLAN 10 exists.

# Trunk verification
show interfaces trunk # Confirms Gi0/0 and Gi0/1 carry VLAN 10.
show interfaces gigabitethernet0/0 switchport # Confirms the D1-facing interface is a static trunk allowing VLAN 10.
show interfaces gigabitethernet0/1 switchport # Confirms the D2-facing interface is a static trunk allowing VLAN 10.
show interfaces status # Confirms both distribution uplinks are physically connected.

# Spanning Tree verification
show spanning-tree vlan 10 # Confirms one redundant uplink is forwarding and the other is blocking or alternate as expected.

# SVI verification
show ip interface brief | include Vlan10 # Confirms the VLAN 10 SVI is up with address 10.10.10.10.
show running-config interface vlan 10 # Confirms the VLAN 10 SVI configuration.

# Default gateway verification
show running-config | include ^ip default-gateway # Confirms the default gateway is the HSRP VIP 10.10.10.1.

# Connectivity verification
ping 10.10.10.1 # Confirms connectivity to the VLAN 10 HSRP virtual gateway.
ping 10.10.10.2 # Confirms connectivity to D1's VLAN 10 SVI.
ping 10.10.10.3 # Confirms connectivity to D2's VLAN 10 SVI.
ping 10.255.0.4 # Confirms routed connectivity to the active firewall inside address.
ping 203.0.113.1 # Confirms connectivity through the firewall to R1.
ping 192.0.2.100 # Confirms end-to-end connectivity through HSRP, the active firewall, PAT, R1, and the ISP.
traceroute 192.0.2.100 # Displays the Layer 3 path toward the simulated external destination.
```

---

## Failover Testing

See [`verification/failover_testing.md`](verification/failover_testing.md) for recorded test output and detailed results.

Perform failover testing only after the normal verification section passes. Restore the topology to its expected operating state before beginning each test.

### Test 1: Manual Firewall Switchover

**Purpose:** Confirm FW2 can assume the Active role, maintain traffic flow, and return to the original firewall roles.

```bash
# Run on A1
ping 192.0.2.100 repeat 10000 # Monitors end-to-end connectivity during the firewall switchover.

# Run on FW1 while FW1 is Active
no failover active # Causes FW1 to relinquish the Active role to FW2.
show failover # Confirms FW1 is Primary/Standby and FW2 is Secondary/Active.

# Run on FW1 while FW1 is Standby
failover active # Forces FW1 to resume the Active role and restores the original firewall roles.
show failover # Confirms FW1 is Primary/Active and FW2 is Secondary/Standby Ready.
```

### Test 2: FW1 Inside-Link Failure

**Purpose:** Confirm failure of FW1’s monitored inside path automatically moves the Active role to FW2.

```bash
# Run on A1
ping 192.0.2.100 repeat 10000 # Monitors end-to-end connectivity during automatic failover.

# Run on D1
configure terminal # Enters global configuration mode.
interface gigabitethernet0/0 # Selects the link connected to FW1 Gi0/2.
shutdown # Simulates failure of FW1's inside connection.
end # Returns to privileged EXEC mode.

# Run on either firewall
show failover # Confirms FW2 became Active and identifies FW1's inside-interface failure.
show failover state # Confirms the automatic role transition and recorded failure reason.

# Run on D1
configure terminal # Enters global configuration mode.
interface gigabitethernet0/0 # Selects the disabled FW1-facing port.
no shutdown # Restores FW1's inside connection.
end # Returns to privileged EXEC mode.

# Run on FW1 after it returns to Standby Ready
show failover # Confirms FW1 recovered and is ready to resume service.
failover active # Restores FW1 as the Active firewall.
show failover # Confirms FW1 is Primary/Active and FW2 is Secondary/Standby Ready.
```

### Test 3: FW1 Outside-Link Failure

**Purpose:** Confirm failure of FW1’s monitored outside path automatically moves the Active role to FW2.

```bash
# Run on A1
ping 192.0.2.100 repeat 10000 # Monitors end-to-end connectivity throughout failure and recovery.

# Run on OS1
configure terminal # Enters global configuration mode.
interface gigabitethernet0/1 # Selects the access port connected to FW1 Gi0/0.
shutdown # Simulates failure of FW1's outside connection.
end # Returns to privileged EXEC mode.

# Run on either firewall
show failover # Confirms FW2 became Active and reports the failed FW1 outside interface.
show failover state # Confirms the automatic role transition.

# Run on OS1
configure terminal # Enters global configuration mode.
interface gigabitethernet0/1 # Selects the disabled FW1-facing port.
no shutdown # Restores FW1's outside connection.
end # Returns to privileged EXEC mode.

# Run on FW1 after it reaches Standby Ready
show failover # Confirms FW1 recovered and is healthy.
failover active # Restores FW1 as the Active firewall.
show failover # Confirms FW1 is Primary/Active and FW2 is Secondary/Standby Ready.
```

### Test 4: FW1 Node Failure

**Purpose:** Confirm complete loss of the Active firewall automatically moves the Active role to FW2.

```bash
# Run on A1
ping 192.0.2.100 repeat 10000 # Monitors end-to-end connectivity throughout failure and recovery.

# In CML
Stop the FW1 node # Simulates complete power or hardware failure of the Active firewall.

# Run on FW2
show failover # Confirms FW2 became Secondary/Active and reports the Primary unit as failed.
show failover state # Confirms FW2 completed the automatic transition to Active.

# In CML
Start the FW1 node # Restores the Primary firewall.

# Run on FW1 after it finishes booting
show failover # Confirms FW1 rejoined the pair as Primary/Standby Ready.
failover active # Restores FW1 as the Active firewall.
show failover # Confirms FW1 is Primary/Active and FW2 is Secondary/Standby Ready.
```

### Test 5: D1 Node Failure

**Purpose:** Confirm complete loss of D1 moves traffic onto the surviving FW2–D2 path.

```bash
# Run on A1
ping 192.0.2.100 repeat 10000 # Monitors end-to-end connectivity throughout failure and recovery.

# In CML
Stop the D1 node # Simulates complete failure of the preferred distribution switch.

# Run on FW2
show failover # Confirms FW2 became Active after FW1 lost its inside connection.
show failover state # Confirms the firewall pair completed the automatic role transition.

# Run on D2
show standby brief # Confirms D2 became HSRP Active for VLANs 10 and 99.

# Run on A1
show spanning-tree vlan 10 # Confirms the D2-facing uplink is forwarding after convergence.

# In CML
Start the D1 node # Restores the preferred distribution switch.

# Run on D1 after it finishes booting
show standby brief # Confirms D1 reclaimed the HSRP Active role through preemption.
show spanning-tree vlan 10 # Confirms D1 resumed its expected Spanning Tree role.

# Run on FW1 after its inside link recovers
show failover # Confirms FW1 rejoined the pair as Primary/Standby Ready.
failover active # Restores FW1 as the Active firewall.
show failover # Confirms FW1 is Primary/Active and FW2 is Secondary/Standby Ready.

# Run on A1
show spanning-tree vlan 10 # Confirms the original forwarding and alternate uplink roles were restored.
```

### Final Restoration and Baseline Confirmation

**Purpose:** Confirm the original operating state has been restored after all failover tests.

```bash
# Run on FW1
show failover # Confirms FW1 is Primary/Active and FW2 is Secondary/Standby Ready.

# Run on D1
show standby brief # Confirms D1 is HSRP Active and D2 is HSRP Standby for VLANs 10 and 99.

# Run on A1
show spanning-tree vlan 10 # Confirms the original VLAN 10 forwarding and alternate uplink roles are restored.
ping 192.0.2.100 # Confirms end-to-end connectivity after all failure scenarios are complete.
```

## Artifacts

| Type                 | Location                                                                         |
|----------------------|----------------------------------------------------------------------------------|
| Configurations       | [`configs/`](configs/)                                                           |
| Diagram              | [`topology/<diagram-file-name>`](topology/<diagram-file-name>)                   |
| Topology File        | [`topology/topology.yaml`](topology/topology.yaml)                               |
| Verification Results | [`verification/verification_commands.md`](verification/verification_commands.md) |
| Failover Results     | [`verification/failover_testing.md`](verification/failover_testing.md)           |

---

## Document Metadata

| Field        | Value         |
|--------------|---------------|
| Lab Version  | 1.0           |
| Last Updated | 2026-08-02    |
| Author       | Aaron Kindelt |
