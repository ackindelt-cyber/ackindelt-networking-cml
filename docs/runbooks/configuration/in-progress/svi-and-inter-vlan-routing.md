# Runbook — SVI and Inter-VLAN Routing
 
## Overview
 
This runbook provides a structured process for configuring and validating switched virtual interfaces (SVIs) and inter-VLAN routing on Cisco IOS multilayer switches.
 
SVIs provide Layer 3 interfaces for individual VLANs and can serve as default gateways for devices within those VLANs. When IP routing is enabled on the switch, traffic can be routed between directly connected VLANs through their respective SVIs.
 
This runbook focuses on a Cisco IOS multilayer switch providing Layer 3 gateway services for multiple VLANs using locally configured SVIs and connected routing. The example reference design uses three VLANs, each with its own SVI and IPv4 gateway address.

## Scope and Assumptions
 
This runbook assumes:
 
- The participating switch supports Layer 3 switching and SVI configuration.
- The required VLANs have already been created.
- IP addressing for each SVI has been defined.
- The switch is intended to provide Layer 3 gateway services for the configured VLANs.
- `ip routing` is already enabled as part of the device baseline
- The VLANs have active Layer 2 membership so their SVIs can transition to an operational state.
- HSRP, VRRP, or other first-hop redundancy protocols are configured separately.
- DHCP relay, ACLs, and other Layer 3 services are configured separately.
- Upstream routing beyond directly connected VLANs is configured separately.
 
This runbook does not cover router-on-a-stick, physical routed interfaces, first-hop redundancy, DHCP relay, dynamic routing protocols, or firewall policy.

---

## Reference Design
 
This runbook uses one Cisco IOS multilayer switch providing Layer 3 gateway services for three VLANs through switched virtual interfaces.
 
Each VLAN is assigned its own IPv4 subnet and SVI. IP routing on the switch allows traffic to be routed between the directly connected VLANs.
 
### Devices
 
| Device | Role                        | Purpose                                |
|--------|-----------------------------|----------------------------------------|
| SW1    | Cisco IOS multilayer switch | Provides SVIs and inter-VLAN routing   |
 
### VLAN and SVI Summary
 
| VLAN | Name    | Subnet           | SVI Address    |
|------|---------|------------------|----------------|
| 10   | USERS   | 10.10.10.0/24    | 10.10.10.1/24  |
| 20   | VOICE   | 10.10.20.0/24    | 10.10.20.1/24  |
| 30   | SERVERS | 10.10.30.0/24    | 10.10.30.1/24  |
 
### Routing Behavior
 
SW1 provides directly connected routing between VLANs 10, 20, and 30.
 
Endpoints within each VLAN use the corresponding SVI address as their default gateway.
 
**Note:** The values shown in this reference design are examples. Replace VLAN IDs, VLAN names, IPv4 subnets, and SVI addresses with those appropriate to the target deployment. In designs using a first-hop redundancy protocol such as HSRP, the individual SVI addresses and virtual gateway address are defined separately.

---

## Prerequisites and Pre-Checks
 
Before configuring SVIs and inter-VLAN routing, confirm that the required VLANs, addressing, and Layer 3 switch capabilities are in place.
 
### Prerequisites
 
- [ ] The multilayer switch is powered on and accessible for configuration.
- [ ] The required device baseline has already been applied.
- [ ] The required VLANs have been created.
- [ ] The IPv4 subnet and SVI address for each VLAN have been defined.
- [ ] The switch supports Layer 3 routing.
- [ ] The switch is intended to provide gateway services for the target VLANs.
- [ ] No conflicting SVI addressing is already present.
- [ ] Any first-hop redundancy configuration is handled separately.
 
### Baseline Verification
 
Run these commands before applying the SVI and inter-VLAN routing configuration.
 
```bash
show vlan brief
show ip interface brief
show running-config | include ^ip routing
show ip route
```
 
### Expected Baseline Results
 
- [ ] The required VLANs exist and are active.
- [ ] The target SVIs are not already configured with conflicting IP addresses.
- [ ] The switch is capable of Layer 3 routing.
- [ ] No unexpected connected routes exist for the target VLAN subnets.
- [ ] No conflicting Layer 3 configuration is present.

---

## Configuration Procedure
 
Use this procedure to configure switched virtual interfaces on a Cisco IOS multilayer switch for inter-VLAN routing.
 
### Configuration Notes
 
- Create the required VLANs before configuring their SVIs.
- Confirm `ip routing` is already enabled as part of the multilayer switch baseline.
- Assign each SVI an IP address from the subnet associated with that VLAN.
- Ensure each SVI uses a unique IP address and subnet.
- Replace example VLAN IDs, IP addresses, and subnet masks with those defined for the target deployment.
- An SVI may remain protocol-down until its VLAN has at least one active Layer 2 interface or trunk carrying that VLAN.
- First-hop redundancy, DHCP relay, ACLs, and upstream routing are configured separately.
 
---
 
### Step 1 — Configure VLAN SVIs
 
```bash
enable # Enter privileged EXEC mode
configure terminal # Enter global configuration mode
 
interface vlan 10 # Create and target the VLAN 10 SVI
ip address 10.10.10.1 255.255.255.0 # Assign the VLAN 10 gateway address
no shutdown # Administratively enable the SVI
 
interface vlan 20 # Create and target the VLAN 20 SVI
ip address 10.10.20.1 255.255.255.0 # Assign the VLAN 20 gateway address
no shutdown # Administratively enable the SVI
 
interface vlan 30 # Create and target the VLAN 30 SVI
ip address 10.10.30.1 255.255.255.0 # Assign the VLAN 30 gateway address
no shutdown # Administratively enable the SVI
 
end # Return to privileged EXEC mode
write memory # Save the running configuration
```

---

## Post-Configuration Validation
 
Use this section to confirm that the configured SVIs are present, operational, and installed as directly connected Layer 3 networks.
 
### Step 1 — Verify SVI Interface State
 
Run on the multilayer switch.
 
```bash
show ip interface brief
```
 
Expected results:
 
- [ ] `Vlan10` is configured with IP address `10.10.10.1`.
- [ ] `Vlan20` is configured with IP address `10.10.20.1`.
- [ ] `Vlan30` is configured with IP address `10.10.30.1`.
- [ ] Each SVI is administratively enabled.
- [ ] Each SVI with active Layer 2 VLAN membership is operationally `up/up`.
- [ ] No target SVI has an unexpected IP address or interface state.
 
---
 
### Step 2 — Verify SVI Configuration
 
Run on the multilayer switch.
 
```bash
show running-config interface vlan 10
show running-config interface vlan 20
show running-config interface vlan 30
```
 
Expected results:
 
- [ ] `Vlan10` is configured with `10.10.10.1 255.255.255.0`.
- [ ] `Vlan20` is configured with `10.10.20.1 255.255.255.0`.
- [ ] `Vlan30` is configured with `10.10.30.1 255.255.255.0`.
- [ ] No conflicting Layer 3 configuration is present on the target SVIs.
- [ ] The target SVIs are not administratively shut down.
 
---
 
### Step 3 — Verify Connected Routing
 
Run on the multilayer switch.
 
```bash
show ip route connected
```
 
Expected results:
 
- [ ] `10.10.10.0/24` appears as a directly connected network through `Vlan10`.
- [ ] `10.10.20.0/24` appears as a directly connected network through `Vlan20`.
- [ ] `10.10.30.0/24` appears as a directly connected network through `Vlan30`.
- [ ] No unexpected connected routes are present for the target VLANs.
- [ ] The switch has the connected Layer 3 routes required to route traffic between the configured VLANs.
 
**Note:** A connected route is installed only while the associated SVI is operational. If an expected route is missing, verify that the VLAN exists and has active Layer 2 membership.

---

### If Validation Fails
 
If post-configuration validation does not produce the expected results:
 
- Identify which validation step failed.
- Confirm that the required VLAN exists and is active.
- Confirm that the SVI has the expected IP address and subnet mask.
- Confirm that the SVI is administratively enabled.
- Confirm that the VLAN has active Layer 2 membership when an `up/up` SVI state is expected.
- Confirm that `ip routing` remains enabled as part of the multilayer switch baseline.
- Correct the affected configuration.
- Re-run the failed validation step.
- If the issue cannot be resolved, return the affected SVI configuration to its previous known-good state before continuing.
---

## Backout Considerations
 
Changing or removing SVI configuration can interrupt Layer 3 gateway services and inter-VLAN routing for the affected VLANs.
 
Before removing or changing the configuration, confirm:
 
- The original SVI configuration is documented.
- The previous IP address and subnet mask for each affected SVI are known.
- Any endpoints or services using the SVI as their default gateway are understood.
- Any first-hop redundancy configuration associated with the SVI is documented.
- Any routing, DHCP relay, or policy configuration dependent on the SVI is understood.
- The affected SVIs can be returned to their previous known-good configuration if required.
 
For lab use, restore the last known working configuration before continuing additional validation.

---

## Document Metadata
 
| Field            | Value                                   |
|------------------|-----------------------------------------|
| Author           | Aaron Kindelt                           |
| Category         | Configuration                           |
| Technology       | Cisco IOS Layer 3 Switching             |
| Applies To       | Cisco IOS multilayer switches           |
| Primary Use Case | SVI creation and inter-VLAN routing     |
| Version          | 1.0                                     |
| Last Updated     | 2026-09-14                              |
