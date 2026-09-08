# Port-Channel Plan

## Status

**Status:** Draft  
**Last Updated:** 2026-08-30  
**Applies To:** Talos Solutions Enterprise Campus v1  
**Document Type:** Design / Build Reference  

---

## Purpose

This document defines the planned port-channel and LACP design for the Talos Solutions Enterprise Campus v1 lab.

The purpose of this document is to document which physical links are bundled together, which devices participate in each EtherChannel, what each port-channel is used for, and what trunking behavior is expected.

---

## Scope

### In Scope

- Core-to-core port-channel design
- Core-to-access port-channel design
- LACP mode
- Port-channel naming and numbering plan
- Member link relationships
- Trunk role expectations
- Native VLAN expectations
- Allowed VLAN expectations
- STP behavior expectations

### Out of Scope

- Final CML interface numbers
- Final VLAN configuration commands
- Final device configurations
- Access port configuration
- Firewall HA links
- Routed firewall/edge links
- Verification command output

---

## Summary

The Talos Solutions Enterprise Campus v1 lab uses LACP EtherChannels for redundant Layer 2 uplinks between the core pair and the access layer.

The design includes:

- One two-link LACP trunk between CORE1 and CORE2
- One two-link LACP bundle from each access switch to CORE1
- One separate two-link LACP bundle from each access switch to CORE2

Each access switch has two separate uplink port-channels:

```text
Access switch -> CORE1
Access switch -> CORE2
```

A single port-channel should not be split across CORE1 and CORE2 because this lab does not use StackWise Virtual, VSS, vPC, MLAG, or another multi-chassis EtherChannel technology.

---

## Port-Channel Standards

| Item | Standard |
|---|---|
| Negotiation protocol | LACP |
| LACP mode | Active |
| Core-to-core role | 802.1Q trunk |
| Core-to-access role | 802.1Q trunk |
| Native VLAN | VLAN 998 |
| Parking VLAN | VLAN 999 |
| DTP | Disabled where supported |
| Access switch uplink model | Separate port-channel to each core |
| Multi-chassis EtherChannel | Not used in v1 |

---

## Planned Port-Channels

| Port-Channel | Local Device | Remote Device | Member Links | Role | Notes |
|---|---|---|---:|---|---|
| Po1 | CORE1 | CORE2 | 2 | Inter-core trunk | Carries shared campus VLANs and firewall inside transit VLAN |
| Po11 | CORE1 | ASW1 | 2 | Core-to-access trunk | ASW1 uplink to CORE1 |
| Po12 | CORE2 | ASW1 | 2 | Core-to-access trunk | ASW1 uplink to CORE2 |
| Po21 | CORE1 | ASW2 | 2 | Core-to-access trunk | ASW2 uplink to CORE1 |
| Po22 | CORE2 | ASW2 | 2 | Core-to-access trunk | ASW2 uplink to CORE2 |
| Po31 | CORE1 | ASW3 | 2 | Core-to-access trunk | ASW3 uplink to CORE1 |
| Po32 | CORE2 | ASW3 | 2 | Core-to-access trunk | ASW3 uplink to CORE2 |

Port-channel numbers are locally significant, but matching the same number on both ends of a link is preferred for readability.

---

## Port-Channel Member Map

Final CML interface numbers are not assigned yet. Member interfaces should be updated after the CML topology is built.

| Port-Channel | Device A | Device A Members | Device B | Device B Members | Purpose |
|---|---|---|---|---|---|
| Po1 | CORE1 | TBD, TBD | CORE2 | TBD, TBD | Inter-core trunk |
| Po11 | CORE1 | TBD, TBD | ASW1 | TBD, TBD | ASW1 to CORE1 uplink |
| Po12 | CORE2 | TBD, TBD | ASW1 | TBD, TBD | ASW1 to CORE2 uplink |
| Po21 | CORE1 | TBD, TBD | ASW2 | TBD, TBD | ASW2 to CORE1 uplink |
| Po22 | CORE2 | TBD, TBD | ASW2 | TBD, TBD | ASW2 to CORE2 uplink |
| Po31 | CORE1 | TBD, TBD | ASW3 | TBD, TBD | ASW3 to CORE1 uplink |
| Po32 | CORE2 | TBD, TBD | ASW3 | TBD, TBD | ASW3 to CORE2 uplink |

---

## Core-to-Core Port-Channel

### Port-Channel

```text
CORE1 <-> CORE2
Port-channel: Po1
```

### Purpose

The CORE1-to-CORE2 port-channel provides the inter-core trunk.

It supports:

- Shared VLANs between CORE1 and CORE2
- HSRP operation
- STP operation
- Firewall inside transit VLAN sharing
- Access-layer failover paths
- Management VLAN reachability

### Expected Allowed VLANs

```text
10,20,30,40,50,900,998,999
```

### Not Expected

VLAN 901 should not be carried on the inter-core trunk.

VLAN 901 is the outside firewall transit VLAN and should remain local to OS1.

---

## Core-to-Access Port-Channels

### ASW1

```text
ASW1 <-> CORE1 = Po11
ASW1 <-> CORE2 = Po12
```

Purpose:

- Provide redundant Layer 2 uplinks from ASW1 to both cores
- Carry required access VLANs
- Allow STP to select the forwarding path per VLAN

### ASW2

```text
ASW2 <-> CORE1 = Po21
ASW2 <-> CORE2 = Po22
```

Purpose:

- Provide redundant Layer 2 uplinks from ASW2 to both cores
- Carry required access VLANs
- Support INFRA1 reachability through VLAN 30
- Allow STP to select the forwarding path per VLAN

### ASW3

```text
ASW3 <-> CORE1 = Po31
ASW3 <-> CORE2 = Po32
```

Purpose:

- Provide redundant Layer 2 uplinks from ASW3 to both cores
- Carry required access VLANs
- Support printer/IoT endpoint reachability
- Allow STP to select the forwarding path per VLAN

---

## Expected Core-to-Access Allowed VLANs

Common expected allowed VLANs:

```text
10,20,30,40,50,998,999
```

VLAN 900 should generally not be carried to access switches unless there is a specific design reason.

VLAN 901 should never be carried to access switches.

---

## Native VLAN

VLAN 998 is the dedicated native VLAN for trunks.

Design intent:

- Do not use VLAN 1 as the native VLAN.
- Do not place endpoints in the native VLAN.
- Do not create an SVI for VLAN 998.
- Keep trunk behavior explicit and predictable.

Expected trunk native VLAN:

```text
998
```

---

## Parking VLAN

VLAN 999 is the parking VLAN for unused or disabled access ports.

Design intent:

- Unused access ports should be assigned to VLAN 999.
- Unused ports should be administratively shut down.
- VLAN 999 should not have an SVI.
- VLAN 999 should not be used for normal endpoint connectivity.

---

## LACP Design Rules

The following rules apply to all v1 port-channels:

- Use LACP instead of static EtherChannel.
- Use `active` mode on both ends unless a specific reason exists not to.
- Member interfaces in the same bundle must have matching configuration.
- Port-channel interface configuration should match the intended trunk behavior.
- Member interfaces should not be configured differently from the port-channel.
- Each access switch must have separate port-channels to CORE1 and CORE2.
- Do not create one port-channel from an access switch split across both core switches.

---

## STP Behavior

The access layer uses traditional Layer 2 redundancy.

Because access switches have separate uplink port-channels to each core, STP controls which uplink forwards for each VLAN.

Expected behavior:

- Each access switch has a logical uplink to CORE1.
- Each access switch has a logical uplink to CORE2.
- STP blocks one redundant logical path per VLAN as needed.
- STP root placement should align with HSRP active gateway placement.
- Per-VLAN forwarding should follow the preferred core for that VLAN when possible.

STP/HSRP role alignment is documented in:

```text
docs/05-vlan-plan.md
```

---

## Port-Channel Verification Expectations

Expected verification commands may include:

```text
show etherchannel summary
show interfaces trunk
show spanning-tree vlan <vlan-id>
show lacp neighbor
show interfaces port-channel <number>
show running-config interface port-channel <number>
```

Detailed command output should be captured in the verification documents, not in this design document.

---

## Design Notes

- Port-channels reduce physical-link failure impact.
- LACP provides negotiation and consistency checking.
- The inter-core port-channel is a critical shared trunk.
- Core-to-access port-channels should be trunk links.
- The firewall HA failover/state link is not a port-channel in v1.
- Routed ISP, edge, and firewall links are not part of this port-channel plan.
- Exact member interfaces will be added after the CML topology is built.

---

## Validation or Success Criteria

The port-channel plan is successful when:

- Every planned LACP bundle is documented.
- Each port-channel has a clear purpose.
- Member link counts are documented.
- Native VLAN expectations are documented.
- Allowed VLAN expectations are documented.
- Access switches use separate uplink bundles to each core.
- No port-channel is incorrectly split across both core switches.
- Final interface numbers are added after the CML build.
- Verification confirms that all port-channels form correctly.

---

## Open Questions

- None currently identified.

---

## Related Documents

- `docs/01-initial-planning.md`
- `docs/03-topology-and-device-roles.md`
- `docs/04-design-decisions.md`
- `docs/05-vlan-plan.md`
- `docs/06-addressing-plan.md`
- `docs/07-interface-map.md`
- `docs/09-routing-plan.md`
- `docs/10-firewall-plan.md`
- `docs/12-management-plan.md`
- `verification/02-l2-verification.md`
- `verification/07-failure-testing.md`

---

## Change Log

| Date | Change |
|---|---|
| 2026-08-30 | Initial draft |