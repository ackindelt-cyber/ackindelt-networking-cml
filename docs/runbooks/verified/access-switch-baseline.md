# Runbook — Access Switch Baseline

## Overview

This runbook provides a structured process for preparing, configuring, and validating a Layer 2 access switch baseline.

The baseline establishes common device settings, management connectivity, Layer 2 operating behavior, spanning-tree edge protections, and unused-port security before production VLAN assignments or uplink-specific configuration are applied.

This runbook focuses on a traditional Layer 2 enterprise access switch. The reference design uses a standalone Cisco IOS switch with a dedicated management VLAN, a parking VLAN for unused interfaces, and upstream connectivity provided separately through trunk or EtherChannel configuration.

## Scope and Assumptions

This runbook assumes a greenfield deployment of a Layer 2 enterprise access switch before the device is placed into production service.

This runbook assumes:

* The switch is being deployed as a new Layer 2 access switch.
* The switch is not currently carrying live production traffic.
* The device runs Cisco IOS, IOSvL2, or a functionally similar IOS-based operating system.
* Layer 3 routing for user and service VLANs is performed upstream rather than on the access switch.
* A dedicated management VLAN and management IP addressing plan already exist.
* A designated parking VLAN is available for unused access interfaces.
* Uplink trunking and EtherChannel configuration are handled separately.
* Production endpoint VLAN assignments are handled separately.

This runbook does not cover brownfield conversion of an existing production switch, migration of live endpoint ports, uplink trunk or EtherChannel configuration, production access-port VLAN assignments, or advanced access-layer security features such as 802.1X, DHCP snooping, Dynamic ARP Inspection, or IP Source Guard.

---

## Reference Design

This runbook uses a single Layer 2 access switch being prepared for deployment in an enterprise access layer.

### Devices

| Device | Role                  |
|--------|-----------------------|
| ASW1   | Layer 2 access switch |

### Baseline VLANs

| VLAN                     | Purpose                            |
|--------------------------|------------------------------------|
| VLAN 50                  | Switch management                  |
| VLAN 999                 | Parking VLAN for unused interfaces |

### Feature Design Values

| Item                       | Value                              |
|----------------------------|------------------------------------|
| Management VLAN            | 50                                 |
| Management subnet          | 10.10.50.0/24                      |
| Management SVI             | 10.10.50.11/24                     |
| Management default gateway | 10.10.50.1                         |
| Parking VLAN               | 999                                |
| Layer 3 routing            | Disabled                           |
| PortFast default           | Enabled                            |
| BPDU Guard default         | Enabled                            |
| Unused interfaces          | Assigned to VLAN 999 and shut down |

**Note:** The values shown in this reference design are examples. Replace VLAN IDs, IP addressing, interface ranges, and other environment-specific values with those appropriate to the target deployment.

---

## Prerequisites and Pre-Checks

Before configuring the access switch baseline, confirm that the device and required deployment information are available.

### Prerequisites

- [ ] The switch is powered on and accessible through the console.
- [ ] The expected Cisco IOS image is installed and the device boots successfully.
- [ ] The management VLAN, management subnet, switch management IP, and default gateway have been assigned.
- [ ] The parking VLAN ID for unused interfaces has been defined.
- [ ] The intended uplink and endpoint interface roles have been identified.

### Baseline Verification

Run these commands before applying the access switch baseline configuration.

```bash
show version
show running-config
show interfaces status
show vlan brief
```

### Expected Baseline Results

- [ ] The switch boots successfully and is running the expected IOS image.
- [ ] No unexpected or previously configured settings are present in the running configuration.
- [ ] Interface status matches the expected greenfield deployment state, with only intended physical connections active.
- [ ] Only default or expected VLANs are present before baseline configuration.
- [ ] No unexpected Layer 3 or management configuration is present on the switch.

---

## Configuration Procedure

Use this procedure to configure the access switch baseline.

### Configuration Notes

- Apply the baseline before configuring production access ports, uplink trunks, or EtherChannels.
- Replace all example VLAN IDs, IP addresses, interface ranges, and gateway values with those defined for the target deployment.
- In the reference design, all physical interfaces are initially placed in the parking VLAN and administratively disabled. Later access-port and uplink procedures remove required interfaces from the parking configuration as they are placed into service.
- Save the configuration only after post-configuration validation confirms the switch is in the expected state.

---

### Step 1 — Configure Access Switch Baseline

```bash
enable # Enter privileged EXEC mode
configure terminal # Enter global configuration mode

hostname ASW1 # Sets hostname on device
no ip domain-lookup # Prevents DNS lookup on mistyped configs
no ip routing # Ensures device is operating as a L2 switch
spanning-tree portfast default # Enables PortFast by default on Layer 2 access ports
spanning-tree portfast bpduguard default # Enables BPDU Guard by default on PortFast-enabled ports

line console 0 # Target console interface 0
logging synchronous # Forces the CLI to retype in progress commands during logging events
exit # Return to global configuration mode

vlan 50 # Create and target VLAN 50
name MANAGEMENT # Names target VLAN
exit # Return to global configuration mode

interface Vlan50 # Create and target SVI
ip address 10.10.50.11 255.255.255.0 # Sets IP address on target SVI
no shutdown # Ensures target SVI is administratively enabled
exit # Return to global configuration mode

ip default-gateway 10.10.50.1 # Sets default gateway for ASW1's management SVI

vlan 999 # Creates and targets VLAN 999
name PARKING # Names target VLAN
exit # Return to global configuration mode

interface range Gi0/0 - 3, Gi1/0 - 3, Gi2/0 - 3, Gi3/0 - 3 # Targets unused interfaces
switchport mode access # Sets all interfaces to access mode
switchport access vlan 999 # Places all targeted interfaces in VLAN 999
shutdown # Administratively shuts down all interfaces
exit # Return to global configuration mode

end # Exit configuration mode
write memory # Saves current running configuration to memory
```

---

## Post-Configuration Validation

Use this section to confirm that the access switch baseline is configured correctly and operating in the expected state after implementation.

### Step 1 — Verify Layer 2 Operating State

Run on the access switch.

```bash
show running-config | include ^no ip routing
```

Expected results:

- [ ] `no ip routing` is present in the running configuration.
- [ ] The switch is operating as a Layer 2 access switch.

---

### Step 2 — Verify Management Configuration

Run on the access switch.

```bash
show running-config | section interface Vlan50
show running-config | include ^ip default-gateway
show ip interface brief
```

Expected results:

- [ ] VLAN 50 is configured as the management SVI.
- [ ] The management SVI has the expected IP address and subnet mask.
- [ ] The management SVI is administratively enabled.
- [ ] The configured default gateway matches the management VLAN gateway.

---

### Step 3 — Verify Spanning-Tree Edge Protections

Run on the access switch.

```bash
show spanning-tree summary
```

Expected results:

- [ ] PortFast default is enabled.
- [ ] BPDU Guard default is enabled.
- [ ] The switch is using the expected spanning-tree mode.

---

### Step 4 — Verify Parking VLAN and Unused Interfaces

Run on the access switch.

```bash
show vlan brief
show interfaces status
show interfaces switchport
```

Expected results:

- [ ] VLAN 999 exists and is named `PARKING`.
- [ ] Unused interfaces are assigned to VLAN 999.
- [ ] Unused interfaces are configured as access ports.
- [ ] Unused interfaces are administratively disabled.

---

### If Validation Fails

If post-configuration validation does not produce the expected results:

- Identify which validation step failed.
- Review the related configuration for missing, incorrect, or conflicting settings.
- Correct the affected configuration.
- Re-run the failed validation step.
- If the issue cannot be resolved, return the device to its previous known-good state before continuing.

---

## Backout Considerations

The access switch baseline is intended for greenfield deployment before the switch is placed into production service.

Before removing or changing the baseline configuration, confirm:

- The original configuration or startup state is documented.
- The management addressing and VLAN assignments are documented.
- Any interfaces removed from the parking VLAN are accounted for.
- The switch can be returned to its previous known-good state.

For lab use, restore the last known working configuration before continuing additional validation.

---

## Document Metadata

| Field            | Value                                           |
|------------------|-------------------------------------------------|
| Author           | Aaron Kindelt                                   |
| Category         | Configuration                                   |
| Technology       | Cisco IOS Access Switching                      |
| Applies To       | Cisco IOS / IOSvL2 Layer 2 access switches      |
| Primary Use Case | Greenfield access switch baseline configuration |
| Version          | 1.0                                             |
| Last Updated     | 2026-09-10                                      |
