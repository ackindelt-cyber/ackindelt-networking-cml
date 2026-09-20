# Runbook — Routed Interface Addressing

## Overview

This runbook provides a structured process for configuring and validating IPv4 addressing on physical Layer 3 interfaces on Cisco IOS routers and multilayer switches.

Routed interfaces provide point-to-point or transit Layer 3 connectivity between network devices. On Cisco IOS routers, physical interfaces operate as Layer 3 interfaces by default. On multilayer switches, a physical switchport must be converted to a routed interface before an IP address can be assigned.

This runbook focuses on assigning IPv4 addresses to physical routed interfaces, enabling the interfaces, and validating directly connected Layer 3 connectivity. The example reference design uses a point-to-point transit network between two Cisco IOS Layer 3 devices.

## Scope and Assumptions

This runbook assumes:

- The participating Cisco IOS Layer 3 devices are powered on and accessible for configuration.
- The required device baselines have already been applied.
- The physical interfaces to be configured have been identified.
- The IPv4 address and subnet mask for each routed interface have been defined.
- The selected interfaces are intended for Layer 3 use and are not required as Layer 2 switchports.
- On multilayer switches, IP routing has already been enabled by the applicable device baseline.
- Static routing, default routing, dynamic routing, ACLs, NAT, and other Layer 3 services are configured separately.

This runbook does not cover SVIs, loopback interfaces, subinterfaces, router-on-a-stick, IPv6 addressing, EtherChannel routed ports, or tunnel interfaces.

---

## Reference Design

This runbook uses two Cisco IOS Layer 3 devices connected by a physical point-to-point routed link.

The link uses the `10.0.0.0/30` network. Each device assigns an IPv4 address directly to its physical interface.

### Devices

| Device | Role                    | Purpose                                |
|--------|-------------------------|----------------------------------------|
| R1     | Cisco IOS router        | First endpoint of the routed link      |
| SW1    | Cisco IOS multilayer switch | Second endpoint of the routed link |

### Network Summary

| Network     | Purpose                  |
|-------------|--------------------------|
| 10.0.0.0/30 | Point-to-point transit   |

### Interface Summary

| Device | Interface | IPv4 Address | Interface Type |
|--------|-----------|--------------|----------------|
| R1     | Gi0/0     | 10.0.0.1/30  | Routed         |
| SW1    | Gi0/0     | 10.0.0.2/30  | Routed         |

**Note:** The values shown in this reference design are examples. Replace device names, interface numbers, IPv4 addresses, and subnet masks with those appropriate to the target deployment.

---

## Prerequisites and Pre-Checks

Before configuring routed interface addressing, confirm that the target interfaces and IPv4 addressing information are ready for implementation.

### Prerequisites

- [ ] The participating devices are powered on and accessible for configuration.
- [ ] The required device baselines have already been applied.
- [ ] The target physical interfaces have been identified.
- [ ] The IPv4 address and subnet mask for each interface have been defined.
- [ ] The selected interfaces are intended for Layer 3 use.
- [ ] The selected interfaces are not members of an EtherChannel.
- [ ] No conflicting IPv4 configuration is present on the target interfaces.

### Baseline Verification

### Baseline Verification

Run the applicable commands before applying routed-interface configuration.

On Cisco IOS routers:

```bash
show ip interface brief
show running-config interface <interface>
```

On Cisco IOS multilayer switches:

```bash
show ip interface brief
show interfaces status
show running-config interface <interface>
```

### Expected Baseline Results

- [ ] The target physical interfaces are present.
- [ ] The target interfaces are available for Layer 3 configuration.
- [ ] No conflicting IPv4 addressing is present.
- [ ] The interfaces are not members of an unintended EtherChannel.
- [ ] The intended IPv4 addressing matches the target design.

---

## Configuration Procedure

Use this procedure to configure IPv4 addressing on physical routed interfaces.

### Configuration Notes

- Cisco IOS router interfaces operate as Layer 3 interfaces by default.
- Multilayer-switch interfaces must be converted from Layer 2 switchports using `no switchport`.
- Configure the correct IPv4 address and subnet mask before enabling the interface.
- Both ends of a directly connected link must use addresses from the same IPv4 subnet.
- Replace example interface numbers and IPv4 addresses with those defined for the target deployment.

---

### Step 1 — Configure Router Routed Interface

Run on R1.

```bash
enable # Enter privileged EXEC mode
configure terminal # Enter global configuration mode

interface gi0/0 # Target the physical routed interface
ip address 10.0.0.1 255.255.255.252 # Assign the interface IPv4 address
no shutdown # Administratively enable the interface

end # Return to privileged EXEC mode
write memory # Save the running configuration
```

---

### Step 2 — Configure Multilayer Switch Routed Interface

Run on SW1.

```bash
enable # Enter privileged EXEC mode
configure terminal # Enter global configuration mode

interface gi0/0 # Target the physical interface
no switchport # Convert the interface from Layer 2 to Layer 3 operation
ip address 10.0.0.2 255.255.255.252 # Assign the interface IPv4 address
no shutdown # Administratively enable the interface

end # Return to privileged EXEC mode
write memory # Save the running configuration
```

---

## Post-Configuration Validation

Use this section to confirm that the routed interfaces are configured correctly and that the directly connected Layer 3 link is operational.

### Step 1 — Verify Interface Addressing and State

Run on both devices.

```bash
show ip interface brief
```

Expected results:

- [ ] The target interface has the expected IPv4 address.
- [ ] The interface is administratively enabled.
- [ ] The interface and line protocol are up when the peer link is operational.
- [ ] No unexpected IPv4 addressing is present on the target interface.

---

### Step 2 — Verify Interface Configuration

Run the applicable command on each device.

```bash
show running-config interface gi0/0
```

Expected results:

- [ ] The configured IPv4 address and subnet mask match the reference design.
- [ ] The interface is not administratively shut down.
- [ ] The multilayer-switch interface contains `no switchport`.
- [ ] No conflicting interface configuration is present.

---

### Step 3 — Verify Connected Route

Run on both devices.

```bash
show ip route connected
```

Expected results:

- [ ] The `10.0.0.0/30` network appears as directly connected.
- [ ] The connected route references the expected physical interface.
- [ ] No unexpected connected route conflicts are present.

---

### Step 4 — Verify Directly Connected Reachability

From R1:

```bash
ping 10.0.0.2
```

From SW1:

```bash
ping 10.0.0.1
```

Expected results:

- [ ] R1 can reach the directly connected SW1 interface.
- [ ] SW1 can reach the directly connected R1 interface.
- [ ] Bidirectional Layer 3 connectivity is operational across the physical link.

---

## If Validation Fails

If post-configuration validation does not produce the expected results:

- Identify which validation step failed.
- Confirm that both interfaces have the correct IPv4 address and subnet mask.
- Confirm that both interfaces are administratively enabled.
- Confirm that both ends of the link use addresses from the same IPv4 subnet.
- On multilayer switches, confirm that the interface is operating as a routed port with `no switchport`.
- Confirm that the physical link is operational.
- Review the interface configuration for missing, incorrect, or conflicting settings.
- Correct the affected configuration.
- Re-run the failed validation step.
- If the issue cannot be resolved, return the affected interfaces to their previous known-good state before continuing.

---

## Backout Considerations

Changing or removing routed-interface addressing can interrupt Layer 3 connectivity across the affected link.

Before removing or changing the configuration, confirm:

- The original interface configuration is documented.
- The previous IPv4 address, subnet mask, and administrative state are known.
- The previous Layer 2 or Layer 3 interface mode is documented.
- Any routing or services dependent on the routed link are understood.
- Both ends of the link can be returned to their previous known-good configuration if required.

For lab use, restore the last known working configuration before continuing additional validation.

---

## Document Metadata

| Field            | Value                                           |
|------------------|-------------------------------------------------|
| Author           | Aaron Kindelt                                   |
| Category         | Configuration                                   |
| Technology       | Cisco IOS Layer 3 Interface Configuration       |
| Applies To       | Cisco IOS routers and multilayer switches       |
| Primary Use Case | Physical routed interface IPv4 addressing       |
| Version          | 1.0                                             |
| Last Updated     | 2026-09-20                                      |