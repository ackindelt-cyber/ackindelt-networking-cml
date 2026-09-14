# Runbook — Static Routing and Default Routes
 
## Overview
 
This runbook provides a structured process for configuring and validating IPv4 static routes and default routes on Cisco IOS Layer 3 devices.
 
Static routes provide manually defined paths to specific remote networks, while default routes provide a gateway of last resort for destinations that are not otherwise present in the routing table.
 
This runbook focuses on Cisco IOS routers and multilayer switches using next-hop IPv4 addresses for static and default routing. The example reference design includes a Layer 3 device with a default route toward an upstream router and a specific static route toward a remote network.

## Scope and Assumptions
 
This runbook assumes:
 
- The participating Layer 3 devices are powered on and accessible for configuration.
- The required device baselines have already been applied.
- IP addressing for all directly connected routed links has already been configured.
- The required remote destination networks have been identified.
- The correct next-hop IPv4 addresses have been defined.
- Static routing is appropriate for the target design.
- Dynamic routing protocols are configured separately.
- Firewall policy, NAT, and VPN configuration are handled separately where applicable.
- The device has working Layer 3 reachability to the configured next-hop address.
 
This runbook does not cover dynamic routing protocols, policy-based routing, floating static routes, recursive static routing design, or IPv6 static routing.

---

## Reference Design
 
This runbook uses two Cisco IOS Layer 3 devices connected through a directly connected transit network.
 
The downstream device uses a default route toward the upstream device. The upstream device uses a specific static route back toward a remote network reachable through the downstream device.
 
### Devices
 
| Device | Role                    | Purpose                                      |
|--------|-------------------------|----------------------------------------------|
| R1     | Upstream Layer 3 device | Provides upstream connectivity               |
| R2     | Downstream Layer 3 device | Provides connectivity to a remote network  |
 
### Transit Network
 
| Device | Interface | IPv4 Address   | Connected To |
|--------|-----------|----------------|--------------|
| R1     | Gi0/0     | 10.0.0.1/30    | R2 Gi0/0     |
| R2     | Gi0/0     | 10.0.0.2/30    | R1 Gi0/0     |
 
### Remote Network
 
| Network        | Reachable Through |
|----------------|-------------------|
| 10.10.10.0/24  | R2                |
 
### Routing Summary
 
| Device | Route Type     | Destination     | Next Hop   |
|--------|----------------|-----------------|------------|
| R1     | Static route   | 10.10.10.0/24   | 10.0.0.2   |
| R2     | Default route  | 0.0.0.0/0       | 10.0.0.1   |
 
**Note:** The values shown in this reference design are examples. Replace interface numbers, IPv4 addresses, destination networks, subnet masks, and next-hop addresses with those appropriate to the target deployment.
 

---

## Prerequisites and Pre-Checks
 
Before configuring static or default routes, confirm that the required Layer 3 interfaces and next-hop connectivity are already in place.
 
### Prerequisites
 
- [ ] The participating Layer 3 devices are powered on and accessible for configuration.
- [ ] The required device baselines have already been applied.
- [ ] The directly connected routed interfaces have already been configured with the correct IPv4 addresses.
- [ ] The required remote destination networks have been identified.
- [ ] The correct next-hop IPv4 addresses have been defined.
- [ ] The next-hop address for each route is reachable through a directly connected network.
- [ ] No conflicting static or dynamic routes are already present for the target destinations.
 
### Baseline Verification
 
Run these commands before applying the static routing configuration.
 
```bash
show ip interface brief
show ip route
show running-config | include ^ip route
```
 
### Expected Baseline Results
 
- [ ] The required routed interfaces are present and operational.
- [ ] The directly connected transit network appears in the routing table.
- [ ] The intended next-hop address is reachable through a directly connected network.
- [ ] No conflicting static route exists for the target destination.
- [ ] No unexpected route is already installed for the target destination.

---

# Configuration Procedure
 
Use this procedure to configure IPv4 static routes and default routes on Cisco IOS Layer 3 devices.
 
### Configuration Notes
 
- Confirm directly connected Layer 3 interfaces and next-hop reachability before configuring static routes.
- Use next-hop IPv4 addresses that are reachable through an existing connected network.
- Replace example destination networks, subnet masks, and next-hop addresses with those defined for the target deployment.
- Avoid creating static routes that conflict with existing dynamic or connected routes.
- Configure only the routes required for the intended design.
 
---
 
### Step 1 — Configure Specific Static Routes
 
```bash
enable # Enter privileged EXEC mode
configure terminal # Enter global configuration mode
 
ip route 10.10.10.0 255.255.255.0 10.0.0.2 # Route 10.10.10.0/24 through next hop 10.0.0.2
 
end # Return to privileged EXEC mode
write memory # Save the running configuration
```
 
---
 
### Step 2 — Configure Default Route
 
```bash
enable # Enter privileged EXEC mode
configure terminal # Enter global configuration mode
 
ip route 0.0.0.0 0.0.0.0 10.0.0.1 # Configure default route through next hop 10.0.0.1
 
end # Return to privileged EXEC mode
write memory # Save the running configuration
```

---

## Post-Configuration Validation
 
Use this section to confirm that the configured static and default routes are present and installed in the routing table as intended.
 
### Step 1 — Verify Static Route Installation
 
Run on R1.
 
```bash
show ip route static
```
 
Expected results:
 
- [ ] A static route to `10.10.10.0/24` is present.
- [ ] The route uses next hop `10.0.0.2`.
- [ ] The route is installed in the routing table as a static route.
- [ ] No unexpected static routes are present for the target destination.
 
---
 
### Step 2 — Verify Default Route Installation
 
Run on R2.
 
```bash
show ip route
```
 
Expected results:
 
- [ ] A default route for `0.0.0.0/0` is present.
- [ ] The default route uses next hop `10.0.0.1`.
- [ ] The route is installed as a static route.
- [ ] `10.0.0.1` is identified as the gateway of last resort.
- [ ] No unexpected default route is present.
 
---
 
### Step 3 — Verify Configured Static Route Statements
 
Run on the applicable Layer 3 devices.
 
```bash
show running-config | include ^ip route
```
 
Expected results:
 
- [ ] R1 contains the expected static route to `10.10.10.0/24` through `10.0.0.2`.
- [ ] R2 contains the expected default route through `10.0.0.1`.
- [ ] No conflicting or unintended static route statements are present.

---

### If Validation Fails
 
If post-configuration validation does not produce the expected results:
 
- Identify which validation step failed.
- Confirm that the destination network and subnet mask are correct.
- Confirm that the configured next-hop IPv4 address is correct.
- Confirm that the next-hop address is reachable through a directly connected network.
- Review the routing table for conflicting or more-preferred routes.
- Confirm that the required routed interfaces are operational.
- Correct the affected configuration.
- Re-run the failed validation step.
- If the issue cannot be resolved, return the affected route configuration to its previous known-good state before continuing.

---

## Backout Considerations
 
Changing or removing static routes can interrupt Layer 3 reachability to remote networks or upstream destinations.
 
Before removing or changing the configuration, confirm:
 
- The original routing configuration is documented.
- The affected destination networks and next-hop addresses are known.
- Any services or downstream networks dependent on the route are understood.
- No alternate route is required before removing the existing static route.
- Any default-route dependency on upstream connectivity is understood.
- The affected routing configuration can be returned to its previous known-good state if required.
 
For lab use, restore the last known working configuration before continuing additional validation.

---

## Document Metadata
 
| Field            | Value                                  |
|------------------|----------------------------------------|
| Author           | Aaron Kindelt                          |
| Category         | Configuration                          |
| Technology       | Cisco IOS Layer 3 Routing              |
| Applies To       | Cisco IOS routers and multilayer switches |
| Primary Use Case | Static and default route configuration |
| Version          | 1.0                                    |
| Last Updated     | 2026-09-14                             |
