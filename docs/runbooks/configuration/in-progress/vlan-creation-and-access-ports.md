# Runbook — VLAN Creation and Access Ports
 
## Overview
 
This runbook provides a structured process for configuring and validating VLAN creation and Layer 2 access port assignment on Cisco IOS switches.
 
VLANs provide Layer 2 segmentation by separating switch ports into distinct broadcast domains. Access ports assign endpoint-facing switch interfaces to a single VLAN so connected devices participate in the intended Layer 2 segment.
 
This runbook focuses on a traditional Cisco IOS switching design where VLANs are created locally on the switch and endpoint-facing interfaces are configured as static access ports. The example reference design uses one switch, multiple VLANs, and dedicated access interfaces assigned to those VLANs.
 
## Scope and Assumptions
 
This runbook assumes:
 
- The switch is powered on and accessible for configuration.
- The required device baseline has already been applied.
- The switch is operating as a Layer 2 switch or multilayer switch with switching capability.
- The required VLAN IDs and VLAN names have been defined.
- The endpoint-facing interfaces to be assigned as access ports have been identified.
- Trunk links are configured separately.
- Layer 3 SVI configuration and inter-VLAN routing are configured separately.
- Spanning-tree root placement, EtherChannel, and other switching features are configured separately.
 
This runbook does not cover trunk configuration, EtherChannel, SVI addressing, inter-VLAN routing, dynamic VLAN assignment, 802.1X, or advanced access-layer security features.

---

## Reference Design
 
This runbook uses a single Cisco IOS switch with multiple locally defined VLANs and endpoint-facing interfaces configured as static Layer 2 access ports.
 
### Devices
 
| Device | Role              | Purpose                                      |
|--------|-------------------|----------------------------------------------|
| SW1    | Cisco IOS switch  | Hosts VLANs and endpoint-facing access ports |
 
### VLAN Summary
 
| VLAN | Name       | Purpose                  |
|------|------------|--------------------------|
| 10   | USERS      | User endpoint network    |
| 20   | PRINTERS      | Voice endpoint network   |
| 30   | SERVERS    | Server endpoint network  |
 
### Access Port Summary
 
| Interface | Access VLAN | Purpose                  |
|-----------|-------------|--------------------------|
| Gi0/1     | 10          | User endpoint            |
| Gi0/2     | 20          | Voice endpoint           |
| Gi0/3     | 30          | Server endpoint          |
 
**Note:** The values shown in this reference design are examples. Replace VLAN IDs, VLAN names, interface numbers, and endpoint roles with those appropriate to the target deployment.

---

## Prerequisites and Pre-Checks
 
Before configuring VLANs and access ports, confirm that the required switch baseline is already in place.
 
### Prerequisites
 
- [ ] The switch is powered on and accessible for configuration.
- [ ] The expected Cisco IOS or IOSvL2 image is installed and the device boots successfully.
- [ ] The required VLAN IDs, VLAN names, and endpoint roles have been defined.
- [ ] The endpoint-facing interfaces to be configured as access ports have been identified.
- [ ] Existing interface assignments have been reviewed to confirm that the target interfaces can be safely repurposed.
 
### Baseline Verification
 
Run these commands before applying the VLAN and access-port configuration.
 
```bash
show running-config
show vlan brief
show interfaces status
show interfaces switchport
```
 
### Expected Baseline Results
 
- [ ] The running configuration matches the expected device baseline.
- [ ] No unexpected VLANs are present.
- [ ] Target interfaces are not already assigned to unintended production VLANs.
- [ ] Target interfaces are operating in the expected switchport mode before modification.
- [ ] No conflicting interface configuration is present on the target access ports.

---

## Configuration Procedure
 
Use this procedure to configure VLANs and Layer 2 access ports.
 
### Configuration Notes
 
- Create the required VLANs before assigning interfaces to them.
- Replace all example VLAN IDs, VLAN names, and interface numbers with those defined for the target deployment.
- Configure endpoint-facing interfaces explicitly as static Layer 2 access ports.
- If a target interface was previously assigned to a parking VLAN or administratively disabled, the access-port configuration replaces the parking VLAN assignment and returns the interface to service.
- Trunk, EtherChannel, SVI, and Layer 3 configuration are handled separately.
 
---
 
### Step 1 — Configure VLANs
 
```bash
enable # Enter privileged EXEC mode
configure terminal # Enter global configuration mode
 
vlan 10 # Create and target VLAN 10
name USERS # Name VLAN 10
 
vlan 20 # Create and target VLAN 20
name PRINTERS # Name VLAN 20
 
vlan 30 # Create and target VLAN 30
name SERVERS # Name VLAN 30
 
end # Return to privileged EXEC mode
write memory # Save the running configuration
```
 
---
 
### Step 2 — Configure Access Ports
 
```bash
enable # Enter privileged EXEC mode
configure terminal # Enter global configuration mode
 
interface Gi0/1 # Target the user endpoint interface
switchport mode access # Configure the interface as a static Layer 2 access port
switchport access vlan 10 # Assign the interface to VLAN 10
no shutdown # Administratively enable the interface
 
interface Gi0/2 # Target the printer endpoint interface
switchport mode access # Configure the interface as a static Layer 2 access port
switchport access vlan 20 # Assign the interface to VLAN 20
no shutdown # Administratively enable the interface
 
interface Gi0/3 # Target the server endpoint interface
switchport mode access # Configure the interface as a static Layer 2 access port
switchport access vlan 30 # Assign the interface to VLAN 30
no shutdown # Administratively enable the interface
 
end # Return to privileged EXEC mode
write memory # Save the running configuration
```
 

---

## Post-Configuration Validation
 
Use this section to confirm that the required VLANs and access ports are configured correctly and operating in the expected state after implementation.
 
### Step 1 — Verify VLAN Configuration
 
Run on the switch.
 
```bash
show vlan brief
```
 
Expected results:
 
- [ ] VLAN 10 exists and is named `USERS`.
- [ ] VLAN 20 exists and is named `VOICE`.
- [ ] VLAN 30 exists and is named `SERVERS`.
- [ ] Each VLAN is active.
- [ ] The expected access interfaces appear under their assigned VLANs.
 
---
 
### Step 2 — Verify Access Port Switchport State
 
Run on the switch.
 
```bash
show interfaces Gi0/1 switchport
show interfaces Gi0/2 switchport
show interfaces Gi0/3 switchport
```
 
Expected results:
 
- [ ] `Gi0/1` is operating as a static access port in VLAN 10.
- [ ] `Gi0/2` is operating as a static access port in VLAN 20.
- [ ] `Gi0/3` is operating as a static access port in VLAN 30.
- [ ] The target interfaces are not operating as trunks.
- [ ] The configured access VLAN matches the intended VLAN for each interface.
 
---
 
### Step 3 — Verify Access Port Configuration
 
Run on the switch.
 
```bash
show running-config interface Gi0/1
show running-config interface Gi0/2
show running-config interface Gi0/3
```
 
Expected results:
 
- [ ] Each target interface is configured with `switchport mode access`.
- [ ] Each target interface has the expected `switchport access vlan` assignment.
- [ ] Each target interface is administratively enabled.
- [ ] No conflicting switchport configuration is present on the target interfaces.
 
---
 
### Step 4 — Verify Interface State
 
Run on the switch.
 
```bash
show interfaces status
```
 
Expected results:
 
- [ ] Each configured access port is administratively enabled.
- [ ] Connected endpoint-facing interfaces show the expected operational state.
- [ ] Each target interface reports the expected access VLAN.
- [ ] No target access port remains assigned to an unintended or parking VLAN.

---

## If Validation Fails
 
If post-configuration validation does not produce the expected results:
 
- Identify which validation step failed.
- Review the related configuration for missing, incorrect, or conflicting settings.
- Correct the affected configuration.
- Re-run the failed validation step.
- If the issue cannot be resolved, return the device to its previous known-good state before continuing.

---

## Backout Considerations
 
Changing VLAN membership or access-port configuration can interrupt Layer 2 connectivity for connected endpoints.
 
Before removing or changing the configuration, confirm:
 
- The original VLAN and interface configuration is documented.
- The previous access VLAN assignment for each affected interface is known.
- Any interfaces that were previously assigned to a parking VLAN are identified.
- The affected endpoint connectivity and VLAN dependencies are understood.
- The switch can be returned to its previous known-good configuration if validation fails.
 
For lab use, restore the last known working configuration before continuing additional validation.

---

## Document Metadata
 
| Field            | Value                                              |
|------------------|----------------------------------------------------|
| Author           | Aaron Kindelt                                      |
| Category         | Configuration                                      |
| Technology       | Cisco IOS Layer 2 Switching                        |
| Applies To       | Cisco IOS / IOSvL2 switches                        |
| Primary Use Case | VLAN creation and static access-port configuration |
| Version          | 1.0                                                |
| Last Updated     | 2026-09-11                                         |
