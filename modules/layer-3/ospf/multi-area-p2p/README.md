# Lab Guide — P2P Multi-Area OSPF

## Overview

This lab demonstrates multi-area OSPF using point-to-point router links.

OSPF allows routers to form neighbor adjacencies, exchange link-state information, and dynamically learn routes. In this lab, five routers are connected in a linear point-to-point topology using three OSPF areas: Area 1, Area 0, and Area 2.

R2 and R4 function as Area Border Routers. Area 0 acts as the OSPF backbone. Routes from Area 1 and Area 2 are propagated through the backbone using Type 3 summary LSAs.

This lab validates interface addressing, point-to-point OSPF neighbor formation, ABR behavior, inter-area route propagation, Type 3 LSA visibility, and routed reachability across the full topology.

**Lab Status:** Validated

**End-to-End Verification:** Successful

---

## Objectives

* [x] Configure router interface IP addressing.
* [x] Configure OSPF process ID 1 across five routers.
* [x] Configure unique OSPF router IDs.
* [x] Configure point-to-point OSPF links.
* [x] Implement OSPF Areas 1, 0, and 2.
* [x] Verify R2 and R4 as Area Border Routers.
* [x] Verify OSPF neighbor adjacencies.
* [x] Verify intra-area and inter-area routes.
* [x] Verify Type 3 LSA propagation between areas.
* [x] Verify routed reachability from Area 1 to Area 2.
* [x] Capture selected OSPF and ICMP traffic for packet-level validation.

---

## Topology

The topology uses five routers connected with point-to-point links. Area 1 sits on the left side of the topology, Area 0 forms the backbone, and Area 2 sits on the right side.

Topology artifacts are available in the `topology/` folder.
 
- [Open topology diagram](topology/diagram.svg)
- [Open CML topology export](topology/topology.yaml)

---

## OSPF Area Design

| Area   | Devices / Links                | Purpose           |
| ------ | ------------------------------ | ----------------- |
| Area 1 | R1-R2 link, R1 LAN, R2 LAN     | Non-backbone area |
| Area 0 | R2-R3 link, R3-R4 link, R3 LAN | Backbone area     |
| Area 2 | R4-R5 link, R4 LAN, R5 LAN     | Non-backbone area |

---

## Router Roles

| Router | Router ID | OSPF Role          | Areas Participating |
| ------ | --------- | ------------------ | ------------------- |
| R1     | 1.1.1.1   | Internal Router    | Area 1              |
| R2     | 2.2.2.2   | Area Border Router | Area 1, Area 0      |
| R3     | 3.3.3.3   | Backbone Router    | Area 0              |
| R4     | 4.4.4.4   | Area Border Router | Area 0, Area 2      |
| R5     | 5.5.5.5   | Internal Router    | Area 2              |

---

## Point-to-Point Links

| Local Device | Local Interface | Local IP     | Peer Device | Peer Interface | Peer IP      | OSPF Area |
| ------------ | --------------- | ------------ | ----------- | -------------- | ------------ | --------- |
| R1           | Gi0/0           | 10.0.0.1/30  | R2          | Gi0/0          | 10.0.0.2/30  | Area 1    |
| R2           | Gi0/1           | 10.0.0.5/30  | R3          | Gi0/0          | 10.0.0.6/30  | Area 0    |
| R3           | Gi0/1           | 10.0.0.9/30  | R4          | Gi0/0          | 10.0.0.10/30 | Area 0    |
| R4           | Gi0/1           | 10.0.0.13/30 | R5          | Gi0/0          | 10.0.0.14/30 | Area 2    |

---

## LAN Segments

| Device | Interface | IP Address      | Network         | OSPF Area | Passive Interface |
| ------ | --------- | --------------- | --------------- | --------- | ----------------- |
| R1     | Gi0/1     | 192.168.10.1/24 | 192.168.10.0/24 | Area 1    | Yes               |
| R2     | Gi0/2     | 192.168.20.1/24 | 192.168.20.0/24 | Area 1    | Yes               |
| R3     | Gi0/2     | 192.168.30.1/24 | 192.168.30.0/24 | Area 0    | Yes               |
| R4     | Gi0/2     | 192.168.40.1/24 | 192.168.40.0/24 | Area 2    | Yes               |
| R5     | Gi0/1     | 192.168.50.1/24 | 192.168.50.0/24 | Area 2    | Yes               |

---

## Client Addressing

| Client | Interface | IP Address       | Default Gateway | Area   |
| ------ | --------- | ---------------- | --------------- | ------ |
| C1     | eth0      | 192.168.10.10/24 | 192.168.10.1    | Area 1 |
| C2     | eth0      | 192.168.20.10/24 | 192.168.20.1    | Area 1 |
| C3     | eth0      | 192.168.30.10/24 | 192.168.30.1    | Area 0 |
| C4     | eth0      | 192.168.40.10/24 | 192.168.40.1    | Area 2 |
| C5     | eth0      | 192.168.50.10/24 | 192.168.50.1    | Area 2 |

---

## Configuration Steps

### Step 1 — R1 Configuration

**Note:** The CLI examples below are annotated for readability. Clean device configurations are available in [`configs/`](configs/).

**R1**

```bash
# R1 Configuration Block
enable # Enters privileged EXEC mode.
configure terminal # Enters global configuration mode.
hostname R1 # Sets hostname to R1.

interface gigabitEthernet0/0 # Targets point-to-point link to R2.
description P2P to R2 # Adds interface description.
ip address 10.0.0.1 255.255.255.252 # Assigns /30 point-to-point IP address.
ip ospf network point-to-point # Sets OSPF network type to point-to-point.
no shutdown # Enables the interface.

interface gigabitEthernet0/1 # Targets R1 LAN interface.
description R1 LAN # Adds interface description.
ip address 192.168.10.1 255.255.255.0 # Assigns R1 LAN gateway IP address.
no shutdown # Enables the interface.

router ospf 1 # Creates OSPF process 1.
router-id 1.1.1.1 # Sets unique OSPF router ID.
log-adjacency-changes # Logs OSPF neighbor state changes.
passive-interface gigabitEthernet0/1 # Suppresses OSPF hellos on the LAN interface.
network 10.0.0.0 0.0.0.3 area 1 # Advertises R1-R2 point-to-point link in Area 1.
network 192.168.10.0 0.0.0.255 area 1 # Advertises R1 LAN in Area 1.

end # Returns to privileged EXEC mode.
write memory # Saves the running configuration.
```

---

### Step 2 — R2 Configuration

**R2**

```bash
# R2 Configuration Block
enable # Enters privileged EXEC mode.
configure terminal # Enters global configuration mode.
hostname R2 # Sets hostname to R2.

interface gigabitEthernet0/0 # Targets point-to-point link to R1.
description P2P to R1 # Adds interface description.
ip address 10.0.0.2 255.255.255.252 # Assigns /30 point-to-point IP address.
ip ospf network point-to-point # Sets OSPF network type to point-to-point.
no shutdown # Enables the interface.

interface gigabitEthernet0/1 # Targets point-to-point link to R3.
description P2P to R3 # Adds interface description.
ip address 10.0.0.5 255.255.255.252 # Assigns /30 point-to-point IP address.
ip ospf network point-to-point # Sets OSPF network type to point-to-point.
no shutdown # Enables the interface.

interface gigabitEthernet0/2 # Targets R2 LAN interface.
description R2 LAN # Adds interface description.
ip address 192.168.20.1 255.255.255.0 # Assigns R2 LAN gateway IP address.
no shutdown # Enables the interface.

router ospf 1 # Creates OSPF process 1.
router-id 2.2.2.2 # Sets unique OSPF router ID.
log-adjacency-changes # Logs OSPF neighbor state changes.
passive-interface gigabitEthernet0/2 # Suppresses OSPF hellos on the LAN interface.
network 10.0.0.0 0.0.0.3 area 1 # Advertises R1-R2 point-to-point link in Area 1.
network 10.0.0.4 0.0.0.3 area 0 # Advertises R2-R3 point-to-point link in Area 0.
network 192.168.20.0 0.0.0.255 area 1 # Advertises R2 LAN in Area 1.

end # Returns to privileged EXEC mode.
write memory # Saves the running configuration.
```

---

### Step 3 — R3 Configuration

**R3**

```bash
# R3 Configuration Block
enable # Enters privileged EXEC mode.
configure terminal # Enters global configuration mode.
hostname R3 # Sets hostname to R3.

interface gigabitEthernet0/0 # Targets point-to-point link to R2.
description P2P to R2 # Adds interface description.
ip address 10.0.0.6 255.255.255.252 # Assigns /30 point-to-point IP address.
ip ospf network point-to-point # Sets OSPF network type to point-to-point.
no shutdown # Enables the interface.

interface gigabitEthernet0/1 # Targets point-to-point link to R4.
description P2P to R4 # Adds interface description.
ip address 10.0.0.9 255.255.255.252 # Assigns /30 point-to-point IP address.
ip ospf network point-to-point # Sets OSPF network type to point-to-point.
no shutdown # Enables the interface.

interface gigabitEthernet0/2 # Targets R3 LAN interface.
description R3 LAN # Adds interface description.
ip address 192.168.30.1 255.255.255.0 # Assigns R3 LAN gateway IP address.
no shutdown # Enables the interface.

router ospf 1 # Creates OSPF process 1.
router-id 3.3.3.3 # Sets unique OSPF router ID.
log-adjacency-changes # Logs OSPF neighbor state changes.
passive-interface gigabitEthernet0/2 # Suppresses OSPF hellos on the LAN interface.
network 10.0.0.4 0.0.0.3 area 0 # Advertises R2-R3 point-to-point link in Area 0.
network 10.0.0.8 0.0.0.3 area 0 # Advertises R3-R4 point-to-point link in Area 0.
network 192.168.30.0 0.0.0.255 area 0 # Advertises R3 LAN in Area 0.

end # Returns to privileged EXEC mode.
write memory # Saves the running configuration.
```

---

### Step 4 — R4 Configuration

**R4**

```bash
# R4 Configuration Block
enable # Enters privileged EXEC mode.
configure terminal # Enters global configuration mode.
hostname R4 # Sets hostname to R4.

interface gigabitEthernet0/0 # Targets point-to-point link to R3.
description P2P to R3 # Adds interface description.
ip address 10.0.0.10 255.255.255.252 # Assigns /30 point-to-point IP address.
ip ospf network point-to-point # Sets OSPF network type to point-to-point.
no shutdown # Enables the interface.

interface gigabitEthernet0/1 # Targets point-to-point link to R5.
description P2P to R5 # Adds interface description.
ip address 10.0.0.13 255.255.255.252 # Assigns /30 point-to-point IP address.
ip ospf network point-to-point # Sets OSPF network type to point-to-point.
no shutdown # Enables the interface.

interface gigabitEthernet0/2 # Targets R4 LAN interface.
description R4 LAN # Adds interface description.
ip address 192.168.40.1 255.255.255.0 # Assigns R4 LAN gateway IP address.
no shutdown # Enables the interface.

router ospf 1 # Creates OSPF process 1.
router-id 4.4.4.4 # Sets unique OSPF router ID.
log-adjacency-changes # Logs OSPF neighbor state changes.
passive-interface gigabitEthernet0/2 # Suppresses OSPF hellos on the LAN interface.
network 10.0.0.8 0.0.0.3 area 0 # Advertises R3-R4 point-to-point link in Area 0.
network 10.0.0.12 0.0.0.3 area 2 # Advertises R4-R5 point-to-point link in Area 2.
network 192.168.40.0 0.0.0.255 area 2 # Advertises R4 LAN in Area 2.

end # Returns to privileged EXEC mode.
write memory # Saves the running configuration.
```

---

### Step 5 — R5 Configuration

**R5**

```bash
# R5 Configuration Block
enable # Enters privileged EXEC mode.
configure terminal # Enters global configuration mode.
hostname R5 # Sets hostname to R5.

interface gigabitEthernet0/0 # Targets point-to-point link to R4.
description P2P to R4 # Adds interface description.
ip address 10.0.0.14 255.255.255.252 # Assigns /30 point-to-point IP address.
ip ospf network point-to-point # Sets OSPF network type to point-to-point.
no shutdown # Enables the interface.

interface gigabitEthernet0/1 # Targets R5 LAN interface.
description R5 LAN # Adds interface description.
ip address 192.168.50.1 255.255.255.0 # Assigns R5 LAN gateway IP address.
no shutdown # Enables the interface.

router ospf 1 # Creates OSPF process 1.
router-id 5.5.5.5 # Sets unique OSPF router ID.
log-adjacency-changes # Logs OSPF neighbor state changes.
passive-interface gigabitEthernet0/1 # Suppresses OSPF hellos on the LAN interface.
network 10.0.0.12 0.0.0.3 area 2 # Advertises R4-R5 point-to-point link in Area 2.
network 192.168.50.0 0.0.0.255 area 2 # Advertises R5 LAN in Area 2.

end # Returns to privileged EXEC mode.
write memory # Saves the running configuration.
```

---

## Verification

See [`verification/verification_commands.md`](verification/verification_commands.md) for recorded command output.

### Router Verification

```bash
# Run on all routers.
show ip interface brief # Confirm configured interfaces are up/up with expected IP addresses.
show ip ospf interface brief # Confirm OSPF-enabled interfaces, areas, and point-to-point state.
show ip ospf neighbor # Confirm directly connected OSPF neighbors are FULL.
show ip route ospf # Confirm OSPF-learned intra-area and inter-area routes.
show ip ospf database # Confirm router LSAs and Type 3 summary LSAs.
show ip ospf border-routers # Confirm known ABRs.
show ip protocols # Confirm router ID, areas, passive interfaces, and advertised networks.
```

### Routed Path Verification

```bash
# Run on R1.
ping 192.168.50.1 source 192.168.10.1 repeat 5 # Confirm reachability from Area 1 toward Area 2.
traceroute 192.168.50.1 source 192.168.10.1 # Confirm routed path through R2, R3, R4, and R5.
```

### Packet Captures

The following packet captures provide additional evidence for selected OSPF and ICMP flows:

| Capture                                                          | Description                                           |
| ---------------------------------------------------------------- | ----------------------------------------------------- |
| [`C1_to_C5_ping.pcap`](verification/captures/C1_to_C5_ping.pcap) | ICMP traffic from C1 to C5 across multiple OSPF areas |
| [`R1_to_R2_ospf.pcap`](verification/captures/R1_to_R2_ospf.pcap) | OSPF traffic on the R1-R2 Area 1 link                 |
| [`R2_to_R3_OSPF.pcap`](verification/captures/R2_to_R3_OSPF.pcap) | OSPF traffic on the R2-R3 Area 0 link                 |
| [`R3_to_R4_OSPF.pcap`](verification/captures/R3_to_R4_OSPF.pcap) | OSPF traffic on the R3-R4 Area 0 link                 |
| [`R4_to_R5_OSPF.pcap`](verification/captures/R4_to_R5_OSPF.pcap) | OSPF traffic on the R4-R5 Area 2 link                 |

---

## Troubleshooting

**Note:** These are quick-reference checks for this lab. They are not intended to be an exhaustive OSPF troubleshooting guide. After any change, re-run the verification steps to confirm the expected behavior.

```bash
# OSPF neighbors do not form.
show ip ospf neighbor # Confirm adjacency state.
show ip interface brief # Confirm both sides of the point-to-point link are up/up.
show ip ospf interface brief # Confirm the interface is participating in the expected area.
show ip ospf interface gigabitEthernet0/x # Confirm network type, hello/dead timers, and area.
show running-config interface gigabitEthernet0/x # Confirm IP address and OSPF network type.
show running-config | section router ospf # Confirm network statements and passive-interface configuration.

# OSPF neighbor is stuck in 2-WAY, EXSTART, or EXCHANGE.
show ip ospf interface gigabitEthernet0/x # Confirm point-to-point type and timers.
show interfaces gigabitEthernet0/x # Confirm MTU and physical interface health.
show ip ospf # Confirm router ID is unique.
show logging # Check for OSPF adjacency or interface messages.

# OSPF routes do not appear.
show ip route ospf # Confirm whether routes are missing or installed.
show ip ospf database # Confirm LSAs exist in the local database.
show ip protocols # Confirm networks are advertised into the correct areas.
show running-config | section router ospf # Confirm network statements match the intended areas.
show ip ospf border-routers # Confirm ABRs are visible where expected.

# Inter-area routes are missing.
show ip ospf database summary # Confirm Type 3 LSAs are present.
show ip ospf border-routers # Confirm ABRs are known.
show ip route ospf # Confirm O IA routes are installed.
show ip protocols # Confirm ABR routers participate in more than one area.

# End-to-end connectivity fails.
show ip route # Confirm route to the destination network.
show ip route ospf # Confirm OSPF installed the destination route.
traceroute <destination> source <source-interface-ip> # Confirm where the path stops.
ping <next-hop-ip> source <local-interface-ip> # Test hop-by-hop reachability.
show arp # Confirm local next-hop resolution.

# Client-to-client testing fails.
ip addr # On Linux clients, confirm client IP address.
ip route # On Linux clients, confirm default gateway.
ping <default-gateway> # Confirm client can reach its local router.
traceroute <remote-client-ip> # Confirm routed path behavior.
```

---

## Artifacts

| Type            | Location                                                                         |
| --------------- | -------------------------------------------------------------------------------- |
| Configurations  | [`configs/`](configs/)                                                           |
| Diagram         | [`topology/diagram.svg`](topology/diagram.svg)                                   |
| Topology File   | [`topology/topology.yaml`](topology/topology.yaml)                               |
| Verification    | [`verification/verification_commands.md`](verification/verification_commands.md) |
| Packet Captures | [`verification/captures/`](verification/captures/)                               |

---

## Document Metadata

| Field        | Value         |
| ------------ | ------------- |
| Lab Version  | 1.0           |
| Last Updated | 2026-06-28    |
| Author       | Aaron Kindelt |
