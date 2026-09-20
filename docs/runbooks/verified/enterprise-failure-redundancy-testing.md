# Runbook — Enterprise Failure and Redundancy Testing
 
## Overview
 
This runbook provides a structured process for validating failure behavior, redundancy, convergence, and recovery across a completed enterprise network design.
 
Enterprise failure and redundancy testing confirms that redundant Layer 2, Layer 3, gateway, firewall, and uplink components behave as intended when links, devices, or active network roles become unavailable. It also confirms that the environment returns to the expected steady state after failed components are restored.
 
This runbook focuses on integrated failure testing after individual network features have already been configured and independently validated. The tests verify the behavior of the complete enterprise topology rather than re-validating each feature in isolation.
 
Typical failure scenarios include EtherChannel member loss, access uplink failure, HSRP gateway failover, spanning-tree reconvergence, firewall failover, and recovery to the intended normal operating state.
 
## Scope and Assumptions
 
This runbook assumes:
 
- The enterprise topology has been fully built and all required features have been configured.
- Individual feature runbooks have already been completed and validated successfully.
- The network is in a stable known-good state before failure testing begins.
- Redundant Layer 2, Layer 3, gateway, and firewall paths have been intentionally designed.
- HSRP, Rapid-PVST, EtherChannel, routing, DHCP relay, firewalling, PAT, and ASAv failover are operational before testing begins.
- Test endpoints are available to verify end-to-end connectivity during failure scenarios.
- The expected normal and failover behavior for each tested component has been documented.
- Failure testing is performed only in a lab or approved maintenance window.
- Each failed component can be safely restored to service after the test.
 
This runbook does not cover initial feature configuration or standalone feature validation. It focuses on integrated enterprise behavior during controlled failures and recovery.

---

## Reference Design
 
This runbook uses a representative redundant enterprise topology with dual core/distribution switches, redundant access uplinks, HSRP gateway redundancy, Rapid-PVST root placement, LACP EtherChannels, and an ASAv active/standby firewall pair.
 
### Devices
 
| Device | Role                          | Redundancy Function                    |
|--------|-------------------------------|----------------------------------------|
| CORE1  | Core/distribution switch      | HSRP, STP root, routed core services   |
| CORE2  | Core/distribution switch      | HSRP, STP secondary/root role          |
| ASW1   | Access switch                 | Redundant uplinks toward the core      |
| ASW2   | Access switch                 | Redundant uplinks toward the core      |
| FW1    | Primary ASAv firewall         | Normal active firewall                 |
| FW2    | Secondary ASAv firewall       | Standby firewall                       |
| EDGE1  | Upstream edge router          | External connectivity                  |
| PC1    | Test endpoint                 | End-to-end connectivity validation     |
 
### Failure Scenarios
 
| Test | Failure Scenario                | Expected Redundancy Behavior                |
|------|---------------------------------|---------------------------------------------|
| 1    | EtherChannel member failure     | Port-Channel remains operational            |
| 2    | Access uplink failure           | Alternate Layer 2 path becomes active       |
| 3    | HSRP active gateway failure     | Standby gateway assumes active role         |
| 4    | Primary STP path failure        | Rapid-PVST reconverges to alternate path    |
| 5    | Active firewall failover        | Standby ASAv assumes active role            |
| 6    | Component restoration           | Network returns to intended steady state    |
 
**Note:** Device names, failure scenarios, and redundancy mechanisms shown in this reference design are examples. Replace them with those appropriate to the target enterprise topology.

---

## Prerequisites and Pre-Checks
 
Before beginning enterprise failure and redundancy testing, confirm that the completed network is stable and that each redundancy feature is operating normally.
 
### Prerequisites
 
- [ ] All enterprise devices are powered on and accessible.
- [ ] All required feature-specific configuration has been completed.
- [ ] Individual feature validation has been completed successfully.
- [ ] HSRP is operational for all redundant gateway VLANs.
- [ ] Rapid-PVST root and alternate paths match the intended design.
- [ ] LACP EtherChannels are fully formed with all expected members bundled.
- [ ] Redundant access uplinks are operational.
- [ ] ASAv active/standby failover is healthy and synchronized.
- [ ] Routing, DHCP relay, firewalling, and PAT are operating as intended.
- [ ] Test endpoints have valid addressing and can reach all required test destinations.
- [ ] The network is in its documented normal operating state before any failure is introduced.
- [ ] HSRP preemption is configured where the preferred gateway is expected to automatically resume the active role after recovery.
 
### Baseline Verification

Run the applicable commands before beginning controlled failure testing.

On Cisco IOS switches:

```bash
show interfaces status
show etherchannel summary
show spanning-tree root
show standby brief
show ip route
```

On Cisco IOS routers:

```bash
show ip interface brief
show ip route
```

On Cisco ASAv firewalls:

```bash
show failover
show interface ip brief
show route
show nat detail
```

From the test endpoint, verify normal end-to-end connectivity using the appropriate test commands for the environment.
 
### Expected Baseline Results
 
- [ ] All expected physical and logical interfaces are operational.
- [ ] EtherChannels contain all expected active member links.
- [ ] Spanning-tree root placement matches the intended design.
- [ ] HSRP active and standby roles match the intended design.
- [ ] Required routes are installed.
- [ ] FW1 is active and FW2 is standby ready.
- [ ] ASAv failover and state synchronization are healthy.
- [ ] Required NAT and PAT rules are present.
- [ ] Test endpoints have expected network connectivity before failure testing begins.
- [ ] No unresolved fault or degraded redundancy state is present.

---

## Failure and Redundancy Testing Procedure
 
Use this procedure to introduce controlled failures one at a time and confirm that the integrated enterprise network continues operating as designed.
 
### Testing Notes
 
- Begin each test from the documented normal operating state.
- Introduce only one failure at a time unless a multi-failure scenario is explicitly being tested.
- Maintain end-to-end test traffic during each failure whenever practical.
- Record the observed convergence behavior and any temporary packet loss.
- Restore the failed component before beginning the next test.
- Re-run the applicable baseline checks after each restoration.
- If the network does not recover to the expected state, stop testing and resolve the issue before introducing another failure.
 
---
 
### Step 1 — Test EtherChannel Member Failure
 
Confirm the Port-Channel and all expected member links are operational before introducing the failure.
 
Run on the affected switch:
 
```bash
show etherchannel summary
show interfaces port-channel 1
```
 
Shut down one physical EtherChannel member.
 
```bash
enable
configure terminal
 
interface <member-interface>
shutdown
 
end
```
 
Verify the remaining bundle.
 
```bash
show etherchannel summary
show interfaces port-channel 1
```
 
Expected results:
 
- [ ] The failed physical member is removed from active forwarding.
- [ ] The remaining EtherChannel member or members remain bundled.
- [ ] The Port-Channel remains operational.
- [ ] VLAN trunking remains available across the Port-Channel.
- [ ] End-to-end test traffic continues across the network.
- [ ] No unintended spanning-tree topology change occurs because of a single member failure.
 
Restore the failed member.
 
```bash
enable
configure terminal
 
interface <member-interface>
no shutdown
 
end
```
 
Confirm the restored member rejoins the EtherChannel.
 
---
 
### Step 2 — Test Access Uplink Failure
 
Confirm the access switch has its expected primary and alternate Layer 2 paths.
 
Run on the affected access switch:
 
```bash
show spanning-tree root
show spanning-tree vlan <vlan-id>
show interfaces trunk
```
 
Shut down the active access uplink.
 
```bash
enable
configure terminal
 
interface <active-uplink>
shutdown
 
end
```
 
Verify Layer 2 convergence.
 
```bash
show spanning-tree vlan <vlan-id>
show interfaces trunk
```
 
Expected results:
 
- [ ] The failed uplink is removed from forwarding.
- [ ] The alternate access-layer path transitions to the required forwarding state.
- [ ] The access switch retains connectivity to the enterprise network.
- [ ] End-to-end test traffic resumes or continues after convergence.
- [ ] No Layer 2 loop forms.
 
Restore the failed uplink and confirm the topology returns to the intended normal state.
 
---
 
### Step 3 — Test HSRP Active Gateway Failure
 
Confirm the normal HSRP roles before introducing the failure.
 
Run on both gateway switches:
 
```bash
show standby brief
```
 
Disable the SVI or other component providing the active HSRP gateway role for the test VLAN.
 
```bash
enable
configure terminal
 
interface vlan <vlan-id>
shutdown
 
end
```
 
Verify HSRP convergence.
 
```bash
show standby brief
```
 
Expected results:
 
- [ ] The former standby gateway assumes the active HSRP role.
- [ ] The HSRP virtual IP address remains available.
- [ ] Client default-gateway addressing does not require modification.
- [ ] End-to-end client connectivity resumes or continues after convergence.
- [ ] The surviving gateway provides Layer 3 forwarding for the affected VLAN.
 
Restore the original gateway SVI.
 
```bash
enable
configure terminal
 
interface vlan <vlan-id>
no shutdown
 
end
```
 
Confirm the resulting HSRP roles match the intended design.
 
---
 
### Step 4 — Test Primary Spanning-Tree Path Failure
 
Confirm the expected root bridge and forwarding topology.
 
Run on the applicable switches:
 
```bash
show spanning-tree vlan <vlan-id>
```
 
Disable the Layer 2 path currently used toward the primary root.
 
```bash
enable
configure terminal
 
interface <primary-path-interface>
shutdown
 
end
```
 
Verify Rapid-PVST convergence.
 
```bash
show spanning-tree vlan <vlan-id>
```
 
Expected results:
 
- [ ] The failed path is removed from the active topology.
- [ ] An alternate path transitions to forwarding.
- [ ] The surviving Layer 2 topology remains loop free.
- [ ] End-to-end traffic resumes or continues after convergence.
- [ ] Gateway reachability remains available through the surviving topology.
 
Restore the failed path and confirm spanning tree returns to the intended steady state.
 
---
 
### Step 5 — Test ASAv Active Firewall Failover
 
Confirm the normal firewall state before initiating failover.
 
Run on both ASAv firewalls:
 
```bash
show failover
```
 
Initiate a controlled role change using the ASAv active/standby failover procedure.
 
Verify the resulting state:
 
```bash
show failover
show interface ip brief
show route
show nat detail
```
 
Expected results:
 
- [ ] The standby firewall assumes the active role.
- [ ] The former active firewall transitions to standby.
- [ ] Inside and outside firewall connectivity remains available.
- [ ] Required routing remains present on the active firewall.
- [ ] PAT remains available for matching inside-to-outside traffic.
- [ ] End-to-end client connectivity resumes or continues through the firewall pair.
- [ ] No unexpected failed or synchronization state is present.
 
Return the firewall pair to the intended normal operating roles and verify the final state.
 
---
 
On Cisco IOS switches:

```bash
show interfaces status
show etherchannel summary
show spanning-tree root
show standby brief
show ip route
```

On Cisco IOS routers:

```bash
show ip interface brief
show ip route
```

On Cisco ASAv firewalls:

```bash
show failover
show interface ip brief
show route
show nat detail
```
 
From the test endpoint, repeat the normal end-to-end connectivity tests.
 
Expected results:
 
- [ ] All intentionally failed interfaces have returned to service.
- [ ] EtherChannels contain all expected members.
- [ ] Spanning-tree root placement matches the intended design.
- [ ] HSRP roles match the intended normal operating state.
- [ ] Required routes are installed.
- [ ] FW1 and FW2 have returned to the intended active/standby state.
- [ ] Firewall synchronization is healthy.
- [ ] PAT and routed firewall behavior are operational.
- [ ] End-to-end client connectivity matches the pre-test baseline.
- [ ] No degraded redundancy state remains after testing.

---

## If Validation Fails
 
If any enterprise failure or redundancy test does not produce the expected results:
 
- Identify the specific failure scenario that did not behave as expected.
- Restore the intentionally failed component to its normal operating state.
- Confirm that the network returns to the documented pre-test baseline before continuing.
- Review the applicable feature-specific runbook for the affected technology.
- Confirm that the underlying redundancy feature is healthy before repeating the integrated test.
- Review Layer 2, Layer 3, gateway, routing, and firewall dependencies associated with the failed scenario.
- Compare observed convergence behavior against the expected design.
- Correct the affected configuration or design issue.
- Re-run the failed test from a known-good starting state.
- Do not continue to additional failure scenarios while an unresolved redundancy or recovery issue remains.

---

## Backout Considerations
 
Controlled failure testing intentionally disrupts active network components and can temporarily affect traffic while redundancy mechanisms converge.
 
Before beginning or continuing failure testing, confirm:
 
- The normal operating state of each tested component is documented.
- The configuration and expected role of each redundant component are known.
- Each intentionally failed interface or device can be restored safely.
- The expected Layer 2, Layer 3, gateway, routing, and firewall dependencies are understood.
- Test endpoints and validation paths are available to confirm recovery.
- Only one failure scenario is introduced at a time unless a multi-failure test is explicitly planned.
- The environment can be returned to its pre-test known-good state before additional testing continues.
 
If a test causes unexpected behavior, restore the failed component immediately and confirm the network returns to the documented baseline before proceeding.
 
For lab use, return all intentionally failed interfaces and devices to their normal operating state and re-run the final recovery validation before ending the test session.

---

## Document Metadata
 
| Field            | Value                                      |
|------------------|--------------------------------------------|
| Author           | Aaron Kindelt                              |
| Category         | Validation / Resiliency                    |
| Technology       | Enterprise Network Redundancy              |
| Applies To       | Multi-device Cisco enterprise environments |
| Primary Use Case | Integrated failure and redundancy testing  |
| Version          | 1.0                                        |
| Last Updated     | 2026-09-16                                 |
