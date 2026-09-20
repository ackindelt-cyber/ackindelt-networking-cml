# Runbook — STP-HSRP Alignment
 
## Overview
 
This runbook provides a structured process for aligning Rapid-PVST root bridge placement with HSRP active gateway roles on Cisco IOS multilayer switches.
 
Aligning spanning-tree and first-hop redundancy roles helps ensure that Layer 2 forwarding paths lead efficiently toward the Layer 3 gateway responsible for a given VLAN. When the HSRP active device is also the spanning-tree root for that VLAN, traffic can reach its default gateway without unnecessary Layer 2 transit through the peer switch.
 
This runbook focuses on coordinating existing Rapid-PVST and HSRP configurations across a redundant pair of multilayer switches. The example reference design distributes VLAN ownership between two switches so that each switch is both the HSRP active gateway and spanning-tree root for its assigned VLANs.
 

## Scope and Assumptions
 
This runbook assumes:
 
- Two Cisco IOS multilayer switches provide redundant Layer 2 and Layer 3 services for the same VLANs.
- Rapid-PVST is already configured and operational.
- HSRP is already configured and operational.
- The HSRP active and standby roles for each VLAN have been intentionally defined.
- The desired spanning-tree root and secondary-root placement for each VLAN has been defined.
- The switches provide gateway services for the affected VLANs through SVIs.
- VLANs are carried consistently across the required trunk and EtherChannel links.
- The existing HSRP and spanning-tree configurations have been validated independently before alignment changes are made.
- Access-layer topology and uplink redundancy are already in place.
 
This runbook does not cover initial HSRP configuration, initial Rapid-PVST root bridge configuration, VLAN creation, trunking, EtherChannel configuration, SVI creation, or general Layer 2 topology design.

---

## Reference Design
 
This runbook uses two Cisco IOS multilayer switches providing redundant Layer 2 and Layer 3 services for multiple VLANs.
 
HSRP active gateway roles are distributed between the two switches. Rapid-PVST root bridge placement is aligned with those gateway roles so that the switch acting as the active default gateway for a VLAN is also the spanning-tree root for that VLAN.
 
### Devices
 
| Device | Role                         | Purpose                                      |
|--------|------------------------------|----------------------------------------------|
| SW1    | Cisco IOS multilayer switch  | Primary gateway and STP root for selected VLANs |
| SW2    | Cisco IOS multilayer switch  | Primary gateway and STP root for selected VLANs |
 
### VLAN Role Alignment
 
| VLAN | HSRP Active | HSRP Standby | STP Root | STP Secondary |
|------|-------------|--------------|----------|---------------|
| 10   | SW1         | SW2          | SW1      | SW2           |
| 20   | SW2         | SW1          | SW2      | SW1           |
| 30   | SW1         | SW2          | SW1      | SW2           |
 
### Design Intent
 
Traffic from access-layer devices should forward toward the switch that owns the active HSRP gateway role for each VLAN.
 
By aligning the spanning-tree root with the HSRP active gateway, the Layer 2 forwarding path and Layer 3 gateway location remain consistent for the affected VLAN.
 
**Note:** The values shown in this reference design are examples. Replace VLAN IDs and device roles with those appropriate to the target deployment.

---

## Prerequisites and Pre-Checks
 
Before aligning spanning-tree root placement with HSRP gateway roles, confirm that both features are already configured and operating as intended.
 
### Prerequisites
 
- [ ] The participating multilayer switches are powered on and accessible for configuration.
- [ ] The required device baselines have already been applied.
- [ ] The required VLANs and SVIs have already been configured.
- [ ] HSRP is configured and operational for the affected VLANs.
- [ ] Rapid-PVST is configured and operational for the affected VLANs.
- [ ] The intended HSRP active and standby roles have been defined.
- [ ] The intended spanning-tree root and secondary-root roles have been defined.
- [ ] Required Layer 2 trunks and EtherChannels are operational.
- [ ] The existing HSRP and spanning-tree configurations have been validated independently.
 
### Baseline Verification
 
Run these commands on both multilayer switches before applying alignment changes.
 
```bash
show standby brief
show spanning-tree root
show spanning-tree vlan 10
show spanning-tree vlan 20
show spanning-tree vlan 30
```
 
### Expected Baseline Results
 
- [ ] HSRP is operational for each affected VLAN.
- [ ] The active and standby HSRP roles match the intended gateway design.
- [ ] Rapid-PVST is operational for each affected VLAN.
- [ ] The current spanning-tree root bridge is identified for each VLAN.
- [ ] The desired HSRP-to-STP alignment has been documented before changes are made.
- [ ] No unexpected HSRP or spanning-tree state is present that should be corrected before alignment.
 
## Configuration Procedure
 
Use this procedure to align Rapid-PVST root bridge placement with the existing HSRP active and standby gateway roles.
 
### Configuration Notes
 
- Confirm the intended HSRP active and standby roles before modifying spanning-tree priorities.
- This procedure does not change HSRP configuration. If the HSRP roles are incorrect, correct them using the HSRP runbook before performing STP alignment.
- Configure the HSRP active gateway as the spanning-tree root bridge for the corresponding VLAN.
- Configure the HSRP standby gateway as the spanning-tree secondary root for the corresponding VLAN.
- This reference design uses STP priority `24576` for the primary root and `28672` for the secondary root.
- Other switches should remain at the default priority unless the design requires otherwise.
- Replace example VLAN IDs and device roles with those defined for the target deployment.
 
---
 
### Step 1 — Align SW1 Spanning-Tree Roles
 
```bash
enable # Enter privileged EXEC mode
configure terminal # Enter global configuration mode
 
spanning-tree vlan 10,30 priority 24576 # Make SW1 root bridge for VLANs where it is HSRP active
spanning-tree vlan 20 priority 28672 # Make SW1 secondary root for VLAN 20 where it is HSRP standby
 
end # Return to privileged EXEC mode
write memory # Save the running configuration
```
 
---
 
### Step 2 — Align SW2 Spanning-Tree Roles
 
```bash
enable # Enter privileged EXEC mode
configure terminal # Enter global configuration mode
 
spanning-tree vlan 20 priority 24576 # Make SW2 root bridge for VLAN 20 where it is HSRP active
spanning-tree vlan 10,30 priority 28672 # Make SW2 secondary root for VLANs where it is HSRP standby
 
end # Return to privileged EXEC mode
write memory # Save the running configuration
```
 

---

## Post-Configuration Validation
 
Use this section to confirm that HSRP gateway roles and Rapid-PVST root bridge placement are aligned as intended for each VLAN.
 
### Step 1 — Verify HSRP Gateway Roles
 
Run on both multilayer switches.
 
```bash
show standby brief
```
 
Expected results:
 
- [ ] SW1 is HSRP active for VLANs 10 and 30.
- [ ] SW2 is HSRP standby for VLANs 10 and 30.
- [ ] SW2 is HSRP active for VLAN 20.
- [ ] SW1 is HSRP standby for VLAN 20.
- [ ] No unexpected HSRP state is present for the affected VLANs.
 
---
 
### Step 2 — Verify Spanning-Tree Root Placement
 
Run on both multilayer switches.
 
```bash
show spanning-tree vlan 10
show spanning-tree vlan 20
show spanning-tree vlan 30
```
 
Expected results:
 
- [ ] SW1 is the spanning-tree root bridge for VLANs 10 and 30.
- [ ] SW2 is the spanning-tree secondary root for VLANs 10 and 30.
- [ ] SW2 is the spanning-tree root bridge for VLAN 20.
- [ ] SW1 is the spanning-tree secondary root for VLAN 20.
- [ ] Primary root switches are configured with base priority `24576`.
- [ ] Secondary root switches are configured with base priority `28672`.
- [ ] No unintended switch has become root for an affected VLAN.
 
**Note:** With the extended system ID enabled, `show spanning-tree` displays the configured bridge priority plus the VLAN ID.
 
---
 
### Step 3 — Verify HSRP and STP Role Alignment
 
Compare the HSRP and spanning-tree results for each VLAN.
 
| VLAN | Expected HSRP Active | Expected STP Root | Aligned |
|------|----------------------|-------------------|---------|
| 10   | SW1                  | SW1               | Yes     |
| 20   | SW2                  | SW2               | Yes     |
| 30   | SW1                  | SW1               | Yes     |
 
Expected results:
 
- [ ] The HSRP active gateway is also the spanning-tree root for VLAN 10.
- [ ] The HSRP active gateway is also the spanning-tree root for VLAN 20.
- [ ] The HSRP active gateway is also the spanning-tree root for VLAN 30.
- [ ] The HSRP standby gateway is the preferred secondary spanning-tree root for each corresponding VLAN.
- [ ] Final gateway and Layer 2 forwarding roles match the intended design.

---

## If Validation Fails
 
If post-configuration validation does not produce the expected results:
 
- Identify which validation step failed.
- Confirm that HSRP active and standby roles match the intended gateway design.
- Confirm that the configured spanning-tree priorities match the intended root and secondary-root roles.
- Verify that the HSRP active gateway is also the spanning-tree root for each affected VLAN.
- Confirm that no unintended switch has a lower spanning-tree bridge priority for the affected VLANs.
- Review trunk and EtherChannel state if the expected spanning-tree topology is not present.
- Correct the affected configuration.
- Re-run the failed validation step.
- If the issue cannot be resolved, return the affected spanning-tree configuration to its previous known-good state before continuing.
 

---

## Backout Considerations
 
Changing STP-HSRP alignment can alter Layer 2 forwarding paths and the relationship between gateway ownership and spanning-tree root placement.
 
Before removing or changing the configuration, confirm:
 
- The intended HSRP active and standby roles are documented.
- The intended spanning-tree root and secondary-root roles are documented.
- The affected VLANs are identified.
- Current trunk and EtherChannel paths are understood.
- The previous spanning-tree priorities are known.
- The effect of returning to the previous root placement is understood.
- Both multilayer switches can be returned to their previous known-good spanning-tree state if required.
 
For lab use, restore the last known working configuration before continuing additional validation.

---

## Document Metadata
 
| Field            | Value                                            |
|------------------|--------------------------------------------------|
| Author           | Aaron Kindelt                                    |
| Category         | Configuration / Design Alignment                 |
| Technology       | Rapid-PVST+ / HSRP                               |
| Applies To       | Cisco IOS multilayer switches                    |
| Primary Use Case | Align STP root placement with HSRP gateway roles |
| Version          | 1.0                                              |
| Last Updated     | 2026-09-20                                       |
