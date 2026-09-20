# Runbook — Core Switch Baseline

## Overview

This runbook provides a structured process for preparing, configuring, and validating a Layer 3 core/distribution switch baseline.

The baseline establishes common device settings, Layer 3 forwarding capability, management connectivity, and unused-port security before production VLANs, routing, redundancy, or uplink-specific configuration are applied.

This runbook focuses on a traditional enterprise collapsed core/distribution switch. The reference design uses a standalone Cisco IOS multilayer switch with a dedicated management VLAN, a parking VLAN for unused interfaces, and Layer 3 routing enabled for later SVI, HSRP, and upstream routing configuration.

## Scope and Assumptions

This runbook assumes a greenfield deployment of a Layer 3 core/distribution switch before the device is placed into production service.

This runbook assumes:

- The switch is being deployed as a new multilayer core/distribution switch.
- The switch is not currently carrying live production traffic.
- The device runs Cisco IOS, IOSvL2, or a functionally similar IOS-based operating system with Layer 3 switching capability.
- Layer 3 routing is enabled on the switch.
- A dedicated management VLAN and management IP addressing plan already exist.
- A designated parking VLAN is available for unused interfaces.
- Production SVIs, HSRP, trunking, EtherChannels, firewall transit links, and routing-specific configuration are handled separately.
- Production endpoint access-port configuration is handled at the access layer.

This runbook does not cover brownfield conversion of an existing production core, live traffic migration, HSRP, production VLAN gateway configuration, inter-core trunking, EtherChannel, firewall transit configuration, static or dynamic routing, or advanced security and management services.

---

## Reference Design

This runbook uses a single Layer 3 core/distribution switch being prepared for deployment in an enterprise collapsed-core design.

### Devices

| Device | Role                             |
|--------|----------------------------------|
| CORE1  | Layer 3 core/distribution switch |

### Baseline VLANs

| VLAN     | Purpose                            |
|----------|------------------------------------|
| VLAN 50  | Switch management                  |
| VLAN 999 | Parking VLAN for unused interfaces |

### Feature Design Values

| Item              | Value                              |
|-------------------|------------------------------------|
| Management VLAN   | 50                                 |
| Management subnet | 10.10.50.0/24                      |
| Management SVI    | 10.10.50.12/24                      |
| Parking VLAN      | 999                                |
| Layer 3 routing   | Enabled                            |
| Unused interfaces | Assigned to VLAN 999 and shut down |

**Note:** The values shown in this reference design are examples. Replace VLAN IDs, IP addressing, interface ranges, and other environment-specific values with those appropriate to the target deployment.

---

## Prerequisites and Pre-Checks

Before configuring the core switch baseline, confirm that the device and required deployment information are available.

### Prerequisites

- [ ] The switch is powered on and accessible through the console.
- [ ] The expected Cisco IOS image is installed and the device boots successfully.
- [ ] The device supports Layer 3 switching and `ip routing`.
- [ ] The management VLAN, management subnet, and switch management IP have been assigned.
- [ ] The parking VLAN ID has been defined.
- [ ] The intended interface roles have been identified.

### Baseline Verification

Run these commands before applying the core switch baseline configuration.

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
- [ ] No unexpected Layer 3, management, or routing configuration is present on the switch.

---

## Configuration Procedure

Use this procedure to configure the core switch baseline.

### Configuration Notes

- Apply the baseline before configuring production SVIs, HSRP, inter-core trunks, EtherChannels, firewall transit links, or routing-specific features.
- Replace all example VLAN IDs, IP addresses, interface ranges, and other environment-specific values with those defined for the target deployment.
- In the reference design, all physical interfaces are initially placed in the parking VLAN and administratively disabled. Later uplink and routed-interface procedures remove required interfaces from the parking configuration as they are placed into service.
- Save the configuration only after post-configuration validation confirms the switch is in the expected state.

---

### Step 1 — Configure Core Switch Baseline

```bash
enable
configure terminal

hostname CORE1 # Sets hostname on device
no ip domain-lookup # Prevents DNS lookup on mistyped configs
ip routing # Enables Layer 3 routing on the switch

line console 0 # Target console interface 0
logging synchronous # Forces the CLI to retype in progress commands during logging events
exit # Return to global configuration mode

vlan 50 # Create and target VLAN 50
name MANAGEMENT # Names target VLAN
exit # Return to global configuration mode

interface Vlan50 # Create and target SVI
ip address 10.10.50.12 255.255.255.0 # Sets IP address on target SVI
no shutdown # Ensures target SVI is administratively enabled
exit # Return to global configuration mode

vlan 999 # Creates and targets VLAN 999
name PARKING # Names target VLAN
exit # Return to global configuration mode

interface range Gi0/0 - 3, Gi1/0 - 3, Gi2/0 - 3, Gi3/0 - 3 # Targets unused interfaces
switchport mode access # Sets all interfaces to access mode
switchport access vlan 999 # Places all targeted interfaces in VLAN 999
shutdown # Administratively shuts down all interfaces
exit # Return to global configuration mode

end
write memory
```

---

## Post-Configuration Validation

Use this section to confirm that the core switch baseline is configured correctly and operating in the expected state after implementation.

### Step 1 — Verify Layer 3 Operating State

Run on the core switch.

```bash
show running-config | include ^ip routing
show ip route
```

Expected results:

- [ ] `ip routing` is present in the running configuration.
- [ ] The switch is operating with Layer 3 routing enabled.
- [ ] The routing table is available with no unexpected production routes present at this stage.

---

### Step 2 — Verify Management Configuration

Run on the core switch.

```bash
show running-config | section interface Vlan50
show ip interface brief
```

Expected results:

- [ ] VLAN 50 is configured as the management SVI.
- [ ] The management SVI has the expected IP address and subnet mask.
- [ ] The management SVI is administratively enabled.
- [ ] No unexpected management interfaces are configured.

---

### Step 3 — Verify Parking VLAN and Unused Interfaces

Run on the core switch.

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

If the issue is not resolved after these checks, stop and document the failed validation results before continuing with deeper troubleshooting.

---

## Backout Considerations

The core switch baseline is intended for greenfield deployment before the switch is placed into production service.

Before removing or changing the baseline configuration, confirm:

- The original configuration or startup state is documented.
- The management VLAN and management IP addressing are documented.
- Any interfaces removed from the parking VLAN are accounted for.
- The switch can be returned to its previous known-good state.
- Any routing configuration added after the baseline is identified before reverting `ip routing` or related Layer 3 settings.

For lab use, restore the last known working configuration before continuing additional validation.

---

## Document Metadata

| Field            | Value                                                      |
|------------------|------------------------------------------------------------|
| Author           | Aaron Kindelt                                              |
| Category         | Configuration                                              |
| Technology       | Cisco IOS Layer 3 Switching                                |
| Applies To       | Cisco IOS / IOSvL2 multilayer core/distribution switches   |
| Primary Use Case | Greenfield core/distribution switch baseline configuration |
| Version          | 1.0                                                        |
| Last Updated     | 2026-09-10                                                 |