# Runbook — Rapid-PVST+ Root Bridge Placement

## Overview

This runbook provides a structured process for configuring and validating Rapid-PVST+ root bridge placement.

Rapid-PVST+ provides Layer 2 loop prevention and per-VLAN spanning tree instances. Root bridge placement determines the preferred Layer 2 forwarding topology for each VLAN. In this design, D1 is configured as the primary root bridge and D2 is configured as the secondary root bridge for VLANs 10 and 20.

This runbook focuses only on STP mode, root bridge priority, secondary root placement, and validation of expected root bridge election behavior.

## Scope and Assumptions

This runbook assumes:

* The environment uses Cisco Rapid-PVST+.
* VLANs already exist on the required switches.
* Trunk links are already configured between switches.
* VLANs 10 and 20 are allowed across required trunks.
* D1 is intended to operate as the primary root bridge.
* D2 is intended to operate as the secondary root bridge.
* A1 is an access-layer switch with redundant uplinks to D1 and D2.
* STP root placement is controlled with bridge priority.

This runbook does not cover PortFast, BPDU Guard, Root Guard, MST, EtherChannel, or HSRP/STP alignment.

---

## Reference Design

This runbook uses a three-switch Rapid-PVST+ design with one primary root bridge, one secondary root bridge, and one non-root access switch.

### Devices

| Device | Role                  | Purpose                      |
| ------ | --------------------- | ---------------------------- |
| D1     | Distribution Switch 1 | Primary STP root bridge      |
| D2     | Distribution Switch 2 | Secondary STP root bridge    |
| A1     | Access Switch         | Non-root access-layer switch |

### VLAN Summary

| VLAN | Purpose  | Primary Root | Secondary Root |
| ---- | -------- | ------------ | -------------- |
| 10   | Users    | D1           | D2             |
| 20   | Printers | D1           | D2             |

### Physical Link Summary

| Link | Device A | Interface A | Device B | Interface B | Purpose                         |
| ---- | -------- | ----------- | -------- | ----------- | ------------------------------- |
| 1    | D1       | Gi0/0       | D2       | Gi0/0       | Distribution inter-switch trunk |
| 2    | D1       | Gi0/1       | A1       | Gi0/0       | Access uplink to primary root   |
| 3    | D2       | Gi0/1       | A1       | Gi0/1       | Access uplink to secondary root |

### STP Design Values

| Item                    | Value       |
| ----------------------- | ----------- |
| STP Mode                | Rapid-PVST+ |
| Primary Root Bridge     | D1          |
| Secondary Root Bridge   | D2          |
| Primary Root Priority   | 4096        |
| Secondary Root Priority | 8192        |
| VLANs                   | 10, 20      |

### Expected STP Port Roles

| Device | Interface | Expected Role | Expected State | Notes                  |
| ------ | --------- | ------------- | -------------- | ---------------------- |
| D1     | Gi0/0     | Designated    | Forwarding     | Link toward D2         |
| D1     | Gi0/1     | Designated    | Forwarding     | Link toward A1         |
| D2     | Gi0/0     | Root Port     | Forwarding     | Best path toward D1    |
| D2     | Gi0/1     | Designated    | Forwarding     | Link toward A1         |
| A1     | Gi0/0     | Root Port     | Forwarding     | Best path toward D1    |
| A1     | Gi0/1     | Alternate     | Discarding     | Backup path through D2 |

**Note:** Expected port roles depend on the reference topology, bridge priorities, and link costs. If the physical topology or path costs change, expected port roles may also change.

---

## Prerequisites and Pre-Checks

Before configuring Rapid-PVST+ root bridge placement, confirm that the Layer 2 baseline is already working.

### Prerequisites

* [ ] D1, D2, and A1 are connected according to the reference design.
* [ ] VLAN 10 exists on D1, D2, and A1.
* [ ] VLAN 20 exists on D1, D2, and A1.
* [ ] Required inter-switch links are configured as trunks.
* [ ] VLANs 10 and 20 are allowed across required trunks.
* [ ] No unintended Layer 2 loops exist outside the reference design.
* [ ] No ports are currently err-disabled.
* [ ] No unexpected switch is already configured with a lower STP priority for VLANs 10 or 20.

### Baseline Verification

Run these commands before applying the Rapid-PVST+ root bridge configuration.

```bash
show vlan brief
show interfaces trunk
show cdp neighbors
show spanning-tree summary
show spanning-tree vlan 10
show spanning-tree vlan 20
show interfaces status
```

### Expected Baseline Results

* [ ] VLANs 10 and 20 exist on all required switches.
* [ ] Trunk links between D1, D2, and A1 are operational.
* [ ] VLANs 10 and 20 are allowed on the required trunks.
* [ ] CDP confirms the expected switch adjacencies.
* [ ] No interfaces are err-disabled.
* [ ] Current STP root placement is understood before making changes.

---

## Configuration Procedure

Use this procedure to configure Rapid-PVST+ root bridge placement for VLANs 10 and 20.

### Configuration Notes

* Rapid-PVST+ should be enabled consistently on all switches in the Layer 2 domain.
* The switch with the lowest bridge ID becomes the root bridge.
* Bridge priority is evaluated before MAC address during root bridge election.
* D1 is configured with priority `4096` so it becomes the primary root bridge.
* D2 is configured with priority `8192` so it becomes the secondary root bridge.
* A1 remains at the default bridge priority and should not become root during normal operation.

---

### Step 1 — Configure Rapid-PVST+ Mode

Run on D1, D2, and A1.

```bash
enable
configure terminal

spanning-tree mode rapid-pvst

end
write memory
```

### Step 2 — Configure D1 as the Primary Root Bridge

Run on D1.

```bash
enable
configure terminal

spanning-tree vlan 10,20 priority 4096

end
write memory
```

### Step 3 — Configure D2 as the Secondary Root Bridge

Run on D2.

```bash
enable
configure terminal

spanning-tree vlan 10,20 priority 8192

end
write memory
```

### Step 4 — Confirm Configuration Was Applied

Run on D1 and D2.

```bash
show running-config | include ^spanning-tree mode
show running-config | include ^spanning-tree vlan
show spanning-tree vlan 10
show spanning-tree vlan 20
```

Expected result:

* [ ] Rapid-PVST+ is enabled.
* [ ] D1 has STP priority `4096` for VLANs 10 and 20.
* [ ] D2 has STP priority `8192` for VLANs 10 and 20.
* [ ] D1 is elected root bridge for VLANs 10 and 20.
* [ ] D2 is not root during normal operation.

---

## Post-Configuration Validation

Use this section to confirm that Rapid-PVST+ root bridge placement is configured correctly and operating in the expected state after implementation.

### Step 1 — Verify STP Mode

Run on D1, D2, and A1.

```bash
show spanning-tree summary
```

Expected results:

* [ ] Switch is running Rapid-PVST+.
* [ ] VLANs 10 and 20 are participating in spanning tree.
* [ ] No unexpected STP instability is present.
* [ ] No excessive topology changes are present.

---

### Step 2 — Verify D1 Root Bridge State

Run on D1.

```bash
show spanning-tree vlan 10
show spanning-tree vlan 20
```

Expected results:

* [ ] D1 is the root bridge for VLAN 10.
* [ ] D1 is the root bridge for VLAN 20.
* [ ] Root ID and local Bridge ID match.
* [ ] D1 bridge priority is `4096` for VLANs 10 and 20.
* [ ] D1 ports toward D2 and A1 are designated forwarding ports.

---

### Step 3 — Verify D2 Secondary Root Behavior

Run on D2.

```bash
show spanning-tree vlan 10
show spanning-tree vlan 20
```

Expected results:

* [ ] D1 is listed as the root bridge for VLAN 10.
* [ ] D1 is listed as the root bridge for VLAN 20.
* [ ] D2 is not the active root bridge during normal operation.
* [ ] D2 bridge priority is `8192` for VLANs 10 and 20.
* [ ] D2 root port points toward D1.
* [ ] D2 remains the next preferred root bridge if D1 becomes unavailable.

---

### Step 4 — Verify A1 Non-Root Behavior

Run on A1.

```bash
show spanning-tree vlan 10
show spanning-tree vlan 20
```

Expected results:

* [ ] D1 is listed as the root bridge for VLAN 10.
* [ ] D1 is listed as the root bridge for VLAN 20.
* [ ] A1 is not the root bridge.
* [ ] A1 root port points toward D1.
* [ ] A1 alternate path points toward D2.
* [ ] A1 bridge priority remains higher than D1 and D2.

---

## Functional Validation

Use this section to confirm that the secondary root bridge becomes root during a controlled primary root failure.

**Note:** Perform failure testing only in a lab or approved maintenance window.

### Step 1 — Confirm Initial Root Bridge State

Run on D1, D2, and A1.

```bash
show spanning-tree vlan 10
show spanning-tree vlan 20
```

Expected results:

* [ ] D1 is root bridge for VLANs 10 and 20.
* [ ] D2 is secondary root candidate for VLANs 10 and 20.
* [ ] A1 is not root for VLANs 10 or 20.
* [ ] A1 root path points toward D1.

---

### Step 2 — Simulate Primary Root Failure

In the lab, shut down the D1 links participating in the Layer 2 topology.

Run on D1.

```bash
enable
configure terminal

interface range gigabitEthernet 0/0 - 1
shutdown

end
```

Expected results:

* [ ] D1 is removed from the active Layer 2 topology.
* [ ] D2 becomes the root bridge for VLANs 10 and 20.
* [ ] A1 selects a root path toward D2.
* [ ] No Layer 2 loop forms during convergence.

---

### Step 3 — Verify D2 Becomes Root

Run on D2 and A1.

```bash
show spanning-tree vlan 10
show spanning-tree vlan 20
```

Expected results:

* [ ] D2 is root bridge for VLAN 10.
* [ ] D2 is root bridge for VLAN 20.
* [ ] Root ID and local Bridge ID match on D2.
* [ ] A1 lists D2 as the root bridge.
* [ ] A1 root port points toward D2.

---

### Step 4 — Restore the Primary Root Bridge

Run on D1.

```bash
enable
configure terminal

interface range gigabitEthernet 0/0 - 1
no shutdown

end
```

Expected results:

* [ ] D1 links return to up/up.
* [ ] D1 rejoins the Layer 2 topology.
* [ ] D1 is again elected root bridge because it has the lowest bridge priority.
* [ ] D2 returns to secondary root behavior.

---

### Step 5 — Verify Final Root Bridge State

Run on D1, D2, and A1.

```bash
show spanning-tree vlan 10
show spanning-tree vlan 20
```

Expected results:

* [ ] D1 is root bridge for VLANs 10 and 20.
* [ ] D2 is not root during normal operation.
* [ ] A1 is not root during normal operation.
* [ ] A1 root path returns to D1.
* [ ] Final STP state matches the reference design.

---

## If Validation Fails

If post-configuration or functional validation does not match the expected results, stop and verify the baseline before making additional changes.

Run these checks first:

```bash
show spanning-tree summary
show spanning-tree vlan 10
show spanning-tree vlan 20
show vlan brief
show interfaces trunk
show cdp neighbors
show interfaces status
```

Confirm the following:

* [ ] Rapid-PVST+ is enabled on all switches.
* [ ] VLANs 10 and 20 exist on all required switches.
* [ ] VLANs 10 and 20 are allowed across the required trunks.
* [ ] D1 has the lowest STP priority for VLANs 10 and 20.
* [ ] D2 has the second-lowest STP priority for VLANs 10 and 20.
* [ ] A1 does not have a lower bridge priority than D1 or D2.
* [ ] The expected trunk links are operational.
* [ ] No unintended switch is advertising a superior BPDU.

If the issue is not resolved after these checks, stop and document the failed validation results before continuing with deeper troubleshooting.

---

## Backout Considerations

Changing STP root bridge placement can affect Layer 2 forwarding paths and may cause traffic to shift between uplinks.

Before removing or changing the configuration, confirm:

* The intended root bridge and secondary root bridge are documented.
* The affected VLANs are identified.
* Trunk paths and allowed VLANs are understood.
* The expected forwarding and alternate paths are documented.
* A maintenance or lab validation window is available if traffic interruption is possible.

For lab use, restore the last known working configuration before continuing additional validation.

---

## Document Metadata

| Field            | Value                     |
| ---------------- | ------------------------- |
| Author           | Aaron Kindelt             |
| Category         | Layer 2 Switching         |
| Technology       | Rapid-PVST+               |
| Applies To       | Layer 2 Switches          |
| Primary Use Case | STP root bridge placement |
| Version          | 1.0                       |
| Last Updated     | 2026-06-28                |
