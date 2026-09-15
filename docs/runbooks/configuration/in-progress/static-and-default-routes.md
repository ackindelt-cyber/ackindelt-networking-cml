# Runbook — Routed Interface Addressing
 
## Overview
 
This runbook provides a structured process for configuring and validating IPv4 addressing on physical Layer 3 interfaces on Cisco IOS routers and multilayer switches.
 
Routed interfaces provide point-to-point or transit Layer 3 connectivity between network devices. On Cisco IOS routers, physical interfaces operate as Layer 3 interfaces by default. On multilayer switches, a physical switchport must be converted to a routed interface before an IP address can be assigned.
 
This runbook focuses on assigning IPv4 addresses to physical routed interfaces, enabling the interfaces, and validating directly connected Layer 3 connectivity. The example reference design uses a point-to-point transit network between two Cisco IOS Layer 3 devices.
 
---
 
## Scope and Assumptions
 
This runbook assumes:
 
- The participating Cisco IOS Layer 3 devices are powered on and accessible for configuration.
- The required device baselines have already been applied.
- The physical interfaces to be configured have been identified.
- The IPv4 address and subnet mask for each routed interface have been defined.
- The selected interfaces are intended for Layer 3 use and are not required as Layer 2 switchports.
- On multilayer switches, the target interface supports routed-port operation.
- The directly connected peer device is configured separately.
- Static routing, default routing, dynamic routing, ACLs, NAT, and other Layer 3 services are configured separately.
 
This runbook does not cover SVIs, loopback interfaces, subinterfaces, router-on-a-stick, IPv6 addressing, EtherChannel routed ports, or tunnel interfaces.
 
---
 
## Reference Design
 
This runbook uses two Cisco IOS Layer 3 devices connected through a directly connected point-to-point transit network.
 
Each device is assigned an IPv4 address from the same subnet on the physical interface connecting the two devices.
 
### Devices
 
| Device | Role                     | Purpose                            |
|--------|--------------------------|------------------------------------|
| R1     | Cisco IOS Layer 3 device | First endpoint of the routed link  |
| R2     | Cisco IOS Layer 3 device | Second endpoint of the routed link |
 
### Routed Link Summary
 
| Device | Interface | IPv4 Address | Connected To |
|--------|-----------|--------------|--------------|
| R1     | Gi0/0     | 10.0.0.1/30  | R2 Gi0/0     |
| R2     | Gi0/0     | 10.0.0.2/30  | R1 Gi0/0     |
 
### Network Summary
 
| Network     | Purpose                     |
|-------------|-----------------------------|
| 10.0.0.0/30 | Point-to-point transit link |
 
**Note:** The values shown in this reference design are examples. Replace interface numbers, IPv4 addresses, subnet masks, and network assignments with those appropriate to the target deployment.
 
---
 
## Prerequisites and Pre-Checks
 
Before configuring routed interface addressing, confirm that the target interfaces and IPv4 addressing information are ready for implementation.
 
### Prerequisites
 
- [ ] The participating Layer 3 devices are powered on and accessible for configuration.
- [ ] The required device baselines have already been applied.
- [ ] The physical interfaces selected for Layer 3 use have been identified.
- [ ] The IPv4 address and subnet mask for each interface have been defined.
- [ ] The selected interfaces are not required for Layer 2 switching.
- [ ] On multilayer switches, the target interfaces support routed-port operation.
- [ ] No conflicting IPv4 addressing is already configured on the target interfaces.
- [ ] The directly connected peer interface has been identified.
 
### Baseline Verification
 
Run these commands before applying the routed interface configuration.
 
```bash
show ip interface brief
show interfaces status
show running-config interface Gi0/0
show ip route connected
```
 
### Expected Baseline Results
 
- [ ] The target interface exists and is available for configuration.
- [ ] No conflicting IPv4 address is configured on the target interface.
- [ ] The interface is not currently being used for unintended Layer 2 switching.
- [ ] No unexpected connected route already exists for the target transit network.
- [ ] The interface configuration can be safely modified for Layer 3 use.
 
---
 
## Configuration Procedure
 
Use this procedure to configure IPv4 addressing on physical routed interfaces.
 
### Configuration Notes
 
- Confirm the correct interface, IPv4 address, and subnet mask before applying configuration.
- Cisco IOS router interfaces operate as Layer 3 interfaces by default.
- On multilayer switches, use `no switchport` to convert a physical switchport into a routed interface before assigning an IP address.
- Replace example interface numbers, IPv4 addresses, and subnet masks with those defined for the target deployment.
- Administratively enable each routed interface after configuration.
- Static routes, default routes, and dynamic routing are configured separately.
 
---
 
### Step 1 — Configure Routed Interface on Cisco IOS Router
 
```bash
enable # Enter privileged EXEC mode
configure terminal # Enter global configuration mode
 
interface gi0/0 # Target the physical routed interface
ip address 10.0.0.1 255.255.255.252 # Assign IPv4 address and subnet mask
no shutdown # Administratively enable the interface
 
end # Return to privileged EXEC mode
write memory # Save the running configuration
```
 
---
 
### Step 2 — Configure Routed Interface on Multilayer Switch
 
```bash
enable # Enter privileged EXEC mode
configure terminal # Enter global configuration mode
 
interface gi0/0 # Target the physical interface
no switchport # Convert the interface from Layer 2 switchport to Layer 3 routed port
ip address 10.0.0.2 255.255.255.252 # Assign IPv4 address and subnet mask
no shutdown # Administratively enable the interface
 
end # Return to privileged EXEC mode
write memory # Save the running configuration
```
 
---
 
## Post-Configuration Validation
 
Use this section to confirm that the routed interfaces are configured correctly, operational, and installed as directly connected Layer 3 networks.
 
### Step 1 — Verify Interface Addressing and State
 
Run on both Layer 3 devices.
 
```bash
show ip interface brief
```
 
Expected results:
 
- [ ] The configured interface shows the expected IPv4 address.
- [ ] The interface is administratively enabled.
- [ ] The interface is operationally `up/up` when the peer link is active.
- [ ] No unexpected IPv4 address is present on the target interface.
 
---
 
### Step 2 — Verify Routed Interface Configuration
 
Run on the applicable Layer 3 device.
 
```bash
show running-config interface Gi0/0
```
 
Expected results:
 
- [ ] The target interface contains the expected IPv4 address and subnet mask.
- [ ] The interface is not administratively shut down.
- [ ] On a multilayer switch, `no switchport` is present.
- [ ] No conflicting Layer 2 or Layer 3 configuration is present.
 
---
 
### Step 3 — Verify Connected Route Installation
 
Run on both Layer 3 devices.
 
```bash
show ip route connected
```
 
Expected results:
 
- [ ] The transit network appears as a directly connected route.
- [ ] The connected route references the expected physical interface.
- [ ] No unexpected connected route is present for the target network.
 
---
 
### Step 4 — Verify Directly Connected Peer Reachability
 
Run from R1.
 
```bash
ping 10.0.0.2
```
 
Run from R2.
 
```bash
ping 10.0.0.1
```
 
Expected results:
 
- [ ] Each device can successfully reach the directly connected peer address.
- [ ] The routed link provides bidirectional Layer 3 connectivity.
- [ ] No packet loss is observed under normal lab conditions.
 
---
 
### If Validation Fails
 
If post-configuration validation does not produce the expected results:
 
- Identify which validation step failed.
- Confirm that the configured interface, IPv4 address, and subnet mask are correct.
- Confirm that the interface is administratively enabled.
- Confirm that the directly connected peer interface is configured in the same subnet.
- On multilayer switches, confirm that the interface is operating as a routed port with `no switchport`.
- Review the routing table for the expected directly connected network.
- Correct the affected configuration.
- Re-run the failed validation step.
- If the issue cannot be resolved, return the affected interface to its previous known-good state before continuing.
 
---
 
## Backout Considerations
 
Changing or removing routed interface configuration can interrupt Layer 3 connectivity across the affected link.
 
Before removing or changing the configuration, confirm:
 
- The original interface configuration is documented.
- The previous IPv4 address and subnet mask are known.
- Any routes or services dependent on the routed link are understood.
- On multilayer switches, the previous Layer 2 or Layer 3 interface role is known.
- The directly connected peer configuration is documented.
- The affected interface can be returned to its previous known-good configuration if required.
 
For lab use, restore the last known working configuration before continuing additional validation.
 
---
 
## Document Metadata
 
| Field            | Value                                     |
|------------------|-------------------------------------------|
| Author           | Aaron Kindelt                             |
| Category         | Configuration                             |
| Technology       | Cisco IOS Layer 3 Interface Configuration |
| Applies To       | Cisco IOS routers and multilayer switches |
| Primary Use Case | Physical routed interface IPv4 addressing |
| Version          | 1.0                                       |
| Last Updated     | 2026-09-14                                |