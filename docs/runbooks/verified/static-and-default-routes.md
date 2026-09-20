# Runbook — Static and Default Routes

## Overview

This runbook provides a structured process for configuring and validating IPv4 static routes and default routes on Cisco IOS Layer 3 devices.

Static routes provide manually defined forwarding paths to specific destination networks. Default routes provide a gateway of last resort for traffic destined to networks that are not otherwise present in the routing table.

This runbook focuses on configuring a destination-specific static route on one Cisco IOS Layer 3 device and a default route on another. The example reference design uses two directly connected Layer 3 devices with a remote network reachable behind the second device.

---

## Scope and Assumptions

This runbook assumes:

- The participating Cisco IOS Layer 3 devices are powered on and accessible for configuration.
- The required device baselines have already been applied.
- The required Layer 3 interfaces have already been configured and validated.
- The directly connected transit network is operational.
- The destination network for the static route has been identified.
- The next-hop IPv4 address for each route has been defined.
- The selected next hop is reachable through an existing connected route.
- Dynamic routing protocols are configured separately.
- Policy-based routing, route tracking, floating static routes, and advanced route redistribution are handled separately.

This runbook does not cover IPv6 static routing, dynamic routing protocols, floating static routes, object tracking, policy-based routing, or route redistribution.

---

## Reference Design

This runbook uses two Cisco IOS Layer 3 devices connected through a directly connected transit network.

R1 uses a destination-specific static route to reach the `10.10.10.0/24` network through R2. R2 uses a default route toward R1 for traffic destined beyond its directly connected networks.

### Devices

| Device | Role                  | Purpose                                      |
|--------|-----------------------|----------------------------------------------|
| R1     | Cisco IOS Layer 3 device | Provides upstream routing toward R2          |
| R2     | Cisco IOS Layer 3 device | Provides access to the remote client network |

### Transit Network

| Device | Interface | IPv4 Address | Connected To |
|--------|-----------|--------------|--------------|
| R1     | Gi0/0     | 10.0.0.1/30  | R2 Gi0/0     |
| R2     | Gi0/0     | 10.0.0.2/30  | R1 Gi0/0     |

### Remote Network

| Network        | Location |
|----------------|----------|
| 10.10.10.0/24  | Behind R2 |

### Routing Summary

| Device | Route Type    | Destination    | Next Hop   |
|--------|---------------|----------------|------------|
| R1     | Static route  | 10.10.10.0/24  | 10.0.0.2   |
| R2     | Default route | 0.0.0.0/0      | 10.0.0.1   |

**Note:** The values shown in this reference design are examples. Replace destination networks, next-hop addresses, interface assignments, and transit addressing with those appropriate to the target deployment.

---

## Prerequisites and Pre-Checks

Before configuring static or default routes, confirm that the underlying Layer 3 interfaces and directly connected networks are already operational.

### Prerequisites

- [ ] The participating Layer 3 devices are powered on and accessible for configuration.
- [ ] The required device baselines have already been applied.
- [ ] The required routed interfaces or SVIs have already been configured.
- [ ] The directly connected transit network is operational.
- [ ] The destination network for each static route has been defined.
- [ ] The required next-hop IPv4 addresses have been defined.
- [ ] Each next hop is reachable through an existing connected network.
- [ ] No conflicting static or default route is already present.
- [ ] The device serving as the next hop has an existing route or directly connected path to the destination network.

### Baseline Verification

Run these commands before applying the routing configuration.

```bash
show ip interface brief
show ip route
show ip route connected
show running-config | include ^ip route
```

### Expected Baseline Results

- [ ] The required Layer 3 interfaces are present and operational.
- [ ] The transit network appears as directly connected.
- [ ] The intended next-hop address is reachable through the routing table.
- [ ] No conflicting route already exists for the target destination.
- [ ] No unexpected default route is already configured.
- [ ] The devices are ready for static route configuration.

---

## Configuration Procedure

Use this procedure to configure destination-specific static routing and default routing.

### Configuration Notes

- Confirm the destination network, subnet mask, and next-hop address before applying configuration.
- The next-hop address must be reachable through an existing connected route.
- Use destination-specific static routes when a defined network requires an explicit forwarding path.
- Use a default route only when traffic for otherwise unknown destinations should follow a common upstream path.
- Replace example destination networks and next-hop addresses with those defined for the target deployment.
- Dynamic routing protocols and advanced route tracking are configured separately.

---

### Step 1 — Configure Static Route on R1

Run on R1.

```bash
enable # Enter privileged EXEC mode
configure terminal # Enter global configuration mode

ip route 10.10.10.0 255.255.255.0 10.0.0.2 # Route the remote client network through R2

end # Return to privileged EXEC mode
write memory # Save the running configuration
```

---

### Step 2 — Configure Default Route on R2

Run on R2.

```bash
enable # Enter privileged EXEC mode
configure terminal # Enter global configuration mode

ip route 0.0.0.0 0.0.0.0 10.0.0.1 # Configure R1 as the gateway of last resort

end # Return to privileged EXEC mode
write memory # Save the running configuration
```

---

## Post-Configuration Validation

Use this section to confirm that the static and default routes are installed correctly and reference the expected next hops.

### Step 1 — Verify Static Route on R1

Run on R1.

```bash
show ip route static
```

Expected results:

- [ ] A static route to `10.10.10.0/24` is present.
- [ ] The route uses `10.0.0.2` as the next hop.
- [ ] The route is installed in the routing table.
- [ ] No conflicting static route exists for the same destination.

---

### Step 2 — Verify Default Route on R2

Run on R2.

```bash
show ip route
```

Expected results:

- [ ] A default route to `0.0.0.0/0` is present.
- [ ] The default route uses `10.0.0.1` as the next hop.
- [ ] R1 is identified as the gateway of last resort.
- [ ] No conflicting default route is present.

---

### Step 3 — Verify Configured Route Statements

Run on both devices.

```bash
show running-config | include ^ip route
```

Expected results:

- [ ] R1 contains the expected static route to `10.10.10.0/24`.
- [ ] R2 contains the expected default route.
- [ ] The configured next-hop addresses match the reference design.
- [ ] No unexpected static route statements are present.

---

## If Validation Fails

If post-configuration validation does not produce the expected results:

- Identify which validation step failed.
- Confirm that the destination network and subnet mask are correct.
- Confirm that the configured next-hop IPv4 address is correct.
- Confirm that the next hop is reachable through an existing connected route.
- Confirm that the required Layer 3 interfaces are operational.
- Review the routing table for conflicting or more-specific routes.
- Review the running configuration for incorrect or duplicate static route statements.
- Correct the affected configuration.
- Re-run the failed validation step.
- If the issue cannot be resolved, return the affected routing configuration to its previous known-good state before continuing.

---

## Backout Considerations

Changing or removing static or default routes can interrupt Layer 3 connectivity to dependent networks.

Before removing or changing the configuration, confirm:

- The original routing configuration is documented.
- The affected destination networks are known.
- The current next-hop addresses are documented.
- Any services or downstream networks dependent on the routes are understood.
- An alternate routing path exists if required.
- The device can be returned to its previous known-good routing configuration if necessary.

For lab use, restore the last known working configuration before continuing additional validation.

---

## Document Metadata

| Field            | Value                                  |
|------------------|----------------------------------------|
| Author           | Aaron Kindelt                          |
| Category         | Configuration                          |
| Technology       | Cisco IOS Static Routing               |
| Applies To       | Cisco IOS Layer 3 devices              |
| Primary Use Case | Static and default route configuration |
| Version          | 1.0                                    |
| Last Updated     | 2026-09-20                             |