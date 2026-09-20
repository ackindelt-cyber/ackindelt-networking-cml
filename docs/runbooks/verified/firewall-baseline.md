# Runbook — ASAv Baseline

## Overview

This runbook provides a structured process for preparing, configuring, and validating a Cisco ASAv firewall baseline.

The baseline establishes common device settings, management connectivity, and initial administrative readiness before production interfaces, firewall high availability, routing, NAT, access-control policy, or other security features are applied.

This runbook focuses on a greenfield Cisco ASAv deployment. The reference design uses a standalone ASAv with a dedicated management interface and environment-specific management addressing.

## Scope and Assumptions

This runbook assumes a greenfield deployment of a Cisco ASAv before the firewall is placed into production service.

This runbook assumes:

- The ASAv is being deployed as a new firewall appliance.
- The firewall is not currently carrying live production traffic.
- The expected ASAv image is installed and the device boots successfully.
- A dedicated management network and management IP addressing plan already exist.
- Administrative access to the firewall is available through the console during initial configuration.
- Production inside, outside, and other routed interfaces are configured separately.
- Active/standby failover configuration is handled separately.
- Static routing, NAT/PAT, access-control policy, and other production security features are handled separately.

This runbook does not cover brownfield migration of an existing production firewall, active/standby failover configuration, production interface configuration, routing, NAT/PAT, ACLs, VPN services, or advanced firewall security features.

---

## Reference Design

This runbook uses a single Cisco ASAv being prepared for deployment before production firewall interfaces, routing, policy, or high-availability features are configured.

### Devices

| Device | Role                |
|--------|---------------------|
| FW1    | Cisco ASAv firewall |

### Management Network

| Item                 | Value          |
|----------------------|----------------|
| Management interface | Management0/0  |
| Management subnet    | 10.10.60.0/24  |
| Management IP        | 10.10.60.11/24 |
| Management gateway   | 10.10.60.1     |

**Note:** The values shown in this reference design are examples. Replace management addressing and other environment-specific values with those appropriate to the target deployment.

---

## Prerequisites and Pre-Checks

Before configuring the ASAv baseline, confirm that the device and required deployment information are available.

### Prerequisites

- [ ] The ASAv is powered on and accessible through the console.
- [ ] The expected ASAv image is installed and the firewall boots successfully.
- [ ] The management interface to be used for administrative access has been identified.
- [ ] The management subnet, management IP address, and management gateway have been assigned.
- [ ] The firewall is not currently carrying live production traffic.

### Baseline Verification

Run these commands before applying the ASAv baseline configuration.

```bash
show version
show running-config
show interface ip brief
show route
```

### Expected Baseline Results

- [ ] The ASAv boots successfully and is running the expected software image.
- [ ] No unexpected or previously configured production settings are present in the running configuration.
- [ ] Interface state matches the expected greenfield deployment condition.
- [ ] No unexpected IP addressing is configured on production interfaces.
- [ ] No unexpected routes are present before baseline configuration.

---

## Configuration Procedure

Use this procedure to configure the ASAv baseline.

### Configuration Notes

- Apply the baseline before configuring production inside, outside, failover, or other traffic-carrying interfaces.
- Replace all example management IP addresses, interface values, and gateway values with those defined for the target deployment.
- Use the dedicated management interface for administrative access during the initial deployment.
- Save the configuration only after post-configuration validation confirms the ASAv is in the expected state.

---

### Step 1 — Configure ASAv Baseline

```bash
enable
configure terminal

hostname FW1 # Sets hostname on device

interface Management0/0 # Target the dedicated management interface
nameif MANAGEMENT # Assigns the logical name MANAGEMENT to the interface
ip address 10.10.60.11 255.255.255.0 # Sets the management IP address
management-only # Restricts the interface to management traffic
no shutdown # Ensures the management interface is administratively enabled
exit # Return to global configuration mode

end
write memory
```

---

## Post-Configuration Validation

Use this section to confirm that the ASAv baseline is configured correctly and operating in the expected state after implementation.

### Step 1 — Verify Management Interface State

Run on the ASAv.

```bash
show interface Management0/0
show interface ip brief
```

Expected results:

- [ ] `Management0/0` is administratively enabled.
- [ ] The management interface has the expected IP address.
- [ ] The interface state matches the expected physical connectivity.

---

### Step 2 — Verify Management Interface Configuration

Run on the ASAv.

```bash
show running-config interface Management0/0
```

Expected results:

- [ ] The interface is named `MANAGEMENT`.
- [ ] The expected management IP address and subnet mask are configured.
- [ ] `management-only` is configured on the dedicated management interface.

---

### Step 3 — Verify Baseline Device Configuration

Run on the ASAv.

```bash
show running-config hostname
show running-config
```

Expected results:

- [ ] The expected hostname is configured.
- [ ] No unintended production interfaces, routes, NAT rules, ACLs, or failover configuration are present.
- [ ] The running configuration matches the intended greenfield baseline.

---

## Backout Considerations

The ASAv baseline is intended for greenfield deployment before the firewall is placed into production service.

Before removing or changing the baseline configuration, confirm:

- The original configuration or startup state is documented.
- The management interface addressing is documented.
- Any management access dependencies are understood before removing or changing the management interface configuration.
- The ASAv can be returned to its previous known-good state.

For lab use, restore the last known working configuration before continuing additional validation.

---

## Document Metadata

| Field            | Value                                  |
|------------------|----------------------------------------|
| Author           | Aaron Kindelt                          |
| Category         | Configuration                          |
| Technology       | Cisco ASAv Firewall                    |
| Applies To       | Cisco ASAv virtual firewalls           |
| Primary Use Case | Greenfield ASAv baseline configuration |
| Version          | 1.0                                    |
| Last Updated     | 2026-09-10                             |
