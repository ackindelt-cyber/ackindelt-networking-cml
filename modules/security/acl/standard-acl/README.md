# Lab Guide — Standard ACL

## Overview

This lab demonstrates how to configure and verify a standard IPv4 Access Control List.

Standard ACLs classify traffic based only on source IP address. In this lab, R1 uses standard ACL `10` to permit traffic sourced from the inside LAN network `192.168.10.0/24`. The ACL is applied outbound on R1’s WAN interface toward R2.

Traffic from inside clients C1 and C3 is generated toward the outside host at `203.0.113.100`. Successful pings and ACL match counters confirm that traffic sourced from the permitted inside LAN matches the standard ACL.

**Lab Status:** Validated

**End-to-End Verification:** Successful

---

## Objectives

* [x] Configure router interface IP addressing.
* [x] Configure a numbered standard ACL.
* [x] Permit traffic sourced from the inside LAN network.
* [x] Apply the ACL outbound on R1’s WAN interface.
* [x] Configure routing between inside and outside networks.
* [x] Generate traffic from inside clients.
* [x] Verify ACL match counters.
* [x] Verify ACL placement using interface inspection.
* [x] Verify routed reachability from inside clients to the outside host.

---

## Topology

The topology uses an inside LAN, a customer edge router, an ISP-side router, and an outside host network.

Topology artifacts are available in the `topology/` folder.
 
- [Open topology diagram](topology/diagram.svg)
- [Open CML topology export](topology/topology.yaml)

---

## Network Summary

| Network         | Purpose                                   |
| --------------- | ----------------------------------------- |
| 192.168.10.0/24 | Inside LAN                                |
| 198.51.100.0/30 | WAN point-to-point link between R1 and R2 |
| 203.0.113.0/24  | Outside network                           |

---

## Addressing Table

| Device | Interface | IP Address       | Connected To | Description             |
| ------ | --------- | ---------------- | ------------ | ----------------------- |
| R1     | Gi0/0     | 192.168.10.1/24  | A1 Gi0/0     | Inside LAN gateway      |
| R1     | Gi0/1     | 198.51.100.2/30  | R2 Gi0/0     | WAN link to R2          |
| R2     | Gi0/0     | 198.51.100.1/30  | R1 Gi0/1     | WAN link to R1          |
| R2     | Gi0/1     | 203.0.113.1/24   | C2 eth0      | Outside network gateway |
| A1     | Gi0/0     | N/A              | R1 Gi0/0     | Uplink to R1            |
| A1     | Gi0/1     | N/A              | C1 eth0      | Access port to C1       |
| A1     | Gi0/2     | N/A              | C3 eth0      | Access port to C3       |
| C1     | eth0      | 192.168.10.10/24 | A1 Gi0/1     | Inside client           |
| C2     | eth0      | 203.0.113.100/24 | R2 Gi0/1     | Outside host            |
| C3     | eth0      | 192.168.10.20/24 | A1 Gi0/2     | Inside client           |

---

## ACL Policy

| ACL | Action | Source          | Placement | Direction | Purpose                                     |
| --- | ------ | --------------- | --------- | --------- | ------------------------------------------- |
| 10  | Permit | 192.168.10.0/24 | R1 Gi0/1  | Outbound  | Permit inside LAN traffic leaving toward R2 |

**Design note:** Standard ACLs match only on source address. This lab uses the ACL to classify and permit traffic sourced from the inside LAN. Any traffic that does not match the permit statement would be denied by the implicit deny at the end of the ACL, but the uploaded verification focuses on permitted inside-source traffic and match counters.

---

## Configuration Steps

### Step 1 — R1 Interface, Routing, and ACL Configuration

**Note:** The CLI examples below are annotated for readability. Clean device configurations are available in [`configs/`](configs/).

**R1**

```bash
# R1 Standard ACL Configuration
enable # Enters privileged EXEC mode.
configure terminal # Enters global configuration mode.
hostname R1 # Sets hostname to R1.

access-list 10 permit 192.168.10.0 0.0.0.255 # Permits traffic sourced from the inside LAN.

interface gigabitEthernet0/0 # Targets inside LAN interface.
description INSIDE_LAN to A1 # Adds interface description.
ip address 192.168.10.1 255.255.255.0 # Assigns inside LAN gateway address.
no shutdown # Enables the interface.

interface gigabitEthernet0/1 # Targets WAN interface to R2.
description OUTSIDE_WAN to R2 # Adds interface description.
ip address 198.51.100.2 255.255.255.252 # Assigns WAN IP address.
ip access-group 10 out # Applies standard ACL 10 outbound toward R2.
no shutdown # Enables the interface.

ip route 0.0.0.0 0.0.0.0 198.51.100.1 # Configures default route toward R2.

end # Returns to privileged EXEC mode.
write memory # Saves the running configuration.
```

---

### Step 2 — R2 Interface and Return Route Configuration

**R2**

```bash
# R2 Routing Configuration
enable # Enters privileged EXEC mode.
configure terminal # Enters global configuration mode.
hostname R2 # Sets hostname to R2.

interface gigabitEthernet0/0 # Targets WAN interface to R1.
description CUSTOMER_WAN to R1 # Adds interface description.
ip address 198.51.100.1 255.255.255.252 # Assigns WAN IP address.
no shutdown # Enables the interface.

interface gigabitEthernet0/1 # Targets outside network interface.
description OUTSIDE_GATEWAY # Adds interface description.
ip address 203.0.113.1 255.255.255.0 # Assigns outside network gateway address.
no shutdown # Enables the interface.

ip route 192.168.10.0 255.255.255.0 198.51.100.2 # Configures return route to inside LAN through R1.

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

interface gigabitEthernet0/2 # Targets access port to C3.
description Access to C3 # Adds interface description.
no shutdown # Enables the interface.

end # Returns to privileged EXEC mode.
write memory # Saves the running configuration.
```

---

## Verification

See [`verification/verification_commands.md`](verification/verification_commands.md) for recorded command output.

**Note:** R2 verification is intentionally not included in the uploaded evidence. R2 acts as the transit router and has no ACL policy applied.

### R1 ACL Verification

```bash
show ip access-lists 10 # Confirm ACL exists and match counters increment after traffic tests.
show ip interface gigabitEthernet0/1 # Confirm ACL 10 is applied outbound on the WAN interface.
show ip route # Confirm R1 has a default route toward R2.
```

### Inside Client Verification

```bash
# Run on C1.
ping -w 5 203.0.113.100 # Confirm C1 can reach the outside host.

# Run on C3.
ping -w 5 203.0.113.100 # Confirm C3 can reach the outside host.
```

---

## Troubleshooting

**Note:** These are quick-reference checks for this lab. They are not intended to be an exhaustive ACL troubleshooting guide. After any change, re-run the verification steps to confirm the expected behavior.

```bash
# Inside hosts cannot reach the outside host.
show ip interface brief # Verify R1 interfaces are up/up.
show ip route # Verify default route toward R2 exists.
show ip interface gigabitEthernet0/1 # Verify ACL 10 is applied outbound on the correct interface.
show ip access-lists 10 # Verify ACL exists and permits the inside source network.

# ACL hit counter does not increment.
show ip access-lists 10 # Verify ACL exists and review match counters.
show running-config | include access-list 10 # Verify the permitted source network.
show ip interface gigabitEthernet0/1 # Confirm ACL 10 is applied outbound.
show running-config interface gigabitEthernet0/1 # Confirm the ip access-group command.

# Expected traffic is denied.
show ip access-lists 10 # Confirm the source network matches the ACL permit statement.
show ip route # Confirm routing toward the destination.
show ip route 203.0.113.100 # Confirm next-hop selection toward outside host.
ping 198.51.100.1 source 198.51.100.2 # Confirm R1 can reach R2 over the WAN link.

# Return traffic fails.
show ip interface gigabitEthernet0/1 # Verify ACL is not applied inbound on R1 WAN interface.
show ip route # Confirm R1 has a route toward the outside network.
show ip route 192.168.10.0 # On R2, confirm return route exists toward inside LAN.

# Client-side tests fail.
ip addr # On Linux clients, confirm IP address.
ip route # On Linux clients, confirm default gateway.
ping 192.168.10.1 # Confirm inside client can reach local gateway.
ping 203.0.113.100 # Confirm reachability to outside host.
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
