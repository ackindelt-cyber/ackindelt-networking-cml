# Runbook — Enterprise Baseline Verification
 
## Overview
 
This runbook provides a structured process for validating that all devices in an enterprise network topology are in the expected baseline state before feature-specific configuration begins.
 
Enterprise baseline verification confirms that device roles, management configuration, Layer 2 and Layer 3 operating modes, interface states, parking VLAN assignments, and other foundational settings are consistent with the intended design.
 
This runbook does not configure network features. It is used as an integrated pre-deployment checkpoint to confirm that the environment is clean, predictable, and ready for additional configuration such as VLANs, trunks, EtherChannel, SVIs, HSRP, routing, DHCP relay, and firewall services.

## Scope and Assumptions
 
This runbook assumes:
 
- The enterprise topology has already been created and all required devices are present.
- Each device has the appropriate role-specific baseline applied.
- Device hostnames and management addressing have been defined.
- Access switches, multilayer core/distribution switches, routers, firewalls, and dedicated transit switches are validated using their respective baseline runbooks.
- Production VLANs, trunks, EtherChannels, SVIs, HSRP, routing, DHCP relay, NAT, and other feature-specific configuration have not yet been applied unless explicitly required by the baseline.
- Interfaces intended for later use are in the expected baseline state before feature deployment begins.
- Unused switch interfaces are assigned to the designated parking VLAN and administratively disabled where required by the device baseline.
- Management interfaces or SVIs may remain operationally down until the Layer 2 or upstream dependencies required to activate them are introduced.
- Any intentional deviation from the standard baseline has been documented before validation begins.
 
This runbook does not configure devices or remediate feature-specific problems. It provides an integrated checkpoint to confirm that the enterprise environment is ready for feature deployment.

---

## Reference Design
 
This runbook uses a representative enterprise topology containing edge, firewall, core/distribution, access, and infrastructure device roles.
 
Each device is expected to have its appropriate role-specific baseline applied before enterprise feature configuration begins.
 
### Devices
 
| Device | Role                          | Baseline Expectation                        |
|--------|-------------------------------|---------------------------------------------|
| EDGE1  | Cisco IOS edge router         | Router baseline applied                     |
| OS1    | Outside transit switch        | Outside transit switch baseline applied     |
| FW1    | Primary Cisco ASAv firewall   | ASAv baseline applied                       |
| FW2    | Secondary Cisco ASAv firewall | ASAv baseline applied                       |
| CORE1  | Core/distribution switch      | Core/distribution switch baseline applied   |
| CORE2  | Core/distribution switch      | Core/distribution switch baseline applied   |
| ASW1   | Access switch                 | Access switch baseline applied              |
| ASW2   | Access switch                 | Access switch baseline applied              |
| ASW3   | Access switch                 | Access switch baseline applied              |
| INF1   | Infrastructure router/server  | Appropriate infrastructure baseline applied |

---

# Prerequisites and Pre-Checks
 
Before beginning enterprise baseline verification, confirm that the topology and role-specific baseline documentation are available.
 
### Prerequisites
 
- [ ] All devices required by the enterprise topology are present and accessible.
- [ ] Each device has been assigned its intended network role.
- [ ] The appropriate role-specific baseline has been applied to each device.
- [ ] Device hostnames and management addressing have been defined.
- [ ] Physical and logical interface mappings are documented.
- [ ] Any approved deviations from the standard baselines are documented.
- [ ] Feature-specific configuration has not yet been applied unless explicitly required by a device baseline.
 
### Baseline Verification
 
Perform an initial access and inventory check before beginning detailed enterprise baseline validation.
 
Run on Cisco IOS devices:
 
```bash
show running-config | include ^hostname
show ip interface brief
show interfaces status
```
 
Run on Cisco ASAv devices:
 
```bash
show running-config hostname
show interface ip brief
show nameif
show running-config interface Management0/0
```
 
 
### Expected Baseline Results
 
- [ ] Every expected device is present and accessible.
- [ ] Each device reports the expected hostname.
- [ ] Management interfaces or SVIs are configured as documented.
- [ ] Physical interfaces are present and available for detailed validation.
- [ ] No unexpected device or interface state prevents the baseline verification process from continuing.
- [ ] Any approved baseline deviations are documented before detailed validation begins.
has context menu

---

### Step 1 — Verify Access Switch Baselines
 
Run on each access switch.
 
```bash
show running-config | include ^hostname
show running-config | include ^no ip routing
show vlan brief
show ip interface brief
show interfaces status
```
 
Expected results:
 
- [ ] Each access switch has the expected hostname.
- [ ] `no ip routing` is present.
- [ ] The required management and parking VLANs are present.
- [ ] The management SVI is configured with the expected address.
- [ ] Unused physical interfaces remain assigned to the parking VLAN and administratively disabled.
- [ ] No unexpected production VLAN, trunk, EtherChannel, or Layer 3 configuration is present.
 
---
 
### Step 2 — Verify Core/Distribution Switch Baselines
 
Run on each core/distribution switch.
 
```bash
show running-config | include ^hostname
show running-config | include ^ip routing
show vlan brief
show ip interface brief
show interfaces status
show ip route
```
 
Expected results:
 
- [ ] Each core/distribution switch has the expected hostname.
- [ ] `ip routing` is enabled.
- [ ] The required management and parking VLANs are present.
- [ ] The management SVI is configured with the expected address.
- [ ] Unused physical interfaces remain assigned to the parking VLAN and administratively disabled.
- [ ] No unexpected production SVIs, routes, HSRP groups, trunks, or EtherChannels are present.
 
---
 
### Step 3 — Verify Router Baselines
 
Run on each Cisco IOS router.
 
```bash
show running-config | include ^hostname
show ip interface brief
show ip route
```
 
Expected results:
 
- [ ] Each router has the expected hostname.
- [ ] Interfaces are in the expected baseline state.
- [ ] No unexpected production IP addressing is present.
- [ ] No unintended static, default, or dynamic routes are present.
- [ ] No unexpected NAT or production feature configuration is present.
 
---
 
### Step 4 — Verify Outside Transit Switch Baseline
 
Run on each outside transit switch.
 
```bash
show running-config | include ^hostname
show running-config | include ^no ip routing
show vlan brief
show interfaces status
```
 
Expected results:
 
- [ ] The switch has the expected hostname.
- [ ] `no ip routing` is present.
- [ ] The required outside transit VLAN exists.
- [ ] Interfaces assigned to the outside transit segment match the intended baseline.
- [ ] Unused interfaces remain in the expected parking state.
- [ ] No unexpected Layer 3 or campus-access configuration is present.
 
---
 
### Step 5 — Verify ASAv Baselines
 
Run on each ASAv firewall.
 
```bash
show running-config hostname
show interface ip brief
show nameif
show running-config interface Management0/0
```
 
Expected results:
 
- [ ] Each firewall has the expected hostname.
- [ ] `Management0/0` is configured with the expected management address.
- [ ] The management interface has the expected `nameif`.
- [ ] `management-only` is present where required by the baseline.
- [ ] No unexpected production inside, outside, NAT, ACL, routing, or failover configuration is present.
 
---
 
### Step 6 — Confirm Enterprise Baseline Readiness
 
Review the results from all device-role checks.
 
Expected results:
 
- [ ] Every expected enterprise device has passed its role-specific baseline check.
- [ ] Device roles and hostnames match the documented topology.
- [ ] Management configuration is present as expected.
- [ ] Unused interfaces remain in their intended baseline state.
- [ ] No unintended production feature configuration is present.
- [ ] Any approved baseline deviations are documented.
- [ ] No unresolved baseline issue remains before feature deployment begins. 

---

### If Validation Fails
 
If enterprise baseline validation does not produce the expected results:
 
- Identify which device or device role failed validation.
- Compare the affected device against its applicable role-specific baseline runbook.
- Confirm that the expected hostname, management configuration, interface state, and baseline feature settings are present.
- Identify any unexpected production configuration or undocumented deviation.
- Correct the affected baseline issue using the appropriate role-specific runbook.
- Re-run the failed validation step.
- Do not begin feature-specific deployment until all unresolved baseline issues have been corrected or formally documented.
 

---

## Backout Considerations
 
This runbook is read-only and does not modify device configuration.
 
No backout procedure is required.
 
If validation identifies an incorrect or incomplete baseline, use the applicable role-specific baseline runbook to correct the affected device before continuing with enterprise feature deployment.

---

## Document Metadata
 
| Field            | Value                                      |
|------------------|--------------------------------------------|
| Author           | Aaron Kindelt                              |
| Category         | Validation                                 |
| Technology       | Enterprise Network Baseline                |
| Applies To       | Multi-device Cisco enterprise environments |
| Primary Use Case | Integrated enterprise baseline readiness   |
| Version          | 1.0                                        |
| Last Updated     | 2026-09-15                                 |
