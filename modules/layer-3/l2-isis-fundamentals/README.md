# Lab Guide — # Lab Guide — L2-IS-IS Fundamentals

## Overview

This lab demonstrates a basic Level 2 IS-IS deployment across a three-router Cisco CML topology.

The lab validates IS-IS adjacency formation, link-state database synchronization, IPv4 route propagation, SPF path selection, and convergence after topology changes.

Lab Status: In Progress

End-to-End Verification: Not Tested

---

## Objectives

* [ ] Configure Level 2 IS-IS across a three-router topology.
* [ ] Verify IS-IS adjacency formation and link-state database synchronization.
* [ ] Validate IPv4 route propagation between router loopbacks.
* [ ] Verify SPF path selection and reconvergence after a link failure.
* [ ] Troubleshoot common IS-IS configuration and adjacency failures.

---

## Topology

## Topology

The lab uses three Cisco IOSv routers connected in a triangle. Each router has direct Layer 3 connectivity to the other two routers, providing redundant paths for IS-IS route calculation and convergence testing.

A loopback interface on each router provides a stable prefix for verifying IS-IS route propagation.

![Topology Diagram](topology/<diagram-file-name>)

---

##  Tables

### Physical Links

| Local Device | Local Interface | Peer Device | Peer Interface | Description |
| ------------ | --------------- | ----------- | -------------- | ----------- |
| R1 | Gi0/0 | R2 | Gi0/0 | R1-R2 routed transit link |
| R1 | Gi0/1 | R3 | Gi0/0 | R1-R3 routed transit link |
| R2 | Gi0/1 | R3 | Gi0/1 | R2-R3 routed transit link |

### Addressing

| Device | Interface | IPv4 Address | Prefix Length | Purpose |
| ------ | --------- | ------------ | ------------- | ------- |
| R1 | Gi0/0 | 10.0.12.1 | /30 | R1-R2 transit |
| R1 | Gi0/1 | 10.0.13.1 | /30 | R1-R3 transit |
| R1 | Loopback0 | 10.255.0.1 | /32 | Stable IS-IS advertised prefix |
| R2 | Gi0/0 | 10.0.12.2 | /30 | R1-R2 transit |
| R2 | Gi0/1 | 10.0.23.1 | /30 | R2-R3 transit |
| R2 | Loopback0 | 10.255.0.2 | /32 | Stable IS-IS advertised prefix |
| R3 | Gi0/0 | 10.0.13.2 | /30 | R1-R3 transit |
| R3 | Gi0/1 | 10.0.23.2 | /30 | R2-R3 transit |
| R3 | Loopback0 | 10.255.0.3 | /32 | Stable IS-IS advertised prefix |

### IS-IS Parameters

| Device | IS-IS Level | Area | System ID | NET | Circuit Type |
| ------ | ------------ | ---- | --------- | --- | ------------ |
| R1 | Level 2 | 49.0001 | 0000.0000.0001 | 49.0001.0000.0000.0001.00 | Point-to-point |
| R2 | Level 2 | 49.0001 | 0000.0000.0002 | 49.0001.0000.0000.0002.00 | Point-to-point |
| R3 | Level 2 | 49.0001 | 0000.0000.0003 | 49.0001.0000.0000.0003.00 | Point-to-point |



---

## Configuration Steps

> **Note:** The CLI examples below are annotated for readability. Clean device configurations are available in [`configs/`](configs/).

> **Design note:** This lab uses a Level 2-only IS-IS domain across three routed point-to-point links. Each router uses a unique NET and System ID. Loopback interfaces are advertised into IS-IS to provide stable prefixes for route propagation and convergence testing.


### Base Configuration

**R1**
```bash
# R1 Base Configuration
enable # Enters privileged EXEC mode.
configure terminal # Enters global configuration mode.
hostname R1 # Sets the router hostname.
no ip domain lookup # Prevents IOS from attempting DNS resolution for mistyped commands.

interface GigabitEthernet0/0 # Enters the interface connected to R2.
ip address 10.0.12.1 255.255.255.252 # Assigns R1's address on the R1-R2 transit network.
no shutdown # Enables the interface.

interface GigabitEthernet0/1 # Enters the interface connected to R3.
ip address 10.0.13.1 255.255.255.252 # Assigns R1's address on the R1-R3 transit network.
no shutdown # Enables the interface.
interface Loopback0 # Creates and enters R1's loopback interface.
ip address 10.255.0.1 255.255.255.255 # Assigns R1's stable loopback address.

end # Returns to privileged EXEC mode.
write memory # Saves the running configuration.
```

**R2**
```bash
# R2 Base Configuration
enable # Enters privileged EXEC mode.
configure terminal # Enters global configuration mode.
hostname R2 # Sets the router hostname.
no ip domain lookup # Prevents IOS from attempting DNS resolution for mistyped commands.

interface GigabitEthernet0/0 # Enters the interface connected to R1.
ip address 10.0.12.2 255.255.255.252 # Assigns R2's address on the R1-R2 transit network.
no shutdown # Enables the interface.

interface GigabitEthernet0/1 # Enters the interface connected to R3.
ip address 10.0.23.1 255.255.255.252 # Assigns R2's address on the R2-R3 transit network.
no shutdown # Enables the interface.
interface Loopback0 # Creates and enters R2's loopback interface.
ip address 10.255.0.2 255.255.255.255 # Assigns R2's stable loopback address.

end # Returns to privileged EXEC mode.
write memory # Saves the running configuration.
```

**R3**
```bash
# R3 Base Configuration
enable # Enters privileged EXEC mode.
configure terminal # Enters global configuration mode.
hostname R3 # Sets the router hostname.
no ip domain lookup # Prevents IOS from attempting DNS resolution for mistyped commands.

interface GigabitEthernet0/0 # Enters the interface connected to R1.
ip address 10.0.13.2 255.255.255.252 # Assigns R3's address on the R1-R3 transit network.
no shutdown # Enables the interface.

interface GigabitEthernet0/1 # Enters the interface connected to R2.
ip address 10.0.23.2 255.255.255.252 # Assigns R3's address on the R2-R3 transit network.
no shutdown # Enables the interface.

interface Loopback0 # Creates and enters R3's loopback interface.
ip address 10.255.0.3 255.255.255.255 # Assigns R3's stable loopback address.

end # Returns to privileged EXEC mode.
write memory # Saves the running configuration.
```

### IS-IS Configuration

**R1**
```bash
# R1 IS-IS Configuration
configure terminal # Enters global configuration mode.

router isis # Creates the IS-IS routing process and enters IS-IS router configuration mode
net 49.0001.0000.0000.0001.00 # Assigns R1 its IS-IS NET: area 49.0001, System ID 0000.0000.0001, NSEL 00.
is-type level-2-only # Restricts the IS-IS process to the Level 2 operation only
metric-style wide # Enables wide IS-IS metrics for modern extended reachability and larger metric values
passive-interface Loopback0 # Prevents IS-IS hello packets and adjacency formation on Loopback0 while allowing its prefix to remain advertised.

interface gigabitethernet0/0 # Enters R1-R2 transit interface
ip router isis # Enables IS-IS on this interface and associates it with the IS-IS process
isis network point-to-point # Treats the two-router Ethernet segment as a point-to-point IS-IS circuit.

interface gigabitethernet0/1 # Enters the R1-R3 transit interface
ip router isis # Enables IS-IS participation on this interface
isis network point-to-point # Treats the Ethernet link as a point-to-point IS-IS circuit

interface loopback0 # Enters R1's loopback interface
ip router isis # Advertises the loopback prefix through IS-IS

end # Returns to privileged EXEC mode.
write memory # Saves the running configuration.
```

**R2**
```bash
configure terminal # Enters global configuration mode.

router isis # Creates the IS-IS routing process and enters IS-IS router configuration mode.
net 49.0001.0000.0000.0002.00 # Assigns R2 its IS-IS NET: area 49.0001, System ID 0000.0000.0002, NSEL 00.
is-type level-2-only # Restricts the IS-IS process to Level 2 operation only.
metric-style wide # Enables modern wide IS-IS metrics.
passive-interface Loopback0 # Prevents IS-IS hello packets and adjacency formation on Loopback0 while allowing its prefix to remain advertised.

interface GigabitEthernet0/0 # Enters the R2-R1 transit interface.
ip router isis # Enables IS-IS participation and advertises the connected prefix.
isis network point-to-point # Treats the R2-R1 Ethernet link as a point-to-point IS-IS circuit.

interface GigabitEthernet0/1 # Enters the R2-R3 transit interface.
ip router isis # Enables IS-IS participation and advertises the connected prefix.
isis network point-to-point # Treats the R2-R3 Ethernet link as a point-to-point IS-IS circuit.

interface Loopback0 # Enters R2's loopback interface.
ip router isis # Advertises R2's loopback prefix into IS-IS.

end # Returns to privileged EXEC mode.
write memory # Saves the running configuration.
```

**R3**
```bash
configure terminal # Enters global configuration mode.

router isis # Creates the IS-IS routing process and enters IS-IS router configuration mode.
net 49.0001.0000.0000.0003.00 # Assigns R3 its IS-IS NET: area 49.0001, System ID 0000.0000.0003, NSEL 00.
is-type level-2-only # Restricts the IS-IS process to Level 2 operation only.
metric-style wide # Enables modern wide IS-IS metrics.
passive-interface Loopback0 # Prevents IS-IS hello packets and adjacency formation on Loopback0 while allowing its prefix to remain advertised.

interface GigabitEthernet0/0 # Enters the R3-R1 transit interface.
ip router isis # Enables IS-IS participation and advertises the connected prefix.
isis network point-to-point # Treats the R3-R1 Ethernet link as a point-to-point IS-IS circuit.

interface GigabitEthernet0/1 # Enters the R3-R2 transit interface.
ip router isis # Enables IS-IS participation and advertises the connected prefix.
isis network point-to-point # Treats the R3-R2 Ethernet link as a point-to-point IS-IS circuit.

interface Loopback0 # Enters R3's loopback interface.
ip router isis # Advertises R3's loopback prefix into IS-IS.

end # Returns to privileged EXEC mode.
write memory # Saves the running configuration.
```

---

## Verification

See [`verification/verification_commands.md`](verification/verification_commands.md) for recorded command output.

**R1**

```bash
# R1 Verification Block
show clns protocol # Verifies the IS-IS process, configured NET, and Level 2-only operation.
show clns interface # Verifies IS-IS interface participation and expected point-to-point circuit behavior.
show isis neighbors # Verifies expected Level 2 IS-IS adjacencies.
show isis database # Verifies the expected LSPs are present in the Level 2 link-state database.
show isis topology # Verifies the paths calculated by IS-IS SPF.
show ip route isis # Verifies expected IS-IS learned routes are installed in the IPv4 routing table.
ping 10.255.0.2 source Loopback0 # Verifies end-to-end reachability to R2's IS-IS-advertised loopback.
ping 10.255.0.3 source Loopback0 # Verifies end-to-end reachability to R3's IS-IS-advertised loopback.
```

**R2**

```bash
# R2 Verification Block
show clns protocol # Verifies the IS-IS process, configured NET, and Level 2-only operation.
show clns interface # Verifies IS-IS interface participation and expected point-to-point circuit behavior.
show isis neighbors # Verifies expected Level 2 IS-IS adjacencies.
show isis database # Verifies the expected LSPs are present in the Level 2 link-state database.
show isis topology # Verifies the paths calculated by IS-IS SPF.
show ip route isis # Verifies expected IS-IS learned routes are installed in the IPv4 routing table.
ping 10.255.0.1 source Loopback0 # Verifies end-to-end reachability to R1's IS-IS-advertised loopback.
ping 10.255.0.3 source Loopback0 # Verifies end-to-end reachability to R3's IS-IS-advertised loopback.
```

**R3**

```bash
# R3 Verification Block
show clns protocol # Verifies the IS-IS process, configured NET, and Level 2-only operation.
show clns interface # Verifies IS-IS interface participation and expected point-to-point circuit behavior.
show isis neighbors # Verifies expected Level 2 IS-IS adjacencies.
show isis database # Verifies the expected LSPs are present in the Level 2 link-state database.
show isis topology # Verifies the paths calculated by IS-IS SPF.
show ip route isis # Verifies expected IS-IS learned routes are installed in the IPv4 routing table.
ping 10.255.0.1 source Loopback0 # Verifies end-to-end reachability to R1's IS-IS-advertised loopback.
ping 10.255.0.2 source Loopback0 # Verifies end-to-end reachability to R2's IS-IS-advertised loopback.
```

---

## Failure Testing

**R1**

### Downed Interface

```bash
# Create downed interface failure state.
configure terminal # Enters global configuration mode.

interface GigabitEthernet0/0 # Enters the R1-R2 transit interface.
shutdown # Simulates failure of the direct R1-R2 transit link.

end # Returns to privileged EXEC mode.

# Verify downed interface and IS-IS convergence on R1.
show ip interface brief # Confirms GigabitEthernet0/0 is administratively down.
show isis neighbors # Confirms the direct R1-R2 Level 2 adjacency is gone while the R1-R3 adjacency remains up.
show ip route 10.255.0.2 # Verifies R1 recalculated a route to R2's loopback through the alternate R1-R3-R2 path.
ping 10.255.0.2 source Loopback0 # Verifies R2's loopback remains reachable after IS-IS reconvergence.
traceroute 10.255.0.2 source Loopback0 # Verifies traffic now reaches R2 through R3 instead of the failed direct link.

# Return configuration to intended state.
configure terminal # Enters global configuration mode.

interface GigabitEthernet0/0 # Enters the R1-R2 transit interface.
no shutdown # Restores the R1-R2 transit link.

end # Returns to privileged EXEC mode.
write memory # Saves the restored intended configuration to startup-config.
```

### Missing IS-IS Interface

```bash
# Create missing interface IS-IS failure state.

configure terminal # Enters global configuration mode.

interface GigabitEthernet0/0 # Enters the R1-R2 transit interface.
no ip router isis # Removes IS-IS participation from Gi0/0 while leaving the Layer 3 interface operational.

end # Returns to privileged EXEC mode.

# Verify missing interface IS-IS and IS-IS convergence on R1.
show ip interface brief # Confirms GigabitEthernet0/0 remains up/up.
show clns interface GigabitEthernet0/0 # Confirms IS-IS is no longer participating on the interface.
show isis neighbors # Confirms the R1-R2 adjacency is gone while the R1-R3 adjacency remains up.
show ip route 10.255.0.2 # Verifies R1 recalculated a route to R2's loopback through the alternate R1-R3-R2 path.
ping 10.255.0.2 source Loopback0 # Verifies R2's loopback remains reachable after IS-IS reconvergence.
traceroute 10.255.0.2 source Loopback0 # Verifies traffic now reaches R2 through R3 instead of the direct R1-R2 link.

# Return configuration to intended state.
configure terminal # Enters global configuration mode.

interface GigabitEthernet0/0 # Enters the R1-R2 transit interface.
ip router isis # Restores IS-IS participation on Gi0/0.

end # Returns to privileged EXEC mode.
write memory # Saves the restored intended configuration to startup-config.
```

### IS-IS Level Mismatch

```bash
# Create IS-IS level mismatch failure state.
configure terminal # Enters global configuration mode.

router isis # Enters the IS-IS routing process.
is-type level-1 # Changes R1 from Level 2-only to Level 1-only operation.

end # Returns to privileged EXEC mode.

# Verify IS-IS level mismatch.
show clns protocol # Confirms R1 is operating as a Level 1-only IS-IS router.
show isis neighbors # Confirms both Level 2 adjacencies are lost.
show ip route isis # Confirms R1 no longer has the remote routes previously learned through Level 2 IS-IS.
ping 10.255.0.2 source Loopback0 # Confirms R2's loopback is no longer reachable through IS-IS.
ping 10.255.0.3 source Loopback0 # Confirms R3's loopback is no longer reachable through IS-IS.

# Return configuration to intended state.
configure terminal # Enters global configuration mode.

router isis # Enters the IS-IS routing process.
is-type level-2-only # Restores R1 to the intended Level 2-only IS-IS operation.

end # Returns to privileged EXEC mode.
write memory # Saves the restored intended configuration to startup-config.
```

### Missing Loopback Prefix

```bash
# Create missing loopback prefix failure state.
configure terminal # Enters global configuration mode.

interface Loopback0 # Enters R1's loopback interface.
no ip router isis # Removes R1's loopback prefix from IS-IS while leaving the loopback interface operational.

end # Returns to privileged EXEC mode.

# Verify missing loopback prefix from IS-IS.
show isis neighbors # Confirms R1's adjacencies with R2 and R3 remain up.
show isis database detail # Verifies R1's LSP no longer advertises the Loopback0 prefix.
show ip route 10.255.0.1 # Run on R2 to confirm R1's loopback route has disappeared from the routing table.
ping 10.255.0.1 source Loopback0 # Run on R2 to confirm R1's loopback is no longer reachable through IS-IS.

# Return configuration to intended state.
configure terminal # Enters global configuration mode.
interface Loopback0 # Enters R1's loopback interface.
ip router isis # Restores R1's loopback prefix to IS-IS.

end # Returns to privileged EXEC mode.
write memory # Saves the restored intended configuration to startup-config.
```

### Missing IS-IS NET

```bash
# Create missing IS-IS NET failure state.
configure terminal # Enters global configuration mode.

router isis # Enters the IS-IS routing process.
no net 49.0001.0000.0000.0001.00 # Removes R1's IS-IS NET and therefore its IS-IS area and System ID identity.

end # Returns to privileged EXEC mode.

# Verify missing IS-IS NET failure.
show clns protocol # Confirms the IS-IS process no longer has a configured NET.
show isis neighbors # Confirms R1 can no longer maintain its expected IS-IS adjacencies.
show isis database # Verifies the effect of the missing NET on R1's Level 2 link-state database.
show ip route isis # Confirms remote IS-IS-learned routes are no longer installed on R1.
ping 10.255.0.2 source Loopback0 # Confirms R2's loopback is no longer reachable through IS-IS.
ping 10.255.0.3 source Loopback0 # Confirms R3's loopback is no longer reachable through IS-IS.

# Return configuration to intended state.
configure terminal # Enters global configuration mode.

router isis # Enters the IS-IS routing process.
net 49.0001.0000.0000.0001.00 # Restores R1's IS-IS NET, including its area address, System ID, and NSEL.

end # Returns to privileged EXEC mode.
write memory # Saves the restored intended configuration to startup-config.
```

---

## Troubleshooting

> **Note:** These are quick-reference checks for this lab. They are not intended to be an exhaustive troubleshooting guide. After any change, re-run the verification steps to confirm the expected behavior.

```bash
# Expected IS-IS adjacency is missing.
show ip interface brief # Confirm the affected transit interface is operational.
show clns interface <interface> # Confirm IS-IS is participating on the interface and using the expected circuit type.
show isis neighbors # Confirm which expected Level 2 adjacencies are present or missing.


# Interface is up/up but the IS-IS adjacency is missing.
show clns interface <interface> # Confirm IS-IS is enabled on the operational interface.
show clns protocol # Confirm the IS-IS process has the expected NET and Level 2-only configuration.
show running-config interface <interface> # Confirm the interface has the expected IS-IS configuration.


# Remote IS-IS route is missing.
show isis neighbors # Confirm the required IS-IS adjacencies are established.
show isis database # Confirm the originating router and expected routing information are present in the LSDB.
show isis topology # Confirm SPF calculated a path toward the destination.
show ip route isis # Confirm the expected IS-IS route is installed in the IPv4 routing table.


# Loopback prefix is missing while IS-IS adjacencies remain healthy.
show running-config interface Loopback0 # Confirm Loopback0 is enabled for IS-IS participation.
show isis database # Confirm the loopback prefix is present in the originating router's LSP.
show ip route <loopback-prefix> # Confirm the remote router installed the advertised loopback route.


# Multiple IS-IS adjacencies are missing.
show clns protocol # Confirm a valid NET is configured and the router is operating at the intended IS-IS level.
show isis neighbors # Determine whether all or only specific adjacencies are affected.
show running-config | section router isis # Confirm the NET, Level 2-only operation, metric style, and passive-interface configuration.


# Connectivity fails after a topology change.
show isis neighbors # Confirm the remaining IS-IS adjacencies after the failure.
show isis topology # Confirm SPF calculated an alternate path.
show ip route <destination> # Confirm the destination is installed using the expected next hop.
ping <destination> source Loopback0 # Confirm end-to-end reachability after IS-IS reconvergence.
traceroute <destination> source Loopback0 # Confirm traffic is using the expected alternate forwarding path.
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
