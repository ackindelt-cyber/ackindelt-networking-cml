# Lab Guide — Static NAT

## Overview

This lab demonstrates how to configure and verify Static Network Address Translation on a Cisco router.

Static NAT creates a permanent one-to-one mapping between an inside local address and an inside global address. In this lab, R1 maps inside host `192.168.10.10` to public address `203.0.113.66`.

R2 simulates an upstream ISP router. The outside host reaches the inside host by sending traffic to the public NAT address, not the private inside address. R1 translates traffic between the inside local address and the inside global address in both directions.

This lab validates NAT inside and outside interface roles, static one-to-one NAT mapping, public NAT block routing, inside-to-outside connectivity, outside-to-inside connectivity, NAT statistics, and NAT translation output.

**Lab Status:** Validated

**End-to-End Verification:** Successful

---

## Objectives

* [x] Configure router interface IP addressing.
* [x] Configure NAT inside and outside interfaces.
* [x] Configure a static one-to-one NAT mapping.
* [x] Configure routing toward the ISP.
* [x] Configure ISP-side routing toward the public NAT block.
* [x] Verify the static NAT translation table.
* [x] Verify NAT statistics.
* [x] Verify inside-to-outside connectivity.
* [x] Verify outside-to-inside connectivity using the inside global address.
* [x] Capture selected translated traffic for packet-level validation.

---

## Topology

The topology uses an inside LAN, a customer edge router, a simulated ISP router, and an outside host network.

![Topology Diagram](topology/diagram.svg)

---

## Network Summary

| Network         | Purpose                                   |
| --------------- | ----------------------------------------- |
| 192.168.10.0/24 | Inside LAN                                |
| 198.51.100.0/30 | WAN point-to-point link between R1 and R2 |
| 203.0.114.0/24  | Outside host network                      |
| 203.0.113.64/27 | Public NAT block routed to R1             |
| 203.0.113.66/32 | Inside global address mapped to C1        |

---

## Addressing Table

| Device | Interface | IP Address       | Connected To | Description             |
| ------ | --------- | ---------------- | ------------ | ----------------------- |
| R1     | Gi0/0     | 192.168.10.1/24  | A1 Gi0/0     | Inside LAN gateway      |
| R1     | Gi0/1     | 198.51.100.2/30  | R2 Gi0/0     | WAN link to ISP         |
| R2     | Gi0/0     | 198.51.100.1/30  | R1 Gi0/1     | WAN link to customer    |
| R2     | Gi0/1     | 203.0.114.1/24   | C2 ens2      | Outside network gateway |
| A1     | Gi0/0     | N/A              | R1 Gi0/0     | Uplink to R1            |
| A1     | Gi0/1     | N/A              | C1 ens2      | Access port to C1       |
| C1     | ens2      | 192.168.10.10/24 | A1 Gi0/1     | Inside host             |
| C2     | ens2      | 203.0.114.100/24 | R2 Gi0/1     | Outside host            |

---

## Static NAT Mapping

| NAT Term         | Address         | Description                                                  |
| ---------------- | --------------- | ------------------------------------------------------------ |
| Inside Local     | 192.168.10.10   | C1 private address on the inside LAN                         |
| Inside Global    | 203.0.113.66    | Public address used to represent C1 outside the NAT boundary |
| Outside Global   | 203.0.114.100   | C2 outside host address                                      |
| Public NAT Block | 203.0.113.64/27 | Public block routed from R2 toward R1                        |

---

## NAT Policy

| Component           | Configuration                        | Purpose                                  |
| ------------------- | ------------------------------------ | ---------------------------------------- |
| Inside interface    | R1 Gi0/0                             | Receives traffic from the inside LAN     |
| Outside interface   | R1 Gi0/1                             | Sends translated traffic toward the ISP  |
| Static NAT mapping  | `192.168.10.10` to `203.0.113.66`    | Permanently maps C1 to a public address  |
| R1 default route    | `0.0.0.0/0` via `198.51.100.1`       | Sends outside-bound traffic toward R2    |
| R2 public NAT route | `203.0.113.64/27` via `198.51.100.2` | Sends public NAT block traffic toward R1 |

**Design note:** R2 does not need to route directly to the private inside LAN for static NAT validation. Outside traffic targets the public inside global address `203.0.113.66`, and R1 translates it to the private inside local address `192.168.10.10`.

---

## Configuration Steps

### Step 1 — R1 Static NAT Configuration

**Note:** The CLI examples below are annotated for readability. Clean device configurations are available in [`configs/`](configs/).

**R1**

```bash
# R1 Static NAT Configuration
enable # Enters privileged EXEC mode.
configure terminal # Enters global configuration mode.
hostname R1 # Sets hostname to R1.

ip nat inside source static 192.168.10.10 203.0.113.66 # Creates permanent one-to-one static NAT mapping.

interface gigabitEthernet0/0 # Targets inside LAN interface.
description INSIDE_LAN to A1 # Adds interface description.
ip address 192.168.10.1 255.255.255.0 # Assigns inside LAN gateway address.
ip nat inside # Marks interface as NAT inside.
no shutdown # Enables the interface.

interface gigabitEthernet0/1 # Targets WAN interface to R2.
description OUTSIDE_WAN to R2 # Adds interface description.
ip address 198.51.100.2 255.255.255.252 # Assigns WAN IP address.
ip nat outside # Marks interface as NAT outside.
no shutdown # Enables the interface.

ip route 0.0.0.0 0.0.0.0 198.51.100.1 # Configures default route toward R2.

end # Returns to privileged EXEC mode.
write memory # Saves the running configuration.
```

---

### Step 2 — R2 ISP-Side Routing Configuration

**R2**

```bash
# R2 ISP-Side Routing Configuration
enable # Enters privileged EXEC mode.
configure terminal # Enters global configuration mode.
hostname R2 # Sets hostname to R2.

interface gigabitEthernet0/0 # Targets WAN interface to R1.
description CUSTOMER_WAN to R1 # Adds interface description.
ip address 198.51.100.1 255.255.255.252 # Assigns WAN IP address.
no shutdown # Enables the interface.

interface gigabitEthernet0/1 # Targets outside host network.
description OUTSIDE_GATEWAY # Adds interface description.
ip address 203.0.114.1 255.255.255.0 # Assigns outside network gateway address.
no shutdown # Enables the interface.

ip route 203.0.113.64 255.255.255.224 198.51.100.2 # Routes public NAT block toward R1.

end # Returns to privileged EXEC mode.
write memory # Saves the running configuration.
```

---

### Step 3 — A1 Access Switch Configuration

**A1**

```bash
# A1 Basic Access Switch Configuration
enable # Enters privileged EXEC mode.
configure terminal # Enters global configuration mode.
hostname A1 # Sets hostname to A1.

interface gigabitEthernet0/0 # Targets uplink to R1.
description Uplink to R1 # Adds interface description.
no shutdown # Enables the interface.

interface gigabitEthernet0/1 # Targets access port to C1.
description Access to C1 # Adds interface description.
no shutdown # Enables the interface.

end # Returns to privileged EXEC mode.
write memory # Saves the running configuration.
```

---

## Verification

See [`verification/verification_commands.md`](verification/verification_commands.md) for recorded command output.

### R1 NAT Verification

```bash
show ip nat translations # Confirm static mapping and active translated traffic.
show ip nat statistics # Confirm NAT inside/outside interfaces and translation counters.
show ip route # Confirm R1 has a default route toward R2.
```

### Inside-to-Outside Verification

```bash
# Run on C1.
ping -w 5 203.0.114.100 # Confirm inside host can reach outside host through static NAT.
```

### Outside-to-Inside Verification

```bash
# Run on C2.
ping -w 5 203.0.113.66 # Confirm outside host can reach inside host using the inside global address.
```

---

## Troubleshooting

**Note:** These are quick-reference checks for this lab. They are not intended to be an exhaustive NAT troubleshooting guide. After any change, re-run the verification steps to confirm the expected behavior.

```bash
# Inside host cannot reach outside host.
show ip interface brief # Verify inside and outside interfaces are up/up.
show ip route # Verify R1 has a default route toward R2.
show arp # Verify ARP resolution for inside host and WAN next hop.
show running-config | section ip nat # Confirm static NAT mapping and interface roles.
show ip nat translations # Confirm static entry exists.

# NAT translation does not appear in the translation table.
show running-config | section ip nat # Verify static NAT configuration.
show running-config interface gigabitEthernet0/0 # Confirm ip nat inside is configured.
show running-config interface gigabitEthernet0/1 # Confirm ip nat outside is configured.
show ip nat statistics # Confirm NAT interfaces are recognized.

# Inside-to-outside works, but outside-to-inside fails.
show ip route 203.0.113.64 # On R2, confirm public NAT block routes toward R1.
show ip nat translations # On R1, confirm static mapping exists.
show running-config | section ip nat # Confirm inside local and inside global addresses.
ping 198.51.100.2 source 198.51.100.1 # From R2, confirm WAN reachability to R1.

# Basic connectivity fails.
show ip interface brief # Confirm all required interfaces are up/up.
show cdp neighbors # Confirm R1-to-R2 and R1-to-A1 physical adjacency.
show arp # Confirm local next-hop resolution.
ping 192.168.10.1 # From C1, confirm local gateway reachability.
ping 203.0.114.1 # From C2, confirm outside gateway reachability.
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
