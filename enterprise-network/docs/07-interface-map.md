# Interface Map

## Status

**Status:** Draft  
**Last Updated:** 2026-08-30  
**Applies To:** Talos Solutions Enterprise Campus v1  
**Document Type:** Design / Build Reference  

---

## Purpose

This document defines the planned physical interface relationships for the Talos Solutions Enterprise Campus v1 lab.

The purpose of this document is to document which devices connect to each other, what each link is used for, and what Layer 2 or Layer 3 role each link is expected to serve.

Final CML interface numbers will be added after the topology is built and the actual interface assignments are confirmed.

---

## Scope

### In Scope

- Physical link relationships
- Link purpose
- Layer 2 / Layer 3 link role
- Firewall HA physical links
- Core interconnect links
- Access switch uplinks
- Infrastructure and endpoint links
- Interface mapping placeholders

### Out of Scope

- Final IP addressing
- Final VLAN configuration
- Port-channel configuration details
- Device configuration commands
- Verification command output
- Failure testing results

---

## Summary

The v1 topology uses a simulated ISP, single edge router, outside transit switch, active/standby ASAv firewall pair, dual collapsed core/distribution switches, three access switches, an infrastructure server, and endpoint nodes.

High-level path:

```text
ISP1 -> EDGE1 -> OS1 -> FW1/FW2 -> CORE1/CORE2 -> ASW1/ASW2/ASW3 -> Clients
```

This document tracks the physical link layout. Port-channel details are documented separately in:

```text
docs/08-port-channel-plan.md
```

---

## Interface Map Standards

Until exact CML interfaces are assigned, interface values should remain marked as `TBD`.

Example:

| Local Device | Local Interface | Remote Device | Remote Interface |
|---|---|---|---|
| CORE1 | TBD | CORE2 | TBD |

After the topology is built in CML, this document should be updated with actual interface names.

---

## Link Role Types

| Link Type | Meaning |
|---|---|
| Routed | Layer 3 point-to-point or routed handoff |
| Access | Layer 2 access port in a single VLAN |
| Trunk | Layer 2 trunk carrying multiple VLANs |
| Port-channel member | Physical member link of an EtherChannel |
| Failover/state | Dedicated firewall HA communication link |

---

## Physical Interface Map

| Link ID | Local Device | Local Interface | Remote Device | Remote Interface | Link Role | Purpose |
|---|---|---|---|---|---|---|
| L01 | ISP1 | TBD | EDGE1 | TBD | Routed | ISP-to-customer-edge transit |
| L02 | EDGE1 | TBD | OS1 | TBD | Routed handoff to access VLAN | EDGE1 connection to VLAN 901 outside firewall transit segment |
| L03 | OS1 | TBD | FW1 | TBD | Access | FW1 outside connection on outside transit VLAN |
| L04 | OS1 | TBD | FW2 | TBD | Access | FW2 outside connection on outside transit VLAN |
| L05 | FW1 | TBD | FW2 | TBD | Failover/state | Dedicated ASAv failover/state link |
| L06 | FW1 | TBD | CORE1 | TBD | Access / firewall transit | FW1 inside connection to firewall transit VLAN |
| L07 | FW2 | TBD | CORE2 | TBD | Access / firewall transit | FW2 inside connection to firewall transit VLAN |
| L08 | CORE1 | TBD | CORE2 | TBD | Port-channel member | Inter-core LACP trunk member 1 |
| L09 | CORE1 | TBD | CORE2 | TBD | Port-channel member | Inter-core LACP trunk member 2 |
| L10 | CORE1 | TBD | ASW1 | TBD | Port-channel member | ASW1 uplink to CORE1 member 1 |
| L11 | CORE1 | TBD | ASW1 | TBD | Port-channel member | ASW1 uplink to CORE1 member 2 |
| L12 | CORE2 | TBD | ASW1 | TBD | Port-channel member | ASW1 uplink to CORE2 member 1 |
| L13 | CORE2 | TBD | ASW1 | TBD | Port-channel member | ASW1 uplink to CORE2 member 2 |
| L14 | CORE1 | TBD | ASW2 | TBD | Port-channel member | ASW2 uplink to CORE1 member 1 |
| L15 | CORE1 | TBD | ASW2 | TBD | Port-channel member | ASW2 uplink to CORE1 member 2 |
| L16 | CORE2 | TBD | ASW2 | TBD | Port-channel member | ASW2 uplink to CORE2 member 1 |
| L17 | CORE2 | TBD | ASW2 | TBD | Port-channel member | ASW2 uplink to CORE2 member 2 |
| L18 | CORE1 | TBD | ASW3 | TBD | Port-channel member | ASW3 uplink to CORE1 member 1 |
| L19 | CORE1 | TBD | ASW3 | TBD | Port-channel member | ASW3 uplink to CORE1 member 2 |
| L20 | CORE2 | TBD | ASW3 | TBD | Port-channel member | ASW3 uplink to CORE2 member 1 |
| L21 | CORE2 | TBD | ASW3 | TBD | Port-channel member | ASW3 uplink to CORE2 member 2 |
| L22 | ASW2 | TBD | INFRA1 | TBD | Access | Infrastructure server connection |
| L23 | ASW1 | TBD | C1 | TBD | Access | User endpoint connection |
| L24 | ASW1 | TBD | C2 | TBD | Access | User endpoint connection |
| L25 | ASW2 | TBD | C3 | TBD | Access | Admin or user endpoint connection |
| L26 | ASW3 | TBD | C4 | TBD | Access | Endpoint connection |
| L27 | ASW3 | TBD | PRN1 | TBD | Access | Printer/IoT endpoint connection |

---

## Link Group Summary

| Link Group | Links | Purpose |
|---|---|---|
| ISP edge | L01 | Connect ISP1 to EDGE1 |
| Edge outside transit | L02 | Connect EDGE1 to the firewall outside segment |
| Firewall outside | L03-L04 | Connect FW1/FW2 outside interfaces to OS1 |
| Firewall HA | L05 | Dedicated FW1/FW2 failover/state communication |
| Firewall inside | L06-L07 | Connect FW1/FW2 inside interfaces to the core pair |
| Core interconnect | L08-L09 | Two-link LACP trunk between CORE1 and CORE2 |
| ASW1 uplinks | L10-L13 | Redundant LACP uplinks from ASW1 to both cores |
| ASW2 uplinks | L14-L17 | Redundant LACP uplinks from ASW2 to both cores |
| ASW3 uplinks | L18-L21 | Redundant LACP uplinks from ASW3 to both cores |
| Infrastructure | L22 | INFRA1 connection to server/infrastructure VLAN |
| Endpoints | L23-L27 | Client and printer/IoT endpoint connections |

---

## Expected VLAN / Network Role by Link

| Link ID | Expected Network Role | Notes |
|---|---|---|
| L01 | ISP1-to-EDGE1 routed transit | Uses ISP/edge transit addressing |
| L02 | Outside firewall transit | EDGE1 routed interface connected to OS1 outside segment |
| L03 | Outside firewall transit | OS1 access port for FW1 outside |
| L04 | Outside firewall transit | OS1 access port for FW2 outside |
| L05 | Firewall failover/state | Dedicated HA link; not used for normal routing |
| L06 | Firewall inside transit | FW1 inside to CORE1 on VLAN 900 |
| L07 | Firewall inside transit | FW2 inside to CORE2 on VLAN 900 |
| L08-L09 | Inter-core trunk | Carries shared VLANs including VLAN 900 |
| L10-L21 | Core-to-access trunks | Carry required access VLANs |
| L22 | Server/infrastructure access | Expected VLAN 30 |
| L23-L24 | User access | Expected VLAN 10 |
| L25 | Admin/user access | Expected VLAN 20 or VLAN 10 |
| L26 | Endpoint access | Expected VLAN 10 or other test VLAN |
| L27 | Printer/IoT access | Expected VLAN 40 |

---

## Firewall Interface Relationship

The firewall pair uses three major physical link types:

| Firewall Link | Purpose |
|---|---|
| Outside interface | Connects each firewall to OS1 and the outside firewall transit subnet |
| Inside interface | Connects each firewall to the core-side firewall transit VLAN |
| Failover/state interface | Connects FW1 and FW2 directly for HA communication |

Conceptual layout:

```text
EDGE1
  |
 OS1
 / \
FW1 FW2
 |   |
CORE1 CORE2
```

---

## Core and Access Interface Relationship

Each access switch has two separate uplink bundles:

```text
ASW1 -> CORE1 port-channel
ASW1 -> CORE2 port-channel
```

The access switch uplinks should not be configured as a single port-channel split across both cores.

This design is required because the lab does not use StackWise Virtual, VSS, vPC, MLAG, or another multi-chassis EtherChannel technology.

---

## Interface Mapping Notes

- Final interface numbers should be filled in after the topology is built in CML.
- Port-channel IDs should be documented in `docs/08-port-channel-plan.md`.
- VLAN details should be documented in `docs/05-vlan-plan.md`.
- IP addressing should be documented in `docs/06-addressing-plan.md`.
- Access-port VLAN assignments should be reviewed before final configuration.
- Any CML interface limitations or unexpected numbering behavior should be documented in `cml/cml-notes.md`.

---

## Design Notes

- OS1 is used as an outside transit switch, not as a user access switch.
- FW1 and FW2 connect to equivalent outside and inside networks for HA operation.
- CORE1 and CORE2 share VLAN 900 across the inter-core trunk.
- Access switches operate as traditional Layer 2 access switches in v1.
- INFRA1 connects inside the campus network, not outside the firewall.
- Endpoint placement should support DHCP, HSRP, STP, firewall, and PAT validation.

---

## Validation or Success Criteria

The interface map is successful when:

- Every physical link in the v1 topology is documented.
- Each link has a clear purpose.
- Each link has a defined Layer 2 or Layer 3 role.
- CML interface numbers are added after the topology is built.
- Port-channel member links match the port-channel plan.
- Firewall HA links are clearly separated from normal traffic links.
- Endpoint and infrastructure links are documented.
- No undocumented physical links are required for v1 operation.

---

## Open Questions

- None currently identified.

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
- `docs/12-management-plan.md`
- `docs/13-build-order.md`
- `docs/14-known-limitations.md`
- `docs/15-future-roadmap.md`

---

## Change Log

| Date | Change |
|---|---|
| 2026-08-30 | Initial draft |