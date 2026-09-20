# Runbook — DHCP Relay
 
## Overview
 
This runbook provides a structured process for configuring and validating DHCP relay on Cisco IOS Layer 3 interfaces.
 
DHCP relay allows client devices on one subnet to obtain addressing information from a DHCP server located on a different subnet. The relay function forwards client DHCP broadcasts as unicast traffic toward the configured DHCP server.
 
This runbook focuses on configuring `ip helper-address` on Cisco IOS switched virtual interfaces or routed Layer 3 interfaces that act as the default gateway for DHCP client networks. The example reference design uses multiple client VLANs forwarding DHCP requests to a centralized DHCP server on a separate server network.

## Scope and Assumptions
 
This runbook assumes:
 
- The client-facing Layer 3 interfaces or SVIs have already been configured.
- The DHCP server has already been configured with the required scopes and options.
- The DHCP server IPv4 address has been defined.
- The Layer 3 device has IP reachability to the DHCP server.
- The client subnets and corresponding gateway interfaces have been identified.
- The Layer 3 interface receiving client DHCP broadcasts is the correct location for the relay configuration.
- Static routing, default routing, dynamic routing, and firewall policy are configured separately.
- DHCP server configuration is handled separately.
 
This runbook does not cover DHCP server scope creation, DHCP snooping, DHCP client configuration, IPv6 DHCP relay, or DHCP services running locally on the Cisco IOS device.

---

## Reference Design
 
This runbook uses a Cisco IOS multilayer switch providing gateway services for multiple client VLANs and forwarding DHCP requests to a centralized DHCP server located on a separate server network.
 
### Devices
 
| Device | Role                        | Purpose                                  |
|--------|-----------------------------|------------------------------------------|
| SW1    | Cisco IOS multilayer switch | Provides client SVIs and DHCP relay      |
| DHCP1  | Centralized DHCP server     | Provides DHCP scopes for client networks |
 
### Client VLAN Summary
 
| VLAN | Name  | Client Subnet     | Gateway SVI     | DHCP Server |
|------|-------|-------------------|-----------------|-------------|
| 10   | USERS | 10.10.10.0/24     | 10.10.10.1/24   | 10.10.30.10 |
| 20   | VOICE | 10.10.20.0/24     | 10.10.20.1/24   | 10.10.30.10 |
 
### Server Network
 
| Device | IPv4 Address |
|--------|--------------|
| DHCP1  | 10.10.30.10  |
 
### Relay Behavior
 
DHCP broadcasts received on the VLAN 10 and VLAN 20 gateway interfaces are relayed to `10.10.30.10`.
 
**Note:** The values shown in this reference design are examples. Replace VLAN IDs, subnet information, gateway addresses, and DHCP server addresses with those appropriate to the target deployment.

---

## Prerequisites and Pre-Checks
 
Before configuring DHCP relay, confirm that the client gateway interfaces, DHCP server, and Layer 3 reachability are already in place.
 
### Prerequisites
 
- [ ] The participating Layer 3 device is powered on and accessible for configuration.
- [ ] The required device baseline has already been applied.
- [ ] The client-facing SVIs or routed interfaces have already been configured.
- [ ] The DHCP server is configured with the required scopes and DHCP options.
- [ ] The DHCP server IPv4 address has been defined.
- [ ] The client subnets requiring DHCP relay have been identified.
- [ ] Routing between the DHCP relay device and the DHCP server network is available in both directions.
- [ ] No conflicting DHCP relay configuration is already present on the target interfaces.
 
### Baseline Verification
 
Run these commands before applying the DHCP relay configuration.
 
```bash
show ip interface brief
show ip route
show running-config | include ip helper-address
```
 
### Expected Baseline Results
 
- [ ] The client gateway interfaces are present and operational.
- [ ] The Layer 3 device has a route to the DHCP server network.
- [ ] The DHCP server address is reachable through the existing routing table.
- [ ] No unexpected `ip helper-address` configuration is present on the target interfaces.
- [ ] The target interfaces can be safely modified to add DHCP relay.

---

## Configuration Procedure
 
Use this procedure to configure DHCP relay on the Layer 3 interfaces that receive DHCP broadcasts from client networks.
 
### Configuration Notes
 
- Configure `ip helper-address` on the client-facing Layer 3 interface or SVI where DHCP broadcasts are received.
- Do not configure the helper address on the DHCP server-facing interface unless that interface also serves a client subnet requiring relay.
- Confirm that the DHCP server address is reachable through the existing routing table before applying the relay configuration.
- Replace example VLAN IDs and DHCP server addresses with those defined for the target deployment.
- Configure a helper address on each client subnet that requires centralized DHCP service.
- DHCP scope creation and DHCP server configuration are handled separately.
 
---
 
### Step 1 — Configure DHCP Relay on Client Gateway Interfaces
 
```bash
enable # Enter privileged EXEC mode
configure terminal # Enter global configuration mode
 
interface vlan 10 # Target the VLAN 10 client gateway SVI
ip helper-address 10.10.30.10 # Relay DHCP requests to the centralized DHCP server
 
interface vlan 20 # Target the VLAN 20 client gateway SVI
ip helper-address 10.10.30.10 # Relay DHCP requests to the centralized DHCP server
 
end # Return to privileged EXEC mode
write memory # Save the running configuration
```

---

## Post-Configuration Validation
 
Use this section to confirm that DHCP relay is configured on the correct client gateway interfaces and that the relay destination remains reachable through the existing Layer 3 topology.
 
### Step 1 — Verify DHCP Relay Configuration
 
Run on the Layer 3 device.
 
```bash
show running-config interface vlan 10
show running-config interface vlan 20
```
 
Expected results:
 
- [ ] `Vlan10` contains `ip helper-address 10.10.30.10`.
- [ ] `Vlan20` contains `ip helper-address 10.10.30.10`.
- [ ] The helper address is configured only on the intended client gateway interfaces.
- [ ] No unexpected or conflicting helper addresses are present.
 
---
 
### Step 2 — Verify DHCP Relay Interface State
 
Run on the Layer 3 device.
 
```bash
show ip interface vlan 10
show ip interface vlan 20
```
 
Expected results:
 
- [ ] Each client gateway interface reports `10.10.30.10` as its helper address.
- [ ] Each target SVI is administratively enabled.
- [ ] Each target SVI is operational when active Layer 2 VLAN membership is present.
- [ ] No unexpected DHCP relay configuration is reported on the target interfaces.
 
---
 
### Step 3 — Verify DHCP Server Route
 
Run on the Layer 3 device.
 
```bash
show ip route 10.10.30.10
```
 
Expected results:
 
- [ ] The routing table contains a valid path toward `10.10.30.10`.
- [ ] The route uses the expected interface or next hop.
- [ ] No unexpected routing condition prevents traffic from reaching the DHCP server network.

---

## If Validation Fails
 
If post-configuration validation does not produce the expected results:
 
- Identify which validation step failed.
- Confirm that the helper address is configured on the correct client-facing Layer 3 interface or SVI.
- Confirm that the configured DHCP server IPv4 address is correct.
- Confirm that the client gateway interface is administratively enabled and operational.
- Confirm that the Layer 3 device has a valid route to the DHCP server.
- Review the interface configuration for missing, incorrect, or conflicting helper-address statements.
- Correct the affected configuration.
- Re-run the failed validation step.
- If the issue cannot be resolved, return the affected interface to its previous known-good state before continuing.
 

---

## Backout Considerations
 
Changing or removing DHCP relay configuration can interrupt dynamic address assignment for clients on the affected subnets.
 
Before removing or changing the configuration, confirm:
 
- The original helper-address configuration is documented.
- The affected client VLANs and gateway interfaces are known.
- The DHCP server address used by each client subnet is documented.
- Any alternate DHCP relay or server path is understood.
- The affected interfaces can be returned to their previous known-good configuration if required.
- DHCP server scopes and routing dependencies are not being changed as part of the same backout unless explicitly planned.
 
For lab use, restore the last known working configuration before continuing additional validation.

---

## Document Metadata
 
| Field            | Value                                |
|------------------|--------------------------------------|
| Author           | Aaron Kindelt                        |
| Category         | Configuration                        |
| Technology       | Cisco IOS DHCP Relay                 |
| Applies To       | Cisco IOS Layer 3 devices            |
| Primary Use Case | Centralized DHCP relay configuration |
| Version          | 1.0                                  |
| Last Updated     | 2026-09-14                           |
