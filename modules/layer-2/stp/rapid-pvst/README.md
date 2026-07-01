# Lab Guide — Rapid PVST+

## Overview

This lab demonstrates how Rapid PVST+ prevents Layer 2 loops and provides faster convergence than traditional STP.

Rapid PVST+ is Cisco’s per-VLAN implementation of RSTP. It improves convergence by using port roles, port states, and the proposal/agreement process to transition eligible ports to forwarding more quickly than legacy STP.

This lab validates root bridge election, port roles, port states, manual root bridge priority changes, and convergence behavior after a simulated link failure.

**Lab Status:** Validated

**End-to-End Verification:** Successful

---

## Objectives

* [x] Enable Rapid PVST+ on all switches.
* [x] Configure VLAN 10 across the Layer 2 topology.
* [x] Identify the initial Rapid PVST+ root bridge for VLAN 10.
* [x] Verify initial Rapid PVST+ port roles and states.
* [x] Change bridge priority to force a new root bridge election.
* [x] Verify updated port roles and states after the root bridge change.
* [x] Simulate a link failure.
* [x] Verify Rapid PVST+ convergence after the failure.


---

## Topology

The topology uses three Layer 2 switches connected in a triangle. This intentionally creates redundant Layer 2 paths so Rapid PVST+ can elect a root bridge, assign port roles, block the redundant path, and reconverge after a simulated link failure.

Topology artifacts are available in the `topology/` folder.
 
- [Open topology diagram](topology/diagram.svg)
- [Open CML topology export](topology/topology.yaml)

**Note:** This topology intentionally creates a Layer 2 loop. Rapid PVST+ must be enabled and functioning before all links are allowed to forward. Running this topology without STP protection could cause a broadcast storm.

---

## Link Tables

### Physical Links

| Local Device | Local Interface | Peer Device | Peer Interface | Description                  |
| ------------ | --------------- | ----------- | -------------- | ---------------------------- |
| S1           | G0/0            | S2          | G0/0           | Trunk link between S1 and S2 |
| S1           | G0/1            | S3          | G0/0           | Trunk link between S1 and S3 |
| S2           | G0/1            | S3          | G0/1           | Trunk link between S2 and S3 |



## Rapid PVST+ States and Roles

**Note:** Cisco IOS output may display a Rapid PVST+ discarding port as `BLK`, even though RSTP does not use the legacy STP blocking/listening model in the same way.

### Port States

| State      | Description                                                                                                                      |
| ---------- | -------------------------------------------------------------------------------------------------------------------------------- |
| Discarding | The port does not forward user traffic or learn MAC addresses. This state prevents loops while the topology is being calculated. |
| Learning   | The port learns MAC addresses but does not forward user traffic yet.                                                             |
| Forwarding | The port forwards user traffic and learns MAC addresses.                                                                         |

### Port Roles

| Role            | Description                                                                                                                 |
| --------------- | --------------------------------------------------------------------------------------------------------------------------- |
| Root Port       | The switch port with the best path toward the root bridge. Non-root switches have one root port per spanning-tree instance. |
| Designated Port | The forwarding port for a network segment. This port advertises the best BPDU on that segment.                              |
| Alternate Port  | A backup path toward the root bridge. This port remains in a non-forwarding state unless the active path fails.             |
| Disabled Port   | A port that is administratively shut down or otherwise not participating in spanning tree.                                  |


---

## Configuration Steps

### Step 1 — Initial Rapid PVST+ Configuration

**Note:** The CLI examples below are annotated for readability. Clean device configurations are available in [`configs/`](configs/).

**Design note:** S1 is configured with the lowest bridge priority for VLAN 10 in the initial state, making it the root bridge before the later root bridge change.

**S1**

```bash
# S1 Configuration Block
enable # Enters privileged EXEC mode.
configure terminal # Enters global configuration mode.
hostname S1 # Sets hostname to S1.

spanning-tree mode rapid-pvst # Enables Rapid PVST+.

vlan 10 # Creates VLAN 10.
name USERS_10 # Sets VLAN 10 name.

interface gigabitEthernet0/0 # Targets interface Gi0/0.
switchport trunk encapsulation dot1q # Sets 802.1Q encapsulation.
switchport mode trunk # Sets interface to trunk mode.
switchport trunk allowed vlan 10 # Allows VLAN 10 on the trunk.
switchport trunk native vlan 1 # Sets native VLAN to 1.

interface gigabitEthernet0/1 # Targets interface Gi0/1.
switchport trunk encapsulation dot1q # Sets 802.1Q encapsulation.
switchport mode trunk # Sets interface to trunk mode.
switchport trunk allowed vlan 10 # Allows VLAN 10 on the trunk.
switchport trunk native vlan 1 # Sets native VLAN to 1.

spanning-tree vlan 10 priority 4096 # Sets S1 as the initial root bridge for VLAN 10.

end # Returns to privileged EXEC mode.
write memory # Saves the running configuration.
```


**S2**

```bash
# S2 Configuration Block
enable # Enters privileged EXEC mode.
configure terminal # Enters global configuration mode.
hostname S2 # Sets hostname to S2.

spanning-tree mode rapid-pvst # Enables Rapid PVST+.

vlan 10 # Creates VLAN 10.
name USERS_10 # Sets VLAN 10 name.

interface gigabitEthernet0/0 # Targets interface Gi0/0.
switchport trunk encapsulation dot1q # Sets 802.1Q encapsulation.
switchport mode trunk # Sets interface to trunk mode.
switchport trunk allowed vlan 10 # Allows VLAN 10 on the trunk.
switchport trunk native vlan 1 # Sets native VLAN to 1.

interface gigabitEthernet0/1 # Targets interface Gi0/1.
switchport trunk encapsulation dot1q # Sets 802.1Q encapsulation.
switchport mode trunk # Sets interface to trunk mode.
switchport trunk allowed vlan 10 # Allows VLAN 10 on the trunk.
switchport trunk native vlan 1 # Sets native VLAN to 1.

end # Returns to privileged EXEC mode.
write memory # Saves the running configuration.
```


**S3**

```bash
# S3 Configuration Block
enable # Enters privileged EXEC mode.
configure terminal # Enters global configuration mode.
hostname S3 # Sets hostname to S3.

spanning-tree mode rapid-pvst # Enables Rapid PVST+.

vlan 10 # Creates VLAN 10.
name USERS_10 # Sets VLAN 10 name.

interface gigabitEthernet0/0 # Targets interface Gi0/0.
switchport trunk encapsulation dot1q # Sets 802.1Q encapsulation.
switchport mode trunk # Sets interface to trunk mode.
switchport trunk allowed vlan 10 # Allows VLAN 10 on the trunk.
switchport trunk native vlan 1 # Sets native VLAN to 1.

interface gigabitEthernet0/1 # Targets interface Gi0/1.
switchport trunk encapsulation dot1q # Sets 802.1Q encapsulation.
switchport mode trunk # Sets interface to trunk mode.
switchport trunk allowed vlan 10 # Allows VLAN 10 on the trunk.
switchport trunk native vlan 1 # Sets native VLAN to 1.

end # Returns to privileged EXEC mode.
write memory # Saves the running configuration.
```

---

## Verification

See [`verification/verification_commands.md`](verification/verification_commands.md) for recorded command output.

### Step 1 — Initial Rapid PVST+ Verification

**S1**

```bash
# S1 Verification Block
show spanning-tree vlan 10 # Confirm S1 is the root bridge and both trunk ports are designated and forwarding.
```

**S2**

```bash
# S2 Verification Block
show spanning-tree vlan 10 # Confirm S2 has one root port and one designated or alternate port.
```

**S3**

```bash
# S3 Verification Block
show spanning-tree vlan 10 # Confirm S3 has one root port and one designated or alternate port.
```

**Note:** S2 and S3 form the non-root side of the triangle. One switch should have the forwarding designated port on the S2-S3 link, and the other should have the alternate non-forwarding port.


---

### Step 2 — Root Bridge Change

**Design note:** S2 is configured with a lower bridge priority than S1 for VLAN 10, forcing a new root bridge election.

**S2**

```bash
# S2 Configuration Block — New Root Bridge
enable # Enters privileged EXEC mode.
configure terminal # Enters global configuration mode.

spanning-tree vlan 10 priority 0 # Sets S2 as the root bridge for VLAN 10.

end # Returns to privileged EXEC mode.
write memory # Saves the running configuration.
```

---

### Step 2 — Root Bridge Change Verification

**S2**

```bash
# S2 Verification Block
show spanning-tree vlan 10 # Confirm S2 is the new root bridge and both trunk ports are designated and forwarding.
```

**S1**

```bash
# S1 Verification Block
show spanning-tree vlan 10 # Confirm S1 now has one root port and one designated or alternate port.
```

**S3**

```bash
# S3 Verification Block
show spanning-tree vlan 10 # Confirm S3 has one root port and one designated or alternate port.
```

**Note:** After S2 becomes the root bridge, S1 and S3 form the non-root side of the triangle. One side of the S1-S3 link should remain forwarding, and the redundant side should remain non-forwarding.


---

### Step 3 — Simulated Link Failure

**Design note:** Use the current `show spanning-tree vlan 10` output to identify an active forwarding path. Shut down one forwarding interface to simulate a link failure and validate Rapid PVST+ convergence.

**All Switches**

```bash
# Identify current Rapid PVST+ port roles and states.
show spanning-tree vlan 10 # Run on all switches before selecting the test interface.
```

**Selected Switch**

```bash
# Simulated Link Failure
enable # Enters privileged EXEC mode.
configure terminal # Enters global configuration mode.

interface gigabitEthernet0/x # Targets the selected forwarding interface.
shutdown # Simulates a link failure.

end # Returns to privileged EXEC mode.
write memory # Saves the running configuration.
```

---

### Step 3 — Link Failure Convergence Verification

**All Switches**

```bash
# Link Failure Verification Block
show spanning-tree vlan 10 # Confirm Rapid PVST+ reconverged and the alternate path moved to forwarding where expected.
show interfaces status # Confirm the intentionally shut interface is down and no unexpected interfaces are err-disabled.
```

**Note:** The expected result is that Rapid PVST+ reconverges after the simulated failure and a previously non-forwarding alternate path transitions to forwarding if it becomes the best available path.


---

## Troubleshooting

**Note:** These are quick-reference checks for this lab. They are not intended to be an exhaustive Rapid PVST+ troubleshooting guide. After any change, re-run the verification steps to confirm the expected behavior.

```bash
# Root bridge is not the expected switch.
show spanning-tree vlan 10 # Confirm the current root bridge and local bridge role.
show spanning-tree vlan 10 root # Confirm root bridge priority, root cost, and root port.
show running-config | section spanning-tree # Confirm Rapid PVST+ mode and manual bridge priority settings.

# Unexpected port roles or states.
show spanning-tree vlan 10 # Confirm current port roles and states.
show spanning-tree vlan 10 detail # Review port role, port state, timers, and BPDU activity.
show interfaces status # Confirm interfaces are connected and not err-disabled.
show interfaces trunk # Confirm trunk links are active and VLAN 10 is allowed.

# VLAN 10 is not participating as expected.
show vlan brief # Confirm VLAN 10 exists on all switches.
show interfaces trunk # Confirm VLAN 10 is allowed and forwarding on trunks.
show running-config interface gigabitEthernet0/x # Confirm trunk configuration on the target interface.

# No convergence after simulated link failure.
show spanning-tree vlan 10 # Confirm an alternate path exists and transitioned as expected.
show spanning-tree vlan 10 detail # Confirm BPDU activity and topology changes.
show interfaces status # Confirm only the intentionally shut interface is down.
show interfaces trunk # Confirm remaining trunk links are still active.

# No BPDUs detected on a port.
show spanning-tree vlan 10 interface gigabitEthernet0/x detail # Confirm BPDU send/receive activity.
show interfaces gigabitEthernet0/x switchport # Confirm the interface is operating as a switchport trunk.
show running-config interface gigabitEthernet0/x # Confirm the interface is not shut down and BPDU filtering is not configured.

# Rapid PVST+ is not operating as expected.
show spanning-tree summary # Confirm spanning-tree mode is rapid-pvst.
show running-config | section spanning-tree # Confirm rapid-pvst is configured on all switches.
show spanning-tree vlan 10 # Confirm ports are not stuck in unexpected states.
show spanning-tree vlan 10 interface gigabitEthernet0/x detail # Confirm point-to-point link behavior where expected.
```

---

## Artifacts

| Type           | Location                                                                         |
| -------------- | -------------------------------------------------------------------------------- |
| Configurations | [`configs/`](configs/)                                                           |
| Diagram        | [`topology/diagram.svg`](topology/diagram.svg)                                   |
| Topology File  | [`topology/topology.yaml`](topology/topology.yaml)                               |
| Verification   | [`verification/verification_commands.md`](verification/verification_commands.md) |


---

## Document Metadata

| Field        | Value         |
| ------------ | ------------- |
| Lab Version  | 1.0           |
| Last Updated | 2026-06-28    |
| Author       | Aaron Kindelt |