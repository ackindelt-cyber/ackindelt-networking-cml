# Runbook — Outside Transit Switch Baseline

## Overview

This runbook provides a structured process for preparing, configuring, and validating a Layer 2 outside transit switch baseline.

The outside transit switch provides a shared Layer 2 segment between an upstream edge router and redundant firewall outside interfaces. The baseline establishes common device settings, Layer 2 operating behavior, the outside transit VLAN, and unused-port handling before the switch is placed into service.

This runbook focuses on a greenfield Cisco IOS Layer 2 switch used exclusively as an outside firewall transit switch. The reference design uses one outside transit VLAN connecting an edge router and two firewall outside interfaces.

## Scope and Assumptions

This runbook assumes a greenfield deployment of a Layer 2 outside transit switch before the device is placed into production service.

This runbook assumes:

- The switch is being deployed as a dedicated outside firewall transit switch.
- The switch is not currently carrying live production traffic.
- The device runs Cisco IOS, IOSvL2, or a functionally similar IOS-based operating system.
- Layer 3 routing is not performed on the outside transit switch.
- A dedicated outside transit VLAN has been defined for the edge router and firewall outside interfaces.
- The edge router and firewall outside interfaces use addressing from the same outside transit subnet.
- A designated parking VLAN is available for unused interfaces.
- Routing, firewall interface configuration, NAT/PAT, firewall policy, and firewall high availability are configured separately.

This runbook does not cover user access switching, inter-VLAN routing, trunking, EtherChannel, firewall configuration, routing between the firewall and edge router, NAT/PAT, or firewall security policy.

---

## Reference Design

This runbook uses a single Layer 2 outside transit switch providing a shared Ethernet segment between one edge router and two firewall outside interfaces.

### Devices

| Device | Role                   |
|--------|------------------------|
| EDGE1  | Customer edge router   |
| OS1    | Outside transit switch |
| FW1    | Primary firewall       |
| FW2    | Secondary firewall     |

### Outside Transit Network

| Item                   | Value         |
|------------------------|---------------|
| Outside transit VLAN   | 901           |
| Outside transit subnet | 10.10.90.0/29 |
| Edge router IP         | 10.10.90.1/29 |
| FW1 outside IP         | 10.10.90.2/29 |
| FW2 outside IP         | 10.10.90.3/29 |
| Parking VLAN           | 999           |

### Interface Summary

| Device | Interface            | Role                                     |
|--------|----------------------|------------------------------------------|
| OS1    | Gi0/0                | Edge router connection                   |
| OS1    | Gi0/1                | FW1 outside connection                   |
| OS1    | Gi0/2                | FW2 outside connection                   |
| OS1    | Remaining interfaces | Parking VLAN / administratively disabled |

### Feature Design Values

| Item                 | Value                              |
|----------------------|------------------------------------|
| Layer 3 routing      | Disabled                           |
| Outside-facing ports | Access ports in VLAN 901           |
| Unused interfaces    | Assigned to VLAN 999 and shut down |

**Note:** The values shown in this reference design are examples. Replace VLAN IDs, addressing, interface numbers, and other environment-specific values with those appropriate to the target deployment.

---

## Prerequisites and Pre-Checks

Before configuring the outside transit switch baseline, confirm that the device and required deployment information are available.

### Prerequisites

- [ ] The switch is powered on and accessible through the console.
- [ ] The expected Cisco IOS image is installed and the device boots successfully.
- [ ] The outside transit VLAN ID and subnet have been defined.
- [ ] The interfaces connecting to the edge router and firewall outside interfaces have been identified.
- [ ] The parking VLAN ID for unused interfaces has been defined.
- [ ] The switch is not currently carrying live production traffic.

### Baseline Verification

Run these commands before applying the outside transit switch baseline configuration.

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

Use this procedure to configure the outside transit switch baseline.

### Configuration Notes

- Apply the baseline before configuring the edge router or firewall interfaces connected to the outside transit segment.
- Replace all example VLAN IDs, interface ranges, and other environment-specific values with those defined for the target deployment.
- The interfaces connecting the edge router and firewall outside interfaces are configured as Layer 2 access ports in the outside transit VLAN.
- All remaining physical interfaces are placed in the parking VLAN and administratively disabled.
- Save the configuration after applying the baseline so the configured state is preserved for validation and later deployment steps.
---

### Step 1 — Configure Outside Transit Switch Baseline

```bash
enable
configure terminal

hostname OS1 # Sets hostname on device
no ip domain-lookup # Prevents DNS lookup on mistyped commands
no ip routing # Ensures device is operating as a Layer 2 switch

line console 0 # Target console interface 0
logging synchronous # Forces the CLI to retype in-progress commands during logging events
exit # Return to global configuration mode

vlan 901 # Create and target the outside transit VLAN
name OUTSIDE_TRANSIT # Names the outside transit VLAN
exit # Return to global configuration mode

interface range Gi0/0 - 2 # Target edge router and firewall outside interfaces
switchport mode access # Sets targeted interfaces to access mode
switchport access vlan 901 # Places targeted interfaces in the outside transit VLAN
no shutdown # Ensures targeted interfaces are administratively enabled
exit # Return to global configuration mode

vlan 999 # Create and target the parking VLAN
name PARKING # Names the parking VLAN
exit # Return to global configuration mode

interface range Gi0/3, Gi1/0 - 3, Gi2/0 - 3, Gi3/0 - 3 # Target unused interfaces
switchport mode access # Sets targeted interfaces to access mode
switchport access vlan 999 # Places targeted interfaces in the parking VLAN
shutdown # Administratively shuts down targeted interfaces
exit # Return to global configuration mode

end
write memory
```
---

## Post-Configuration Validation

Use this section to confirm that the outside transit switch baseline is configured correctly and operating in the expected state after implementation.

### Step 1 — Verify Layer 2 Operating State

Run on the outside transit switch.

```bash
show running-config | include ^no ip routing
```

Expected results:

- [ ] `no ip routing` is present.
- [ ] The switch is operating as a Layer 2 device.

---

### Step 2 — Verify Outside Transit VLAN

Run on the outside transit switch.

```bash
show vlan brief
```

Expected results:

- [ ] VLAN 901 exists.
- [ ] VLAN 901 is named `OUTSIDE_TRANSIT`.
- [ ] The edge router and firewall-facing interfaces are assigned to VLAN 901.
- [ ] VLAN 999 exists and is named `PARKING`.

---

### Step 3 — Verify Outside Transit Interface Configuration

Run on the outside transit switch.

```bash
show interfaces status
show interfaces switchport
```

Expected results:

- [ ] The edge router and firewall-facing interfaces are configured as Layer 2 access ports.
- [ ] The edge router and firewall-facing interfaces use VLAN 901 as their access VLAN.
- [ ] The required outside transit interfaces are administratively enabled.
- [ ] Interface operational state matches the expected physical connectivity.

---

### Step 4 — Verify Parking VLAN Configuration

Run on the outside transit switch.

```bash
show vlan brief
show interfaces status
show interfaces switchport
```

Expected results:

- [ ] All unused physical interfaces are assigned to VLAN 999.
- [ ] All unused physical interfaces are configured as Layer 2 access ports.
- [ ] All unused physical interfaces are administratively disabled.
- [ ] No required outside transit interface is assigned to the parking VLAN.

---

### Step 5 — Verify Baseline Device Configuration

Run on the outside transit switch.

```bash
show running-config | include ^hostname
show running-config
```

Expected results:

- [ ] The expected hostname is configured.
- [ ] `no ip domain-lookup` is configured.
- [ ] Console logging synchronization is configured.
- [ ] No unintended Layer 3 interfaces, routing configuration, trunks, EtherChannels, or production VLAN configuration are present.
- [ ] The running configuration matches the intended outside transit switch baseline.

### If Validation Fails

If post-configuration validation does not produce the expected results:

- Identify which validation step failed.
- Review the related configuration for missing, incorrect, or conflicting settings.
- Correct the affected configuration.
- Re-run the failed validation step.
- If the issue cannot be resolved, return the device to its previous known-good state before continuing.

---

## Backout Considerations

The outside transit switch baseline is intended for greenfield deployment before the switch is placed into production service.

Before removing or changing the baseline configuration, confirm:

- The original configuration or startup state is documented.
- The outside transit VLAN and connected interfaces are documented.
- The edge router and firewall outside-interface dependencies are understood.
- Any interfaces removed from the parking VLAN are accounted for.
- The switch can be returned to its previous known-good state.

For lab use, restore the last known working configuration before continuing additional validation.

---

## Document Metadata

| Field            | Value                                       |
|------------------|---------------------------------------------|
| Author           | Aaron Kindelt                               |
| Category         | Configuration                               |
| Technology       | Cisco IOS Layer 2 Switching                 |
| Applies To       | Cisco IOS / IOSvL2 outside transit switches |
| Primary Use Case | Greenfield outside transit switch baseline  |
| Version          | 1.0                                         |
| Last Updated     | 2026-09-10                                  |
