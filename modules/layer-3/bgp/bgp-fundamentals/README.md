# Lab Guide — BGP Fundamentals

## Overview

This lab demonstrates the fundamentals of Border Gateway Protocol (BGP) in a multi-autonomous-system environment. The topology uses two Talos Solutions edge routers in the same autonomous system, two simulated service providers, and an external service network to demonstrate both external BGP (eBGP) and internal BGP (iBGP).

The lab validates BGP neighbor formation, prefix advertisement and learning, BGP path attributes, best-path selection, and installation of BGP-learned routes into the routing table.

Lab Status: In Progress

End-to-End Verification: Not Tested

---

## Objectives

* [ ] Configure routed interfaces and IP addressing across all inter-router links.

* [ ] Establish eBGP peer relationships between separate autonomous systems and iBGP between the Talos Solutions edge routers.

* [ ] Advertise and learn BGP prefixes across the topology.

* [ ] Verify BGP neighbor state, learned routes, path attributes, and best-path selection.

* [ ] Validate that the selected BGP routes are installed in the routing table and provide end-to-end reachability.

---

## Topology

This lab uses five routers across four autonomous systems. Talos Solutions operates two edge routers in AS 65001. Each Talos edge router connects to a separate simulated service provider using eBGP, while the two Talos edge routers peer with each other using iBGP. Both service providers connect to an external service router in AS 65030, creating multiple BGP paths to the same external network.

![Topology Diagram](topology/bgp-fundamentals-topology.png)

---

## Link Tables

### Physical Links

| Local Device | Local Interface | Local IP / Prefix | Peer Device | Peer Interface | Peer IP / Prefix | Description                         |
| ------------ | --------------- | ----------------- | ----------- | -------------- | ---------------- | ----------------------------------- |
| TSE1         | G0/0            | 10.0.10.1 /30     | ISPA        | G0/0           | 10.0.10.2 /30    | Talos Solutions to ISP-A            |
| TSE2         | G0/0            | 10.0.20.1 /30     | ISPB        | G0/0           | 10.0.20.2 /30    | Talos Solutions to ISP-B            |
| ISPA         | G0/1            | 10.0.30.1 /30     | EXS1        | G0/0           | 10.0.30.2 /30    | ISP-A to external service           |
| ISPB         | G0/1            | 10.0.40.1 /30     | EXS1        | G0/1           | 10.0.40.2 /30    | ISP-B to external service           |
| TSE1         | G0/1            | 10.0.50.1 /30     | TSE2        | G0/1           | 10.0.50.2 /30    | Talos Solutions internal transit    |

### Autonomous Systems

| Routing Domain   | Device(s)  | ASN   |
| ---------------- | ---------- | ----- |
| Talos Solutions  | TSE1, TSE2 | 65001 |
| ISP-A            | ISPA       | 65010 |
| ISP-B            | ISPB       | 65020 |
| External Service | EXS1       | 65030 |

### BGP Peer Relationships

| Local Device | Local AS | Peer Device | Peer AS | Peer Type | Network        |
| ------------ | -------- | ----------- | ------- | --------- | -------------- |
| TSE1         | 65001    | ISPA        | 65010   | eBGP      | 10.0.10.0 /30  |
| TSE2         | 65001    | ISPB        | 65020   | eBGP      | 10.0.20.0 /30  |
| ISPA         | 65010    | EXS1        | 65030   | eBGP      | 10.0.30.0 /30  |
| ISPB         | 65020    | EXS1        | 65030   | eBGP      | 10.0.40.0 /30  |
| TSE1         | 65001    | TSE2        | 65001   | iBGP      | 10.0.50.0 /30  |

### BGP Test Prefixes

| Routing Domain   | Device | Interface | Prefix         | Purpose                      |
| ---------------- | ------ | --------- | -------------- | ---------------------------- |
| Talos Solutions  | TSE1   | Loopback0 | 192.0.2.0 /24  | Talos BGP test prefix        |
| External Service | EXS1   | Loopback0 | 203.0.113.0 /24| External service test prefix |

---

### Pre-Configuration

> **Design note:** BGP requires existing IP connectivity between peers. This section establishes the routed underlay and external test prefix before any BGP processes or neighbor relationships are configured.

**TSE1**

```bash
#TSE1 Preconfig
enable # Enters privileged EXEC mode.
configure terminal # Enters global configuration mode.
hostname TSE1 # Sets the router hostname.
no ip domain-lookup # Prevents mistyped commands from triggering DNS lookups.

interface g0/0 # Selects the routed link toward ISPA.
description Link to ISPA # Documents the connected peer.
ip address 10.0.10.1 255.255.255.252 # Assigns TSE1's address on the 10.0.10.0/30 network.
no shutdown # Enables the interface.

interface g0/1 # Selects the routed link toward TSE2.
description Internal transit link to TSE2 # Documents the connected peer.
ip address 10.0.50.1 255.255.255.252 # Assigns TSE1's address on the 10.0.50.0/30 network.
no shutdown # Enables the interface.

loopback0 # Creates the Talos Solutions test network interface.
description Talos Solutions test prefix # Documents the purpose of the loopback.
ip address 192.0.2.1 255.255.255.0 # Assigns the Talos Solutions BGP test prefix

end # Returns to privileged EXEC mode.
write memory # Saves the running configuration.
```

**TSE2**

```bash
#TSE2 Preconfig
enable # Enters privileged EXEC mode.
configure terminal # Enters global configuration mode.
hostname TSE2 # Sets the router hostname.
no ip domain-lookup # Prevents mistyped commands from triggering DNS lookups.

interface g0/0 # Selects the routed link toward ISPB.
description Link to ISPB # Documents the connected peer.
ip address 10.0.20.1 255.255.255.252 # Assigns TSE2's address on the 10.0.20.0/30 network.
no shutdown # Enables the interface.

interface g0/1 # Selects the routed link toward TSE1.
description Internal transit link to TSE1 # Documents the connected peer.
ip address 10.0.50.2 255.255.255.252 # Assigns TSE2's address on the 10.0.50.0/30 network.
no shutdown # Enables the interface.

end # Returns to privileged EXEC mode.
write memory # Saves the running configuration.
```

**ISPA**

```bash
#ISPA Preconfig
enable # Enters privileged EXEC mode.
configure terminal # Enters global configuration mode.
hostname ISPA # Sets the router hostname.
no ip domain-lookup # Prevents mistyped commands from triggering DNS lookups.

interface g0/0 # Selects the routed link toward TSE1.
description Link to TSE1 # Documents the connected peer.
ip address 10.0.10.2 255.255.255.252 # Assigns ISPA's address on the 10.0.10.0/30 network.
no shutdown # Enables the interface.

interface g0/1 # Selects the routed link toward EXS1.
description Link to EXS1 # Documents the connected peer.
ip address 10.0.30.1 255.255.255.252 # Assigns ISPA's address on the 10.0.30.0/30 network.
no shutdown # Enables the interface.

end # Returns to privileged EXEC mode.
write memory # Saves the running configuration.
```

**ISPB**

```bash
#ISPB Preconfig
enable # Enters privileged EXEC mode.
configure terminal # Enters global configuration mode.
hostname ISPB # Sets the router hostname.
no ip domain-lookup # Prevents mistyped commands from triggering DNS lookups.

interface g0/0 # Selects the routed link toward TSE2.
description Link to TSE2 # Documents the connected peer.
ip address 10.0.20.2 255.255.255.252 # Assigns ISPB's address on the 10.0.20.0/30 network.
no shutdown # Enables the interface.

interface g0/1 # Selects the routed link toward EXS1.
description Link to EXS1 # Documents the connected peer.
ip address 10.0.40.1 255.255.255.252 # Assigns ISPB's address on the 10.0.40.0/30 network.
no shutdown # Enables the interface.

end # Returns to privileged EXEC mode.
write memory # Saves the running configuration.
```

**EXS1**

```bash
#EXS1 Preconfig
enable # Enters privileged EXEC mode.
configure terminal # Enters global configuration mode.
hostname EXS1 # Sets the router hostname.
no ip domain-lookup # Prevents mistyped commands from triggering DNS lookups.

interface g0/0 # Selects the routed link toward ISPA.
description Link to ISPA # Documents the connected peer.
ip address 10.0.30.2 255.255.255.252 # Assigns EXS1's address on the 10.0.30.0/30 network.
no shutdown # Enables the interface.

interface g0/1 # Selects the routed link toward ISPB.
description Link to ISPB # Documents the connected peer.
ip address 10.0.40.2 255.255.255.252 # Assigns EXS1's address on the 10.0.40.0/30 network.
no shutdown # Enables the interface.

interface loopback0 # Creates the external service test interface.
description External service test prefix # Documents the purpose of the loopback.
ip address 203.0.113.1 255.255.255.0 # Assigns the external test prefix that will later be advertised through BGP.

end # Returns to privileged EXEC mode.
write memory # Saves the running configuration.
```

### BGP Configuration

> **Design note:** BGP configuration is added only after direct Layer 3 connectivity between all intended peers has been established and verified.

**TSE1**

```bash
#TSE1 BGP Configuration
router bgp 65001 # Creates/enters the BGP process for local AS65001
neighbor 10.0.10.2 remote-as 65010 #Defines ISPA as a BGP neighbor in remote AS 65010
neighbor 10.0.50.2 remote-as 65001 # Defines TSE2 as a BGP neighbor in the same AS
neighbor 10.0.50.2 next-hop-self # Sets TSE1 as the next hop for BGP routes advertised to TSE2 by TSE1.
network 192.0.2.0 mask 255.255.255.0 # Originates the 192.0.2.0/24 Talos Solutions test prefix

```

**TSE2**

```bash
#TSE2 BGP Configuration
router bgp 65001 # Creates/enters the BGP process for local AS65001
neighbor 10.0.20.2 remote-as 65020 #Defines ISPB as a BGP neighbor in remote AS65020
neighbor 10.0.50.1 remote-as 65001 #Defines TSE1 as a BGP neighbor in the same AS
neighbor 10.0.50.1 next-hop-self #Sets TSE2 as the next hop for BGP routes advertised to TSE1 by TSE2
```

**ISPA**

```bash
router bgp 65010 # Creates/enters the BGP process for local AS65010
neighbor 10.0.10.1 remote-as 65001 #Defines TSE1 as a BGP neighbor in remote AS65001
neighbor 10.0.30.2 remote-as 65030 #Defines EXS1 as a BGP neighbor in remote AS65030

```

**ISPB**

```bash
#ISPB BGP Configuration
router bgp 65020 #Creates/enters the BGP process for local AS65020
neighbor 10.0.20.1 remote-as 65001 # Defines TSE2 as a BGP neighbor in remote AS65001
neighbor 10.0.40.2 remote-as 65030 # Defines EXS1 as a BGP neighbor in remote AS65030
```

**EXS1**

```bash
#EXS1 BGP Configuration
router bgp 65030 #Creates/enters the BGP process for local AS65030
neighbor 10.0.30.1 remote-as 65010 #Defines ISPA as a BGP neighbor in remote AS65010
neighbor 10.0.40.1 remote-as 65020 #Defines ISPB as a BGP neighbor in remote AS65020
network 203.0.113.0 mask 255.255.255.0 # Originates the 203.0.113.0/24 external service prefix into BGP.
```
## Verification

See [`verification/verification_commands.md`](verification/verification_commands.md) for recorded command output.

**TSE1**

```bash
# TSE1 Verification
show ip bgp summary # Verifies BGP neighbors, remote AS numbers, session state, and received prefix counts
show ip bgp # Displays BGP prefixes, available paths, next hops, path attributes, and selected best paths
show ip route 203.0.113.0 # Confirms the selected route to the external service prefix is installed in the routing table
ping 203.0.113.1 source loopback0 # Confirms bidirectional BGP-learned reachability between the Talos Solutions and external service test prefixes
```

**TSE2**

```bash
# TSE2 Verification
show ip bgp summary # Verifies BGP neighbors, remote AS numbers, session state, and received prefix counts
show ip bgp # Displays BGP prefixes, available paths, next hops, path attributes, and selected best paths
show ip route 203.0.113.0 # Confirms the selected route to the external service prefix is installed in the routing table
show ip route 192.0.2.0 # Confirms the Talos Solutions test prefix learned through iBGP is installed in the routing table
```

**ISPA**

```bash
# ISPA Verification
show ip bgp summary # Verifies BGP neighbors, remote AS numbers, session state, and received prefix counts
show ip bgp # Confirms the Talos Solutions and external service prefixes are present in the BGP table
show ip route 203.0.113.0 # Confirms the external service prefix is installed in the routing table
show ip route 192.0.2.0 # Confirms the Talos Solutions test prefix is installed in the routing table
```

**ISPB**

```bash
# ISPB Verification
show ip bgp summary # Verifies BGP neighbors, remote AS numbers, session state, and received prefix counts
show ip bgp # Confirms the Talos Solutions and external service prefixes are present in the BGP table
show ip route 203.0.113.0 # Confirms the external service prefix is installed in the routing table
show ip route 192.0.2.0 # Confirms the Talos Solutions test prefix is installed in the routing table
```

**EXS1**

```bash
# EXS1 Verification
show ip bgp summary # Verifies BGP neighbors, remote AS numbers, session state, and received prefix counts
show ip bgp # Confirms the external service prefix is originated and the Talos Solutions prefix is learned through BGP
show ip route 192.0.2.0 # Confirms the selected BGP route to the Talos Solutions test prefix is installed in the routing table
show ip route 203.0.113.0 # Confirms the external service prefix exists locally as the connected route used for BGP origination
```

---

## Troubleshooting

> **Note:** These are quick-reference checks for this lab. They are not intended to be an exhaustive troubleshooting guide. After any change, re-run the verification steps to confirm the expected behavior.

```bash
# BGP neighbor relationship does not establish.
show ip interface brief # Confirm the local peer-facing interface is up/up and using the expected IP address.
ping <neighbor-ip> # Confirm basic Layer 3 reachability to the intended BGP neighbor.
show ip bgp summary # Confirm the configured neighbor, remote AS, and current BGP session state.
show running-config | section router bgp # Confirm the local AS, neighbor IP, and remote AS configuration.

# Expected prefix is missing from the BGP table.
show ip bgp # Confirm whether the expected prefix was learned or originated through BGP.
show ip bgp summary # Confirm the BGP session to the expected advertising neighbor is established.
show running-config | section router bgp # Confirm the expected neighbor and network statements are configured.
show ip route <prefix> # Confirm a locally originated prefix exists in the routing table with the exact prefix length required by the BGP network statement.

# Prefix is present in BGP but missing from the routing table.
show ip bgp # Confirm the prefix is present and identify the selected best path and next hop.
show ip route <next-hop> # Confirm the BGP next hop is reachable.
show ip route <prefix> # Check whether another route or routing source is installed for the destination.

# Routes appear correct but end-to-end connectivity fails.
show ip route <destination-prefix> # Confirm the router has a forwarding route toward the destination.
show ip route <source-prefix> # Confirm a return route exists toward the source network.
ping <next-hop> # Confirm reachability to the next-hop router before troubleshooting farther downstream.
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
