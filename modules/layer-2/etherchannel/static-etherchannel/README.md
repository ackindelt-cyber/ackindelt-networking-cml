# Lab Guide — Static EtherChannel

## Overview
This lab demonstrates how to configure and verify static EtherChannel using mode on between Layer 2 switches.

Static EtherChannel logically bundles multiple physical links into a single Port-Channel, increasing available bandwidth and allowing Spanning Tree Protocol to treat the bundle as one logical link.

Static EtherChannel does not use a negotiation protocol. If the bundle is misconfigured, the switches may not detect the mismatch before traffic is affected. This lab uses static EtherChannel for technology validation; LACP is generally preferred for production-aligned designs.

**Lab Status:** Validated

**End-to-End Verification:** Successful

---

## Objectives

- [x] Configure static EtherChannel using `mode on` between all switch pairs.
- [x] Verify Port-Channel formation and bundled member interface state.
- [x] Validate STP behavior across the aggregated links.

---

## Topology

The topology uses three Layer 2 switches with static EtherChannel bundles between each switch pair.

Topology artifacts are available in the `topology/` folder.
 
- [Open topology diagram](topology/diagram.svg)
- [Open CML topology export](topology/topology.yaml)

---

## Link Tables

### Physical Links

| Local Device | Local Interface | Peer Device | Peer Interface | Description         |
| ------------ | --------------- | ----------- | -------------- | ------------------- |
| S1           | G0/0            | S2          | G0/0           | Member link for Po1 |
| S1           | G0/1            | S2          | G0/1           | Member link for Po1 |
| S1           | G0/2            | S3          | G0/0           | Member link for Po2 |
| S1           | G0/3            | S3          | G0/1           | Member link for Po2 |
| S2           | G0/2            | S3          | G0/2           | Member link for Po3 |
| S2           | G0/3            | S3          | G0/3           | Member link for Po3 |

### Logical Links

| Port-Channel | Devices  | Member Interfaces          | Purpose                                     |
| ------------ | -------- | -------------------------- | ------------------------------------------- |
| Po1          | S1 to S2 | S1 G0/0-G0/1, S2 G0/0-G0/1 | Static EtherChannel trunk between S1 and S2 |
| Po2          | S1 to S3 | S1 G0/2-G0/3, S3 G0/0-G0/1 | Static EtherChannel trunk between S1 and S3 |
| Po3          | S2 to S3 | S2 G0/2-G0/3, S3 G0/2-G0/3 | Static EtherChannel trunk between S2 and S3 |


---

## Configuration Steps

**Note:** The CLI examples below are annotated for readability. Clean device configurations are available in [`configs/`](configs/).

**Static EtherChannel warning:** Static EtherChannel does not use a negotiation protocol. Bundle mismatches may not be detected before traffic is affected. LACP is generally preferred for production-aligned designs.


**S1**

```bash
# S1 Configuration Block
enable # Enters privileged EXEC mode.
configure terminal # Enters global configuration mode.
hostname S1 # Sets hostname to S1.

interface range gigabitEthernet0/0-1 # Sets target interface range.
switchport trunk encapsulation dot1q # Sets 802.1Q encapsulation on the target interfaces.
switchport mode trunk # Sets target interfaces to trunk mode.
channel-group 1 mode on # Bundles target interfaces into Po1.
interface port-channel 1 # Targets Port-Channel 1.
switchport mode trunk # Sets Port-Channel 1 to trunk mode.

interface range gigabitEthernet0/2-3 # Sets target interface range.
switchport trunk encapsulation dot1q # Sets 802.1Q encapsulation on the target interfaces.
switchport mode trunk # Sets target interfaces to trunk mode.
channel-group 2 mode on # Bundles target interfaces into Po2.
interface port-channel 2 # Targets Port-Channel 2.
switchport mode trunk # Sets Port-Channel 2 to trunk mode.

end # Returns to privileged EXEC mode.
write memory # Saves the running configuration.
```


**S2**

```bash
# S2 Configuration Block
enable # Enters privileged EXEC mode.
configure terminal # Enters global configuration mode.
hostname S2 # Sets hostname to S2.

interface range gigabitEthernet0/0-1 # Sets target interface range.
switchport trunk encapsulation dot1q # Sets 802.1Q encapsulation on the target interfaces.
switchport mode trunk # Sets target interfaces to trunk mode.
channel-group 1 mode on # Bundles target interfaces into Po1.
interface port-channel 1 # Targets Port-Channel 1.
switchport mode trunk # Sets Port-Channel 1 to trunk mode.

interface range gigabitEthernet0/2-3 # Sets target interface range.
switchport trunk encapsulation dot1q # Sets 802.1Q encapsulation on the target interfaces.
switchport mode trunk # Sets target interfaces to trunk mode.
channel-group 3 mode on # Bundles target interfaces into Po3.
interface port-channel 3 # Targets Port-Channel 3.
switchport mode trunk # Sets Port-Channel 3 to trunk mode.

end # Returns to privileged EXEC mode.
write memory # Saves the running configuration.
```


**S3**

```bash
# S3 Configuration Block
enable # Enters privileged EXEC mode.
configure terminal # Enters global configuration mode.
hostname S3 # Sets hostname to S3.

interface range gigabitEthernet0/0-1 # Sets target interface range.
switchport trunk encapsulation dot1q # Sets 802.1Q encapsulation on the target interfaces.
switchport mode trunk # Sets target interfaces to trunk mode.
channel-group 2 mode on # Bundles target interfaces into Po2.
interface port-channel 2 # Targets Port-Channel 2.
switchport mode trunk # Sets Port-Channel 2 to trunk mode.

interface range gigabitEthernet0/2-3 # Sets target interface range.
switchport trunk encapsulation dot1q # Sets 802.1Q encapsulation on the target interfaces.
switchport mode trunk # Sets target interfaces to trunk mode.
channel-group 3 mode on # Bundles target interfaces into Po3.
interface port-channel 3 # Targets Port-Channel 3.
switchport mode trunk # Sets Port-Channel 3 to trunk mode.

end # Returns to privileged EXEC mode.
write memory # Saves the running configuration.
```


---

## Verification

See [`verification/verification_commands.md`](verification/verification_commands.md) for recorded command output.

**S1**

```bash
# S1 Verification Block
show etherchannel summary # Confirm both Port-Channels formed and member interfaces are bundled.
show interfaces port-channel 1 # Confirm Po1 is up, operating as a Layer 2 switchport, and forwarding.
show interfaces port-channel 2 # Confirm Po2 is up, operating as a Layer 2 switchport, and forwarding.
show spanning-tree # Confirm STP recognizes the Port-Channels as logical links.
show interfaces gigabitEthernet0/0 etherchannel # Confirm Gi0/0 is active and mapped to Po1.
show interfaces gigabitEthernet0/2 etherchannel # Confirm Gi0/2 is active and mapped to Po2.
```

**S2**

```bash
# S2 Verification Block
show etherchannel summary # Confirm both Port-Channels formed and member interfaces are bundled.
show interfaces port-channel 1 # Confirm Po1 is up, operating as a Layer 2 switchport, and forwarding.
show interfaces port-channel 3 # Confirm Po3 is up, operating as a Layer 2 switchport, and forwarding.
show spanning-tree # Confirm STP recognizes the Port-Channels as logical links.
show interfaces gigabitEthernet0/0 etherchannel # Confirm Gi0/0 is active and mapped to Po1.
show interfaces gigabitEthernet0/2 etherchannel # Confirm Gi0/2 is active and mapped to Po3.
```

**S3**

```bash
# S3 Verification Block
show etherchannel summary # Confirm both Port-Channels formed and member interfaces are bundled.
show interfaces port-channel 2 # Confirm Po2 is up, operating as a Layer 2 switchport, and forwarding.
show interfaces port-channel 3 # Confirm Po3 is up, operating as a Layer 2 switchport, and forwarding.
show spanning-tree # Confirm STP recognizes the Port-Channels as logical links.
show interfaces gigabitEthernet0/0 etherchannel # Confirm Gi0/0 is active and mapped to Po2.
show interfaces gigabitEthernet0/2 etherchannel # Confirm Gi0/2 is active and mapped to Po3.
```
---

## Troubleshooting

**Note:** These are quick-reference checks for this lab. They are not intended to be an exhaustive EtherChannel troubleshooting guide. After any change, re-run the verification steps to confirm the expected behavior.

```bash
# Port-Channel does not form or member links are not bundled.
show etherchannel summary # Confirm the Port-Channel formed and member links show as bundled.
show interfaces gigabitEthernet0/x etherchannel # Confirm the physical interface is mapped to the expected Port-Channel.

# Physical interfaces are down.
show interfaces status # Confirm physical link status.
show interfaces gigabitEthernet0/x # Confirm the target interface is up/up.

# Traffic does not pass across the Port-Channel.
show interfaces port-channel x # Confirm Port-Channel state, errors, and forwarding status.
show interfaces port-channel x switchport # Confirm trunk configuration on the Port-Channel.
show interfaces trunk # Confirm interfaces are operating in trunk mode.

# STP blocks the Port-Channel.
show spanning-tree # Confirm STP role and state.
show interfaces port-channel x # Confirm Port-Channel state, errors, and forwarding status.
show etherchannel summary # Confirm the Port-Channel formed and member links show as bundled.

# Physical interface shows suspended or standalone.
show interfaces gigabitEthernet0/x etherchannel # Check why the interface is not bundled.
```


---


## Artifacts

| Type           | Location                                                                         |
| -------------- | -------------------------------------------------------------------------------- |
| Configurations | [`configs/`](configs/)                                                           |
| Diagram        | [`topology/diagram.svg`](topology/diagram.svg)                         |
| Topology File  | [`topology/topology.yaml`](topology/topology.yaml)                               |
| Verification   | [`verification/verification_commands.md`](verification/verification_commands.md) |



---

## Document Metadata

| Field        | Value         |
| ------------ | ------------- |
| Lab Version  | 1.0           |
| Last Updated | 2026-06-27    |
| Author       | Aaron Kindelt |
