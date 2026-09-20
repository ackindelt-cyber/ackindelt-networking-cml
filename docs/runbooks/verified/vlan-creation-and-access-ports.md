# Runbook — VLAN Creation and Access Ports

## Overview

This runbook provides a structured process for configuring and validating VLAN creation and Layer 2 access-port assignment on Cisco IOS switches.

VLANs provide Layer 2 segmentation by separating switch ports into distinct broadcast domains. Access ports assign endpoint-facing switch interfaces to the appropriate data VLAN. Access ports connected to IP phones can also use a dedicated voice VLAN so that voice and workstation traffic remain logically separated while sharing the same physical switch port.

This runbook focuses on a traditional Cisco IOS access-layer design where VLANs are created locally and endpoint-facing interfaces are configured as static Layer 2 access ports. The example reference design includes a standard user access port, a phone-facing access port carrying separate data and voice VLANs, and a server access port.

---

## Scope and Assumptions

This runbook assumes:

- The switch is powered on and accessible for configuration.
- The required device baseline has already been applied.
- The switch is operating as a Layer 2 switch or multilayer switch with switching capability.
- The required VLAN IDs and VLAN names have been defined.
- The endpoint-facing interfaces to be configured as access ports have been identified.
- The selected endpoint interfaces are operating as Layer 2 switchports.
- Trunk links are configured separately.
- Layer 3 SVI configuration and inter-VLAN routing are configured separately.
- Spanning-tree root placement, EtherChannel, and other switching features are configured separately.
- Voice QoS policy and IP phone services are configured separately.

This runbook does not cover trunk configuration, EtherChannel, SVI addressing, inter-VLAN routing, dynamic VLAN assignment, 802.1X, voice QoS policy, or advanced access-layer security features.

---

## Reference Design

This runbook uses a single Cisco IOS switch with multiple locally defined VLANs and endpoint-facing interfaces configured as static Layer 2 access ports.

VLAN 10 provides user data connectivity, VLAN 20 provides dedicated voice connectivity, and VLAN 30 provides server connectivity.

### Devices

| Device | Role             | Purpose                                      |
|--------|------------------|----------------------------------------------|
| SW1    | Cisco IOS switch | Hosts VLANs and endpoint-facing access ports |

### VLAN Summary

| VLAN | Name    | Purpose                |
|------|---------|------------------------|
| 10   | USERS   | User endpoint network  |
| 20   | VOICE   | Voice endpoint network |
| 30   | SERVERS | Server endpoint network |

### Access Port Summary

| Interface | Access VLAN | Voice VLAN | Purpose                       |
|-----------|-------------|------------|-------------------------------|
| Gi0/1     | 10          | None       | User workstation              |
| Gi0/2     | 10          | 20         | IP phone with attached PC     |
| Gi0/3     | 30          | None       | Server endpoint               |

**Note:** The values shown in this reference design are examples. Replace VLAN IDs, VLAN names, interface numbers, and endpoint roles with those appropriate to the target deployment.

---

## Prerequisites and Pre-Checks

Before configuring VLANs and access ports, confirm that the required switch baseline is already in place.

### Prerequisites

- [ ] The switch is powered on and accessible for configuration.
- [ ] The required device baseline has already been applied.
- [ ] The required VLAN IDs, VLAN names, and endpoint roles have been defined.
- [ ] The endpoint-facing interfaces to be configured as access ports have been identified.
- [ ] The selected interfaces are operating as Layer 2 switchports.
- [ ] Existing interface assignments have been reviewed to confirm that the target interfaces can be safely repurposed.
- [ ] Any port intended for an IP phone has been identified before voice VLAN configuration is applied.

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
- [ ] Target interfaces are operating as Layer 2 switchports.
- [ ] No conflicting access or voice VLAN configuration is present on the target interfaces.
- [ ] The target interfaces can be safely repurposed.

---

## Configuration Procedure

Use this procedure to configure VLANs and Layer 2 access ports.

### Configuration Notes

- Create the required VLANs before assigning interfaces to them.
- Configure endpoint-facing interfaces explicitly as static Layer 2 access ports.
- A phone-facing access port can carry workstation data in the configured access VLAN while carrying tagged phone traffic in a separate voice VLAN.
- Replace all example VLAN IDs, VLAN names, and interface numbers with those defined for the target deployment.
- If a target interface was previously assigned to a parking VLAN or administratively disabled, the new access-port configuration replaces the parking VLAN assignment and returns the interface to service.
- Trunk, EtherChannel, SVI, Layer 3, and voice QoS configuration are handled separately.

---

### Step 1 — Configure VLANs

```bash
enable # Enter privileged EXEC mode
configure terminal # Enter global configuration mode

vlan 10 # Create and target VLAN 10
name USERS # Name VLAN 10

vlan 20 # Create and target VLAN 20
name VOICE # Name VLAN 20

vlan 30 # Create and target VLAN 30
name SERVERS # Name VLAN 30

end # Return to privileged EXEC mode
write memory # Save the running configuration
```

---

### Step 2 — Configure Standard User Access Port

```bash
enable # Enter privileged EXEC mode
configure terminal # Enter global configuration mode

interface Gi0/1 # Target the user workstation interface
switchport mode access # Configure the interface as a static Layer 2 access port
switchport access vlan 10 # Assign untagged endpoint traffic to VLAN 10
no shutdown # Administratively enable the interface

end # Return to privileged EXEC mode
write memory # Save the running configuration
```

---

### Step 3 — Configure Data and Voice Access Port

```bash
enable # Enter privileged EXEC mode
configure terminal # Enter global configuration mode

interface Gi0/2 # Target the IP phone and workstation interface
switchport mode access # Configure the interface as a static Layer 2 access port
switchport access vlan 10 # Assign workstation data traffic to VLAN 10
switchport voice vlan 20 # Assign tagged IP phone voice traffic to VLAN 20
no shutdown # Administratively enable the interface

end # Return to privileged EXEC mode
write memory # Save the running configuration
```

---

### Step 4 — Configure Server Access Port

```bash
enable # Enter privileged EXEC mode
configure terminal # Enter global configuration mode

interface Gi0/3 # Target the server endpoint interface
switchport mode access # Configure the interface as a static Layer 2 access port
switchport access vlan 30 # Assign the interface to VLAN 30
no shutdown # Administratively enable the interface

end # Return to privileged EXEC mode
write memory # Save the running configuration
```

---

## Post-Configuration Validation

Use this section to confirm that the required VLANs, access ports, and voice VLAN assignment are configured correctly.

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
- [ ] Standard access interfaces appear under their expected access VLANs.

---

### Step 2 — Verify Standard User Access Port

Run on the switch.

```bash
show interfaces Gi0/1 switchport
show running-config interface Gi0/1
```

Expected results:

- [ ] `Gi0/1` is operating as a static access port.
- [ ] The configured access VLAN is VLAN 10.
- [ ] The interface is not operating as a trunk.
- [ ] The interface is administratively enabled.
- [ ] No conflicting switchport configuration is present.

---

### Step 3 — Verify Data and Voice Access Port

Run on the switch.

```bash
show interfaces Gi0/2 switchport
show running-config interface Gi0/2
```

Expected results:

- [ ] `Gi0/2` is operating as a static access port.
- [ ] The configured data access VLAN is VLAN 10.
- [ ] The configured voice VLAN is VLAN 20.
- [ ] The interface is not operating as a conventional trunk.
- [ ] The interface is administratively enabled.
- [ ] No conflicting switchport configuration is present.

---

### Step 4 — Verify Server Access Port

Run on the switch.

```bash
show interfaces Gi0/3 switchport
show running-config interface Gi0/3
```

Expected results:

- [ ] `Gi0/3` is operating as a static access port.
- [ ] The configured access VLAN is VLAN 30.
- [ ] The interface is not operating as a trunk.
- [ ] The interface is administratively enabled.
- [ ] No conflicting switchport configuration is present.

---

### Step 5 — Verify Interface State

Run on the switch.

```bash
show interfaces status
```

Expected results:

- [ ] Each configured access port is administratively enabled.
- [ ] Connected endpoint-facing interfaces show the expected operational state.
- [ ] Standard access ports report the expected access VLAN.
- [ ] No target access port remains assigned to an unintended or parking VLAN.

---

## If Validation Fails

If post-configuration validation does not produce the expected results:

- Identify which validation step failed.
- Confirm that the required VLAN exists and is active.
- Confirm that the target interface is operating as a Layer 2 access port.
- Confirm that the configured access VLAN matches the intended endpoint network.
- For phone-facing ports, confirm that the configured voice VLAN matches the intended voice network.
- Review the related interface configuration for missing, incorrect, or conflicting settings.
- Correct the affected configuration.
- Re-run the failed validation step.
- If the issue cannot be resolved, return the affected interface or VLAN configuration to its previous known-good state before continuing.

---

## Backout Considerations

Changing VLAN membership, access-port configuration, or voice VLAN assignment can interrupt Layer 2 connectivity for connected endpoints.

Before removing or changing the configuration, confirm:

- The original VLAN and interface configuration is documented.
- The previous access VLAN assignment for each affected interface is known.
- Any previous voice VLAN assignment is documented.
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
| Last Updated     | 2026-09-20                                         |