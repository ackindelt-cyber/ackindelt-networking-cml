# Interface Map

## Status

**Status:** Draft  
**Last Updated:** 2026-09-10  
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
ISP1 -> TSE1 -> TSOS1 -> TSFW1/TSFW2 -> TSCOR1/TSCOR2 -> TSAS1/TSAS2/TSAS3 -> Clients
```

This document tracks the physical link layout. Port-channel details are documented separately in:

```text
docs/08-port-channel-plan.md
```
```

---

## Interface Map Standards

Until exact CML interfaces are assigned, interface values should remain marked as `TBD`.

Example:

| Local Device | Local Interface | Remote Device | Remote Interface |
|--------------|-----------------|---------------|------------------|
| TSCOR1       | TBD             | TSCOR2        | TBD              |

After the topology is built in CML, this document should be updated with actual interface names.

---

## Link Role Types

| Link Type           | Meaning                                  |
|---------------------|------------------------------------------|
| Routed              | Layer 3 point-to-point or routed handoff |
| Access              | Layer 2 access port in a single VLAN     |
| Trunk               | Layer 2 trunk carrying multiple VLANs    |
| Port-channel member | Physical member link of an EtherChannel  |
| Failover/state      | Dedicated firewall HA communication link |

---

## Physical Interface Map

| Link ID | Local Device | Local Interface | Remote Device | Remote Interface | Link Role                     | Purpose                                                       |
|---------|--------------|-----------------|---------------|------------------|-------------------------------|---------------------------------------------------------------|
| L01     | ISP1         | Gi0/0           | EDGE1         | Gi0/0            | Routed                        | ISP-to-customer-edge transit                                  |
| L02     | TSE1         | Gi0/1           | TSOS1         | Gi0/0            | Routed handoff to access VLAN | TSE1 connection to VLAN 901 outside firewall transit segment  |
| L03     | TSOS1        | Gi0/0           | TSFW1         | Gi0/0            | Access                        | TSFW1 outside connection on outside transit VLAN              |
| L04     | TSOS1        | Gi0/2           | TSFW2         | Gi0/0            | Access                        | TSFW2 outside connection on outside transit VLAN              |
| L05     | TSFW1        | Gi0/1           | TSFW2         | Gi0/1            | Failover/state                | Dedicated ASAv failover/state link                            |
| L06     | TSFW1        | Gi0/2           | TSCOR1        | Gi0/0            | Access / firewall transit     | TSFW1 inside connection to firewall transit VLAN              |
| L07     | TSFW2        | Gi0/2           | TSCOR2        | Gi0/0            | Access / firewall transit     | TSFW2 inside connection to firewall transit VLAN              |
| L08     | TSCOR1       | Gi0/1           | TSCOR2        | Gi0/1            | Port-channel member           | Inter-core LACP trunk member 1                                |
| L09     | TSCOR1       | Gi0/2           | TSCOR2        | Gi0/2            | Port-channel member           | Inter-core LACP trunk member 2                                |
| L10     | TSCOR1       | Gi0/3           | TSAS1         | Gi0/0            | Port-channel member           | TSAS1 uplink to TSCOR1 member 1                               |
| L11     | TSCOR1       | Gi1/0           | TSAS1         | Gi0/1            | Port-channel member           | TSAS1 uplink to TSCOR1 member 2                               |
| L12     | TSCOR2       | Gi0/2           | TSAS1         | Gi0/3            | Port-channel member           | TSAS1 uplink to TSCOR2 member 1                               |
| L13     | TSCOR2       | Gi0/3           | TSAS1         | Gi1/0            | Port-channel member           | TSAS1 uplink to TSCOR2 member 2                               |
| L14     | TSCOR1       | Gi1/1           | TSAS2         | Gi0/0            | Port-channel member           | TSAS2 uplink to TSCOR1 member 1                               |
| L15     | TSCOR1       | Gi1/2           | TSAS2         | Gi0/1            | Port-channel member           | TSAS2 uplink to TSCOR1 member 2                               |
| L16     | TSCOR2       | Gi1/1           | TSAS2         | Gi0/2            | Port-channel member           | TSAS2 uplink to TSCOR2 member 1                               |
| L17     | TSCOR2       | Gi0/3           | TSAS2         | Gi1/2            | Port-channel member           | TSAS2 uplink to TSCOR2 member 2                               |
| L18     | TSCOR1       | Gi1/3           | TSAS3         | Gi0/0            | Port-channel member           | TSAS3 uplink to TSCOR1 member 1                               |
| L19     | TSCOR1       | Gi2/0           | TSAS3         | Gi0/1            | Port-channel member           | TSAS3 uplink to TSCOR1 member 2                               |
| L20     | TSCOR2       | Gi1/3           | TSAS3         | Gi0/2            | Port-channel member           | TSAS3 uplink to TSCOR2 member 1                               |
| L21     | TSCOR2       | Gi2/0           | TSAS3         | Gi0/3            | Port-channel member           | TSAS3 uplink to TSCOR2 member 2                               |
| L22     | TSAS1        | Gi1/0           | TSINF1        | Gi0/0            | Access                        | Infrastructure server connection                              |
| L23     | TSAS1        | Gi1/1           | TSPC1         | Ens2             | Access                        | User endpoint connection                                      |
| L24     | TSAS2        | Gi1/0           | TSPC2         | Ens2             | Access                        | User endpoint connection                                      |
| L25     | TSAS3        | Gi1/0           | TSPC3         | Ens2             | Access                        | Admin or user endpoint connection                             |

---

## Link Group Summary

| Link Group           | Links   | Purpose                                                |
|----------------------|---------|--------------------------------------------------------|
| ISP edge             | L01     | Connect ISP1 to TSE1                                  |
| Edge outside transit | L02     | Connect TSE1 to the firewall outside segment          |
| Firewall outside     | L03-L04 | Connect TSFW1/TSFW2 outside interfaces to TSOS1       |
| Firewall HA          | L05     | Dedicated TSFW1/TSFW2 failover/state communication    |
| Firewall inside      | L06-L07 | Connect TSFW1/TSFW2 inside interfaces to the core pair|
| Core interconnect    | L08-L09 | Two-link LACP trunk between TSCOR1 and TSCOR2         |
| TSAS1 uplinks        | L10-L13 | Redundant LACP uplinks from TSAS1 to both cores       |
| TSAS2 uplinks        | L14-L17 | Redundant LACP uplinks from TSAS2 to both cores       |
| TSAS3 uplinks        | L18-L21 | Redundant LACP uplinks from TSAS3 to both cores       |
| Infrastructure       | L22     | TSINF1 connection to server/infrastructure VLAN       |
| Endpoints            | L23-L25 | TSPC1-TSPC3 endpoint connections                      |

---

## Expected VLAN / Network Role by Link

| Link ID | Expected Network Role        | Notes                                                        |
|---------|------------------------------|--------------------------------------------------------------|
| L01     | ISP1-to-TSE1 routed transit  | Uses ISP/edge transit addressing                             |
| L02     | Outside firewall transit     | TSE1 routed interface connected to TSOS1 outside segment     |
| L03     | Outside firewall transit     | TSOS1 access port for TSFW1 outside                          |
| L04     | Outside firewall transit     | TSOS1 access port for TSFW2 outside                          |
| L05     | Firewall failover/state      | Dedicated HA link; not used for normal routing               |
| L06     | Firewall inside transit      | TSFW1 inside to TSCOR1 on VLAN 900                           |
| L07     | Firewall inside transit      | TSFW2 inside to TSCOR2 on VLAN 900                           |
| L08-L09 | Inter-core trunk             | Carries shared VLANs including VLAN 900                      |
| L10-L21 | Core-to-access trunks        | Carry required access VLANs                                  |
| L22     | Server/infrastructure access | Expected VLAN 30                                             |
| L23     | User access                  | Expected VLAN 10                                             |
| L24     | User access                  | Expected VLAN 10                                             |
| L25     | Admin/user access            | Expected VLAN 20 or VLAN 10                                  |

---

## Firewall Interface Relationship

The firewall pair uses three major physical link types:

| Firewall Link           | Purpose                                                              |
|-------------------------|----------------------------------------------------------------------|
| Outside interface       | Connects each firewall to TSOS1 and the outside firewall transit subnet |
| Inside interface        | Connects each firewall to the core-side firewall transit VLAN         |
| Failover/state interface| Connects TSFW1 and TSFW2 directly for HA communication                |

Conceptual layout:

```text
TSE1
  |
TSOS1
 /   \
TSFW1 TSFW2
  |     |
TSCOR1 TSCOR2
```

---

## Core and Access Interface Relationship

Each access switch has two separate uplink bundles:

```text
TSAS1 -> TSCOR1 port-channel
TSAS1 -> TSCOR2 port-channel
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

- TSOS1 is used as an outside transit switch, not as a user access switch.
- TSFW1 and TSFW2 connect to equivalent outside and inside networks for HA operation.
- TSCOR1 and TSCOR2 share VLAN 900 across the inter-core trunk.
- TSAS1, TSAS2, and TSAS3 operate as traditional Layer 2 access switches in v1.
- TSINF1 connects inside the campus network, not outside the firewall.
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