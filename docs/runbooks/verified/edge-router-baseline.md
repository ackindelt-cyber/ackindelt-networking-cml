# Runbook — Edge Router Baseline

## Overview

This runbook provides a structured process for preparing, configuring, and validating a Layer 3 edge router baseline.

The edge router provides routed connectivity between an upstream service-provider or simulated ISP network and downstream enterprise security infrastructure. The baseline establishes common device settings and Layer 3 forwarding capability before production-facing interfaces, routing, NAT, security policy, or other edge-specific features are applied.

This runbook focuses on a greenfield Cisco IOS edge router. The reference design uses a standalone router positioned between an upstream ISP connection and a downstream firewall outside transit network.

## Scope and Assumptions

This runbook assumes a greenfield deployment of a Layer 3 edge router before the device is placed into production service.

This runbook assumes:

- The router is being deployed as a new enterprise edge router.
- The router is not currently carrying live production traffic.
- The device runs Cisco IOS, IOSv, or a functionally similar IOS-based operating system.
- Layer 3 routing capability is available on the device.
- Upstream ISP-facing and downstream firewall-transit interfaces have been identified.
- Interface IP addressing is configured separately.
- Static routes, default routes, or dynamic routing are configured separately.
- NAT/PAT, ACLs, VPN services, and other security features are configured separately.

This runbook does not cover ISP-facing interface configuration, firewall-transit interface configuration, static or dynamic routing, NAT/PAT, ACLs, VPN services, or advanced edge security features.

---

## Reference Design

This runbook uses a single Cisco IOS edge router being prepared for deployment before production interface addressing, routing, NAT, or security features are configured.

### Devices

| Device | Role                   | Purpose                                      |
|--------|------------------------|----------------------------------------------|
| EDGE1  | Enterprise edge router | Provides upstream and downstream routed connectivity |

### Feature Design Values

| Item                 | Value           |
|----------------------|-----------------|
| Hostname             | EDGE1           |
| Device role          | Layer 3 routing |
| DNS lookup           | Disabled        |
| Console log handling | Synchronous     |

**Note:** Production interface addressing, routing, NAT/PAT, ACLs, and other edge-specific services are configured in separate procedures.

---

## Prerequisites and Pre-Checks

Before configuring the edge router baseline, confirm that the device and required deployment information are available.

### Prerequisites

- [ ] The router is powered on and accessible through the console.
- [ ] The expected Cisco IOS image is installed and the device boots successfully.
- [ ] The device is intended to operate as an enterprise edge router.
- [ ] Production interface addressing and routing requirements have been documented for later configuration.
- [ ] The router is not currently carrying live production traffic.

### Baseline Verification

Run these commands before applying the edge router baseline configuration.

```bash
show version
show running-config
show ip interface brief
show ip route
```

### Expected Baseline Results

- [ ] The router boots successfully and is running the expected IOS image.
- [ ] No unexpected or previously configured production settings are present in the running configuration.
- [ ] Interface state matches the expected greenfield deployment condition.
- [ ] No unexpected IP addressing is configured on production interfaces.
- [ ] No unexpected routes are present before baseline configuration.

---

## Configuration Procedure

Use this procedure to configure the edge router baseline.

### Configuration Notes

- Apply the baseline before configuring production interface addressing or routing.
- Replace the example hostname with the value defined for the target deployment.
- Production interfaces remain unaddressed until their respective interface configuration procedures are applied.
- Routing, NAT/PAT, ACLs, and other edge-specific services are configured separately.
- Save the configuration after the baseline has been applied.

---

### Step 1 — Configure Edge Router Baseline

```bash
enable
configure terminal

hostname EDGE1 # Sets hostname on device
no ip domain-lookup # Prevents DNS lookup on mistyped commands

line console 0 # Target console interface 0
logging synchronous # Forces the CLI to retype in-progress commands during logging events
exit # Return to global configuration mode

end
write memory
```

---

## Post-Configuration Validation

Use this section to confirm that the edge router baseline is configured correctly and operating in the expected state after implementation.

### Step 1 — Verify Baseline Device Configuration

Run on the edge router.

```bash
show running-config | include ^hostname
show running-config | include ^no ip domain-lookup
show running-config | section line con
```

Expected results:

- [ ] The expected hostname is configured.
- [ ] `no ip domain-lookup` is configured.
- [ ] `logging synchronous` is configured under the console line.

---

### Step 2 — Verify Interface Baseline State

Run on the edge router.

```bash
show ip interface brief
```

Expected results:

- [ ] Production interfaces have no unexpected IP addressing.
- [ ] Interface administrative state matches the expected greenfield deployment condition.
- [ ] No unintended Layer 3 interfaces have been configured.

---

### Step 3 — Verify Routing Baseline State

Run on the edge router.

```bash
show ip route
show running-config
```

Expected results:

- [ ] The IPv4 routing table is available.
- [ ] No unexpected static or dynamic routes are present.
- [ ] No unintended interface addressing, NAT, ACL, or routing protocol configuration is present.
- [ ] The running configuration matches the intended greenfield edge router baseline.

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

The edge router baseline is intended for greenfield deployment before the router is placed into production service.

Before removing or changing the baseline configuration, confirm:

- The original configuration or startup state is documented.
- The router hostname and baseline administrative settings are documented.
- Any production interface addressing or routing added after the baseline is identified before reverting the device configuration.
- Any operational dependencies introduced after the baseline are understood.
- The router can be returned to its previous known-good state.

For lab use, restore the last known working configuration before continuing additional validation.

---

## Document Metadata

| Field            | Value                                     |
|------------------|-------------------------------------------|
| Author           | Aaron Kindelt                             |
| Category         | Configuration                             |
| Technology       | Cisco IOS Routing                         |
| Applies To       | Cisco IOS / IOSv edge routers             |
| Primary Use Case | Greenfield edge router baseline           |
| Version          | 1.0                                       |
| Last Updated     | 2026-09-10                                |
