# Runbook — LACP EtherChannel
 
## Overview
 
This runbook provides a structured process for configuring and validating Layer 2 EtherChannel using Link Aggregation Control Protocol (LACP) on Cisco IOS switches.
 
LACP combines multiple physical Ethernet links into a single logical Port-Channel interface, increasing available bandwidth and providing link redundancy while allowing spanning tree to treat the bundle as a single logical connection.
 
This runbook focuses on statically defined Layer 2 LACP EtherChannels between Cisco IOS switches. The example reference design uses two physical links bundled into a single Port-Channel operating as an 802.1Q trunk.

## Scope and Assumptions
 
This runbook assumes:
 
- The participating switches are powered on and accessible for configuration.
- The required device baselines have already been applied.
- The physical interfaces selected for the EtherChannel have been identified.
- The participating interfaces have matching speed, duplex, switchport mode, native VLAN, and allowed VLAN settings where applicable.
- The required VLANs already exist on both switches.
- The Port-Channel number has been defined.
- LACP will be used as the EtherChannel negotiation protocol.
- Spanning-tree root placement and other Layer 2 design features are handled separately.
 
This runbook does not cover PAgP, static `on` EtherChannel, Layer 3 EtherChannel, cross-stack or multi-chassis EtherChannel, or vendor-specific link aggregation implementations.

---

## Reference Design
 
This runbook uses two Cisco IOS switches connected by two physical Ethernet links bundled into a single LACP EtherChannel.
 
The resulting Port-Channel operates as an 802.1Q trunk carrying VLANs 10, 20, and 30. VLAN 99 is used as the native VLAN.
 
### Devices
 
| Device | Role             | Purpose                                  |
|--------|------------------|------------------------------------------|
| SW1    | Cisco IOS switch | First endpoint of the LACP EtherChannel  |
| SW2    | Cisco IOS switch | Second endpoint of the LACP EtherChannel |
 
### EtherChannel Summary
 
| Device | Member Interfaces | Port-Channel | LACP Mode | Link Type |
|--------|-------------------|--------------|-----------|-----------|
| SW1    | Gi0/0, Gi0/1      | Po1          | active    | 802.1Q trunk |
| SW2    | Gi0/0, Gi0/1      | Po1          | active    | 802.1Q trunk |
 
### Trunk Summary
 
| Port-Channel | Allowed VLANs | Native VLAN |
|--------------|---------------|-------------|
| Po1          | 10,20,30      | 99          |
 
### VLAN Summary
 
| VLAN | Name    | Purpose                |
|------|---------|------------------------|
| 10   | USERS   | User endpoint network  |
| 20   | VOICE   | Voice endpoint network |
| 30   | SERVERS | Server endpoint network |
| 99   | NATIVE  | 802.1Q native VLAN     |
 
**Note:** The values shown in this reference design are examples. Replace interface numbers, Port-Channel number, LACP mode, VLAN IDs, native VLAN, and allowed VLAN list with those appropriate to the target deployment.

---

## Prerequisites and Pre-Checks
 
Before configuring the LACP EtherChannel, confirm that the participating interfaces and VLAN information are ready for implementation.
 
### Prerequisites
 
- [ ] The participating switches are powered on and accessible for configuration.
- [ ] The required device baselines have already been applied.
- [ ] The required VLANs exist on both switches.
- [ ] The physical interfaces selected for the EtherChannel have been identified.
- [ ] The selected member interfaces are not currently used for another production connection.
- [ ] The selected member interfaces are not already assigned to another EtherChannel.
- [ ] The Port-Channel number has been defined.
- [ ] The allowed VLAN list and native VLAN have been defined.
- [ ] Member interfaces use compatible speed and duplex settings.
 
### Baseline Verification
 
Run these commands on both switches before applying the EtherChannel configuration.
 
```bash
show interfaces status
show interfaces switchport
show etherchannel summary
show vlan brief
```
 
### Expected Baseline Results
 
- [ ] The selected member interfaces are present and available for configuration.
- [ ] The selected member interfaces are not members of another EtherChannel.
- [ ] No conflicting switchport configuration is present on the selected interfaces.
- [ ] The required VLANs exist and are active.
- [ ] The intended Port-Channel number is not already in use for another purpose.
- [ ] The selected interfaces have compatible physical characteristics for bundling.

---

## Configuration Procedure
 
Use this procedure to configure a Layer 2 LACP EtherChannel between Cisco IOS switches.
 
### Configuration Notes
 
- Configure all physical member interfaces with matching Layer 2 settings before adding them to the EtherChannel.
- Use the same Port-Channel number on both switches for consistency, although the number is locally significant.
- Configure LACP in `active` mode on both ends of the reference design.
- Apply trunk-specific VLAN configuration to the logical Port-Channel interface.
- Replace example interface numbers, Port-Channel numbers, VLAN IDs, and VLAN lists with those defined for the target deployment.
- Interfaces previously assigned to a parking VLAN or administratively disabled must be returned to service as part of the EtherChannel configuration.
- After the EtherChannel is formed, make future Layer 2 configuration changes on the Port-Channel interface rather than individual member interfaces.
 
---
 
### Step 1 — Configure SW1 LACP EtherChannel
 
```bash
enable # Enter privileged EXEC mode
configure terminal # Enter global configuration mode
 
interface range gi0/0 - 1 # Targets physical EtherChannel member interfaces
switchport mode trunk # Configures member interfaces as Layer 2 trunks
channel-group 1 mode active # Adds interfaces to Port-Channel 1 using active LACP
no shutdown # Administratively enables member interfaces
 
interface port-channel 1 # Targets the logical EtherChannel interface
switchport mode trunk # Configures Port-Channel 1 as a Layer 2 trunk
switchport trunk allowed vlan 10,20,30 # Allows VLANs 10, 20, and 30 across the EtherChannel
switchport trunk native vlan 99 # Sets VLAN 99 as the native VLAN
 
end # Return to privileged EXEC mode
write memory # Save the running configuration
```
 
---
 
### Step 2 — Configure SW2 LACP EtherChannel
 
```bash
enable # Enter privileged EXEC mode
configure terminal # Enter global configuration mode
 
interface range gi0/0 - 1 # Targets physical EtherChannel member interfaces
switchport mode trunk # Configures member interfaces as Layer 2 trunks
channel-group 1 mode active # Adds interfaces to Port-Channel 1 using active LACP
no shutdown # Administratively enables member interfaces
 
interface port-channel 1 # Targets the logical EtherChannel interface
switchport mode trunk # Configures Port-Channel 1 as a Layer 2 trunk
switchport trunk allowed vlan 10,20,30 # Allows VLANs 10, 20, and 30 across the EtherChannel
switchport trunk native vlan 99 # Sets VLAN 99 as the native VLAN
 
end # Return to privileged EXEC mode
write memory # Save the running configuration
```

---

## Post-Configuration Validation
 
Use this section to confirm that the LACP EtherChannel formed correctly and that the resulting Port-Channel is operating as the intended 802.1Q trunk.
 
### Step 1 — Verify EtherChannel Formation
 
Run on both switches.
 
```bash
show etherchannel summary
```
 
Expected results:
 
- [ ] `Po1` appears in the EtherChannel summary.
- [ ] The EtherChannel protocol is shown as `LACP`.
- [ ] `Po1` is operating as a Layer 2 Port-Channel.
- [ ] `Po1` is in use.
- [ ] Both physical member interfaces are bundled in the Port-Channel.
- [ ] No member interface is shown as suspended, standalone, or otherwise failed to bundle.
 
---
 
### Step 2 — Verify LACP Neighbor State
 
Run on both switches.
 
```bash
show lacp neighbor
```
 
Expected results:
 
- [ ] LACP neighbor information is present for each physical member interface.
- [ ] Both member interfaces have discovered an LACP partner.
- [ ] The expected remote switch is participating in the LACP relationship.
- [ ] No expected member interface is missing from the LACP neighbor output.
 
---
 
### Step 3 — Verify Port-Channel Trunk State
 
Run on both switches.
 
```bash
show interfaces port-channel 1 switchport
show interfaces trunk
```
 
Expected results:
 
- [ ] `Port-channel1` is administratively configured for trunking.
- [ ] `Port-channel1` is operationally trunking.
- [ ] The native VLAN is VLAN 99.
- [ ] VLANs 10, 20, and 30 are allowed on the trunk.
- [ ] `Port-channel1` appears in the trunk summary.
- [ ] The expected VLANs are active and forwarding across the Port-Channel.
 
---
 
### Step 4 — Verify Port-Channel Operational State
 
Run on both switches.
 
```bash
show interfaces port-channel 1
```
 
Expected results:
 
- [ ] `Port-channel1` is administratively up.
- [ ] The line protocol is up.
- [ ] The interface is operating as a Layer 2 Port-Channel.
- [ ] No unexpected errors or interface conditions are present.

---

## If Validation Fails
 
If post-configuration validation does not produce the expected results:
 
- Identify which validation step failed.
- Confirm that all intended physical member interfaces are assigned to the correct Port-Channel.
- Confirm that LACP is configured consistently on both ends of the EtherChannel.
- Review member interfaces for mismatched switchport, trunk, VLAN, speed, or duplex settings.
- Confirm that the Port-Channel trunk configuration matches on both switches.
- Correct the affected configuration.
- Re-run the failed validation step.
- If the issue cannot be resolved, return the affected interfaces and Port-Channel to their previous known-good state before continuing.

---

## Backout Considerations
 
Changing or removing EtherChannel configuration can interrupt Layer 2 connectivity across the bundled link.
 
Before removing or changing the configuration, confirm:
 
- The original physical interface configuration is documented.
- The existing Port-Channel configuration is documented.
- The previous switchport mode, allowed VLAN list, and native VLAN settings are known.
- Any services or downstream devices dependent on the EtherChannel are understood.
- Both ends of the EtherChannel can be returned to their previous known-good configuration if required.
- Member interfaces can be safely removed from the Port-Channel before restoring their prior standalone configuration.
 
For lab use, restore the last known working configuration before continuing additional validation.

---

## Document Metadata
 
| Field            | Value                                      |
|------------------|--------------------------------------------|
| Author           | Aaron Kindelt                              |
| Category         | Configuration                              |
| Technology       | Cisco IOS Layer 2 Switching / EtherChannel |
| Applies To       | Cisco IOS / IOSvL2 switches                |
| Primary Use Case | Layer 2 LACP EtherChannel configuration    |
| Version          | 1.0                                        |
| Last Updated     | 2026-09-11                                 |