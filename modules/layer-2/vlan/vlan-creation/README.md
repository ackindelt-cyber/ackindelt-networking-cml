# Lab Guide — VLAN Creation

## Overview

This lab demonstrates how to configure and verify basic VLAN segmentation on a Layer 2 switch.

VLANs allow a switch to separate hosts into different Layer 2 broadcast domains. In this lab, two clients are placed in VLAN 10 and two clients are placed in VLAN 20. Hosts in the same VLAN can communicate, while hosts in different VLANs cannot communicate because no Layer 3 routing is configured between the VLANs.

This lab validates VLAN creation, VLAN naming, access-port assignment, PortFast and BPDU Guard configuration, intra-VLAN connectivity, and inter-VLAN isolation.

**Lab Status:** Validated

**End-to-End Verification:** Successful

---

**CML VLAN Export Note:** Cisco CML topology exports and exported device configuration files may not preserve VLAN database state. VLAN IDs, VLAN names, and intended VLAN design are documented in this README and should not be inferred from "topology.yaml" or exported configs alone.

---

## Objectives

* [x] Create and name VLAN 10 and VLAN 20.
* [x] Assign access ports to the correct VLANs.
* [x] Enable PortFast and BPDU Guard on client-facing access ports.
* [x] Verify VLAN membership and port assignments.
* [x] Verify MAC address learning by VLAN.
* [x] Test intra-VLAN connectivity.
* [x] Verify isolation between VLANs when no Layer 3 routing is configured.
* [x] Capture selected ICMP traffic for packet-level validation.

---

## Topology

The topology uses one Layer 2 switch and four clients. C1 and C2 are assigned to VLAN 10. C3 and C4 are assigned to VLAN 20.

Topology artifacts are available in the `topology/` folder.
 
- [Open topology diagram](topology/diagram.svg)
- [Open CML topology export](topology/topology.yaml)

---

## Addressing Tables

### Switch Port Assignments

| Device | Interface | Mode   | Connected To | VLAN | Description    |
| ------ | --------- | ------ | ------------ | ---- | -------------- |
| S1     | Gi0/0     | Access | C1 eth0      | 10   | VLAN 10 client |
| S1     | Gi0/1     | Access | C2 eth0      | 10   | VLAN 10 client |
| S1     | Gi0/2     | Access | C3 eth0      | 20   | VLAN 20 client |
| S1     | Gi0/3     | Access | C4 eth0      | 20   | VLAN 20 client |

### Client Addressing

| Client | Interface | IP Address       | VLAN |
| ------ | --------- | ---------------- | ---- |
| C1     | eth0      | 192.168.10.10/24 | 10   |
| C2     | eth0      | 192.168.10.11/24 | 10   |
| C3     | eth0      | 192.168.20.10/24 | 20   |
| C4     | eth0      | 192.168.20.11/24 | 20   |

### VLANs

| VLAN | Name     | Subnet          |
| ---- | -------- | --------------- |
| 10   | USERS_10 | 192.168.10.0/24 |
| 20   | USERS_20 | 192.168.20.0/24 |

---

## Configuration Steps

### Step 1 — VLAN and Access Port Configuration

**Note:** The CLI examples below are annotated for readability. Clean device configurations are available in [`configs/`](configs/).

**S1**

```bash
# S1 Configuration Block
enable # Enters privileged EXEC mode.
configure terminal # Enters global configuration mode.
hostname S1 # Sets hostname to S1.

vlan 10 # Creates VLAN 10.
name USERS_10 # Names VLAN 10.

vlan 20 # Creates VLAN 20.
name USERS_20 # Names VLAN 20.

interface range gigabitEthernet0/0-1 # Selects access ports for VLAN 10 clients.
switchport mode access # Sets interfaces to access mode.
switchport access vlan 10 # Assigns interfaces to VLAN 10.
spanning-tree portfast edge # Enables PortFast edge behavior on client-facing ports.
spanning-tree bpduguard enable # Protects access ports from unexpected BPDUs.

interface range gigabitEthernet0/2-3 # Selects access ports for VLAN 20 clients.
switchport mode access # Sets interfaces to access mode.
switchport access vlan 20 # Assigns interfaces to VLAN 20.
spanning-tree portfast edge # Enables PortFast edge behavior on client-facing ports.
spanning-tree bpduguard enable # Protects access ports from unexpected BPDUs.

end # Returns to privileged EXEC mode.
write memory # Saves the running configuration.
```

---

## Verification

See [`verification/verification_commands.md`](verification/verification_commands.md) for recorded command output.

### S1 Verification

```bash
# S1 Verification Block
show vlan brief # Confirm VLAN 10 and VLAN 20 exist and ports are assigned correctly.
show spanning-tree summary # Confirm VLANs are active and forwarding.
show running-config | section interface # Confirm access mode, PortFast, and BPDU Guard on active access ports.
show mac address-table # Confirm client MAC addresses are learned on the expected VLANs and interfaces.
```

### Client Verification

```bash
# C1 Verification Block
ping 192.168.10.11 # Confirm same-VLAN connectivity from C1 to C2.
ping 192.168.20.10 # Confirm VLAN 10 cannot reach VLAN 20 without routing.
ping 192.168.20.11 # Confirm VLAN 10 cannot reach VLAN 20 without routing.

# C3 Verification Block
ping 192.168.20.11 # Confirm same-VLAN connectivity from C3 to C4.
ping 192.168.10.10 # Confirm VLAN 20 cannot reach VLAN 10 without routing.
ping 192.168.10.11 # Confirm VLAN 20 cannot reach VLAN 10 without routing.
```

### Packet Captures

The following packet captures provide additional evidence for selected ICMP flows:

| Capture                                                          | Description                                           |
| ---------------------------------------------------------------- | ----------------------------------------------------- |
| [`C1_to_C2_ping.pcap`](verification/captures/C1_to_C2_ping.pcap) | ICMP traffic from C1 to C2 within VLAN 10             |
| [`C2_to_C3_ping.pcap`](verification/captures/C2_to_C3_ping.pcap) | ICMP test between VLAN 10 and VLAN 20 without routing |

---

## Troubleshooting

**Note:** These are quick-reference checks for this lab. They are not intended to be an exhaustive VLAN troubleshooting guide. After any change, re-run the verification steps to confirm the expected behavior.

```bash
# VLANs are missing.
show vlan brief # Confirm VLANs exist and are active.
show running-config | section vlan # Confirm VLAN creation and naming.

# Access ports are in the wrong VLAN.
show vlan brief # Confirm access-port VLAN membership.
show interfaces gigabitEthernet0/x switchport # Confirm operational access mode and access VLAN.
show running-config interface gigabitEthernet0/x # Confirm interface configuration.

# Same-VLAN hosts cannot communicate.
show interfaces status # Confirm client-facing interfaces are connected.
show mac address-table dynamic # Confirm client MAC addresses are learned.
show mac address-table interface gigabitEthernet0/x # Confirm MAC learning on the expected interface.
show vlan brief # Confirm both hosts are assigned to the same VLAN.

# Different VLANs can communicate unexpectedly.
show ip interface brief # Confirm no Layer 3 SVI or routed interface is providing routing.
show running-config | include ip routing # Confirm IP routing is not enabled.
show running-config | section interface Vlan # Check for unexpected SVI configuration.

# Access port is blocking or err-disabled.
show spanning-tree interface gigabitEthernet0/x detail # Confirm STP state, PortFast, and BPDU Guard behavior.
show interfaces status err-disabled # List err-disabled interfaces.
show logging # Confirm reason for err-disable or link-state changes.
show cdp neighbors # Confirm no unexpected infrastructure device is connected to an access port.
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
