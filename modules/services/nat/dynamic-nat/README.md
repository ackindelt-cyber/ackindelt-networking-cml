# Lab Guide — Dynamic NAT

## Overview

This lab demonstrates how to configure and verify Dynamic Network Address Translation on a Cisco router.

Dynamic NAT translates inside local addresses to addresses from a configured public address pool. Unlike PAT, Dynamic NAT does not overload many inside hosts onto a single public IP address. Each active inside host consumes an available address from the NAT pool.

In this lab, R1 translates inside traffic from `192.168.10.0/24` to addresses from the public NAT pool `203.0.113.66` through `203.0.113.70`. R2 simulates an upstream ISP router and routes the public NAT block back toward R1. C2 represents an outside host.

This lab validates NAT inside and outside interface roles, NAT pool configuration, ACL-based NAT classification, public NAT block routing, dynamic translation creation, and inside-to-outside connectivity.

**Lab Status:** Validated

**End-to-End Verification:** Successful

---

## Objectives

* [x] Configure router interface IP addressing.
* [x] Configure NAT inside and outside interfaces.
* [x] Define a Dynamic NAT pool.
* [x] Create an ACL to identify inside traffic eligible for translation.
* [x] Bind the ACL to the NAT pool.
* [x] Configure routing toward the ISP and public NAT block.
* [x] Generate inside-to-outside traffic.
* [x] Verify NAT statistics and translations.
* [x] Verify inside-to-outside reachability using translated traffic.

---

## Topology

The topology uses an inside LAN, a customer edge router, a simulated ISP router, and an outside host network.

Topology artifacts are available in the `topology/` folder.
 
- [Open topology diagram](topology/diagram.svg)
- [Open CML topology export](topology/topology.yaml)

---

## Network Summary

| Network                     | Purpose                                   |
| --------------------------- | ----------------------------------------- |
| 192.168.10.0/24             | Inside LAN                                |
| 198.51.100.0/30             | WAN point-to-point link between R1 and R2 |
| 203.0.114.0/24              | Outside host network                      |
| 203.0.113.64/27             | Public NAT block routed to R1             |
| 203.0.113.66 - 203.0.113.70 | Dynamic NAT pool used by R1               |

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
| C1     | ens2      | 192.168.10.10/24 | A1 Gi0/1     | Inside client           |
| C2     | ens2      | 203.0.114.100/24 | R2 Gi0/1     | Outside host            |

---

## NAT Pool

| Pool Name | Start IP     | End IP       | Netmask         | Usable Pool Addresses | Purpose                         |
| --------- | ------------ | ------------ | --------------- | --------------------- | ------------------------------- |
| NAT_POOL  | 203.0.113.66 | 203.0.113.70 | 255.255.255.248 | 5                     | Dynamic NAT public address pool |

---

## NAT Policy

| Component         | Configuration                               | Purpose                                            |
| ----------------- | ------------------------------------------- | -------------------------------------------------- |
| Inside interface  | R1 Gi0/0                                    | Receives traffic from the inside LAN               |
| Outside interface | R1 Gi0/1                                    | Sends translated traffic toward the ISP            |
| NAT ACL           | ACL 1 permits `192.168.10.0/24`             | Identifies inside traffic eligible for translation |
| NAT pool          | `NAT_POOL`                                  | Supplies public addresses for dynamic translations |
| NAT binding       | `ip nat inside source list 1 pool NAT_POOL` | Links eligible inside traffic to the public pool   |

**Design note:** Outside-initiated traffic does not create a Dynamic NAT translation. Return traffic is supported only after an inside host creates an active translation.

---

## Configuration Steps

### Step 1 — R1 Dynamic NAT Configuration

**Note:** The CLI examples below are annotated for readability. Clean device configurations are available in [`configs/`](configs/).

**R1**

```bash
# R1 Dynamic NAT Configuration
enable # Enters privileged EXEC mode.
configure terminal # Enters global configuration mode.
hostname R1 # Sets hostname to R1.

ip nat pool NAT_POOL 203.0.113.66 203.0.113.70 netmask 255.255.255.248 # Creates Dynamic NAT public address pool.

access-list 1 permit 192.168.10.0 0.0.0.255 # Identifies inside LAN traffic eligible for NAT.

ip nat inside source list 1 pool NAT_POOL # Binds ACL 1 to the Dynamic NAT pool.

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

ip route 203.0.113.64 255.255.255.224 198.51.100.2 # Routes the public NAT block toward R1.

end # Returns to privileged EXEC mode.
write memory # Saves the running configuration.
```

**Design note:** R2 does not need a route to the private inside LAN for NAT validation. Return traffic from the outside host is destined to the translated public address, not the original inside local address.

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
show running-config | section nat # Confirm NAT inside/outside roles, NAT pool, and NAT binding.
show ip nat statistics # Confirm NAT pool allocation, inside/outside interfaces, and translation counters.
show ip nat translations # Confirm inside local to inside global translation.
show ip route # Confirm R1 has a default route toward R2.
```

### Client Verification

```bash
# Run on C1.
ping -w 5 203.0.114.100 # Confirm inside client can reach the outside host through Dynamic NAT.
```

### Outside-Initiated Traffic

Outside-initiated traffic is intentionally not used as a primary validation test in this lab. Dynamic NAT translations are created by inside-initiated traffic. An outside host cannot reliably initiate traffic to an inside host unless a valid translation already exists.

---

## Troubleshooting

**Note:** These are quick-reference checks for this lab. They are not intended to be an exhaustive NAT troubleshooting guide. After any change, re-run the verification steps to confirm the expected behavior.

```bash
# Inside host cannot reach outside host.
show ip interface brief # Verify inside and outside interfaces are up/up.
show ip route # Verify R1 has a default route toward R2.
show arp # Verify ARP resolution for inside host and WAN next hop.
show running-config | section nat # Confirm NAT pool, NAT binding, and interface roles.
show access-lists 1 # Confirm ACL matches inside LAN traffic.

# NAT translation does not appear.
show running-config interface gigabitEthernet0/0 # Confirm ip nat inside is configured.
show running-config interface gigabitEthernet0/1 # Confirm ip nat outside is configured.
show running-config | section ip nat # Confirm NAT pool and inside source statement.
show access-lists 1 # Confirm ACL permits the inside source network.
show ip nat statistics # Confirm pool usage and translation counters.

# NAT pool exhausts.
show ip nat statistics # Confirm allocated and available pool addresses.
show ip nat translations # Identify active translations consuming pool addresses.
clear ip nat translation * # Clear translations during lab reset.

# Return traffic fails.
show ip nat translations # Confirm active translation exists.
show ip route 203.0.113.64 # On R2, confirm public NAT block routes toward R1.
show ip route # On R1, confirm route back to inside LAN exists as connected.
ping 198.51.100.1 source 198.51.100.2 # Confirm R1-to-R2 WAN reachability.

# NAT works temporarily, then stops.
show ip nat translations # Confirm translation timers and active entries.
show ip nat statistics # Confirm expired translations and current pool allocation.
show logging # Review relevant interface or NAT messages.
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
