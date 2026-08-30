# Required Runbooks

## Status

**Status:** Draft  
**Last Updated:** 2026-08-30  
**Applies To:** Talos Solutions Enterprise Campus v1  
**Document Type:** Planning / Build Reference  

---

## Purpose

This document tracks the reusable runbooks required to build the Talos Solutions Enterprise Campus v1 lab.

The purpose of this document is to identify which Exists module runbooks can be reused, which runbooks need to be split or replaced, which runbooks are in progress, and which runbooks still need to be created before the enterprise lab can be built cleanly.

---

## Scope

### In Scope

- Required v1 runbooks
- Exists reusable runbooks
- Runbooks that need to be split or replaced
- Runbooks currently in progress
- Runbooks still needed
- Suggested module placement
- Purpose of each runbook
- Enterprise lab dependency notes

### Out of Scope

- Full runbook content
- Device configurations
- Verification output
- Troubleshooting procedures
- Final enterprise lab build documentation
- Future v2 multi-site runbooks

---

## Summary

The Talos Solutions Enterprise Campus v1 lab should be built from reusable module runbooks where possible.

The enterprise lab is not intended to replace the module runbooks. Instead, the enterprise lab should consume validated module patterns and combine them into a larger integrated topology.

The required v1 runbook set covers:

- Device baseline preparation
- VLAN creation
- Access ports
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
- ASAv routed firewall and PAT
- ASAv active/standby failover
- Enterprise baseline verification
- Enterprise failure and redundancy testing

---

## Runbook Status Values

| Status | Meaning |
|---|---|
| Exists | Runbook already exists and can be reused for the enterprise lab |
| In progress | Runbook is actively being built or refined |
| Needed | Runbook still needs to be created before the enterprise build is complete |
| Retire after replacement | Existing runbook should be retired after cleaner replacement runbooks are created |
| Optional future | Useful later, but not required for baseline v1 |

---

## Required Runbook Set

This table tracks the active runbooks required for the v1 enterprise build.

Folder placement is intentionally not finalized in this document. The priority is to identify which runbooks are required, what their current status is, and how they support the enterprise lab.

| Runbook | Status | Enterprise Use | Notes |
|---|---|---|---|
| `rapid-pvst-root-bridge-placement.md` | Exists | STP root placement | Reused for CORE1/CORE2 preferred root placement |
| `hsrp.md` | Exists | Gateway redundancy | Reused for CORE1/CORE2 HSRP gateway design |
| `basic-dhcp-server.md` | Exists | DHCP server | Reused for INFRA1 DHCP service |
| `vlan-creation-and-access-ports.md` | Needed | VLANs and access ports | Replaces part of the old combined VLAN/trunk/SVI runbook |
| `802.1q-trunking.md` | Needed | Trunking | Replaces part of the old combined VLAN/trunk/SVI runbook |
| `svi-and-inter-vlan-routing.md` | Needed | SVIs and inter-VLAN routing | Replaces part of the old combined VLAN/trunk/SVI runbook |
| `asav-routed-firewall-and-pat.md` | In progress | Firewall routed mode and PAT | Required for outbound firewall behavior |
| `asav-active-standby-failover.md` | In progress | Firewall HA | Required for FW1/FW2 active/standby failover |
| `access-switch-baseline.md` | Needed | Access switch preparation | Used by ASW1, ASW2, and ASW3 |
| `core-distribution-switch-baseline.md` | Needed | Core switch preparation | Used by CORE1 and CORE2 |
| `router-baseline.md` | Needed | Router preparation | Used by ISP1, EDGE1, and INFRA1 |
| `asav-baseline.md` | Needed | Firewall preparation | Used by FW1 and FW2 before HA, routing, and PAT |
| `lacp-etherchannel.md` | Needed | Port-channels | Used for inter-core and core-to-access LACP bundles |
| `static-routing-and-default-routes.md` | Needed | Static routing | Used by ISP1, EDGE1, FW1/FW2, CORE1/CORE2, and INFRA1 |
| `dhcp-relay.md` | Needed | DHCP relay | Used by CORE1/CORE2 SVIs for client VLANs |
| `stp-hsrp-alignment.md` | Needed | STP/HSRP alignment | Documents the design pattern that aligns L2 and L3 preferred paths |
| `enterprise-baseline-verification.md` | Needed | Integrated verification | Validates normal-state operation of the full enterprise lab |
| `enterprise-failure-and-redundancy-testing.md` | Needed | Failure testing | Validates failover and redundancy behavior across the integrated lab |

---

## Runbook Replacement / Retirement

The following Exists runbook should be retired after the replacement runbooks are created.

| Current Runbook | Status | Replacement Plan |
|---|---|---|
| `vlan-trunks-svi-inter-vlan-routing.md` | Retire after replacement | Split into `vlan-creation-and-access-ports.md`, `802.1q-trunking.md`, and `svi-and-inter-vlan-routing.md` |

Reason:

The current combined runbook covers too many separate concepts. Splitting it will make each replacement runbook easier to reuse, validate, and reference from the enterprise build.

---

## Final Active Runbook Count

After the combined VLAN/trunk/SVI/inter-VLAN routing runbook is retired and replaced by the three split runbooks, the v1 active runbook set should contain:

```text
18 active runbooks
```

Count:

| Status | Count |
|---|---:|
| Exists | 3 |
| In progress | 2 |
| Needed | 13 |
| Total active v1 runbooks | 18 |

The retired combined VLAN/trunk/SVI/inter-VLAN routing runbook should not be counted as an active v1 runbook after replacement.

---

## Enterprise Lab Dependency Map

| Enterprise Lab Area | Required Runbooks |
|---|---|
| Base device preparation | `access-switch-baseline.md`, `core-distribution-switch-baseline.md`, `router-baseline.md`, `asav-baseline.md` |
| VLAN and access ports | `vlan-creation-and-access-ports.md` |
| Trunking | `802.1q-trunking.md` |
| Port-channels | `lacp-etherchannel.md` |
| Core Layer 3 | `svi-and-inter-vlan-routing.md` |
| Gateway redundancy | `hsrp.md` |
| STP root placement | `rapid-pvst-root-bridge-placement.md` |
| STP/HSRP design alignment | `stp-hsrp-alignment.md` |
| Static routing | `static-routing-and-default-routes.md` |
| DHCP server | `basic-dhcp-server.md` |
| DHCP relay | `dhcp-relay.md` |
| Firewall baseline | `asav-baseline.md` |
| Firewall routed mode and PAT | `asav-routed-firewall-and-pat.md` |
| Firewall HA | `asav-active-standby-failover.md` |
| Integrated verification | `enterprise-baseline-verification.md` |
| Failure testing | `enterprise-failure-and-redundancy-testing.md` |

---

## Runbook Purpose Details

### `access-switch-baseline.md`

Purpose:

- Prepare Layer 2 access switches for enterprise use.

Expected coverage:

- Hostname
- Disable DNS lookup
- Basic console settings
- VLAN creation where needed
- Access switch management SVI
- Default gateway
- PortFast default
- BPDU Guard default
- Interface descriptions
- Unused port handling
- Save configuration

Used by:

- ASW1
- ASW2
- ASW3
- Possibly OS1 if kept as a simple Layer 2 switch baseline case

---

### `core-distribution-switch-baseline.md`

Purpose:

- Prepare collapsed core/distribution switches for Layer 2 and Layer 3 campus services.

Expected coverage:

- Hostname
- Disable DNS lookup
- Enable `ip routing`
- Basic console settings
- VLAN creation readiness
- SVI readiness
- Trunk readiness
- Interface descriptions
- Save configuration

Used by:

- CORE1
- CORE2

---

### `router-baseline.md`

Purpose:

- Prepare IOS routers used as routed infrastructure nodes.

Expected coverage:

- Hostname
- Disable DNS lookup
- Basic console settings
- Routed interface preparation
- Static route readiness
- Interface descriptions
- Save configuration

Used by:

- ISP1
- EDGE1
- INFRA1

Design note:

INFRA1 is a DHCP server, but it is still an IOSv router from a baseline configuration perspective.

---

### `asav-baseline.md`

Purpose:

- Prepare ASAv firewalls before routing, NAT, policy, or failover configuration.

Expected coverage:

- Hostname
- Interface naming readiness
- Interface descriptions
- Basic interface state
- Security-level planning
- Basic management considerations
- Save configuration

Used by:

- FW1
- FW2

---

### `vlan-creation-and-access-ports.md`

Purpose:

- Configure VLANs and access ports on IOSvL2 switches.

Expected coverage:

- VLAN creation
- VLAN naming
- Access port assignment
- Parking VLAN
- Unused port shutdown
- Basic access port validation

Used by:

- OS1
- CORE1
- CORE2
- ASW1
- ASW2
- ASW3

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

- CORE1
- CORE2
- ASW1
- ASW2
- ASW3

---

### `svi-and-inter-vlan-routing.md`

Purpose:

- Configure SVIs and validate inter-VLAN routing.

Expected coverage:

- SVI creation
- SVI IP addressing
- `ip routing`
- Connected route validation
- Inter-VLAN ping testing
- Troubleshooting down/down or up/down SVIs

Used by:

- CORE1
- CORE2

---

### `lacp-etherchannel.md`

Purpose:

- Configure and validate LACP EtherChannels.

Expected coverage:

- LACP active mode
- Member interface configuration
- Port-channel interface configuration
- Trunking over port-channel
- EtherChannel verification
- Common mismatch troubleshooting

Used by:

- CORE1 to CORE2
- CORE1/CORE2 to ASW1
- CORE1/CORE2 to ASW2
- CORE1/CORE2 to ASW3

---

### `rapid-pvst-root-bridge-placement.md`

Purpose:

- Configure and validate Rapid PVST+ root bridge placement.

Expected coverage:

- STP mode
- Root primary
- Root secondary
- Per-VLAN root placement
- STP verification
- Basic STP troubleshooting

Used by:

- CORE1
- CORE2
- ASW1
- ASW2
- ASW3

---

### `hsrp.md`

Purpose:

- Configure and validate HSRP gateway redundancy.

Expected coverage:

- HSRP VIPs
- Active/standby roles
- Priority
- Preemption
- Gateway testing
- Failover testing

Used by:

- CORE1
- CORE2

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

- CORE1
- CORE2
- ASW1
- ASW2
- ASW3

Design note:

This deserves its own runbook because STP/HSRP alignment is a core design principle of the enterprise lab.

---

### `static-routing-and-default-routes.md`

Purpose:

- Configure and validate static routes and default routes.

Expected coverage:

- Static default routes
- Specific static routes
- Route summaries
- Next-hop validation
- Routing table verification
- Ping/traceroute validation

Used by:

- ISP1
- EDGE1
- FW1/FW2
- CORE1
- CORE2
- INFRA1

---

### `basic-dhcp-server.md`

Purpose:

- Configure and validate a basic IOS DHCP server.

Expected coverage:

- DHCP service
- Excluded addresses
- DHCP pools
- Default router option
- Optional DNS/domain options
- DHCP binding verification
- DHCP pool verification

Used by:

- INFRA1

---

### `dhcp-relay.md`

Purpose:

- Configure and validate DHCP relay from routed VLANs to INFRA1.

Expected coverage:

- `ip helper-address`
- Relay on client VLAN SVIs
- DHCP relay traffic flow
- Client lease validation
- Common relay troubleshooting

Used by:

- CORE1
- CORE2

---

### `asav-routed-firewall-and-pat.md`

Purpose:

- Configure and validate routed ASAv firewall behavior and outbound PAT.

Expected coverage:

- Inside/outside interfaces
- Security levels
- Static routing
- Object NAT or manual NAT
- PAT validation
- Basic stateful traffic behavior
- Firewall route validation

Used by:

- FW1
- FW2 as HA pair members

---

### `asav-active-standby-failover.md`

Purpose:

- Configure and validate ASAv active/standby failover.

Expected coverage:

- Failover roles
- Failover/state link
- Active/standby interface addressing
- Failover status validation
- Interface monitoring
- Active firewall failure test
- Standby takeover test

Used by:

- FW1
- FW2

---

### `enterprise-baseline-verification.md`

Purpose:

- Validate normal-state operation of the fully integrated enterprise lab.

Expected coverage:

- Interface status
- VLANs
- Trunks
- Port-channels
- STP
- HSRP
- Routing
- DHCP
- Firewall HA
- PAT
- Management reachability
- End-to-end client testing

Used by:

- Full Talos Solutions Enterprise Campus v1 topology

---

### `enterprise-failure-and-redundancy-testing.md`

Purpose:

- Validate failure and redundancy behavior across the integrated enterprise lab.

Expected coverage:

- Single LACP member failure
- Full uplink bundle failure
- CORE1 failure
- CORE2 failure
- HSRP failover
- STP reconvergence
- Firewall active/standby failover
- Firewall failback if tested
- DHCP behavior after failover
- PAT behavior after firewall failover
- Client reachability before and after failures

Used by:

- Full Talos Solutions Enterprise Campus v1 topology

---

## Optional Future Runbooks

The following runbooks may be useful later but are not required for baseline v1:

| Runbook | Purpose |
|---|---|
| `dns-server.md` | Configure lightweight DNS service |
| `syslog-server.md` | Configure centralized syslog collection |
| `ntp-server.md` | Configure time synchronization |
| `ssh-management.md` | Configure SSH access and local users |
| `management-acls.md` | Restrict management-plane access |
| `port-security.md` | Add access port security |
| `dhcp-snooping.md` | Add DHCP snooping |
| `dynamic-arp-inspection.md` | Add DAI |
| `ip-source-guard.md` | Add IP Source Guard |
| `snmpv3.md` | Add SNMPv3 monitoring |
| `netflow.md` | Add flow export |
| `dmz-and-inbound-nat.md` | Add DMZ and inbound NAT testing |
| `site-to-site-vpn.md` | Add VPN connectivity |
| `ospf-integrated-enterprise.md` | Add dynamic routing to integrated topology |
| `ansible-enterprise-build.md` | Automate enterprise lab configuration |
| `pyats-enterprise-validation.md` | Automate enterprise lab validation |

---

## Runbook Completion Criteria

A runbook is considered ready for enterprise use when it includes:

- Purpose
- Scope
- Topology assumptions
- Configuration steps
- Inline command explanations or comments
- Verification steps
- Expected results
- Troubleshooting notes
- Cleanup or reset notes if needed
- Related enterprise lab dependencies

---

## Relationship to Enterprise Documents

The runbooks explain how to configure and validate individual technologies.

The enterprise documents explain how those technologies are combined into the Talos Solutions Enterprise Campus v1 design.

The distinction should remain clear:

| Artifact Type | Purpose |
|---|---|
| Module runbook | Teaches or documents one reusable technology pattern |
| Enterprise design doc | Explains how the technology is used in the enterprise lab |
| Enterprise config file | Stores the final device configuration |
| Enterprise verification doc | Proves the integrated lab works |

---

## Design Notes

- The enterprise lab should consume validated module runbooks rather than reinvent each technology from scratch.
- Some runbooks are reusable across many future labs.
- Enterprise-specific behavior should be documented in the enterprise docs, not forced into generic runbooks.
- OS1 does not need its own dedicated runbook unless its role grows.
- The STP/HSRP alignment runbook is valuable because it connects two separate technologies into one enterprise design pattern.
- Enterprise baseline verification and failure testing should remain separate from individual technology runbooks.

---

## Validation or Success Criteria

This required runbooks document is successful when:

- Every v1 technology dependency is mapped to a runbook.
- Exists reusable runbooks are identified.
- Runbooks needing replacement are identified.
- In-progress runbooks are tracked.
- Missing runbooks are clearly listed.
- The enterprise lab build can reference this document during planning.
- The runbook set supports the full v1 build and validation process.
- Future runbooks are captured without expanding baseline v1 scope.

---

## Open Questions


---

## Related Documents

- `docs/01-initial-planning.md`
- `docs/02-documentation-plan.md`
- `docs/03-topology-and-device-roles.md`
- `docs/04-design-decisions.md`
- `docs/05-vlan-plan.md`
- `docs/06-addressing-plan.md`
- `docs/08-port-channel-plan.md`
- `docs/09-routing-plan.md`
- `docs/10-firewall-plan.md`
- `docs/11-dhcp-plan.md`
- `docs/13-build-order.md`
- `docs/14-known-limitations.md`
- `docs/15-future-roadmap.md`
- `docs/07-interface-map.md`
- `docs/12-management-plan.md`

---

## Change Log

| Date | Change |
|---|---|
| 2026-08-30 | Initial draft |