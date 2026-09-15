# Runbook — ASAv Routed Firewall and PAT
 
## Overview
 
This runbook provides a structured process for configuring and validating a Cisco ASAv as a routed firewall providing connectivity between trusted internal networks and an external network.
 
Routed firewall operation places the ASAv directly in the Layer 3 forwarding path between inside and outside networks. Interface addressing, security levels, routing, and network address translation work together to control and forward traffic through the firewall.
 
This runbook focuses on configuring ASAv routed interfaces, basic routing, and dynamic Port Address Translation (PAT) for inside-to-outside traffic. The example reference design uses a single ASAv positioned between an internal Layer 3 network and an upstream edge router.
 
High-availability configuration and active/standby failover are handled separately.


## Scope and Assumptions
 
This runbook assumes:
 
- The ASAv is powered on and accessible for configuration.
- The required ASAv baseline has already been applied.
- The physical or virtual interfaces used for inside and outside connectivity have been identified.
- IPv4 addressing for the inside and outside interfaces has been defined.
- The upstream next-hop address has been defined.
- The internal network or networks requiring outbound translation have been identified.
- The ASAv will operate in routed firewall mode.
- Inside-to-outside traffic is permitted by the intended security policy.
- High-availability and active/standby failover are configured separately.
- Layer 2 switching, VLAN creation, HSRP, and upstream routing are configured separately on adjacent devices.
 
This runbook does not cover transparent firewall mode, VPN configuration, advanced access-control policy, object groups beyond those required for PAT, site-to-site VPNs, dynamic routing protocols, or active/standby failover.

---

## Reference Design
 
This runbook uses a single Cisco ASAv operating in routed firewall mode between an internal Layer 3 network and an upstream edge router.
 
The ASAv provides Layer 3 forwarding between the inside and outside networks, uses a default route toward the upstream router, and translates internal client addresses to the outside interface address using dynamic PAT.
 
### Devices
 
| Device | Role                     | Purpose                                  |
|--------|--------------------------|------------------------------------------|
| R1     | Upstream edge router     | Provides upstream network connectivity   |
| FW1    | Cisco ASAv               | Provides routed firewalling and PAT      |
| CORE1  | Internal Layer 3 device  | Provides routing for internal networks   |
 
### Firewall Interface Summary
 
| Device | Interface | Nameif  | Security Level | IPv4 Address  | Connected To |
|--------|-----------|---------|----------------|---------------|--------------|
| FW1    | Gi0/0     | outside | 0              | 10.0.0.2/30   | R1            |
| FW1    | Gi0/1     | inside  | 100            | 10.0.1.1/30   | CORE1         |
 
### Transit Networks
 
| Network      | Purpose                     |
|--------------|-----------------------------|
| 10.0.0.0/30  | FW1-to-R1 outside transit   |
| 10.0.1.0/30  | FW1-to-CORE1 inside transit |
 
### Internal Network
 
| Network        | Purpose                 |
|----------------|-------------------------|
| 10.10.10.0/24  | Internal client network |
 
### Routing Summary
 
| Device | Route Type    | Destination     | Next Hop   |
|--------|---------------|-----------------|------------|
| FW1    | Default route | 0.0.0.0/0       | 10.0.0.1   |
| FW1    | Static route  | 10.10.10.0/24   | 10.0.1.2   |
 
### NAT Behavior
 
Traffic sourced from `10.10.10.0/24` and exiting the `outside` interface is dynamically translated to the FW1 outside interface address using PAT.
 
**Note:** The values shown in this reference design are examples. Replace interface numbers, interface names, security levels, IPv4 addresses, internal networks, and next-hop addresses with those appropriate to the target deployment.

---

## Prerequisites and Pre-Checks
 
Before configuring routed firewall interfaces, routing, and PAT, confirm that the ASAv baseline and adjacent Layer 3 connectivity are ready for implementation.
 
### Prerequisites
 
- [ ] The ASAv is powered on and accessible for configuration.
- [ ] The required ASAv baseline has already been applied.
- [ ] The inside and outside interfaces have been identified.
- [ ] IPv4 addressing for the inside and outside interfaces has been defined.
- [ ] The upstream next-hop address has been defined.
- [ ] The internal network or networks requiring PAT have been identified.
- [ ] The inside-facing Layer 3 device has been configured with the expected transit addressing.
- [ ] The upstream edge router has been configured with the expected transit addressing.
- [ ] No conflicting interface, route, or NAT configuration is already present on the ASAv.
 
### Baseline Verification
 
Run these commands before applying the routed firewall and PAT configuration.
 
```bash
show running-config hostname
show interface ip brief
show nameif
show route
show nat
```
 
### Expected Baseline Results
 
- [ ] The ASAv has the expected hostname.
- [ ] The intended inside and outside interfaces are present and available for configuration.
- [ ] No conflicting IPv4 addressing is present on the target interfaces.
- [ ] No unexpected production routes are installed.
- [ ] No conflicting NAT rules are present.
- [ ] The ASAv is ready for routed inside/outside interface configuration.

---

## Configuration Procedure
 
Use this procedure to configure ASAv inside and outside routed interfaces, static routing, and dynamic PAT for inside-to-outside traffic.
 
### Configuration Notes
 
- Confirm the correct interface assignments, IPv4 addresses, and next-hop addresses before applying configuration.
- Configure `nameif` and security levels consistently with the intended trust boundary.
- The reference design uses security level `100` for the inside interface and `0` for the outside interface.
- Configure a default route toward the upstream edge router.
- Configure routes toward internal networks through the inside-facing Layer 3 device when those networks are not directly connected to the ASAv.
- PAT uses the outside interface address for translated inside traffic.
- Replace example interface numbers, IPv4 addresses, internal networks, and next-hop addresses with those defined for the target deployment.
- Advanced ACL policy and active/standby failover are configured separately.
 
---
 
### Step 1 — Configure Outside Interface
 
```bash
enable # Enter privileged EXEC mode
configure terminal # Enter global configuration mode
 
interface GigabitEthernet0/0 # Target the outside firewall interface
nameif outside # Assign the logical interface name
security-level 0 # Assign the outside security level
ip address 10.0.0.2 255.255.255.252 # Assign the outside IPv4 address
no shutdown # Administratively enable the interface
 
end # Return to privileged EXEC mode
write memory # Save the running configuration
```
 
---
 
### Step 2 — Configure Inside Interface
 
```bash
enable # Enter privileged EXEC mode
configure terminal # Enter global configuration mode
 
interface GigabitEthernet0/1 # Target the inside firewall interface
nameif inside # Assign the logical interface name
security-level 100 # Assign the inside security level
ip address 10.0.1.1 255.255.255.252 # Assign the inside IPv4 address
no shutdown # Administratively enable the interface
 
end # Return to privileged EXEC mode
write memory # Save the running configuration
```
 
---
 
### Step 3 — Configure Static Routing
 
```bash
enable # Enter privileged EXEC mode
configure terminal # Enter global configuration mode
 
route outside 0.0.0.0 0.0.0.0 10.0.0.1 # Configure default route toward the upstream edge router
route inside 10.10.10.0 255.255.255.0 10.0.1.2 # Route the internal client network through the inside Layer 3 device
 
end # Return to privileged EXEC mode
write memory # Save the running configuration
```
 
---
 
### Step 4 — Configure Inside-to-Outside PAT
 
```bash
enable # Enter privileged EXEC mode
configure terminal # Enter global configuration mode
 
object network INSIDE_NET # Create the network object for the internal client network
subnet 10.10.10.0 255.255.255.0 # Define the internal network represented by the object
nat (inside,outside) dynamic interface # Translate inside traffic to the outside interface address using PAT
 
end # Return to privileged EXEC mode
write memory # Save the running configuration
```

---

## Post-Configuration Validation
 
Use this section to confirm that the ASAv routed interfaces, routing, and PAT configuration are present and operating in the expected state.
 
### Step 1 — Verify Firewall Interface State
 
Run on the ASAv.
 
```bash
show interface ip brief
show nameif
```
 
Expected results:
 
- [ ] `GigabitEthernet0/0` is configured as `outside`.
- [ ] The outside interface has IPv4 address `10.0.0.2`.
- [ ] The outside interface is administratively enabled and operational when the upstream link is active.
- [ ] `GigabitEthernet0/1` is configured as `inside`.
- [ ] The inside interface has IPv4 address `10.0.1.1`.
- [ ] The inside interface is administratively enabled and operational when the internal link is active.
- [ ] The inside and outside interfaces have the expected security levels.
 
---
 
### Step 2 — Verify Firewall Routing
 
Run on the ASAv.
 
```bash
show route
```
 
Expected results:
 
- [ ] The outside transit network is present as a directly connected route.
- [ ] The inside transit network is present as a directly connected route.
- [ ] A default route through `10.0.0.1` is installed on the outside interface.
- [ ] A route to `10.10.10.0/24` through `10.0.1.2` is installed on the inside interface.
- [ ] No unexpected route conflicts are present.
 
---
 
### Step 3 — Verify PAT Configuration
 
Run on the ASAv.
 
```bash
show nat detail
show running-config object network INSIDE_NET
```
 
Expected results:
 
- [ ] The `INSIDE_NET` object represents `10.10.10.0/24`.
- [ ] A dynamic NAT rule exists from `inside` to `outside`.
- [ ] The rule translates matching traffic to the outside interface address.
- [ ] The PAT rule is enabled and ordered as intended.
- [ ] No conflicting NAT rule affects the internal network.
 
---
 
### Step 4 — Verify Routed Firewall Configuration
 
Run on the ASAv.
 
```bash
show running-config interface GigabitEthernet0/0
show running-config interface GigabitEthernet0/1
show running-config route
```
 
Expected results:
 
- [ ] The outside interface contains the expected `nameif`, security level, and IPv4 address.
- [ ] The inside interface contains the expected `nameif`, security level, and IPv4 address.
- [ ] Both interfaces are administratively enabled.
- [ ] The configured static and default routes match the reference design.
- [ ] No conflicting interface or routing configuration is present.

---

### If Validation Fails
 
If post-configuration validation does not produce the expected results:
 
- Identify which validation step failed.
- Confirm that the inside and outside interfaces have the expected `nameif`, security level, IPv4 address, and administrative state.
- Confirm that the default route points to the correct upstream next hop.
- Confirm that routes toward internal networks point to the correct inside next hop.
- Confirm that the `INSIDE_NET` object contains the expected internal subnet.
- Confirm that the dynamic PAT rule is configured from `inside` to `outside` and uses the outside interface address.
- Review the NAT table for conflicting or higher-priority rules.
- Correct the affected configuration.
- Re-run the failed validation step.
- If the issue cannot be resolved, return the affected firewall configuration to its previous known-good state before continuing.
 
---

## Backout Considerations
 
Changing or removing routed firewall, routing, or PAT configuration can interrupt connectivity between internal and external networks.
 
Before removing or changing the configuration, confirm:
 
- The original inside and outside interface configuration is documented.
- The previous interface names, security levels, IPv4 addresses, and administrative states are known.
- The existing static and default routes are documented.
- The internal networks dependent on the firewall path are understood.
- The current NAT and PAT rules are documented.
- Any upstream or downstream routing dependencies are understood.
- The firewall can be returned to its previous known-good configuration if required.
 
For lab use, restore the last known working configuration before continuing additional validation.

---

## Document Metadata
 
| Field            | Value                                     |
|------------------|-------------------------------------------|
| Author           | Aaron Kindelt                             |
| Category         | Configuration                             |
| Technology       | Cisco ASAv Routed Firewall / NAT          |
| Applies To       | Cisco ASAv firewalls                      |
| Primary Use Case | Routed firewall connectivity and PAT      |
| Version          | 1.0                                       |
| Last Updated     | 2026-09-15                                |
