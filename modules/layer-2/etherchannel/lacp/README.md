# Lab Guide — LACP EtherChannel

## Overview

This lab demonstrates how to configure and verify LACP EtherChannel between Layer 2 switches.

LACP EtherChannel bundles multiple physical links into a single logical port-channel. This provides link redundancy and increased aggregate bandwidth while allowing the switches to negotiate whether member links are suitable for the bundle.

This lab validates LACP negotiation, port-channel formation, bundled member interfaces, port-channel health, and STP behavior across aggregated links.

**Lab Status:** Validated

**End-to-End Verification:** Successful

---

## Objectives

* [x] Configure LACP EtherChannel between all switch pairs.
* [x] Verify LACP negotiation and neighbor relationships.
* [x] Verify EtherChannel formation and port-channel operational state.
* [x] Verify member interfaces are bundled into the correct port-channels.
* [x] Validate STP behavior across aggregated logical links.

---

## Topology

The topology uses three Layer 2 switches connected in a triangle. Each switch pair has two physical links bundled into one LACP EtherChannel.

Topology artifacts are available in the `topology/` folder.
 
- [Open topology diagram](topology/diagram.svg)
- [Open CML topology export](topology/topology.yaml)

**Note:** STP sees each port-channel as a single logical link rather than as separate physical member interfaces. The physical topology has six links, but the logical Layer 2 topology is a triangle of three port-channels.

---

## Link Tables

### Physical Links

| Local Device | Local Interface | Peer Device | Peer Interface | Description                  |
| ------------ | --------------- | ----------- | -------------- | ---------------------------- |
| S1           | G0/0            | S2          | G0/0           | Physical member link for Po1 |
| S1           | G0/1            | S2          | G0/1           | Physical member link for Po1 |
| S1           | G0/2            | S3          | G0/0           | Physical member link for Po2 |
| S1           | G0/3            | S3          | G0/1           | Physical member link for Po2 |
| S2           | G0/2            | S3          | G0/2           | Physical member link for Po3 |
| S2           | G0/3            | S3          | G0/3           | Physical member link for Po3 |

### Logical Links

| Port-Channel | Devices | Member Interfaces           | Protocol / Mode    | Description                         |
| ------------ | ------- | --------------------------- | ------------------ | ----------------------------------- |
| Po1          | S1 ↔ S2 | S1 G0/0-G0/1 ↔ S2 G0/0-G0/1 | LACP active/active | LACP EtherChannel between S1 and S2 |
| Po2          | S1 ↔ S3 | S1 G0/2-G0/3 ↔ S3 G0/0-G0/1 | LACP active/active | LACP EtherChannel between S1 and S3 |
| Po3          | S2 ↔ S3 | S2 G0/2-G0/3 ↔ S3 G0/2-G0/3 | LACP active/active | LACP EtherChannel between S2 and S3 |

---

## LACP and EtherChannel Notes

| Item         | Description                                                                          |
| ------------ | ------------------------------------------------------------------------------------ |
| LACP         | Link Aggregation Control Protocol. Dynamically negotiates EtherChannel formation.    |
| Active Mode  | The interface actively sends LACP packets to form a bundle.                          |
| Passive Mode | The interface responds to LACP packets but does not initiate negotiation.            |
| Port-Channel | The logical interface created from bundled physical member links.                    |
| Member Link  | A physical interface assigned to a channel group.                                    |
| `(P)`        | EtherChannel summary flag showing a member interface is bundled in the port-channel. |
| `(SU)`       | EtherChannel summary flags showing a Layer 2 port-channel that is in use.            |

**Design note:** This lab uses LACP active mode on both sides of each EtherChannel. Active/active is explicit, easy to verify, and avoids relying on only one side to initiate negotiation.

---

## Configuration Steps

### Step 1 — LACP EtherChannel Configuration

**Note:** The CLI examples below are annotated for readability. Clean device configurations are available in [`configs/`](configs/).

**S1**

```bash
# S1 Configuration Block
enable # Enters privileged EXEC mode.
configure terminal # Enters global configuration mode.
hostname S1 # Sets hostname to S1.

interface range gigabitEthernet0/0-1 # Selects member interfaces for Po1.
switchport trunk encapsulation dot1q # Sets 802.1Q encapsulation.
switchport mode trunk # Sets interfaces to trunk mode.
channel-group 1 mode active # Adds interfaces to channel group 1 using LACP active mode.

interface range gigabitEthernet0/2-3 # Selects member interfaces for Po2.
switchport trunk encapsulation dot1q # Sets 802.1Q encapsulation.
switchport mode trunk # Sets interfaces to trunk mode.
channel-group 2 mode active # Adds interfaces to channel group 2 using LACP active mode.

end # Returns to privileged EXEC mode.
write memory # Saves the running configuration.
```

**S2**

```bash
# S2 Configuration Block
enable # Enters privileged EXEC mode.
configure terminal # Enters global configuration mode.
hostname S2 # Sets hostname to S2.

interface range gigabitEthernet0/0-1 # Selects member interfaces for Po1.
switchport trunk encapsulation dot1q # Sets 802.1Q encapsulation.
switchport mode trunk # Sets interfaces to trunk mode.
channel-group 1 mode active # Adds interfaces to channel group 1 using LACP active mode.

interface range gigabitEthernet0/2-3 # Selects member interfaces for Po3.
switchport trunk encapsulation dot1q # Sets 802.1Q encapsulation.
switchport mode trunk # Sets interfaces to trunk mode.
channel-group 3 mode active # Adds interfaces to channel group 3 using LACP active mode.

end # Returns to privileged EXEC mode.
write memory # Saves the running configuration.
```

**S3**

```bash
# S3 Configuration Block
enable # Enters privileged EXEC mode.
configure terminal # Enters global configuration mode.
hostname S3 # Sets hostname to S3.

interface range gigabitEthernet0/0-1 # Selects member interfaces for Po2.
switchport trunk encapsulation dot1q # Sets 802.1Q encapsulation.
switchport mode trunk # Sets interfaces to trunk mode.
channel-group 2 mode active # Adds interfaces to channel group 2 using LACP active mode.

interface range gigabitEthernet0/2-3 # Selects member interfaces for Po3.
switchport trunk encapsulation dot1q # Sets 802.1Q encapsulation.
switchport mode trunk # Sets interfaces to trunk mode.
channel-group 3 mode active # Adds interfaces to channel group 3 using LACP active mode.

end # Returns to privileged EXEC mode.
write memory # Saves the running configuration.
```

---

## Verification

See [`verification/verification_commands.md`](verification/verification_commands.md) for recorded command output.

### Step 1 — LACP EtherChannel Verification

**S1**

```bash
# S1 Verification Block
show etherchannel summary # Confirm Po1 and Po2 are formed and member interfaces are bundled.
show lacp neighbor # Confirm LACP neighbors are present for channel groups 1 and 2.
show interfaces port-channel 1 # Confirm Po1 is up/up and reflects aggregated bandwidth.
show interfaces port-channel 2 # Confirm Po2 is up/up and reflects aggregated bandwidth.
show spanning-tree # Confirm STP sees Po1 and Po2 as logical links.
show interfaces gigabitEthernet0/0 etherchannel # Confirm Gi0/0 is bundled into Po1 using LACP.
show interfaces gigabitEthernet0/2 etherchannel # Confirm Gi0/2 is bundled into Po2 using LACP.
```

**S2**

```bash
# S2 Verification Block
show etherchannel summary # Confirm Po1 and Po3 are formed and member interfaces are bundled.
show lacp neighbor # Confirm LACP neighbors are present for channel groups 1 and 3.
show interfaces port-channel 1 # Confirm Po1 is up/up and reflects aggregated bandwidth.
show interfaces port-channel 3 # Confirm Po3 is up/up and reflects aggregated bandwidth.
show spanning-tree # Confirm STP sees Po1 and Po3 as logical links.
show interfaces gigabitEthernet0/0 etherchannel # Confirm Gi0/0 is bundled into Po1 using LACP.
show interfaces gigabitEthernet0/2 etherchannel # Confirm Gi0/2 is bundled into Po3 using LACP.
```

**S3**

```bash
# S3 Verification Block
show etherchannel summary # Confirm Po2 and Po3 are formed and member interfaces are bundled.
show lacp neighbor # Confirm LACP neighbors are present for channel groups 2 and 3.
show interfaces port-channel 2 # Confirm Po2 is up/up and reflects aggregated bandwidth.
show interfaces port-channel 3 # Confirm Po3 is up/up and reflects aggregated bandwidth.
show spanning-tree # Confirm STP sees Po2 and Po3 as logical links.
show interfaces gigabitEthernet0/0 etherchannel # Confirm Gi0/0 is bundled into Po2 using LACP.
show interfaces gigabitEthernet0/2 etherchannel # Confirm Gi0/2 is bundled into Po3 using LACP.
```

**Note:** In this triangle topology, STP may block one port-channel to prevent a Layer 2 loop. That is expected. The important point is that STP evaluates each EtherChannel as one logical interface.

---

## Troubleshooting

**Note:** These are quick-reference checks for this lab. They are not intended to be an exhaustive EtherChannel troubleshooting guide. After any change, re-run the verification steps to confirm the expected behavior.

```bash
# Port-channel does not form.
show etherchannel summary # Confirm port-channels exist and member links are bundled with the P flag.
show interfaces gigabitEthernet0/x etherchannel # Confirm the target interface state, channel group, mode, and protocol.
show lacp neighbor # Confirm LACP neighbor information is present.

# Member interface is standalone, suspended, or down.
show etherchannel summary # Check for I, s, D, u, or other non-bundled flags.
show interfaces status # Confirm physical link status.
show interfaces gigabitEthernet0/x # Confirm line protocol, errors, and operational state.
show running-config interface gigabitEthernet0/x # Confirm channel-group mode and trunk configuration.

# LACP neighbor is not detected.
show lacp neighbor # Confirm the expected partner is listed.
show interfaces gigabitEthernet0/x etherchannel # Confirm LACP protocol and local/partner information.
show running-config interface gigabitEthernet0/x # Confirm mode active or passive is configured.

# Traffic does not pass across the port-channel.
show interfaces port-channel x # Confirm port-channel state, counters, and errors.
show interfaces port-channel x switchport # Confirm Layer 2 switchport settings on the logical interface.
show interfaces trunk # Confirm trunking state.
show etherchannel summary # Confirm member links are bundled.

# STP blocks a port-channel.
show spanning-tree # Confirm STP role and state for each port-channel.
show interfaces port-channel x # Confirm the port-channel itself is up/up.
show etherchannel summary # Confirm the port-channel is formed and member links are bundled.

# Port-channel settings do not match member interfaces.
show running-config interface port-channel x # Review logical port-channel configuration.
show running-config interface gigabitEthernet0/x # Review physical member interface configuration.
show interfaces trunk # Confirm trunking is consistent.
show etherchannel summary # Confirm all expected members are bundled.
```

---

## Artifacts

| Type           | Location                                                                         |
| -------------- | -------------------------------------------------------------------------------- |
| Configurations | [`configs/`](configs/)                                                           |
| Diagram        | [`topology/diagram.svg`](topology/etherchannel.svg)                                   |
| Topology File  | [`topology/topology.yaml`](topology/topology.yaml)                               |
| Verification   | [`verification/verification_commands.md`](verification/verification_commands.md) |

---

## Document Metadata

| Field        | Value         |
| ------------ | ------------- |
| Lab Version  | 1.0           |
| Last Updated | 2026-06-28    |
| Author       | Aaron Kindelt |

