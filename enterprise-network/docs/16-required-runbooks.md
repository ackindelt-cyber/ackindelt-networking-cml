# Required Runbooks

## Status

**Status:** Draft  
**Last Updated:** 2026-09-10  
**Applies To:** Talos Solutions Enterprise Campus v1  
**Document Type:** Planning / Build Reference  

---

## Purpose

This document tracks the reusable runbooks required to build the Talos Solutions Enterprise Campus v1 lab.

The purpose of this document is to identify which reusable runbooks are available, which runbooks are still in progress, which runbooks still need to be created, and how each runbook supports the integrated enterprise build.

---

## Scope

### In Scope

- Required v1 runbooks
- Available reusable runbooks
- Runbooks currently in progress
- Runbooks still needed
- Purpose of each runbook
- Enterprise lab dependencies
- Runbooks scheduled for replacement or retirement

### Out of Scope

- Full runbook content
- Final device configurations
- Recorded verification output
- Final troubleshooting documentation
- Final enterprise lab build documentation
- Future v2 multi-site runbooks

---

## Summary

The Talos Solutions Enterprise Campus v1 lab is built from reusable module runbooks wherever practical.

The enterprise lab does not replace the individual module runbooks. Instead, the enterprise build consumes validated configuration and verification patterns and combines them into a larger integrated topology.

The required v1 runbook set covers:

- Device baseline preparation
- Outside transit switch preparation
- VLAN creation and access ports
- 802.1Q trunking
- SVIs and inter-VLAN routing
- LACP EtherChannel
- Rapid PVST+ root bridge placement
- HSRP
- STP/HSRP alignment
- Static and default routing
- DHCP server
- DHCP relay
- ASAv baseline
- ASAv routed firewall operation and PAT
- ASAv active/standby failover
- Enterprise baseline verification
- Enterprise failure and redundancy testing

---

## Current Enterprise Devices

| Device   | Role                                      |
|----------|-------------------------------------------|
| ISP1     | Simulated upstream ISP router             |
| TSE1     | Enterprise edge router                    |
| TSOS1    | Outside transit switch                    |
| TSFW1    | Primary ASAv firewall                     |
| TSFW2    | Secondary ASAv firewall                   |
| TSCOR1   | Core/distribution switch                  |
| TSCOR2   | Core/distribution switch                  |
| TSAS1    | Access switch                             |
| TSAS2    | Access switch                             |
| TSAS3    | Access switch                             |
| TSINF1   | Infrastructure router / DHCP server       |
| TSPC1    | Client endpoint                           |
| TSPC2    | Client endpoint                           |
| TSPC3    | Client endpoint                           |

---

## Runbook Status Values

| Status                   | Meaning                                                                    |
|--------------------------|----------------------------------------------------------------------------|
| Available                | Runbook exists and can be reused for the enterprise lab                    |
| In progress              | Runbook is actively being built or refined                                 |
| Needed                   | Runbook still needs to be created before the enterprise build is complete  |
| Retire after replacement | Existing runbook should be retired after replacement runbooks are complete |
| Optional future          | Useful later, but not required for enterprise v1                           |

---

## Required Runbook Set

This table tracks the active runbooks required for the Talos Solutions Enterprise Campus v1 build.

| Runbook                                         | Status      | Enterprise Use               | Notes                                                       |
|-------------------------------------------------|-------------|------------------------------|-------------------------------------------------------------|
| `access-switch-baseline.md`                     | Available   | Access switch preparation    | Used by TSAS1, TSAS2, and TSAS3                             |
| `core-distribution-switch-baseline.md`          | Available   | Core switch preparation      | Used by TSCOR1 and TSCOR2                                   |
| `router-baseline.md`                            | Available   | IOS router preparation       | Used by ISP1, TSE1, and TSINF1                              |
| `asav-baseline.md`                              | Available   | Firewall preparation         | Used by TSFW1 and TSFW2                                     |
| `outside-transit-switch-baseline.md`            | Available   | Outside switch preparation   | Used by TSOS1                                               |
| `rapid-pvst-root-bridge-placement.md`           | Available   | STP root placement           | Used for TSCOR1/TSCOR2 preferred root placement             |
| `hsrp.md`                                       | Available   | Gateway redundancy           | Used for TSCOR1/TSCOR2 HSRP gateway design                  |
| `basic-dhcp-server.md`                          | Available   | DHCP server                  | Used by TSINF1                                              |
| `vlan-creation-and-access-ports.md`             | Needed      | VLANs and access ports       | Replaces part of the older combined VLAN runbook            |
| `802.1q-trunking.md`                            | Needed      | Trunking                     | Replaces part of the older combined VLAN runbook            |
| `svi-and-inter-vlan-routing.md`                 | Needed      | SVIs and inter-VLAN routing  | Replaces part of the older combined VLAN runbook            |
| `lacp-etherchannel.md`                          | Needed      | Port-channels                | Used for inter-core and core-to-access LACP bundles         |
| `static-routing-and-default-routes.md`          | Needed      | Static routing               | Used across routed infrastructure                           |
| `dhcp-relay.md`                                 | Needed      | DHCP relay                   | Used on TSCOR1/TSCOR2 client VLAN SVIs                      |
| `stp-hsrp-alignment.md`                         | Needed      | STP/HSRP alignment           | Aligns Layer 2 and Layer 3 preferred forwarding paths       |
| `asav-routed-firewall-and-pat.md`               | In progress | Firewall routing and PAT     | Required for outbound firewall behavior                     |
| `asav-active-standby-failover.md`               | In progress | Firewall HA                  | Required for TSFW1/TSFW2 active/standby operation           |
| `enterprise-baseline-verification.md`           | Needed      | Integrated verification      | Validates normal-state operation of the complete topology   |
| `enterprise-failure-and-redundancy-testing.md`  | Needed      | Failure testing              | Validates redundancy and failover across the complete lab   |

---

## Final Active Runbook Count

After the older combined VLAN/trunk/SVI/inter-VLAN routing runbook is retired and replaced by the three focused runbooks, the v1 active runbook set contains:

```text
19 active runbooks
```

| Status                   | Count |
|--------------------------|-------|
| Available                | 8     |
| In progress              | 2     |
| Needed                   | 9     |
| Total active v1 runbooks | 19    |

---

## Runbook Replacement / Retirement

The following older combined runbook should be retired after its replacement runbooks are completed.

| Current Runbook                         | Status                   | Replacement Plan                                                                                                     |
|-----------------------------------------|--------------------------|----------------------------------------------------------------------------------------------------------------------|
| `vlan-trunks-svi-inter-vlan-routing.md` | Retire after replacement | Split into `vlan-creation-and-access-ports.md`, `802.1q-trunking.md`, and `svi-and-inter-vlan-routing.md`             |

The existing combined runbook covers too many independent concepts. Splitting it makes each procedure easier to reuse, validate, maintain, and reference from the enterprise build.

---

## Enterprise Lab Dependency Map

| Enterprise Lab Area          | Required Runbooks                                                                                                                          |
|------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------|
| Access switch baseline       | `access-switch-baseline.md`                                                                                                                |
| Core switch baseline         | `core-distribution-switch-baseline.md`                                                                                                     |
| Router baseline              | `router-baseline.md`                                                                                                                       |
| Outside switch baseline      | `outside-transit-switch-baseline.md`                                                                                                       |
| Firewall baseline            | `asav-baseline.md`                                                                                                                         |
| VLANs and access ports       | `vlan-creation-and-access-ports.md`                                                                                                        |
| Trunking                     | `802.1q-trunking.md`                                                                                                                       |
| Port-channels                | `lacp-etherchannel.md`                                                                                                                     |
| Core Layer 3                 | `svi-and-inter-vlan-routing.md`                                                                                                            |
| Gateway redundancy           | `hsrp.md`                                                                                                                                  |
| STP root placement           | `rapid-pvst-root-bridge-placement.md`                                                                                                      |
| STP/HSRP alignment           | `stp-hsrp-alignment.md`                                                                                                                    |
| Static routing               | `static-routing-and-default-routes.md`                                                                                                     |
| DHCP server                  | `basic-dhcp-server.md`                                                                                                                     |
| DHCP relay                   | `dhcp-relay.md`                                                                                                                            |
| Firewall routed mode and PAT | `asav-routed-firewall-and-pat.md`                                                                                                          |
| Firewall HA                  | `asav-active-standby-failover.md`                                                                                                          |
| Integrated verification      | `enterprise-baseline-verification.md`                                                                                                      |
| Failure testing              | `enterprise-failure-and-redundancy-testing.md`                                                                                             |

---

## Runbook Purpose Details

### `access-switch-baseline.md`

Purpose:

- Prepare Layer 2 access switches for enterprise deployment.

Expected coverage:

- Hostname
- Disable DNS lookup
- Disable Layer 3 routing
- Basic console settings
- Management VLAN
- Management SVI
- Management default gateway
- PortFast default
- BPDU Guard default
- Parking VLAN
- Unused port handling
- Baseline validation
- Save configuration

Used by:

- TSAS1
- TSAS2
- TSAS3

---

### `core-distribution-switch-baseline.md`

Purpose:

- Prepare collapsed core/distribution switches for Layer 2 and Layer 3 campus services.

Expected coverage:

- Hostname
- Disable DNS lookup
- Enable `ip routing`
- Basic console settings
- Management VLAN
- Management SVI
- Parking VLAN
- Unused port handling
- Layer 3 baseline validation
- Save configuration

Used by:

- TSCOR1
- TSCOR2

---

### `router-baseline.md`

Purpose:

- Prepare Cisco IOS routers used as routed infrastructure nodes.

Expected coverage:

- Hostname
- Disable DNS lookup
- Basic console settings
- Clean interface baseline state
- Clean routing baseline state
- Baseline validation
- Save configuration

Used by:

- ISP1
- TSE1
- TSINF1

Design note:

The same generic router baseline is reused for all three IOS routed infrastructure devices. Role-specific interface addressing, routing, and services are applied through separate runbooks.

---

### `outside-transit-switch-baseline.md`

Purpose:

- Prepare the dedicated Layer 2 outside transit switch connecting the enterprise edge router to the firewall pair.

Expected coverage:

- Hostname
- Disable DNS lookup
- Disable Layer 3 routing
- Basic console settings
- Outside transit VLAN
- TSE1-facing access port
- TSFW1-facing access port
- TSFW2-facing access port
- Parking VLAN
- Unused port shutdown
- Baseline validation
- Save configuration

Used by:

- TSOS1

---

### `asav-baseline.md`

Purpose:

- Prepare ASAv firewalls before production interface, routing, NAT, policy, or failover configuration.

Expected coverage:

- Hostname
- Dedicated management interface
- Management interface naming
- Management IP addressing
- Management-only interface behavior
- Baseline interface validation
- Baseline running-configuration validation
- Save configuration

Used by:

- TSFW1
- TSFW2

---

### `vlan-creation-and-access-ports.md`

Purpose:

- Configure VLANs and endpoint access ports on Cisco IOS Layer 2 and multilayer switches.

Expected coverage:

- VLAN creation
- VLAN naming
- Access port assignment
- Access port validation
- VLAN membership validation

Used by:

- TSCOR1
- TSCOR2
- TSAS1
- TSAS2
- TSAS3

Design note:

TSOS1 receives its outside transit VLAN and port assignments through its dedicated baseline runbook and does not require the normal campus access-port procedure for that role.

---

### `802.1q-trunking.md`

Purpose:

- Configure and validate 802.1Q trunks.

Expected coverage:

- Trunk mode
- Native VLAN
- Allowed VLAN list
- DTP disablement where supported
- Trunk verification
- Common trunk troubleshooting

Used by:

- TSCOR1
- TSCOR2
- TSAS1
- TSAS2
- TSAS3

---

### `svi-and-inter-vlan-routing.md`

Purpose:

- Configure SVIs and validate inter-VLAN routing.

Expected coverage:

- SVI creation
- SVI IP addressing
- Layer 3 switching
- Connected route validation
- Inter-VLAN reachability
- SVI operational-state validation

Used by:

- TSCOR1
- TSCOR2

---

### `lacp-etherchannel.md`

Purpose:

- Configure and validate LACP EtherChannels.

Expected coverage:

- LACP active mode
- Member interface configuration
- Port-channel interface configuration
- 802.1Q trunking over port-channel
- EtherChannel verification
- Common mismatch troubleshooting

Used by:

- TSCOR1 to TSCOR2
- TSCOR1/TSCOR2 to TSAS1
- TSCOR1/TSCOR2 to TSAS2
- TSCOR1/TSCOR2 to TSAS3

---

### `rapid-pvst-root-bridge-placement.md`

Purpose:

- Configure and validate Rapid PVST+ root bridge placement.

Expected coverage:

- STP mode
- Root primary placement
- Root secondary placement
- Per-VLAN root placement
- STP verification
- Basic STP troubleshooting

Used by:

- TSCOR1
- TSCOR2
- TSAS1
- TSAS2
- TSAS3

---

### `hsrp.md`

Purpose:

- Configure and validate HSRP gateway redundancy.

Expected coverage:

- HSRP virtual IP addresses
- Active/standby roles
- Priority
- Preemption
- Gateway testing
- Failover testing

Used by:

- TSCOR1
- TSCOR2

---

### `stp-hsrp-alignment.md`

Purpose:

- Align Layer 2 forwarding preference with Layer 3 gateway preference.

Expected coverage:

- STP root placement per VLAN
- HSRP active placement per VLAN
- Preferred core mapping
- Per-VLAN load sharing
- Validation of expected forwarding paths

Used by:

- TSCOR1
- TSCOR2
- TSAS1
- TSAS2
- TSAS3

Design note:

STP/HSRP alignment is documented separately because it combines two independent technologies into a single enterprise forwarding design.

---

### `static-routing-and-default-routes.md`

Purpose:

- Configure and validate static routes and default routes.

Expected coverage:

- Static default routes
- Specific static routes
- Route summaries where applicable
- Next-hop validation
- Routing table verification
- Ping and traceroute validation

Used by:

- ISP1
- TSE1
- TSFW1
- TSFW2
- TSCOR1
- TSCOR2
- TSINF1

---

### `basic-dhcp-server.md`

Purpose:

- Configure and validate a basic Cisco IOS DHCP server.

Expected coverage:

- DHCP service
- Excluded addresses
- DHCP pools
- Default-router option
- Optional DNS and domain options
- DHCP binding verification
- DHCP pool verification

Used by:

- TSINF1

---

### `dhcp-relay.md`

Purpose:

- Configure and validate DHCP relay from client VLANs to TSINF1.

Expected coverage:

- `ip helper-address`
- Relay configuration on client VLAN SVIs
- DHCP relay traffic flow
- Client lease validation
- Common relay troubleshooting

Used by:

- TSCOR1
- TSCOR2

---

### `asav-routed-firewall-and-pat.md`

Purpose:

- Configure and validate routed ASAv firewall behavior and outbound PAT.

Expected coverage:

- Inside and outside interfaces
- Interface security levels
- Firewall routing
- Object NAT or manual NAT
- PAT validation
- Stateful traffic behavior
- Firewall route validation

Used by:

- TSFW1
- TSFW2

Design note:

The routed firewall configuration must be compatible with the active/standby HA design used by the firewall pair.

---

### `asav-active-standby-failover.md`

Purpose:

- Configure and validate ASAv active/standby failover.

Expected coverage:

- Failover roles
- Failover and state link
- Active/standby interface addressing
- Failover status validation
- Interface monitoring
- Active firewall failure
- Standby takeover
- Recovery and failback behavior where tested

Used by:

- TSFW1
- TSFW2

---

### `enterprise-baseline-verification.md`

Purpose:

- Validate normal-state operation of the fully integrated Talos Solutions Enterprise Campus v1 topology.

Expected coverage:

- Physical interface status
- VLANs
- Access ports
- Trunks
- Port-channels
- STP
- HSRP
- Routing
- DHCP
- Firewall HA
- PAT
- Management reachability
- End-to-end client connectivity

Used by:

- Full Talos Solutions Enterprise Campus v1 topology

---

### `enterprise-failure-and-redundancy-testing.md`

Purpose:

- Validate failure and redundancy behavior across the integrated enterprise topology.

Expected coverage:

- Single LACP member failure
- Full uplink bundle failure
- TSCOR1 failure
- TSCOR2 failure
- HSRP failover
- STP reconvergence
- TSFW1/TSFW2 failover
- Firewall failback where tested
- DHCP behavior during infrastructure failure
- PAT behavior after firewall failover
- Client reachability before, during, and after failures

Used by:

- Full Talos Solutions Enterprise Campus v1 topology

---

## Optional Future Runbooks

The following runbooks may be useful later but are not required for enterprise v1:

| Runbook                         | Purpose                                  |
|---------------------------------|------------------------------------------|
| `dns-server.md`                 | Configure lightweight DNS service        |
| `syslog-server.md`              | Configure centralized syslog collection  |
| `ntp-server.md`                 | Configure time synchronization           |
| `ssh-management.md`             | Configure SSH access and local users     |
| `management-acls.md`            | Restrict management-plane access         |
| `port-security.md`              | Add access-port security                 |
| `dhcp-snooping.md`              | Add DHCP snooping                        |
| `dynamic-arp-inspection.md`     | Add Dynamic ARP Inspection               |
| `ip-source-guard.md`            | Add IP Source Guard                      |
| `snmpv3.md`                     | Add SNMPv3 monitoring                    |
| `netflow.md`                    | Add flow export                          |
| `dmz-and-inbound-nat.md`        | Add DMZ and inbound NAT testing          |
| `site-to-site-vpn.md`           | Add VPN connectivity                     |
| `ospf-integrated-enterprise.md` | Add dynamic routing to the topology      |
| `ansible-enterprise-build.md`   | Automate enterprise configuration        |
| `pyats-enterprise-validation.md`| Automate enterprise validation           |

---

## Runbook Completion Criteria

A runbook is considered ready for enterprise use when it includes:

- Purpose
- Scope
- Reference design or topology assumptions
- Prerequisites and pre-checks
- Configuration steps
- Inline command explanations or comments
- Post-configuration validation
- Expected results
- Validation failure guidance
- Backout considerations
- Enterprise lab dependencies where applicable

---

## Relationship to Enterprise Documents

The runbooks explain how to configure and validate reusable network technologies and device roles.

The enterprise documents explain how those reusable patterns are combined into the Talos Solutions Enterprise Campus v1 design.

| Artifact Type               | Purpose                                                   |
|-----------------------------|-----------------------------------------------------------|
| Module runbook              | Documents one reusable technology or device-role pattern  |
| Enterprise design document  | Explains how technologies are used in the enterprise lab  |
| Enterprise configuration    | Stores final device configuration                         |
| Enterprise verification     | Proves the integrated topology operates as designed       |
| Enterprise failure testing  | Proves redundancy and recovery behavior                   |

---

## Design Notes

- The enterprise lab should consume validated module runbooks rather than recreate each technology from scratch.
- Generic runbooks should remain independent of Talos Solutions-specific addressing and interface assignments.
- Enterprise-specific values belong in the enterprise design and implementation documentation.
- TSOS1 uses a dedicated outside transit switch baseline because its role differs from a normal campus access switch.
- ISP1, TSE1, and TSINF1 share the same generic IOS router baseline.
- STP/HSRP alignment remains a dedicated design runbook because it combines Layer 2 and Layer 3 forwarding preferences.
- Enterprise baseline verification and enterprise failure testing remain separate from individual technology runbooks.
- Individual technology runbooks should be validated independently before being consumed by the integrated enterprise build where practical.

---

## Validation or Success Criteria

This required runbooks document is successful when:

- Every v1 technology dependency is mapped to a runbook.
- Existing reusable runbooks are identified.
- Runbooks requiring replacement are identified.
- In-progress runbooks are tracked.
- Missing runbooks are clearly identified.
- Current Talos Solutions device names are used consistently.
- The enterprise build can reference this document during implementation.
- The active runbook set supports the complete v1 build and validation process.
- Future runbooks are captured without expanding enterprise v1 scope.

---

## Open Questions

None currently identified.

---

## Related Documents

- `docs/01-initial-planning.md`
- `docs/02-documentation-plan.md`
- `docs/03-topology-and-device-roles.md`
- `docs/04-design-decisions.md`
- `docs/05-vlan-plan.md`
- `docs/06-addressing-plan.md`
- `docs/07-interface-map.md`
- `docs/08-port-channel-plan.md`
- `docs/09-routing-plan.md`
- `docs/10-firewall-plan.md`
- `docs/11-dhcp-plan.md`
- `docs/12-management-plan.md`
- `docs/13-build-order.md`
- `docs/14-known-limitations.md`
- `docs/15-future-roadmap.md`

---

## Change Log

| Date       | Change                                                                                  |
|------------|-----------------------------------------------------------------------------------------|
| 2026-08-30 | Initial draft                                                                           |
| 2026-09-10 | Updated runbook status, added baseline runbooks, and aligned current enterprise naming  |