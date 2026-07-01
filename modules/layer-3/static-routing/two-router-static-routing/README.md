# Lab Guide — Static Two Router

## Overview

This lab demonstrates basic static routing between two routers.

Static routes allow routers to forward traffic to remote networks without using a dynamic routing protocol. In this lab, R1 and R2 are connected by a point-to-point `/30` link. Each router has one directly connected LAN. Default static routes are configured on both routers so traffic destined for remote networks is forwarded across the point-to-point link.

This lab validates interface addressing, default static route configuration, routing table behavior, the absence of dynamic routing protocols, neighbor discovery, ARP resolution, and routed reachability between LAN gateways.

**Lab Status:** Validated

**End-to-End Verification:** Successful

---

## Objectives

* [x] Configure router interface IP addressing.
* [x] Configure a point-to-point link between R1 and R2.
* [x] Configure LAN interfaces on both routers.
* [x] Configure default static routes on both routers.
* [x] Verify connected and static routes in the routing table.
* [x] Confirm no dynamic routing protocol is configured.
* [x] Verify ARP and CDP neighbor information.
* [x] Verify routed reachability between LAN gateways.
* [x] Capture selected ICMP traffic for packet-level validation.

---

## Topology

The topology uses two routers connected by a point-to-point link. Each router has one LAN segment.

Topology artifacts are available in the `topology/` folder.
 
- [Open topology diagram](topology/diagram.svg)
- [Open CML topology export](topology/topology.yaml)

---

## Addressing Table

### Router Links

| Device | Interface | IP Address      | Connected To | Description               |
| ------ | --------- | --------------- | ------------ | ------------------------- |
| R1     | Gi0/0     | 10.0.0.1/30     | R2 Gi0/0     | Point-to-point link to R2 |
| R1     | Gi0/1     | 192.168.0.1/24  | R1 LAN       | R1 LAN gateway            |
| R2     | Gi0/0     | 10.0.0.2/30     | R1 Gi0/0     | Point-to-point link to R1 |
| R2     | Gi0/1     | 192.168.10.1/24 | R2 LAN       | R2 LAN gateway            |

### Client Addressing

| Client | Interface | IP Address      | Default Gateway | Description           |
| ------ | --------- | --------------- | --------------- | --------------------- |
| C1     | eth0      | 192.168.0.2/24  | 192.168.0.1     | Test client on R1 LAN |
| C2     | eth0      | 192.168.10.2/24 | 192.168.10.1    | Test client on R2 LAN |

---

## Static Route Design

| Router | Route Type           | Destination | Next Hop | Purpose                          |
| ------ | -------------------- | ----------- | -------- | -------------------------------- |
| R1     | Default static route | 0.0.0.0/0   | 10.0.0.2 | Send non-local traffic toward R2 |
| R2     | Default static route | 0.0.0.0/0   | 10.0.0.1 | Send non-local traffic toward R1 |

**Design note:** This lab intentionally uses default static routes instead of specific static routes. In a two-router lab, this keeps the configuration simple and clearly demonstrates next-hop forwarding.

---

## Configuration Steps

### Step 1 — R1 Interface and Static Route Configuration

**Note:** The CLI examples below are annotated for readability. Clean device configurations are available in [`configs/`](configs/).

**R1**

```bash
# R1 Configuration Block
enable # Enters privileged EXEC mode.
configure terminal # Enters global configuration mode.
hostname R1 # Sets hostname to R1.

interface gigabitEthernet0/0 # Targets point-to-point interface to R2.
description P2P to R2 # Adds interface description.
ip address 10.0.0.1 255.255.255.252 # Assigns /30 point-to-point IP address.
no shutdown # Enables the interface.

interface gigabitEthernet0/1 # Targets R1 LAN interface.
description R1 LAN # Adds interface description.
ip address 192.168.0.1 255.255.255.0 # Assigns R1 LAN gateway IP address.
no shutdown # Enables the interface.

ip route 0.0.0.0 0.0.0.0 10.0.0.2 # Creates default route pointing to R2.

end # Returns to privileged EXEC mode.
write memory # Saves the running configuration.
```

---

### Step 2 — R2 Interface and Static Route Configuration

**R2**

```bash
# R2 Configuration Block
enable # Enters privileged EXEC mode.
configure terminal # Enters global configuration mode.
hostname R2 # Sets hostname to R2.

interface gigabitEthernet0/0 # Targets point-to-point interface to R1.
description P2P to R1 # Adds interface description.
ip address 10.0.0.2 255.255.255.252 # Assigns /30 point-to-point IP address.
no shutdown # Enables the interface.

interface gigabitEthernet0/1 # Targets R2 LAN interface.
description R2 LAN # Adds interface description.
ip address 192.168.10.1 255.255.255.0 # Assigns R2 LAN gateway IP address.
no shutdown # Enables the interface.

ip route 0.0.0.0 0.0.0.0 10.0.0.1 # Creates default route pointing to R1.

end # Returns to privileged EXEC mode.
write memory # Saves the running configuration.
```

---

## Verification

See [`verification/verification_commands.md`](verification/verification_commands.md) for recorded command output.

### Router Verification

```bash
# Run on R1 and R2.
show ip interface brief # Confirm configured interfaces are up/up with expected IP addresses.
show ip route # Confirm connected routes and default static route.
show ip protocols # Confirm no dynamic routing protocol is configured.
show arp # Confirm local and next-hop ARP entries.
show cdp neighbors # Confirm the direct router-to-router link.
```

### Routed Path Verification

```bash
# Run on R1.
ping 192.168.10.1 source 192.168.0.1 # Confirm R1 LAN gateway can reach R2 LAN gateway.
traceroute 192.168.10.1 source 192.168.0.1 # Confirm next hop is R2 over the point-to-point link.
```

### Packet Captures

The following packet capture provides additional evidence for selected ICMP traffic:

| Capture                                                          | Description                                               |
| ---------------------------------------------------------------- | --------------------------------------------------------- |
| [`R1_to_R2_ping.pcap`](verification/captures/R1_to_R2_ping.pcap) | ICMP traffic from R1 LAN source toward the R2 LAN gateway |

---

## Troubleshooting

**Note:** These are quick-reference checks for this lab. They are not intended to be an exhaustive static routing troubleshooting guide. After any change, re-run the verification steps to confirm the expected behavior.

```bash
# Static route is missing from the routing table.
show running-config | include ip route # Confirm the static route exists in the running configuration.
show ip route static # Confirm static routes installed in the routing table.
show ip route # Confirm connected routes and default route.
show ip interface brief # Confirm next-hop interface is up/up.

# Static route is configured but not used.
show ip route <destination-network> # Confirm the route selected for a destination.
show running-config | include ip route # Confirm the next-hop IP is correct.
ping <next-hop-ip> source <local-p2p-ip> # Confirm next-hop reachability.
show arp # Confirm next-hop MAC address resolution.

# One-way connectivity occurs.
show ip route # Confirm both routers have return paths.
show running-config | include ip route # Confirm static routes exist on both routers.
ping 10.0.0.2 source 10.0.0.1 # From R1, confirm point-to-point reachability to R2.
ping 10.0.0.1 source 10.0.0.2 # From R2, confirm point-to-point reachability to R1.

# LAN gateway reachability fails.
show ip interface gigabitEthernet0/1 # Confirm LAN interface is up/up with correct IP address.
show ip route connected # Confirm LAN network appears as connected.
ping <remote-lan-gateway> source <local-lan-gateway> # Test routed reachability between LAN gateways.
traceroute <remote-lan-gateway> source <local-lan-gateway> # Confirm where the path stops.

# Dynamic routing appears unexpectedly.
show ip protocols # Confirm no dynamic routing protocol is active.
show running-config | section router # Check for unexpected routing protocol configuration.
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
